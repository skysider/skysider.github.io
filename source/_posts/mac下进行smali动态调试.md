---
layout: post
title: mac下进行smali动态调试
categories:
  - 逆向
date: 2017-07-16 14:36:35
tags:
- 调试
- android
- smali
---

### 第一种方法（jeb）

1.  在模拟器或者真机中安装并运行apk
2.  以调试模式启动应用
```
    adb shell dumpsys activity top | head // 获取package name和activity name
    adb shell am start -D -n packagename/.activityname
```
3.  在jeb中设置断点，点击debug，选择设备和对应的调试进程
<!-- more -->

### 第二种方法（Android Studio + Smalidea）

1.  使用apktool或者Android Crack Tool解包apk
2.  将解包的工程导入到Android Studio中
3.  以调试模式启动应用
```
    adb shell am start -D -n packagename/.MainActivity
```
4.  启动monitor，选中调试应用，开启8700端口
5.  在smali中设置断点，并且设置远程调试（Run->Edit Configurations-> + -> remote，设置端口为8700），点击debug
