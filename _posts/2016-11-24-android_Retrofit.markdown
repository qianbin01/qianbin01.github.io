---
layout:     post
title:      "android热门开源框架之网络篇(3)-Retrofit"
subtitle:   "OkHttp的进一步封装"
date:       2016-11-24 23:57:00
author:     "QB"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - http
    - 网络
    - curl
    - Retrofit
---

# Retrofit是什么

Retrofit与okhttp共同出自于Square公司，retrofit就是对okhttp做了一层封装。把网络请求都交给给了Okhttp，我们只需要通过简单的配置就能使用retrofit来进行网络请求了。<br>
再来看[官网](http://square.github.io/retrofit/)对Retrofit的描述<br>
A type-safe REST client for Android and Java<br>
一款针对Android和Java类型安全的http客户端
# Retrofit特性

1. 将rest API封装为java接口，我们根据业务需求来进行接口的封装，实际开发可能会封装多个不同的java接口以满足业务需求。
2. 使用Retrofit提供的封装方法将我们的生成我们接口的实现类，采用注解方便我们使用
3. 调用我们实现类对象的接口方法。

# Retrofit怎么用
[github官方文档](https://github.com/square/retrofit)<br>
1.导入工程<br>
Gradle

```  java 
compile 'com.squareup.retrofit2:retrofit:2.1.0'
```

Maven

```  java
<dependency>
  <groupId>com.squareup.retrofit2</groupId>
  <artifactId>retrofit</artifactId>
  <version>2.1.0</version>
</dependency>
```
2.简单Retrofit使用步骤
（一）创建接口

``` java
public interface LoginService {
//Result为response响应封装类
@GET("Login")
Call<Result> login(@Query(username) String username,@Query(password) String password);
}
```

1. @GET("Login")请求具体路径
2. @Query(请求参数名)
3. @Path，@QueryMap等注解可自行查询官方文档

（二）创建Retrofit对象

``` java
Retrofit retrofit = new Retrofit.Builder()  //获取Retrofit对象
                                .baseUrl(url) //链式结构 构造baseUrl
                                .addConverterFactory(GsonConverterFactory.create())//采用默认gson解析器
                                .build();//构造
LoginService service = retrofit.create(LoginService.class);//获取API接口的实现类的实例对象

```
（三）调用接口方法

``` java
Call<Result> result=service.login(qb,123456);
```

# 个人见解
Retrofit,将网络请求交给了OkHttp处理，自身采用接口的方式，让代码解耦，阅读更加清晰
用户简单封装后即可效率开发，推荐大家使用。

# 后记
目前比较火的一种网络异步请求方式为Rxjava+Retrofit，根据这个写了个demo<br>
地址大家可以参考参考[mvp下rxjava+retrofit的登录网络请求demo](http://qbfighting.cc/2016/11/24/android_mvp下rxjava+retrofit/)

