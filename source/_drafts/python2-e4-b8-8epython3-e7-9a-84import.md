---
layout: post
title: python2与python3的import
categories:
  - 安全
url: 177.html
id: 177
comments: false
date: 2016-04-01 09:40:29
tags:
---

首先，要明确一点的是，不管是python2还是python3，如果脚本里使用了相对路径，都无法直接运行该模块。如果要单独运行该模块，需要使用 python -m module_name 这种方式。 python3导入时，采用绝对路径。 python2导入时，采用相对路径。 python3竟然支持中文！