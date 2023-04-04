---
title: HEXO本地一键备份本地
date: 2021-10-30 14:15:29
tags: Hexo
cover: false
---

![[object Object]](https://socialify.git.ci/sudoskys/HEXObackup/image?description=1&font=Rokkitt&issues=1&language=1&name=1&pattern=Floating%20Cogs&stargazers=1&theme=Light)

https://github.com/sudoskys/HEXObackup

# HEXObackup

HEXO是以本地的形式存储的...

那么，为了避免，文件意外丢失的情况发生，备份数据就十分重要。

所以，我花了些时间去写了一个PYthon脚本并打包出来，可通用哦。

## 功能

批量备份文件夹（压缩到）D盘的BACKup文件夹内，分日期可回溯。


## 用法

### step 1

在你的用户文件夹内建立一个子文件夹，然后将打包设置后的程序的快捷方式放入其中


### step 2

将此文件夹添加至环境变量中的PATH，`win+r` 打开运行，输入快捷方式的名称（快捷方式的名称最好简化些，比如 backup-->bcm,set-->bcs）


## 如何配置

打开 set ，程序会创立并打开D盘的一个文档，里面填需要备份的文件夹，一行一个。

当然，你可以修改这个程序，实现你需要的效果。（比如与OneDrive同步文件搭配或者修改逻辑，将它直接上传至云存储）

--------------
![MEO](https://tva1.sinaimg.cn/large/9bd9b167gy1g2qklwroj6j21hc0u0kha.jpg "LOVE")

# 英文版本

AS we know
HEXO is stored locally...

So, in order to avoid accidental loss of files, it is very important to back up data.

So, I spent some time writing a PYthon script and packaged it, which is universal.

## Function

The batch backup folder (compressed to) the BACKup folder of D drive, the date can be traced back.


## Usage

### step 1

Create a subfolder in your user folder, and then put the shortcut of the packaged program into it


### step 2

Add this folder to the PATH in the environment variable, open and run `win+r`, and enter the name of the shortcut (the name of the shortcut is better to be simplified, such as backup-->bcm,set-->bcs)


## How to configure

Open set, the program will create and open a file in D drive, fill in the folder to be backed up, one per line.

Of course, you can modify this program to achieve the effect you need. (For example, synchronize files with OneDrive or modify the logic, and upload it directly to cloud storage)

