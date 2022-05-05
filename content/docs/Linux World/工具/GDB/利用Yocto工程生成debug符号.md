---
title: 利用Yocto工程生成debug符号
weight: 1
---

# 实现目的

嵌入式设备在进行gdb调试时，由于存储容量限制，文件系统中一般不会包含debug版本的libc库，导致总是会提示找不到符号表，如下

```shell
(gdb) n

Program received signal SIGABRT, Aborted.
0xb6c0fbe6 in ?? ()
(gdb) bt
#0  0xb6c0fbe6 in ?? ()
#1  0xb6c1ebce in ?? ()
```

如果程序需要依赖多个库，就需要单独去编译每一个依赖的库为debug版本，不太现实，可以利用yocto工程一次性解决，轻松实现目标，生成debug版本的安装包或者debug版本的文件系统。

# 利用Yocto工程生成debug符号

默认SDK包中并不包含debug版本的库和源码文件，为了让gdb找到符号表，需要重新编译SDK，开启debug相关选项，通过如下两步，可以获得一个debug安装包，安装之后，会在原有SDK安装目录下添加debug版本的库和源码文件，这样在执行GDB调试时，指定好相应的目录就能找到符号表了

## 1、修改conf/local.conf文件

新增如下：

```shell
#debug时添加以下选项
EXTRA_IMAGE_FEATURES ?= "tools-debug ssh-server-openssh"

#解决bitbake执行到打包时失败
RDEPENDS_packagegroup-core-standalone-sdk-target_remove = "libatomic-dev"
```

## 2、执行如下bitbake命令，生成带debug符号的SDK安装包

```shell
MACHINE=am335x-evm bitbake -c populate_sdk tisdk-base-image
```

在~/tool/tisdk/build/arago-tmp-external-arm-glibc/deploy/sdk/目录下，会生成arago-2020.09-toolchain-2020.09.sh和arago-2020.09-toolchain-2020.09.sharago-2020.09-armv7a-linux-gnueabi-tisdk.sh两个安装包，如下

```shell
qq@ubuntu:~/tool/tisdk/build/arago-tmp-external-arm-glibc/deploy/sdk$ ls -lth
total 1.2G
-rwxr-xr-x 2 qq qq 646M Apr  2 02:36 arago-2020.09-armv7a-linux-gnueabi-tisdk.sh
-rw-r--r-- 2 qq qq 7.5K Apr  2 02:32 arago-2020.09-armv7a-linux-gnueabi-tisdk.host.manifest
-rw-r--r-- 2 qq qq  49K Apr  2 02:30 arago-2020.09-armv7a-linux-gnueabi-tisdk.target.manifest
-rw-r--r-- 2 qq qq 332K Apr  2 02:30 arago-2020.09-armv7a-linux-gnueabi-tisdk.testdata.json
-rwxr-xr-x 2 qq qq 499M Feb 25 17:29 arago-2020.09-toolchain-2020.09.sh
-rw-r--r-- 2 qq qq   66 Feb 25 16:54 arago-2020.09-toolchain-2020.09.host.manifest
-rw-r--r-- 2 qq qq  81K Feb 25 16:53 arago-2020.09-toolchain-2020.09.target.manifest
-rw-r--r-- 2 qq qq 361K Feb 25 16:53 arago-2020.09-toolchain-2020.09.testdata.json
```

## 3、安装生成的两个debug包

3.1、安装后，安装目录中会包含sysroot目录，并且sysroot目录中包含所有程序和库对应的debug symbols

3.2、所有可执行文件的源码保存到/usr/src/debug/，相对于sysroot目录

The SDK contains a copy of GDB.

It also contains a sysroot for the target with debug symbols for all the programs and libraries that are part of the target image.

Finally, the SDK contains the source code for the executables.

In each of these directories, you will find a subdirectory named .debug/ that contains the symbols for each program and library. 

GDB knows to look in .debug/ when searching for symbol information. 

需要将路径指向上一步中生成的linux-devkit目录， /home/ubuntu/test_ti_sdk/linux-devkit

```shell
ubuntu:~/tool/tisdk/build/arago-tmp-external-arm-glibc/deploy/sdk$./arago-2020.09-toolchain-2020.09.sh 
Arago SDK installer version 2020.09
===================================
Enter target directory for SDK (default: /tmp/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy): /home/ubuntu/test_ti_sdk/linux-devkit                
The directory "/home/ubuntu/test_ti_sdk/linux-devkit" already contains a SDK for this architecture.
If you continue, existing files will be overwritten! Proceed [y/N]? y
Extracting SDK....................................................................................done
Setting it up...done
SDK has been successfully set up and is ready to be used.
Each time you wish to use the SDK in a new shell session, you need to source the environment setup script e.g.
 $ . /home/ubuntu/test_ti_sdk/linux-devkit/environment-setup-armv7at2hf-neon-linux-gnueabi
```

安装完成后，可以找到.debug相关目录，如下

```shell
ubuntu:~/test_ti_sdk$find ./linux-devkit -name  ".debug" -type d
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/bin/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/share/Linux-PAM/xtests/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/bin/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/libexec/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/libexec/mtd-utils/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/lib/python3.8/site-packages/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/lib/bash/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/lib/systemd/user-environment-generators/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/lib/engines-1.1/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/lib/audit/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/lib/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/lib/xtables/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/sbin/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/lib/systemd/system-generators/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/lib/systemd/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/lib/udev/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/lib/security/pam_filter/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/lib/security/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/lib/.debug
./linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/sbin/.debug
```

同时会在usr/src/debug/目录下包含源码，The source code for the executables is stored in /usr/src/debug/, relative to the sysroot.

```shell
ubuntu:~/test_ti_sdk$ls linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/src/debug/
acl         base-passwd  dosfstools  flac       iptables    libcap         libnss-mdns  libpng          libusb1    mtd-utils  opkg     shared-mime-info  usbutils
alsa-lib    bzip2        e2fsprogs   freetype   kbd         libcap-ng      libogg       libsamplerate0  libvorbis  ncurses    phytool  strace            util-linux
alsa-utils  cifs-utils   elfutils    glib-2.0   kmod        libdaemon      libpam       libsndfile1     libxml2    netperf    psplash  systemd           xz
attr        dbus         ethtool     i2c-tools  less        libffi         libpcap      libsolv         ltrace     nfs-utils  rpcbind  tcpdump           zlib
avahi       devmem2      expat       iperf3     libarchive  libjpeg-turbo  libpcre      libtirpc        lzo        openssl    shadow   tcp-wrappers
```

# 开启debug选项编译程序

## makfile编译开启debug选项

```shell
#-g生成调试信息，-Og开启优化但是不影响GDB
LOCAL_CFLAGS += -g -Og
```

On some architectures, GCC will not generate stack-frame pointers with the higher levels of optimization (-O2 and above). If you find yourself in a situation where you really have to compile with -O2, but still want backtraces, you can override the default behavior with -fno-omit-frame-pointer  

```shell
 -fno-omit-frame-pointer  
```

## cmake编译开启debug选项

```cmake
SET(CMAKE_BUILD_TYPE "Debug")
SET(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0  -g -ggdb")
SET(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

```cmake
add_compile_options(-O0  -g -ggdb)
```

## Qt工程设置debug选项

```shell
QMAKE_CXXFLAGS += -g -Og
QMAKE_CXXFLAGS_RELEASE += -O1
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

