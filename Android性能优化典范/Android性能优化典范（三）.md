---
title: Android性能优化典范（三）
date: 2015-09-15 16:28:17
tags: Android性能优化典范
categories: 技术
---

[Android性能优化典范](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)的课程最近更新到第三季了，这次一共12个短视频课程，包括的内容大致有：更高效的ArrayMap容器，使用Android系统提供的特殊容器来避免自动装箱，避免使用枚举类型，注意onLowMemory与onTrimMemory的回调，避免内存泄漏，高效的位置更新操作，重复layout操作的性能影响，以及使用Batching，Prefetching优化网络请求，压缩传输数据等等使用技巧。下面是对这些课程的总结摘要，认知有限，理解偏差的地方请多多交流指正！

![](resources/E7C1236FA835601C32BF7E628221652E.jpg)

**1) Fun with ArrayMaps**
-------------------------
<!--more-->
程序内存的管理是否合理高效对应用的性能有着很大的影响，有的时候对容器的使用不当也会导致内存管理效率低下。Android为移动操作系统特意编写了一些更加高效的容器，例如SparseArray，今天要介绍的是一个新的容器，叫做 [ArrayMap](https://android.googlesource.com/platform/frameworks/base.git/+/master/core/java/android/util/ArrayMap.java)。

我们经常会使用到HashMap这个容器，它非常好用，但是却很占用内存。下图演示了HashMap的简要工作原理：

![](resources/F29E41FC2EA45D65769B8AEFDD22A155.jpg)

为了解决HashMap更占内存的弊端，Android提供了内存效率更高的ArrayMap。它内部使用两个数组进行工作，其中一个数组记录key hash过后的顺序列表，另外一个数组按key的顺序记录Key-Value值，如下图所示：

![](resources/42DB6E22C92F20118AA30D62CFDC243B.jpg)

当你想获取某个value的时候，ArrayMap会计算输入key转换过后的hash值，然后对hash数组使用二分查找法寻找到对应的index，然后我们可以通过这个index在另外一个数组中直接访问到需要的键值对。如果在第二个数组键值对中的key和前面输入的查询key不一致，那么就认为是发生了碰撞冲突。为了解决这个问题，我们会以该key为中心点，分别上下展开，逐个去对比查找，直到找到匹配的值。如下图所示：

![](resources/B749F90BA08411B98770B1F902992945.jpg)

随着数组中的对象越来越多，查找访问单个对象的花费也会跟着增长，这是在内存占用与访问时间之间做权衡交换。

既然ArrayMap中的内存占用是连续不间断的，那么它是如何处理插入与删除操作的呢？请看下图所示，演示了Array的特性：

![](resources/0BD3BFC1B10AEB182156AFC77AADB2CE.jpg)

![](resources/451CEF2B65FB18B8A5D71C50974659A8.jpg)

很明显，ArrayMap的插入与删除的效率是不够高的，但是如果数组的列表只是在一百这个数量级上，则完全不用担心这些插入与删除的效率问题。HashMap与ArrayMap之间的内存占用效率对比图如下：

![](resources/2AE0AE9F3F50AF084A80714D0513339C.jpg)

与HashMap相比，ArrayMap在循环遍历的时候也更加简单高效，如下图所示：

![](resources/AD0D923F15F432B92AA5488B501F198A.jpg)

前面演示了很多ArrayMap的优点，但并不是所有情况下都适合使用ArrayMap，我们应该在满足下面2个条件的时候才考虑使用ArrayMap：

* 对象个数的数量级最好是千以内；
* 数据组织形式包含Map结构。

我们需要学会在特定情形下选择相对更加高效的实现方式。

2) Beware Autoboxing
--------------------

有时候性能问题也可能是因为那些不起眼的小细节引起的，例如在代码中不经意的“自动装箱”。我们知道基础数据类型的大小：boolean(8 bits), int(32 bits), float(32 bits)，long(64 bits)，为了能够让这些基础数据类型在大多数Java容器中运作，会需要做一个autoboxing的操作，转换成Boolean，Integer，Float等对象，如下演示了循环操作的时候是否发生autoboxing行为的差异：

![](resources/6E526A32AEB20E7BF4AE81D6D6C02126.jpg)

![](resources/76AB63A7AEE5B0007EC558FA077C25F6.jpg)

Autoboxing的行为还经常发生在类似HashMap这样的容器里面，对HashMap的增删改查操作都会发生了大量的autoboxing的行为。

![](resources/4B99F65F156D80B655ACAA76DA558852.jpg)

为了避免这些autoboxing带来的效率问题，Android特地提供了一些如下的Map容器用来替代HashMap，不仅避免了autoboxing，还减少了内存占用：

![](resources/8C5D561C8A3BF3479E33BFCE9D76DE23.jpg)

3) SparseArray Family Ties
--------------------------

为了避免HashMap的autoboxing行为，Android系统提供了SparseBoolMap，SparseIntMap，SparseLongMap，LongSparseMap等容器。关于这些容器的基本原理请参考前面的ArrayMap的介绍，另外这些容器的使用场景也和ArrayMap一致，需要满足数量级在千以内，数据组织形式需要包含Map结构。

4) The price of ENUMs
---------------------

在StackOverFlow等问答社区常常出现关于在Android系统里面使用枚举类型的性能讨论，关于这一点，Android官方的Training课程里面有下面这样一句话：

> Enums often require more than twice as much memory as static constants. You should strictly avoid using enums on Android.

![](resources/594E44A9750EB5C5B619904E779972E9.jpg)

关于enum的效率，请看下面的讨论。假设我们有这样一份代码，编译之后的dex大小是2556 bytes，在此基础之上，添加一些如下代码，这些代码使用普通static常量相关作为判断值：

![](resources/D36B2DD4DE533313EEC18D99FD427FF3.jpg)

增加上面那段代码之后，编译成dex的大小是2680 bytes，相比起之前的2556 bytes只增加124 bytes。假如换做使用enum，情况如下：

![](resources/65D3CAA3C496D3D19A077CFCDA93F42D.jpg)

使用enum之后的dex大小是4188 bytes，相比起2556增加了1632 bytes，增长量是使用static int的13倍。不仅仅如此，使用enum，运行时还会产生额外的内存占用，如下图所示：

![](resources/2FCA1FD92B72241C1237172C26004E0E.jpg)

Android官方强烈建议不要在Android程序里面使用到enum。

5) Trimming and Sharing Memory
------------------------------

Android系统的一大特色是多任务，用户可以随意在不同的app之间进行快速切换。为了确保你的应用在这种复杂的多任务环境中正常运行，我们需要了解下面的知识。

为了让background的应用能够迅速的切换到forground，每一个background的应用都会占用一定的内存。Android系统会根据当前的系统内存使用情况，决定回收部分background的应用内存。如果background的应用从暂停状态直接被恢复到forground，能够获得较快的恢复体验，如果background应用是从Kill的状态进行恢复，就会显得稍微有点慢。

![](resources/DCD52E7E5D8EE95FCA70F12DE46AFFF5.jpg)

Android系统提供了一些回调来通知应用的内存使用情况，通常来说，当所有的background应用都被kill掉的时候，forground应用会收到onLowMemory()的回调。在这种情况下，需要尽快释放当前应用的非必须内存资源，从而确保系统能够稳定继续运行。Android系统还提供了onTrimMemory()的回调，当系统内存达到某些条件的时候，所有正在运行的应用都会收到这个回调，同时在这个回调里面会传递以下的参数，代表不同的内存使用情况，下图介绍了各种不同的回调参数：

![](resources/C9F5B4B5C46678F2C7FC85FE7430B195.jpg)

关于每个参数的更多介绍，请参考《 [Android Training - 管理应用的内存](http://hukai.me/android-training-managing_your_app_memory/)》，另外onTrimMemory()的回调可以发生在Application，Activity，Fragment，Service，Content Provider。

从Android 4.4开始，ActivityManager提供了isLowRamDevice()的API，通常指的是Heap Size低于512M或者屏幕大小\<=800\*480的设备。

6) DO NOT LEAK VIEWS
--------------------

内存泄漏的概念，下面一张图演示下：

![](resources/15F5750E255B7BC3D343228FC18D25E5.jpg)

通常来说，View会保持Activity的引用，Activity同时还和其他内部对象也有可能保持引用关系。当屏幕发生旋转的时候，activity很容易发生泄漏，这样的话，里面的view也会发生泄漏。Activity以及view的泄漏是非常严重的，为了避免出现泄漏，请特别留意以下的规则：

### 6.1) 避免使用异步回调

异步回调被执行的时间不确定，很有可能发生在activity已经被销毁之后，这不仅仅很容易引起crash，还很容易发生内存泄露。

![](resources/4C8D7895BCC82144FEB4579B541CDD3D.jpg)

### 6.2) 避免使用Static对象

因为static的生命周期过长，使用不当很可能导致leak，在Android中应该尽量避免使用static对象。

![](resources/B5BC4E3B0E354EF3862730C63F910180.jpg)

### 6.3) 避免把View添加到没有清除机制的容器里面

假如把view添加到 [WeekHashMap](http://stackoverflow.com/questions/5511279/what-is-a-weakhashmap-and-when-to-use-it)，如果没有执行清除操作，很可能会导致泄漏。

![](resources/183E8DD7D4CBF2749C535CD86B1F9867.jpg)

7) Location & Battery Drain
---------------------------

开启定位功能是一个相对来说比较耗电的操作，通常来说，我们会使用类似下面这样的代码来发出定位请求：

![](resources/665F39DC952280C524AC498CB633C1C2.jpg)

上面演示中有一个方法是setInterval()指的意思是每隔多长的时间获取一次位置更新，时间相隔越短，自然花费的电量就越多，但是时间相隔太长，又无法及时获取到更新的位置信息。其中存在的一个优化点是，我们可以通过判断返回的位置信息是否相同，从而决定设置下次的更新间隔是否增加一倍，通过这种方式可以减少电量的消耗，如下图所示：

![](resources/AD58592B8F7481AEB82B5B471ECFA60E.jpg)

在位置请求的演示代码中还有一个方法是setFastestInterval()，因为整个系统中很可能存在其他的应用也在请求位置更新，那些应用很有可能设置的更新间隔时间很短，这种情况下，我们就可以通过setFestestInterval的方法来过滤那些过于频繁的更新。

通过GPS定位服务相比起使用网络进行定位更加的耗电，但是也相对更加精准一些，他们的图示关系如下：

![](resources/002D35D75FC1DB8DD7D65C22AD93DDB6.jpg)

为了提供不同精度的定位需求，同时屏蔽实现位置请求的细节，Android提供了下面4种不同精度与耗电量的参数给应用进行设置调用，应用只需要决定在适当的场景下使用对应的参数就好了，通过LocationRequest.setPriority()方法传递下面的参数就好了。

![](resources/4F93493443349E1BD5BEE3FE86ECD1A2.jpg)

8) Double Layout Taxation
-------------------------

布局中的任何一个View一旦发生一些属性变化，都可能引起很大的连锁反应。例如某个button的大小突然增加一倍，有可能会导致兄弟视图的位置变化，也有可能导致父视图的大小发生改变。当大量的layout()操作被频繁调用执行的时候，就很可能引起丢帧的现象。

![](resources/DF396BCA5677021C6B054AEADD0B6822.jpg)

例如，在RelativeLayout中，我们通常会定义一些类似alignTop，alignBelow等等属性，如图所示：

![](resources/78B3F987AFF7FB2CE8C45DAC041C2593.jpg)

为了获得视图的准确位置，需要经过下面几个阶段。首先子视图会触发计算自身位置的操作，然后RelativeLayout使用前面计算出来的位置信息做边界的调整的操作，如下面两张图所示：

![](resources/F4886106FB91107E133B978A7040EF3A.jpg)

![](resources/DF6D67803BBD48405F55D6277BDA1A0F.jpg)

经历过上面2个步骤，relativeLayout会立即触发第二次layout()的操作来确定所有子视图的最终位置与大小信息。

除了RelativeLayout会发生两次layout操作之外，LinearLayout也有可能触发两次layout操作，通常情况下LinearLayout只会发生一次layout操作，可是一旦调用了measureWithLargetChild()方法就会导致触发两次layout的操作。另外，通常来说，GridLayout会自动预处理子视图的关系来避免两次layout，可是如果GridLayout里面的某些子视图使用了weight等复杂的属性，还是会导致重复的layout操作。

如果只是少量的重复layout本身并不会引起严重的性能问题，但是如果它们发生在布局的根节点，或者是ListView里面的某个ListItem，这样就会引起比较严重的性能问题。如下图所示：

![](resources/18B002260D413C5DBD82D151DC6079C3.jpg)

我们可以使用Systrace来跟踪特定的某段操作，如果发现了疑似丢帧的现象，可能就是因为重复layout引起的。通常我们无法避免重复layout，在这种情况下，我们应该尽量保持View Hierarchy的层级比较浅，这样即使发生重复layout，也不会因为布局的层级比较深而增大了重复layout的倍数。另外还有一点需要特别注意，在任何时候都请避免调用requestLayout()的方法，因为一旦调用了requestLayout，会导致该layout的所有父节点都发生重新layout的操作。

![](resources/B5FB699E1F38B5363A4C06E14F4CE50A.jpg)

9) Network Performance 101
--------------------------

在性能优化[第一季](http://www.csdn.net/article/2015-01-20/2823621-android-performance-patterns)与[第二季](http://www.csdn.net/article/2015-04-29/2824583-android-performance-patterns-season-2)的课程里面都介绍过，网络请求的操作是非常耗电的，其中在移动蜂窝网络情况下执行网络数据的请求则尤其比较耗电。关于如何减少移动网络下的网络请求的耗电量，有两个重要的原则需要遵守：第一个是减少移动网络被激活的时间与次数，第二个是压缩传输数据。

### 9.1) 减少移动网络被激活的时间与次数

通常来说，发生网络行为可以划分为如下图所示的三种类型，一个是用户主动触发的请求，另外被动接收服务器的返回数据，最后一个是数据上报，行为上报，位置更新等等自定义的后台操作。

![](resources/AA2C6A76E1EA2AB18D96FF14318ED99A.jpg)

我们绝对坚决肯定不应该使用Polling(轮询)的方式去执行网络请求，这样不仅仅会造成严重的电量消耗，还会浪费许多网络流量，例如：

![](resources/A999F0D0AD8813B266A61C382EDA7F92.jpg)

Android官方推荐使用Google Cloud Messaging(在大陆，然并卵)，这个框架会帮助把更新的数据推送给手机客户端，效率极高！我们应该遵循下面的规则来处理数据同步的问题：

首先，我们应该使用回退机制来避免固定频繁的同步请求，例如，在发现返回数据相同的情况下，推迟下次的请求时间，如下图所示：

![](resources/8C4D435D920E3570302BFBCF0738EA3D.jpg)

其次，我们还可以使用Batching(批处理)的方式来集中发出请求，避免频繁的间隔请求，如下图所示：

![](resources/B2452C38BF5DE2EADDC32D48F5A3572B.jpg)

最后，我们还可以使用Prefetching（预取）的技术提前把一些数据拿到，避免后面频繁再次发起网络请求，如下图所示：

![](resources/45B3545D808A8B4B2E9B6B8C6BA0DF76.jpg)

Google Play Service中提供了一个叫做 [GCMNetworkManager](https://developers.google.com/cloud-messaging/network-manager)的类来帮助我们实现上面的那些功能，我们只需要调用对应的API，设置一些简单的参数，其余的工作就都交给Google来帮我们实现了。

![](resources/3D5636BFE235C3958AF6E9BE49F2E6A4.jpg)

### 9.2) 压缩传输数据

关于压缩传输数据，我们可以学习以下的一些课程（真的够喝好几壶了）：

10) Effective Network Batching
------------------------------

在性能优化课程的[第一季](http://www.csdn.net/article/2015-01-20/2823621-android-performance-patterns)与[第二季](http://www.csdn.net/article/2015-04-29/2824583-android-performance-patterns-season-2)里面，我们都有提到过下面这样一个网络请求与电量消耗的示意图：

![](resources/4776F1ED699D53599B216A0B2B4315A8.jpg)

发起网络请求与接收返回数据都是比较耗电的，在网络硬件模块被激活之后，会继续保持几十秒的电量消耗，直到没有新的网络操作行为之后，才会进入休眠状态。前面一个段落介绍了使用Batching的技术来捆绑网络请求，从而达到减少网络请求的频率。那么如何实现Batching技术呢？通常来说，我们可以会把那些发出的网络请求，先暂存到一个PendingQueue里面，等到条件合适的时候再触发Queue里面的网络请求。

![](resources/929231FBA36D48FE55ED51C1336090DA.jpg)

可是什么时候才算是条件合适了呢？最简单粗暴的，例如我们可以在Queue大小到10的时候触发任务，也可以是当手机开始充电，或者是手机连接到WiFi等情况下才触发队列中的任务。手动编写代码去实现这些功能会比较复杂繁琐，Google为了解决这个问题，为我们提供了GCMNetworkManager来帮助实现那些功能，仅仅只需要调用API，设置触发条件，然后就OK了。

11) Optimizing Network Request Frequencies
------------------------------------------

前面的段落已经提到了应该减少网络请求的频率，这是为了减少电量的消耗。我们可以使用Batching，Prefetching的技术来避免频繁的网络请求。Google提供了GCMNetworkManager来帮助开发者实现那些功能，通过提供的API，我们可以选择在接入WiFi，开始充电，等待移动网络被激活等条件下再次激活网络请求。

12) Effective Prefetching
-------------------------

假设我们有这样的一个场景，最开始网络请求了一张图片，隔了10秒需要请求另外一张图片，再隔6秒会请求第三张图片，如下图所示：

![](resources/A155E9AF651592ACFB5DAEA47D68751F.jpg)

类似上面的情况会频繁触发网络请求，但是如果我们能够预先请求后续可能会使用到网络资源，避免频繁的触发网络请求，这样就能够显著的减少电量的消耗。可是预先获取多少数据量是很值得考量的，因为如果预取数据量偏少，就起不到减少频繁请求的作用，可是如果预取数据过多，就会造成资源的浪费。

![](resources/F4409A21F514BF5FD077C7D7EAE78961.jpg)

我们可以参考在WiFi，4G，3G等不同的网络下设计不同大小的预取数据量，也可以是按照图片数量或者操作时间来作为阀值。这需要我们需要根据特定的场景，不同的网络情况设计合适的方案。