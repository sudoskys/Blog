---
title: Gitbook 配置不完全指北
date: 2021-5-12 07:01:27
tags: 
   - Web
   - 指南
copyright_info: 原创编写|材料采集与引用已注明
copyright_author: LuoYing
cover: false
---
## 配置环境

先在windows 下安装nodejs

官网：https://nodejs.org/en/

然后通过cmd调出DOS命令窗口测试下是否安装完成

输入命令：node

## 搭建gitbook平台

### 安装

 1、使用npm全局安装gitbook-cli

```undefined
npm install gitbook-cli -g 
```

 2、使用gitbook --version来查看gitbook的版本

```undefined
gitbook --version 
```

 3、新建一个文件夹，初始化gitbook，会自动生成两个文件。<br /> README.md —— 书籍的介绍写在这个文件里<br /> SUMMARY.md —— 书籍的目录结构在这里配置

```kotlin
gitbook init 
```

 4、接着，使用gitbook serve命令来启动gitbook本地服务器，预览书籍内容。

```undefined
gitbook serve 
```

 serve 命令也可以指定端口：

```undefined
gitbook serve --port 2333 
```

 5、打开localhost:4000，会出现如下页面

 <br />

 ![](//upload-images.jianshu.io/upload_images/15401334-7fab3bfb5c4a2296.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

<br /> 6、生成静态网页<br /> 执行 gitbook build 命令构建书籍，默认将生成的静态网站输出到 _book 目录。实际上，这一步也包含在 gitbook serve 里面，因为它们是 HTML，所以 gitbook 通过 Node.js 提供服务了。

```bash
gitbook build #生成静态网页 
```

 可以生成 PDF 格式的电子书：

```undefined
gitbook pdf ./ ./mybook.pdf 
```

 生成 epub 格式的电子书：

```undefined
gitbook epub ./ ./mybook.epub 
```

 生成 mobi 格式的电子书：

```undefined
gitbook mobi ./ ./mybook.mobi 
```

 如果生成不了，还需要安装工具[ebook-convert](https://links.jianshu.com/go?to=https%3A%2F%2Fcalibre-ebook.com)，安装好后，还需要执行以下命令

```bash
ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/local/bin 
```

 编辑 SUMMARY.md 文件，内容修改为：

```cpp
* [快速入门](README.md)

* 业务组件
  * [日历组件](component/rx-touch-calendar/README.md)
  * [展示瑞信头像组件](component/rx-getPhoto-list/README.md)
```

### 目录结构

```css
.
├── README.md
├── SUMMARY.md
├── book.json
├── component
│   ├── rx-getPhoto-list
│   └── rx-touch-calendar
└── style.css
```

 book.json，该文件用于存放配置信息。

 【title】书本的标题<br /> 【author】作者的相关信息<br /> 【description】本书的简单描述<br /> 【language】gitbook使用的语言<br /> 【root】指定存放 GitBook 文件（除了 book.json）的根目录<br /> 【structure】指定自述文件，摘要，词汇表等的路径

 gitbook支持许多插件，可以扩展gitbook的功能。

 【打赏功能：donate】

```json
{
    "plugins": ["donate"],
    "pluginsConfig": {
        "donate": {
          "wechat": "例：/images/qr.png",
          "alipay": "http://blog.willin.wang/static/images/qr.png",
          "title": "默认空",
          "button": "默认值：Donate",
          "alipayText": "默认值：支付宝捐赠",
          "wechatText": "默认值：微信捐赠"
        }
    }
}
```

 【广告功能：ad】

```json
{
  "plugins": ["ad"],
  "pluginsConfig": {
    "ad": {
      "contentTop": "<div>Ads at the top of the page</div>",
      "contentBottom": "%3Cdiv%3EAds%20at%20the%20bottom%20of%20the%20page%3C/div%3E"
    }
  }
}

// note: contentBottom is escape('<div>Ads at the bottom of the page</div>')
```

 【目录宽度可调节：splitter】

```json
{
    "plugins": ["splitter"]
}
```

 【github图标】

```json
{
    "plugins": [ "github" ],
    "pluginsConfig": {
        "github": {
            "url": "https://github.com/your/repo"
        }
    }
}
```

 【自定义页脚：tbfed-pagefooter】

```json
{
    "plugins": [ "tbfed-pagefooter" ],
    "pluginsConfig": {
        "tbfed-pagefooter": {
             "copyright":"&copy Taobao FED Team",
             "modify_label": "该文件修订时间：",
             "modify_format": "YYYY-MM-DD HH:mm:ss"
        }
    }
}
```

 【目录章节可折叠：expandable-chapters】

```bash
{
    {
        plugins: ["expandable-chapters"]
    }
    {
        "pluginsConfig": {
            "expandable-chapters":{}
        }
    }
}
```

 【畅言评论：changyan】

```json
{
    "plugins": [
        "changyan"
    ],
    "pluginsConfig": {
        "changyan": {
            "appid": "your changyan's appid",
            "conf": "the conf in the code generate by changyan"
        }
    }
}
```

 【返回顶部：back-to-top-button】

```json
{
    "plugins" : [ "back-to-top-button" ]
}
```

   上面支持列举了一些常用的插件，想要了解更多可以阅读官方文档，插件在book.json配置好后，需要安装。

```undefined
sudo gitbook install 
```

 book.json模板

```json
{
  "title": "UI",
  "description": "UI组件库",
  "author": "zhuyongbo",
  "language": "zh-hans",
  "links": {
    "sidebar": {
      "开放平台": "http://e.cnpc.com.cn/opensdk/"
    }
  },
  "styles":{
    "website":"style.css"
  },
  "plugins": [
    "-lunr",
    "-search",
    "-livereload",
    "-sharing",
    "expandable-chapters",
    "search-plus",
    "splitter",
    "github",
    "-sharing",
    "emphasize",
    "include-codeblock",
    "tbfed-pagefooter",
    "back-to-top-button",
    "anchor-navigation-ex"
  ],
  "pluginsConfig": {
    "github": {
      "url": "https://github.com/webzhuyongbo"
    },
    "sharing": {
      "douban": false,
      "facebook": false,
      "google": false,
      "hatenaBookmark": false,
      "instapaper": false,
      "line": false,
      "linkedin": false,
      "messenger": false,
      "pocket": false,
      "qq": false,
      "qzone": false,
      "stumbleupon": false,
      "twitter": false,
      "viber": false,
      "vk": false,
      "weibo": false,
      "whatsapp": false,
      "all": [
        "weibo","qq","qzone","google","douban"
      ]
    },
    "anchor-navigation-ex": {
      "associatedWithSummary":false,
      "showLevel":true,
      "multipleH1": true,
      "mode": "float",

      "pageTop": {
        "showLevelIcon": false,
        "level1Icon": "fa fa-hand-o-right",
        "level2Icon": "fa fa-hand-o-right",
        "level3Icon": "fa fa-hand-o-right"
      }
    },
    "tbfed-pagefooter": {
      "copyright": "©北京信息技术有限责任公司",
      "modify_label": "文档更新时间：",
      "modify_format": "YYYY-MM-DD HH:mm:ss"
    }
  }
}
```

  **去掉gitbook的版权信息** ：

 创建样式表文件“styles/website.css”，添加代码如下：

```css
.gitbook-link {
    display: none !important;
}
```

 编辑“book.json”文件，添加如下代码：

```json
{
    "styles": {
        "website": "styles/website.css"
    }
}
```

 好了，让我们看一下我们文档的效果图。

 <br />

 ![](//upload-images.jianshu.io/upload_images/15401334-605ff1cd49a90fa6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 报错处理

#### Installing速度很慢

**问题背景**

执行 gitbook -V 或 gitbook init 命令，均会显示： Installing GitBook 3.2.3 .......，之后便是漫长的等待，遥遥无期的那种，可以用 strace -ttp pid 跟踪发现其实进程还是在干活的，只是速度很慢。

**问题原因**

问题一：npm 默认使用安装使用的国外镜像，这个速度是比较慢的。<br />问题二：版本问题，3.2.3这个版本在本机可能不适用

**解决方案**

问题一：切换为使用国内速度较快的淘宝镜像。<br />问题二：卸载3.2.3 安装3.0.0

**具体操作**

```shell
# 问题一：
npm config set registry=http://registry.npm.taobao.org -g
# 问题二：
gitbook uninstall 3.2.3 #卸载
gitbook fetch 3.0.0 #安装
```

#### Gitbook -V或初始化报错：TypeError: cb.apply is not a function三种解决方法

 **开始动手**

```
[root@pes nodejs]# gitbook -V
CLI version: 2.3.2
Installing GitBook 3.2.3
/data/soft/nodejs/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
                 ^

TypeError: cb.apply is not a function
    at /data/soft/nodejs/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287:18
    at FSReqCallback.oncomplete (fs.js:169:5)
```

　　打开polyfills.js文件，路径报错中含有着，找到这个函数

```
function statFix (orig) {
  if (!orig) return orig
  // Older versions of Node erroneously returned signed integers for
  // uid + gid.
  return function (target, cb) {
    return orig.call(fs, target, function (er, stats) {
      if (!stats) return cb.apply(this, arguments)
      if (stats.uid < 0) stats.uid += 0x100000000
      if (stats.gid < 0) stats.gid += 0x100000000
      if (cb) cb.apply(this, arguments)
    })
  }
}
```

在第62-64行调用了这个函数

```lua
//fs.stat = statFix(fs.stat) 
//fs.fstat = statFix(fs.fstat) 
//fs.lstat = statFix(fs.lstat)
```

把这三行代码注释掉一般就解决报错了。

---

**如果没有解决**

往下看：

使用淘宝镜像
npm下载路径，检查是不是淘宝镜像：

```gitbook
npm config get registry
```

```gitbook
npm config set registry https://registry.npm.taobao.org
```

切换成淘宝镜像
再检查是不是淘宝镜像：

```gitbook
npm config get registry

```

```gitbook
安装
gitbook init
```

---

**如果仍然没有解决**

并且你的gitbook -V返回值没有GitBook version而是

```gitbook
CLI version: 2.3.2
Installing GitBook 3.2.3
```

说明gitbook并没有随着`npm install gitbook-cli -g`的运行而安装还需要一个gitbook本身的安装过程才能完成，遗憾的是，这个过程反复报错，无法顺利完成gitbook的安装。

原因在于，nodejs的版本不对，不支持这个gitbook.

让我们切换成nodejs的v10.21.0版本，

1.卸载Nodejs  [Window下完全卸载删除Nodejs](https://www.cnblogs.com/JourneyOfFlower/p/15007719.html)

2.下载地址：https://nodejs.org/dist/v10.21.0/node-v10.21.0-x64.msi

3.安装完成看看

```gitbook
C:\WINDOWS\system32>node -v
v10.21.0

C:\WINDOWS\system32>npm -v
6.14.4
```

#### 初始化出现错误

![](https://img2020.cnblogs.com/blog/1295751/202107/1295751-20210723144524777-1385412351.png)

 后面参考了 [gitbook从入门到放弃_简明AI工作室](https://xiaosongshine.blog.csdn.net/article/details/116235787)

发现是Nodejs **版本过高** ，需要**降低Nodejs**的版本到 **v12.22.1**

**方案一：**

卸载Nodejs  [Window下完全卸载删除Nodejs](https://www.cnblogs.com/JourneyOfFlower/p/15007719.html)

安装Nodejs **v12.22.1** [https://nodejs.org/dist/](https://nodejs.org/dist/)

![](https://img2020.cnblogs.com/blog/1295751/202107/1295751-20210723163513669-1049529915.png)

**方案二：**

[Nodejs降低或切换使用的版本](https://www.cnblogs.com/JourneyOfFlower/p/15049226.html)

再执行 gitbook init rycloud

#### 生成的静态文件无法跳转

解决方法：修改js文件，[参照](https://blog.csdn.net/weixin_42057852/article/details/81776917)

* 找到项目目录`gitbook`
* 找到目录下的`theme.js`文件
* 将`if(m)`改成`if(false) `
* 具体内容可以见附录2

## Npm实在太慢？镜像也慢？

Yarn是facebook发布的一款取代npm的包管理工具。

#### 特点

快速，Yarn 缓存了每个下载过的包，所以再次使用时无需重复下载。 同时利用并行下载以最大化资源利用率，因此安装速度更快。

安全，在执行代码之前，Yarn 会通过算法校验每个安装包的完整性。

可靠，使用详细、简洁的锁文件格式和明确的安装算法，Yarn 能够保证在不同系统上无差异的工作。

#### 安装

使用npm安装
`npm install -g yarn`
查看版本：`yarn --version`

---

安装node.js,下载yarn的安装程序:
提供一个.msi文件，在运行时将引导您在Windows上安装Yarn

https://yarnpkg.com/en/docs/install#windows-stable

---

Yarn 淘宝源安装，分别复制粘贴以下代码行到黑窗口运行即可

```gitbook

yarn config set registry https://registry.npm.taobao.org -g
yarn config set sass_binary_site http://cdn.npm.taobao.org/dist/node-sass -g

```

yarn的常用命令：
安装yarn`npm install -g yarn`

安装成功后，查看版本号：`yarn --version`

```gitbook
yarn install gitbook-cli
yarn add [gitbook-cli -g]
```

创建文件夹 yarn `md yarn`
进入yarn文件夹`cd yarn`
初始化项目`yarn init // 同npm init` 执行输入信息后，会生成package.json文件

参考：[使用“npm init”初始化项目 - 你是远方 - 博客园 (cnblogs.com)](https://www.cnblogs.com/WD-NewDemo/p/11141384.html)

yarn的配置项

```gitbook
yarn config list // 显示所有配置项
yarn config get <key> //显示某配置项
yarn config delete <key> //删除某配置项
yarn config set <key> <value> [-g|--global] //设置配置项
```

安装包

```gitbook
yarn install //安装package.json里所有包，并将包及它的所有依赖项保存进yarn.lock
yarn install --flat 
//安装一个包的单一版本
yarn install --force 
//强制重新下载所有包
yarn install --production 
//只安装dependencies里的包
yarn install --no-lockfile 
//不读取或生成yarn.lock
yarn install --pure-lockfile 
//不生成yarn.lock
```

添加包（会更新package.json和yarn.lock）

```gitbook
yarn add [package] // 在当前的项目中添加一个依赖包，会自动更新到package.json和yarn.lock文件中
yarn add [package]@[version] // 安装指定版本，这里指的是主要版本，如果需要精确到小版本，使用-E参数
yarn add [package]@[tag] // 安装某个tag（比如beta,next或者latest）
//不指定依赖类型默认安装到dependencies里，你也可以指定依赖类型：

yarn add --dev/-D // 加到 devDependencies
yarn add --peer/-P // 加到 peerDependencies
yarn add --optional/-O // 加到 optionalDependencies
//默认安装包的主要版本里的最新版本，下面两个命令可以指定版本：

yarn add --exact/-E // 安装包的精确版本。例如yarn add foo@1.2.3会接受1.9.1版，但是yarn add foo@1.2.3 --exact只会接受1.2.3版
yarn add --tilde/-T // 安装包的次要版本里的最新版。例如yarn add foo@1.2.3 --tilde会接受1.2.9，但不接受1.3.0

```

发布包`yarn publish`
移除一个包`yarn remove <packageName>：移除一个包，会自动更新package.json和yarn.lock`
更新一个依赖 `yarn upgrade 用于更新包到基于规范范围的最新版本`
运行脚本

`yarn run 用来执行在 package.json 中 scripts 属性下定义的脚本`
显示某个包的信息

`yarn info <packageName> 可以用来查看某个模块的最新版本信息`
缓存

`yarn cache yarn cache list # 列出已缓存的每个包 yarn cache dir # 返回 全局缓存位置 yarn cache clean # 清除缓存`
<br />[yarn的安装和使用_yw00yw的博客-CSDN博客_yarn安装](https://blog.csdn.net/yw00yw/article/details/81354533)

[yarn基本命令 - baoyadong - 博客园 (cnblogs.com)](https://www.cnblogs.com/xuzhudong/p/9342430.html)

可以入手yarn了

#### yarn报错？

##### 对于 Found incompatible module

https://www.jianshu.com/p/5cb4f48ed11b

推测是版本问题，设置yarn如下

`yarn config set ignore-engines true`

重新install项目包，发现还有问题。

删除node_modules包和yarn.lock文件，重新yarn install。

# yarn怎么在gitbook中应用？

**很简单，gitbokk命令前面加上 yarn即可**

**例如**

```gitbook
//初始化你的项目
yarn init
//初始化填什么？参考就明白里面的值是什么了https://www.cnblogs.com/WD-NewDemo/p/11141384.html

//安装包
yarn add gitbook-cli
//初始化
yarn gitbook -V
yarn gitbook init

//！！！这里是最终我执行成功的代码，我使用了yarn！！！
//本人亲自验证
//接下来你只需要编辑GitBook 项目结构即可
//写完了可以打包
yarn gitbook build//输出静态网页

yarn 
```

具体写什么，怎么写看这里。

[使用 Gitbook 打造你的电子书 - 简书 (jianshu.com)](https://www.jianshu.com/p/f38d8ff999cb)

## 参考

<br />参考链接：https://www.jianshu.com/p/f8cee64d2153

进阶指路[使用 Gitbook 打造你的电子书 - 简书 (jianshu.com)](https://www.jianshu.com/p/f38d8ff999cb)

大神指路[Gitbook 安裝 | Cowman`''`s Gitbook (cowmanchiang.me)](https://cowmanchiang.me/gitbook/gitbook/contents/install.html)

其他方案[Docute](https://docute.org/zh/)

[gitbook的安装与使用 - Journey`&&`Flower - 博客园 (cnblogs.com)](https://www.cnblogs.com/JourneyOfFlower/p/15012108.html)

[3.1 npm init 使用 · 通俗易懂的 npm 入门教程 (gitbooks.io)](https://dkvirus.gitbooks.io/-npm/content/di-sanzhang-npm-chuang-jian-xiang-mu/31-npm-init-shi-yong.html)

[解决安装gitbook时卡顿在 Installing GitBook 3.2.3 的问题 - Python全栈之巅 (pythonsky.cn)](https://www.pythonsky.cn/technical-talk/233.html)

## 附录

### 如何从Windows中删除Node.js：

1.从卸载程序卸载程序和功能。开始菜单右键。

2.重新启动（或者您可能会从任务管理器中杀死所有与节点相关的进程）。

3.寻找这些文件夹并删除它们（及其内容）（如果还有）。根据您安装的版本，UAC设置和CPU架构，这些可能或可能不存在：

C:\Program Files (x86)\Nodejs<br />C:\Program Files\Nodejs<br />C:\Users\{User}\AppData\Roaming\npm（或%appdata%\npm）<br />C:\Users\{User}\AppData\Roaming\npm-cache（或%appdata%\npm-cache）

4.检查您的%PATH%环境变量以确保没有引用Nodejs或npm存在。

5.如果仍然没有卸载，请where node在命令提示符下键入，您将看到它所在的位置 - 删除（也可能是父目录）。

6.重新启动，很好的措施。

参考地址：https://stackoverflow.com/questions/20711240/how-to-completely-remove-node-js-from-windows

### Gitbook新版本"gitbook build"命令导出的html不能跳转的解决办法

**可能原因**
新版本的gitbook不支持了这个功能

**具体原因**
由于点击事件被js代码禁用，所以点击没有反应，但是如果右键，在新窗口/新标签页打开的话是可以跳转的

**解决办法**
找到js代码，并修改

找到项目目录`gitbook`
找到目录下的`theme.js`文件
找到下面的代码
将`if(m)`改成`if(false)`

由于代码是压缩后的，会没有空格，搜索的时候可以直接搜索： `if(m)for(n.handler&&`

```gitbook
if (m)
    for (n.handler && (i = n,
    n = i.handler,
    o = i.selector),
    o && de.find.matchesSelector(Ye, o),
    n.guid || (n.guid = de.guid++),
    (u = m.events) || (u = m.events = {}),
    (a = m.handle) || (a = m.handle = function(t) {
        return "undefined" != typeof de && de.event.triggered !== t.type ? de.event.dispatch.apply(e, arguments) : void 0
    }
    ),
    t = (t || "").match(qe) || [""],
    l = t.length; l--; )
        s = Ze.exec(t[l]) || [],
        h = g = s[1],
        d = (s[2] || "").split(".").sort(),
        h && (f = de.event.special[h] || {},
        h = (o ? f.delegateType : f.bindType) || h,
        f = de.event.special[h] || {},
        c = de.extend({
            type: h,
            origType: g,
            data: r,
            handler: n,
            guid: n.guid,
            selector: o,
            needsContext: o && de.expr.match.needsContext.test(o),
            namespace: d.join(".")
        }, i),
        (p = u[h]) || (p = u[h] = [],
        p.delegateCount = 0,
        f.setup && f.setup.call(e, r, d, a) !== !1 || e.addEventListener && e.addEventListener(h, a)),
        f.add && (f.add.call(e, c),
        c.handler.guid || (c.handler.guid = n.guid)),
        o ? p.splice(p.delegateCount++, 0, c) : p.push(c),
        de.event.global[h] = !0)
    }
```

————————————————
原文链接：https://blog.csdn.net/weixin_42057852/article/details/81776917

### 不足

gitbook无法胜任大量文档的工作，基本百余就会卡的不行。

这点需要考虑