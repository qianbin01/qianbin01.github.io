---
layout:     post
title:      "android热门开源框架之网络篇(1)-Volley"
subtitle:   "浅谈volley原理及初步使用"
date:       2016-11-21 12:00:00
author:     "QB"
header-img: "img/post-bg-android.jpg"
tags:
    - Volley
    - 网络交互
    - Android
---

> 这篇文章发表于[我的csdn](http://blog.csdn.net/hold_bin/article/details/52107289)


在android开发一定阶段后不可避免的接触到网络连接，然后向往常一样查找android有关网络请求的实现，然后发现了HttpURLConnection和HttpClient，紧接着被一大串的代码吓到，不够人性啊，Android开源框架这么多，果断寻找相应的框架，于是便遇到了Volley。

Volley在2013年Google I/O大会上被提出：使得Android应用网络操作更方便更快捷；抽象了底层Http Client等实现的细节，让开发者更专注与产生RESTful Request。另外，Volley在不同的线程上异步执行所有请求而避免了阻塞主线程。

总结下Volley具有以下特点
自动调度网络请求
多个并发的网络连接
通过使用标准的HTTP缓存机制保持磁盘和内存响应的一致
支持请求优先级
支持取消请求的强大API，可以取消单个请求或多个
易于定制
健壮性：便于正确的更新UI和获取数据
包含调试和追踪工具

先上github地址
[volley-github](https://github.com/mcxiaoke/android-volley)

Volley的基本使用<br>

一、构建一个请求队列
```java
RequestQueue mQueue=Volley.newRequestQueue(getApplicationContext());
```
二、添加一个GET请求(以json对象为例)
```java
mQueue.add(new JsonObjectRequest(Method.GET, url, null,  
            new Listener() {  
                @Override  
                public void onResponse(JSONObject response) { 
                   //请求成功回调函数 
                    Log.d(TAG, "response : " + response.toString());  
                }  
            }, null));  
mQueue.start(); 
```
三、图片请求
```java
ImageRequest imageRequest = new ImageRequest(  
        url,  
        new Response.Listener<Bitmap>() {  
            @Override  
            public void onResponse(Bitmap response) {  
                imageView.setImageBitmap(response);  
            }  
        }, 0, 0, Config.RGB_565, new Response.ErrorListener() {  
            @Override  
            public void onErrorResponse(VolleyError error) {  
                imageView.setImageResource(R.drawable.default_image);  
            }  
        });
```
参数详解<br>
1.图片资源地址 <br>
2.请求成功回调<br>
3.4参数分别设定最大宽度高度，超出则压缩，0表示都不进行压缩 <br>
5.图片的编码格式<br>
 6.请求失败回调<br>
 
 四、防止Activity销毁时产生crash
通常会在Activity重载生命周期函数中进行防守
```java
@Override 
pubic void onStop() {  
    mQueue.cancelAll(this);  
}  
```
或通过取消某个设置的tag请求来防守<br>

五、Volley总结
Volley适用于网络图片加载与json数据的获取，但遇到大数据也十分不好用需要进行很多的方法封装，故使用时请自行思考。
