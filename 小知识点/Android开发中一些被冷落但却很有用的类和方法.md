---
title: Android开发中一些被冷落但却很有用的类和方法
date: 2014-02-17 19:23:12
tags: 总结分类
categories: 技术
---
  
### MediaMetadataRetriever

顾名思义，就是用来获取媒体文件一些相关信息的类。包括一首歌的标题，作者，专辑封面和名称，时长，比特率等等。如果是视频的话，可以获取视频的长宽，预览图。
    
    http://developer.android.com/intl/zh-cn/reference/android/media/MediaMetadataRetriever.html

### TouchDelegate`

用于更改View的触摸区域。场景：比如在RecyclerView的ItemView里包含了CheckBox组件, 然后想实现点击ItemView的时候，也可以触发CheckBox，就可以使用此类。
   
    http://developer.android.com/intl/zh-cn/training/gestures/viewgroup.html#delegate
### ArgbEvaluator
用于计算不同颜色值之间的插值，配合ValueAnimator.ofObject或者ViewPager.PageTransformer使用，可以实现不同颜色之间的平滑过渡。

    http://developer.android.com/intl/zh-cn/reference/android/animation/ArgbEvaluator.html
### Palette
用于提取一张图片的颜色。

    http://developer.android.com/intl/zh-cn/reference/android/support/v7/graphics/Palette.html

<!--more-->
### ViewDragHelper
做过自定义ViewGroup的童鞋都应该知道这个东西吧，用来处理触摸事件的神器，妈妈再也不用担心我自定义控件了。
   
    http://developer.android.com/intl/zh-cn/reference/android/support/v4/widget/ViewDragHelper.html
    http://www.cnblogs.com/lqstayreal/p/4500219.html
### PageTransformer
用于定义ViewPager页面切换时的动画效果（淡入淡出，放大缩小神马的…）官方有例子，直接看吧。

    http://developer.android.com/intl/zh-cn/training/animation/screen-slide.html
### ViewFlipper
可以实现简单轮播效果的一个组件。

    http://developer.android.com/intl/zh-cn/reference/android/widget/ViewFlipper.html
### LocalBroadcastManager
用于在APP内部使用的，效率和安全性更好的广播工具类。

    http://developer.android.com/intl/zh-cn/reference/android/support/v4/content/LocalBroadcastManager.html`
### Messenger
进程间通信的一个工具类。内部也是由AIDL实现的，但是用起来超级方便。

    http://developer.android.com/intl/zh-cn/reference/android/os/Messenger.html
    http://blog.csdn.net/lmj623565791/article/details/47017485`
### Formatter.formatFileSize
根据文件大小自动转为以KB, MB, GB为单位的工具类。想想以前都是自己计算的…

    http://developer.android.com/intl/zh-cn/reference/android/text/format/Formatter.html
### Activity.recreate
重新创建Activity。有什么用呢？可以在程序更换主题后，立马刷新当前Activity，而不会有明显的重启Activity的动画。

    http://developer.android.com/intl/zh-cn/reference/android/app/Activity.html#recreate%28%29
### View.getContext
顾名思义，就不用解释了吧…以前在写RecyclerView的Adapter的时候，为了使用LayoutInflater，经常傻乎乎地在构造函数中传入一个外部的context….是不是只有我不知道而已（笑cry脸）

    http://developer.android.com/intl/zh-cn/reference/android/view/View.html#getContext()
### View.post
方便在非UI线程对界面进行修改，与Handler的作用类似。并且由于post的Runnable会保证在该View绘制完成的前提下才调用，所以一般也可以用于获取View的宽高。

    http://developer.android.com/intl/zh-cn/reference/android/view/View.html#post(java.lang.Runnable)
### Activity.runOnUiThread
与View.post类似，方便在非UI线程中对界面进行修改。

    http://developer.android.com/intl/zh-cn/reference/android/app/Activity.html#runOnUiThread(java.lang.Runnable)
### Fragment.setUserVisibleHint
Fragment可以重写此方法，然后根据参数的布尔值（true的话表示当前Fragment对用户可见），来执行一些逻辑。

    http://developer.android.com/intl/zh-cn/reference/android/support/v4/app/Fragment.html#setUserVisibleHint(boolean)
### android:animateLayoutChanges
这是一个非常酷炫的属性。在父布局加上 android:animateLayoutChanges="true" 后，如果触发了layout方法（比如它的子View设置为GONE），系统就会自动帮你加上布局改变时的动画特效！！

    http://developer.android.com/intl/zh-cn/training/animation/layout.html
### android:clipToPadding
设置父view是否允许其子view在它的padding（这里指的是父View的padding）中绘制。是不是有点绕？举个实际场景吧：假如有个ListView，我们想要在初始位置时，第一项Item离顶部有10dp的距离，就可以在ListView的布局中加入android:clipToPadding="false" android:paddingTop="10dp"即可。是不是很方便呢？
    
    http://developer.android.com/intl/zh-cn/reference/android/view/ViewGroup.html#attr_android:clipToPadding
未完待续，不定期更新… 欢迎补充！
