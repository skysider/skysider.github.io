---
title: 调试工具之GDB
tags: 
- gdb
category:
- debug
---

### 运行程序

**基本方式**

```shell
gdb path_to_program [corefile|pid]
```

**参数相关**

设置参数

```shell
gdb -args path_to_program <arg1> <arg2> ...
```

或者

```shell
gdb> set args <arg1> <arg2>
```

显示参数

```shell
show args
```

**启动**

```
start
starti # For programs containing an elaboration phase, the starti command will stop execution at the start of the elaboration phase.
run
```

**单步和继续**

```
step
stepi
next
nexti
continue
```

**调试多个程序**

```
info inferiors
inferior <infno>
add-inferior [-copies n] [-exec executable]
clone-inferior [-copies n] [infno]
remove-inferiors <infno>
detach inferor <infno>
```

**调试多个线程**

```
info threads
thread <thread-id>
thread apply [thread-id-list] [all] args
set print thread-event
set scheduler-locking on/off/step # on-只调试当前线程 off-不锁定任何线程，所有线程都执行 step-单步时只有当前线程执行
```



### 断点设置

**普通断点 **（breakpoint）

```
break 
break file.c:100 thread 1
```

**观察点** (watchpoint)

```shell
watch *(int *)0x600200 # 写观察点
rwatch *(int *)0x600200 # 读观察点
awatch *(int *)0x600200 # 读写观察点
```

**捕获点** (catchpoint)

```shell
catch 
```

**查看、删除、禁用断点**

````
info b
````



### 输入输出重定向

```
run > outfile
run < infile
tty /dev/ttyS0
```







