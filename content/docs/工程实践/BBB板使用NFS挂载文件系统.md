---
title: BBB板使用NFS挂载文件系统
weight: 3
---
# 版本说明

* BBB板使用NFS挂载文件系统
| 日期     | 版本 | 修改内容 |
| -------- | ---- | -------- |
| 2022/03/03 | V0.1 | 创建     |
# 将TI SDK中提供的文件系统目录NFS导出

解压文件系统

```shell
cd /usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/
sudo mkdir tisdk-default-image-am335x-evm
sudo tar -Jxf tisdk-default-image-am335x-evm.tar.xz -C tisdk-default-image-am335x-evm
```

## 修改/etc/exports增加NFS导出目录后，重新加载

```shell
$ vi /etc/exports
#增加以下内容
/usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-default-image-am335x-evm *(rw,nohide,insecure,no_subtree_check,async,no_root_squash)
```

## 使NFS目录生效

```shell
$ sudo exportfs -a
```

## 查看NFS导出列表

```shell
#查看NFS导出列表
$ showmount -e
Export list for ubuntu:
/usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-default-image-am335x-evm *
/usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-tiny-image-am335x-evm    *
/usr/local/ti-sdk-am335x-evm-07.03.00.005/targetNFS 
```

测试NFS目录是否成功

```shell
3B-pi@raspberrypi:~ $ sudo mkdir /mnt/tisdk-default-image-am335x-evm ;
3B-pi@raspberrypi:~ $ sudo mount -t nfs 192.168.31.85:/usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-default-image-am335x-evm /mnt/tisdk-default-image-am335x-evm -o nolock
```

# NFS方式启动BBB

上电后，按下SPACE键，进入uboot提示界面

## 分析uboot的nfsboot环境变量

```shell
nfs_options=,vers=3
nfsrootfstype=ext4 rootwait fixrtc
root_dir=/home/userid/targetNFS    #主机文件系统目录
server_ip=192.168.1.100				#tftp服务器地址，即主机地址
bootfile=zImage						#通过tftp从服务器下载的内核文件名
fdtfile=undefined					#通过tftp从服务器下载的设备树文件名

nfsargs=setenv bootargs console=${console} ${optargs} ${cape_disable} ${cape_enable} ${cape_uboot} root=/dev/nfs rw rootfstype=${nfsrootfstype} nfsroot=${nfsroot} ip=${ip} ${cmdline}
nfsboot=echo Booting from ${server_ip} ...; setenv nfsroot ${server_ip}:${root_dir}${nfs_options}; setenv ip ${client_ip}:${server_ip}:${gw_ip}:${netmask}:${hostname}:${device}:${autoconf}; setenv autoload no; setenv serverip ${server_ip}; setenv ipaddr ${client_ip}; tftp ${loadaddr} ${tftp_dir}${bootfile}; tftp ${fdtaddr} ${tftp_dir}dtbs/${fdtfile}; run nfsargs; bootz ${loadaddr} - ${fdtaddr}
```

## 准备内核镜像文件

通过bootfile环境变量指定了内核镜像文件名为zImage，而TI SDK中提供的内核镜像名为zImage-am335x-evm.bin，这里干脆拷贝一份

```shell
cp tftpboot/zImage-am335x-evm.bin  tftpboot/zImage
```

## 设置uboot环境变量

**特别注意：**由于默认的nfsboot环境变量，tftp下载设备树时，会从tftp服务目录之下的dtbs目录下载设备树，例如服务器的tftp目录为/home/qq/tftpboot，它会从/home/qq/tftpboot/dtbs/目录下，下载am335x-boneblack.dtb文件 

```shell
cd tftpboot
mkdir dtbs
cp am335x-boneblack.dtb  dtbs/
```



```shell
setenv client_ip 192.168.31.87
setenv ipaddr 192.168.31.87
setenv gw_ip 192.168.31.1
setenv server_ip 192.168.31.85
setenv fdtfile am335x-boneblack.dtb
setenv root_dir /usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-base-image-am335x-evm
setenv root_dir /usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-default-image-am335x-evm
setenv root_dir /usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-tiny-image-am335x-evm
setenv root_dir /usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-base-image-am335x-evm_manual_hplip
setenv root_dir /usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-base-image-am335x-evm_debug_hplip_cups
```

## 启动NFS文件系统

```shell
==> run nfsboot
```

# qtcreator配置交叉编译工具

## 安装qtcreator

```shell
sudo apt install qtcreator
```

## 设置编译工具

![qt-build_run-Kits-1](./images/qt-build_run-Kits-1.png)

![qt-build_run-Qt Versions-1](/images/BBB板使用NFS挂载文件系统/qt-build_run-Qt Versions-1.png)

![qt-build_run-Compilers-1](/images/BBB板使用NFS挂载文件系统/qt-build_run-Compilers-1.png)

![qt-build_run-Debuggers-1](/images/BBB板使用NFS挂载文件系统/qt-build_run-Debuggers-1.png)

![addDevice](/images/BBB板使用NFS挂载文件系统/addDevice.png)

# 遇到问题

## 不能解析qt工程，运行时报错如下

```shell
/usr/local/ti-sdk-am335x-evm-07.03.00.005/linux-devkit/sysroots/x86_64-arago-linux/mkspecs/features/toolchain.prf(39): system(execute) requires one or two arguments.
Project ERROR: Cannot run compiler 'arm-none-linux-gnueabihf-g++ --sysroot=/usr/local/ti-sdk-am335x-evm-07.03.00.005/linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi'. Output:
===================
===================
Maybe you forgot to setup the environment?
/usr/local/ti-sdk-am335x-evm-07.03.00.005/linux-devkit/sysroots/x86_64-arago-linux/mkspecs/features/toolchain.prf(85): Variable QMAKE_CXX.COMPILER_MACROS is not defined.
/usr/local/ti-sdk-am335x-evm-07.03.00.005/linux-devkit/sysroots/x86_64-arago-linux/mkspecs/features/toolchain.prf(210): system(execute) requires one or two arguments.
Project ERROR: Cannot run compiler 'arm-none-linux-gnueabihf-g++ --sysroot=/usr/local/ti-sdk-am335x-evm-07.03.00.005/linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi'. Output:
===================
===================
Maybe you forgot to setup the environment?
Error while parsing file /home/qq/qt_project/test_tisdk/test_tisdk.pro. Giving up.
```

解决办法

https://doc-snapshots.qt.io/qtcreator-4.0/creator-build-settings.html#batch-editing

将执行source之后的PATH变量保存到qtcreator-->Options-->build&run -->kits -->Environment中

![build&run kits Environment-1](/images/BBB板使用NFS挂载文件系统/build&run kits Environment-1.png)


## 运行时错误

```shell
Failed to create wl_display (No such file or directory)
qt.qpa.plugin: Could not load the Qt platform plugin "wayland" in "" even though it was found.
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, wayland-egl, wayland.

sh: line 1:  2516 Aborted                 (core dumped) DISPLAY=':0.0' /home/debian/test_tisdk
Application finished with exit code 134.
```

解决办法

使用vnc方式运行，添加运行时参数-platform vnc

![vnc模式运行](/images/BBB板使用NFS挂载文件系统/vnc模式运行.png)

# wayland测试

https://forum.qt.io/topic/60865/qt5-wayland-gui-application/3

```shell
mkdir -p /tmp/$USER-weston
chmod 0700 /tmp/$USER-weston
export XDG_RUNTIME_DIR=/tmp/$USER-weston
weston --tty=1 --backend=fbdev-backend.so &

Then I successfully ran my QT5 gui application with the wayland platform specifier:

./myQt5App -platform wayland
```

https://stackoverflow.com/questions/49851562/qt-wayland-failed-to-create-display-no-such-file-or-directory

I finally fixed this by deploying the libs needed by wayland-egl plugin:

the lib "libQt5WaylandClient.so.5" should be included in the deployed package. which used by the plugin.

```shell
root@am335x-evm:~# weston
Date: 2021-05-24 UTC
[08:59:53.616] weston 8.0.0
               https://wayland.freedesktop.org
               Bug reports to: https://gitlab.freedesktop.org/wayland/weston/issues/
               Build: 8.0.0
[08:59:53.619] Command line: weston
[08:59:53.619] OS: Linux, 5.4.106-g023faefa70, #1 PREEMPT Mon May 24 09:04:10 UTC 2021, armv7l
[08:59:53.620] warning: XDG_RUNTIME_DIR "/tmp/root-weston" is not configured
correctly.  Unix access mode must be 0700 (current mode is 755),
and must be owned by the user (current owner is UID 0).
Refer to your distribution on how to get it, or
http://www.freedesktop.org/wiki/Specifications/basedir-spec
on how to implement it.
[08:59:53.623] Using config file '/etc//weston.ini'
[08:59:53.628] Output repaint window is 7 ms maximum.
[08:59:53.631] Loading module '/usr/lib/libweston-8/x11-backend.so'
[08:59:53.632] Failed to load module: /usr/lib/libweston-8/x11-backend.so: cannot open shared object file: No such file or directory
[08:59:53.632] fatal: failed to create compositor backend
root@am335x-evm:~# eglinfo
-sh: eglinfo: command not found
root@am335x-evm:~# modinfo pvrsrvkm
filename:       /lib/modules/5.4.106-g023faefa70/extra/pvrsrvkm.ko
license:        Dual MIT/GPL
author:         Imagination Technologies Ltd. <gpl-support@imgtec.com>
license:        Dual MIT/GPL
author:         Imagination Technologies Ltd. <gpl-support@imgtec.com>
srcversion:     533BB7E5866E52F63B9ACCB
alias:          of:N*T*Cti,omap4-sgx540-120C*
alias:          of:N*T*Cti,omap4-sgx540-120
alias:          of:N*T*Cti,omap3-sgx530-121C*
alias:          of:N*T*Cti,omap3-sgx530-121
alias:          of:N*T*Cti,am3352-sgx530C*
alias:          of:N*T*Cti,am3352-sgx530
alias:          of:N*T*Cti,am4376-sgx530C*
alias:          of:N*T*Cti,am4376-sgx530
alias:          of:N*T*Cti,dra7-sgx544C*
alias:          of:N*T*Cti,dra7-sgx544
alias:          of:N*T*Cti,am654-sgx544C*
alias:          of:N*T*Cti,am654-sgx544
depends:
name:           pvrsrvkm
vermagic:       5.4.106-g023faefa70 preempt mod_unload modversions ARMv7 p2v8
parm:           gPVRDebugLevel:Sets the level of debug output (default 0x7) (uint)
root@am335x-evm:~# lsmod
Module                  Size  Used by
xfrm_user              32768  2
xfrm_algo              16384  1 xfrm_user
sha512_generic         20480  0
sha512_arm             24576  0
sha256_generic         16384  0
libsha256              20480  1 sha256_generic
sha1_generic           16384  0
sha1_arm_neon          20480  0
sha1_arm               16384  1 sha1_arm_neon
md5                    16384  0
ecb                    16384  0
aes_arm                16384  0
aes_generic            40960  1 aes_arm
aes_arm_bs             24576  0
crypto_simd            16384  1 aes_arm_bs
cryptd                 24576  1 crypto_simd
des_generic            16384  0
libdes                 28672  1 des_generic
cbc                    16384  0
pru_rproc              24576  0
icss_iep               20480  0
irq_pruss_intc         16384  1 pru_rproc
prueth_ecap            16384  0
musb_dsps              20480  0
musb_hdrc             106496  1 musb_dsps
udc_core               28672  1 musb_hdrc
phy_am335x             16384  2
usbcore               225280  1 musb_hdrc
phy_generic            16384  1 phy_am335x
usb_common             16384  5 phy_am335x,udc_core,musb_hdrc,musb_dsps,usbcore
phy_am335x_control     16384  1 phy_am335x
pruss                  16384  1 pru_rproc
pvrsrvkm              405504  0
pm33xx                 16384  0
omap_aes_driver        24576  0
crypto_engine          16384  1 omap_aes_driver
omap_crypto            16384  1 omap_aes_driver
libaes                 16384  4 omap_aes_driver,aes_arm_bs,aes_arm,aes_generic
ti_emif_sram           16384  1 pm33xx
omap_sham              32768  0
wkup_m3_ipc            16384  1 pm33xx
at24                   20480  0
rtc_omap               20480  4 pm33xx
omap_wdt               16384  0
wkup_m3_rproc          16384  1
musb_am335x            16384  0
sch_fq_codel           20480  1
uio_module_drv         16384  0
uio                    20480  1 uio_module_drv
cryptodev              53248  1

```

```
libicu-dev is already the newest version (57.1-4).
libicu-dev set to manually installed.
```

运行程序 

```shell
./application -platform wayland
或者
export QT_QPA_PLATFORM=wayland ./application
```

https://web.stanford.edu/class/archive/cs/cs106b/cs106b.1164/handouts/qt-creator-troubleshooting.html

测试各种模式，除了vnc模式和offscreen，其它都运行失败

```shell
root@am335x-evm:/home/debian# ./test_tisdk -platform vnc
QVncServer created on port 5900
^C
root@am335x-evm:/home/debian# ^C
root@am335x-evm:/home/debian# ./test_tisdk -platform xcb
qt.qpa.plugin: Could not find the Qt platform plugin "xcb" in ""
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, wayland-egl, wayland.

Aborted (core dumped)
root@am335x-evm:/home/debian# ./test_tisdk -platform linuxfb
Unable to figure out framebuffer device. Specify it manually.
linuxfb: Failed to initialize screen
qt.qpa.input: xkbcommon not available, not performing key mapping
no screens available, assuming 24-bit color
Cannot create window: no screens available
Aborted (core dumped)
root@am335x-evm:/home/debian# ./test_tisdk -platform eglfs
MESA-LOADER: failed to open kms_swrast (search paths /usr/lib/dri)
failed to load driver: kms_swrast
MESA-LOADER: failed to open swrast (search paths /usr/lib/dri)
failed to load swrast driver
Could not create GBM device (No such device)
Could not open DRM device
Aborted (core dumped)
root@am335x-evm:/home/debian# ./test_tisdk -platform minimalegl
Opened display 0x311a0

Could not initialize egl display

EGL error
Aborted (core dumped)
root@am335x-evm:/home/debian# ./test_tisdk -platform waylang-egl
qt.qpa.plugin: Could not find the Qt platform plugin "waylang-egl" in ""
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, wayland-egl, wayland.

Aborted (core dumped)
root@am335x-evm:/home/debian# ./test_tisdk -platform waylang
qt.qpa.plugin: Could not find the Qt platform plugin "waylang" in ""
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

Available platform plugins are: eglfs, linuxfb, minimal, minimalegl, offscreen, vnc, wayland-egl, wayland.

Aborted (core dumped)
root@am335x-evm:/home/debian# ./test_tisdk -platform offscreen
This plugin does not support propagateSizeHints()

```

# uboot启动分析

## 命令说明

```shell
=>    gpio -h
gpio - query and control gpio pins

Usage:
gpio <input|set|clear|toggle> <pin>
    - input/set/clear/toggle the specified pin
gpio status [-a] [<bank> | <pin>]  - show [all/claimed] GPIOs
```

```shell
test -n 字符串	字符串的长度不为零则为真
test -e 文件名	如果文件存在则为真
```

## 以下为uboot环境变量中boot的值

从打印日志可以看出，boot值为启动时执行的第一个命令

```shell
boot=${devtype} dev ${mmcdev}
if ${devtype} rescan; then
    gpio set 54
    setenv bootpart ${mmcdev}:1
    if test -e ${devtype} ${bootpart} /etc/fstab; then setenv mmcpart 1; fi
    echo Checking for: /uEnv.txt ...
    if test -e ${devtype} ${bootpart} /uEnv.txt; then
        if run loadbootenv; then
            gpio set 55
            echo Loaded environment from /uEnv.txt
            run importbootenv
        fi
        echo Checking if uenvcmd is set ...
        if test -n ${uenvcmd}; then
            gpio set 56
            echo Running uenvcmd ...
            run uenvcmd
        fi
        echo Checking if client_ip is set ...
        if test -n ${client_ip}; then
            if test -n ${dtb}; then
                setenv fdtfile ${dtb}
                echo using ${fdtfile} ...
            fi
            gpio set 56
            if test -n ${uname_r}; then
                echo Running nfsboot_uname_r ...
                run nfsboot_uname_r
            fi
            echo Running nfsboot ...
            run nfsboot
        fi
    fi
    echo Checking for: /${script} ...
    if test -e ${devtype} ${bootpart} /${script}; then
        gpio set 55
        setenv scriptfile ${script}
        run loadbootscript
        echo Loaded script from ${scriptfile}
        gpio set 56
        run bootscript
    fi
    echo Checking for: /boot/${script} ...
    if test -e ${devtype} ${bootpart} /boot/${script}; then
        gpio set 55
        setenv scriptfile /boot/${script}
        run loadbootscript
        echo Loaded script from ${scriptfile}
        gpio set 56
        run bootscript
    fi
    echo Checking for: /boot/uEnv.txt ...
    for i in 1 2 3 4 5 6 7; do
        setenv mmcpart ${i}
        setenv bootpart ${mmcdev}:${mmcpart}
        if test -e ${devtype} ${bootpart} /boot/uEnv.txt; then
            gpio set 55
            load ${devtype} ${bootpart} ${loadaddr} /boot/uEnv.txt
            env import -t ${loadaddr} ${filesize}
            echo Loaded environment from /boot/uEnv.txt
            if test -n ${dtb}; then
                echo debug: [dtb=${dtb}] ...
                setenv fdtfile ${dtb}
                echo Using: dtb=${fdtfile} ...
            fi
            echo Checking if uname_r is set in /boot/uEnv.txt...
            if test -n ${uname_r}; then
                gpio set 56
                setenv oldroot /dev/mmcblk${mmcdev}p${mmcpart}
                echo Running uname_boot ...
                run uname_boot
            fi
        fi
    done
fi
```

## 设置默认NFS启动

从上面脚本可以得知，想要每次直接进行NFS挂载，需要在根目录下，存在uEnv.tx文件，且uEnv.txt中包含client_ip变量，如下即可，之后每次都会先检查eMMC中的根目录的uEnv.txt，判断是否需要NFS启动了

```shell
#uname_r=4.19.94-ti-r42
client_ip=192.168.31.87
gw_ip=192.168.31.1
server_ip=192.168.31.85
dtb=am335x-boneblack.dtb
root_dir=/usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-tiny-image-am335x-evm
```

下面为uboot日志

```shell
gpio: pin 56 (gpio 56) value is 0
gpio: pin 55 (gpio 55) value is 0
gpio: pin 54 (gpio 54) value is 0
gpio: pin 53 (gpio 53) value is 1
Card did not respond to voltage select!
Card did not respond to voltage select!
switch to partitions #0, OK
mmc1(part 0) is current device
Scanning mmc 1:1...
gpio: pin 56 (gpio 56) value is 0
gpio: pin 55 (gpio 55) value is 0
gpio: pin 54 (gpio 54) value is 0
gpio: pin 53 (gpio 53) value is 1
switch to partitions #0, OK
mmc1(part 0) is current device
gpio: pin 54 (gpio 54) value is 1
Checking for: /uEnv.txt ...
Checking for: /boot.scr ...
Checking for: /boot/boot.scr ...
Checking for: /boot/uEnv.txt ...
gpio: pin 55 (gpio 55) value is 1
```

