---
layout: post
title: Gesture手势检测和滑动冲突
category:
 - flutter
tags:
 - 手势
 - 滑动冲突
---

# Gesture手势检测和滑动冲突

最近flutter稳定版本由1.5升级到了1.7，`GestureDetector`手势检测类也得到了完善，新增了很多事件回调方法。接下来主要介绍下`GestureDetector`基本使用、注意事项、1.5到1.7升级引起的问题和滑动冲突。

## GestureDetector简介

* `GestureDetector`接受`child`参数，检测`child`上手势操作的回调

* `down` 和 `cancel`

  `GestureRecognizer`之间相互竞争，`down`事件可能会有多个手势生出并接收回调，当某一个事件流程正式胜出，其他`GestureRecognizer`会调用其`cancel方法`

  ```dart
  // down
  onTapDown;
  onVerticalDragDown;
  onHorizontalDragDown;
  onPanDown;
  // cancel
  onTapCancel;
  onVerticalDragCancel;
  onHorizontalDragCancel;
  onPanCancel;
  ```

* `tap`单击

  事件回调顺序`onTapDown -> onTapUp -> onTap`

  * `onTap`: 单击回调
  * `onTapUp`: 单击手势抬起回调
  
* `longPress`长按
  
  事件回调顺序`onLongPressStart -> onLongPress -> onLongPressMoveUpdate -> onLongPressEnd -> onLongPressUp`
  
  * `onLongPressStart`：长按开始
  * `onLongPress`：长按回调
  * `onLongPressMoveUpdate`：长按移动
  * `onLongPressEnd`：长按事件结束
  * `onLongPressUp`：长按手势抬起
  
* `onDoubleTap`双击回调
  
* `drag`水平/垂直拖拽
  
  事件回调顺序`onXXDragStart -> onXXDragUpdate -> onXXDragEnd`
  
  * `onXXDragStart`：拖拽开始
  * `onXXDragUpdate`：拖拽过程移动
  * `onXXDragEnd`：拖拽结束

* `scale`缩放
  
  事件回调顺序`onScaleStart -> onScaleUpdate -> onScaleEnd`
  
  * `onScaleStart`：缩放开始
  
  * `onScaleUpdate`：缩放更新
  
  * `onScaleEnd`：缩放结束
  
* `pan`拖拽
  
  事件回调顺序`onPanStart -> onPanUpdate -> onPanEnd`
  
  * `onPanStart`：拖拽开始
  * `onPanUpdate`：拖拽过程移动
  * `onPanEnd`：拖拽结束
  

##  Listener简介

如果需要处理原始用户手势事件，可以使用`Listner`，其提供了原始手指操作的回调，核心方法有`onPointerDown`，`onPointerMove`和`onPointerUp`，每一个事件流都是按照`onPointerDown -> onPointerMove -> onPointerUp`的顺序进行的，可以继承至`Listner`定制自定义手势检测。

* `onPointerDown`：手指按下屏幕
* `onPointerMove`：手指滑动
* `onPointerUp`：手指离开屏幕

## GestureDetector手势事件冲突

* `scale`和`pan`事件冲突，不能同时存在，建议直接使用`scale`，`ScaleUpdateDetails`参数中，如果`scale == 1`则可以认为进入`pan`流程，前后两个`focalPoint`的差值即是`DragUpdateDetails`中的`delta`
* `HorizontalDrag`和`VerticalDrag`不能共存，且如果存在`scale`的话，`Drag`事件优先级比较高，`scale`事件将被忽略。如果既要检测横向拖拽又要检测纵向拖拽，有两种方案：
  * `GestureDetector`嵌套，分别提供`vertivalDrag`和`horizontalDrag`的回调，优势是简单方便，劣势是不能检测任意方向上的滑动
  * `pan`事件处理，直接处理`pan`事件流，可以检测到任意方向上的拖拽，有点是比较灵活和全面，缺点是稍显复杂

## 升级1.7遇到的问题和解决方案

1.5及1.5以前的版本可以嵌套`GestureDetector`同时检测`scale`和`pan`，嵌套内层的`GestureDetector`优先处理事件流，如果内层嵌套不处理，则外层嵌套处理事件流，非常方便。

1.7上`scale`事件完全成为`pan`事件流的超集，不能共存，且不能嵌套，需要在`scale`相关回调中自行判断是滑动、缩放还是转动事件，通常，在不考虑转动事件下，认为若`scale == 1`为拖拽事件流，否则为缩放事件流，再进行相应操作。

## 滑动冲突的解决方案

典型场景，`TabBarView`结合内部可以滑动的view，例如大图页面`ScalableImageWidget`，会产生典型的滑动冲突。

解决方案是内部`ScalableImageWidget`优先级较高，决定是否处理滑动事件，若不处理，则移交给外部`TabBarView`处理。具体如下：

1. root页面持有`TabBarView`和其`child ScalableImageWidget`构造方法，设置标志位`_needHandleScroll`决定`TabBarView`的`physics`属性取`PageScrollPhysics`还是`NeverScrollableScrollPhysics`，设置`ScalableImageWidget`回调`scrollStateCallback`
2. `ScalableImageWidget`根据边界条件决定是否消费滑动事件，若不消费，则调用`ScrollStateCallback`回调设置root页面`_needHandleScroll`标志位，将处理权交给`TabBarView`
3. 若`TabBarView`获取了滑动事件处理权，则监听滑动动画的执行，滑动动画结束后将处理权移交给内部`child`