---
title: 记录一次修复 Arch Linux 的体验
date: 2024-10-24 12:00:00
tags: Arch Linux
---

## 前言

在更新过程中中断软件包的更新，导致系统无法启动。可能是由于断电、重启或硬盘问题引起的。

## 解决方案

准备一个 Live CD，启动进入 Live CD。插上手机打开数据线网络共享。如果没有手机，可以用如下方法连接 WiFi。

```shell
iwctl # 进入交互式命令行
device list # 列出无线网卡设备名，比如无线网卡叫 wlan0
station wlan0 scan # 扫描网络
station wlan0 get-networks # 列出所有 WiFi 网络
station wlan0 connect wifi-name # 进行连接，注意这里无法输入中文。回车后输入密码即可
exit # 连接成功后退出
```

如果你的手机只有 CN 网络，请删除镜像列表：

```shell
rm /etc/pacman.d/mirrorlist
```

然后添加中科大镜像源：

```shell
echo "Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch" > /etc/pacman.d/mirrorlist
```

输入 `fdisk -l` 查看硬盘信息，然后输入 `lsblk` 查看硬盘分区信息。

### 挂载硬盘

将对应的硬盘分区挂载到 `/mnt` 目录。每台机器的硬盘分区都不一样，我的硬盘是 btrfs 文件系统，所以挂载如下：

```shell
mount -t btrfs -o subvol=/@,compress=zstd /dev/nvme0n1p3 /mnt # 挂载 / 目录
mount -t btrfs -o subvol=/@home,compress=zstd /dev/nvme0n1p3 /mnt/home # 挂载 /home 目录
mount /dev/nvme0n1p1 /mnt/boot # 挂载 /boot 目录
swapon /dev/nvme0n1p2 # 挂载交换分区
```

不要乱挂载，输入错误请输入 `umount -R /mnt` 卸载所有分区。

### 检查 fstab 文件

在 Live CD 下，输入 `cat /mnt/etc/fstab` 检查 fstab 文件。

如果 fstab 文件损坏，请输入 `nano /mnt/etc/fstab` 编辑 fstab 文件。

或者输入 `genfstab -U /mnt > /mnt/etc/fstab` 生成 fstab 文件。

### 尝试进入系统并更新

输入 `arch-chroot /mnt` 进入系统。
运行 `pacman -Syu` 更新系统。

如果是引导程序损坏，输入以下命令安装 GRUB 到 EFI 分区：

```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
```

为了引导 Windows 10，还需要添加新的一行：

```shell
vim /etc/default/grub
```

添加 `GRUB_DISABLE_OS_PROBER=false` 后保存退出。

然后输入 `grub-mkconfig -o /boot/grub/grub.cfg` 生成 GRUB 配置文件。检查输出是否匹配。

然后输入 `exit` 退出 chroot 环境。

### Pacman 不能在 chroot 环境使用

如果遇到以下错误：

```shell
ldconfig: File /usr/lib/libxxx.so.1.0.2 is empty, not checked.
```

```shell
pacman: error when loading shared libraries: /usr/lib/libbz2.so.1.0 file too short
```

通常是更新时系统中断，导致 pacman 损坏。输入 `exit` 退出 chroot 环境。

先删除 `/mnt/var/lib/pacman/db.lck` 文件来删除数据库锁。

然后输入以下命令挂载目录：

```shell
mount -t proc proc /mnt/proc
mount -t sysfs sysfs /mnt/sys
mount -t dev dev /mnt/dev
```

输入以下命令重新安装系统中的所有软件包：

```shell
pacman --root /mnt -Qnq | pacman --cachedir=/mnt/var/cache/pacman/pkg --root /mnt -S --dbonly -
pacman --root /mnt -Qnq | pacman --cachedir=/mnt/var/cache/pacman/pkg --root /mnt -S -
```

或者：

```shell
pacman --sysroot /mnt -Syyu $(pacman --sysroot /mnt -Qnq) --overwrite '*'
```

参考链接：

- [Arch Linux 论坛](https://bbs.archlinux.org/viewtopic.php?id=293993)
- [Gist 参考](https://gist.github.com/metzenseifner/cb61ecfd614a93c5927ba3cd62d68127)

如果遇到密钥问题，请看下面的密钥问题部分。

然后进入 chroot 环境：

```shell
arch-chroot /mnt
```

再次尝试更新：

```shell
pacman -Syy
pacman -S archlinux-keyring
pacman -Su
```

### 密钥问题

如果遇到以下错误：

```shell
error: key "XXXX" could not be looked up remotely
error: required key missing from keyring
error: failed to commit transaction (unexpected error)
Errors occurred, no packages were upgraded.
error: failed to commit transaction (invalid or corrupted package (PGP signature))
gnupg/pubring.gpg
```

请参考文档：[GnuPG](https://wiki.archlinux.org/title/GnuPG)

遇到任何密钥问题，下面命令可以解决。

首先需要进入 chroot 环境：

```shell
arch-chroot /mnt
```

然后输入以下命令：

```shell
pacman -Sy archlinux-keyring
pacman-key --init
mv /etc/pacman.d/gnupg /etc/pacman.d/gnupg.bak # 如果 init 失败，可以执行这行
pacman-key --populate
pacman-key --refresh-keys
```

如果你在 Live CD 下遇到密钥问题，请输入以下命令：

```shell
pacman -S archlinux-keyring
pacman-key --init
pacman-key --populate
```

然后重新按照上面的文章步骤操作。

### 完成

```shell
exit # 退回安装环境
umount -R /mnt # 卸载新分区
reboot # 重启
```
