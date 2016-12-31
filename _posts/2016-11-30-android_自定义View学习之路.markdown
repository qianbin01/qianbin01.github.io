---
layout:     post
title:      "android自定义View学习之路"
subtitle:   ""
date:       2016-11-30 10:58:00
author:     "QB"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - 学习
    - 自定义View
---


# 理论有关

## 绘制及坐标相关
自定义view基本流程

![](http://oh343spqg.bkt.clouddn.com/view2.jpg)

获得坐标方法

``` java
getTop();       //获取子View左上角距父View顶部的距离
getLeft();      //获取子View左上角距父View左侧的距离
getBottom();    //获取子View右下角距父View顶部的距离
getRight();     //获取子View右下角距父View左侧的距离
```

![](http://oh343spqg.bkt.clouddn.com/view1.jpg)

## 事件分发核心要点

1. 事件分发原理: 责任链模式，事件层层传递，直到被消费。
2. View 的 dispatchTouchEvent 主要用于调度自身的监听器和 onTouchEvent。
3. View的事件的调度顺序是 onTouchListener > onTouchEvent > onLongClickListener > onClickListener 。
4. 不论 View 自身是否注册点击事件，只要 View 是可点击的就会消费事件。
5. 事件是否被消费由返回值决定，true 表示消费，false 表示不消费，与是否使用了事件无关。
6. ViewGroup 中可能有多个 ChildView 时，将事件分配给包含点击位置的 ChildView。
7. ViewGroup 和 ChildView 同时注册了事件监听器(onClick等)，由 ChildView 消费。
8. 一次触摸流程中产生事件应被同一 View 消费，全部接收或者全部拒绝。
9. 只要接受 ACTION_DOWN 就意味着接受所有的事件，拒绝 ACTION_DOWN 则不会收到后续内容。
10. 如果当前正在处理的事件被上层 View 拦截，会收到一个 ACTION_CANCEL，后续事件不会再传递过来。

## 单点触控与多点触控

1. 多点触控时必须使用 getActionMasked() 来获取事件类型。
2. 单点触控时由于事件数值不变，使用 getAction() 和 getActionMasked() 两个方法都可以。
3. 使用 getActionIndex() 可以获取到这个index数值。不过请注意，getActionIndex() 只在 down 和 up 时有效，move 时是无效的。
4. PointId 在手指按下时产生，手指抬起或者事件被取消后消失，是一个事件流程中唯一不变的标识，可以在手指按下时 通过 getPointerId(int pointerIndex) 获得

# 表格有关

## Canvas 常用api速查表格

操作类型	 |	相关API	 |	备注
----  |	 ------  |	-----
绘制颜色  |	drawColor, drawRGB, drawARGB	 |	使用单一颜色填充整个画布
绘制基本形状	 |	drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc	 |	依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧
绘制图片	 |	drawBitmap, drawPicture	 |	绘制位图和图片
绘制文本 |		drawText, drawPosText, drawTextOnPath	 |	依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字
绘制路径 |		drawPath	 |	绘制路径，绘制贝塞尔曲线时也需要用到该函数
顶点操作 |		drawVertices, drawBitmapMesh	 |	通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用
画布剪裁 |		clipPath, clipRect	 |	设置画布的显示区域
画布快照	 |	save, restore, saveLayerXxx, restoreToCount, getSaveCount  |		依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 回滚到指定状态、 获取保存次数
画布变换 |		translate, scale, rotate, skew	 |	依次为 位移、缩放、 旋转、错切
Matrix(矩阵) |		getMatrix, setMatrix, concat	 |	实际上画布的位移，缩放等操作的都是图像矩阵Matrix， 只不过Matrix比较难以理解和使用，故封装了一些常用的方法。

## 单点触控表格

 事件 |	简介
 ---- | -----
ACTION_DOWN	 | 手指 初次接触到屏幕 时触发。
ACTION_MOVE	| 手指 在屏幕上滑动 时触发，会多次触发。
ACTION_UP	| 手指 离开屏幕 时触发。
ACTION_CANCEL	| 事件 被上层拦截 时触发。
ACTION_OUTSIDE	 | 手指 不在控件区域 时触发。

方法 |	简介
 ---- | ------
getAction()	 | 获取事件类型。
getX()	| 获得触摸点在当前 View 的 X 轴坐标。
getY()	| 获得触摸点在当前 View 的 Y 轴坐标。
getRawX()	| 获得触摸点在整个屏幕的 X 轴坐标。
getRawY()	| 获得触摸点在整个屏幕的 Y 轴坐标。
