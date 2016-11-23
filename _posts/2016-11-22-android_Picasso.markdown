---
layout:     post
title:      "android热门开源框架之图片篇-Picasso"
subtitle:   "一句话轻松解决图片加载"
date:       2016-11-22 10:00:00
author:     "QB"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - 图片加载
    - Picasso
    - oom
---

> 这篇文章发表于[我的csdn](http://blog.csdn.net/hold_bin/article/details/53037115)

## 原文
直接上代码，代码中注释详细，可直接观察

Model接口采用泛型，方便复用
```java
    void setData(List<T> list);
    List<T> getData();
    void deleteData(int position);
    void addOne(int index,T t);//index为数据插入位置
```
model实现类比较简单不进行展示

Presenter类
```java
    //view接口对象
    private IDemoView view;
    //model接口对象
    private IBiz data;
    private List<DemoModel> modelList;
    //模拟上拉可加载次数为3次
    private int index = 2;
    //下拉上拉核心操作
    public void addOne() {
        //模拟下拉操作，回调显示在主线程，实际可根据网络配置
        Observable
                .timer(2, TimeUnit.SECONDS, AndroidSchedulers.mainThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribe(new Action1<Long>() {
                    @Override
                    public void call(Long aLong) {
                        DemoModel demoModel = new DemoModel();
                        demoModel.setText("down item");
                        data.addOne(0, demoModel);
                        //view改变
                        view.addFinish();
                    }
                });
    }

    public void addMore() {
        //模拟上拉操作，回调显示在主线程，实际可根据网络配置
        Observable
                .timer(2, TimeUnit.SECONDS, AndroidSchedulers.mainThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribe(new Action1<Long>() {
                    @Override
                    public void call(Long aLong) {
                        if (index >= 0) {
                                DemoModel demoModel = new DemoModel();
                                demoModel.setText("up item" + index);
                                data.addOne(data.getData().size(), demoModel);
                            --index;
                            //view改变
                            view.addMoreFinish("上拉加载更多成功");
                        } else {
                            view.addMoreFinish("暂时没有新数据");
                        }
                    }
                });
    }
```
View 接口
```java
    void getData();

    void setData();

    void addOne();

    void deletaData(int position);//demo中加入左滑删除功能，读者可自行下载demo研究

    void addFinish();

    void addMore();

    void addMoreFinish(String notice);
```

View实现类
部分成员变量
``` java
    //MVP-P
    private DemoPresenter demoPresenter;

    //数据源
    private List<DemoModel> mList;
    //recycler布局管理器
    private LinearLayoutManager linearLayoutManager;
    //适配器
    private DemoAdapter mAdapter;

    //当前可见最后项
    private int lastVisibleItem;

    //判断上滑还是下拉
    private static int ADD_FLAG = 0;
    private static int ADD_HEAD = 0;
    private static int ADD_BOTTOM = 1;

//监听事件：
 swipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                addOne();//下拉刷新
            }
        });
        recycler.setOnItemListener(new ItemRemoveRecyclerView.OnItemClickListener() {
            @Override
            public void onItemClick(View view, int position) {
                System.out.println("click item" + position);//item点击事件
            }

            @Override
            public void onDeleteClick(int position) {//item左滑删除按钮事件
                deletaData(position);
                mAdapter.notifyItemRemoved(position);
                mAdapter.notifyItemRangeChanged(0, mList.size());
            }
        });
        recycler.addOnScrollListener(new RecyclerView.OnScrollListener() {
            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
                boolean isSignificantDelta = Math.abs(dy) > 10;//滑动幅度限制
                lastVisibleItem = linearLayoutManager.findLastVisibleItemPosition();
                if (isSignificantDelta) {
                    if (dy > 0) {//判断上下滑
                        ADD_FLAG = ADD_BOTTOM;
                    } else {
                        ADD_FLAG = ADD_HEAD;
                    }
                }
            }

            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                if (newState == RecyclerView.SCROLL_STATE_IDLE && lastVisibleItem + 1 == mAdapter.getItemCount() && ADD_FLAG == ADD_BOTTOM) {
                    addMore();//上拉加载更多
                }
            }
        });

   // 回调实现：
   //下拉刷新回调
    @Override
    public void addFinish() {
        mAdapter.notifyDataSetChanged();
        swipeRefreshLayout.setRefreshing(false);
        Toast.makeText(this, "下拉刷新成功", Toast.LENGTH_SHORT).show();
    }


    //上拉加载回调
    @Override
    public void addMoreFinish(String notice) {
        mAdapter.notifyDataSetChanged();
        mAdapter.changeMoreStatus(DemoAdapter.PULLUP_LOAD_MORE);
        Toast.makeText(this, notice, Toast.LENGTH_SHORT).show();
    }
```
Adapter类实现

```java
  //部分成员变量：
    private static final int TYPE_ITEM = 0;  //普通Item View
    private static final int TYPE_FOOTER = 1;//底部footview
    public static final int PULLUP_LOAD_MORE = 0;
    //正在加载中
    public static final int LOADING_MORE = 1;
    //上拉加载更多状态-默认为0
    private int load_more_status = 0;


        //item选择不同的type
        //判断类型选择不同item布局
        if (holder instanceof ViewHolder) {
            ((ViewHolder) holder).content.setText(mList.get(position).getText());
            holder.itemView.setTag(position);
        } else if (holder instanceof FootViewHolder) {
            FootViewHolder footViewHolder = (FootViewHolder) holder;
            switch (load_more_status) {
                case PULLUP_LOAD_MORE:
                    footViewHolder.foot_view_item_tv.setText("上拉加载更多...");
                    break;
                case LOADING_MORE:
                    footViewHolder.foot_view_item_tv.setText("正在加载更多数据...");
                    break;
            }
        }

   //上拉状态改变
   //上拉加载提示语状态修改
    public void changeMoreStatus(int status){
        load_more_status=status;
        notifyDataSetChanged();
    }
```

ps:本demo中实现了左滑删除的功能，可自行下载demo测试
大功告成
详情可自行下载[github](https://github.com/qianbin01/MVP_RecyclerView)

## 修改
mvp数据操作应放在model层中，本文放在p层中(偷懒)，在此指出。

