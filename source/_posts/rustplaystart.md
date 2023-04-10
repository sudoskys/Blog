---
title: Rust入门
date: 2021-8-22 06:22:37
tags:
   - Rust
   - 配置
cover: false
---

## 安装准备

### 准备环境

##### 路径准备

请选择一个空余空间较大的盘(因为后期包的累积可能会占用巨量的存储空间)，建立以下路径：

```markdown
E:\Environment
E:\Environment\MINGW
E:\Environment\RUST
E:\Environment\RUST\cargo
E:\Environment\RUST\rustup
```

我选择E盘作为我的开发资源盘~这里如果你选择其他盘，请务必注意之后操作也要更改。

##### 变量准备

右击我的电脑，选择属性-->高级系统设置-->环境变量

点击新建，填入以下条目

**配置自定义路径**

变量名：`CARGO_HOME`

内容：（刚才创建的对应文件路径,如: ）`E:\Environment\RUST\CARGO`

变量名：`RUSTUP_HOME`

内容：（刚才创建的对应文件路径, 如: ）`E:\Environment\RUST\RUSTUP`

**配置自定义国内中科院源:**

 变量名：`RUSTUP_DIST_SERVER`

内容：`https://mirrors.ustc.edu.cn/rust-static`

 变量名：`RUSTUP_UPDATE_ROOT`

内容：`https://mirrors.ustc.edu.cn/rust-static/rustup`

### 下载安装必备文件

2021/8/25打包到GITHUB了

[国内可访问](https://raw.githubusercontent.com/sudoskys2024/Sakura-animation/master/RUST.zip?raw=true)

#### 下载RUST

安装包地址: https://www.rust-lang.org/tools/install

#### 下载Mingw64

下载地址: http://www.mingw-w64.org/doku.php ,之后解压到相关路径。

PS:这里只需要解压到E:\Environment\mingw64里面，里面没有可执行文件哦。

> Rustup 在 Windows 环境下会优先使用 MSVC 作 linker，也就是 target 为 (arch)-pc-windows-msvc，然而其所需 Visual C++ Build Tools体积太大了，而且装在C盘。因此执行 rustup-init.exe 会看到红色提示（已经有 MSVC 编译器的不会INSTALL红色提示）
>
> 它默认使用 MSVC，所以我们要使用MINGW64
>

[MingGW64 版本区别于各版本说明 - PCYO - 你是谁的，谁是谁，谁为谁憔悴](https://www.pcyo.cn/linux/20181212/216.html)

#### 配置Mingw64

解压到磁盘某个位置，比如你的安装目录，或者直接解压到 C 盘根目录也可以，然后把 Bin 目录（请把路径copy下来）添加到环境变量 Path（这个条目的里面），记得跟其它目录用 `;` 隔开，保存。

## 安装

运行rustup-init.exe

输入2回车选择："自定义安装"

第二步: 输入: x86_64-pc-windows-gnu

后面的是 stable （使用的调试器是GDB，故需要修改为`x86_64-pc-windows-gnu`。`default toolchain`请选择`stable`，即稳定版。`nightly`为前瞻版（更新频率快），'beta’为测试版，实际使用时我们仍需使用到`nightly`版，后文会提到。）和「确认修改环境变量」就行（或许还需要输入default）。

接下来的就是等待。安装完成后，可以分别执行以下命令检查是否成功安装。

```bat
rustc --version 
cargo --version
```

最后一步确认: 经过之前的定义, 下载可以默认安装了.

## RUST新人指路使用

### Cargo

cargo既是一个类似于npm、pip的包管理软件，又是一个像maven一样的项目框架。输入> cargo help了解一下....

```markdown
build       编译当前包
check       检查当前包并寻出错误，但不进行编译
clean       删除编译结果（即target文件夹）
doc         构建当前包以及依赖项得文档
new         新建一个crate
init        以当前文件夹初始化一个crate
run         编译并执行src/main.rs
test        执行测试项
bench       执行基准测试项
update      更新所需的依赖项并预编译
search      搜索crates
publish     打包发布
install     安装cargo相关可执行文件，默认路径为 $HOME/.cargo/bin
uninstall   卸载相关可执行文件
```

[cargo简介 - Rust 中文教程 - 极客学院Wiki (jikexueyuan.com)](https://wiki.jikexueyuan.com/project/rust-primer/cargo-projects-manager/cargo-projects-manager.html)

### 集成VS Code

在此之前，先设置一遍可能用到的环境变量。<br />

`RUST` : 某toolchain的目录，如`%USER%\.rust\toolchains\stable-x86_64-pc-windows-gnu`

`RUST_SRC_PATH` : 改版本rust的源码目录，如`%RUST%\lib\rustlib\src\rust\src`，若你的rustlib中没有src。执行>` rustup component add rust-src`

`RUSTBINPATH` : `%CARGO_PATH%\bin`<br />

#### 下载安装racer（用于Rust代码自动补全）

`cargo install racer`

如果不成功，先将rustup更新成nightly版本，再进行下载：

`rustup install nightly`

`cargo +nightly install racer`

#### 打开VS Code

搜索插件`rust`(rls),就是下载量最高的那个，安装即可

**为了能调试软件**，再安装插件`CodeLLDB`，当然，也可以选择使用GDB

`crates`是辅助开发者在使用Cargo.toml时管理依赖的插件，推荐下载

#### 编译调试

新建一个文件夹比如testProj，子目录结构如下（main.rs和Cargo.toml为空即可）

```markdown
testProj
|-  src                     // 放置源文件的目录
    |- main.rs              // 源文件
|-  Cargo.toml              // Cargo的配置文件
```

然后用VS Code打开testProj文件夹

选择mian.rs，输入如下内容(//号为注释符)，保存：

```rust
fn main() {
    println!("Hello World!"); //测试输出
}
```

选择Cargo.toml，输入如下内容(#号为注释符)，保存：

```rust
[package]
name = "TargetName" #项目名
version = "0.0.1" #版本号
authors = ["YourName <YourEmail@example.com>"] #作者信息
```

通过CMD切换到项目根目录，通过`cargo build`编译程序得到可执行文件：

设置VS Code 的launch.json，输入如下内容，保存：

```rust
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "Debug",
            "program": "${workspaceFolder}/target/debug/testProj", //改成可执行文件实际名称！
            "args": [],
            "cwd": "${workspaceFolder}"
        }
    ]
}
```

这东西在vs左边的运行里面。

7、打上断点，开始调试吧：

## 参考/摘录

[配置 Rust 开发环境 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/29975631)

[windows下快速安装rust(GNU)_RSDTE-CSDN博客](https://blog.csdn.net/weixin_40411915/article/details/100109793)

[MinGW-W64下载与安装 - linying1991 - 博客园 (cnblogs.com)](https://www.cnblogs.com/linying1991/p/12164039.html)

[Rust Windows环境搭建_6日Simmp的博客-CSDN博客_rust windows](https://blog.csdn.net/weixin_43882409/article/details/87616268)

[基于VS Code 完成Rust开发环境配置及调试（Windows）_cyberobot的博客-CSDN博客](https://blog.csdn.net/qq_23918781/article/details/103292798?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.baidujsUnder6&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-3.baidujsUnder6)

### 附录

打开软件后会显示如下信息(translated)：

```html
Welcome to Rust!
欢迎来玩Rust!

This will download and install the official compiler for the Rust programming
language, and its package manager, Cargo.
本向导将会下载并安装Rust官方编译器与包管理工具Cargo。

It will add the cargo, rustc, rustup and other commands to Cargo's bin
directory, located at:
接下来会将cargo、rustc、rustup等二进制文件下载到Cargo的bin文件夹下：

  ~\.cargo\bin

This path will then be added to your PATH environment variable by modifying the
HKEY_CURRENT_USER/Environment/PATH registry key.
该路径将会添加至当前用户的PATH环境变量中。

You can uninstall at any time with rustup self uninstall and these changes will
be reverted.
你可以自由卸载，并恢复至初。

Current installation options:
当前安装参数：

   default host triple: x86_64-pc-windows-msvc
     default toolchain: stable
  modify PATH variable: yes

1. Proceed with installation (default)     确认并安装（默认）
2. Customize installation                  自定义安装
3. Cancel installation                     取消安装
```

该软件默认检查环境变量CARGO_HOME、RUSTUP_HOME，分别为.cargo、.rustup设置目录，按需修改（需要注意的是后期包的累积可能会占用巨量的存储空间）。
<br />

### 一些链接

Rust 语言官网 [Rust](https://www.rust-lang.org/)

一些中文资料<br />*[Rust 程序设计语言（第二版）](https://github.com/KaiserY/trpl-zh-cn)<br />*[RustPrimer](https://github.com/rustcc/RustPrimer) 给初学者的 Rust 中文教程<br />*[Rust by Example 中文版](https://github.com/rust-lang-cn/rust-by-example-cn)<br />*[Rust 宏小册](https://github.com/DaseinPhaos/tlborm-chinese)

中文社区 [RustChina](https://rust-lang-cn.org/)

常用的英文文档<br />*[The Rust Standard Library](https://doc.rust-lang.org/std/) Rust 标准库文档<br />*[crates.io](https://crates.io/) Rust Package Registry<br />*[docs.rs](https://docs.rs/) 第三方库的文档 (爬虫抓取 crates.io 生成)<br />*[doc.crates.io](http://doc.crates.io/index.html) crates.io 和 cargo 的文档<br />[rustup](https://github.com/rust-lang-nursery/rustup.rs) rustup 的文档

可能用到的<br />*[Book](https://doc.rust-lang.org/book/) Book 的英文原版<br />*[CookBook](https://github.com/rust-lang-nursery/rust-cookbook)<br />*[awesome-rust](https://github.com/rust-unofficial/awesome-rust)<br />*[rust-ffi-guide](https://github.com/Michael-F-Bryan/rust-ffi-guide)<br />*[rust-ffi-omnibus](https://github.com/shepmaster/rust-ffi-omnibus)<br />*[nomicon](https://github.com/rust-lang-nursery/nomicon)<br />*[api-guidelines](https://github.com/rust-lang-nursery/api-guidelines)<br />*[rust-internals-docs](https://github.com/Manishearth/rust-internals-docs)
