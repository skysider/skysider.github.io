---
layout: post
title: windbg常用命令
categories: 逆向
comments: false
date: 2017-07-29 15:41:14
tags:
- 调试
- windbg
- windows
---

### 命令介绍

windbg支持三种类型的命令，**标准命令**、**元命令**和**扩展命令**。 标准命令提供最基本的调试功能，不区分大小写，如`k`，`g`，`dt`，`bp`等 元命令提供标准命令没有提供的功能，也内建在调试引擎中，以 "**.**" 开头，如`.sympath `, `.reload`等 扩展命令用于扩展某一方面的调试功能，实现在动态加载的扩展模块中，以 **!** 开头，如 `!analyze`等（需要将第三方dll文件放到winext目录，使用时先用`.load xxx.dll`加载，然后使用`!xxx `使用扩展模块功能）。
<!-- more -->
### 查看帮助

*   `?` —— 打印出所有标准命令
*   `.help` —— 打印出所有元命令
*   `.chain` —— 给出一个扩展命令集的链表
*   `!<module_name>.help` —— 查看扩展模块帮助
*   `.hh` —— 打开windbg的chm帮助文件

### 断点相关

*   `bp <address>` —— 设置软件断点，针对某个地址，例如 `bp c87fb320`, `bp edgehtml!ProcessCSSText`，当`edgehtml!ProcessCSSText`位置发生变化时，断点的位置不变
*   `bu <symbol>` ——设置延迟断点，针对符号，例如 `bu edgehtml!ProcessCSSText`，当符号地址变化时，对应的断点地址也会变化
*   `bm <reg>` —— 设置符号断点，支持匹配表达式，例如 `bm edgehtml!Process*`
*   `ba <access> <size> <addr>` —— 设置处理器断点，access包括 e（执行）、r（读）、w（写），例如 `ba w4 0xcccccccc`
*   `bl` —— 列出所有断点
*   `be <index>` —— 激活指定编号断点
*   `bd <index>` —— 禁止断点
*   `bc <index>` —— 清除断点

### 读写/搜索内存

*   `!address` —— 查看进程的所有内存页属性
    *   `!address —summary`，显示进程的内存统计信息
    *   `!address 7ffd8000` ，查看7ffd8000地址处内存页属性
*   `d <type> <address range>` ——根据指定的类型查看存储在某地址中的数据
    *   `da` —— 显示ASCII字符，每行最多显示48字符，例如 `da rip`, `da rip L4`, `da rip rip+16`
    *   `db` —— 显示字节值和ASCII字符
    *   `dw` —— 显示字值（2字节）
    *   `dd` —— 显示双字，默认长度为32 Dwords，`dd poi(ebp+4)`，poi——解引用指针
    *   `dD` —— 显示双精度浮点数（8字节），默认 15 个数字
    *   `df` —— 显示单精度浮点数（4字节），默认 16个数字
    *   `dq` —— 显示四字值（8字节）
    *   `du` —— 显示Unicode字符串
    *   `ds` —— 显示ASCII字符串
*   `d<type>s <address range>` —— 打印地址上的二进制值，同时搜索符号信息
    *   `dds 0xdeafbeaf L20` —— 打印0xdeafbeaf开始的0x20个双字二进制值，并检索符号
    *   `dqs 0xdeadbeafdeadbeaf `—— 打印0xdeafbeaf开始的16（默认值）个四字二进制值，并检索符号
*   `e <type> <address> <value>` —— 修改指定内存中的数据
    *   `ea 0x445634 "abc"` ——在0x445634地址写入ASCII字符串abc，不包含结束符0
    *   `eza 0x445634 "abc"` —— 在0x445634地址写入ASCII字符串abc， 包含结束符0
    *   `eu 0x445634 “abc”` —— 在0x445634地址写入Unicode字符串abc，不包含结束符0
    *   `ezu 0x445634 “abc” `—— 在0x445634地址写入Unicode字符串abc，包含结束符0
    *   `ed nCounter 80` —— 修改变量nCounter的值为80
    *   `ew 00007ff9a9ddfc06 cc` —— 修改00007ff9\`a9ddfc06处的双字节为0x00cc
*   `.writemem <file> <address range>` —— 将指定内存的内容写入文件中
    *   `.writemem D:\\Test\\0041a5e4.bin 0041a5e4 L1000` ，将内存地址处0x0041a5e4后面0x1000长度的内容拷贝存储到D:\\Test\\0041a5e4.bin中
*   `S  \[<options>\] <range> <values>` —— 搜索内存
    *   `s -w 55230000 L0x100 0x1212 0x2212` ，在起始地址0x55230000之后的0x100个单位内搜索0x1212 0x2212 0x1234系列的起始地址
    *   `s -u 52230000 52270000 "web"` ， 在55230000和55270000之间搜索Unicode字符串“web”
    *   `s -d 55230000 L0x100 0xdeadbeaf`，在起始地址0x55230000之后的0x100个单位内搜索0xdeadbeaf

### 读写寄存器

*   `r [[<reg> [= <expr>]]] `—— 查看或设置寄存器
    *   `r` —— 查看所有寄存器值
    *   `r eax` —— 查看eax值
    *   `r rax = rip` —— 设置rax的值为rip值

### 符号加载与查看

*   `.symopt` —— 显示所有符号选项
*   `.reload` —— 重载符号表
*   `ld *` —— 加载模块的符号信息
    *   `ld *`  —— 为所有模块加载符号信息
    *   `ld kernel32` —— 为kernel32加载符号信息
*   `x [<*|module>!]<*|symbol>` —— 查看符号信息
    *   `x *!` ，列出所有模块的符号信息
    *   `x edgehtml!`，列出edgehtml模块的符号信息
    *   `x edgehtml!CDOM*`，列出edgehtml模块中所有以CDOM开始的符号信息
*   `lm` —— 列出所有模块的信息
*   `lmv m ntdll `—— 查看ntdll的加载信息（简略）
*   `lmvm <module name>` —— 查看指定模块的详细信息
*   `!dlls -l `—— 按照加载顺序列出所有加载模块
*   `!dlls -c <function_name>` —— 查找函数所在模块
*   `ln <addr>` —— 查看地址addr处或附近的符号信息

### 调用堆栈

*   `k <num>`—— 显示当前调用堆栈
    *   `k 5`，显示最近5层函数调用信息
    *   `kb 4`，打印出前3个函数参数的当前调用堆栈
    *   `kD`，从当前esp（rsp）开始，向高地址方向搜索符号（等价于 dds esp或dqs rsp）
*   `.frame` —— 显示当前栈帧
    *   `.frame 4` —— 显示编号为n的栈帧
*   `!uniqstack` —— 显示所有线程的调用堆栈

### 查看堆

*   `!heap -s` —— 显示进程堆的个数
*   `dt _HEAP 001400000`  —— 选取一个堆地址，打印该堆的内存结构
*   `!heap -a 001400000` —— 选取一个堆地址，打印堆结构

### 调试执行

*   `g`——继续执行
    *   `gH`，强制让调试器返回已经处理了这个异常
    *   `gN`，强制让调试器返回没有处理这个异常
    *   `gu`，执行到当前函数完成时停下，遇到ret指令停下
*   `wt` —— Trace and watch data， 在函数起始地址处执行该命令，跟踪并打印该函数内部调用过程
*   `ctrl+break` —— 暂停正在运行的程序
*   `p` —— 单步执行（step over)
    *   `p 2`，单步执行2条指令
    *   `pc` (step to next call)， 执行到下一个函数调用处停下
    *   `pa 7c801b0b` ( step to address)，执行到7c801b0b处停下
    *   `pt`，step到下一条ret指令
*   `t `—— 单步步入（step into）
    *   `tc` —— 执行到下一个call指令处停下
    *   `ta 7c801b0b`，执行到7c801b0b处停下
    *   `tb`，执行到分支指令处（calls、returns、jumps、loops）停下
    *   `tt`，trace到下一条ret指令

### 查看汇编

*   `u`  —— 反汇编
    *   `u` ，反汇编当前ip寄存器地址后的8条指令
    *   `ub` ，反汇编当前ip寄存器地址的前8条指令
    *   `u main+0x29 L30`，反汇编main+0x29地址的后30条指令
    *   `uf CTest::add` ，反汇编CTest类的add函数
    *   `uf /c main`，查看main中的函数调用有哪些

### 查看数据类型与局部变量

*   `dt` —— 打印类型信息
    *   `dt ntdll!\_IMAGE\_DOS\_HEADER`，打印ntdll中的\_IMAGE\_DOS\_HEADER结构
    *   `dt nRet`，打印局部变量nRet的类型与值
    *   `dt myApp!g\_app`，打印myApp进程里全局变量g\_app的内存布局
    *   `dt WindbgTest!CTest 0x0041f8b4`，将0x0041f8d4地址处内容按照模块WindbgTest的CTest结构来解析
    *   `dt -b -r3 <structure>`，-b 递归显示所有子类型信息，-r指定递归显示深度
*   `dv `—— 显示局部变量

### 诊断

*   `!analyze -v `—— 详细显示当前异常信息
*   `!analyze -hang` —— 诊断线程调用栈上是否有任何线程阻塞了其他线程
*   `!analyze -f` —— 查看异常分析信息，尽管调试器未诊断出异常

### 进程与线程

*   `!peb` —— 格式化输出PEB（Process Environment Block）信息
*   `!teb` —— 格式化输出TEB（Thread Environment Block）信息
*   `!tls -1` —— 显示当前线程的所有slot信息
*   `|` —— 列出所有调试进程
    *   `|N`，查看序号为N的调试进程
    *   `|Ns`，切换序号为N的进程为当前调试进程
*   `~` —— 列出所有线程
    *   `~*k` —— 列出所有线程堆栈信息
    *   `~.` —— 查看当前线程
    *   `~0` —— 查看主线程
    *   `~#`  —— 查看导致当前事件或异常的线程
    *   `~N` —— 查看N号线程
    *   `~Ns` —— 切换序号为N的线程为当前调试线程
    *   `~Nf` —— 冻结序号为N的线程
    *   `~Nu` —— 解冻序号为N的线程
    *   `~Nn` —— Suspend序号为N的线程
    *   `~Nm` —— Resume序号为N的线程

### 参考：

1.  http://www.cnblogs.com/kekec/archive/2012/12/02/2798020.html
