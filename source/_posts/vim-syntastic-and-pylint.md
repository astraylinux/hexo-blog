title: VIM语法检查插件Syntastic和pylint的安装配置
date: 2014-03-05 14:40:01
categories:
- VIM
tags:
- VIM
- edit
- config
- install
- tools
---

一直在用vim写脚本，写代码，却一直没有想过它可以像其他的IDE那样，有语法检查。最近在写一些库的程序，在搜索程序规范的时候无意间发现了[Syntastic](https://github.com/scrooloose/syntastic)这个vim插件。通过这个插件和它的扩展包，可以实现对各个语言的语法检查支持，严格还有代码规范的警告，非常适合用来写代码库。

效果
![](https://github.com/scrooloose/syntastic/raw/master/_assets/screenshot_1.png)
<!--more-->
分别在几个位置给出了错误提示:
1.用location list 列出所有错误。
2.命令行窗口显示当前错误。
3.错误标记，有警告和错误。
4.鼠标悬停可以出现错误提示框
5.状态栏标记。

[语法检测包](https://github.com/scrooloose/syntastic/wiki/Syntax-Checkers)

安装Syntastic,可以选择用[Vundle](https://github.com/gmarik/Vundle.vim)或[pathogen](https://github.com/tpope/vim-pathogen)来安装Syntastic，git上是用pathogen

##使用pathogen安装Syntastic
```bash
mkdir -p ~/.vim/autolscrooloose/syntasticoad ~/.vim/bundle && \
curl -LSso ~/.vim/autoload/pathogen.vim https://tpo.pe/pathogen.vim
```
然后在.vimrc上加入一行,加在靠前的位置，如果不行就放到最后面。
```bash
execute pathogen#infect()
```
然后进入./vim/bundle安装syntastic
```bash
cd ~/.vim/bundle && \
git clone https://github.com/scrooloose/syntastic.git
```
接着打开一个新的vim，在命令模式下输入Helptags，如果有报错的话，检查以下几个地方：

1.有同时创建~/.vim/autoload和~/.vim/bundle两个目录
2.有添加`execute pathogen#infect()`到～/.vimrc文件中
3.成功地通过git下载了syntastic
4.对以所提到的所有目录都有读写权限


##使用Vundle安装Syntastic
我个人比较喜欢vundle管理插件，非常方便，还可以写成脚本，通过命令批量安装。这就便利管理插件非常方便，你可以在git上放vim的配置和vundle的配置文件，然后用一个安装脚本执行，就可以在新电脑上配置好vim。
安装vundle也很方便
```bash
git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```
接着，可以打开一个新的vim，在命令模式下执行搜索与安装
```bash
:BundleSearch syntastic
#会有一个新的buffer显示搜索到的插件，光标移动到那个上面点`i`就可以安装了
```
也可以在.vimrc中加入配置，使用命令安装（这种方式方便做插件收集，还不用保存插件文件，推荐）
```vim
set nocompatible    " configure Vundle
filetype off        " required! turn off
set rtp+=~/.vim/bundle/vundle/
call vundle#rc()

" 使用Vundle来管理插件
" vim plugin bundle control, command model
"     :BundleInstall     install 安装配置的插件
"     :BundleInstall!    update  更新
"     :BundleClean       remove plugin not in list 删除本地无用插件
Bundle 'gmarik/vundle'

" syntastic，vundle是从git上下载插件的，配置中scrooloose是作者名，后面是repo
Bundle 'scrooloose/syntastic'
```
安装syntastic,打开一个新的vim运行
```bash
:BundleInstall
```
或者在命令行运行
```bash
vim +BundleInstall +qall
```

建议新建一个配置文件~/.vimrc.bundles，在vimrc中包含进去，然后在这个配置文件中写上面的配置。这样可以在里面放上其他你需要的插件，通过一个文件统一管理。
```vim
if filereadable(expand("~/.vimrc.bundles"))
  source ~/.vimrc.bundles
endif
```
安装的时候要指定配置文件
```bash
vim -u ~/.vimrc.bundles +BundleInstall +qall
```

##配置Syntastic
在vimrc中，或者插件管理的配置文件中加入下面配置
```vim
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*

let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0
```
这样算是开启了syntastic,不过默认的syntastic和语法支持包并不多，很多要自己安装。可以在[Syntax Checkers](https://github.com/scrooloose/syntastic/wiki/Syntax-Checkers)中找到语法插件。


##配置python语法检查，pylint
最近主要是写python程序，因此安装了pylint, 可以用vundle安装。
然后在配置文件中加入
```vim
let g:syntastic_python_checkers=['pylint']
let g:syntastic_python_pylint_args='--disable=C0111,R0903,C0301'
```
这个时候用vim打开py文件，就有像图片那样的效果了。另外，pylint检查的东西太多，是比较慢的，要求没那么高的，可以使用[pyflakes](https://github.com/kevinw/pyflakes-vim)或者[pep8](https://github.com/jcrocholl/pep8),安装和配置方法都是一样的。

后面那个配置是指要pylint忽略哪些提示, pylint的提示非常严谨，包括语法风格问题都会提示。不喜欢的提示可以用配置去。提示信息格式为:
```bash
MESSAGE_TYPE: LINE_NUM:[OBJECT:] MESSAGE
```
MESSAGE_TYPE 有如下几种：
(C) 惯例。违反了编码风格标准
(R) 重构。写得非常糟糕的代码。
(W) 警告。某些 Python 特定的问题。
(E) 错误。很可能是代码中的错误。
(F) 致命错误。阻止 Pylint 进一步运行的错误。


pylint还有更个性的配置，可以安装pylint命令，通过命令生成pylintrc配置文件，放到/etc/目录下或~/.pylintrc,里面有很多配置，包括语法识别的正则。
```bash
 pylint --generate-rcfile > /etc/pylintrc
```

不过作者是建议不要使用pylintrc，严格地按规范写代码。
如果安装了pylint命令，用命令检查代码，它会给出一份报告，并给代码评一个分数，最高10分。
```bash
$ pylint test.py
Report
======
78 statements analysed.

Messages by category
--------------------

+-----------+-------+---------+-----------+
|type       |number |previous |difference |
+===========+=======+=========+===========+
|convention |0      |1        |-1.00      |
+-----------+-------+---------+-----------+
|refactor   |0      |0        |=          |
+-----------+-------+---------+-----------+
|warning    |0      |0        |=          |
+-----------+-------+---------+-----------+
|error      |0      |0        |=          |
+-----------+-------+---------+-----------+



Global evaluation
-----------------
Your code has been rated at 10.00/10

Statistics by type
------------------

+---------+-------+-----------+-----------+------------+---------+
|type     |number |old number |difference |%documented |%badname |
+=========+=======+===========+===========+============+=========+
|module   |1      |1          |=          |100.00      |0.00     |
+---------+-------+-----------+-----------+------------+---------+
|class    |0      |0          |=          |0           |0        |
+---------+-------+-----------+-----------+------------+---------+
|method   |0      |0          |=          |0           |0        |
+---------+-------+-----------+-----------+------------+---------+
|function |8      |8          |=          |0.00        |0.00     |
+---------+-------+-----------+-----------+------------+---------+



Duplication
-----------

+-------------------------+------+---------+-----------+
|                         |now   |previous |difference |
+=========================+======+=========+===========+
|nb duplicated lines      |0     |0        |=          |
+-------------------------+------+---------+-----------+
|percent duplicated lines |0.000 |0.000    |=          |
+-------------------------+------+---------+-----------+



External dependencies
---------------------
::

    config (test)
    pylib
      \-sql (test)
    spiderlib
      \-net (test)
      \-tools (test)



Raw metrics
-----------

+----------+-------+------+---------+-----------+
|type      |number |%     |previous |difference |
+==========+=======+======+=========+===========+
|code      |75     |72.82 |NC       |NC         |
+----------+-------+------+---------+-----------+
|docstring |4      |3.88  |NC       |NC         |
+----------+-------+------+---------+-----------+
|comment   |2      |1.94  |NC       |NC         |
+----------+-------+------+---------+-----------+
|empty     |22     |21.36 |NC       |NC         |
+----------+-------+------+---------+-----------+


```
