---
layout:       post
title:        "Raspberry | 树莓派4b系统安装、配置"
subtitle:     "新购8G内存版本的树莓派4b，安装、配置过程记录"
date:         2020-12-21
updated:      2020-12-29
author:       "权芹乐"
header-img:   "img/in-post/raspberry-pi-4b.webp"
catalog:      true
tags:
    - Raspberry 4b
    - 树莓派
---

[toc]

# 购买树莓派的原因

* 家里没有NAS，想随时访问那几块闲置的移动硬盘不方便
* 给小米电视额外增加一个输入源
* 希望从外网访问家里的网络
* 通过树莓派下载视频到移动硬盘

# 硬件情况

* 2020年产 Raspberry 4b，内存4G
* 32G TF存储卡
* 安装系统、配置过程中，树莓派不外接 键盘、鼠标、显示器
* 存储卡插入笔记本
* 树莓派连接笔记本提供的无线热点
* 在笔记本上，ssh连接树莓派完成安装配置操作

# 选择系统

网上看了一些文章说，4G内存的树莓派运行64位系统完全没问题，所以，我选择了64位系统。

先是尝试安装了`Ubuntu 64位LST版`（[安装包]），过程很不顺利，放弃了。

[安装包]:https://ubuntu.com/download/raspberry-pi

最终安装的`官方64位版OS`。

所以，先在笔记本上下载[raspios_arm64 zip包][1]。

[1]:https://downloads.raspberrypi.org/raspios_arm64/images/

# 写入OS

官方提供的烧录OS工具 [Raspberry Pi Imager][2] 非常简单、易用，但是，对于我来说不适用，原因是其一它没有提供官方的64位系统，其二使用这种方法的话，我没有找到在不外接键盘、鼠标情况下，访问系统的方法。

[2]:https://www.raspberrypi.org/software/

我使用的工具是[Win32DiskImager][3]，将系统zip写入存储卡。

[3]:https://www.raspberrypi.org/documentation/installation/installing-images/windows.md

# 预配置

在笔记本上打开存储卡，在boot目录下
* 开启ssh访问：新建空文件`ssh`
* 配置热点连接：新建`wpa_supplicant.conf`文件，内容如下

```txt
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=CN
ap_scan=1
fast_reauth=1

network={
  ssid="hotspot-name-on-your-pc"
  psk="hotspot-password"
  priority=1
  id_str="hotspot"
}
network={
  ssid="wifi-name"
  psk="wifi-password"
  priority=2
  id_str="home5g"
}
```

官方对无线网配置的讲解：
https://www.raspberrypi.org/documentation/configuration/wireless/headless.md
https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md

注意：psk加双引号直接写明文密码，不加双引号则需要转换成32位预加密`wpa_passphrase "your pasword"`

# 启动Pi

先将内存卡插入树莓派，再插入电源后，树莓派将会自动启动。

# 登录Pi

在热点管理or路由器管理界面，查找树莓派的IP。

`ssh`访问树莓派：
```
ssh pi@IP-of-raspberry
```
树莓派官方系统的默认登录名和密码是`pi / raspberry`。

# apt改国内源

本想换成国内的源，尝试了阿里和清华的源，暂时都不支持arm64位，所以，使用默认的吧。如果嫌慢可以挂vpn，会好很多。

# pip/pip3改国内源

```
mkdir ~/.pip
vi ~/.pip/pip.conf

[global]
timeout=100
index-url=http://mirrors.aliyun.com/pypi/simple/
extra-index-url=https://pypi.tuna.tsinghua.edu.cn/simple/
[install]
trusted-host=
    mirrors.aliyun.com
    pypi.tuna.tsinghua.edu.cn
```

# 配置：开启VNC、设置分辨率，等

```
sudo raspi-config
```
使用方向键控制光标，Enter键选择：

* 在`Interfacing Options`中，找到`VNC`，选择`Yes`
* 在`Boot Options`中，找到`Desktop/CLI`，选择`Desktop Autologin`
* 在`Advanced Options`中，找到`Resolution`，选择`1920*1080`
* 在`Advanced Options`中，选择`Expand Filesystem`
* 通过`Tab`键，选择`Finish`，`Enter`触发reboot

至此，树莓派配置完毕，重启后就可以vnc访问了，也可以连到电视上了。

# 【可选】安装aria
安装如下内容：
+ aria
+ AriaNg：前端web管理界面，推荐使用[All in One包](https://github.com/mayswind/AriaNg/releases)

开始时，使用`sudo apt install aria2`，自己修改配置，可惜下载速度不理想。然后，改用了git上的一个热门项目[“Aria2一键安装管理脚本（增强版）”](https://github.com/P3TERX/aria2.sh)

安装成功后，重现运行脚本，完成如下配置：
+ 选择`修改 配置`，再`修改 Aria2 下载目录`
+ 开启`自动更新 BT-Tracker`

此时，打开AriaNg网页，`Aria2 状态`一直显示“连接中”，并且，错误弹窗提示：认证失败!
> 解决办法
> 1. 打开配置文件`/root/.aria2c/aria2.conf`，找到`rpc-secret=`复制其内容
> 2. 在web中，打开`AriaNg 设置——RPC (localhost:6800)`，粘贴“Aria2 RPC 密钥”。该页面的url示例，形如，http://loclhost/AriaNg/index.html#!/settings/ariang
