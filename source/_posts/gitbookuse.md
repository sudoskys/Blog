---
title: Gitbook 配置不完全指北
date: 2021-5-12 07:01:27
tags:
  - Web
  - 指南
---

>此文章已经过时，请您使用最新的[Gitbook 官方文档](https://docs.gitbook.com/)。

## 配置环境

先在 windows 下安装 nodejs

官网：<https://nodejs.org/en/>

然后通过 cmd 调出 DOS 命令窗口测试下是否安装完成

输入命令：node

## 搭建 gitbook 平台

### 安装

1、使用 npm 全局安装 gitbook-cli

```shell
npm install gitbook-cli -g
```

2、使用 gitbook --version 来查看 gitbook 的版本

```shell
gitbook --version
```

3、新建一个文件夹，初始化 gitbook，会自动生成两个文件。<br /> README.md —— 书籍的介绍写在这个文件里<br /> SUMMARY.md —— 书籍的目录结构在这里配置

```shell
gitbook init
```

4、接着，使用 gitbook serve 命令来启动 gitbook 本地服务器，预览书籍内容。

```shell
gitbook serve
```

serve 命令也可以指定端口：

```shell
gitbook serve --port 2333
```

5、打开 localhost:4000，会出现如下页面

![exp](//upload-images.jianshu.io/upload_images/15401334-7fab3bfb5c4a2296.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

6、生成静态网页，执行 gitbook build 命令构建书籍，默认将生成的静态网站输出到 \_book 目录。实际上，这一步也包含在 gitbook serve 里面，因为它们是 HTML，所以 gitbook 通过 Node.js 提供服务了。

```shell
gitbook build #生成静态网页
```

可以生成 PDF 格式的电子书：

```shell
gitbook pdf ./ ./mybook.pdf
```

生成 epub 格式的电子书：

```shell
gitbook epub ./ ./mybook.epub
```

生成 mobi 格式的电子书：

```shell
gitbook mobi ./ ./mybook.mobi
```

如果生成不了，还需要安装工具[ebook-convert](https://links.jianshu.com/go?to=https%3A%2F%2Fcalibre-ebook.com)，安装好后，还需要执行以下命令

```shell
ln -s /Applications/calibre.app/Contents/MacOS/ebook-convert /usr/local/bin
```

编辑 SUMMARY.md 文件，内容修改为：

```markdown
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

- **title**: 书本的标题
- **author**: 作者的相关信息
- **description**: 本书的简单描述
- **language**: gitbook 使用的语言
- **root**: 指定存放 GitBook 文件（除了 book.json）的根目录
- **structure**: 指定自述文件，摘要，词汇表等的路径

gitbook 支持许多插件，可以扩展 gitbook 的功能。

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

【github 图标】

```json
{
  "plugins": ["github"],
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
  "plugins": ["tbfed-pagefooter"],
  "pluginsConfig": {
    "tbfed-pagefooter": {
      "copyright": "&copy Taobao FED Team",
      "modify_label": "该文件修订时间：",
      "modify_format": "YYYY-MM-DD HH:mm:ss"
    }
  }
}
```

【目录章节可折叠：expandable-chapters】

```json
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
  "plugins": ["changyan"],
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
  "plugins": ["back-to-top-button"]
}
```

上面支持列举了一些常用的插件，想要了解更多可以阅读官方文档，插件在 book.json 配置好后，需要安装。

```shell
sudo gitbook install
```

book.json 模板

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
  "styles": {
    "website": "style.css"
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
      "all": ["weibo", "qq", "qzone", "google", "douban"]
    },
    "anchor-navigation-ex": {
      "associatedWithSummary": false,
      "showLevel": true,
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

**去掉 gitbook 的版权信息** ：

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

![exp2](//upload-images.jianshu.io/upload_images/15401334-605ff1cd49a90fa6.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

## 报错处理

### Installing 速度很慢

- 问题背景

执行 gitbook -V 或 gitbook init 命令，均会显示： Installing GitBook 3.2.3 .......，之后便是漫长的等待，遥遥无期的那种，可以用 strace -ttp pid 跟踪发现其实进程还是在干活的，只是速度很慢。

- 问题原因

问题一：npm 默认使用安装使用的国外镜像，这个速度是比较慢的。
问题二：版本问题，3.2.3 这个版本在本机可能不适用

- 解决方案

问题一：切换为使用国内速度较快的淘宝镜像。
问题二：卸载 3.2.3 安装 3.0.0

- 具体操作

```shell
# 问题一：
npm config set registry=http://registry.npm.taobao.org -g
# 问题二：
gitbook uninstall 3.2.3 #卸载
gitbook fetch 3.0.0 #安装
```

### Gitbook -V 或初始化报错：TypeError: cb.apply is not a function 三种解决方法

#### 开始动手

```shell
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

打开 polyfills.js 文件，路径报错中含有着，找到这个函数

```javascript
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

在第 62-64 行调用了这个函数

```javascript
//fs.stat = statFix(fs.stat)
//fs.fstat = statFix(fs.fstat)
//fs.lstat = statFix(fs.lstat)
```

把这三行代码注释掉一般就解决报错了。

#### 如果没有解决

往下看：

使用淘宝镜像
npm 下载路径，检查是不是淘宝镜像：

```shell
npm config get registry
```

```shell
npm config set registry https://registry.npm.taobao.org
```

切换成淘宝镜像
再检查是不是淘宝镜像：

```shell
npm config get registry

```

```shell
安装
gitbook init
```

如果仍然没有解决，并且你的 gitbook -V 返回值没有 GitBook version 而是

```shell
CLI version: 2.3.2
Installing GitBook 3.2.3
```

说明 gitbook 并没有随着`npm install gitbook-cli -g`的运行而安装还需要一个 gitbook 本身的安装过程才能完成，遗憾的是，这个过程反复报错，无法顺利完成 gitbook 的安装。

原因在于，nodejs 的版本不对，不支持这个 gitbook.

#### 初始化出现错误

![ex3](https://img2020.cnblogs.com/blog/1295751/202107/1295751-20210723144524777-1385412351.png)

后面参考了 [gitbook 从入门到放弃\_简明 AI 工作室](https://xiaosongshine.blog.csdn.net/article/details/116235787)

发现是 Nodejs **版本过高** ，需要**降低 Nodejs**的版本到 **v12.22.1**

**方案一：**

卸载 Nodejs [Window 下完全卸载删除 Nodejs](https://www.cnblogs.com/JourneyOfFlower/p/15007719.html)

安装 Nodejs **v12.22.1** [https://nodejs.org/dist/](https://nodejs.org/dist/)

![ex2](https://img2020.cnblogs.com/blog/1295751/202107/1295751-20210723163513669-1049529915.png)

**方案二：**

[Nodejs 降低或切换使用的版本](https://www.cnblogs.com/JourneyOfFlower/p/15049226.html)

再执行 gitbook init rycloud

#### 生成的静态文件无法跳转

解决方法：修改 js 文件，[参照](https://blog.csdn.net/weixin_42057852/article/details/81776917)

- 找到项目目录`gitbook`
- 找到目录下的`theme.js`文件
- 将`if(m)`改成`if(false)`
- 具体内容可以见附录 2

## 附录

### 如何从 Windows 中删除 Node.js

1.从卸载程序卸载程序和功能。开始菜单右键。

2.重新启动（或者您可能会从任务管理器中杀死所有与节点相关的进程）。

3.寻找这些文件夹并删除它们（及其内容）（如果还有）。根据您安装的版本，UAC 设置和 CPU 架构，这些可能或可能不存在：

```shell
C:\Program Files (x86)\Nodejs
C:\Program Files\Nodejs
C:\Users\{User}\AppData\Roaming\npm（或%appdata%\npm）
C:\Users\{User}\AppData\Roaming\npm-cache（或%appdata%\npm-cache）
```

4.检查您的%PATH%环境变量以确保没有引用 Nodejs 或 npm 存在。

5.如果仍然没有卸载，请 where node 在命令提示符下键入，您将看到它所在的位置 - 删除（也可能是父目录）。

6.重新启动

参考地址：<https://stackoverflow.com/questions/20711240/how-to-completely-remove-node-js-from-windows>

### Gitbook 新版本"gitbook build"命令导出的 html 不能跳转的解决办法

**可能原因**
新版本的 gitbook 不支持了这个功能

**具体原因**
由于点击事件被 js 代码禁用，所以点击没有反应，但是如果右键，在新窗口/新标签页打开的话是可以跳转的

**解决办法**
找到 js 代码，并修改

找到项目目录`gitbook`
找到目录下的`theme.js`文件
找到下面的代码
将`if(m)`改成`if(false)`

由于代码是压缩后的，会没有空格，搜索的时候可以直接搜索： `if(m)for(n.handler&&`

```javascript
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

原文链接：<https://blog.csdn.net/weixin_42057852/article/details/81776917>

### 不足

gitbook 无法胜任大量文档的工作，基本百余就会卡的不行。

这点需要考虑
