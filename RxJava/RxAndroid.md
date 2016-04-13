---
title: 从案例学习RxAndroid
date: 2016-03-25 21:01:45
tags: RxAndroid
categories: 技术
---



<a herf="https://medium.com/@kurtisnusbaum/rxandroid-basics-part-1-c0d5edcf6850">原文链接</a>



如果你在阅读这篇文章，相信你一定很想了解RxJava以及如何在Android应用中使用它。可能你已经见过RxJava的代码了，但仍然有些疑惑，愿你能在这篇文章里找到答案。

当我第一次使用RxJava的时候我只是在照搬代码，这些代码能跑起来，但是我对RxJava的基础部分仍然存在误解，而且我找不到好的源码来学习。所以为了理解RxJava，我不得不一点一点学习，踩了不少坑。

为了不让你把我踩过的坑再踩一遍，我会基于我的学习成果写一些例子出来，目的就是让你能够对RxJava有足够的了解，并能在你的Android应用中使用它。

源码可以在这里找到。在每个例子的开始，我会写清每个代码段是属于哪个Activity的。我会将本文分为两个部分，在前三个例子里，我会着重讲解如何用RxJava异步加载数据；在后三个例子里，我会探索一些更高级的用法。

在开始说代码之前，先澄清几个概念。RxJava最核心的东西就是Observable和Observer。Observable会发出数据，而与之相对的Observer则会通过订阅Observable来进行观察。

Observer可以在Observable发出数据、报错或者声明没有数据可以发送时进行相应的操作。这三个操作被封装在Observer接口中，相应的方法为onNext()，onError()和onCompleted()。

明确了这些概念以后，让我们来看一些例子。
<!--more-->
案例1：基础

现在要写一个用来展示一个颜色列表的Activity。我们要写一个能发送一个字符串列表、然后结束的Observeable。而后我们会通过这个字符串列表来填充颜色列表，这里要使用到Observable.just()方法。由这个方法创建的Observable对象的特点是：所有Observer一旦订阅这个Observable就会立即调用onNext()方法并传入Observable.just()的参数，而后因为Observable没有数据可以发送了，onComplete()方法会被调用。

Observable<List<String>> listObservable = Observable.just(getColorList());


注意这里的getColorList()是一个不耗时的方法。虽然现在看来这个方法无足轻重，但一会我们会回到这个方法。

下一步，我们写一个Observer来观察Observable。

listObservable.subscribe(new Observer<List<String>>() {

    @Override
    public void onCompleted() { }

    @Override
    public void onError(Throwable e) { }

    @Override
    public void onNext(List<String> colors) {
        mSimpleStringAdapter.setStrings(colors);
    }
});


而后神奇的事情就发生了。如我刚才所说，一旦通过subscribe()方法订阅Observable，就会发生一系列事情：

- onNext()方法被调用，被发送的颜色列表会作为参数传入。

- 既然不再有数据可以发送（我们在Observable.just()中只让Observable发送一个数据），onComplete()方法会被调用。

请记住：通过Observable被订阅后的行为来区分它们。

在这个例子中我们不关心Observable何时完成数据的传输，所以我们不用在onComplete()方法里写代码。而且在这里不会有异常抛出，所以我们也不用管onError()方法。

写了这么多你可能觉得很多余，毕竟我们本可以在adapter中直接设置作为数据源的颜色列表。请带着这个疑问，和我看下面这个更有趣一些的例子。

案例2：异步加载

在这里我们要写一个显示电视剧列表的Activity。在Android中RxJava的主要用途就在于异步数据加载。首先让我们写一个Observable：

Observable<List<String>> tvShowObservable = Observable.fromCallable(new Callable<List<String>>() {

    @Override
    public List<String> call() {
        return mRestClient.getFavoriteTvShows();
    }
});


在刚才的例子中，我们使用Observable.just()来创建Observable，你可能认为在这里可以通过Observable.just(mRestClient.getFavoriteTvShows())来创建Observable。

但在这里我们不能这么做，因为mRestClient.getFavoriteTvShows()会发起网络请求。如果在这里我们使用Observable.just()，mRestClient.getFavoriteTvShows()会被立即执行并阻塞UI线程。

使用Observable.fromCallable()方法有两点好处：

- 获取要发送的数据的代码只会在有Observer订阅之后执行。

- 获取数据的代码可以在子线程中执行。

这两点好处有时可能非常重要。现在让我们订阅这个Observable。

mTvShowSubscription = tvShowObservable
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<List<String>>() {

        @Override
        public void onCompleted() { }

        @Override
        public void onError(Throwable e) { }

        @Override
        public void onNext(List<String> tvShows){
            displayTvShows(tvShows);
        }
    });


让我们一个方法一个方法地来看这段代码。subscribeOn会修改我们刚刚创建的Observable。在默认情况下Observable的所有代码，包括刚才说到的只有在被订阅之后才会执行的代码，都会在执行subscribe()方法的线程中运行。而通过subscribeOn()方法，这些代码可以在其他线程中执行。但具体是哪个线程呢？

在这个例子中我们让代码在”IO Scheduler”中执行（Schedulers.io()）。现在我们可以只把Scheduler当做一个可以工作的子线程，这个描述对于现在的我们已经足够了，不过这其中还有更深层次的内容。

不过我们的确遇到了一个小障碍。既然Observable会在IO Scheduler中运行，那么它与Observer的连接也会在IO Scheduler中完成。这就意味着Observer的onNext()方法也会在IO Scheduler中运行，而onNext()方法会操作UI中的View，但View只能在UI主线程中操作。

事实上解决这个问题也很简单，我们可以告诉RxJava我们要在UI线程中观察这个Observable，也就是，我们想让onNext()方法在UI线程中执行。这一点我们可以通过在observeOn()方法中指定另一个Scheduler来完成，在这里也就是AndroidSchedules.mainThread()所返回的Scheduler(UI线程的Scheduler)。

而后我们调用subscribe()方法。这个方法最重要，因为Callable只会在有Observer订阅后运行。还记得刚才我说Observable通过其被订阅后的行为来区分吗？这就是一个很好的例子。

还有最后一件事。这个mTvShowSubscription到底是什么？每当Observer订阅Observable时就会生成一个Subscription对象。一个Subscription代表了一个Observer与Observable之间的连接。有时我们需要操作这个连接，这里拿在Activity的onDestroy()方法中的代码举个例子：

if (mTvShowSubscription != null && !mTvShowSubscription.isUnsubscribed()) {
    mTvShowSubscription.unsubscribe();
}


如果你与多线程打过交道，你肯定会意识到一个大坑：当Activity执行onDestroy()后线程才结束（甚至永不结束）的话，就有可能发生内存泄漏与NullPointerException空指针异常。

Subscription就可以解决这个问题，我们可以通过调用unsubscribe()方法告诉Observable它所发送的数据不再被Observer所接收。在调用unsubscribe()方法后，我们创建的Observer就不再会收到数据了，同时也就解决了刚才说的问题。

说到这里难点已经过去，让我们来总结一下：

- Observable.fromCallable()方法可以拖延Observable获取数据的操作，这一点在数据需要在其他线程获取时尤其重要。

- subscribeOn()让我们在指定线程中运行获取数据的代码，只要不是UI线程就行。

- observeOn()让我们在合适的线程中接收Observable发送的数据，在这里是UI主线程。

- 记住要让Observer取消订阅以免Observable异步加载数据时发生意外。

案例3：使用Single

这次我们还是写一个展示电视剧列表的Activity，但这次我们走一种更简单的风格。Observable挺好用的，但在某些情况下过于重量级。比如说，你可能一经发现在过去的两个方法中我们只是让Observable发送一个数据，而且我们从来也没写过onComplete()回调方法。

其实呢，Observable还有一个精简版，叫做Single。Single几乎和Observable一模一样，但其回调方法不是onComplete()/onNext()/onError()，而是onSuccess()/onError()。

我们现在把刚才写过的Observable用Single重写一遍。首先我们要创建一个Single:

Single<List<String>> tvShowSingle = Single.fromCallable(new Callable<List<String>>() {
    @Override
    public List<String> call() throws Exception {
        mRestClient.getFavoriteTvShows();
    }
});


然后订阅一下：

mTvShowSubscription = tvShowSingle
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new SingleSubscriber<List<String>>() {

        @Override
        public void onSuccess(List<String> tvShows) {
            displayTvShows(tvShows);
        }

        @Override
        public void onError(Throwable error) {
            displayErrorMessage();
        }
    });


这段代码和刚才很像，我们调用subscribeOn()方法以确保getFavoriteTvShows()在子线程中执行。而后我们调用observeOn()以确保Single的数据被发送到UI线程。

但这次我们不再使用Observer，而是使用一个叫SingleSubscriber的类。这个类和Observer非常像，只不过它只有上述两个方法：onSuccess()和onError()。SingleSubscriber之于Single就如Observer之于Observable。

订阅一个Single的同时也会自动创建一个Subscription对象。这里的Subscription和案例2中没有区别，一定要在onDestroy()中解除订阅。

最后一点：在这里我们添加了处理异常的代码，所以如果mRestClient出了问题，onError()就会被调用。建议你亲手写一个案例玩一玩，体验一下有异常时程序是怎么运行的。

案例4：Subjects

现在我们写一个Activity，里面要展示一个数字并有一个自增按钮。在看代码之前，先介绍另一个有关RxJava的概念，Subject。Subject这个对象既是Observable又是Observer，我会把Subject想象成一个管道：从一端把数据注入，结果就会从另一端输出。

Subject有好几类，在这里我们使用最简单的：PublishSubject。使用PublishSubject时，一旦数据从一端注入，结果会立即从另一端输出。

首先我们要写这个管道的输出端。刚才说了Subject也是Observable，也就是说我们可以像观察任何一个Observable一样观察它。这段代码的功能就是观察管道的输出端到底输出了什么。我们在这里写一个很简单的Observer来更新mCounterDisplay控件。

mCounterEmitter = PublishSubject.create();
mCounterEmitter.subscribe(new Observer<Integer>() {

    @Override
    public void onCompleted() { }

    @Override
    public void onError(Throwable e) { }

    @Override
    public void onNext(Integer integer) {
        mCounterDisplay.setText(String.valueOf(integer));
    }
});


与前面的几个例子不同，在这个例子中onNext()会被调用多次。每次发送新的数据时，mCounterDisplay都会展示新的数据。但是PublishSubject怎么发送数据呢？让我们看一下mIncrementButton的监听代码。

mIncrementButton.setOnClickListener(new View.OnClickListener() {

    @Override
    public void onClick(View v) {
        mCounter++;
        mCounterEmitter.onNext(mCounter);
    }
});


可以看到mIncrementButton在onClick()回调方法中做了两件事情：

- 让mCounter变量自增。

- 调用mCounterEmitter的onNext()方法并传入mCounter。

由于Subject同时也是Observer，所以它也有onNext()方法，因此我们可以通过调用onNext()方法把数据注入管道的输入端，可以理解为同我们在一端中观察自增按钮是否被点击，然后把信息告知管道另一端的Observer。

案例5：Map()

我们现在要写一个只显示一个数字的Activity。这将是一个很简单的Activity，因为我们要在这里使用map方法。如果你接触过函数式编程，你可能对map并不陌生。你可以把map当做一个方法，它接收一个数据，然后输出另一个数据，当然输入输出的两个数据之间是有联系的。

我们先写一个只发送一个数字4的Single对象。

Single.just(4).map(new Func1<Integer, String>() {

    @Override
    public String call(Integer integer) {
        return String.valueOf(integer);
    }
}).subscribe(new SingleSubscriber<String>() {

    @Override
    public void onSuccess(String value) {
        mValueDisplay.setText(value);
    }

    @Override
    public void onError(Throwable error) { }
});


我们最终要显示Single所发送的数据，但首先我们需要将这个数据从Integer转为String，而这里的解决方法就是使用map()函数。正如刚才所说，map接收一个数据，进行处理而后输出它，这正是我们需要的。现在Single会发送数字4，我们使用map()方法将其转为String，而后交给Observer去展示它。

这个例子中对于map方法的使用很轻量，不过map可是非常强大的，在下一个例子中你可以看到，map可以被用来执行任意代码，在处理数据方面起到很重要的作用。

案例6：综合使用

现在我们要写一个用来根据名字搜索城市的Activity。在这个Activity中，我们要使用在这两篇文章中所学的所有知识并写一个比较大的例子。同时还要介绍一个新的概念：deboundce。开始。

现在我们要写一个PublishSubject，并能接收用书输入进输入框的数据，而后根据输入获取符合的列表，并展示。

mTextWatchSubscription = mSearchResultsSubject
    .debounce(400, TimeUnit.MILLISECONDS)
    .observeOn(Schedulers.io())
    .map(new Func1<String, List<String>>() {

        @Override
        public List<String> call(String s) {
            return mRestClient.searchForCity(s);
        }
    })
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<List<String>>() {

        @Override
        public void onCompleted() { }

        @Override
        public void onError(Throwable e) { }

        @Override
        public void onNext(List<String> cities) {
            handleSearchResults(cities);
        }
    });

mSearchInput.addTextChangedListener(new TextWatcher() {

    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) { }

    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
        mSearchResultsSubject.onNext(s.toString());
    }

    @Override
    public void afterTextChanged(Editable s) { }
});


这段代码有不少内容，让我们一点一点分析。

首先你会看到debounce()方法。这是啥？有啥用？如果你看一下我们是如何给输入框添加监听器的，你会发现每当输入的内容改变时都会有输入发送到mSearchResultsSubject，不过我们不想让用户每点一个键都向服务器请求一次。我们想等一会，等用户停止输入（代表差不多输完）的时候再请求服务器。

而debounce()方法就是做这个的。这个方法告诉mSearchResultsSubject在没有数据传入达400毫秒时才发送数据。意思就是，仅当用户400ms都没有改变输入内容时，Subject才会发送最新的搜索字符串。这样以来我们就不会进行无意义的网络请求了，UI也不会每输入一个字符都更新。

我们想通过RestClient来访问服务器，而因为RestClient涉及IO操作，我们需要在IO Scheduler中进行这个操作，所以要写observeOn(Schedulers.io())。

好了，现在我们会把搜索字段发送到IO Scheduler中，在这里map就要发挥作用了，我们在map方法中通过关键字获取搜索结果的列表。在map中我们可以调用任意外部方法，在这里使用RestClient获取搜索结果。

因为map方法会在IO Scheduler中运行，而我们又要用其返回值填充View，所以要重新切换到UI线程，所以要写observeOn(AndroidSchedulers.mainThread())。现在搜索结果会被发送到UI线程。要注意两个observeOn()方法的顺序，这一点至关重要。现在我们总结一下数据发送的顺序。

mSearchResultsSubject
            |
            |
            V
        debounce
          |||
          |||
          V
          map
          |
          |
          V
        observer


一个竖杠代表数据在UI线程中发送，三个竖杠代表数据在IO Scheduler中发送。

最终，我们获得搜索结果，并展示给用户。

有关RxJava就说这么多了，希望这篇文章能帮你了解RxJava的基础。强烈建议你自己探索有关RxJava的其他方面。如果你有问题或者只是想说点什么，欢迎在下方留言。

