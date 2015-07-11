title: Install and config Hexo (simple)
date: 2015-06-02 18:10:01
categories: Linux
tags:
  - install
  - Linux
  - config
  - server
keywords: Install and config Hexo, 安装和配置Hexo. linux, 博客, 技术
---

The process of my Hexo with github installation!
OS: Centos 7 64bit in Linode vps

## Step 1 Install

We need npm(Node Package Manager) to install Hexo.

``` bash
 yum install npm
 mkdir hexo
 cd hexo
 npm install hexo --save
 npm install hexo-server --save
 npm install hexo-deployer-git --save
 #use hexo start
 hexo init  
 npm install  #I not run this command at first, Hexo has "Can not Get/" error
 hexo g       #generate
 hexo s	      #server
```

If hexo work well, you can through http://youhost:4000 to open it.

<!--more-->

## Step 2 Config with github

We should create a repository name like "astraylinux.github.io" (astraylinux is mine)
Then config the _config.yml

``` bash
 deploy:
   type: git
   repo: ssh://git@github.com/astraylinux/astraylinux.github.io
   branch: master
```

run command

``` bash
 hexo g
 hexo d  #deploy
```

Now http://astraylinux.github.io working.

## Step 3 config themes

I found great themes:
``` bash
 https://github.com/litten/hexo-theme-yilia
 https://github.com/iissnan/hexo-theme-next
```

``` bash
 git clone https://github.com/iissnan/hexo-theme-next themes/next
```






