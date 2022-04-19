+++
title = "ZeroTier内网穿透"
description = "ZeroTier内网穿透"
tags = [
    "zerotier",
    "内网穿透",
    "远程控制",
]
date = "2022-04-17"
categories = [
    "工具",
]
menu = "main"

weight= 10

+++
# ZeroTier内网穿透

## 安装方式1：脚本自动安装

```shell
sudo curl -s https://install.zerotier.com|sudo bash
```

在ubuntu下成功安装，在树莓派上可能失败

## 安装方式2：源码安装

```shell
wget https://github.com/zerotier/ZeroTierOne/archive/refs/tags/1.8.4.tar.gz
tar zxvf 1.8.4.tar.gz
cd ZeroTierOne-1.8.4/
make -j
make install
```

## 注册账号

申请ID,参考https://blog.csdn.net/kai3123919064/article/details/109662499

## 运行

1、在每个设备端上运行服务端zerotier-one，并将每一个设备添加到相同的NetworkID组，这样同一下
NetworkID组下的所有成员就能相互穿透了

先安装zerotier-one服务并运行，这样以后开机自动启动此服务

```shell
#在源码目录下可以看到此文件
~/Public/ZeroTierOne-1.8.4 $ cat debian/zerotier-one.service
[Unit]
Description=ZeroTier One
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/sbin/zerotier-one
Restart=always
KillMode=process

[Install]
WantedBy=multi-user.target
```

注意检查上面的/usr/sbin/zerotier-one路径是否存在

```shell
#将此文件 拷贝到/lib/systemd/system/目录下
$ cp debian/zerotier-one.service  /lib/systemd/system/
#启动服务
$ systemctl start zerotier-one.service
#查看服务状态
$ systemctl status zerotier-one.service
● zerotier-one.service - ZeroTier One
   Loaded: loaded (/lib/systemd/system/zerotier-one.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2022-03-17 22:22:47 CST; 3 days ago
 Main PID: 487 (zerotier-one)
    Tasks: 5 (limit: 2059)
   CGroup: /system.slice/zerotier-one.service
           └─487 /usr/sbin/zerotier-one
```

2、在**每个设备**都添加到自己的局域网，每台设备都执行以下命令后，这样在相同your_network_id下的每个设备之间就组成局域网了，太简单了，但是还需要在下一步中的网页端上进行确认

```shell
sudo zerotier-cli join your_network_id
```

3、在网页端中https://my.zerotier.com/network/your_network_id，启动刚刚添加的设备

![image-20220321195005343](/images/zerotier内网穿透/image-20220321195005343.png)

4、在设备端可以看到，新增了一个网络接口和相应的路由表，并且IP地址显示为上图网页管理界面中的IP

![image-20220321195019361](/images/zerotier内网穿透/image-20220321195019361.png)

![image-20220321195039280](/images/zerotier内网穿透/image-20220321195039280.png)

5、此时网页管理界面中显示的两个成员就能相互访问了

```shell
ssh root@192.168.192.212
```

**perfect!!!**

