---
title: Google Develop for Android 系列九 － 工具
date: 2015-06-16 21:12:17
tags: Google Develop for Android 系列
categories: 技术
---


原文： [Developing for Android IX Tools](https://medium.com/google-developers/developing-for-android-ix-tools-375134af1098)

安卓中有许多帮助调试与分析app所存在问题的工具，从内存分析工具Allocation Tracker（ddms和Android Studio中都有）到设备端的性能分析工具。

重要的不是你知道这些工具的存在，而是你实际使用了它们，用它们验证app是否和你期望的表现一致（是否达到60 fps，是否避免了garbage churn，等等），让安卓成为一个更好的平台。

常用的性能分析工具有两类：主机端的和设备端的。主机端的工具是指那些在电脑上运行的（通过命令行，DDMS，Android Studio等等）。设备端工具是指实时显示当前设备所发生事情的信息的工具，主要是与性能有关的指标。

主机端工具
-----

### Systrace
<!--more-->
[Systrace](http://developer.android.com/tools/help/systrace.html) 是一个在时间轴上显示系统内所发生的事件以及事件持续时间的强大工具。你可以使用这个工具抓取系统中的数据，然后在浏览器（在mac上safari无法打开，要用chrome）中打开包含追踪结果的文件，根据图像分析结果。这个工具可以在Android Studio中运行，也可以使用命令行：$SDK\_ROOT/platform-tools/systrace/systrace.py。

注：关于systrace的详细使用请参考官方文档：<http://developer.android.com/tools/help/systrace.html> 。也可以参考这个人的两篇中文博客 [Android 4.1 systrace使用方法(一)](http://blog.csdn.net/oujunli/article/details/8138172) 、 [Android 4.1性能分析的一般步骤(二)](http://blog.csdn.net/oujunli/article/details/8138290) 

### AllocationTracker 内存分配检测工具

AllocationTracker可以从ddms运行，也可以直接从Android Studio里面运行。它可以让你找出点击开始截获与获取数据两个按钮的时间段内的所有内存分配情况。这个工具对于想知道在本不想分配内存的时间段内，是否有内存分配是非常有用的，比如，如果你想找出在一个特定动画期间是否有分配内存，你可以在设备上先开始动画，然后点击Start Allocations”（“Start Allocations”）按钮，数秒之后，再在动画结束之前点击“Get Allocations” 按钮（你可能需要为动画设置一个比默认更长的持续时间，让操作有足够的时间完成）。所有动画期间的内存分配行为都将检测到。更多的详细信息参考[内存调试](https://developer.android.com/tools/debugging/debugging-memory.html)一文。

 注：还可以参考这篇文章：[你不知道的Eclipse用法：使用Allocation tracker跟踪Android应用内存分配](http://blog.csdn.net/p106786860/article/details/9248693)。 

### Traceview

Traceview，也可以从ddms或Android Studio运行。它时一个方法分析工具，有race mode（将跟踪每个方法调用）和 sample mode （在指定时间内采样）两种运行模式。跟踪模式是找出跟踪期间整个app代码路径的好方法，同时也能对不同方法执行的相对时间有大致了解。但是这个检测比较昂贵，采样模式不必测量每个方法调用，因此避免了这些开销，但缺点是会遗漏代码流程中的一些细节。同时要注意采样模式并不总是有效（因为运行时只能采样GC安全的点），它无法采样内联函数。而且它对于JNI方法的采样比例要多余java方法。

### Hierarchyviewer

Hierarchyviewer既可以作为单独的工具运行也可以从Android Studio运行，它可以显示被选择应用的整个view树结构，同时还允许你看到任意view的属性值。这个工具是了解树复杂程度的绝佳方法。是否维护了太多多数时候都不可见的view？是否有层次太深的结构，而使用简单的布局就能避免这种复杂的层次？这个工具是诊断这些问题的好方法。

### MAT (内存分析工具)

有时候你会怀疑activity中是否存在内存泄漏，因为内存正在无边无际的增长。也许有像activity这种级别的大对象正在发生严重泄漏，可能发生在onfiguration 改变期间。这个时候MAT就是追踪这些问题的好工具。

首先使用ddms或者Android Studio触发一个heap dump。这会捕获期间的对象层的状态。然后你可以用hprof-conv 命令行工具将这些数据转换成可以背MAT使用的版本。 最后你可以使用MAT工具加载这些数据（独立的工具或者Eclipse插件）以查看内存图像，查找哪些对象常驻内存及常驻的原因。

可以参考2011年Google I/O大会上关于MAT的教程：[Memory Management for Android Apps](https://www.youtube.com/watch?v=_CruQY55HOk) ，以及从[内存调试](https://developer.android.com/tools/debugging/debugging-memory.html) 一文获取更多信息。

### Memory Monitor

Memory Monitor 是 Android Studio内置的新工具，用于跟踪选中进程中堆随时间变化的使用情况。

### meminfo

运行adb shell dumpsys meminfo’可以输出关于整个系统或者指定进程的各种数据。详情见[内存调试](https://developer.android.com/tools/debugging/debugging-memory.html)一文。

设备端工具
-----

在设备上可以启用几种“工具”或者模式，用来直接从屏幕获取有关应用和整个系统在运行时的信息。它们多数是从设置里面的开发者选项中得到。

### StrictMode

[Strict mode](http://developer.android.com/reference/android/os/StrictMode.html) 是通过开发者选项的“启用严格模式”来启用。在违反严格模式的情况下会在屏幕的边缘闪现红色，比如在UI线程中做了过多的工作。关于违反严格模式的信息会输出到log中。

### GPU呈现模式分析（Profile GPU rendering）

这个工具通过开发者选项的“GPU呈现模式分析”启用。它可以显示渲染每帧图像所需要的时间，以及渲染的四个阶段花了多长时间：创建DisplayList（processing of drawing commands），将渲染内容同步到渲染线程（通常包含了位图纹理的上传），发送DisplayList（ative processing of rendering commands into OpenGL commands and parameters and uploading them to the GPU），以及交换缓冲区(which encompasses both the time to return a buffer and the time spent waiting for pending GPU commands to finish processing)。

### 调试GPU过度绘制

这个工具是在开发者选项的“调试GPU过度绘制”中开启的。显示屏幕上的每一个像素在当前帧被绘制的次数，从而让你了解你的应用因为在同一区域绘制了太多次数而对GPU造成了多少开销（一般是因为view被其他不透明View挡住而造成的）。

一定程度的过度绘制时正常的，比如，文字总是会重绘在下面的背景之上，阴影或者其他半透明的元素总是会重绘在它们后面的东西之上。但是大量不透明区域的过度绘制久值得探讨了。

### 动画程序时长缩放（Animator duration scale）

这个工具是在开发者选项的“动画程序时长缩放”中开启的。它可以增加或者减少动画的持续时间。这在我们调试动画的时候很有用，通过它，你可以看到在默认动画设置下看不到的东西。注意开发者选项中还有两个类似的时长缩放设置:窗口动画缩放和过渡动画缩放。这两个缩放控制不同的窗口和系统动画，而“动画程序时长缩放”专门控制应用中使用了Animator object的动画。

这个工具是通过adb shell 命令（比如‘adb shell screenrecord /sdcard/myscreenrecord.mp4’）执行的。用于录制设备的帧图像并以mpeg 视频的形式存储这些帧。用Ctrl-C 停止，然后运行‘adb pull’可以将这个文件放到电脑主机上。这个工具对于动画的调试以及其他难以做到实时分析的交互场景时非常有用的。

注：现在android studio也自带了屏幕录制功能。

### 显示硬件层更新（Show hardware layer updates）

这个工具是在开发者选项的“显示硬件层更新”中开启的。当一个[layer](http://developer.android.com/reference/android/view/View.html#setLayerType%28int,%20android.graphics.Paint%29)更新的时候将在屏幕闪现绿色。硬件层可以在view的动画进行时提高那些没有发生改变的view的性能。但是如果被渲染为一个层的同时，在view层或者view的子树中发生了改变，将导致严重的性能损失。在开发者选项中启用这个工具可以让你看到是否有不期望的硬件层更新。