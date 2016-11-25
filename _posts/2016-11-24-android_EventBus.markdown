---
layout:     post
title:      "android热门开源框架之事件篇-EventBus初级篇"
subtitle:   "优雅的handler替代品"
date:       2016-11-24 22:00:00
author:     "QB"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - EventBus
    - 组件间通讯
    - handler
---

#  EventBus 是什么
EventBus是Android下高效的发布/订阅事件总线机制。作用是可以代替传统的Intent,Handler,Broadcast或接口函数在Fragment,Activity,Service,线程之间传递数据，执行方法。特点是代码简洁，是一种发布订阅设计模式（Publish/Subsribe），或称作观察者设计模式。
 
 上结构图
 ![](http://oh343spqg.bkt.clouddn.com/EventBus.png)
 
1. Publisher是发布者， 通过post()方法将消息事件Event发布到事件总线
2. EventBus是事件总线， 遍历所有已经注册事件的订阅者们，找到里边的onEvent等4个方法，分发Event
3. Subscriber是订阅者， 收到事件总线发下来的消息。即onEvent方法被执行。注意参数类型必须和发布者发布的参数一致。   

#  EventBus 怎么用
[github官方文档](https://github.com/greenrobot/EventBus)<br>

1.导入工程<br>
Gradle:
    
```  java
compile 'org.greenrobot:eventbus:3.0.0'
```

Maven:
    
```  java 
<dependency>
    <groupId>org.greenrobot</groupId>
    <artifactId>eventbus</artifactId>
    <version>3.0.0</version>
</dependency>
```

2.定义事件

```  java
//类名，成员变量可自行修改，保持发送时接受时一致即可
public static class MessageEvent { /* Additional fields if needed */ }
```

3.准备观察者

```  java
@Subscribe(threadMode = ThreadMode.MAIN)  //订阅在主线程，具体可参考文档有哪些方式
public void onMessageEvent(MessageEvent event) {/* 这里做你需要做的事件 */};
//也可通过以下方式 选择运行方式
public void onEvent(MsgEvent1 msg)
public void onEventMainThread(MsgEvent1 msg)
public void onEventBackgroundThread(MsgEvent1 msg)
public void onEventAsync(MsgEvent1 msg)
```

ps：Eventbus需要注册和反注册

```  java
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}

@Override
public void onStop() {
    super.onStop();
    EventBus.getDefault().unregister(this);
}
```

 4.发布事件
 
```  java
EventBus.getDefault().post(new MessageEvent());
```

# EventBus的好处

EventBus是一个很棒的工具，它可用来对程序组件进行解耦。
相比于广播代码更加简洁，线程切换方便，使用门槛较低
使用自定义信息的形式代替了intent传递参数，减少书写错误产生的可能。

# EventBus的简易demo

Activity

```  java
public class MainActivity extends AppCompatActivity {
    private Button btnPost;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        btnPost = (Button) findViewById(R.id.btnPost);
        btnPost.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                EventBus.getDefault().post(new DemoEvent("发送的数据！！！"));
            }
        });
    }

    @Subscribe(threadMode = ThreadMode.MAIN)
    public void onEventDemo(DemoEvent event) {
        Toast.makeText(MainActivity.this, event.msg, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onStart() {
        super.onStart();
        EventBus.getDefault().register(this);
    }

    @Override
    public void onStop() {
        super.onStop();
        EventBus.getDefault().unregister(this);
    }
}

```

DemoEvent

```  java

public class DemoEvent {
    public String msg;
    public DemoEvent(String msg) {
        this.msg = msg;
    }
}

```

布局文件较简单就不贴了<br>
[demo-github地址](https://github.com/qianbin01/EventBusDemo)

