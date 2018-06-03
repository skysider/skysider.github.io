---
layout: post
title: 一个用于CTF PWN的docker容器
categories:
  - ctf
date: 2017-02-21 12:20:05
tags:
- pwn
- docker
---

### 1. 介绍

一个基于phusion/baseimage的docker容器，用于ctf中的Pwn类题目

### 2. 使用
```
docker run  -it \
            --rm \
            -h ${ctf_name} \
            --name ${ctf_name} \
            -v $(pwd)/${ctf_name}:/ctf/work \
            -p 23946:23946 \
            --cap-add=SYS_PTRACE \
            skysider/pwndocker
```
<!-- more -->
### 3. 包含的软件

- [pwntools](https://github.com/Gallopsled/pwntools)  —— CTF framework and exploit development library
- [pwndbg](https://github.com/pwndbg/pwndbg)  —— a GDB plug-in that makes debugging with GDB suck less, with a focus on features needed by low-level software developers, hardware hackers, reverse-engineers and exploit developers
- [ROPgadget](https://github.com/JonathanSalwan/ROPgadget)  —— facilitate ROP exploitation tool
- [roputils](https://github.com/inaz2/roputils) 	—— A Return-oriented Programming toolkit
- [one_gadget](https://github.com/david942j/one_gadget) —— A searching one-gadget of execve('/bin/sh', NULL, NULL) tool for amd64 and i386
- [angr](https://github.com/angr/angr)   ——  A platform-agnostic binary analysis framework
- [radare2](https://github.com/radare/radare2) ——  A rewrite from scratch of radare in order to provide a set of libraries and tools to work with binary files
- [LibcSearcher](https://github.com/lieanu/LibcSearcher) —— A libc search tool based on leaked function address
- linux_server[64] 	—— IDA 7.0 debug server for linux
-  [tmux](https://tmux.github.io/) 	—— a terminal multiplexer
-  [ltrace](https://linux.die.net/man/1/ltrace)	—— trace library function call
- [strace](https://linux.die.net/man/1/strace) —— trace system call
