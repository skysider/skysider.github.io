---
layout: post
title: CentOS 安装 cx_Freeze
categories:
  - 开发
url: 53.html
id: 53
comments: false
date: 2015-07-25 19:46:22
tags:
---

前言：之前有一个项目要用到cx_Freeze打包python3的脚本，而且需要测试linux不同服务器的发行版，主要是Ubuntu和CentOS，后来发现在CentOS编译生成的文件基本可以直接在Ubuntu下运行，CentOS为了追求稳定性所以一些库的版本比较低，而Ubuntu的库的版本相对较高，所以自然是向下兼容的。 这里我测试过的CentOS版本有6.4, 6.5, 6.6以及7 首先，第一步肯定是安装Python3

wget https://www.python.org/ftp/python/3.4.3/Python-3.4.3.tgz
yum groupinstall "Development tools"
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel

其次，便是利用Python3提供的pip来安装cx_Freeze

ln -s libpython3.4m.a libpython3.4.a 
pip3 install cx_Freeze

需要注意的是，如果不执行第一条指令，cx_Freeze会安装失败，查看记录，会发现失败的原因是找不到python3.4库文件，后来发现python3.4的库文件改成libpython3.4m.a了，所以gcc通过lpython3.4是找不到的，我们只需要建立一个软链接即可