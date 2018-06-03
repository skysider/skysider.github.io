---
layout: post
title: Mac下进行android so动态调试
tags:
  - 调试
  - android
categories:
  - 逆向
date: 2017-05-07 20:00:31
---


### 环境：

*   MacOS（宿主机）
    *   android-studio(包含adb、monitor、jdb等工具)
    *   ida pro(包含android_server)


*   Android模拟器或手机(android）
    *   待调试app

<!-- more -->
#### 注意：

1.  如果是模拟器进行调试，ro.debuggable默认为1，不需要二次打包
2.  如果是真机进行调试，有2种比较方便的方法
    *   对app进行二次打包，修改`AndroidManifest.xml`中`application`，添加`android:debuggable="true"`
    *   安装xposed框架（需要root，刷第三方recovery），之后安装xinstaller模块，设置xinstaller启动专家模式，在其他设置中开启“调试应用”

### 运行：

#### 1.  运行android_server（默认开启23946端口）
```
adb push android_server /data/local/tmp/
adb shell
su
chmod 777 /data/local/tmp/android_server
/data/local/tmp/android_server
```
#### 2.  以debug模式启动程序
```
adb shell dumpsys activity top | head # 查看top activity信息，作为下面-n的参数
adb shell am start -D -n com.yaotong.crackme/.MainActivity
```
#### 3. 开启ddms

monitor，选中要调试的app（开启8700端口）

![monitor.png](https://i.loli.net/2018/05/12/5af6ccf276286.png)

#### 4. ida attach target app and suspend on libary loading，F9继续运行

#### 5. 用jdb将app恢复执行
```
jdb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=8700
```
#### 6. add breakpoint at JNI_OnLoad

### 参考：

1.  http://wooyun.jozxing.cc/static/drops/tips-6840.html
2.  http://wooyun.jozxing.cc/static/drops/mobile-5942.html
