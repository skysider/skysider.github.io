---
layout: post
title: pwn中system调用失败分析
tags:
  - pwn
  - getshell
categories:
  - ctf
date: 2017-04-20 23:44:27
---


在ctf比赛中，有时调试一个pwn题目，发现直到调用system函数、传参时都是对的，但是system函数会执行失败，就是无法拿到shell，在这里总结了一下可能的原因：

1.  在调用system函数时，esp指针指向的区域前面不存在一定空间的可写数据区，原因是在函数执行过程中，会维护自己的栈帧(sub esp, xxxx) —— fake frame时需要注意，会触发`__libc_sigaction`错误，fault address
<!-- more -->
2.  system函数的调用流程：system -> do_system->execve，execve函数执行时，会有三个参数:
```
    __execve (SHELL_PATH, (char *const *) new_argv, __environ);
```
    其中，
```
    SHELL_PATH = "/bin/bash";
    const char *new_argv[4];
    new_argv[0] = SHELL_NAME; // "sh"
    new_argv[1] = "-c";
    new_argv[2] = line;
    new_argv[3] = NULL;

    environ="HOME=skysider" // or ""
```
当environ指向的栈数据被无效数据覆盖时，就会调用失败。因此可以采用gdb动态调试的方法，若发现system函数能够执行到execve函数，可以观察此时execve的几个参数值是否正常，若异常，就可以去寻找对应的原因。

一种解决的方法是不调用system函数，而是调用execve函数，在调用时指定environ为NULL即可，即 `execve("/bin/sh", 0, 0)` 调试技巧：
```
b system
b execve
stack // 查看堆栈数据,SHELL_PATH和environ
x/s *(char**)(*(char **)($esp+8)+8) // 查看 line
```
![getshell.png](https://i.loli.net/2018/05/12/5af6ccf26068d.png)
