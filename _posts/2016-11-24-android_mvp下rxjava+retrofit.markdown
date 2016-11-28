---
layout:     post
title:      "mvp下rxjava+retrofit的登录网络请求demo"
subtitle:   ""
date:       2016-11-24 23:59:00
author:     "QB"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Demo
    - RecyclerView
    - mvp
    - rxjava
    - retrofit
---

> 这篇文章发表于[我的csdn](http://blog.csdn.net/hold_bin/article/details/53009329)

## 前言
最近在练手一个app，功能模块的不断扩大后，发现代码可读性和维护性越来越差，故打算系统的学习mvp模式，正好结合目前流行的rxjava+retrofit网络请求方式，写了个demo
有啥建议或意见可发邮箱236490794@qq.com  

[废话不多说，直接看正题 ](#build) 


<p id = "build"></p>
---

## 正文
首先上项目结构图
![](http://oh343spqg.bkt.clouddn.com/retrofit_project.png)


# data包：
User简单的请求类封装，Result结果类的封装，不进行展示

# http包：
HttpService.java

``` java
public interface HttpService {
    @GET("Login")
    Observable<Result> login(@Query("phone") String phone, @Query("password") String password);
}
```

HttpMethods.java

``` java
public class HttpMethods {
    public static final String BASE_URL = "http://youbangserver.cn/YouBang/";
    private static final int DEFAULT_TIMEOUT = 5;
    private Retrofit retrofit;
    private HttpService httpService;

    private HttpMethods() {
        OkHttpClient.Builder httpClientBuilder = new OkHttpClient.Builder();
        httpClientBuilder.connectTimeout(DEFAULT_TIMEOUT, TimeUnit.SECONDS);

        retrofit = new Retrofit.Builder()
                .client(httpClientBuilder.build())
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
                .baseUrl(BASE_URL)
                .build();

        httpService = retrofit.create(HttpService.class);
    }

    private static class SingletonHolder {
        private static final HttpMethods INSTANCE = new HttpMethods();
    }

    //获取单例
    public static HttpMethods getInstance() {
        return SingletonHolder.INSTANCE;
    }

    public void login(Subscriber<Result> subscriber, String phone, String password) {
        httpService.login(phone, password)
                .subscribeOn(Schedulers.io())
                .unsubscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(subscriber);
    }
}
```

比较基础的rxjava+retrofit的封装模式，采用的api是这几天搭建的一个服务器，可以根据自己的需求进行修改。

# presenter包

LoginPresenter.java

``` java
public class LoginPresenter {
    private ILoginActivity loginActivity;
    Subscriber<Result> subscriber;

    public LoginPresenter(ILoginActivity loginActivity) {
        this.loginActivity = loginActivity;
    }

    public void login() {
        subscriber = new Subscriber<Result>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {
                loginActivity.onFailure(e);
            }

            @Override
            public void onNext(Result result) {
                loginActivity.onSuccess(result);
            }
        };
        HttpMethods.getInstance().login(subscriber, loginActivity.getUserPhone(), loginActivity.getUserPassword());
    }
}
```
 对view接口进行操作，在onNext\onError中通过接口传递回调

# view包：
ILoginActivity.java

``` java
public interface ILoginActivity {
    String getUserPhone();

    String getUserPassword();

    void onSuccess(Result result);

    void onFailure(Throwable e);
}
``` 

提供参数设置方法，及回调接口
LoginActivity.java

```  java
public class LoginActivity extends AppCompatActivity implements ILoginActivity {
    @Bind(R.id.etPhone)
    EditText etPhone;
    @Bind(R.id.etPassword)
    EditText etPassword;
    @Bind(R.id.btnLogin)
    Button btnLogin;
    @Bind(R.id.tvResult)
    TextView tvResult;
    private LoginPresenter loginPresenter = new LoginPresenter(this);

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
    }

    @Override
    public String getUserPhone() {
        return etPhone.getText().toString();
    }

    @Override
    public String getUserPassword() {
        return etPassword.getText().toString();
    }

    @Override
    public void onSuccess(Result result) {
        tvResult.setText(result.toString());
    }

    @Override
    public void onFailure(Throwable e) {
        tvResult.setText(e.toString());
    }

    @OnClick(R.id.btnLogin)
    public void onClick() {
        loginPresenter.login();
    }
}
```

实现接口方法，持有presenter,对象，将事件传递给presenter类进行操作

上效果图
![](http://oh343spqg.bkt.clouddn.com/retrofit_login.png)

## 提醒
记住加网络权限