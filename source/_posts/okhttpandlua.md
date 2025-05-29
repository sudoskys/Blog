---
title: OKhttp在Androlua中的应用探索
date: 2021-10-29 23:45:55
tags: 
   - Android
   - Okhttp
---

## OKhttp在Androlua中的应用

HTTP是现代应用程序常用的数据交换网络协议。高效使用HTTP可以加快资源加载速度,节省带宽。

OkHttp是一个高效的HTTP客户端,具有以下特点:

* 支持HTTP/2,允许同一主机的所有请求共享一个socket连接
* 使用连接池减少请求延迟
* 透明GZIP压缩减小响应数据大小
* 响应缓存避免重复请求

OkHttp在网络问题时仍能保持稳定。它会自动恢复常见的连接问题,并在第一个IP失败时尝试备用IP。OkHttp使用最新的TLS技术(SNI, ALPN)建立新连接,握手失败时会回退到TLS 1.0。

> OkHttp支持Android 2.3及以上版本,Java要求JDK 1.7+。

官方仓库:
[https://github.com/square/okhttp](https://github.com/square/okhttp)

## 使用示例

### 异步GET请求

```lua
require "import"
import "android.app.*"
import "android.os.*"
import "android.widget.*"
import "android.view.*"
import "okhttp3.*"

-- 构建请求
request = Request.Builder()
  .url("http://www.baidu.com/")
  .build()

-- 异步请求
client = OkHttpClient()
call = client.newCall(request)
call.enqueue(Callback{
  onFailure = function(call, e)
    print("请求失败")
  end,
  
  onResponse = function(call, response)
    print(response.body().string())
  end
})
```

异步请求会被加入到Dispatcher的runningAsyncCalls队列。请求是否立即发起受maxRequests和maxRequestsPerHost限制。符合条件时,会从readyAsyncCalls移到runningAsyncCalls,然后由OkHttp内部线程池执行。

### 同步GET请求

```lua
-- 前面代码相同

-- 同步请求
response = call.execute()
print(response.body().string())
```

同步请求会阻塞调用线程,应在子线程中执行。Android 3.0+不允许主线程网络访问。

### POST提交String

```lua
import "okhttp3.*"

mediaType = MediaType.parse("text/plain; charset=utf-8")

requestBody = RequestBody.create(mediaType, "Hello World")
request = Request.Builder()
  .url("https://api.example.com/post")
  .post(requestBody)
  .build()

-- 发起请求
```

### POST提交文件

```lua
file = File("/path/to/file")
requestBody = RequestBody.create(MediaType.parse("application/octet-stream"), file)
request = Request.Builder()
  .url("https://api.example.com/upload")
  .post(requestBody) 
  .build()
```

### POST提交表单

```lua
requestBody = FormBody.Builder()
  .add("username", "test")
  .add("password", "123456")
  .build()
  
request = Request.Builder()
  .url("https://api.example.com/login")
  .post(requestBody)
  .build()
```

### 图片上传实例

```lua
-- 选择图片
intent = Intent(Intent.ACTION_PICK)
intent.setType("image/*")
this.startActivityForResult(intent, 1)

function onActivityResult(requestCode, resultCode, data)
  if data then
    -- 获取图片路径
    cursor = this.getContentResolver().query(data.getData(), null, null, null, null)
    cursor.moveToFirst()
    filePath = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA))
    
    -- 构建multipart请求体
    requestBody = MultipartBody.Builder()
      .setType(MultipartBody.FORM)
      .addFormDataPart("file", File(filePath).getName(),
        RequestBody.create(MediaType.parse("image/*"), File(filePath)))
      .build()
      
    -- 发起上传请求  
    request = Request.Builder()
      .url("https://api.example.com/upload")
      .post(requestBody)
      .build()
      
    OkHttpClient().newCall(request).enqueue(Callback{
      onResponse = function(call, response)
        print(response.body().string())
      end
    })
  end
end
```

## OkHttp优缺点

优点:
* 支持HTTP/2,可合并同主机请求
* 连接池减少延迟
* GZIP压缩
* 响应缓存
* 拦截器机制灵活

缺点:
* 回调需手动切换到主线程
* 和线程绑定使用较复杂

## 参考资料

* [OkHttp官方文档](https://square.github.io/okhttp/)
* [OkHttp3源码分析](https://www.jianshu.com/p/b0353ed71151)
* [Android OkHttp完全解析](https://blog.csdn.net/lmj623565791/article/details/47911083)
