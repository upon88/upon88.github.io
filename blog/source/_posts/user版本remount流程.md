---
layout: android
title: user版本remount流程
date: 2021-12-09 17:14:24
tags:
---

先进行unlock操作。可按如下操作进行：
1.进入设置，开发者模式，打开OEM unlocking选项.
2.adb reboot bootloader进入fastboot模式.
3.连接电脑，输入命令fastboot flashing unlock，然后根据命令按音量上键解锁.
出现如下提示算成功：
4.成功之后输入fastboot reboot重启手机.
5.开机之后，依次adb root，adb disable-verity，adb reboot
6.开机之后adb root，adb remount.此时成功会提示：
然后正常push即可。
注：刷机选择升级刷机方式，可不用1-4步骤操作，直接执行5-6即可。
