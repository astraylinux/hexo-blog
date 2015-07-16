title: Mongodb插入及更新速度优化
tags:
  - server
  - mongo
  - optimization
  - C
  - python
categories:
  - server
date: 2015-07-06 11:29:44
keywords: Mongo optimize update insert 优化 更新
---

继续之前的数据导入流程的优化，最后的Mongodb导入部分成为了最大的瓶颈。在我们的测试机中upsert操作有2-3k/s的更新速度，而到线上，访问机器和服务机器不是同一台时，速度只剩下1.5k/s，完全不能满足更新的要求。只好再研究研究Mongo的性能及优化。

<!--more-->
![](http://res.astraylinux.com/server/mongo_upsert_optimization.png)

##Insert速度
Insert对mongo来说压力较小，不过如果还是每条数据都请求一次服务器的话，性能不是很好。好在Mongo有提供批量插入的接口，尝试每次五百条数据插入，就算是不同机器，速度也在7k左右，是可以接受的。插入性能并不是优化的重点。

##Python接口
导入程序是用python写的，python的性能是比较差的，我一开始并不认为语言会成为瓶颈，直到我尝试用多线程，多线程下导入性能甚至不如单线程。谷歌一下，得到如下解释：
>在多核机器上，我们期望多线程的代码使用额外的核，从而提高整体性能。不幸的是，主Python解释器（CPython）的内部并不是真正的多线程，是通过一个全局解释锁（GIL）来进行处理的。

>GIL是必须的，因为Python解释器是非线程安全的。这意味着当从线程内尝试安全的访问Python对象的时候将有一个全局的强制锁。在任何时候，仅仅一个单一的线程能够获取Python对象或者C API。每100个字节的Python指令解释器将重新获取锁，这（潜在的）阻塞了I/0操作。因为锁，CPU密集型的代码使用线程库时，不会获得性能的提高，但是当它使用多处理库时，性能可以获得提高。

也就是说，python并不支持真正的多线程，只是通过跳跃执行的代码模仿多线程，也就是说每个python程序只能使用一个CPU核心。这种机制在处理一些CPU密集的操作时并不会有性能上的提升，而在一些需要等待的慢处理时，就能得到性能上的提升。

也就是说，用python导入时，CPU的消耗多少决定了多线程的效果。通过测试发现，在导入时python单线程的CPU使用率在单核的60-80%，也就是说，两个线程需要120-160%的CPU，但是python并不能使用多个CPU核心，加上多线程调度的消耗，导致了多线程性能不如单线程的窘境。虽然多进程可以解决这个问题，不过CPU的消耗和多进程的管理成本又是个大问题。

##C语言接口
Python在效率这块，确实不能令人满意，只好转投C。关于使用C语言操作mongo，我在[上一篇](http://astraylinux.com/2015/07/03/server-use-mongo-c-driver/)博文已经写过了, 主要是使用"mongo_c_driver"来连接和更新数据，并将代码生成.so文件，由python调用执行。

使用C语言代码，单线程CPU使用率降到了20-30%，并且可以使用多CPU核心，大大提高了速度。本机测试时没有带宽压力，单线程可以达5k+/s的更新速度，在5个线程时都是倍数增长，达到了25k/s。不过也要注意Mongo的服务端压力，五个线程时服务程序的CPU使用率已经高达300-400%，必需做好压力控制。

最后将包含C代码的导入程序上线，在线上可能由于网络消耗，开启3个线程也只能达到5k+的速度，算是可以接受的程度。

##考虑用全部重新插入代替更新
由于数据插入的速度远快于更新速度，所以用插入代替更新也是值得考虑的方法。按之前测试的结果来看，批量插入数据的速度可以达到更新的3倍以上，且CPU压力更小。因此，如果导入的数据量达到了库里数据的三分之一的话，可以使用批量插入的方式生成一个新表，再取代原来的表。不过，这么做的前提是旧数据可以被新数据完全取代，否则丢失了数据就得不偿失了。

##补充
在stackflow上问了一下，才发现犯了大错，MongoDB在2.x较后的版本都有支持批量操作。使用mongodb的bulk类收集数据并批量发送。详见[Bulk Write Operations](http://docs.mongodb.org/manual/core/bulk-write-operations/).

有我的一段报告在[stackflow](http://stackoverflow.com/questions/31379185/could-mongodb-send-multiply-update-commands-at-once)上。按照[Blakes Seven](http://stackoverflow.com/users/5031275/blakes-seven)说的，仔细看了bulk的说明.
>Bulk write operations can be either ordered or unordered. With an ordered list of operations, MongoDB executes the operations serially. If an error occurs during the processing of one of the write operations, MongoDB will return without processing any remaining write operations in the list.

>With an unordered list of operations, MongoDB can execute the operations in parallel. If an error occurs during the processing of one of the write operations, MongoDB will continue to process remaining write operations in the list.

>Executing an ordered list of operations on a sharded collection will generally be slower than executing an unordered list since with an ordered list, each operation must wait for the previous operation to finish.

注意bulk是有有序和无序之分，有序的bulk在服务器上是以队列方式执行的， 无序则是多线程并行执行的。对我的项目来说，要更新的数据都是不相关的，可以使用无序的方式执行。
使用bulk，就连python都能达到7k/s以上的速度。用C实现，更是达到了单线程一万多。已经超过了php接口导入redis的速度了。
