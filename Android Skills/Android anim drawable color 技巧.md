---
title: Android anim drawable color技巧
date: 2014-01-23 18:32:45
tags: drawable
categories: 技术
---


### Anim Set属性

<!--more-->

    <set xmlns:android="http://schemas.android.com/apk/res/android">

        <!--
            Set 属性

            shareInterpolator true时，interpolator应用到所有子元素中。
         -->

        <!--
            公共属性

            detachWallpaper 是否在壁纸上运行，设置了壁纸背景的窗口动画(window animation)才有效。
            设为true，则动画只在窗口运行，壁纸背景保持不变。

            fillEnabled true时，fillBefore的值才有效，否则fillBefore会被忽略。
            fillBefore true时，动画执行完后，View会还原动画执行前的初始状态；否则不会，默认即为true。

            repeatCount 动画重复执行的次数，默认为0，不重复；-1或infinite，无限重复。
            repeatMode 动画重复执行的模式，默认restart 动画重复执行时从起点开始；reverse 动画会反方向执行。

            startOffset 动画执行之前的等待时间，毫秒为单位；重复执行时，每次执行前同样执行等待时间。
            zAdjustment 动画的内容在动画运行时在z轴上的位置：
                            normal 默认值，在z轴上
                            top 在z轴最上层
           <?xml version="1.0" encoding="utf-8"?>

    <selector xmlns:android="http://schemas.android.com/apk/res/android">

        <item android:state_selected="true" android:color="#FFFFFFFF" />

        <item android:state_focused="true" android:color="#FFFFFFFF" />

        <item android:state_pressed="true" android:color="#FFFFFFFF" />

        <item android:color="#FF000000" />

    </selector>                     bottom 在z轴最下层
            interpolator 动画速率的变化
         -->

        <!--
            alpha 渐变动画
            对应的 AlphaAnimation
            AlphaAnimation alpha = (AlphaAnimation) AnimationUtils.loadAnimation(this, R.anim.alpha);
            view.startAnimation(alpha);

            duration 动画持续时间
            fromAlpha 动画开始时透明度，默认值1.0，最小值0.0
            toAlpha 动画结束时透明度，默认值1.0，最小值0.0
         -->
        <alpha
            android:duration="500"
            android:fromAlpha="0.0"
            android:toAlpha="1.0" />

        <!--
            scale 缩放动画
            对应的 ScaleAnimation
            ScaleAnimation scale = (ScaleAnimation) AnimationUtils.loadAnimation(this, R.anim.scale);
            view.startAnimation(scale);

            duration 动画持续时间
            fromXScale 动画开始时,x轴上（宽度）的缩放比例。0.0为没有大小，1.0为正常大小，大于1.0为放大，小于1.0为缩小。
            fromYScale 动画开始时,y轴上（高度）的缩放比例。0.0为没有大小，1.0为正常大小，大于1.0为放大，小于1.0为缩小。
            toXScale 动画结束时,x轴上（宽度）的缩放比例。0.0为没有大小，1.0为正常大小，大于1.0为放大，小于1.0为缩小。
            toYScale 动画结束时,y轴上（高度）的缩放比例。0.0为没有大小，1.0为正常大小，大于1.0为放大，小于1.0为缩小。
            pivotX 缩放时的固定不变的x坐标，0%为最左边，100%为最右边。
            pivotY 缩放时的固定不变的y坐标，0%为最顶部，100%为最底部。
        -->
        <scale
            android:duration="500"
            android:fromXScale="0.0"
            android:fromYScale="0.0"
            android:pivotX="0%"
            android:pivotY="100%"
            android:toXScale="1.0"
            android:toYScale="1.0" />

        <!--
            translate 平移动画
            对应的 TranslateAnimation
            TranslateAnimation translate = (TranslateAnimation) AnimationUtils.loadAnimation(this, R.anim.translate);
            view.startAnimation(translate);

            duration 动画持续时间
            fromXDelta 起始位置的X坐标的偏移量
            fromYDelta 结束位置的X坐标的偏移量
            toXDelta 起始位置的Y坐标的偏移量
            toYDelta 结束位置的Y坐标的偏移量

            1.-100到100，以"%"结束，表示相对于View本身的百分比位置；-100%表示控件左边或者上边，100%表示控件右边或者下边
            2.以"%p"（parent）结束，表示相对于View的父View的百分比位置；
            3.没有任何后缀，表示相对于View本身具体的像素值
         -->
        <translate
            android:duration="500"
            android:fromXDelta="100%"
            android:fromYDelta="-100%"
            android:toXDelta="-100%"
            android:toYDelta="100%" />

        <!--
            rotate 旋转动画
            对应的 RotateAnimation
            RotateAnimation rotate = (RotateAnimation) AnimationUtils.loadAnimation(this, R.anim.rotate);
            view.startAnimation(rotate);

            duration 动画持续时间
            fromDegrees 旋转开始的角度
            toDegrees 旋转结束的角度
            pivotX 旋转中心点的X坐标，纯数字表示相对于View本身左边缘的像素偏移量；
            带"%"后缀时表示相对于View本身左边缘的百分比偏移量；带"%p"后缀时表示相对于父View左边缘的百分比偏移量

            pivotY 旋转中心点的Y坐标，纯数字表示相对于View本身顶部边缘的像素偏移量；
            带"%"后缀时表示相对于View本身顶部边缘的百分比偏移量；带"%p"后缀时表示相对于父View顶部边缘的百分比偏移量
        -->
        <rotate
            android:duration="500"
            android:fromDegrees="0"
            android:pivotX="50%"
            android:pivotY="50%"
            android:toDegrees="360" />

    </set>

  

### Animi interpolator

    <set xmlns:android="http://schemas.android.com/apk/res/android">

        <!--
            加速度设置
            <动画标签
            android:interpolator="@资源id"/>
         -->

        <!--
            对应的类 AccelerateDecelerateInterpolator
            对应的资源 @android:anim/accelerate_decelerate_interpolator
            作用 动画开始与结束时速率改变比较慢，在中间时加速
        -->
        <accelerateDecelerateInterpolator />

        <!--
            对应的类 AccelerateInterpolator
            对应的资源 @android:anim/accelerate_interpolator
            作用 动画开始时速率改变比较慢，然后开始加速

            factor 加速速率，默认值1.0。
         -->
        <accelerateInterpolator android:factor="1" />

        <!--
            对应的类 AnticipateInterpolator
            对应的资源 @android:anim/anticipate_interpolator
            作用 动画开始的时候向后然后往前抛

            tension 向后的拉力，默认值2.0；0.0时，不会有向后的动画
         -->
        <anticipateInterpolator android:tension="2" />

        <!--
            对应的类 AnticipateOvershootInterpolator
            对应的资源 @android:anim/anticipate_overshoot_interpolator
            作用 动画开始的时候向后抛，然后再向前抛；会抛超过目标值后再返回到最后的值
        -->
        <anticipateOvershootInterpolator
            android:extraTension="2"
            android:tension="2" />

        <!--
            对应的类 BounceInterpolator
            对应的资源 @android:anim/bounce_interpolator
            作用 动画结束的时候会弹跳
        -->
        <bounceInterpolator />


        <!--
            对应的类 CycleInterpolator
            对应的资源 @android:anim/bounce_interpolator
            作用 动画循环做周期运动，速率改变沿着正弦曲线

            cycles 循环次数，默认值1
        -->
        <cycleInterpolator android:cycles="1" />

        <!--
            对应的类 DecelerateInterpolator
            对应的资源 @android:anim/decelerate_interpolator
            作用 动画开始时速率改变比较快，然后开始减速线

            factor 减速速率，默认为1.0
        -->
        <decelerateInterpolator android:factor="1" />

        <!--
            对应的类 LinearInterpolator
            对应的资源 @android:anim/decelerate_interpolator
            作用 动画匀速播放，没有可更改设置的属性
         -->
        <linearInterpolator />

        <!--
           对应的类 OvershootInterpolator
           对应的资源 @android:anim/overshoot_interpolator
           作用 动画向前抛，抛超过最后值，然后才返回

           tension 超出后的拉力，默认为2.0
        -->
        <overshootInterpolator android:tension="2" />

    </set>

 

### Anim animation-list实现帧动画

    <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
        android:oneshot="false">
        <!-- oneshot="true"表示只展示一遍，oneshot="false"表示不停止的循环展示
             android:duration 表示展示所用的该图片的时间长度  -->
        <item
            android:drawable="@mipmap/up"
            android:duration="500" />
        <item
            android:drawable="@mipmap/down"
            android:duration="500" />
    </animation-list>

  

### Anim 从左淡入

    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
        <!--
            fromXDelta 属性为动画起始时 X坐标上的位置
            toXDelta   属性为动画结束时 X坐标上的位置
            fromYDelta 属性为动画起始时 Y坐标上的位置
            toYDelta   属性为动画结束时 Y坐标上的位置
         -->
        <translate android:fromXDelta="-50%p" android:toXDelta="0"
            android:duration="400"/>
        <alpha android:fromAlpha="0.0" android:toAlpha="1.0"
            android:duration="400" />
    </set>

 

### Anim 从左淡出

    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
        <!--
            fromXDelta 属性为动画起始时 X坐标上的位置
            toXDelta   属性为动画结束时 X坐标上的位置
            fromYDelta 属性为动画起始时 Y坐标上的位置
            toYDelta   属性为动画结束时 Y坐标上的位置
         -->
        <translate android:fromXDelta="0" android:toXDelta="-50%p"
            android:duration="400"/>
        <alpha android:fromAlpha="1.0" android:toAlpha="0.0"
            android:duration="400" />
    </set>


### Anim 从右淡入

    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
        <!--
            fromXDelta 属性为动画起始时 X坐标上的位置
            toXDelta   属性为动画结束时 X坐标上的位置
            fromYDelta 属性为动画起始时 Y坐标上的位置
            toYDelta   属性为动画结束时 Y坐标上的位置
         -->
        <translate android:fromXDelta="50%p" android:toXDelta="0"
            android:duration="400"/>
        <alpha android:fromAlpha="0.0" android:toAlpha="1.0"
            android:duration="@integer/config_mediumAnimTime" />
    </set>

 
### Anim 从右淡出

    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
        <!--
            fromXDelta 属性为动画起始时 X坐标上的位置
            toXDelta   属性为动画结束时 X坐标上的位置
            fromYDelta 属性为动画起始时 Y坐标上的位置
            toYDelta   属性为动画结束时 Y坐标上的位置
         -->
        <translate android:fromXDelta="0" android:toXDelta="50%p"
            android:duration="400"/>
        <alpha android:fromAlpha="1.0" android:toAlpha="0.0"
            android:duration="400" />
    </set>


### Drawable shape属性

    <?xml version="1.0" encoding="utf-8"?>
    <shape xmlns:android="http://schemas.android.com/apk/res/android"
        android:shape="rectangle">

        <!-- android:shape：
            rectangle 矩形
            oval 椭圆形
            line 线性形状
            ring 环形
         -->

        <!-- 圆角
            radius 四角半径
            bottomLeftRadius 左下角半径
            bottomRightRadius 右下角半径
            topLeftRadius 左上角半径
            topRightRadius 右上角半径
        -->
        <corners
            android:bottomLeftRadius="6dp"
            android:bottomRightRadius="6dp"
            android:radius="12dp"
            android:topLeftRadius="6dp"
            android:topRightRadius="6dp" />

        <!-- 渐变
            startColor 起始颜色
            endColor 结束颜色
            angle 方向角度。当angle=0时，渐变色是从左向右；逆时针方向转，当angle=90时为从下往上。
            gradientRadius 渐变色半径（当 android:type="radial" 时才使用。单独使用 android:type="radial"会报错）
            type 渐变类型：
                linear 线性渐变，默认设置
                radial 放射性渐变，以开始色为中心。
                sweep 扫描线式的渐变。
            useLevel（如果要使用LevelListDrawable对象）：
                true无渐变色
                false有渐变色
            centerX 渐变中心X点坐标的相对位置
            centerY 渐变中心Y点坐标的相对位置
        -->
        <gradient
            android:angle="90"
            android:centerColor="@android:color/black"
            android:centerX="0"
            android:centerY="0"
            android:endColor="@android:color/black"
            android:gradientRadius="90"
            android:startColor="@android:color/white"
            android:type="radial"
            android:useLevel="true" />

        <!-- 内间距 -->
        <padding
            android:bottom="6dp"
            android:left="6dp"
            android:right="6dp"
            android:top="6dp" />


        <!-- 大小 -->
        <size
            android:width="60dp"
            android:height="60dp" />


        <!-- 填充 -->
        <solid android:color="@android:color/white" />


        <!-- 描边
            dashGap 虚线的宽度
            dashWidth   间隔宽度
        -->
        <stroke
            android:width="1dp"
            android:color="@android:color/black"
            android:dashGap="1dp"
            android:dashWidth="1dp" />

    </shape>

  
### Drawable selector属性

    <?xml version="1.0" encoding="utf-8"?>
    <selector xmlns:android="http://schemas.android.com/apk/res/android">

        <!-- 点击时 -->
        <item android:state_pressed="true" android:drawable="@mipmap/pressed"/>

        <!-- 选中时 -->
        <item android:state_selected="true" android:drawable="@mipmap/pressed"/>

        <!-- 获得焦点时 -->
        <item android:state_focused="true" android:drawable="@mipmap/pressed"/>

        <!-- 失去焦点时 -->
        <item android:state_focused="false" android:drawable="@mipmap/pressed"/>

        <!-- 不可用时 -->
        <item android:state_enabled="false" android:drawable="@mipmap/disabled"/>

        <!-- 可用时 -->
        <item android:state_enabled="true" android:drawable="@mipmap/enabled"/>

        <!-- 正常时 -->
        <item android:drawable="@mipmap/normal"/>

    </selector>

  

### Drawable selector+shape混搭

    <?xml version="1.0" encoding="utf-8"?>
    <selector xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:state_selected="true">
            <shape>
                <solid android:color="#ff38b8c1" />
                <corners android:radius="4px"></corners>
                <padding android:bottom="5dp" android:left="10dp" android:right="10dp" android:top="5dp" />
            </shape>
        </item>
        <item android:state_checked="true">
            <shape>
                <solid android:color="#ff38b8c1" />
           <?xml version="1.0" encoding="utf-8"?>

    <selector xmlns:android="http://schemas.android.com/apk/res/android">

        <item android:state_selected="true" android:color="#FFFFFFFF" />

        <item android:state_focused="true" android:color="#FFFFFFFF" />

        <item android:state_pressed="true" android:color="#FFFFFFFF" />

        <item android:color="#FF000000" />

    </selector>         <corners android:radius="4px"></corners>
                <padding android:bottom="5dp" android:left="10dp" android:right="10dp" android:top="5dp" />
            </shape>
        </item>
        <item android:state_pressed="true">
            <shape>
                <solid android:color="#ff38b8c1" />
                <corners android:radius="4px"></corners>
                <padding android:bottom="5dp" android:left="10dp" android:right="10dp" android:top="5dp" />
            </shape>
        </item>
        <item>
            <shape>
                <solid android:color="#00000000" />
                <corners android:radius="4px"></corners>
                <padding android:bottom="5dp" android:left="10dp" android:right="10dp" android:top="5dp" />
            </shape>
        </item>
    </selector>


### Drawable selector通用模板

上述的情况颇多，一般用这个模板都能实现大部分需求。

    <?xml version="1.0" encoding="utf-8"?>
    <selector xmlns:android="http://schemas.android.com/apk/res/android">

        <!-- 点击时 -->
        <item android:state_pressed="true" android:drawable="@mipmap/pressed"/>

        <!-- 选中时 -->
        <item android:state_selected="true" android:drawable="@mipmap/pressed"/>

        <!-- 获得焦点时 -->
        <item android:state_focused="true" android:drawable="@mipmap/pressed"/>

        <!-- 正常时 -->
        <item android:drawable="@mipmap/normal"/>

    </selector>


### Drawable layer-list 边框 底框不绘制

    <?xml version="1.0" encoding="utf-8"?>
    <layer-list xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:bottom="-2dp">
            <shape>
                <corners
                    android:topLeftRadius="2dp"
                    android:topRightRadius="2dp" />
                <solid android:color="#ffffffff" />
                <stroke
                    android:width="0.5dp"
                    android:color="@color/blue" />
            </shape>
        </item>
    </layer-list>

   

### Color selector通用模板

>color的selector属性和drawable的seletor属性是一样的，可以说只是从使用角度区分了selector。本质上，所有的selector都是一样的。

    <?xml version="1.0" encoding="utf-8"?>
    <selector xmlns:android="http://schemas.android.com/apk/res/android">
        <item android:state_selected="true" android:color="#FFFFFFFF" />
        <item android:state_focused="true" android:color="#FFFFFFFF" />
        <item android:state_pressed="true" android:color="#FFFFFFFF" />
        <item android:color="#FF000000" />
    </selector>