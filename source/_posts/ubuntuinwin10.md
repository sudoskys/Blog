---
title: 乌班图（Win10子系统）安装配置实践记录
date: 2021-3-13 08:28:10
tags: Ubuntu
copyright_info: 原创编写|材料采集与引用已注明
copyright_author: LuoYing
---
> win10 子系统最大的方便之处在于能访问win10的文件系统和目录，其次能够实现win10的cpu 和内存的动态调度这个比虚拟机处理快很多了。
>
> 但是缺点是没有Ubuntu的桌面，搭建非常麻烦，不适合图像开发。
>
> 大多时候用于Ubuntu server的开发，后台开发还是非常合适的。
>
> <br />微欧小v—https://www.zhihu.com/question/449404869/answer/1920214736<br />
>

### 微软配置

打开设置->安全和更新->开发者选项，选择为“开发人员模式”，修改完成之后我们需要在设置中直接搜索“Windows功能”，选择“启用或关闭Windows功能”，然后下滑菜单找到“适用于Linux的Windows子系统” 勾选并确定。

然后我们可以开始准备安装子系统了，这个时候不要着急，需要下载两个东西。

# 安装方式一

### 乌班图系统下载（子系统版）

[Manually download Windows Subsystem for Linux (WSL) Distros | Microsoft Docs](https://docs.microsoft.com/en-us/windows/wsl/install-manual)

https://wsldownload.azureedge.net/Ubuntu_2004.2020.424.0_x64.appx

# 安装方式二

### 使用LxRunOffline安装WSL到任意位置

1. 安装LxRunOffline（下载、解压、配置环境变量即可），[下载网址](https://github.com/DDoSolitary/LxRunOffline/releases)，我下载的是3.5.0版本
2. 下载wsl（修改后缀为zip，然后解压）
3. 使用LxRunOffline安装（详见文末操作）

```python
# lxrunoffline i -n <WSL名称> -d <安装路径> -f <安装包路径>.tar.gz

C:\Users\15758>lxrunoffline i -n ubuntu -d D:\MyApp\Ubuntu18 -f D:\MyApp\Ubuntu18\install.tar.gz

[WARNING] Ignoring an unsupported file "dev/full" of type 0020000.
[WARNING] Ignoring an unsupported file "dev/null" of type 0020000.
[WARNING] Ignoring an unsupported file "dev/ptmx" of type 0020000.
[WARNING] Ignoring an unsupported file "dev/random" of type 0020000.
[WARNING] Ignoring an unsupported file "dev/tty" of type 0020000.
[WARNING] Ignoring an unsupported file "dev/urandom" of type 0020000.
[WARNING] Ignoring an unsupported file "dev/zero" of type 0020000.
[WARNING] Love this tool? Would you like to make a donation: https://github.com/DDoSolitary/LxRunOff
```

双击ubuntu1804.exe

# 安装方式三

## 应用商场安装

...打开微软自带商店安装

...

...

## LxRunOffline下载

下载地址在这

#### 二选一

https://raw.githubusercontent.com/DDoSolitary/LxRunOffline/releases/download/v3.5.0/LxRunOffline-v3.5.0-mingw.zip

https://github.com/DDoSolitary/LxRunOffline/releases

下载文件LxRunOffline-vxxxx.zip  然后解压到某个一个目录中，之后我们需要设置环境变量，用来移动我们的子系统，所以不可以删除.

#### 下载Mingw64

下载地址: http://www.mingw-w64.org/doku.php ,之后解压到相关路径。

PS:这里只需要解压到E:\Environment\mingw64里面，里面没有可执行文件哦。

> 在 Windows 环境下会优先使用 MSVC 作 linker，也就是 target 为 (arch)-pc-windows-msvc，然而其所需 Visual C++ Build Tools体积太大了，而且装在C盘。所以我们要使用MINGW64
>

[MingGW64 版本区别于各版本说明 - PCYO - 你是谁的，谁是谁，谁为谁憔悴](https://www.pcyo.cn/linux/20181212/216.html)

#### 配置Mingw64

解压到磁盘某个位置，比如你的安装目录，或者直接解压到 C 盘根目录也可以，然后把 Bin 目录（请把路径copy下来）添加到环境变量 Path（这个条目的里面），记得跟其它目录用 `;` 隔开，保存。

#### 配置LxRunOffline环境变量

右击”此电脑“->属性->高级系统设置，在高级面板中选择环境变量。选择系统变量中的Path变量，点击编辑按钮，点击新建，并把刚刚LxRunOffline的解压地址粘贴到新的项目中。切记这个目录里必须包含LxRunOffline.exe文件！

然后重启电脑，使环境变量生效，打开命令行工具，输入LxRunOffline，如果有提示证明成功了。

## 迁移乌班图

打开CMD，输入`LxRunOffline list` 查看我们的子系统版本，我安装的是ubuntu 20.04，记下我们的版本号.

**LxRunOffline move -n {version} -d {dir}**

在这里`{version}`填写我们刚才查到的版本号，`{dir}`改为我们需要移动到的目录

例如我要把Ubuntu-18.04移动到我新建的文件夹D:\WinLinux下

那么我需要输入

`LxRunOffline move -n Ubuntu-18.04 -d D:\WinLinux`

然后确认回车耐心等待移动结束就行。

## 安装后初始设置

> ### 乌班图账户设置
>
> 输密码的时候不会显示,你只要输入正确的密码,然后enter就行了
>
> ### 其他
>
> 1. 在Ubuntu中可以通过`/mnt/小写盘符/路径`访问到win10对应路径，例如`cd /mnt/d/workspace/`就是进入win10系统中D盘的workspace目录
> 2. win10向Ubuntu粘贴文本，在win上复制ok后，在Ubuntu的命令窗口右击鼠标完成粘贴
> 3. Ubuntu向win粘贴文本，鼠标选中需要复制的文本，同样鼠标右键完成复制
> 4. 子Linux系统和 win10 是使用的相同网络，端口也都是共用的，避免端口占用冲突
>

#### 设置初始 root 密码

`sudo passwd`

#### 配置软件源/加速国内访问速度

备份配置文件

`cp /etc/apt/sources.list /etc/apt/sources_bk.list`

修改配置文件

```shell

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

**执行sudo apt-get install更换了软件源，需要update**，请执行下面的命令：

`sudo apt-get update`

#### 选做图形化界面

**都用wsl了，还装桌面干嘛，没有任何桌面比得上windows**

##### 在 Ubuntu 端的配置

1、安装 xorg

`sudo apt-get install xorg`

xorg是xfce桌面需要的一个基础依赖，开机时提供登陆界面

![](https://pic1.zhimg.com/80/v2-d544732994a212b30e0547eb4167bc50_720w.jpg)

<br />

2、安装xfce4

`sudo apt-get install xfce4`

![](https://pic1.zhimg.com/80/v2-78e4f5d0f42ecb18b8a4f3087410a610_720w.jpg)

<br />

3、安装并配置xrdp

Xrdp 通过远程桌面的方式来访问另外一台主机

`sudo apt-get install xrdp`

4、设置使用3390端口

`sudo sed -i 's/port=3389/port=3390/g' /etc/xrdp/xrdp.ini`

5、向xsession中写入xfce4-session

`sudo echo xfce4-session >~/.xsession`

6、重启xrdp服务

`sudo service xrdp restart`

#### 在 Windows 端配置

1、`win`+`S`，搜索 `远程桌面`

![](https://pic1.zhimg.com/80/v2-c5ffbc80f758233f89d455b5cb0f87fc_720w.jpg)

<br />

2、配置连接信息

![](https://pic2.zhimg.com/80/v2-92ea5c2403aa70f072ea05a413bd21d9_720w.jpg)

<br />

3、运行连接，过程会有防火墙，同样允许就行

![](https://pic1.zhimg.com/80/v2-c6d8017c4b8eb1604973402b20536e6c_720w.jpg)

<br />

4、连接到 Ubuntu

![](https://pic2.zhimg.com/80/v2-a6a556429a0e05df4cc5eac52de24071_720w.jpg)

<br />

5、登录到 Ubuntu

![](https://pic3.zhimg.com/80/v2-3631b3fd799f7fd9c7634f995b97c0e6_720w.jpg)

<br />

6、登录后看到桌面，有那味儿了

![](https://pic4.zhimg.com/80/v2-ff771af077844000224481c9bc576717_720w.jpg)

<br />

7、打开本地的 windows 盘符，和终端看看

![](https://pic1.zhimg.com/80/v2-93b254fd761c15210eccd5747ed2efc4_720w.jpg)

<br />

#### 有需求人士选做：配置adb环境

......前提是win10有安装adb环境，(不清楚的搜一搜)

<1>将adb.exe、AdbWinApi.dll、AdbWinUsbApi.dll、fastboot.exe

　　拷贝到

　　`C:WindowsSystem32`和`C:WindowsSysWOW64`目录下

<2>配置Linux子系统

　　`# sudo cp adb fastboot /usr/local/bin`

## 参考/摘录

[win10 linux 安装目录更改,Win10更换默认子系统安装位置 Ubuntu Linux win10子系统瞎折腾系列..._深深妈妈在澳洲的博客-CSDN博客](https://blog.csdn.net/weixin_29517217/article/details/116632530)

[win10搭建Ubuntu子系统（wls）+配置adb环境 - 勇~勇 - 博客园 (cnblogs.com)](https://www.cnblogs.com/lyongyong/p/14595816.html)

[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

### 双系统指路

[2020最详细安装Ubuntu指南 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/135953477)

### 虚拟机指路

[史上最全最新Ubuntu20.04安装教程（图文） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/355314438)

[Ubuntu安装最佳实践（防踩坑指南） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/114518326)

[玩转 Windows 自带的 Linux 子系统 （图文指南） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/258563812)

## 常见错误处理

### Permission denied的解决办法

ubuntu安装好后，root初始密码（默认密码）不知道，需要设置。先用安装ubuntu的时候创建的用户登录到系统，然后输入命令：`sudo passwd`  ，回车，接下来会提示您：输入新密码

重复输入密码，最后提示您passwd：password updated sucessfully
此时已完成root密码的设置
接着输入命令：`su root`<br />

## Unable to locate package错误

**执行sudo apt-get install更换了软件源，需要update**，请执行下面的命令：

`sudo apt-get update`

[unable to locate package-cnscgyl-ChinaUnix博客](http://blog.chinaunix.net/uid-22002627-id-3475650.html)