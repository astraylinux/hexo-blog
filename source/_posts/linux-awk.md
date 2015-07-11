title: awk笔记和应用
tags:
  - Linux
  - tools
  - Bash
  - program
categories:
  - Linux
date: 2014-02-11 15:29:44
keywords: Linux bash awk sed 脚本 服务器
---

以前解决问题都是直接写程序，最开始只会用C和C++，后来学会了python，可以更快地写一些小程序。不过，在处理文件，分析文件等操作时，awk经常能更好地满足需求，开发速度也更快。
awk是很强大的工具，可能它本身看来只能做一些统计之类的工作，可是当你将它与python或其他编程语言结合使用的话，常常会有令人惊喜的效果。

<!--more-->
  
  
  
  
  
  
  
##简介（wiki）

>AWK是一种优良的文本处理工具，Linux及Unix环境中现有的功能最强大的数据处理引擎之一。这种编程及数据操作语言（其名称得自于它的创始人阿尔佛雷德·艾侯、彼得·溫伯格和布萊恩·柯林漢姓氏的首个字母）的最大功能取决于一个人所拥有的知识。AWK提供了极其强大的功能：可以进行正则表达式的匹配，样式装入、流控制、数学运算符、进程控制语句甚至于内置的变量和函数。它具备了一个完整的语言所应具有的几乎所有精美特性。实际上AWK的确拥有自己的语言：AWK程序设计语言，三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。gawk是AWK的GNU版本。
最简单地说，AWK是一种用于处理文本的编程语言工具。AWK在很多方面类似于Unix shell编程语言，尽管AWK具有完全属于其本身的语法。它的设计思想来源于SNOBOL4、sed、Marc Rochkind设计的有效性语言、语言工具yacc和lex，当然还从C语言中获取了一些优秀的思想。在最初创造AWK时，其目的是用于文本处理，并且这种语言的基础是，只要在输入数据中有模式匹配，就执行一系列指令。该实用工具扫描文件中的每一行，查找与命令行中所给定内容相匹配的模式。如果发现匹配内容，则进行下一个编程步骤。如果找不到匹配内容，则继续处理下一行。
  
  
##调用方式

* 命令行方式
`awk [-F  field-separator]  'commands'  input-file(s)`
其中，commands 是真正awk命令，[-F域分隔符]是可选的，默认是空格。 input-file(s) 是待处理的文件。在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。

* shell脚本方式
将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。相当于shell脚本首行的：#!/bin/sh换成：#!/bin/awk

* 用awk命令调用脚本
`awk -f awk-script-file input-file(s)`
其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。

  
##语法
awk [ -F re] [parameter...] ['pattern {action}'] [-f progfile][in_file...] 

* -F re：允许awk更改其字段分隔符。\$0代表整行字符串，\$1、\$2、\$3...$N 代表第X个字段。

* parameter：该参数帮助为不同的变量赋值。 

* 'pattern {action}'：awk的程序语句段。这个语句段必须用单拓号：'和'括起，以防被shell解释。其中pattern参数可以是egrep正则表达式中的任何一个，它可以使用语法/re/再加上一些样式匹配技巧构成。与sed类似，你也可以使用","分开两样式以选择某个范围。action参数总是被大括号包围，它由一系统awk语句组成，各语句之间用";"分隔。awk解释它们，并在pattern给定的样式匹配的记录上执行其操作。与shell类似，你也可以使用“#”作为注释符，它使“#”到行尾的内容成为注释，在解释执行时，它们将被忽略。你可以省略pattern和action之一，但不能两者同时省略，当省略pattern时没有样式匹配，表示对所有行（记录）均执行操作，省略action时执行缺省的操作――在标准输出上显示。 

* -f progfile： 允许awk调用并执行progfile指定有程序文件。progfile是一个文本文件，他必须符合awk的语法。 

* in_file：awk的输入文件，awk允许对多个输入文件进行处理。值得注意的是awk不修改输入文件。如果未指定输入文件，awk将接受标准输入，并将结果显示在标准输出上。awk支持输入输出重定向。

例:显示文本文件myfile中第七行到第十五行中以字符%分隔的第一字段，第三字段和第七字段： 

awk -F % 'NR==7,NR==15 {print $1 $3 $7}'  myfile

  
##流程控制
awk的流程控制语句跟其他语言基本相同。 for语句支持in语法。
```bash
#============== if else
if(表达式)
    语句1;
else
    语句2;

#============== while
while(表达式)
    语句;

#============== do while
do {
    语句;
}while(条件判断语句）

#============== for
for(初始表达式;终止条件;步长表达式) {
    语句;
}

for( key in array){
    语句;
}
```
awk的判断除了常见的大小，相等的比较外，还支持正则

* x~re ：x匹配正则表达式re
* x!~re：x不匹配正则表达式re


  
##BEGIN和END:
在awk中两个特别的表达式，BEGIN和END，这两者都可用于pattern中（参考前面的awk语法），提供BEGIN和END的作用是给程序赋予初始状态和在程序结束之后执行一些扫尾的工作。任何在BEGIN之后列出的操作（在{}内）将在awk开始扫描输入之前执行，而END之后列出的操作将在扫描完全部的输入之后执行。因此，通常使用BEGIN来显示变量和预置（初始化）变量，使用END来输出最终结果。
例：统计历史命令中使用率最高的几个命令
```bash
history|awk '
{
    a[$2]+=1
}END{
    for(x in a)
        print x"\t"a[x]
}'|sort -rnk 2|head
```

  
##定义函数
定义和调用用户自己的函数是几乎每个高级语言都具有的功能，awk也不例外，但原始的awk并不提供函数功能，只有在nawk或较新的awk版本中才可以增加函数。

函数的使用包含两部分：函数的定义与函数调用。其中函数定义又包括要执行的代码（函数本身）和从主程序代码传递到该函数的临时调用。

awk函数的定义方法如下：
```bash
function 函数名(参数表){
    函数体
}
```
例:还是统计命令使用率，加上定义一个函数计算使用率
```bash
history|awk '
{
    a[$2] += 1;
    sum += 1;
}END{
    for(x in a)
        print x"\t"a[x]"\t"rate(a[x], sum);
}

function rate(num, sum){
    return num/sum;
}'|sort -rnk 2|head
```

  
##获取外部变量

获取普通的外部变量
```bash
# a="test argument"
# echo |awk '{print a}' a="$a"
test argument
```
格式如：awk ‘{action}’  变量名="变量值"
可以在action中获得值, 注意：变量名与值放到’{action}’后面。这种变量在：BEGIN的action不能获得。

BEGIN程序块中变量
```bash
# a="test argument"
# echo | awk -v a="$a" 'BEGIN{print a}'
test argument
# echo | awk -v a="$a" '{print a}'
test argument
```
格式如：awk –v 变量名=变量值 [–v 变量2=值2 …] 'BEGIN{action}’
注意：用-v 传入变量可以在3中类型的action 中都可以获得到，但顺序在  action前面。

获得环境变量
```bash
echo |awk '{for(i in ENVIRON){print i"="ENVIRON[i]}}'
```
只需要调用：awk内置变量 ENVIRON,就可以直接获得环境变量。它是一个字典数组。环境变量名 就是它的键值。

  
##在awk中执行shell命令
system()是一个不适合字符或数字类型的嵌入函数，该函数的功能是处理作为参数传递给它的字符串。system对这个参数的处理就是将其作为命令处理，也就是说将其当作命令行一样加以执行。这使得用户在自己的awk程序需要时可以灵活地执行命令或脚本。

  
##内建变量
AWK的内建变量包括域变量，例如\$1, \$2, \$3，以及\$0。这些变量给出了记录中域的内容。 内建变量也包括一些其他变量：

|变量 | 含义 | 缺省值 |
| ---- | ---- | ---- |
|ARGIND|当前被处理文件的ARGV标志符||
|CONVFMT|数字转换格式|%.6g|
|FIELDWIDTHS|输入字段宽度的空白分隔字符串||
|FILENAME|当前输入文件的名字||
|FS|输入字段分隔符|空格|
|IGNORECASE|控制大小写敏感|0（大小写敏感）|
|NF|当前记录中的字段个数||
|NR|已经读出的记录数||
|OFMT|数字的输出格式|%.6g |
|OFS|输出字段分隔符|空格 |
|ORS|输出的记录分隔符|新行 |
|RS|输入的记录他隔符|新行 |

除了内建变量，我们还可以自己定义变量，命名规则与C语言基本一致。awk的变量是没有类型的，可以像python那样使用一般数据类型，甚至还有list和map。这些可以看详细的awk文档。


  
##awk的常规表达式元字符
用于匹配pattern(正则表达式)

|符号|含义|
| ---- | ---- |
|^| 在字符串的开头开始匹配
|$| 在字符串的结尾开始匹配
|.| 与任何单个字符串匹配
|[ABC]| 与[]内的任一字符匹配
|[A-Ca-c]| 与A-C及a-c范围内的字符匹配（按字母表顺序）
|[^ABC] |与除[]内的所有字符以外的任一字符匹配
|Desk I Chair |与Desk和Chair中的任一个匹配
|[ABC][DEF]| 关联。与A、B、C中的任一字符匹配，且其后要跟D、E、F中的任一个字符。
|*| 与A、B或C中任一个出现0次或多次的字符相匹配
|+ |与A、B或C中任何一个出现1次或多次的字符相匹配
|？ |与一个空串或A、B或C在任何一个字符相匹配
|（Blue I Black）berry| 合并常规表达式，与Blueberry或Blackberry相匹配

  
##应用实例, 分析nginx日志
这里我的一个access.log的例子
>66.249.64.147 - - [07/Jul/2015:22:40:49 +0000] "GET /tags/training/ HTTP/1.1" 200 2508 "-" "Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)" "-" 0.000 -

统计接口的访问量，并按访问量排序，并记录平均响应时间
```bash
tail -n 10000 access.log| awk -F"\"" '{
    split($2,lst," ");
    url=lst[2];
    split($0,lst2," ");
    tt=lst2[length(lst2)];
    split(url,lst1,"?");
    a[lst1[1]]+=1;
    b[lst1[1]]+=tt;
}END{
    for(x in a)
        print x"\t"a[x]"\t"b[x]/a[x]
}'|sort -t'	' +1rg -2
```

统计接口各种返回状态出现的次数
```bash
tail -n 10000 access.log|awk -F'T /' '{print $2}'|awk '{
    split($1,lst,"?");
    a[lst[1]"\t"$3]+=1;
}END{
    for(x in a)
        print x"\t"a[x]
}'|sort
```

统计每分钟的访问量
```bash
tail -n 5000 access.log|awk '{a[$4]+=1}END{for(x in a)print x"\t"a[x]}'|sort
```

  
##awk的内置函数

| 函数 | 用途或返回值 |
| ---- | ---- |
|atan2( y, x )}|	返回 y/x 的反正切。
|cos( x )|	返回 x 的余弦；x 是弧度。
|sin( x )|	返回 x 的正弦；x 是弧度。
|exp( x )|	返回 x 幂函数。
|log( x )|	返回 x 的自然对数。
|sqrt( x )	|返回 x 平方根。
|int( x )|	返回 x 的截断至整数的值。
|rand( )|	返回任意数字 n，其中 0 <= n < 1。
|srand( [Expr] )|	将 rand 函数的种子值设置为 Expr 参数的值，或如果省略 Expr 参数则使用某天的时间。返回先前的种子值。
|gsub( Ere, Repl, [ In ] )|除了正则表达式所有具体值被替代这点，它和 sub 函数完全一样地执行，。
|sub( Ere, Repl, [ In ] )|	用 Repl 参数指定的字符串替换 In 参数指定的字符串中的由 Ere 参数指定的扩展正则表达式的第一个具体值。sub 函数返回替换的数量。出现在 Repl 参数指定的字符串中的 &（和符号）由 In 参数指定的与 Ere 参数的指定的扩展正则表达式匹配的字符串替换。如果未指定 In 参数，缺省值是整个记录（$0 记录变量）。
|index( String1, String2 )|	在由 String1 参数指定的字符串（其中有出现 String2 指定的参数）中，返回位置，从 1 开始编号。如果 String2 参数不在 String1 参数中出现，则返回 0（零）。
|length [(String)]|	返回 String 参数指定的字符串的长度（字符形式）。如果未给出 String 参数，则返回整个记录的长度（$0 记录变量）。
|blength [(String)]	|返回 String 参数指定的字符串的长度（以字节为单位）。如果未给出 String 参数，则返回整个记录的长度（$0 记录变量）。
|substr( String, M, [ N ] )	|返回具有 N 参数指定的字符数量子串。子串从 String 参数指定的字符串取得，其字符以 M 参数指定的位置开始。M 参数指定为将 String 参数中的第一个字符作为编号 1。如果未指定 N 参数，则子串的长度将是 M 参数指定的位置到 String 参数的末尾 的长度。
|match( String, Ere )|	在 String 参数指定的字符串（Ere 参数指定的扩展正则表达式出现在其中）中返回位置（字符形式），从 1 开始编号，或如果 Ere 参数不出现，则返回 0（零）。RSTART 特殊变量设置为返回值。RLENGTH 特殊变量设置为匹配的字符串的长度，或如果未找到任何匹配，则设置为 -1（负一）。
|split( String, A, [Ere] )|	将 String 参数指定的参数分割为数组元素 A[1], A[2], . . ., A[n]，并返回 n 变量的值。此分隔可以通过 Ere 参数指定的扩展正则表达式进行，或用当前字段分隔符（FS 特殊变量）来进行（如果没有给出 Ere 参数）。除非上下文指明特定的元素还应具有一个数字值，否则 A 数组中的元素用字符串值来创建。
|tolower( String )|	返回 String 参数指定的字符串，字符串中每个大写字符将更改为小写。大写和小写的映射由当前语言环境的 LC_CTYPE 范畴定义。
|toupper( String )|	返回 String 参数指定的字符串，字符串中每个小写字符将更改为大写。大写和小写的映射由当前语言环境的 LC_CTYPE 范畴定义。
|sprintf(Format, Expr, Expr, . . . )|	根据 Format 参数指定的 printf 子例程格式字符串来格式化 Expr 参数指定的表达式并返回最后生成的字符串。
|close( Expression )|用同一个带字符串值的 Expression 参数来关闭由 print 或 printf 语句打开的或调用 getline 函数打开的文件或管道。如果文件或管道成功关闭，则返回0；其它情况下返回非零值。如果打算写一个文件，并稍后在同一个程序中读取文件，则 close 语句是必需的。
|system(Command )|执行 Command 参数指定的命令，并返回退出状态。等同于 system 子例程。
|mktime( YYYY MM DD HH MM SS[ DST])|生成时间格式|
|strftime([format [, timestamp]])|	格式化时间输出，将时间戳转为时间字符串 |
|systime()|	得到时间戳,返回从1970年1月1日开始到当前时间(不计闰年)的整秒数|
