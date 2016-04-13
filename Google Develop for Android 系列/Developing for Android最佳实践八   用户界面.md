---
title: Google Develop for Android 系列八 － 用户界面
date: 2015-06-15 10:12:17
tags: Google Develop for Android 系列
categories: 技术
---


原文：[Developing for Android VIII The Rules: User Interface](https://medium.com/google-developers/developing-for-android-viii-e91ced595fac)

这节主要涉及UI开发最佳范例的一些重要细节，着重围绕性能和用户体验进行讲解。

避免过度绘制
------

正如我们在第一章（[理解移动应用上下文](https://medium.com/google-developers/developing-for-android-i-understanding-the-mobile-context-fd2351b131f8) ）的GPU一节中所讨论的，很多设备的GPU填充率都是受限的，因此，那些具有严重过度绘制问题的应用都将面临的渲染性能问题。要避免不透明view完全遮住其它view的情况，这种情况下不可见的UI也在做绘制的操作，就会导致某个像素在同一帧的时间内被绘制了多次。检测你的应用是否过渡绘制的办法是在系统设置的开发者选项中开启“*调试GPU过度绘制*”选项，然后根据情况修复问题。

注：1.GPU的填充率分为像素填充率和纹理填充率。2.关于过度绘制这一点，其实在 [这篇文章](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0416/2735.html) 的GPU Problem: Overdraw一节中说的最详细。

不要设置窗口背景为空
----------

避免过度绘制的技巧之一就是在所有View都不透明的情况下去除窗口的背景。反正如果有一个或者多个不透明的View完全挡住了窗口，用户怎么也看不到，还不如干脆不要背景。

去除窗口背景是一个有效的方法，但也往往是一个解决过度绘制问题的复杂办法，而且它经常在某些情况下导致渲染的artifacts（不知道artifacts啥意思）。虽然可以在app的manifest中设置一个空的窗口背景，但是因为系统无法正确绘制启动窗口，这会导致图形绘制artifacts。更正确的做法是保留manifest中启动窗口的背景，转而在activity的onCreate()方法中通过调用getWindow().setBackground(null)设置成null。但即便是这种方法也会导致artifacts。比如，如果keyboard/IME设置成了adjustResize，然后动画进入一个背景为null的activity，在键盘后面可能存在artifacts，因为window manager没有要绘制的背景。还有，全屏且带有overscroll bounce gaps的ListView也可能会有artifacts。

。。。。受不了了 artifacts artifacts artifacts不停。
<!--more-->
正确解决这种情况下过度绘制问题的方法恰恰是使用启动窗口。不要再窗口与view之间的容器中使用不透明背景颜色，而是将这个颜色设置给主题里的windowBackground属性，而容器则保持透明背景。

注：实在觉得这段优点多余，如果要避免过度绘制，我们肯定不会在根view上设置不必要的背景颜色，想都不会想到要去掉窗口的背景颜色。

不要禁用启动窗口（Starting Window）
-------------------------

正如我们在[Performance](https://medium.com/google-developers/developing-for-android-iii-2efc140167fd)一章的快速启动一节中所讨论的那样，系统为了达到快速启动的用户体验，在应用还在后台加载的时候就调用了启动窗口。可以告在主题设置*windowDisablePreview*属性，告诉系统不要这样做，

\<item name="android:windowDisablePreview"\>\</item\>。

有些应用为了自定义启动屏幕的或者达到其他新颖的用户体验，或者因为启动窗口的样式比如色彩和启动以后的内容差别太大，的确会设置windowDisablePreview为true。但是这样做的问题是这些方法要花费更长的时间（因为初始化与显示内容所花费的时间比系统启动窗口要长的多），在activity启动之前用户必须被迫接受毫无提示的等待。

为了让用户感受到最好的用户体验，请保持系统对启动窗口的默认设置。如果你需要自定义，可以利用启动窗口将继承activity的主题这一特点，这里可以更正确的调整activity的外观。并且如果你想要全新的、自定义的启动体验，你还可以为启动窗口指定一个自定义的drawable（通过主题的*windowBackground*属性设置）。 

确保可以轻易退出沉浸模式
------------

4.4之后，可以为应用开启沉浸式的全屏模式，用户想退出全屏模式必须滑动屏幕的上下边缘才能显示导航栏与状态栏。这只适用于沉浸式的游戏，用户点击屏幕（不小心点到虚拟键）不至于退出游戏。其他app，尤其是以内容阅读为主的app，比如播放器，应该让用户点击屏幕就能退出，而不是滑动上下栏。关于如何正确使用沉浸模式，详情见[沉浸模式开发者指南](https://developer.android.com/training/system-ui/immersive.html)。

在启动窗口中设置正确的状态栏导航栏颜色
-------------------

如果你的应用有带颜色的状态栏、导航栏，那么你的主题（在app启动的时候被启动窗口使用）也应该有相同的颜色。这样可以避免在启动窗口到内容窗口过渡的时候出现让人不愉快的闪现。分别设置android:statusBarColor 和 android:navigationBarColor来设置启动窗口的状态栏、导航栏颜色。如果你需要不同颜色的activity窗口，在onCreate中通过getWindow().setStatusBarColor() 和 getWindow.setNavigationBarColor()来改变。

使用正确的Context
------------

可以方便的获取到Application的Context，但是使用这个Context去创建UI是不正确的，因为这个Context没有携带正确的主题信息。我们应该使用Activity的Context。比如想取得Activity的资源就应该通过Activity context，而不是Application Context。

### 不要在异步回调中引用View相关的对象

不要在网络操作或者其他耗时的异步事务中引用一个View，Activity或者Fragment。不如下面的代码：

`// ...`

`// some activity code`

`// `

`void onClick(View view) {`

`webservice.fetchPosts(``new` `Callback() {`

`public void onResult(Response repsonse) {`

`// View可能不再有效，Activity可能已经消失`

`}`

`});`

`}`

这里对view的引用可能导致Context（它可能是被View间接引用的）的泄漏或者崩溃，因为在回调方法被触发的时候我们引用了也许不再有效的对象。正确的做法是使用eventbus或者弱引用的回调，同时小心谨慎的处理attachment 和 detachment情形。

兼容从右到左的设计
---------

自API 17开始，布局的某些属性要使用start/end变量而不是left/right变量。比如涉及到内边距的时候，应该使用paddingStart和paddingEnd，而不是paddingLeft和paddingRight，这种新规则可以让应用在从右到左的语言环境下正确显示布局。

可以在开发者选项里启用“强制RTL布局方向”来检测你的app是否表现正常。

注：据我所知，阿拉伯语就是从右到左，在这个东西出来之前，要做到国际化还得专门为阿拉伯语写布局。其实这个对与中文应用来说无所谓，所以绝大多数开发者都可以无视。

但是这也解惑了我之前对于drawerlayout那些属性的疑问。

将数据缓存到本地
--------

“本地缓存，全局同步”

把数据缓存到本地是让用户感觉你app运行如飞的重要手段之一。除了缓存诸如图片之类的大文件之外，还应该缓存从服务器上获得的数据。如果可能，你还应该用建立本地数据库这种更合理的方式来保存数据（不仅仅是作为服务器响应的序列化数据）。

只要有可能，都应该使用本地数据来更新UI。这有助于保持数据的一致性，同时，因为app对网络的依赖降低，可以大大提高用户体验。

在2010年的谷歌I／O大会上有关于这个话题的讨论：[安卓的 REST 客户端程序](https://www.youtube.com/watch?v=xHXn3Kg2IQE)。现在有了比当初更好的组建来实现这个方法(ORM, JobQueue, EventBus)，但是那次讨论的架构和方法仍然是有效的。

缓存用户的输入数据
---------

如果有用户输入要传递到服务器，在传递之前优先缓存输入。因为网络通信，这样做的主要目的是避免用户输入数据的丢失，因为如果没有缓存该数据，在网络连接失败的时候会导致让人发狂的丢失。缓存输入数据可以让你的app在网络可用的时候重新发送数据。

同时，你应该在UI上有帮助用户区分完成与未完成输入的提示

将网络后台任务和磁盘后台任务分开
----------------

如果使用一个线程池来应付所有的后台操作，一些执行很快的任务可能会因为网络请求这样的超长操作而被阻碍。可以考虑针对快速的后台任务使用单独的线程池，比如像本地存储这类操作。这可以保证本地数据的改变可以及时的反应到UI上，而不会受昂贵的网络请求或者进程间通信的影响。