---
title: fastboot刷机常用命令
date: 2021-12-09 16:53:52
tags: fastboot
---
# 重启进入fastboot模式
```shell
adb reboot bootloader
```

# 解锁
解锁BootLoader的方法是在开发者模式中开起OEM unlocking开关
```shell
fastboot devices  //查看设备
fastboot flashing unlock //解锁
```
# fastboot关闭 AVB
```shell
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
```
或者：
```shell
fastboot --disable-verity flash vbmeta vbmeta.img
fastboot --disable-verity flash vbmeta_system vbmeta_system.img
```
注意：关闭avb可能失败，还没去找解决方案。

# 擦除
```shell
fastboot erase system
fastboot erase cache
fastboot erase config
fastboot erase data
fastboot erase logs
fastboot erase factory
```

# 刷机
```shell
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash recovery recovery.img
```

# 重启
```shell
fastboot reboot
```
