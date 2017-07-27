title: 我的Linux命令行艺术
tags:
  - linux
  - cmd
categories:
  - linux
date: 2017-07-26 23:51:12

----------

### 命令
```bash
#临时http服务器
python -m SimpleHTTPServer 7777   #python2
python -m http.server 7777    #python3

mtr --report ip #检查网络路由情况
uniq -u  #(只显示不重复的行 -d(只显示重复的行)
rpm -qf <文件名>   #查看命令在哪个rpm包
rpm -qa  #查看所有安装的包
netstat -lntp #检查端口监听情况
#通过使用 <(some command) 可以将输出视为文件。例如
diff /etc/hosts <(ssh somehost cat /etc/hosts)

if [[ ]]， 中>, <, =, >=, <=会把两边当会字符串处理， 而-gt, -lt, -eq, -ge, -le则把两边当数字。
```
<!--more-->

### 命令行艺术里的命令安装
* **silversearcher-ag**: ag命令，用来查找代码里的文字，结果显示更友好
* **lynx-cur**: lynx -dump html_file， 将html转为文本，提取内容 
* **pandoc**: Markdown，HTML，以及所有文档格式之间的转换
* **xmlstarlet**: 处理xml文件
* **jq**: 将很乱的json字符串格式化显示  cat test.json |jq .    （有个点）
* **ncdu**: du命令的改版,很好用

### 设置
* 命令行下可以执行 set -o vi 来使用 vi 风格的快捷键，而执行 set -o emacs 可以把它改回来（可在.bashrc中设置）
* 为了便于编辑长命令，在设置你的默认编辑器后（例如 export EDITOR=vim），ctrl-x ctrl-e 会打开一个编辑器来编辑当前输入的命令。在 vi 风格下快捷键则是 escape-v
* 在 Bash 脚本中，使用 set -x 去调试输出（或者使用它的变体 set -v，它会记录原始输入，包括多余的参数和注释）。尽可能地使用严格模式：使用 set -e 令脚本在发生错误时退出而不是继续运行；使用 set -u 来检查是否使用了未赋值的变量；试试 set -o pipefail，它可以监测管道中的错误。当牵扯到很多脚本时，使用 trap 来检测 ERR 和 EXIT。一个好的习惯是在脚本文件开头这样写，这会使它能够检测一些错误，并在错误发生时中断程序并输出信息：
```bash
      set -euo pipefail
      trap "echo 'error: Script failed: see failed command above'" ERR
```



### 新安装的Ubuntu
|应用|开发|
|---|---|
|Chrome|jdk|
|virtualbox|git|
|remmina|npm|
|Terminator|eclipse|
|meld diff||
|Ksnapshot||
|GIMP||
```bash
sudo apt-get install nload virtualbox google-chrome terminator remmina git npm
#去掉zeitgeist的执行权限，如果要从侧边栏去掉东西 ，再给权限执行一下就好了
```

中文本地化后 会安装2个字体  fonts-arphic-ukai 和 fonts-arphic-uming, 都删掉
```bash
sudo apt-get remove fonts-arphic-ukai fonts-arphic-uming
```

### 新安装的centos
```bash
#安装软件
yum install vim git python-setuptools gcc autoconf automake autoconf213 unzip ncdu
#开发常用包
yum install pcre-devel xz-devel libpcap-devel glib2-devel ncurses-devel
#编译安装package
iftop  ag  proxychains4
#python软件
yum install python-devel python2-pip
easy_install shadowsocks virtualenv glances

```

### 需安装服务
```
1. lnmp  # Nginx、Mysql、PHP
2. redis
3. shadowsocks  # Client
4. memcached
5. gitlab
```

