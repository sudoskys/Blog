---
title: Node环境与NPM安装配置
date: 2021-10-30 08:28:10
tags: Node
copyright_info: 原创编写|材料采集与引用已注明
copyright_author: LuoYing
headimg: https://tva4.sinaimg.cn/large/9bd9b167ly1g2qkkni7fwj21hc0u0e5m.jpg
---


## NODEjs Download

### 1 step

Click here [下载 | Node.js (nodejs.org)](https://nodejs.org/zh-cn/download/) to download a .msi flie for next step.

Please select the correct version according to your operating system. For example, if you are using a 64-bit operating system, please download the 64-bit version, if you are using a 32-bit version, please download the 32-bit version

Node.js默认安装目录为 "C:\Program Files\nodejs\" , 个人建议修改为其他磁盘避免C盘文件过多。

### 2 step

check the PATH by click run-->cmd-->type 'path' ,then, 

```shell
PATH=C:\oraclexe\app\oracle\product\10.2.0\server\bin;C:\Windows\system32;
C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;
c:\python32\python;C:\MinGW\bin;C:\Program Files\GTK2-Runtime\lib;
C:\Program Files\MySQL\MySQL Server 5.5\bin;C:\Program Files\nodejs\;
C:\Users\rg\AppData\Roaming\npm
```

If this show on your computer screen,congratulations!

### 3 step

check the version of node.js:  use `win+r ` to open cmd ,and type `node --version` （`Npm --version`） ,then enter ,show like `v0.11.11` .




## Npm

When you install the nodejs,the npm was install too.

### 1 step

To ensure normal access in China, we need to configure the mirror source.

### 临时使用

```
npm --registry https://registry.npm.taobao.org install express
```

### 永久使用

```shell
npm config set registry https://registry.npm.taobao.org
```

### 恢复使用

```
npm config set registry https://registry.npmjs.org
```

you can look your setting 

```shell
npm config ls
```

### 2 step

NPM更新

```shell
npm install -g npm
```


## NVM

Different applications require different versions of Node.js, switching and installing a new version of Node.js is annoying, and there will be inexplicable problems. nvm is to solve the problems of Node.js installation and version switching.


## USEFUL INFORMATION THERE

npm包全局目录：C:/Users/[username]/AppData/Roaming/npm/node_modules

很多时候，NODE要结合GIT使用才好玩。

 [Git (git-scm.com)](https://git-scm.com/)
 

## Reference

[如何正确使用淘宝npm镜像 - SegmentFault 思否](https://segmentfault.com/a/1190000027083723)

[有关 「 Nodejs 」 的内容 | 收集优质资源 (learn-anything.cn)](https://learn-anything.cn/tag/nodejs)

[(2 封私信 / 73 条消息) npm - 知乎 (zhihu.com)](https://www.zhihu.com/topic/19625829/hot)

[npm，yarn如何查看源和换源 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/35856841)

[整理总结：npm常用命令与操作篇 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/122224879)
