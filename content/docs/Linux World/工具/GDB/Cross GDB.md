---
title: Cross GDB
weight: 3
---

# GBD调试方案选择

1、如果使用纯命令行方式，学习难度大，但占用资源低，操作起来更流畅

2、选择IDE集成的GDB调试，需要个人电脑上安装虚拟机，搭建linux交叉编译环境，在虚拟机上编译，远程cross GDB，缺点是个人的电脑性能不够

3、在性能强大的docker主机上，通过X11服务连接远程桌面，启动IDE进行GDB调试，当3个人同时操作时，docker服务器也会存在性能不够

因此最终选择方案1，尽管难度更大

# Cross GDB

需要告诉主机上的GDB在哪里去找debug符号和源码

GDB, running on the host, needs to be told where to look for debug symbols and source code, especially for shared libraries

You need debug symbols and source code for the binaries you want to debug on the host, but not on the target. Often, there is not enough storage space for them on the target, and they will need to be stripped before deploying to the target.

## 目标板上使用gdbserver运行程序

在目标板子上运行gdbserver，验证一下这里的hello-world程序是否可以是stripped的

```shell
# gdbserver :10000 ./hello-world
Process hello-world created; pid = 103
Listening on port 10000
```

## 在主机上运行gdb

注意这里的hello-world程序必须是unstripped的

```shell
aarch64-poky-linux-gdb hello-world
#连接远程gdbserver
(gdb) target remote 192.168.7.249:9999
#设置sysroot目录
(gdb) set sysroot /opt/poky/3.1.5/sysroots/aarch64-poky-linux
#显示源码查找目录，$cdir为编译目录，$cwd为当前目录
(gdb) show directories
Source directories searched: $cdir:$cwd
#指定源码查找目录
#Paths added in this way take precedence because they are searched before those from sysroot or solib-search-path
(gdb) directory /home/chris/MELP/src/lib_mylib
```

## 设置代码文件路径替换

GDB has a simple way to cope with an entire directory tree being moved like this: substitute-path. So, when debugging with the Yocto Project SDK, you need to use these commands

```shell
(gdb) set substitute-path from to

(gdb) set sysroot [Directory]
(gdb) set sysroot remote:/
(gdb) set sysroot remote:[Remote directory]
(gdb) set solib-absolute-prefix [Directory]
(gdb) show sysroot
```

## 导入符号表

gdb多线程及动态库调试，需要手动导入带符号表的库路径，可以查看当前已导入的库：

You may have additional shared libraries that are stored outside the sysroot. In that case, you can use set solib-search-path, which can contain a colon-separated list of directories to search for shared libraries. GDB searches solib-search-path only if
it cannot find the binary in the sysroot 

```shell
(gdb) set solib-absolute-prefix ../out/target/product/tiny4412/symbols
(gdb) set solib-search-path ../out/target/product/tiny4412/symbols/system/lib
(gdb) info sharedlibrary
```

The latter is encoded into the object files with the tag DW_AT_comp_dir. You can see these tags using objdump --dwarf

```shell
aarch64-poky-linux-objdump --dwarf ./helloworld | grep DW_AT_comp_dir
```

# 遇到错误

## 1、C库函数符号找不到

https://stackoverflow.com/questions/48278881/gdb-complaining-about-missing-raise-c/48287761#48287761

```cpp
Program received signal SIGABRT, Aborted.
0x76cd0f70 in __GI_raise (sig=sig@entry=6) at ../nptl/sysdeps/unix/sysv/linux/raise.c:56
56  ../nptl/sysdeps/unix/sysv/linux/raise.c: No such file or directory.
```

```shell
#查看当前运行的源文件
(gdb) info source
Current source file is ../sysdeps/unix/sysv/linux/raise.c
Compilation directory is /build/glibc-S7Ft5T/glibc-2.23/signal
Source language is c.
Producer is GNU C11 5.4.0 20160609 -mtune=generic -march=x86-64 -g -O2 -O3 -std=gnu11 -fgnu89-inline -fno-stack-protector -fmerge-all-constants -frounding-math -fPIC -ftls-model=initial-exec.
Compiled with DWARF 2 debugging format.
Does not include preprocessor macro info.
```

解决办法

```shell
sudo mkdir /opt/src
cd /opt/src/
sudo apt source libc6
find $PWD -maxdepth 1 -type d -name "glibc*"
ls /opt/src/glibc-2.23/sysdeps/unix/sysv/linux/raise.c 
```

设置库路径

```cpp
(gdb) set substitute-path /build/glibc-S7Ft5T/glibc-2.23 /opt/src/glibc-2.23
```

## 2、查找不到库文件

https://stackoverflow.com/questions/16254546/gdb-can-not-open-shared-object-file

```shell
(gdb) r
Starting program: /home/ubuntu/targetNFS/test/build/test_main 
/home/ubuntu/targetNFS/test/build/test_main: error while loading shared libraries: libiconv.so.2: cannot open shared object file: No such file or directory
[Inferior 1 (process 39876) exited with code 0177]
```

```shell
(gdb) set env LD_LIBRARY_PATH /usr/local/lib/
(gdb) set environment LD_LIBRARY_PATH /usr/local/lib/
```

## 3、找不到符号

\1. 确保在编译proc的时候，加上了-g参数，用于调试。

\2. 用如下命令列出proc所具有的全部的符号

> nm proc

\3. 在proc所具有的全部的符号中，搜索foo，得到

> _ZN3NamespaceA6ClassA16fooEP6ClassB
> _ZN3NamespaceA6ClassA16fooEb

可以确定完整的foo函数签名应该是

> NamespaceA::ClassA::foo

执行

> b NamespaceA::ClassA::foo

gdb多线程及动态库调试，需要手动导入带符号表的库路径，可以查看当前已导入的库：

```shell
set solib-absolute-prefix ../out/target/product/tiny4412/symbols
set solib-search-path ../out/target/product/tiny4412/symbols/system/lib
info sharedlibrary
```

中间有链接静态库需要调试时，在编译静态库时需要去掉编译优化，否则单步调试会乱跳；

```shell
LOCAL_CFLAGS += -g -O0
```

## 3、找不到自己的代码文件

解决方法1：修改gcc 选项

```shell
-fdebug-prefix-map=/full/build/path=.
-fdebug-prefix-map=old_path=new_path
```

解决方法2：修改gdb配置

```shell
#利用如下命令设置替换路径规则
set subsitute-path from  to

#举个例子如下
/home/ubuntu/test/src/xxx.cpp   #编译到程序中的代码路径
/mnt/nfs/test/src/xxx.cpp		#NFS目录下实际真正的代码文件所在路径
#那么应该如下设置
(gdb) set subsitute-path /home/ubuntu  /mnt/nfs
```

推荐使用方法2

https://stackoverflow.com/questions/9607155/make-gcc-put-relative-filenames-in-debug-information

# 参考链接

[Using Qt Creator to cross-compile and debug Raspberry Pi Qt5 apps](https://jumpnowtek.com/rpi/Qt-Creator-Setup-for-RPi-cross-development.html)

[Qt creator gdb remote debugging with debug helpers](https://stackoverflow.com/questions/25011953/qt-creator-gdb-remote-debugging-with-debug-helpers)

