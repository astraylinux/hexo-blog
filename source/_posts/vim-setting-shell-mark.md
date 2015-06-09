title: VIM配置语言简记
date: 2014-06-05 14:40:01
categories:
- VIM
tags:
- VIM
- edit
- config
---

一直在用vim，以前找了一些简单的资料配置了一下，并没有仔细去了解这它设置的内容。这次特地花了一些时间学习。仔细学习，才更觉得vim的强大，光是说明文档，应该就够整一本厚厚的说明书了。vim配置相关的文件主要有vimrc和.vim目录下的扩展文件,vimrc的配置就是一门比较完的脚本语言。

vimrc的配置是一种脚本语言，可以导入其他配置、定义变量、执行程序、编写函数，vim的插件就是用这个脚本语言编写的。脚本的语法可以从[http://vimcdoc.sourceforge.net/doc/usr_41.html](http://vimcdoc.sourceforge.net/doc/usr_41.html)学习，中文版的。简要提一下一些要点。vim语法比较奇怪的注释用`"`开头。


<!--more-->

###变量
变量使用let赋值,unlet删除
vim也是弱类型的语言, 也有支持数组与字典，具体看文档。
```vim
let s:count = 1
let g:index = 1
```
变量名前面可以带表示作用范围的符号
b:name          缓冲区的局部变量
w:name          窗口的局部变量
g:name          全局变量 (也用于函数中)
v:name          Vim 预定义的变量

在命令模式输入let，可以查看当前已定义的变量。

###表达式、条件和循环
表达式没什么特别的，跟其他语言都差不多，一些细节还是要查查文档。
条件和循环是以end加关键字结束的，比如 endif

条件，if判断可以有括号也可以没有
```vim
if {condition}  
    {statements}
endif

if &term == 'xterm'
  ' Do something'
endif
"或
if (&term == 'xterm')
  ' Do something'
endif
```

循环，模式都那样。
```vim
while counter < 40
    "do"
endwhile

for a in range(3)
  echo a
endfor
```

###函数
函数定义的格式
```vim
function {name}({var1}, {var2}, ...)
  {body}
endfunction
```
例子
```vim
function Min(num1, num2)
  if a:num1 < a:num2
    let smaller = a:num1
  else
    let smaller = a:num2
  endif
  return smaller
endfunction
```
函数的调用可以直接像其他语言那样,比如min(1,2)
也可以用call命令调用call min(1,2), 使用call的话，还可以实现范围调用

当一个函数定义时给出了
"range" 关键字时，表示它会自行处理该范围。
 Vim 在调用这样一个函数时给它传递两个参数: "a:firstline" 和"a:lastline"，用来表示该范围所包括的第一行和最后一行。例如:
```vim
function Count_words() range
  let lnum = a:firstline
  let n = 0
  while lnum <= a:lastline
    let n = n + len(split(getline(lnum)))
    let lnum = lnum + 1
  endwhile
  echo "found " . n . " words"
endfunction
```
你可以这样调用上面的函数:
```vim
:10,30call Count_words()
```
另一种使用范围的方式是在定义函数时不给出 "range" 关键字。Vim 将把光标移动到范围内的每一行，并分别对该行调用此函数。例如:
```vim
:function  Number()
:  echo "line " . line(".") . " contains: " . getline(".")
:endfunction
```
如果你用下面的方式调用该函数:
```vim
:10,15call Number()
```
它将被执行六次。

另外如果要重定义一个已经存在的函数，在 "function" 命令后加上 !:
function!  Min(num1, num2, num3)

在命令行输入function可以获得已定义函数的清单

它也支持面向对象编程（这货还是编辑器吗。。。), 不是编写插件的话，一般也是用不着的。

###编写插件
vim插件即是使用上面的脚本语言写成的脚本程序，以`.vim`结尾。把编写好的vim脚本放到.vim目录下的plugin里，vim启动时就会加载了。vim脚本一般会通过在vimrc里的变量来设置。

vim有两种类型的插件(主要是作用范围不一样):
全局插件: 适用于所有类型的文件。
文件类型插件: 仅适用于某种类型的文件。

编写方法:[http://vimcdoc.sourceforge.net/doc/usr_41.html#41.11](http://vimcdoc.sourceforge.net/doc/usr_41.html#41.11)



