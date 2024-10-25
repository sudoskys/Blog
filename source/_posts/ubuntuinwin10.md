---
title: 乌班图（Win10子系统）安装配置实践记录
date: 2021-3-13 08:28:10
tags: Ubuntu
---

> Win10 子系统的优势在于可以访问 Win10 的文件系统和目录，并实现 CPU 和内存的动态调度，速度比虚拟机快。
> 
> 缺点是没有 Ubuntu 的桌面，搭建麻烦，不适合图像开发。适用于后台开发。

## 微软配置

1. 打开设置 -> 安全和更新 -> 开发者选项，选择“开发人员模式”。
2. 在设置中搜索“Windows功能”，选择“启用或关闭 Windows 功能”。
3. 勾选“适用于 Linux 的 Windows 子系统”，并确定。

## 安装方式

### 方式一：手动下载

[手动下载 WSL 发行版](https://docs.microsoft.com/en-us/windows/wsl/install-manual)

### 方式二：使用 LxRunOffline

1. 安装 LxRunOffline（下载、解压、配置环境变量），[下载网址](https://github.com/DDoSolitary/LxRunOffline/releases)。
2. 下载 WSL（修改后缀为 zip，然后解压）。
3. 使用 LxRunOffline 安装。

```shell
# 示例命令
lxrunoffline i -n ubuntu -d D:\MyApp\Ubuntu18 -f D:\MyApp\Ubuntu18\install.tar.gz
```

### 方式三：应用商店安装

打开微软商店，搜索并安装 Ubuntu。

## 配置 LxRunOffline 环境变量

1. 右击“此电脑” -> 属性 -> 高级系统设置 -> 环境变量。
2. 在系统变量中编辑 Path，添加 LxRunOffline 的解压路径。
3. 重启电脑，验证 LxRunOffline 是否可用。

## 迁移乌班图

使用 `LxRunOffline move` 命令移动子系统到新目录。

```shell
LxRunOffline move -n Ubuntu-18.04 -d D:\WinLinux
```

## 安装后初始设置

### 设置 Ubuntu 账户

输入密码时不会显示，直接输入并回车即可。

### 访问 Win10 文件系统

在 Ubuntu 中通过 `/mnt/小写盘符/路径` 访问 Win10 路径。

### 设置初始 root 密码

```shell
sudo passwd
```

### 配置软件源

备份并修改 `/etc/apt/sources.list`，使用国内镜像源。

```shell
# 示例配置
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
```

执行 `sudo apt-get update` 更新软件包列表。

## 选做：图形化界面

### 在 Ubuntu 端配置

1. 安装 xorg 和 xfce4
   ```shell
   sudo apt-get install xorg xfce4
   ```
2. 安装并配置 xrdp
   ```shell
   sudo apt-get install xrdp
   sudo sed -i 's/port=3389/port=3390/g' /etc/xrdp/xrdp.ini
   echo xfce4-session >~/.xsession
   sudo service xrdp restart
   ```

### 在 Windows 端配置

1. 搜索并打开“远程桌面”。
2. 配置连接信息并连接到 Ubuntu。

## 常见错误处理

### Permission denied

设置 root 密码：

```shell
sudo passwd
```

### Unable to locate package

执行以下命令更新软件包列表：

```shell
sudo apt-get update
```

## 参考

- [Win10 Linux 安装目录更改](https://blog.csdn.net/weixin_29517217/article/details/116632530)
- [Win10 搭建 Ubuntu 子系统](https://www.cnblogs.com/lyongyong/p/14595816.html)
