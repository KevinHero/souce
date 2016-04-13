---
title: ActionBar沉浸式实现
date: 2015-10-09 21:56:17
tags: 沉浸式
categories: 技术
---
 

studio，中引入沉浸式兼容库 
    
    compile 'com.readystatesoftware.systembartint:systembartint:1.0.3' 
    
eclipse，可以导入相应的那个类。

### 第一类，兼容actionbar

* 第一步：设置activity主题android:theme=”@style/ActionBarTheme”
<!--more-->

	<style name="ActionBarTheme"parent="android:Theme.Holo.Light.DarkActionBar">
	  <!--API 14 themecustomizationscangohere. -->
	  <item name="android:actionBarStyle">@style/ActionBarStyle</item>
	</style>
	<style name="ActionBarStyle"   parent="android:Widget.Holo.Light.ActionBar.Solid.Inverse">
	  <item name="android:background">@color/actionbar_bg</item>
	</style>

* 第二步：设置状态栏透明，然后设置状态栏沉浸的颜色


	@TargetApi(19)
	private void setTranslucentStatus(boolean on) {
	Window win = getWindow();
	WindowManager.LayoutParams winParams = win.getAttributes();
	final int bits = WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS;
		if (on) {
				winParams.flags |= bits;
			} else {
				winParams.flags &= ~bits;
			}
			win.setAttributes(winParams);
		}
	@Override
	protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_main);

	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
				setTranslucentStatus(true);
			}

	SystemBarTintManager tintManager = new SystemBarTintManager(this);
	tintManager.setStatusBarTintEnabled(true);
	//设置沉浸的颜色        
	tintManager.setStatusBarTintResource(R.color.statusbar_bg);
	}

* 第三步：设置适应windows，在布局文件设置


	android:fitsSystemWindows=”true”
	如果不设置，应用的ui会顶上去，顶进system ui
	ok

### 第二类 没有actionbar的activity

* 第一步，设置主题，android:theme=”@style/FullBleedTheme”


	<stylename="FullBleedTheme"parent="android:Theme.Holo.Light.NoActionBar">
	<!--API 14 themecustomizationscangohere. -->
	</style>
	<stylename="FullBleedTheme"parent="android:Theme.Holo.Light.NoActionBar.TranslucentDecor">
	<!--API 19 themecustomizationscangohere. -->
	</style>
    
> 或者用toolbar只能设置Theme.AppCompat.NoActionBar主题


	<style name="AppThemeToolbar" parent="Theme.AppCompat.NoActionBar">
		<itemname="colorPrimary">#2196F3</item>
		<itemname="colorPrimaryDark">#2196F3</item>
		<!--<item name="colorPrimaryDark">#1565C0</item>-->
		<itemname="colorAccent">#E91E63</item>
	</style>



* 第二步：同上一个第二步。
设置状态栏透明+颜色


	mTintManager = new SystemBarTintManager(this);
	mTintManager.setStatusBarTintEnabled(true);
	mTintManager.setNavigationBarTintEnabled(true);  mTintManager.setStatusBarTintResource(R.color.statusbar_bg);

* 第三步：


	android:fitsSystemWindows=”true”
	android:clipToPadding=”false
	<item name="android:fitsSystemWindows">true</item>
	<item name="android:clipToPadding">false</item>```


### 可能出现的问题
* android:fitsSystemWindows属性的奇怪问题

> 官方解释是布局的时候是否考虑系统的状态栏，标题栏，通知栏之类的。

>我的实际使用是，为true：那么布局的时候会把系统的状态栏，标题栏，通知栏的高度考虑进去。布局的内容会在状态栏，标题栏，通知栏的下面，不会被遮挡。

>但是在项目开发的过程中，突然发现对话框，AlertDialog的内容会超出背景大小，ProgressDialog的内容会超出背景大小并且不居中。

>也是很偶然才发现是这个属性造成的。我也没理清楚，大概是这个属性也使对话框考虑了系统的一些元素的缘故。

>所以我们在使用这个属性的时候不用直接把这个属性加到theme中。在相关的布局layout中使用就可以了。

* Android Toast显示文字超出了背景，文字布局中

>项目中突然出现了上述的情况，先开始以为是theme的问题，但是查了很久的资料，也做了很多实验，但是没有效果，还是之前的样子。一个很偶然的情况，```Toast.makeText(getActivity(), “密码不可为空”,Toast.LENGTH_SHORT).show();```改成了```Toast.makeText(App.getInstance(), “密码不可为空”,Toast.LENGTH_SHORT).show();```发现就可以了。

>别问我，我也不知道原因。

>但是这提醒我们，以后的Toast 的上下文参数，直接用ApplicationContext就对了。

Copyright (c) 2016 Copyright Holder All Rights Reserved.
