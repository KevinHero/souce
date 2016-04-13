---
title: 图像、背景、View更新、布局、内存
date: 2014-11-05 22:01:14
tags: 图像、背景、View更新、布局、内存
categories: 技术
---



###  图像

选择适当的图像尺寸
视图背景图像总会填充整个视图区域

    1.视图背景图像总会填充整个视图区域
    2.避免实时缩放
    3.最好预先缩放到视图大小

<!--more-->
- - -


    package com.zyy.android_csdn.skill;

    import android.graphics.Bitmap;
    import android.view.View;

    /**
     *
     * 精致的图片
     *
     * @author CaMnter
     *
     */
    public class ExquisiteImage {

    	// 被缩放的图片
    	private Bitmap originalImage;

    	// 视图
    	private View view;

    	public ExquisiteImage(Bitmap originalImage, View view) {

    		this.originalImage = originalImage;

    		this.view = view;

    	}

    	/**
    	 * @return 适应View的Bitmap
    	 */
    	public Bitmap getBitmap() {

    		/**
    		 *
    		 * 第一参数：被缩放图像
    		 * 第二参数：视图宽度
    		 * 第三参数：视图高度
    		 * 第四参数：双线性过滤器
    		 *
    		 */
    		return Bitmap.createScaledBitmap(this.originalImage,
    				this.view.getWidth(), this.view.getHeight(), true);

    	}

    }



### 背景


            默认情况下, 窗口有一个不透明的背景

            有时可以不需要
          1.最高层的视图是不透明的
             2.最高层的视图覆盖整个窗口
                <1>.layout_width = fill_parent
                <2>.layout_height = fill_parent
             更新看不见的背景是浪费时间

- - -


    	@Override
    	protected void onCreate(Bundle savedInstanceState) {

    		super.onCreate(savedInstanceState);

    		setContentView(R.layout.activity_main);

    		// 删除窗口背景
    		getWindow().setBackgroundDrawable(null);

    		...

    	}





### View更新

当屏幕需要更新时, 调用 invalidate()

     1.简单方便
     2.但是会更新整个视图, 代价太大了


最好先找到无效的区域，然后调用

    1.invalidate(Rect dirty);
    2.invalidate(int left, int top, int right, int bottom);




优化上：就是 原来的 刷新了整个视图，现在的只是刷新了 改变后的位置和原来的位置 在内存的一块区域。






### 布局

越简单越好


如果一个窗口包含很多视图：
     
     1.启动时间长
     2.测量时间长
     3.布局时间长
     4.绘制时间长


如果视图树深度太深

    1.StackOverflowException
    2.用户界面反应速度很慢


解决办法：

    1.使用TextView的复合drawables减少层次
    2.使用ViewStub延迟展开视图
    3.使用<merge>合并中间视图
    4.使用RelativeLayout减少层次
    5.使用自定义视图
    6.使用自定义布局






#### 内存

不要随意创建 Java 对象

在以下性能敏感的代码中，尽量避免创建Java对象：

    1.测量：onMeasure()
    2.布局：onLayout()
    3.绘图：dispatchDraw(), onDraw()
    4.事件处理：dispatchTouchEvent(), onTouchEvent()
    5.Adapter：getView(), bindView()


GC，垃圾回收

	1.整个程序会暂停
	2.慢（大概好几百个毫秒）



管理好Java对象

	1.使用软引用：内存缓存的最佳选择
	2.使用弱引用：避免内存泄露

-----



	private final HashMap<String, SoftReference<T>> mCache;

	public void put(String key, T value) {
		mCache.put(key, new SoftReference<T>(value));
	}

	public T get(String key, ValueBuilder builder) {
		T value = null;
		SoftReferece<T> reference = mCache.get(key);
		if (reference != null) {
			value = reference.get();
		}
		if (value == null) {
			value = builder.build(key);
			mCache.put(key, new SoftReference<T>(value));
		}
		return value;
	}
