---
title: GDB高级用法
weight: 7
---
# Just-in-time debugging  

在目标板子上运行gdbserver，其中109为进程号

```shell
# gdbserver --attach :10000 109
Attached; pid = 109
Listening on port 10000
```

在目标板子上执行完成后，执行detach，使程序继续运行

```shell
(gdb) detach
Detaching from program: /home/chris/MELP/helloworld/helloworld,
process 109
Ending remote debugging.
```

# Debugging forks and threads  

## 多进程调试

多进程程序在执行debug时，debug session默认是进入父进程，可以通过follow-fork-mode选项修改为进入子进程.Unfortunately,current versions (10.1) of gdbserver do not support this option, so it only works for native debugging. If you really need to debug the child process while using gdbserver, a workaround is to modify the code so that the child loops on a variable immediately after the fork, giving you the opportunity to attach a new gdbserver session to it and then to set the variable so that it drops out of the loop  

```shell
follow-fork-mode
```

## 多线程调试

默认情况下，到达断点时，所有线程停止

当从断点处恢复继续运行时，所有线程都开始运行，如果此时执行单步调试，那可能会引起问题

```shell
#为on时，只有被断点打断的线程才会恢复，其它线程仍然停止，默认为off，代表从断点处恢复继续运行时，所有线程都开始运行
#可以在调试中切换此选项配置
scheduler-locking
​```                        |
```

## GDB-youtube

set can-use-hw-watchpoints 0

```shell
(gdb)command 2

(gdb)p $pc
(gdb)reverse-stepi
#stack point
(gdb)print $sp
2 = (void *) 0xffffdc98
(gdb)print *(int**) 0xffffdc98
3 = (int *) 0x5e4c5d00
(gdb)watch *(int**) 0xffffdc98
(gdb)reverse-continue
```

