---
layout:     post
title:      "android热门开源框架之网络篇(2)-OkHttp"
subtitle:   "OkHttp的初窥门径"
date:       2016-11-24 23:30:00
author:     "QB"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - 接口
    - http
    - curl
    - OkHttp
---



# OkHttp是什么

[官网](http://square.github.io/okhttp/)介绍：<br>
HTTP is the way modern applications network. It’s how we exchange data & media. Doing HTTP efficiently makes your stuff load faster and saves bandwidth.

OkHttp is an HTTP client that’s efficient by default:

- HTTP/2 support allows all requests to the same host to share a socket.
- Connection pooling reduces request latency (if HTTP/2 isn’t available).
- Transparent GZIP shrinks download sizes.
- Response caching avoids the network completely for repeat requests.

OkHttp perseveres when the network is troublesome: it will silently recover from common connection problems. If your service has multiple IP addresses OkHttp will attempt alternate addresses if the first connect fails. This is necessary for IPv4+IPv6 and for services hosted in redundant data centers. OkHttp initiates new connections with modern TLS features (SNI, ALPN), and falls back to TLS 1.0 if the handshake fails.

Using OkHttp is easy. Its request/response API is designed with fluent builders and immutability. It supports both synchronous blocking calls and async calls with callbacks.

OkHttp supports Android 2.3 and above. For Java, the minimum requirement is 1.7.<br>

简单的说：<b>OKHttp是一款高效的HTTP请求框架</b>

# OkHttp怎么用
[github官方文档](https://github.com/square/okhttp)<br>
1.导入工程<br>
Gradle

```  java 
compile 'com.squareup.okhttp3:okhttp:3.4.2'
```

Maven

```  java
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.4.2</version>
</dependency>
```

2.简单使用列表

- 一般的get请求
- 一般的post请求
- 基于Http的文件上传
- 文件下载
- 加载图片

（一）get请求

``` java
//创建OkHttpClient
OkHttpClient mOkHttpClient = new OkHttpClient();
//创建一个Request
Request request = new Request.Builder()
                .url("https://github.com/hongyangAndroid")
                .build();
//new call
Call call = mOkHttpClient.newCall(request); 
//请求加入调度（异步方式，同步为call.execute）
call.enqueue(new Callback()
        {
            @Override
            public void onFailure(Request request, IOException e)
            {
            }

            @Override
            public void onResponse(final Response response) throws IOException
            {
                    //网络请求响应返回处理,(非UI线程，若想界面UI更新需用到Handler)
            }
        });             
```
以上就是发送一个get请求的步骤，首先构造一个Request对象，参数最起码有个url，当然你可以通过Request.Builder设置更多的参数比如：header、method等。

（二）post请求
仅仅是Request构造方式不同

``` java
Request request = buildMultipartFormRequest(
        url, new File[]{file}, new String[]{fileKey}, null);
FormEncodingBuilder builder = new FormEncodingBuilder();   
builder.add("name","QB");

Request request = new Request.Builder()
                   .url(url)
                .post(builder.build())
                .build();
 mOkHttpClient.newCall(request).enqueue(new Callback(){});
```
（三）基于Http的文件上传
OkHttp中主要通过MultipartBuilder来构造我们文件上传的方式

``` java

File file = new File(filePath);

RequestBody fileBody = RequestBody.create(MediaType.parse("application/octet-stream"), file);

RequestBody requestBody = new MultipartBuilder()
     .type(MultipartBuilder.FORM)
     .addPart(Headers.of(
          "Content-Disposition", 
              "form-data; name=\"name\""), 
          RequestBody.create(null, "QB"))
     .addPart(Headers.of(
         "Content-Disposition", 
         "form-data; name=\"mFile\"; 
         filename=\"test.mp4\""), fileBody)
     .build();

Request request = new Request.Builder()
    .url(url)
    .post(requestBody)
    .build();

Call call = mOkHttpClient.newCall(request);
call.enqueue(new Callback()
{
    //Response
    //...
});

```

（四、五）文件下载，加载图片<br>
请求方式相同，在回调的response中进行byte[]编码或生成文件、或生成bitmap，相关知识自行查阅

# OKHttp 简单Demo
简单的okhttp get请求登录demo，Gson解析，记住修改demo中的url<br>
[demo-github地址](https://github.com/qianbin01/OkHttpDemo)

