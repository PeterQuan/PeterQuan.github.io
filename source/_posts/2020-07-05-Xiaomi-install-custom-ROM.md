---
layout:       post
title:        "小米6刷机"
subtitle:     "小米刷欧版miui"
date:         2020-07-05
updated:      2020-07-05
author:       "权芹乐"
catalog:      true
tags:
    - 刷机
    - Android
---

[toc]

# 刷官方ROM

## 下载官方ROM
+ 下载所需官方版ROM
+ 复制到手机存储，路径任意，根目录即可

## 开启隐藏功能【手动选择安装包】
+ MIUI版本 界面，连续点击中间“11”图标
+ 点击右上角【...】，手动选择安装包、重启到Recovery出现

# 刷第三方ROM

第三方ROM，例如，[xda](https://forum.xda-developers.com/mi-6/development/rom-evolution-x-4-20-2-sagit-t4089445) 或 [eu](https://xiaomi.eu/community/threads/miui-11-0-stable-release.52628/)

## 1. 开启`USB调试`
开启`开发者模式`：我的设备 → 全部参数 → MIUI版本 连续点击7次

开启`USB调试`：更多设置 → 开发人员选项 → USB调试模式。

手机USB插到电脑上。

## 2. 解锁BL（Bootloader）
http://www.miui.com/unlock/download.html

## 3. 安装TWRP

> What is TWRP? TWRP is a Custom Recovery.
>
> 存在多种操作方式，其中，手机上安装app的方式需要root手机，所以，改用命令行`adb+fastboot`的方式

+ 进入Fastboot

`adb reboot bootloader`
如果未按预期进入Fastboot，则，先手动关机，再长按 `音量下`+`电源` 开机

+ 下载合适的 `twrp.img`

https://twrp.me/xiaomi/xiaomimi6.html

+ 刷twrp.img
```
fastboot flash recovery twrp.img
```

+ 进入 Recovery TWRP
```
fastboot boot twrp.img
// OR
fastboot reboot
```
如果未按预期进入TWRP，则，在Fastboot模式下，长按`音量上`+`电源`，等看到小米log时，只松开`电源`键，直到出现TWRP，再松开`音量上`。

清数据，FORMAT /data partition (NEVER wipe System or Persist!)

+ 刷ROOM镜像

把ROOM镜像放到手机存储

+ 完成


# 其他

## 新系统引导界面，卡在Google账号验证
Tip: 刷机前，应在老系统中退出Google账号登陆，否则就会遇到这个问题。  
如果你的网络可以穿墙，在手机尚无法安装设置其他app的情况下，其实这也不算个事儿。

解决办法：关闭新系统新机引导界面
1. 进入 Recovery TWRP
2. 在菜单中完成/system分区。以防万一，adb中再次挂载`adb remount /system`
3. 进入shell，关闭引导
```
adb shell
echo "ro.setupwizard.mode=DISABLED" >> /system/build.prop
```
4. 系统重启
