---
title: Node.js环境与NPM安装配置指南
date: 2021-10-30 08:28:10
tags: Node.js
---

## Node.js安装

### 第1步：下载

1. 访问[Node.js官方下载页面](https://nodejs.org/zh-cn/download/)下载适合您系统的.msi安装文件。
2. 请根据您的操作系统选择正确的版本:
   - 64位系统选择64位版本
   - 32位系统选择32位版本

> 提示：Node.js默认安装目录为"C:\Program Files\nodejs\"。建议修改为其他磁盘,以避免C盘文件过多。

### 第2步：检查环境变量

1. 按`Win+R`打开运行窗口
2. 输入`cmd`并回车打开命令提示符
3. 输入`path`命令查看环境变量,应包含类似以下内容:

```
PATH=C:\Windows\system32;C:\Windows;C:\Program Files\nodejs\;C:\Users\用户名\AppData\Roaming\npm
```

如果看到类似输出,说明安装成功!

### 第3步：验证版本

在命令提示符中输入以下命令检查Node.js和npm版本:

```shell
node --version
npm --version
```

如显示类似`v14.17.6`的版本号,则安装完成。

## NPM配置

### 设置镜像源

为保证在中国地区的正常访问,建议配置npm镜像源:

```shell
# 永久使用淘宝镜像
npm config set registry https://registry.npm.taobao.org

# 查看当前配置
npm config ls
```

### NPM更新

定期更新npm到最新版本:

```shell
npm install -g npm
```

## NVM工具(可选)

NVM可以方便地管理多个Node.js版本。不同项目可能需要不同版本的Node.js,NVM可以解决版本切换的问题。

## 实用信息

- npm全局包目录: `C:/Users/[用户名]/AppData/Roaming/npm/node_modules`
- 推荐同时安装Git版本控制工具: [Git官网下载](https://git-scm.com/)

## 参考资料

- [如何正确使用淘宝npm镜像](https://segmentfault.com/a/1190000027083723)
- [Node.js学习资源汇总](https://learn-anything.cn/tag/nodejs)
- [npm常用命令与操作总结](https://zhuanlan.zhihu.com/p/122224879)
