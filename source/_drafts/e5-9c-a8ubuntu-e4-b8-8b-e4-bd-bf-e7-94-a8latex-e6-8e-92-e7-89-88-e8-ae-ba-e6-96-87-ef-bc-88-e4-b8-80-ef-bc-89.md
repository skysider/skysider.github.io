---
layout: post
title: 在ubuntu下使用latex排版论文（一）
categories:
  - 论文
url: 28.html
id: 28
comments: false
date: 2015-07-25 19:59:03
tags:
---

本篇主要介绍一下在ubuntu下如何安装latex以及增加对中文的支持。 在ubuntu下安装latex，直接用apt就可以搞定：

sudo apt-get install texlive-full

这条命令会让你安装1.4G左右的东西，实际上texlive的镜像就是这么大，如果你的网速快的话，最多几十分钟就可以搞定，慢的话，你就先去做别的事吧。 安装完成之后，你可以运行命令

tex --version

你可以看到你安装的tex的版本，我的版本是2013。 [![2015-07-25 20:03:07的屏幕截图](http://skysider.com/wp-content/uploads/2015/07/2015-07-25-200307的屏幕截图-300x111.png)](http://skysider.com/wp-content/uploads/2015/07/2015-07-25-200307的屏幕截图.png) 安装结束之后，如果要使用中文字体，由于ctex默认使用的windows字体，所以需要从windows系统中将字体拷出来，建议放到/usr/share/fonts/windows目录下，注意一下执行 ll 命令查看字体文件是否是可读的，否则会出现字体文件找不到的情况，可以执行

chmod 755 /usr/share/fonts/windows/*

然后你需要执行以下命令来让系统知道新字体的存在

cd /usr/share/fonts/windows
sudo mkfontscale 
sudo mkfontdir 
sudo fc-cache -fv

运行结束之后，可以运行命令

fc-list :lang=zh

查看字体是否已经安装成功 tex已经安装成功，接下来如何编辑tex文档呢？我们可以执行

sudo apt-get install texmaker

安装成功之后，我们就可以使用taxmaker来编辑tex文档了 [![2015-07-25 20:23:27的屏幕截图](http://skysider.com/wp-content/uploads/2015/07/2015-07-25-202327的屏幕截图.png)](http://skysider.com/wp-content/uploads/2015/07/2015-07-25-202327的屏幕截图.png) 使用xelatex进行编译，就可以生成pdf了 [![2015-07-25 20:20:30的屏幕截图](http://skysider.com/wp-content/uploads/2015/07/2015-07-25-202030的屏幕截图.png)](http://skysider.com/wp-content/uploads/2015/07/2015-07-25-202030的屏幕截图.png) 如果要使用texmaker的快速构建生成包含中文的pdf文档，就需要对texmaker进行配置

选项->配置TexMaker->快速构建->XeLatex+View PDF