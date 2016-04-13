---
title: Android内核 Binder
date: 2014-06-12 22:01:14
tags: Binder
categories: 技术
---



   Binder,英文的意思是别针、回形针。平常我们经常用回形针把几张纸夹起来。而在Android
中，Binder用于完成进程间通信（IPC），就是把多个线程夹起来。比如，普通应用程序可以
调用音乐播放服务的播放、暂停、停止等功能。

<!--more-->
   Binder工作在Linux层面，属于一个驱动，只是这个驱动不需要硬件，或者说Binder操作的硬
件是基于一小段内存。从线程的角度来讲，Binder驱动代码运行在内核态，客户端程序调用Binder
是通过系统调用完成的。





Binder框架
   Binder是一种架构，这种架构提供了服务端接口、Binder驱动、客户端接口三个模块，如下图 ：


   首先来看服务端。一个Binder服务端实际上就是一个Binder类的对象，该对象一旦创建，内部就
启动一个隐藏线程。该线程接下来会接收Binder驱动发送的消息，收到消息后，会执行到Binder对
象中的onTransact()函数，并按照该函数的参数执行不同的服务代码。因此，要实现一个Binder服
务，就必须重载onTransact()方法。

   可以想象，重载onTransact()函数的主要内容是把onTransact()函数的参数转换为服务函数的参数
而onTransact()函数的参数来源是客户端调用transact()函数时输入的，因此，如果transact()有固定
格式的输入，那么onTransact()就会有固定格式的输出。

   那再看Binder驱动。任意一个服务端Binder对象被创建时，同时会在Binder驱动中创建一个mRemote
对象，该对象的类型也是Binder类。客户端访问远程服务时，都是通过mRemote对象。

   最后来看应用程序客户端。客户端要想访问远程服务，必须获取远程服务在Binder对象中对应的mRemote
引用。获取该mRemote对象后，就可以调用其transact()方法，而在Binder驱动中，mRemote对象也重载了
transact()方法，重载的内容包括以下几项：

1.以线程间消息通信的模式，向服务端发送客户端传递过来的参数。
        2.挂起当前线程，当前线程正是客户端线程，并等待服务端线程执行完指定服务函数后通知(notify)。
3.接收服务端线程的通知，然后继续执行客户端线程，并返回到客户端代码区。

   以上可以得出，客户端似乎是直接调用远程服务对应的Binder，而事实上则是通过Binder驱动进行
了中转。即存在两个Binder对象，一个是服务端的Binder对象，另一个则是Binder驱动中的Binder对
象，所不同的是Binder驱动中的对象不会再额外产生一个线程。


   那么，我们来看看Android Binder的源代码：

    /**
     * Default constructor initializes the object.
     */
    public Binder() {
        init();

        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Binder> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Binder class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
    }

    private native final void init();
    private native final void destroy();

   这两段代码可以看出，在Binder的构造函数中，执行了init()方法。通过JNI的形式调用了底层C
的init()函数，内部就完成了"启动隐藏线程"。


    /**
     * Default implementation is a stub that returns false.  You will want
     * to override this to do the appropriate unmarshalling of transactions.
     *
     * <p>If you want to call this, call transact().
     */
    protected boolean onTransact(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException {
        if (code == INTERFACE_TRANSACTION) {
            reply.writeString(getInterfaceDescriptor());
            return true;
        } else if (code == DUMP_TRANSACTION) {
            ParcelFileDescriptor fd = data.readFileDescriptor();
            String[] args = data.readStringArray();
            if (fd != null) {
                try {
                    dump(fd.getFileDescriptor(), args);
                } finally {
                    try {
                        fd.close();
                    } catch (IOException e) {
                        // swallowed, not propagated back to the caller
                    }
                }
            }
            // Write the StrictMode header.
            if (reply != null) {
                reply.writeNoException();
            } else {
                StrictMode.clearGatheredViolations();
            }
            return true;
        }
        return false;
    }

    /**
     * Default implementation rewinds the parcels and calls onTransact.  On
     * the remote side, transact calls into the binder to do the IPC.
     */
    public final boolean transact(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException {
        if (false) Log.v("Binder", "Transact: " + code + " to " + this);
        if (data != null) {
            data.setDataPosition(0);
        }
        boolean r = onTransact(code, data, reply, flags);
        if (reply != null) {
            reply.setDataPosition(0);
        }
        return r;
    }


   这两段代码可以看出，onTransact中主要对data进行了处理，即把onTranSact()函数的参数转换
为服务函数的参数；而在transact中调用了onTransact()函数也恰巧说明了：transact()有固定格式的
输入，那么onTransact()就会有固定格式的输出。




简单实现Binder架构


设计Server端

设计Server端实际上非常简单，从代码的角度而言，只要基于Binder类新建一个Server类即可。
MusicPlayerService:

    package com.zyy.android_csdn.skill;

    import android.os.Binder;
    import android.os.Parcel;
    import android.os.RemoteException;
    import android.util.Log;

    /**
     *
     * 请叫我"Server端"
     *
     * @author CaMnter
     *
     */

    public class MusicPlayerServiceBinder extends Binder implements
    		IMusicPlayerService {

    	@Override
    	protected boolean onTransact(int code, Parcel data, Parcel reply, int flags)
    			throws RemoteException {

    		switch (code) {
    		case 1000:

    			// enforceInterface为了校验 与客户端的writeInterfaceToken()对应
    			data.enforceInterface("MusicPlayerService");

    			// data 中读取客户端传递的参数
    			String filePath = data.readString();

    			/*
    			 * 如果该IPC调用的客户端期望返回一些结果，则可以在返回包裹reply 中调用Parcel提供的相关函 数写入相应的结果
    			 */
    			// reply.writeXXX()

    			start(filePath);

    			break;

    		default:
    			break;
    		}

    		return super.onTransact(code, data, reply, flags);

    	}

    	public void start(String filePath) {

    		Log.i("CaMnter", filePath);

    	}

    	public void stop() {

    	}

    }


    IMusicPlayerService：
    package com.zyy.android_csdn.skill;

    /**
     *
     * 请叫我 "MusicPlayerService 标识接口"
     *
     * @author CaMnter
     *
     */

    public interface IMusicPlayerService {

    }




   以上定义了服务类，重载了Binder的onTransact()方法，code变量用于标识客户端期望调用服务端
的哪个函数，因此，双方需要约束一组int值，不同的值代表不同的服务端函数，该值和客户端的transact()
函数中的第一个参数code是一致的。这里假定1000是双方约束要调用start函数的值。

   enforceInterface()是为了某种校验，与客户端的writeInterfaceToken()对应。

   readString()用于从包裹中取出一个字符串。取出filePath变量后，就可以调用服务端的start()函数。

   如果该IPC调用的客户端期望返回一些结果，我们则可以再返回包裹reply中调用Parcel提供的相关函数写入相应的方法





设计Binder 客户端

   要想使用服务端（MusicPlayerServiceBinder）,首先要获取服务端在Binder驱动中对应的
mRemote变量的引用，获取的方法在以后的会讲到，这里我们就 new MusicPlayerServiceBinder()
出来代替mRemote，合情合理的，因为mRemote也同属于IBinder类型。这个不多提，获得这个
mRemote后，就可以调用mRemote的transact()方法，该方法的函数原型就让我们参照一下Binder
的部分源代码吧：

    /**
     * Default implementation rewinds the parcels and calls onTransact.  On
     * the remote side, transact calls into the binder to do the IPC.
     */
    public final boolean transact(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException {
        if (false) Log.v("Binder", "Transact: " + code + " to " + this);
        if (data != null) {
            data.setDataPosition(0);
        }
        boolean r = onTransact(code, data, reply, flags);
        if (reply != null) {
            reply.setDataPosition(0);
        }
        return r;
    }


   final的"终结"方法。其中data表示要传递给远程Binder的包裹（Parcel），远程服务函数所需要的
参数必须放入这个Parcel data 里。Parcel中只能放入一些特定类型的变量，这些类型有：String、int
、long等，要查看Parcel可以放入的全部数据类型，可以参照一下Parcel类。当然值得一提的是，Parcel
还提供了一个writeParcel()方法，可以再Parcel中包含一个小包裹（Parcel）。因此，要进行Binder远程
服务调用时，服务函数的参数要么是一个原子类，要么必须继承Parcel类。否则，不能传递。

接下来，我做了一个简单的实现，直观上就是客户端调用了mRemote.transact()：

    package com.zyy.android_csdn.skill;

    import android.app.Service;
    import android.content.Intent;
    import android.os.IBinder;
    import android.os.Parcel;
    import android.os.RemoteException;

    /**
     *
     * 请叫我"Binder 客户端"
     *
     * @author CaMnter
     *
     */

    public class MusicPlayerService extends Service implements IMusicPlayerService {

    	private MusicPlayerServiceBinder musicPlayerServiceBinder = new MusicPlayerServiceBinder();

    	@Override
    	public IBinder onBind(Intent intent) {
    		return this.musicPlayerServiceBinder;
    	}

    	public MusicPlayerService() {

    		super();

    		this.startTransact();

    	}

    	public void startTransact() {

    		IBinder mRemote = musicPlayerServiceBinder;

    		String filePath = "/sdcard/music/helloword.mp3";

    		int code = 1000;

    		Parcel data = Parcel.obtain();

    		Parcel reply = Parcel.obtain();

    		data.writeInterfaceToken("MusicPlayerService");

    		data.writeString(filePath);

    		try {

    			mRemote.transact(code, data, reply, 0);

    		} catch (RemoteException e) {

    			e.printStackTrace();

    		}

    		reply.recycle();

    		data.recycle();

    	}

    }



   以上代码。首先，Parcel类型的data、reply不是客户端自己创建的，而是调用Parcel.obtain()申请
的， 这正如生活中的邮局一样，用户一般只能用邮局提供的信封（EMS）。其中data和reply变量都是
由客户端提供，reply变量用户客户端把返回结果放入其中。


   writeInterfaceToken()方法标注远程服务名称，理论上而言，这个名称不是必需的，因为客户端既然
已经获取指定远程服务的Binder引用（这里我IBinder mRemote = musicPlayerServiceBinder; 来模拟
获取过程），那么就不会调用到其他远程服务。该名称将作为Binder驱动确保客户端的确想调用指定的服
务端。

   writeString()方法用于向Parcel类型中添加一个String变量。注意，Parcel中添加的内容是有序的，这个
顺序必须是客户端和服务端事先约束好的，在服务端onTransact方法中会按照约定的顺序取出变量。

   接着，mRemote.transact(code, data, reply, 0)。调用transact()方法，客户端线程进入了Binder驱动
Binder驱动会挂起当前的线程(客户端线程)，并向远程服务发送一个消息，消息中包含了客户端传来的包裹
（Parcel）。服务端拿到包裹（Parcel）后，会对包裹（Parcel）进行拆解，然后执行指定的服务函数，执
行完毕后，再将执行完毕后的执行结果放入客户端提供的reply包裹（Parcel）中。然后服务端想Binder驱动
发送一个notify的消息，从而使得客户端线程从Binder驱动代码区返回到客户端代码区。

再看：

    /**
     * Default implementation rewinds the parcels and calls onTransact.  On
     * the remote side, transact calls into the binder to do the IPC.
     */
    public final boolean transact(int code, Parcel data, Parcel reply,
            int flags) throws RemoteException {
        if (false) Log.v("Binder", "Transact: " + code + " to " + this);
        if (data != null) {
            data.setDataPosition(0);
        }
        boolean r = onTransact(code, data, reply, flags);
        if (reply != null) {
            reply.setDataPosition(0);
        }
        return r;
    }


   可以发现transact()方法的最后一个参数是 int flags。这个flags参数的含义是执行IPC调用的模式，分为
两种：一种 0 表示 双向的，其含义表示服务端执行完指定服务后会返回一定的数据；另一张 1 表示 单向的，其
含义是不返回任何数据。

   最后，客户端（我这的是MusicPlayerService）就可以从reply中解析服务端（我这的是MusicPlayerServiceBinder）
返回的数据了，同样，返回包裹中包含的数据也必须是有序的，而且这个顺序也必须是服务端和客户端事先约定好的。





绑定服务，实现Binder架构


    BinderActivity:

    package com.zyy.android_csdn;

    import com.zyy.android_csdn.skill.IMusicPlayerService;

    import android.support.v7.app.ActionBarActivity;
    import android.content.ComponentName;
    import android.content.Intent;
    import android.content.ServiceConnection;
    import android.os.Bundle;
    import android.os.IBinder;
    import android.util.Log;
    import android.view.Menu;
    import android.view.MenuItem;

    public class BinderActivity extends ActionBarActivity {

    	/**
    	 *
    	 * onServiceConnected(ComponentName name, IBinder service)中的 service保存为一个全局变量
    	 *
    	 *
    	 * 从而在客户端的任何地方都可以随时调用该远程服务 这就解决了第一个重要问题
    	 *
    	 *
    	 * 即客户端如何 获取远程服务的Binder引用。
    	 *
    	 *
    	 */
    	public static IMusicPlayerService iMusicPlayerService;

    	private ServiceConnection serviceConnection = new ServiceConnection() {

    		@Override
    		public void onServiceDisconnected(ComponentName name) {

    		}

    		@Override
    		public void onServiceConnected(ComponentName name, IBinder service) {

    			/**
    			 *
    			 * 保存这个IBinder
    			 *
    			 */
    			BinderActivity.iMusicPlayerService = (IMusicPlayerService) service;

    			Log.i("CaMnter", iMusicPlayerService.getClass().hashCode() + "");

    		}
    	};

    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
    		super.onCreate(savedInstanceState);
    		setContentView(R.layout.adapter);

    		new Thread("CaMnterThread") {

    			@Override
    			public void run() {

    				BinderActivity.this
    						.bindService(new Intent("com.zyy.MusicPlayerService"),
    								BinderActivity.this.serviceConnection,
    								BIND_AUTO_CREATE);

    			}

    		}.start();

    	}

    	@Override
    	public boolean onCreateOptionsMenu(Menu menu) {
    		getMenuInflater().inflate(R.menu.main, menu);
    		return true;
    	}

    	@Override
    	public boolean onOptionsItemSelected(MenuItem item) {
    		int id = item.getItemId();
    		if (id == R.id.action_settings) {
    			return true;
    		}
    		return super.onOptionsItemSelected(item);
    	}
    }



   这里就是用一个ServiceConnection取出了Binder类型然后转成标识接口IMusicPlayerService，接着就是
绑定服务了。




  我们可以看到 "/sdcard/music/helloword.mp3" 证明了一个简单的Binder架构已经实现了。


ServiceConnection里的学问和技巧

   请注意，onServiceConnected()的第二个变量service。当客户端请求AmS（ActivityManagerService）
启动某个服务（Service）后，该服务（Service）如果正常启动，那么Ams（ActivityManagerService）就
会远程调用Ams（ActivityManagerService）就会调用ActivityThread类中的ApplicationThread对象，调用
的参数中会包含服务（Service）的Binder引用，然后ApplicationThread中会回调bindService中的ServiceConnection接口
（这里我的是 BinderActivity.this.serviceConnection）。因此，在客户端中，可以再onServiceConnected()
方法中将第二个变量service保存为一个全局变量（这里我做了 BinderActivity.iMusicPlayerService = (IMusicPlayerService) service;）
从而客户端的任何地方都可以随时调用该远程服务。这就解决了第一个重要问题，就是客户端如何获取远程服务的Binder引用。


   客户端 -> Ams(ActivityManagerService) -> 调用ActivityThread类中的ApplicationThread（参数包含Binder引用）
   -> 回调bindService的ServiceConnection





Binder和AIDL

   关于第二个问题，Android的SDK中提供了一个aidl工具，该工具可以把一个aidl文件转换为一
个Java类文件，在该Java类文件，同时重载了 transact和onTransactO方法，统一 了存入包裹和
读取包裹参数，从而使设计者可以把注意力放到服务代码本身上。

可以首先编写一个IMusicPlayerServiceAIDL.aidl：

    package com.zyy.android_csdn.skill;

    interface IMusicPlayerServiceAIDL{

    	boolean start(String filePath);

    	void stop();

    }


   aidl文件的语法基本类似于Java，package指定输出后的Java文件对应的包名。如果该文件需要引
用其他Java类，则可以使用import关键字，但需要注意的是，包裹（Parcel，这里指的是 String filePath
 属于原子型）内只能写入以下三个类型的内容。

• Java原子类型，如int、long、String等变量。
• Binder 引用。
• 实现了 Parcelable的对象。


   接下来，我们看看该aidl生成的IMusicPlayerService.java文件的代码：

    /*
     * This file is auto-generated.  DO NOT MODIFY.
     * Original file: D:\\git-camnter\\CaMnter_CSDN\\src\\com\\zyy\\android_csdn\\skill\\IMusicPlayerServiceAIDL.aidl
     */
    package com.zyy.android_csdn.skill;

    public interface IMusicPlayerServiceAIDL extends android.os.IInterface {
    	/** Local-side IPC implementation stub class. */
    	public static abstract class Stub extends android.os.Binder implements
    			com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL {
    		private static final java.lang.String DESCRIPTOR = "com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL";

    		/** Construct the stub at attach it to the interface. */
    		public Stub() {
    			this.attachInterface(this, DESCRIPTOR);
    		}

    		/**
    		 * Cast an IBinder object into an
    		 * com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL interface,
    		 * generating a proxy if needed.
    		 */
    		public static com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL asInterface(
    				android.os.IBinder obj) {
    			if ((obj == null)) {
    				return null;
    			}
    			android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
    			if (((iin != null) && (iin instanceof com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL))) {
    				return ((com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL) iin);
    			}
    			return new com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL.Stub.Proxy(
    					obj);
    		}

    		@Override
    		public android.os.IBinder asBinder() {
    			return this;
    		}

    		@Override
    		public boolean onTransact(int code, android.os.Parcel data,
    				android.os.Parcel reply, int flags)
    				throws android.os.RemoteException {
    			switch (code) {
    			case INTERFACE_TRANSACTION: {
    				reply.writeString(DESCRIPTOR);
    				return true;
    			}
    			case TRANSACTION_start: {
    				data.enforceInterface(DESCRIPTOR);
    				java.lang.String _arg0;
    				_arg0 = data.readString();
    				boolean _result = this.start(_arg0);
    				reply.writeNoException();
    				reply.writeInt(((_result) ? (1) : (0)));
    				return true;
    			}
    			case TRANSACTION_stop: {
    				data.enforceInterface(DESCRIPTOR);
    				this.stop();
    				reply.writeNoException();
    				return true;
    			}
    			}
    			return super.onTransact(code, data, reply, flags);
    		}

    		private static class Proxy implements
    				com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL {
    			private android.os.IBinder mRemote;

    			Proxy(android.os.IBinder remote) {
    				mRemote = remote;
    			}

    			@Override
    			public android.os.IBinder asBinder() {
    				return mRemote;
    			}

    			public java.lang.String getInterfaceDescriptor() {
    				return DESCRIPTOR;
    			}

    			@Override
    			public boolean start(java.lang.String filePath)
    					throws android.os.RemoteException {
    				android.os.Parcel _data = android.os.Parcel.obtain();
    				android.os.Parcel _reply = android.os.Parcel.obtain();
    				boolean _result;
    				try {
    					_data.writeInterfaceToken(DESCRIPTOR);
    					_data.writeString(filePath);
    					mRemote.transact(Stub.TRANSACTION_start, _data, _reply, 0);
    					_reply.readException();
    					_result = (0 != _reply.readInt());
    				} finally {
    					_reply.recycle();
    					_data.recycle();
    				}
    				return _result;
    			}

    			@Override
    			public void stop() throws android.os.RemoteException {
    				android.os.Parcel _data = android.os.Parcel.obtain();
    				android.os.Parcel _reply = android.os.Parcel.obtain();
    				try {
    					_data.writeInterfaceToken(DESCRIPTOR);
    					mRemote.transact(Stub.TRANSACTION_stop, _data, _reply, 0);
    					_reply.readException();
    				} finally {
    					_reply.recycle();
    					_data.recycle();
    				}
    			}
    		}

    		static final int TRANSACTION_start = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    		static final int TRANSACTION_stop = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    	}

    	public boolean start(java.lang.String filePath)
    			throws android.os.RemoteException;

    	public void stop() throws android.os.RemoteException;
    }


以上代码主要完成了三件事：

定义一个Java Interface，内部包含aidl文件所声明的服务函数，类名字为IMusicPlayerServiceAIDL
并且该类基于Ilnterface接口，即需要提供一个asBinder()函数。

定义一个Proxy类，该类将作为客户端程序访问服务端的代理。所谓的代理主要就是为了前面所提到的
第二个重要问题——统一包裹（Parcel）内写入参数的顺序。

定义一个Stub类，这是一个 abstract 类，基于Binder类，并且实现了IMusicPlayerServiceAIDL接口，
主要由服务端来使用。该类之所以要定义为一个abstract类，是因为具体的服务函数必须由程序员实现，因此
，IMusicPlayerServiceAIDL接口里定义的函数在Stub类中可以没有具体的实现。同时，在Stub类中重载了
onTransact()方法，由于transact()方法内部给包裹（Parcel）内写入参数的顺序是由aidl工具定义的，也因此
aidl工具自然知道应该按照何种顺序从包裹（Parcel）中取出相应的参数。


   在Stub类中还定义了一些int常量，比如TRANSACTION_start，这些常量与服务函数对应，transact()
和onTransactO方法的第一个参数code的值即来源于此。


   在Stub类中，除了以上所述的任务外，Stub还提供了一个asInterface()函数。提供这个函数的作用是
这样的：首先需要明确的是，aidl所产生的代码完全可以由应用程序员手工编写，IMusicPlayerServiceAIDL
中的函数只是一种编码习惯而已，aslnterface即如此，提供这个函数的原因是：服务端提供的服务除了其他
进程可以使用外（前者），在服务进程内部的其他类可以使用该服务（后者），对于后者，显然是不需要经
过IPC调用的，而可以直接在进程内部调用的，而Binder内部有一个queryLocallnterface (String description)
函数，那么，我们顺便来看一看Binder内的这部分关于queryLocallnterface (String description) 的源代码：


    /**
     * Use information supplied to attachInterface() to return the
     * associated IInterface if it matches the requested
     * descriptor.
     */
    public IInterface queryLocalInterface(String descriptor) {
        if (mDescriptor.equals(descriptor)) {
            return mOwner;
        }
        return null;
    }


    /**
     * Convenience method for associating a specific interface with the Binder.
     * After calling, queryLocalInterface() will be implemented for you
     * to return the given owner IInterface when the corresponding
     * descriptor is requested.
     */
    public void attachInterface(IInterface owner, String descriptor) {
        mOwner = owner;
        mDescriptor = descriptor;
    }

    /* mObject is used by native code, do not remove or rename */
    private long mObject;
    private IInterface mOwner;
    private String mDescriptor;

   以上的三段代码可以看出，queryLocallnterface根据输入的字符串判断该Binder对象时一个本地的Binder
引用。我们再看看如下图所示：



   当创建一个Binder对象时，服务端进程内部创建一个Binder对象，Binder驱动中也会创建一个Binder
对象。如果从远程获取服务端的Binder，则只会返回Binder驱动中的的Binder对象，而如果从服务端内
部获取Binder对象的话，则会获取服务端本身的Binder对象

   再结合aidl生成的asInterface()的源码：

		/**
		 * Cast an IBinder object into an
		 * com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL interface,
		 * generating a proxy if needed.
		 */
		public static com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL asInterface(
				android.os.IBinder obj) {
			if ((obj == null)) {
				return null;
			}
			android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
			if (((iin != null) && (iin instanceof com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL))) {
				return ((com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL) iin);
			}
			return new com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL.Stub.Proxy(
					obj);
		}


   我可以看到这句：android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR)。调用了
queryLocalInterface()方法，asInterface()提供了一个统一的接口。无论是远程客户端还是本地端，当
获取Binder对象后，可以把获取的Binder对象作为asInterface()的参数，从而返回一个IMusicPlayerServicAIDL
接口。又可以从return new com.zyy.android_csdn.skill.IMusicPlayerServiceAIDL.Stub.Proxy(obj）中看
出，该接口要么使用Proxy类，要么直接使用Stub所实现的相应服务函数。



那么问题来了!

    我们经常使用的getSystemService(String serviceName)方法获取一个系统服务，那么这些系统服务的
Binder引用时如何传递给客户端?

请关注我的 Android ServiceManager 篇
