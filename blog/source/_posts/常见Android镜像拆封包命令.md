---
title: 常见Android镜像拆封包命令
date: 2021-12-21 09:52:14
tags:
---
# 拆封ramdisk包
新建文件夹(以下雷同)
```shell
mkdir test;cd test;cp XXX/ramdisk.img ./
```
拆包：
```shell
mkdir rootfs;cp ramdisk.img rootfs/ ;cd rootfs/;mv ramdisk.img ramdisk.img.gz;gunzip ramdisk.img.gz
mkdir root;cd root;cpio -i -F ../ramdisk.img
```
封包：
```shell
cd root;find . | cpio -o -H newc >../tmp_ramdisk.img
cd .. ;gzip -c tmp_ramdisk.img > new_ramdisk.img
```
# 拆封boot.img包
新建文件夹
```shell
mkdir test;cd test;cp XXX/boot.img ./
```
拆包工具：unpack_bootimg（需要source/lunch/make unpack_bootimg生成即可）
```shell
# unpack_bootimg --boot_img boot.img --out boot-out/
boot_magic: ANDROID!
kernel_size: 17983189
kernel load address: 0x20280000
ramdisk size: 0
ramdisk load address: 0x0
second bootloader size: 0
second bootloader load address: 0x0
kernel tags load address: 0x20280100
page size: 2048
os version: 11.0.0
os patch level: 2021-11
boot image header version: 1
product name:
command line args: buildvariant=userdebug
additional command line args:
recovery dtbo size: 0
recovery dtbo offset: 0x0
boot header size: 1648
```
封包工具：mkbootimg（对应的参数根据解包的参数而定）
```shell
mkbootimg --cmdline 'buildvariant=userdebug' --kernel boot-out/kernel --ramdisk boot-out/ramdisk -o new_boot.img --base 0x20280000
```
注：（通过该命令打包之后，boot.img的大小对不上了，不清楚原因）
```shell
ls -l new_boot.img
-rw-rw-r-- 1 root root 17985536 12月 20 15:25 new_boot.img

ls -l boot.img
-rw-rw-r-- 1 root root 33554432 12月 20 15:10 boot.img
```
再解压，发现没有dtbo的打印
```shell
# unpack_bootimg --boot_img new_boot.img --out new_boot-out/
boot_magic: ANDROID!
kernel_size: 17983189
kernel load address: 0x20288000
ramdisk size: 0
ramdisk load address: 0x0
second bootloader size: 0
second bootloader load address: 0x0
kernel tags load address: 0x20280100
page size: 2048
os version: 0.0.0
os patch level: 2000-00
boot image header version: 0
product name:
command line args: buildvariant=userdebug
additional command line args:
```

# 拆dtbo.img包
提取dtb数据：
```shell
mkdtimg dump dtbo.img -b my.dtb
```
将dtb文件转换为dts文件：
```shell
dtc -I dtb -O dts my.dtb.0 -o my.dts
```
# 拆封system.img包
## 常见格式：
1、ext2/3/4
直接挂载：
```shell
mount -o loop system.img /mnt
```
Android 10以上
```shell
sudo mount -t ext4 -o ro system.img /mnt 由于vbmeta原因只能以只读的方式挂载
```
2、带MBR分区信息的镜像(system_qemu.img)
查看信息
```shell
fdisk -lu system_qemu.img
sgdisk --print system_qemu.img
```
使用dd命令拆解
```shell
name=vbmeta;start=2048;end=4095 (根据sgdisk命令获取)
dd if=system_qemu.img of=split/${name}.img skip=${start} bs=512 count=$((${end}-${start}+1))
name=super;start=4096;end=6311935 (根据sgdisk命令获取)
dd if=system_qemu.img of=split/${name}.img skip=${start} bs=512 count=$((${end}-${start}+1))
```
解压super.img(system.img+vendor.img)
```shell
lpunpack super.img
```
3、sparse image稀疏镜像格式
```shell
simg2img system.img system_new.img
sudo mount system_new.img /mnt
```
