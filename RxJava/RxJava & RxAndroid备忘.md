---
title: RxJava & RxAndroid备忘
date: 2016-04-06 01:14:17
tags: RxJava
categories: 技术
---


    来源 <http://chenqichao.me/2015/07/01/119-Mastering-RxAndroid/> 

今天在刷G+的时候看到Dave Smith推荐了一个视频[《Learning RxJava (for Android) by example》](https://www.youtube.com/watch?v=k3D0cWyNno4)点进去看了一下，原来是位熟悉的”阿三哥”，视频封面如下：（没有歧视的意思，不要喷我啊~，为什么感到熟悉？接着往下看）

[](http://7mnllc.com1.z0.glb.clouddn.com/kp.png)

[![](http://7mnllc.com1.z0.glb.clouddn.com/kp.png)](http://7mnllc.com1.z0.glb.clouddn.com/kp.png)

几乎同时也看到了JetBrains在G+也推荐了篇在Medium上的博文[《RxAndroid And Kotlin (Part 1)》](https://medium.com/@ahmedrizwan/rxandroid-and-kotlin-part-1-f0382dc26ed8)，然后想到前几天转了InfoQ上的[《Kotlin：Android世界的Swift》](http://www.infoq.com/cn/news/2015/06/Android-JVM-JetBrains-Kotlin)，再加上隐约记得之前在AndroidCN看到过[@hi大头鬼hi](http://weibo.com/brucefromsdu)写的《深入浅出RxJava》，最后还想到了这篇[《Kotlin在Android工程中的应用》](http://www.jianshu.com/p/a7fadc79e0fb)，Holy shit…大脑能瞬间闪过这么多关联的文章和博文，于是把这些资料找了出来，觉得有必要把这些内容记下来，
方便日后查阅，因此有了今天这篇文章，取名叫《RxJava & RxAndroid备忘》是希望列出的参考资料能让大家尽快熟悉和掌握了RxJava和RxAndroid。

等等..还没解释为什么对这位阿三哥的声音感到熟悉呢? 其实是因为之前听过也推荐过Kaushik Gopal和他的小伙伴Donn Felker录制的关于Android开发的Podcast《FragmentedPodcast》，每一集都很精彩（目前更新到第十期），感兴趣的可以关注他们。唯一的需要克服的就是三哥的英语口音…另外，真心觉得这种类型的Podcast很不错，类似还有官方团队Chet和Tor录制的《Android Backstage》，虽然国内也有类似《内核恐慌》的技术播客，但只是针对Android或者iOS的目前并没有发现（如果你有推荐可以直接评论或者联系我），再者就是希望以后开始工作了可以找到同样感兴趣的人，可以一起来做这样有趣的事情。
<!--more-->
[](http://7mnllc.com1.z0.glb.clouddn.com/fragmented-logo.png)

[![](http://7mnllc.com1.z0.glb.clouddn.com/fragmented-logo.png)](http://7mnllc.com1.z0.glb.clouddn.com/fragmented-logo.png)

首先需要明确一个观点：Rx并不是一种新的语言，而是一种普通的Java模式，类似于观察者模式（Observer Pattern），可以将它看作一个普通的Java类库，因此你可以立即使用RxJava。而RxAndroid是RxJava的一个针对Android平台的扩展，主要用于 Android 开发。《深入浅出RxJava》系列的四篇文章已经非常详细的介绍了Rx的相关内容，所以建议大家直接可以先点进去仔细阅读一下~，

[](http://7mnllc.com1.z0.glb.clouddn.com/RxJava-Android.jpg)

[![](http://7mnllc.com1.z0.glb.clouddn.com/RxJava-Android.jpg)](http://7mnllc.com1.z0.glb.clouddn.com/RxJava-Android.jpg)

下面列出参考链接（如你有需要补充的可以直接评论~）