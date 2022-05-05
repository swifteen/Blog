---
title: Core文件调试
weight: 6
---

# Core files使用

## 配置core文件生成

去掉限制，生成core文件，在shell中执行，或者添加到/etc/profile文件中

```shell
ulimit -c unlimited
```

## 配置core文件名规则

设置core文件名，附带进程号

Writing a 1 to it causes the PID number of the dying process to be appended to the filename, which is somewhat useful as long as you can associate the PID number with a program name from log files  

```shell
echo 1 > /proc/sys/kernel/core_uses_pid
```

设置core文件名称，更高级的用法是使用管道符，进行重定向到自定义的程序中，验证是否能将core文件输出到U盘目录下，参考man core

```shell
#创建core文件保存目录
mkdir /corefiles/
echo /corefiles/core.%e.%s.%p.%t > /proc/sys/kernel/core_pattern
```

```shell
           %e  The process or thread's comm value, which typically is the same as the executable filename (without path prefix,
               and truncated to a maximum of 15 characters), but may have been modified to be something different; see the
               discussion of /proc/[pid]/comm and /proc/[pid]/task/[tid]/comm in proc(5).
           %p  PID of dumped process, as seen in the PID namespace in which the process resides.
           %s  Number of signal causing dump.
           %t  Time of dump, expressed as seconds since the Epoch,1970-01-01 00:00:00 +0000 (UTC).
```

要使 core_pattern 保持不变，重启之后仍然有效，你可以通过设置 /etc/sysctl.conf 里的 “kernel.core_pattern” 实现。

```shell
# Change name of core file to start with the command name
# so you get things like: emacs.core mozilla-bin.core X.core
kernel.core_pattern = /corefiles/core.%e.%s.%p.%t
```

## 调试core文件

```shell
#在主机 上运行
arm-poky-linux-gnueabi-gdb app_name -c /corefiles/core.sort-debug.1431425613
```

```shell
gdb /opt/usr/bin/zq_gui_main -c /corefiles/core.zq_gui_main.6.4200.1631870548
```

# 地址空间布局随机化

Linux refers to this mitigation strategy as “address space layout randomization” or **ASLR** for short.

To make writing these examples easier and consistent when I come back to them, I will temporary disable this on the test system.

```shell
echo 0 > /proc/sys/kernel/randomize_va_space
```

