---
title: OKhttpandlua
date: 2021-10-29 23:45:55
tags:
---
# OKhttp在Androlua中的初级应用

HTTP是现代应用常用的一种交换数据和媒体的网络方式，高效地使用HTTP能让资源加载更快，节省带宽。

OkHttp是一个高效的HTTP客户端，它有以下默认特性：

* 支持HTTP/2，允许所有同一个主机地址的请求共享同一个socket连接
* 连接池减少请求延时
* 透明的GZIP压缩减少响应数据的大小
* 缓存响应内容，避免一些完全重复的请求

 当网络出现问题的时候OkHttp依然坚守自己的职责，它会自动恢复一般的连接问题，如果你的服务有多个IP地址，当第一个IP请求失败时，OkHttp会交替尝试你配置的其他IP，OkHttp使用现代TLS技术(SNI, ALPN)初始化新的连接，当握手失败时会回退到TLS 1.0。

>  OkHttp 支持 Android 2.3 及以上版本Android平台， 对于 Java, JDK 1.7及以上.

官方仓库地址

[ https://github.com/square/okhttp](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fsquare%2Fokhttp)

DEX下载

[Okhttp · 洛樱/STUDYuseto - 码云 - 开源中国 (gitee.com)](https://gitee.com/luo-ying2020/studyuseto/tree/master/Okhttp)

## 请求示例

### 异步GET请求

导入OKhttp库

-new OkHttpClient;<br />-构造Request对象；<br />-通过前两步中的对象构建Call对象；<br />-通过Call#enqueue(Callback)方法来提交异步请求；

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"

import "okhttp3.*"
--请求辅助类
request = Request.Builder()
.url("http://www.baidu.com/")
.build();

--异步请求
client = OkHttpClient()
callz = client.newCall(request)
callz.enqueue(Callback{
  onFailure=function(call,e)
   --print("请求失败")
  end,

  onResponse=function(call,response)
   --print(response.body().string())
  end
})
```

异步发起的请求会被加入到 `Dispatcher` 中的 `runningAsyncCalls`双端队列中，该请求是否可以立即发起受 maxRequests 和 maxRequestsPerHost 两个条件的限制。如果符合条件，那么就会从 readyAsyncCalls 取出并存到 runningAsyncCalls 中，然后交由 OkHttp 内部的线程池来执行。

<br /><br />作者：业志陈<br />链接：https://www.jianshu.com/p/1b3d39c79e7e<br />来源：简书<br />著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 同步GET请求

前面几个步骤和异步方式一样，只是最后一部是通过 `Call#execute()` 来提交请求，注意这种方式会阻塞调用线程，所以在Android中应放在子线程中执行，否则有可能引起ANR异常，`Android3.0` 以后已经不允许在主线程访问网络，网络请求过程就会直接在调用者所在线程上完成，不受 Dispatcher 的控制

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"

import "okhttp3.*"
--请求辅助类
request = Request.Builder()
.url("http://www.baidu.com/")
.build();

--异步请求
client = OkHttpClient()
callz = client.newCall(request)
callz.execute(Callback{
  onFailure=function(call,e)
   --print("请求失败")
  end,

  onResponse=function(call,response)
   --print(response.body().string())
  end
})

```

### POST方式提交String

与前面的区别就是在构造Request对象时，需要多构造一个RequestBody对象，用它来携带我们要提交的数据。在构造 `RequestBody` 需要指定`MediaType`，用于描述请求/响应 `body` 的内容类型，关于 `MediaType` 的更多信息可以查看 [RFC 2045](https://links.jianshu.com/go?to=https%3A%2F%2Ftools.ietf.org%2Fhtml%2Frfc2045)

```lua

import 'com.kn.okhttp.*'
import 'okhttp3.*'

--指定MediaType
mediaType = MediaType.parse('text/x-markdown; charset=utf-8')


--构造Request对象
requestBody = '114514'--提交字符串
request = Request.Builder()
.url('https://api.github.com/markdown/raw')--请求URL
.post(RequestBody.create(mediaType,requestBody))
.build()

--提交请求
okHttpClient = OkHttpClient()
okHttpClient.newCall(request).enqueue(Callback{
onFailure = function(call,e)--失败请求
print(e)
end,
onResponse = function(call,response)--请求成功
headers = response.headers()
print(headers)--展示返回头部
print(response.body().string())--展示返回内容
end
})
```

### POST方式提交文件

```lua
import 'com.kn.okhttp.*'
import 'okhttp3.*'
--指定MediaType
mediaType = MediaType.parse("text/x-markdown; charset=utf-8");
okHttpClient = OkHttpClient();
file = File(activity.getLuaDir()..'/init.lua');--提交的文件

--构建
request = Request.Builder()
.url("https://api.github.com/markdown/raw")--URL
.post(RequestBody.create(mediaType, file))
.build();

okHttpClient.newCall(request).enqueue(Callback {
onFailure = function(call, e) --失败回调
print(e)
end,

onResponse = function(call,response) --成功
print(response.headers())
print(response.body().string())
end
});
```

### POST方式提交表单

```lua
--记得导入基础包
import 'com.kn.okhttp.*'
import 'okhttp3.*'
okHttpClient = OkHttpClient();
requestBody = FormBody.Builder()
.add("keyword", "114514")--提交的表单
.build();

request = Request.Builder()
.url("https://gitee.com/luo-ying2020/studyuseto")--URL
.post(requestBody)
.build();

okHttpClient.newCall(request).enqueue(Callback {
onFailure = function(call, e) --失败
print(e);
end,
onResponse = function(call, response) --成功
print(response.headers())
print(response.body().string())
end
});
```

### 图床实战

```lua
--sm.ms上传，除注释外都是固定模板，复制粘贴即可
import 'com.kn.okhttp.*'
import 'okhttp3.*'
--调用选择图库
import "android.content.Intent"
local intent= Intent(Intent.ACTION_PICK)
intent.setType("image/*")
this.startActivityForResult(intent, 1)
-------

--回调
function onActivityResult(requestCode,resultCode,intent)
if intent then
local cursor =this.getContentResolver ().query(intent.getData(), nil, nil, nil, nil)
cursor.moveToFirst()
import "android.provider.MediaStore"
local idx = cursor.getColumnIndex(MediaStore.Images.ImageColumns.DATA)
fileSrc = cursor.getString(idx)--返回路径
filename = fileSrc:match('.+%/(.+)')--截取名字

requestBody = MultipartBody.Builder()
.setType(MultipartBody.FORM)
.addFormDataPart('smfile',filename,RequestBody.create(MediaType.parse('multipart/form-data'),File(fileSrc)))--对应表单
.build()
request = Request.Builder()
.header('User-Agent','Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36')
.url('https://sm.ms/api/upload')--URL
.post(requestBody)
.build()
okHttpClient = OkHttpClient()
okHttpClient.newCall(request).enqueue(Callback{
onFailure = function(call,e)--失败请求
print(e)
end,

onResponse = function(call,response)--请求成功
headers = response.headers()
print(headers)--展示返回头部
print(response.body().string())--展示返回内容
end
})
end
end--nirenr
```

## 主流程源码分析

指路

[Okhttp3源码分析 - 简书 (jianshu.com)](https://www.jianshu.com/p/b0353ed71151)

 [Android知名三方库OKHttp(二) - 手写简易版 - 简书 (jianshu.com)](https://www.jianshu.com/p/65226c0f2a3e)


## OkHttp3架构分析

指路

[OkHttp3架构分析 - 简书 (jianshu.com)](https://www.jianshu.com/p/9deec36f2759)

[三方库源码笔记（11）-OkHttp 源码详解 - 简书 (jianshu.com)](https://www.jianshu.com/p/1b3d39c79e7e)


## 编码处理-应对GB2312的办法

```lua
--在回调方法onResponse方法中，获取数据的bytes，然后将其转为gb2312
print(String(response.body().bytes(),"gb2312"))
```

## OKhttp优缺点

#### 优点

支持SPDY, 可以合并多个到同一个主机的请求。

使用连接池技术减少请求的延迟(如果SPDY是可用的话) 。
使用GZIP压缩减少传输的数据量，避免重复的网络请求、拦截器等等。

#### 缺点

第一缺点是回调需要切到主线程，主线程要自己去写，还有就是传入调用比较复杂。

和线程一挂钩就很难用

## 参考 摘录

<br />[Okhttp3基本使用 - 简书 (jianshu.com)](https://www.jianshu.com/p/da4a806e599b)

代码手册APP--AndroluaBox

nirenr写的代码


