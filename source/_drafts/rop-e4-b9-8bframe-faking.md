---
layout: post
title: rop之stack pivot
categories:
  - ctf
  - pwn
url: 385.html
id: 385
comments: false
date: 2016-10-03 18:48:57
tags:
---

frame faking是一种伪造ebp，利用leave，ret指令返回到一段已知的可写地址的技术，这种利用方式的优点在于并不需要比较大的空间。下面具体描述了这种利用方式的构造姿势：

< \- stack grows this way
   addresses grow this way  - >
                        
                       saved FP   saved vuln. function's return address
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
| buffer fill-up(*) | fake_ebp0 | leaveret | 
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
                         |
   \+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\+         (*) this time, buffer fill-up must not
   |                                   overwrite the saved frame pointer !
   v
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
| fake\_ebp1 | f1 | leaveret | f1\_arg1 | f1_arg2 ...                     
-----|-----------------------------------------
     |                       the first frame
     +-+
       |
       v
     ------------------------------------------------
     | fake\_ebp2 | f2 | leaveret | f2\_arg1 | f2_argv2 ...
     -----|------------------------------------------
          |                  the second frame  
          \+\-\- ...

  fake\_ebp0 should be the address of the "first frame", fake\_ebp1 - the
address of the second frame, etc.

上面为x86下frame faking的利用方式，x64下稍有不同，需要利用到构造参数的rop，这部分内容已经在[linux x64 rop利用总结](http://skysider.com/?p=371)中进行了详细讲述。 上面的利用方式需要提前写好相应frame的内容，后来想到一种可以一次完成getshell的变形：

< \- stack grows this way
   addresses grow this way  - >
                        
                       saved FP   saved vuln. function's return address
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
| buffer fill-up(*) | fake\_ebp0 | f0 | leaveret | f0\_arg1 | f0\_arg2 | f0\_arg3
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
                         |
   \+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\+         (*) this time, buffer fill-up must not
   |                                   overwrite the saved frame pointer !
   v
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
| fake\_ebp1 | f1 | leaveret | f1\_arg1 | f1_arg2 ...                     
-----|-----------------------------------------
     |                       the first frame
     +-+
       |
       v
     ------------------------------------------------
     | fake\_ebp2 | f2 | leaveret | f2\_arg1 | f2_argv2 ...
     -----|------------------------------------------
          |                  the second frame  
          \+\-\- ...

  fake\_ebp0 should be the address of the "first frame", fake\_ebp1 - the
address of the second frame, etc.

二者的区别在于缓冲区溢出时调用了函数f0，然后返回到fake ebp0指向的区域，f0可以是read的地址，那我们就可以完成后面若干frame的数据写入，然后直接转到第一个frame开始执行，x64架构下由于函数传参使用寄存器，所以我们不需要多次采用frame faking，只需要采用一次frame faking即可，x64下简化的frame faking：

< \- stack grows this way
   addresses grow this way  - >
                        
                       saved FP   saved vuln. function's return address
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
| buffer fill-up(*) | fake_rbp0 | rop f0 | leaveret
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-|\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
                         |
   \+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\+         (*) this time, buffer fill-up must not
   |                                   overwrite the saved frame pointer !
   v
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-
| fake_rbp1 | rop1 | rop2 | rop3 | ...                     
\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-

这种利用方式要求在f0调用过程中并不会修改rbp的值，由于x64架构下采用三参数函数调用会改变rbp的值，所以我们需要对函数略作修改： ![](http://skysider.com/wp-content/uploads/2016/10/Screen-Shot-2017-04-25-at-8.39.43-PM-300x127.png)

def three\_param\_func\_payload(setreg\_rop, func\_addr\_addr, param1, param2, param3, callptr_rop, rbp=None):
    payload = p64(setreg_rop)
    payload += p64(0)
    payload += p64(1)
    payload += p64(func\_addr\_addr)
    payload += p64(param3)
    payload += p64(param2)
    payload += p64(param1)
    payload += p64(callptr_rop)
    if rbp:
        payload += 'a'*(8*2)
        payload += p64(rbp)
        payload += 'a'*(8*4)
    else:
        payload += 'a'*(8*7)
    return payload

payload = three\_param\_func_payload(0x40567a, binary.got\['read'\], 0, bss, 1024, 0x405660, bss)
payload += gadget("leave, ret")

增加了rbp变量，使它的值为fake_rbp0的值。 =========================================================================== 还有一个方法可以直接改变rsp的值，利用

pop rsp; pop r13; pop r14; pop r15; ret

来改变rsp的值，此时rsp+8*3指向下一个rop gadget，此时可以节省r13、r14、r15的空间