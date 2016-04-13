---
title: Android Textview 技巧
date: 2014-05-15 22:01:14
tags: Textview
categories: 技术
---


### 部分文字设置颜色

    TextView tv = new TextView(this);
    //添加css样式
    tv.setText(Html.fromHtml("<font color=\"E32910\">"+"红色"+"</font>的字"));

<!--more-->

### 文字透明度设置

可以通过设置color设置

    <color name="text_color">#FFFFFF</color>



这是一个白色，假如要设置70%的白色

0-255 十进制的

255*70% ＝ 178

178（十进制） ＝ B2（16进制

    <color name="text_color">#B2FFFFFF</color>

就是70%透明度的白色

### 代码设置DrawableLeft

    //第一种

    view.setCompoundDrawablesWithIntrinsicBounds(int drawableLeft, 0, 0, 0);

    //第二种

    view.setCompoundDrawablesWithIntrinsicBounds(Drawable drawableLeft, null, null, null);

    //第三种

    Drawable drawable= getResources().getDrawable(R.drawable.drawable);  
    //不设置Bounds,则不会显示.  

    drawable.setBounds(0, 0, drawable.getMinimumWidth(), drawable.getMinimumHeight());  
    view.setCompoundDrawables(drawable,null,null,null);  

### 内容换行

    /n

### 设置行间距

    1、android:lineSpacingExtra
    设置行间距，如”3dp”。

    2、android:lineSpacingMultiplier
    设置行间距的倍数，如”1.2″。

### 设置下划线

    //第一种方式
    textView.setText(Html.fromHtml("<u>"+"下划线"+"</u>"));

    //第二种方式
    textView.getPaint().setFlags(Paint.UNDERLINE_TEXT_FLAG );


### 抗锯齿

    textView.getPaint().setAntiAlias(true);



### 设置中划线

    //中划线
    textview.getPaint().setFlags(Paint. STRIKE_THRU_TEXT_FLAG);

    //设置中划线并加清晰
    textView.getPaint().setFlags(Paint. STRIKE_THRU_TEXT_FLAG|Paint.ANTI_ALIAS_FLAG);

    //取消划线
    textView.getPaint().setFlags(0);



### 单行显示

在开头显示省略号

    android:singleLine="true"  
    android:ellipsize="start"



在结尾显示省略号

    android:singleLine="true"  
    android:ellipsize="end"

在中间显示省略号

    android:singleLine="true"  
    android:ellipsize="middle"


### 添加滚动条

    textView.setMovementMethod(ScrollingMovementMethod.getInstance());
