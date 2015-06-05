title: bash命令提示符个性设置
date: 2012-07-16 13:30:00
categories:
- Linux
tags:
- Bash
- Linux
- config
---

昨天看到一篇关于命令提示符的设置文章，原文地址[8个实用而有趣Bash命令提示行](http://coolshell.cn/articles/1399.html)，今天试了一下，确实很有意思。自己做了一下组织，可以更方便地设置，并总结一下。
查找了一些相关资料，关于bash下颜色设置的参考了以前转载的文章[linux终端中输出彩色字体](http://blog.csdn.net/astraylinux/article/details/6991623)。
<!--more-->

##命令提示符参数
| 符号 | 效果 |
| ---- | ---- |
| \\! | 显示该命令的历史记录编号。|
| \# | 显示当前命令的命令编号。|　
| \\$ | 显示$符作为提示符，如果用户是root的话，则显示#号。　|
| \\ | 显示反斜杠。　　　|
| \\d | 显示当前日期。　　　|
| \\h | 显示主机名。　　　
| \\n | 打印新行。　|
| \\n | 显示nnn的八进制值。　　　|
| \\s | 显示当前运行的shell的名字。　|
| \\t | 显示当前时间。　|
| \\u | 显示当前用户的用户名。　|
| \\W | 显示当前工作目录的名字。　|
| \\w | 显示当前工作目录的完全路径。|

##提示自定义
本来我也是跟着那篇文章那样设置，实在是太乱了，想要自己设置一些功能，结果越改越乱，为了实现笑脸功能和长格式显示，变成这样：

```bash
PS1="\`if [ \$? = 0 ]; then echo \[\e[1\;35m\]\(^_^\)\[\e[0m\]; else echo \[\e[1\;31m\]\(\>_\<\)\[\e[0m\]; fi\`\[\e[35;1m\]\$(/bin/date)\[\e[32m\] : \w\n\[\e[1;36m\](0_0)\u@\h: \[\e[1;34m\]\$(/usr/bin/tty | /bin/sed -e 's:/dev/::'): \[\e[1;36m\]F:\$(/bin/ls -1 | /usr/bin/wc -l | /bin/sed 's: ::g') \[\e[1;33m\]\$(/bin/ls -lah | /usr/bin/head -n 1 | /usr/bin/cut -d ' ' -f 2)b ]\[\e[0m\] \\$ \[\e[0m\]"
```

弄得头疼，后来想想，这也是bash，可以用shell，可以用变量来设置，就清楚多了：　　

```bash
GREEN="\[\e[32;1m\]" 
WHITE="\[\e[37;1m\]"
BLACK="\[\e[30;1m\]"
RED="\[\e[31;1m\]"
YELLOW="\[\e[33;1m\]"
BLUE="\[\e[34;1m\]"
PURPLE="\[\e[35;1m\]"
DARK_GREEN="\[\e[36;1m\]"
CLEAR="\[\e[0m\]"


_JOB_="jobs:\j"
_FILECOUNT_="F:\$(/bin/ls -1 | /usr/bin/wc -l | /bin/sed 's: ::g')"
_FACE_="\`if [ \$? = 0 ]; then echo \[\e[1\;35m\]^_^\[\e[0m\]; else echo \[\e[1\;31m\]\>_\<\[\e[0m\]; fi\`"
_HISTORY_="\!"
_DATE_="\$(/bin/date)"
_CLOCK_="\t"
_SIZE_="\$(/bin/ls -lah | /usr/bin/head -n 1 | /usr/bin/cut -d ' ' -f 2)"

PS1="$GREEN($WHITE\u@\h$GREEN)-($WHITE$_JOB_$GREEN)-($WHITE$_HISTORY_$GREEN)-($WHITE\w$GREEN)\n($WHITE$_FILECOUNT_$GREEN)\\$ $CLEAR"

```

显示了用户名，主机名，后台程序数，命令历史行数，完全路径及当前文件夹下的文件总数，效果：
![](http://my.csdn.net/uploads/201207/16/1342415252_2980.png)
再进一步，可以把颜色设置的也组合在一起，效果与上图相同。

```bash
JOB="$GREEN($WHITE$_JOB_$GREEN)"
FILECOUNT="$GREEN($WHITE$_FILECOUNT_$GREEN)"
MYPATH="$GREEN($WHITE\w$GREEN)"
HISTORY="$GREEN($WHITE$_HISTORY_$GREEN)"
USER_HOST="$GREEN($WHITE\u@\h$GREEN)"

PS1="$USER_HOST-$JOB-$HISTORY-$MYPATH\n$FILECOUNT\\$ $CLEAR"

```

晒晒我最后用的效果(^_^)
![](http://my.csdn.net/uploads/201207/16/1342416924_1302.png)




