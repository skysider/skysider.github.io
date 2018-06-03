---
layout: post
title: python换源
categories:
  - python
url: 363.html
id: 363
comments: false
date: 2016-09-29 11:38:00
tags:
---

**临时使用：**

pip install pythonModuleName -i http://pypi.douban.com --trusted-host=pypi.douban.com

**永久使用：** 修改配置文件：linux的文件在~/.pip/pip.conf，windows在%HOMEPATH%\\pip\\pip.ini），修改内容为：

\[global\]
index-url = https://pypi.douban.com/simple
trusted-host = pypi.douban.com