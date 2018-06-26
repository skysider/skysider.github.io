# GDB

设置程序参数

```
gdb --args /bin/ls -l
```

断点自动执行指令

```
commands [list ...]
... command-list ...
end
```

For example:

```
commands 1
silent
printf "x is %d\n",x
c
end
```

