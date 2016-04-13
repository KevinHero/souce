---
title: 异步线程的实现（Looper、MessageQueue、Handler）
date: 2014-06-08 22:01:14
tags: 异步线程的实现（Looper、MessageQueue、Handler）
categories: 技术
---





什么是异步消息处理线程?

对于普通线程来说，执行完run()方法内的代码后线程就结束了。所谓异步消息处理线程
而言，线程启动后会进入一个无限循环体之中，每循环一次，就从其内部的消息队列中取出一
个消息，并回调该消息相应的消息处理函数，执行完一个消息后再继续回到循环体之中。除非
消息队列为空，线程会暂停，直到消息队列中有新的消息了，则继续无限循环。




     异步消息处理线程其本质上也是一个线程，只不过这种线程的执行代码被设计成如上所描述
的逻辑而已。一般来说，当同时处在以下两种需求时使用异步消息处理线程：
<!--more-->

1.任务需要常驻。比如说用于处理用户交互的任务。
2.任务需要根据外部传递的消息而执行不同的操作。


当有这两种需求的时候，就应该使用一个异步消息处理线程去接管。




实现异步消息处理线程的思路




实现异步线程要解决的问题具体包括：

1.每个异步线程内部包含一个消息队列（MessageQueue)，队列中的消息一般采用排队机制
即先到达的消息会先得到处理。

2.线程的执行体中使用while (tru e )进行无限循环，循环体中从消息队列中取出消息，并且根
据消息的来源，回调其对应的消息处理函数。

3.其他外部线程可以向本线程的消息队列中发送消息，消息队列内部的读/写操作必须进行加锁
即消息队列不能同时进行读/ 写操作。







Android中异步线程的实现（Looper、MessageQueue、Handler）




   在线程内部有一个或多个Handler对象，外部程序通过该Handler对象向线程发送异步消息，消
息经由Handler传递到MessageQueue对象中。线程内部只能包含一个MessageQueue对象，线
程主执行函数中从MessageQueue中读取消息，并回调Handler对象中的回调函数handleMessage()。



线程局部存储（Thread Local Storage)

我们通过调用Looper类的静态方法prepare()为线程创建MessageQueue对象:

    // sThreadLocal.get() will return null unless you've called prepare().
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
    private static Looper sMainLooper;  // guarded by Looper.class

    final MessageQueue mQueue;
    final Thread mThread;

    private Printer mLogging;

     /** Initialize the current thread as a looper.
      * This gives you a chance to create handlers that then reference
      * this looper, before actually starting the loop. Be sure to call
      * {@link #loop()} after calling this method, and end it by calling
      * {@link #quit()}.
      */
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }



  从以上的Looper部分源代码可以可以看出，.变量sThreadLocal的类型是ThreadLocal,该类的作
用是提供“线程局部存储”，那么什么是线程局部存储（TLS）?这个问题嘛...可以从变量作用于的
角度来理解。

变量的常见作用域一般包括以下几种。

1.函数内部的变量。其作用域就是该函数，即每次调用该函数时，该变量都会重新回到初始值。

2.类内部的变量。其作用域是该类所产生的对象，即只要对象没有销毁，则对象内部的变量值
一直保持。

3.类内部的静态变量。其作用域是整个过程，即只要在该进程中，则该变量的值就一直保持，无
论使用该类构造过多少个对象，该变量只有一个赋值，并一直保持。


  对于类内部的静态变量而言，无论是从进程中哪个线程引用该变量，其值总是相同的，因为在编
译器内部为静态变量分配了单独的内存空间。但有时我们却希望，当从同一个线程中引用该变量时
其值总是相同，而从不同的线程中引用该变量时，其值应该不同，即我们需要一种作用域为线程的
变量定义，这就是“ 线程局部存储”（同一个线程引用变量值相同，不同线程引用则变量值不相同）。



ThreadLocal就是能够提供这种功能的类，Looper内部的sThreadLocal变量是当该进程第一次调用
Looper.prepare()时被复制的，之后该进程中的其他线程调用prepare()函数时，sThreadLocal变量就已经
被赋值了。sThreadLocal对象内部会根据调用prepare()线程的id 保存一个数据对象，这个数据对象就是
所谓的“线程局部存储” 对象，该对象是通过sThreadLocal的set()方法设置进去的，Looper类中保存
的这个对象是一个Looper对象。

prepare()函数中首先调用sThreadLocal.get()函数获取该线程对应的Looper对象，如果该线程已经存
在Looper对象，则提示出错，否则，为该线程创建一个新的Looper对象。为什么一个线程中只能有一
个Looper对象呢？这仅仅是异步线程所需要的，因为每个Looper对象都会定义一个MessageQueue对
象，一个异步线程中只能有一个消息队列，所以也就只能有一个Looper对象，这与“线程局部存储”
本身没有什么关系，换句话说，这不是ThreadLocal  所导致的结果，可以使用ThreadLocal类来保存
任何数据对象，这就是为什么ThreadLocal是一个模板类的原因。


不同作用域的变量类型
变量作用域类型
意义
函数成员变量	仅在函数内部有效
类成员变量	仅在对象内部有效
线程局部存储（TLS）变量	在本线程内的任何对象内保持一致
静态变量	在本进程内的任何对象内保持一致
跨进程通信（IPC）变量	一般使用Binder进行定义，在所有进程中保持一致



Looper
Looper的作用有两点，第一是为调用该类中静态函数prepare()的线程创建一个消息队列；第二
是提供静态函数loop()，使调用该函数的线程进行无限循环，并从消息队列中读取消息。


创建一个消息队列
在Looper的静态函数prepare()中，会给线程局部存储变量中添加一个新的Looper对象，Looper
的构造函数中则会创建一个MessageQueue对象：

     /** Initialize the current thread as a looper.
      * This gives you a chance to create handlers that then reference
      * this looper, before actually starting the loop. Be sure to call
      * {@link #loop()} after calling this method, and end it by calling
      * {@link #quit()}.
      */
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }


    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }


Looper.prepare()  -> 创建Looper -> 创建Looper时创建MessageQueue -> 然后调用Looper.loop()

当需要把一个线程变成异步消息处理线程时，应该在Thread类的run()函数中先调用Looper.prepare()
 ( sThreadLocal.set(new Looper(quitAllowed)); ） 为线程创建一个MessageQueue对象
（mQueue = new MessageQueue(quitAllowed); ） ，然后调用Looper.loop()函数，是当前线程进
入消息处理循环，让我们再来看看loop()函数的代码：

    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg);

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }


loop()代码的执行流程：

1.调用myLooper函数返回当前线程的Looper对象，从如下myLooper()源码可以看出，该函
数内部仅仅通过调用sThreadLocal.get()方法返回当前线程id对象的Looper对象。

    /**
     * Return the Looper object associated with the current thread.  Returns
     * null if the calling thread is not associated with a Looper.
     */
    public static Looper myLooper() {
        return sThreadLocal.get();
    }


2.进入for(;;)无限循环：

<1>.调用MessageQueue对象的next()函数取出队列中的消息（ Message msg = queue.next(); ）
注意：如果当前队列为空则，则当前线程会被挂起，也就是说，next()函数内部会暂停当前线程。

<2>.回调msg.target.dispatchMessage()函数，完成对该消息的处理，也就是说，消息的具体处理实
际上是由程序指定的。msg变量的类型是Message，msg.target的类型是Handler。可以看如下Message源码：

    /** If set message is in use.
     * This flag is set when the message is enqueued and remains set while it
     * is delivered and afterwards when it is recycled.  The flag is only cleared
     * when a new message is created or obtained since that is the only time that
     * applications are allowed to modify the contents of the message.
     *
     * It is an error to attempt to enqueue or recycle a message that is already in use.
     */
    /*package*/ static final int FLAG_IN_USE = 1 << 0;

    /** If set message is asynchronous */
    /*package*/ static final int FLAG_ASYNCHRONOUS = 1 << 1;

    /** Flags to clear in the copyFrom method */
    /*package*/ static final int FLAGS_TO_CLEAR_ON_COPY_FROM = FLAG_IN_USE;

    /*package*/ int flags;

    /*package*/ long when;

    /*package*/ Bundle data;

    /*package*/ Handler target;

    /*package*/ Runnable callback;

    /**
     * Retrieve the a {@link android.os.Handler Handler} implementation that
     * will receive this message. The object must implement
     * {@link android.os.Handler#handleMessage(android.os.Message)
     * Handler.handleMessage()}. Each Handler has its own name-space for
     * message codes, so you do not need to
     * worry about yours conflicting with other handlers.
     */
    public Handler getTarget() {
        return target;
    }

<3>.每次处理完消息后，需要调用Message的recycle()回收该Message对象占用的系统资源。因为
Message类内部使用了一个数据池去保存Message对象，从而避免不停地创建和删除Message类对象
因此，每次处理完该消息后，需要将该Message对象表明为空闲，以便Message对象可以被重用。
Android5.0的Message.recycle()源码如下：

    /**
     * Return a Message instance to the global pool.
     * <p>
     * You MUST NOT touch the Message after calling this function because it has
     * effectively been freed.  It is an error to recycle a message that is currently
     * enqueued or that is in the process of being delivered to a Handler.
     * </p>
     */
    public void recycle() {
        if (isInUse()) {
            if (gCheckRecycle) {
                throw new IllegalStateException("This message cannot be recycled because it "
                        + "is still in use.");
            }
            return;
        }
        recycleUnchecked();
    }

    /**
     * Recycles a Message that may be in-use.
     * Used internally by the MessageQueue and Looper when disposing of queued Messages.
     */
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }



MessageQueue

消息队列采用排队方式对消息进行处理，就是先到的消息会先得到处理，但是如果消息本身指定
了被处理的时刻，则必须等到该时刻才能处理消息。消息在MessageQueue中使用Message类表
示，队列中的消息以链表的结构进行存储，Message对象内部包含一个next变量，该变量指向下一
个消息。


MessageQueue中的两个主要函数是 "取出消息" 和 "添加消息" ，分别为函数next()和enqueueMessage()。

next()源码 ：
    Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (false) Log.v("MessageQueue", "Returning message: " + msg);
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf("MessageQueue", "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }


<1>.nativePollOnce(ptr, nextPollTimeoutMillis) ：这是一个JNI函数，其作用是从消息队列中
取出一个消息。MessageQueue类内部本身并没有保存消息队列，真正的消息队列数据保存在JNI
中的C 代码中，也就是说，在C 环境中创建了一个NativeMessageQueue数据对象，这就是
nativePollOnce()第一个参数的意义。它是一个in t型变量，在C 环境中，该变量将被强制转换为一
个NativeMessageQueue对象。在C环境中，如果消息队列中没有消息，将导致当前线程被挂起
（wait ) ;如果消息队列中有消息，则C 代码中将把该消息赋值给Java环境中的mMessages变量。

<2>.在synchronized(this)关键字中，th is被用做取消息和写消息的锁，在enqueueMessage()函
数中也使用synchronized(this)进行代码同步。本步代码比较简单，仅仅是判断消息所指定的执行时
间是否到了。如果到了，就返回该消息，并将mMessages变量置空；如果时间还没有到，则尝试读
取下一个消息。

<3>.如果mMessages为空，则说明C 环境中的消息队列没有可执行的消息了， 因此，执行
mPendingldleHandlers列表中的“ 空闲回调函数”。程序员可以向MessageQueue中注册一些
“空闲回调函数”，从而当线程中没有消息可处理时去执行这些“空闲代码”。


enqueueMessage():

    boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w("MessageQueue", e.getMessage(), e);
                msg.recycle();
                return false;
            }

            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }


<1>.将参数msg赋值给mMessages （mMessages = msg; ） 。

<2>.调用nativeWake(mPtr)。这是一个JNI函数，其内部会将mMessages消息添加到C 环境中
的消息队列中，并且如果消息线程正处于挂起(wait)状态，则唤醒该线程。




Handler

虽然MessageQueue提供了直接读/写的函数接口，但是一般不直接读/写消息队列。在Looper.loop()
函数中，当取出消息后，会回调msg.target对象的handleMessage()函数，而msg.target的类型正是
Handler。


一般使用Handler类向消息队列中发送消息，并重载Handler类的handleMessage()函数添加消息处理代码。


Handler对象只能添加到有消息队列的线程中，否则会发生异常。我们可以从Handler类的构造
函数中看得出来：

    /**
     * Use the {@link Looper} for the current thread with the specified callback interface
     * and set whether the handler should be asynchronous.
     *
     * Handlers are synchronous by default unless this constructor is used to make
     * one that is strictly asynchronous.
     *
     * Asynchronous messages represent interrupts or events that do not require global ordering
     * with represent to synchronous messages.  Asynchronous messages are not subject to
     * the synchronization barriers introduced by {@link MessageQueue#enqueueSyncBarrier(long)}.
     *
     * @param callback The callback interface in which to handle messages, or null.
     * @param async If true, the handler calls {@link Message#setAsynchronous(boolean)} for
     * each {@link Message} that is sent to it or {@link Runnable} that is posted to it.
     *
     * @hide
     */
    public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }


因此，在构造Handler对象前，必须已经执行过Looper.prepare()，但prepare()不能被执行两
次。创建Handler对象可以再执行Looper.loop()函数之前，也可以再执行之后。我们平时一般
在Activity对象的初始化代码中添加Handler对象，但是实际上，在Activity对象被构造前，Activity
所在的线程已经执行了Looper.prepare()。所以为什么明明Handler里有判断（if(mLooper == null））
我们感觉我们没调用Looper.prepare()方法也能初始化Handler。


一个线程中可以包含多个Handler对象。在Looper.loop()函数中，不同的Message对应不同的Handler
对象，从而回调不同的handleMessage()函数。


异步消息处理线程在Framework中被广泛使用，除了用于多线程消息传递外，它还和跨进程调用（IPC）
一起被使用，用于实现异步跨进程调用。所以我们可以这样，以后只要看到Handler对象，就应该想到异
步消息处理线程。






Rights Reserved
