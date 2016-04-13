---
title: EventBus 3.0的用法详解（二）
date: 2015-06-27 23:59:14
tags: EventBus
categories: 技术
---



前一篇文章简单介绍了`EventBus 3.0`的用法，现在是时候详解其用法了。首先声明，`EventBus 3.0`的改动针对2.4的改动并不是特别大，但是对于其性能的提升是另外一个说法了，所以建议学习EventBus 3.0。<!--more-->

#### 注解
##### 新增的``@Subscribe`

    threadMode = ThreadMode.MainThread

用注解的方式代替约定的方法名规范，是其最大的改变。在2.4中，你可能需要这么定义：

    public void onEventMainThread(MessageEvent event) {
            log(event.message);
        }

该方法为接收消息后在主线程中处理事件，而在3.0中：

    @Subscribe(threadMode = ThreadMode.MainThread) //在ui线程执行
    public void onUserEvent(UserEvent event) {
            log(event.message);
        }

其中ThreadMode提供了四个常量：

* MainThread 主线程
* BackgroundThread 后台线程
* Async 后台线程
* PostThread 发送线程（默认）

`BackgroundThread`:当事件是在UI线程发出，那么事件处理实际上是需要新建单独线程，如果是在后台线程发出，那么事件处理就在该线程。该事件处理方法应该是快速的，避免阻塞后台线程。

Async：发送事件方不需要等待事件处理完毕。这种方式适用于该事件处理方法需要较长时间，例如网络请求。

    sticky = true

##### 默认情况下，其为false。什么情况下使用sticky呢？

相信大多数使用过`EventBus 2.4`的同学或多或少的使用过：

    EventBus.getDefault().postSticky(new VoteEvent(obj));
    EventBus.getDefault().registerSticky(this);

你会发现非常的麻烦，那么在3.0中：

    EventBus.getDefault().postSticky(new VoteEvent(obj));
    EventBus.getDefault().register(this);
    @Subscribe(sticky = true)

什么时候使用`sticy`,当你希望你的事件不被马上处理的时候，举个栗子，比如说，在一个详情页点赞之后，产生一个`VoteEvent`，`VoteEvent`并不立即被消费，而是等用户退出详情页回到商品列表之后，接收到该事件，然后刷新`Adapter`等。其实这就是之前我们用`startActivityForResult`和`onActivityResult`做的事情。

  priority = 1

相信大部分人知道该用法，值越小优先级越低，默认为0。

##### 建议
推荐大家在使用EventBus的时候，创建一个事件类，把你的每一个参数（或者可能发生冲突的参数），封装成一个类：

    public class Event  {  
        public static class UserListEvent {  
            public List<User> users ;  
        }
        public static class ItemListEvent {  
            public List<Item> items;  
        }    
    }  

##### 添加processor
按照Markus Junginger的说法（EventBus创作者），在3.0中，如果你想进一步提升你的app的性能，你需要添加：

    provided 'de.greenrobot:eventbus-annotation-processor:3.0.0-beta1'

其在编译的时候为注册类构建了一个索引，而不是在运行时，这样的结果是其让EventBus 3.0的性能提升了一倍，相比2.4来说，其会是它的3到6倍。大家可以感受下：
![](https://segmentfault.com/img/bVsgvf)
