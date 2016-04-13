---
title: ImageView ScaleType属性
date: 2014-05-11 22:01:14
tags: ImageView
categories: 技术
---


原图
![](http://img.blog.csdn.net/20150913233900119)


<!--more-->
### android:scaleType=”center”

按原图大小显示图片，但图片宽高大于ImageView的宽高时，截图图片中间部分显示。

     <ImageView
         android:id="@+id/center"
         android:layout_width="200dp"
         android:layout_height="200dp"
         android:layout_marginBottom="20dp"
         android:scaleType="center"
         android:src="@mipmap/mm" />


![](http://img.blog.csdn.net/20150913233833622)

center

### android:scaleType=”centerCrop”

android:scaleType=”centerCrop”是最适用的显示方式

按比例放大原图直至等于某边ImageView的宽高显示。

    <ImageView
         android:id="@+id/centerCrop"
         android:layout_width="200dp"
         android:layout_height="200dp"
         android:layout_marginBottom="20dp"
         android:scaleType="centerCrop"
         android:src="@mipmap/mm" />


![](http://img.blog.csdn.net/20150913234247941)

centerCrop
### android:scaleType=”centerInside”

当原图宽高或等于ImageView的宽高时，按原图大小居中显示；反之将原图缩放至ImageView的宽高居中显示。

    <ImageView
         android:id="@+id/centerInside"
         android:layout_width="200dp"
         android:layout_height="200dp"
         android:layout_marginBottom="20dp"
         android:scaleType="centerInside"
         android:src="@mipmap/mm" />

![](http://img.blog.csdn.net/20150913234228091)

centerInside

### android:scaleType=”fitCenter”

按比例拉伸图片，拉伸后图片的高度为ImageView的高度，且显示在ImageView的中间。

    <ImageView
         android:id="@+id/fitCenter"
         android:layout_width="200dp"
         android:layout_height="200dp"
         android:layout_marginBottom="20dp"
         android:scaleType="fitCenter"
         android:src="@mipmap/mm" />

![](http://img.blog.csdn.net/20150913234209942)

fitCenter

### android:scaleType=”fitEnd”

按比例拉伸图片，拉伸后图片的高度为ImageView的高度，且显示在ImageView的右边。

    <ImageView
         android:id="@+id/fitEnd"
         android:layout_width="200dp"
         android:layout_height="200dp"
         android:layout_marginBottom="20dp"
         android:scaleType="fitEnd"
         android:src="@mipmap/mm" />

![](http://img.blog.csdn.net/20150913234153776)

fitEnd

### android:scaleType=”fitStart”

按比例拉伸图片，拉伸后图片的高度为ImageView的高度，且显示在ImageView的左边。

    <ImageView
         android:id="@+id/fitStart"
         android:layout_width="200dp"
         android:layout_height="200dp"
         android:layout_marginBottom="20dp"
         android:scaleType="fitStart"
         android:src="@mipmap/mm" />

![](http://img.blog.csdn.net/20150913234133129)

fitStart

### android:id=”@+id/fitXY”

拉伸图片（不按比例）以填充ImageView的宽高。

    <ImageView
         android:id="@+id/fitXY"
         android:layout_width="200dp"
         android:layout_height="200dp"
         android:layout_marginBottom="20dp"
         android:scaleType="fitXY"
         android:src="@mipmap/mm" />

![](http://img.blog.csdn.net/20150913234100805)

fitXY

### android:scaleType=”matrix”

保持原图的效果（不随着ImageView的大小而变化），图片的左上角和ImageView的左上角对齐。

    <ImageView
         android:id="@+id/matrix"
         android:layout_width="200dp"
         android:layout_height="200dp"
         android:layout_marginBottom="20dp"
         android:scaleType="matrix"
         android:src="@mipmap/mm" />

![](http://img.blog.csdn.net/20150913234030545)
matrix
