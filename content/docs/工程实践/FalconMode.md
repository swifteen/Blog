---
title: FalconMode模式
weight: 2
---
# 版本说明

* FalconMode模式
| 日期     | 版本 | 修改内容 |
| -------- | ---- | -------- |
| 2021/11/21 | V0.1 | 创建     |
# FalconMode模式

Normal mode:
U-Boot SPL -> U-Boot -> Kernel

Falcon mode:
U-Boot SPL -> Kernel

https://forum.digikey.com/t/beaglebone-black-u-boot-overlays-and-falcon-mode/3008/7

Once you are booted up into the Linux Kernel there should be NO difference.

![image-20210930005718845](/images/FalconMode模式/image-20210930005718845.png)

# 测试启动时间

| 模式               | 文件系统                    | tftp下载时间 | 内核到telnet时间 |
| ------------------ | --------------------------- | ------------ | ---------------- |
| 正常uBooty启动模式 | tisdk-tiny-image-am335x-evm | 6秒          | 10秒             |
| 正常uBooty启动模式 | tisdk-base-image-am335x-evm | 6秒          | 40~60秒          |
| Falcon模式         |                             |              |                  |

Out-of-the-Box (OOB) boot times of the TI Processor Linux SDK

Multi-user refers to an Initialization Run Time Target or Run Level. This is the initialization for the OOB PLSDK. 

Single-user refers to shell.

# 官方README.falcon

https://github.com/u-boot/u-boot/blob/master/doc/README.falcon

Falcon Mode relies on the SPL framework. In fact, to make booting faster,
U-Boot is split into two parts: the SPL (Secondary Program Loader) and U-Boot
image.

use the "spl export" command to generate the kernel parameters
area or the DT

However at the end of an succesful 'spl export' run it will print the
RAM address of temporary storage. The RAM address of FDT will also be
set in the environment variable 'fdtargsaddr', the new length of the
prepared FDT will be set in the environment variable 'fdtargslen'.
These environment variables can be used in scripts for writing updated
FDT to persistent storage.

```shell
setenv falcon_args_file args
setenv boot_os 1
setenv spl_load_image_fat_os uImage
setenv falcon_image_file uImage
```

![img](https://e2e.ti.com/cfs-file/__key/communityserver-discussions-components-files/791/falcon_5F00_mode3.test.png)



https://blog.csdn.net/donglicaiju76152/article/details/77920015

```shell
nand read 0x82000000 NAND.kernel
nand read 0x88000000 NAND.u-boot-spl-os
run nandargs
spl export fdt 0x82000000 - 0x88000000
md <address>
nand write <address> bootparms 0x4000
```

## 注意事项

需要使用uImage，不能使用zImage

The falcon mode is supported only by **uImage**. You should **make uImage** with the **LOADADDR=0x80008000**.

```shell
=> run args_mmc
=> run loadimage
8942296 bytes read in 610 ms (14 MiB/s)
=> run loadfdt
58129 bytes read in 56 ms (1013.7 KiB/s)
=> spl export fdt ${loadaddr} - ${fdtaddr}
```

https://e2e.ti.com/support/processors-group/processors/f/processors-forum/544446/how-to-use-falcon-mode-in-u-boot

```shell
./mkimage -A arm -O linux -T kernel -C none -a 0x80008000 -e
```

# fdtoverlay工具使用

Falcon mode assumes one dtb, so you either patch your based device tree or you manually use fdtoverlay to apply the overlay to the base dtb: voodoo@hestia:~$ fdtoverlay --help Usage: apply a number of overlays to a base blob fdtoverlay <options> [<overlay.dtbo> [<overlay.dtbo>]] <type> s=string,…

```shell
debian@beaglebone:~$ fdtoverlay --help
Usage: apply a number of overlays to a base blob
        fdtoverlay <options> [<overlay.dtbo> [<overlay.dtbo>]]

<type>  s=string, i=int, u=unsigned, x=hex
        Optional modifier prefix:
                hh or b=byte, h=2 byte, l=4 byte (default)

Options: -[i:o:vhV]
  -i, --input <arg>  Input base DT blob
  -o, --output <arg> Output DT blob
  -v, --verbose      Verbose messages
  -h, --help         Print this help and exit
  -V, --version      Print version and exit

#下面命令将am335x-boneblack-uboot.dtb这个基础设备树上，添加了4层设备配置节点，生成了一个新的am335x-falcon.dtb设备树
fdtoverlay -i 
am335x-boneblack-uboot.dtb 
BB-BONE-eMMC1-01-00A0.dtbo 
BB-HDMI-TDA998x-00A0.dtbo
BB-ADC-00A0.dtbo
BB-UART1-00A0.dtbo
-o am335x-falcon.dtb
```

# 设备树DTB

```shell
$ sudo apt-get install device-tree-compiler

Device trees do not need to be compiled with "architecture-aware" tools. The dtc compiler on your ubuntu machine is probably current enough to compile your device tree. Or you can download the latest source and compile it yourself. The dtc compiler can be found here:

https://git.kernel.org/pub/scm/utils/dtc/dtc.git

There are some good documents in that package that will help you better understand device trees in general.

It's pretty easy to compile (and disassemble) device trees. For example

$ dtc -O dtb -o p4080ds.dtb p4080ds.dts
To get the device tree in text from from the device tree blob, do this:

$ dtc -I dtb -O dts p4080ds.dtb
```

On linux we can directly open dtb file by using **fdtdump**

```
fdtdump dtb_file.dtb > /tmp/test.txt 
```

https://elinux.org/Device_Tree_Usage

https://elinux.org/Device_Tree_Reference

# 减少启动时间

## 从硬件角度

1、从更快的存储设备上启动

2、Power Management Integrated Circuits (PMIC)

3、Processor Operating Performance Point (OPP)

AM335x from PORz runs ROM at 600MHz OPP100 voltage.After MLO loads and runs, the processor clock and voltages are set as needed.

## 从软件角度

### uBoot阶段

- [ ] 使用Falcon模式，跳过uBoot，直接加载内核
- [ ] 内核命令行参数添加quiet
- [ ] Add loops per jiffy LPJ to the command line （在BBB板子上测试了两次，都是lpj=4980736）
- [ ] DisableConsole - Turn off serial console output during boot
### kernel阶段 

- [ ] 修改内核编译选项，减少内核大小
- [ ] 使用压缩的内核镜像，切换内核压缩方式到LZ4
- [ ] 修改设备树只开启必要的功能
- [ ] 选择哪一些驱动被初始化
- [ ] No Probe Missing Devices - Disable probes for non-existent devices (including keyboards, etc.)
- [ ] Load Drivers Later - Use modules where possible to move driver initialization later in the boot sequence

### rc脚本


- [ ] 减少rc脚本
- [ ] 延后rc脚本

### 用户空间


- [ ] 用户空间使用单用户，不要使用多用户模式

- [ ] 文件系统分区，将只读分区与写分区隔离

- [ ] Avoid writes to flash memory

- [ ] Keep writable files in RAM, and write them to flash after boot

- [ ] Stripping your program,To get the highest savings, use "strip --strip-unneeded <app>"

- [ ]  Compiler options for program size,You can use "gcc -Os" to optimize for size.



# LPJ值

https://elinux.org/Preset_LPJ

测试了两次，都是lpj=4980736

```shell
May 24 14:08:04 user.info kernel: [    0.000000] OMAP clockevent source: timer2 at 24000000 Hz
May 24 14:08:04 user.info kernel: [    0.000014] sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 89478484971ns
May 24 14:08:04 user.info kernel: [    0.000033] clocksource: timer1: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
May 24 14:08:04 user.info kernel: [    0.000041] OMAP clocksource: timer1 at 24000000 Hz
May 24 14:08:04 user.crit kernel: [    0.000283] timer_probe: no matching timers found
May 24 14:08:04 user.info kernel: [    0.000457] Console: colour dummy device 80x30
May 24 14:08:04 user.err kernel: [    0.000493] WARNING: Your 'console=ttyO0' has been replaced by 'ttyS0'
May 24 14:08:04 user.err kernel: [    0.000499] This ensures that you still see kernel messages. Please
May 24 14:08:04 user.err kernel: [    0.000503] update your kernel commandline.
May 24 14:08:04 user.info kernel: [    0.000549] Calibrating delay loop... 996.14 BogoMIPS (lpj=4980736)
May 24 14:08:04 user.info kernel: [    0.089150] pid_max: default: 32768 minimum: 301
May 24 14:08:04 user.info kernel: [    0.089359] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
May 24 14:08:04 user.info kernel: [    0.089372] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
May 24 14:08:04 user.info kernel: [    0.090211] CPU: Testing write buffer coherency: ok
May 24 14:08:04 user.info kernel: [    0.090275] CPU0: Spectre v2: using BPIALL workaround
```

# Library Optimizer Tool

http://libraryopt.sourceforge.net/

The Library Optimizer Tool is used to reduce the size of shared libraries for an embedded system or other size-contrained environment.

# Static Linking

If your set of applications is small, sometimes it makes more sense to statically link your applications than to use shared libraries. Shared libraries by default include all symbols (functions and data structures) for the features a library provides. However, when you static link a program to a library, only the symbols that are actually referenced are linked in and included in the program.

# /proc/meminfo

https://lwn.net/Articles/28345/

# MTD

http://www.linux-mtd.infradead.org/faq/general.html

http://www.linux-mtd.infradead.org/doc/ubi.html

http://www.linux-mtd.infradead.org/faq/ubi.html

# mmap函数

Using mmap() instead of read() for initial application data load

An application may load a large amount of data when it is first initialized. This can result in a long delay as the file data is read into memory. It is possible to avoid the initial cost of this read, by using mmap() instead of read().

Instead of loading all of the data into memory with the read system call, the file can be mapped into memory with the mmap system call. Once the data file is mapped, individual pages will be demand loaded during execution, when the application reads them. Depending on the initial working set size of the data in the file, this can result in significant time savings. (For example, if an application only initially uses 50% of the data from the file, then only 50% of the data will be read into memory from persistent storage. There is extra overhead due to the cost of page-faults incurred in loading the pages on demand. However, this page fault overhead is offset by the savings in the number of page reads (compared to the read() case).



http://0pointer.de/blog/projects/systemd.html

Starting more in parallel means that if we have to run something, we should not serialize its start-up (as sysvinit does), but run it all at the same time, so that the available CPU and disk IO bandwidth is maxed out, and hence the overall start-up time minimized.



Traditionally on Unix a process that does double-forking can escape the supervision of its parent,

# tiny_fs启动日志

```shell
Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 5.4.106-g023faefa70 (oe-user@oe-host) (gcc version 9.2.1 20191025 (GNU Toolchain for the A-profile Architecture 9.2-2019.12 (arm-9.10))) #1 PREEMPT Mon May 24 09:04:10 UTC 2021
[    0.000000] CPU: ARMv7 Processor [413fc082] revision 2 (ARMv7), cr=10c5387d
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: TI AM335x BeagleBone Black
[    0.000000] Memory policy: Data cache writeback
[    0.000000] efi: Getting EFI parameters from FDT:
[    0.000000] efi: UEFI not found.
[    0.000000] cma: Reserved 48 MiB at 0x9c800000
[    0.000000] CPU: All CPU(s) started in SVC mode.
[    0.000000] AM335X ES2.1 (sgx neon)
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 129666
[    0.000000] Kernel command line: console=ttyO0,115200n8 root=/dev/nfs rw rootfstype=ext4 rootwait fixrtc nfsroot=192.168.31.85:/usr/local/ti-sdk-am335x-evm-07.03.00.005/filesystem/tisdk-tiny-image-am335x-evm,vers=3 ip=192.168.31.87:192.168.31.85:192.168.31.1:255.255.255.0::eth0:off
[    0.000000] Dentry cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.000000] Inode-cache hash table entries: 32768 (order: 5, 131072 bytes, linear)
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 453832K/523264K available (9216K kernel code, 295K rwdata, 3092K rodata, 1024K init, 254K bss, 20280K reserved, 49152K cma-reserved, 0K highmem)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] rcu: Preemptible hierarchical RCU implementation.
[    0.000000]  Tasks RCU enabled.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
[    0.000000] NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
[    0.000000] IRQ: Found an INTC at 0x(ptrval) (revision 5.0) with 128 interrupts
[    0.000000] random: get_random_bytes called from start_kernel+0x2b4/0x470 with crng_init=0
[    0.000000] OMAP clockevent source: timer2 at 24000000 Hz
[    0.000015] sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 89478484971ns
[    0.000033] clocksource: timer1: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
[    0.000042] OMAP clocksource: timer1 at 24000000 Hz
[    0.000286] timer_probe: no matching timers found
[    0.000457] Console: colour dummy device 80x30
[    0.000493] WARNING: Your 'console=ttyO0' has been replaced by 'ttyS0'
[    0.000498] This ensures that you still see kernel messages. Please
[    0.000502] update your kernel commandline.
[    0.000550] Calibrating delay loop... 996.14 BogoMIPS (lpj=4980736)
[    0.089147] pid_max: default: 32768 minimum: 301
[    0.089357] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.089369] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.090201] CPU: Testing write buffer coherency: ok
[    0.090268] CPU0: Spectre v2: using BPIALL workaround
[    0.091064] Setting up static identity map for 0x80100000 - 0x80100060
[    0.091200] rcu: Hierarchical SRCU implementation.
[    0.091276] EFI services will not be available.
[    0.091648] devtmpfs: initialized
[    0.101487] VFP support v0.3: implementor 41 architecture 3 part 30 variant c rev 3
[    0.101853] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.101874] futex hash table entries: 256 (order: -1, 3072 bytes, linear)
[    0.105462] pinctrl core: initialized pinctrl subsystem
[    0.106218] DMI not present or invalid.
[    0.106672] NET: Registered protocol family 16
[    0.108840] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.132269] l3-aon-clkctrl:0000:0: failed to disable
[    0.134331] cpuidle: using governor ladder
[    0.134359] cpuidle: using governor menu
[    0.149334] No ATAGs?
[    0.149344] hw-breakpoint: debug architecture 0x4 unsupported.
[    0.164048] debugfs: Directory '49000000.edma' with parent 'dmaengine' already present!
[    0.164083] edma 49000000.edma: TI EDMA DMA engine driver
[    0.165756] iommu: Default domain type: Translated
[    0.167761] SCSI subsystem initialized
[    0.168195] mc: Linux media interface: v0.10
[    0.168239] videodev: Linux video capture interface: v2.00
[    0.168326] pps_core: LinuxPPS API ver. 1 registered
[    0.168334] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.168353] PTP clock support registered
[    0.168384] EDAC MC: Ver: 3.0.0
[    0.169692] Advanced Linux Sound Architecture Driver Initialized.
[    0.170897] clocksource: Switched to clocksource timer1
[    0.177850] thermal_sys: Registered thermal governor 'fair_share'
[    0.177859] thermal_sys: Registered thermal governor 'bang_bang'
[    0.177875] thermal_sys: Registered thermal governor 'step_wise'
[    0.177881] thermal_sys: Registered thermal governor 'user_space'
[    0.177886] thermal_sys: Registered thermal governor 'power_allocator'
[    0.178442] NET: Registered protocol family 2
[    0.179511] tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 4096 bytes, linear)
[    0.179542] TCP established hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    0.179580] TCP bind hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    0.179617] TCP: Hash tables configured (established 4096 bind 4096)
[    0.179730] UDP hash table entries: 256 (order: 0, 4096 bytes, linear)
[    0.179748] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes, linear)
[    0.179909] NET: Registered protocol family 1
[    0.180480] RPC: Registered named UNIX socket transport module.
[    0.180493] RPC: Registered udp transport module.
[    0.180498] RPC: Registered tcp transport module.
[    0.180503] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.180518] PCI: CLS 0 bytes, default 64
[    0.181556] hw perfevents: enabled with armv7_cortex_a8 PMU driver, 5 counters available
[    0.182739] Initialise system trusted keyrings
[    0.183094] workingset: timestamp_bits=14 max_order=17 bucket_order=3
[    0.187549] squashfs: version 4.0 (2009/01/31) Phillip Lougher
[    0.188342] NFS: Registering the id_resolver key type
[    0.188381] Key type id_resolver registered
[    0.188388] Key type id_legacy registered
[    0.188429] ntfs: driver 2.1.32 [Flags: R/O].
[    0.189098] Key type asymmetric registered
[    0.189113] Asymmetric key parser 'x509' registered
[    0.189162] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 244)
[    0.189172] io scheduler mq-deadline registered
[    0.189179] io scheduler kyber registered
[    0.194203] OMAP GPIO hardware version 0.1
[    0.219145] omap-mailbox 480c8000.mailbox: omap mailbox rev 0x400
[    0.232000] pinctrl-single 44e10800.pinmux: 142 pins, size 568
[    0.278706] Serial: 8250/16550 driver, 10 ports, IRQ sharing enabled
[    0.283028] 44e09000.serial: ttyS0 at MMIO 0x44e09000 (irq = 29, base_baud = 3000000) is a 8250
[    0.910654] printk: console [ttyS0] enabled
[    0.917463] omap_rng 48310000.rng: Random Number Generator ver. 20
[    0.923850] random: fast init done
[    0.927477] random: crng init done
[    0.947065] brd: module loaded
[    0.956774] loop: module loaded
[    0.964693] libphy: Fixed MDIO Bus: probed
[    1.030923] davinci_mdio 4a101000.mdio: davinci mdio revision 1.6, bus freq 1000000
[    1.038628] libphy: 4a101000.mdio: probed
[    1.044140] davinci_mdio 4a101000.mdio: phy[0]: device 4a101000.mdio:00, driver SMSC LAN8710/LAN8720
[    1.053556] cpsw 4a100000.ethernet: initialized cpsw ale version 1.4
[    1.059940] cpsw 4a100000.ethernet: ALE Table size 1024
[    1.065342] cpsw 4a100000.ethernet: cpts: overflow check period 500 (jiffies)
[    1.072635] cpsw 4a100000.ethernet: Detected MACID = 64:33:db:30:5c:a6
[    1.081079] i2c /dev entries driver
[    1.087040] cpuidle: enable-method property 'ti,am3352' found operations
[    1.094525] sdhci: Secure Digital Host Controller Interface driver
[    1.100737] sdhci: Copyright(c) Pierre Ossman
[    1.106464] omap_gpio 44e07000.gpio: Could not set line 6 debounce to 200000 microseconds (-22)
[    1.115257] omap_hsmmc 48060000.mmc: Got CD GPIO
[    1.171240] omap_hsmmc 47810000.mmc: RX DMA channel request failed
[    1.177990] sdhci-pltfm: SDHCI platform and OF driver helper
[    1.186248] ledtrig-cpu: registered to indicate activity on CPUs
[    1.196794] davinci-mcasp 48038000.mcasp: IRQ common not found
[    1.204286] NET: Registered protocol family 10
[    1.210089] Segment Routing with IPv6
[    1.214008] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    1.220619] NET: Registered protocol family 17
[    1.225569] Key type dns_resolver registered
[    1.230093] omap_voltage_late_init: Voltage driver support not added
[    1.237154] Loading compiled-in X.509 certificates
[    1.267316] mmc1: new high speed MMC card at address 0001
[    1.273483] mmcblk1: mmc1:0001 M62704 3.56 GiB
[    1.278259] mmcblk1boot0: mmc1:0001 M62704 partition 1 2.00 MiB
[    1.284810] mmcblk1boot1: mmc1:0001 M62704 partition 2 2.00 MiB
[    1.291419] mmcblk1rpmb: mmc1:0001 M62704 partition 3 512 KiB, chardev (243:0)
[    1.302104]  mmcblk1: p1
[    1.311542] tps65217 0-0024: TPS65217 ID 0xe version 1.2
[    1.453148] tda998x 0-0070: found TDA19988
[    1.460370] tilcdc 4830e000.lcdc: bound 0-0070 (ops tda998x_ops)
[    1.466493] [drm] Supports vblank timestamp caching Rev 2 (21.10.2013).
[    1.473148] [drm] No driver support for vblank timestamp query.
[    1.479736] [drm] Initialized tilcdc 1.0.0 20121205 for 4830e000.lcdc on minor 0
[    1.624968] Console: switching to colour frame buffer device 240x67
[    1.664525] tilcdc 4830e000.lcdc: fb0: tilcdcdrmfb frame buffer device
[    1.671308] omap_i2c 44e0b000.i2c: bus 0 rev0.11 at 400 kHz
[    1.678948] omap_i2c 4819c000.i2c: bus 2 rev0.11 at 100 kHz
[    1.689514] hctosys: unable to open rtc device (rtc0)
[    1.696161] cpsw 4a100000.ethernet: initializing cpsw version 1.12 (0)
[    1.801937] SMSC LAN8710/LAN8720 4a101000.mdio:00: attached PHY driver [SMSC LAN8710/LAN8720] (mii_bus:phy_addr=4a101000.mdio:00, irq=POLL)
[    3.921758] cpsw 4a100000.ethernet eth0: Link is Up - 100Mbps/Full - flow control off
[    3.950973] IPv6: ADDRCONF(NETDEV_CHANGE): eth0: link becomes ready
[    3.981089] IP-Config: Complete:
[    3.984352]      device=eth0, hwaddr=64:33:db:30:5c:a6, ipaddr=192.168.31.87, mask=255.255.255.0, gw=192.168.31.1
[    3.994813]      host=192.168.31.87, domain=, nis-domain=(none)
[    4.000767]      bootserver=192.168.31.85, rootserver=192.168.31.85, rootpath=
[    4.008715] ALSA device list:
[    4.011980]   No soundcards found.
[    4.049139] VFS: Mounted root (nfs filesystem) on device 0:17.
[    4.055963] devtmpfs: mounted
[    4.062264] Freeing unused kernel memory: 1024K
[    4.081524] Run /sbin/init as init process
INIT: version 2.96 booting
```

