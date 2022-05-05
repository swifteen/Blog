---
title: Native GDB
weight: 4
---

# GBD调试方案选择

1、如果使用纯命令行方式，学习难度大，但占用资源低，操作起来更流畅

2、选择IDE集成的GDB调试，需要个人电脑上安装虚拟机，搭建linux交叉编译环境，在虚拟机上编译，远程cross GDB，缺点是个人的电脑性能不够

3、在性能强大的docker主机上，通过X11服务连接远程桌面，启动IDE进行GDB调试，当3个人同时操作时，docker服务器也会存在性能不够

因此最终选择方案1，尽管难度更大

# Native GDB 

使用yocto编译文件系统时，修改配置文件

add gdb to the target image by adding this to conf/local.conf  

```shell
EXTRA_IMAGE_FEATURES ?= "tools-debug dbg-pkgs"
```

## 搭建opkg管理服务端安装gdb

https://blog.csdn.net/lqjjdx/article/details/112170199

https://e2e.ti.com/support/processors-group/processors/f/processors-forum/266848/opkg-updation

执行bitbake命令生成包索引文件

```shell
MACHINE=am335x-evm bitbake package-index
```

在deploy目录下，生成Packages.gz文件，如下

```shell
ubuntu:~/tool/tisdk/build/arago-tmp-external-arm-glibc/deploy$find . -name Packages.gz
./ipk/x86_64-nativesdk/Packages.gz
./ipk/armv7at2hf-neon/Packages.gz
./ipk/am335x_evm/Packages.gz
./ipk/armv7ahf-neon/Packages.gz
./ipk/all/Packages.gz
```

搭建opkg管理服务端

```shell
cd ~/tool/tisdk/build/arago-tmp-external-arm-glibc/deploy/ipk
#启动服务
python3 -m http.server --bind 0.0.0.0 5499 
```

访问http://192.168.7.170:5499/，可以看到如下目录

```shell
Directory listing for /
all/
am335x_evm/
arago/
armv5e/
armv7ahf-neon/
armv7at2hf-neon/
buildtools-dummy-nativesdk/
Packages
sdk-provides-dummy-nativesdk/
sdk-provides-dummy-target/
x86_64-nativesdk/
```
## 设置arm机器上的opkg配置

根据上面存在的目录，修改两个配置文件/etc/opkg/arch.conf 和/etc/opkg/base-feeds.conf ，如下

```shell
root@am335x-evm:~# cat /etc/opkg/arch.conf       
arch all 1
#arch any 6
#arch noarch 11
#arch armv5hf-vfp 16
#arch armv5thf-vfp 21
#arch armv5ehf-vfp 26
#arch armv5tehf-vfp 31
#arch armv6hf-vfp 36
#arch armv6thf-vfp 41
#arch armv7ahf-vfp 46
#arch armv7at2hf-vfp 51
arch armv7ahf-neon 56
arch armv7at2hf-neon 61
arch am335x_evm 66
```

```shell
root@am335x-evm:~# cat /etc/opkg/base-feeds.conf 
src/gz all http://192.168.7.170:5499/all
#src/gz any http://192.168.7.170:5499/any
#src/gz noarch http://192.168.7.170:5499/noarch
#src/gz armv5hf-vfp http://192.168.7.170:5499/armv5hf-vfp
#src/gz armv5thf-vfp http://192.168.7.170:5499/armv5thf-vfp
#src/gz armv5ehf-vfp http://192.168.7.170:5499/armv5ehf-vfp
#src/gz armv5tehf-vfp http://192.168.7.170:5499/armv5tehf-vfp
#src/gz armv6hf-vfp http://192.168.7.170:5499/armv6hf-vfp
#src/gz armv6thf-vfp http://192.168.7.170:5499/armv6thf-vfp
#src/gz armv7ahf-vfp http://192.168.7.170:5499/armv7ahf-vfp
#src/gz armv7at2hf-vfp http://192.168.7.170:5499/armv7at2hf-vfp
src/gz armv7ahf-neon http://192.168.7.170:5499/armv7ahf-neon
src/gz armv7at2hf-neon http://192.168.7.170:5499/armv7at2hf-neon
src/gz am335x_evm http://192.168.7.170:5499/am335x_evm
```

执行opkg update ,更新包索引

```shell
root@am335x-evm:~# opkg update                   
Downloading http://192.168.7.170:5499/all/Packages.gz.
Updated source 'all'.
Downloading http://192.168.7.170:5499/armv7ahf-neon/Packages.gz.
Updated source 'armv7ahf-neon'.
Downloading http://192.168.7.170:5499/armv7at2hf-neon/Packages.gz.
Updated source 'armv7at2hf-neon'.
Downloading http://192.168.7.170:5499/am335x_evm/Packages.gz.
Updated source 'am335x_evm'.
```

## 安装native gdb

```shell
root@am335x-evm:~# opkg install gdb
Installing busybox-syslog (1.31.1) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/busybox-syslog_1.31.1-r0.arago21.tisdk0_armv7at2hf-neon.ipk.
Installing python3-codecs (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-codecs_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-tkinter (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-tkinter_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-2to3 (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-2to3_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-pydoc (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-pydoc_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-html (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-html_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-typing (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-typing_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-difflib (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-difflib_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-idle (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-idle_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-db (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-db_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-numbers (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-numbers_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-json (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-json_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-smtpd (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-smtpd_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-plistlib (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-plistlib_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-syslog (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-syslog_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-multiprocessing (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-multiprocessing_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-curses (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-curses_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-xmlrpc (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-xmlrpc_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-terminal (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-terminal_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-venv (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-venv_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-fcntl (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-fcntl_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-pprint (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-pprint_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-sqlite3 (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-sqlite3_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-image (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-image_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-compile (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-compile_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-audio (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-audio_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-unixadmin (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-unixadmin_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-mmap (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-mmap_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-ctypes (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-ctypes_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-profile (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-profile_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-resource (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-resource_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-netserver (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-netserver_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-asyncio (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-asyncio_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-mailbox (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-mailbox_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-debugger (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-debugger_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-misc (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-misc_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-unittest (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-unittest_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-doctest (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-doctest_3.8.2-r0_armv7at2hf-neon.ipk.
Installing python3-modules (3.8.2) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/python3-modules_3.8.2-r0_armv7at2hf-neon.ipk.
Installing gdb (9.1) on root
Downloading http://192.168.7.170:5499/armv7at2hf-neon/gdb_9.1-r0_armv7at2hf-neon.ipk.
Configuring python3-pprint.
Configuring python3-debugger.
Configuring busybox-syslog.
update-alternatives: Linking /sbin/klogd to /bin/busybox.nosuid
update-alternatives: Linking /sbin/syslogd to /bin/busybox.nosuid
Created symlink /etc/systemd/system/syslog.service -> /lib/systemd/system/busybox-syslog.service.
Created symlink /etc/systemd/system/multi-user.target.wants/busybox-syslog.service -> /lib/systemd/system/busybox-syslog.service.
Created symlink /etc/systemd/system/multi-user.target.wants/busybox-klogd.service -> /lib/systemd/system/busybox-klogd.service.
Configuring python3-tkinter.
Configuring python3-2to3.
Configuring python3-audio.
Configuring python3-codecs.
Configuring python3-pydoc.
Configuring python3-misc.
Configuring python3-fcntl.
Configuring python3-mailbox.
Configuring python3-html.
Configuring python3-typing.
Configuring python3-difflib.
Configuring python3-idle.
Configuring python3-db.
Configuring python3-numbers.
Configuring python3-json.
Configuring python3-smtpd.
Configuring python3-plistlib.
Configuring python3-syslog.
Configuring python3-multiprocessing.
Configuring python3-curses.
Configuring python3-xmlrpc.
Configuring python3-terminal.
Configuring python3-netserver.
Configuring python3-venv.
Configuring python3-asyncio.
Configuring python3-compile.
Configuring python3-ctypes.
Configuring python3-unittest.
Configuring python3-doctest.
Configuring python3-image.
Configuring python3-mmap.
Configuring python3-profile.
Configuring python3-resource.
Configuring python3-sqlite3.
Configuring python3-unixadmin.
Configuring python3-modules.
Configuring gdb.
```

## Native GDB调试过程

```shell
root@am335x-evm:~# gdb /opt/usr/bin/gui_main 
GNU gdb (GDB) 9.1
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "arm-linux-gnueabi".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /opt/usr/bin/gui_main...
(gdb) r
Starting program: /opt/usr/bin/gui_main 
warning: File "/lib/libthread_db-1.0.so" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
To enable execution of this file add
        add-auto-load-safe-path /lib/libthread_db-1.0.so
line to your configuration file "/home/root/.gdbinit".
To completely disable this security protection add
        set auto-load safe-path /
line to your configuration file "/home/root/.gdbinit".
For more information about this security protection see the
"Auto-loading safe path" section in the GDB manual.  E.g., run from the shell:
        info "(gdb)Auto-loading safe path"
warning: Unable to find libthread_db matching inferior's thread library, thread debugging will not be available.
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
qt.qpa.input: xkbcommon not available, not performing key mapping
Cannot open keyboard input device '/dev/input/event0' (No such file or directory)
Failed to open keyboard device /dev/input/event0

Program received signal SIGABRT, Aborted.
0x4e1e8d16 in ?? () from /lib/libc.so.6
(gdb) bt
#0  0x4e1e8d16 in ?? () from /lib/libc.so.6
#1  0x4e1f74a4 in raise () from /lib/libc.so.6
#2  0x4e1e87a2 in abort () from /lib/libc.so.6
#3  0xb6c05246 in QMessageLogger::fatal(char const*, ...) const () from /usr/lib/libQt5Core.so.5
#4  0xb6c04fc2 in qt_assert(char const*, char const*, int) () from /usr/lib/libQt5Core.so.5
#5  0xb6fc7538 in ZqGuiSettingMachine::timerEvent (this=0x7b160, event=0xbefff8d0)
    at /home/ubuntu/targetNFS/test/src/test_main.cpp:156
#6  0xb6d7e032 in QObject::event(QEvent*) () from /usr/lib/libQt5Core.so.5
#7  0xb68706e8 in QWidget::event(QEvent*) () from /usr/lib/libQt5Widgets.so.5
#8  0xb6844b42 in QApplicationPrivate::notify_helper(QObject*, QEvent*) () from /usr/lib/libQt5Widgets.so.5
#9  0xb684b3ac in QApplication::notify(QObject*, QEvent*) () from /usr/lib/libQt5Widgets.so.5
#10 0xb6f7d000 in ?? () from /usr/lib/libQt5Core.so.5
Backtrace stopped: previous frame identical to this frame (corrupt stack?)
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

