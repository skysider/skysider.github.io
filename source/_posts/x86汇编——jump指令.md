---
layout: post
title: x86汇编——jmp指令
categories:
  - 逆向
date: 2017-05-25 00:19:11
tags:
- 汇编
---

### 相对地址跳转

| 伪代码         | 机器码               | 示例                   |
| -------------- | -------------------- | ---------------------- |
| jmp short s    | eb+offset（1个字节） | eb03 ，ebfd            |
| jmp near ptr s | e9+offset（4个字节） | e996000000, e964ffffff |

<!-- more -->
### 绝对地址跳转

| 位数 | 伪代码                  | 机器码                  | 示例                     |
| ---- | ----------------------- | ----------------------- | ------------------------ |
| 32   | push addr; jmp esp;     | 68+addr(4个字节 )+ffe4  | 68afbeaddeffe4           |
| 64   | mov rax, addr; jmp rax; | 48b8+addr(8个字节)+ffe0 | 48b8afbeaddeafbeafdeffe0 |
