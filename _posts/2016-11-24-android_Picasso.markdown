---
layout:     post
title:      "android热门开源框架之图片篇-Picasso"
subtitle:   "一行代码轻松解决图片加载"
date:       2016-11-23 23:00:00
author:     "QB"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - 图片加载
    - Picasso
    - oom
---

> 在学习安卓的时候你是否经常被OOM、非UI线程更新等问题所苦恼？<br>Picasso,一行代码带你轻松解决图片加载。

#  Picasso是什么

 picasso是Square公司开源的一个Android图形缓存库,[官方地址](http://square.github.io/picasso/)，可以实现图片下载和缓存功能。仅仅只需要**一行代码**就能完全实现图片的异步加载。<br>
  Picasso不仅实现了图片异步加载的功能，还解决android中加载图片时需要解决的常见问题：<br>
  
  1. 在adapter中需要取消已经不在视野范围的ImageView图片资源的加载，否则会导致图片错位,Picasso已经解决了这个问题。<br>
  
  2. 使用复杂的图片压缩转换来尽可能的减少内存消耗<br>
  
  3. 自带内存和硬盘二级缓存功能<br>
  

#  Picasso怎么用
[github官方文档](https://github.com/square/picasso)<br>
1.导入工程<br>
Gradle

```  java 
compile 'com.squareup.picasso:picasso:2.5.2'
```

Maven

```  java
<dependency>
  <groupId>com.squareup.picasso</groupId>
  <artifactId>picasso</artifactId>
  <version>2.5.2</version>
</dependency>
```

<big><b>2.一行代码加载图片</b><big>

```  java
Picasso.with(context).load(url).into(imageview);
```

3.adapter中imageview加载问题

``` java
@Override 
public void getView(int position, View convertView, ViewGroup parent) {
  SquaredImageView view = (SquaredImageView) convertView;
  if (view == null) {
    view = new SquaredImageView(context);
  }
  String url = getItem(position);
  Picasso.with(context).load(url).into(view);
}
```

4.图片转换：转换图片以适应布局大小并减少内存占用

```  java
Picasso.with(context)
  .load(url)
  .resize(50, 50)
  .centerCrop()
  .into(imageView);
```

5.资源文件的加载：除了加载网络图片picasso还支持加载Resources, assets, files, contentproviders中的资源文件。

```  java
Picasso.with(context).load(R.drawable.icon).into(imageView1);
Picasso.with(context).load(new File(...)).into(imageView2);
Picasso.with(context).load(bitmap).into(imageView3);
```

# Picasso的原理简单剖析

从简单的一行代码加载图片开始看看到底干了些啥
Picasso.with(context).load(url).into(imageView)

```  java
public static Picasso with(Context context) {
        if (singleton == null) {
            singleton = new Builder(context).build();
        }
        return singleton;
    }
                                                                                                                  
    public Picasso build() {
            Context context = this.context;
                                                                                                              
            if (downloader == null) {
                downloader = Utils.createDefaultDownloader(context);
            }
            if (cache == null) {
                cache = new LruCache(context);
            }
            if (service == null) {
                service = new PicassoExecutorService();
            }
            if (transformer == null) {
                transformer = RequestTransformer.IDENTITY;
            }
                                                                                                              
            Stats stats = new Stats(cache);
                                                                                                              
            Dispatcher dispatcher = new Dispatcher(context, service, HANDLER,
                    downloader, cache, stats);
                                                                                                              
            return new Picasso(context, dispatcher, cache, listener,
                    transformer, stats, debugging);
        }
```

从源码中可以看出,Picasso.with()的时候会将执行所需的所有必备元素创建出来，如缓存cache、执行executorService、调度dispatch等。在load()时创建Request，在into()中创建action、bitmapHunter，并最终交给dispatcher执行。<br>
详细源码自行观察[github官方文档](https://github.com/square/picasso)

