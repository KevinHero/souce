---
title: Android MotionEvent详解
date: 2015-12-21 12:12:45
tags: MotionEvent
categories: 技术
---



### 事件坐标的含义

 我们都知道，每个触摸事件都代表用户在屏幕上的一个动作，而每个动作必定有其发生的位置。在MotionEvent中就有一系列与标触摸事件发生位置相关的函数：

getX()和getY()：由这两个函数获得的x,y值是相对的坐标值，相对于消费这个事件的视图的左上点的坐标。
getRawX()和getRawY():有这两个函数获得的x,y值是绝对坐标，是相对于屏幕的。<!--more-->
 在之前的文章中，我们曾经分析过事件如何通过层层分发，最终到达消费它的视图手中。其中**ViewGroup**的`dispatchTransformedTouchEvent`函数有如下一段代码:
   
    final float offsetX = mScrollX - child.mLeft;
    final float offsetY = mScrollY - child.mTop;
    event.offsetLocation(offsetX, offsetY);
    handled = child.dispatchTouchEvent(event);
    event.offsetLocation(-offsetX, -offsetY);
    
 这段代码清晰展示了父视图把事件分发给子视图时，getX()和getY所获得的相关坐标是如何改变的。当父视图处理事件时，上述两个函数获得的相对坐标是相对于父视图的，然后通过上边这段代码，调整了相对坐标的值，让其变为相对于子视图啦。

 涉及MotionEvent使用的代码一般如下:

    action = MotionEventCompat.getActionMasked(event);
    switch(action) {
        MotionEvent.ACTION_DOWN:
            break;
        MotionEvent.ACTION_MOVE:
            break;
        MotionEvent.ACTION_UP:
            break;

 这里就引入了关于MotionEvent的一个重要概念，事件类型。事件类型就是指MotionEvent对象所代表的动作。比如说，当你的一个手指在屏幕上滑动一下时，系统会产生一系列的触摸事件对象,他们所代表的动作有所不同。有的事件代表你手指按下这个动作,有的事件代表你手指在屏幕上滑动,还有的事件代表你手指离开屏幕。这些事件的事件类型就分别为ACTION_DOWN,ACTION_MOVE,和**ACTION_UP**。上述这个动作所产生的一系列事件，被称为一个事件流，它包括一个**ACTION_DOWN**事件，很多个**ACTION_MOVE**事件，和一个ACTION_UP事件。

![](https://d9.usercdn.com/2/i/04033/9g2l6srojvha.gif)

 当然，除了这三个类型外，还有很多不同的事件类型,比如ACTION_CANCEL。它代表当前的手势被取消。要理解这个类型，就必须要了解ViewGroup分发事件的机制。一般来说，如果一个子视图接收了父视图分发给它的ACTION_DOWN事件，那么与ACTION_DOWN事件相关的事件流就都要分发给这个子视图，但是如果父视图希望拦截其中的一些事件，不再继续转发事件给这个子视图的话，那么就需要给子视图一个ACTION_CANCEL事件。
 其他的类型会在接下来的博文中一一解释。

#### Pointer

 细心的同学会发现，在上一节我描述用户手指在屏幕上滑动的例子时，特地说明了手指的数量为一个。那么当用户两个或者多个手指在屏幕上滑动时，系统又会产生怎样的事件流呢？
 为了可以表示多个触摸点的动作，MotionEvent中引入了Pointer的概念，一个pointer就代表一个触摸点，每个pointer都有自己的事件类型，也有自己的横轴坐标值。一个MotionEvent对象中可能会存储多个pointer的相关信息，每个pointer都会有一个自己的id和index。pointer的id在整个事件流中是不会发生变化的，但是index会发生变化。
 MotionEvent类中的很多方法都是可以传入一个int值作为参数的，其实传入的就是pointer的index值。比如getX(pointerIndex)和getY(pointerIndex)，此时，它们返回的就是index所代表的触摸点相关事件坐标值。
 由于pointer的index值在不同的MotionEvent对象中会发生变化，但是id值却不会变化。所以，当我们要记录一个触摸点的事件流时，就只需要保存其id,然后使用findPointerIndex(int)来获得其index值，然后再获得其他信息。

    private final static  INVALID_ID = -;
    private  mActivePointerId = INVALID_ID;
    private  mSecondaryPointerId = INVALID_ID;
    private float mPrimaryLastX = -;
    private float mPrimaryLastY = -;
    private float mSecondaryLastX = -;
    private float mSecondaryLastY = -;
    public boolean onTouchEvent(MotionEvent event) {
            action = MotionEventCompat.getActionMasked(event);

    switch (action) {
                MotionEvent.ACTION_DOWN:
                    index = event.getActionIndex();
                mActivePointerId = event.getPointerId(index);
                mPrimaryLastX = MotionEventCompat.getX(event,index);
                mPrimaryLastY = MotionEventCompat.getY(event,index);
                break;
                MotionEvent.ACTION_POINTER_DOWN:
                index = event.getActionIndex();
                mSecondaryPointerId = event.getPointerId(index);
                mSecondaryLastX = event.getX(index);
                mSecondaryLastY = event.getY(index);
                break;
                MotionEvent.ACTION_MOVE:
                index = event.findPointerIndex(mActivePointerId);
                    secondaryIndex = MotionEventCompat.findPointerIndex(event,mSecondaryPointerId);
                final float x = MotionEventCompat.getX(event,index);
                final float y = MotionEventCompat.getY(event,index);
                final float secondX = MotionEventCompat.getX(event,secondaryIndex);
                final float secondY = MotionEventCompat.getY(event,secondaryIndex);
                break;
                MotionEvent.ACTION_POINTER_UP:
                xxxxxx(涉及pointer id的转换，之后的文章会讲解)
                break;
                MotionEvent.ACTION_UP:
                MotionEvent.ACTION_CANCEL:
                mActivePointerId = INVALID_ID;
                mPrimaryLastX =-;
                mPrimaryLastY = -;
                break;

    return ;

 除了pointer的概念，MotionEvent还引入了两个事件类型：

`ACTION_POINTER_DOWN`:代表用户又使用一个手指触摸到屏幕上，也就是说，在已经有一个触摸点的情况下，有新出现了一个触摸点。

`ACTION_POINTER_UP`:代表用户的一个手指离开了触摸屏，但是还有其他手指还在触摸屏上。也就是说，在多个触摸点存在的情况下，其中一个触摸点消失了。它与ACTION_UP的区别就是，它是在多个触摸点中的一个触摸点消失时（此时，还有触摸点存在，也就是说用户还有手指触摸屏幕）产生，而ACTION_UP可以说是最后一个触摸点消失时产生。
 那么，用户先两个手指先后接触屏幕，同时滑动，然后在先后离开这一套动作所产生的事件流是什么样的呢？
 它所产生的事件流如下：

先产生一个ACTION_DOWN事件，代表用户的第一个手指接触到了屏幕。
再产生一个ACTION_POINTER_DOWN事件，代表用户的第二个手指接触到了屏幕。
很多的ACTION_MOVE事件，但是在这些MotionEvent对象中，都保存着两个触摸点滑动的信息，相关的代码我们会在文章的最后进行演示。
一个ACTION_POINTER_UP事件，代表用户的一个手指离开了屏幕。
如果用户剩下的手指还在滑动时，就会产生很多ACTION_MOVE事件。
一个ACTION_UP事件，代表用户的最后一个手指离开了屏幕

![](https://d9.usercdn.com/2/i/04033/2tkzlfdu5yrq.gif)

#### getAction 和 getActionMasked

 看到文章开头那段代码的同学可能会有点疑问：好像在很多代码里，大家都是通过getAction获得事件类型的，那么它和getActionMasked又有什么不同呢？
 从上一节我们可以得知，一个MotionEvent对象中可以包含多个触摸点的事件。当MotionEvent对象只包含一个触摸点的事件时，上边两个函数的结果是相同的，但是当包含多个触摸点时，二者的结果就不同啦。
 getAction获得的int值是由pointer的index值和事件类型值组合而成的，而getActionWithMasked则只返回事件的类型值
 举个例子（注:假设了int中不同位所代表的含义，可能不是例子所中的前8位代表id,后8位代表事件类型）:

getAction returns x0105.
getActionMasked will return x0005
其中x0100就是pointer的index值。
 一般来说，getAction() & ACTION_POINTER_INDEX_MASK就获得了pointer的id,等同于getActionIndex函数;getAction()& ACTION_MASK就获得了pointer的事件类型，等同于getActionMasked函数。

 为了效率，Android系统在处理ACTION_MOVE事件时会将连续的几个多触点移动事件打包到一个MotionEvent对象中。我们可以通过getX(int)和getY(int)来获得最近发生的一个触摸点事件的坐标，然后使用getHistorical(int,int)和getHistorical(int,int)来获得时间稍早的触点事件的坐标，二者是发生时间先后的关系。所以，我们应该先处理通过getHistoricalXX相关函数获得的事件信息，然后在处理当前的事件信息。
 下边就是Android Guide中相关的例子:

    printSamples(MotionEvent ev) {
        final  historySize = ev.getHistorySize();
        final  pointerCount = ev.getPointerCount();
        ( h = ; h < historySize; h++) {
            System.out.printf("At time %d:", ev.getHistoricalEventTime(h));
            ( p = ; p < pointerCount; p++) {
                System.out.printf("  pointer %d: (%f,%f)",
                    ev.getPointerId(p), ev.getHistoricalX(p, h), ev.getHistoricalY(p, h));


        System.out.printf("At time %d:", ev.getEventTime());
        ( p = ; p < pointerCount; p++) {
            System.out.printf("  pointer %d: (%f,%f)",
                ev.getPointerId(p), ev.getX(p), ev.getY(p));
