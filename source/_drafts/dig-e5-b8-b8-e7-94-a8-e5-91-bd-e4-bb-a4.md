---
layout: post
title: dig常用命令
categories:
  - 安全
url: 108.html
id: 108
comments: false
date: 2015-10-09 17:14:21
tags:
---

dig testfire.net dig testfire.net MS  // 查找域名服务器信息 dig tetfire.net @8.8.8.8 // 使用指定域名服务器进行查询 dig testfire.net A +noall +answer //域名地址查询，输出简略信息 dig axfr testfire.net @usc3.akam.net  // dns区域传送 dig -x 8.8.8.8 // 反向解析 dig -x 8.8.8.8 +short // 简略信息