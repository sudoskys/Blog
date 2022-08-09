---
title: Rust踩坑
date: 2021-8-22 06:25:37
tags: 
   - Rust
   - 指南
copyright_info: 原创编写|材料采集与引用已注明
copyright_author: LuoYing
cover: false
---

## Rust Cargo 指定国内镜像

> Blocking waiting for file lock on package cache
>

### 删除缓存

如果Cargo.lock被其他程序正在写入，一般关掉那个程序就行。

在cargo文件夹下面，.package_cache被加锁阻塞<br />对此就需要进入对应的cargo文件夹下面，删除.package_cache文件然后重新进入Vs Code.

### 指定镜像

[Rust Cargo 指定国内镜像_李磊的博客-CSDN博客_cargo 镜像](https://blog.csdn.net/setlilei/article/details/106204105?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-2.base&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-2.base)

如当前用户目录为`/root` 项目目录为`/root/rust-test` 则在 `/root/.cargo` 目录或 `/root/rust-test/.cargo` 目录 创建文件`config`

如当前用户目录为 /root 项目目录为 /root/rust-test 则在 /root/.cargo 目录 或 /root/rust-test/.cargo 目录 创建名称为 config 的文件

内容如下

```python
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
# 指定镜像
replace-with = 'sjtu'

##选一个
# 清华大学
[source.tuna]
registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

# 中国科学技术大学
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

# 上海交通大学
[source.sjtu]
registry = "https://mirrors.sjtug.sjtu.edu.cn/git/crates.io-index"

# rustcc社区
[source.rustcc]
registry = "https://code.aliyun.com/rustcc/crates.io-index.git"
```

执行`cargo build`成功后即使用成功