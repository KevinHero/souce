---
title: Android内核 ServiceManager
date: 2014-06-15 18:15:38
tags: ServiceManager
categories: 技术
---




getSystemService很尽职，你知道吗?


   那个...嗯...曾记否上篇的`Android Binder`中留下的疑问?就是在Context的实现类中使用
getSystemService(String serviceName)方法获取一个系统服务，那么这些系统服务的Binder
引用时如何传递给客户端的呢?其实这里有个关键点，就是系统服务并不是通过startService()启
动的。
<!--more-->
   getSystemService函数的实现是在Context的实现类中，该函数返回的Service比较多。那
么就让我们看一下Activity、ContextThemeWrapper的部分源代码吧：

Activity：

    @Override
    public Object getSystemService(@ServiceName @NonNull String name) {
        if (getBaseContext() == null) {
            throw new IllegalStateException(
                    "System services not available to Activities before onCreate()");
        }

        if (WINDOW_SERVICE.equals(name)) {
            return mWindowManager;
        } else if (SEARCH_SERVICE.equals(name)) {
            ensureSearchManager();
            return mSearchManager;
        }
        return super.getSystemService(name);
    }


   1.首先， if (getBaseContext() == null)判断 mBase（Context类型）是否为空。如果为空则 ，提
示Context没创建前调用了getSystemService的错误。至于，我为什么能准确地mBase这个名字呢?让
我们再看一段关于ContextWrapper（Activity的父类、Context的子类）的源代码：

    Context mBase;

    public ContextWrapper(Context base) {
        mBase = base;
    }

    /**
     * Set the base context for this ContextWrapper.  All calls will then be
     * delegated to the base context.  Throws
     * IllegalStateException if a base context has already been set.
     *
     * @param base The new base context for this wrapper.
     */
    protected void attachBaseContext(Context base) {
        if (mBase != null) {
            throw new IllegalStateException("Base context already set");
        }
        mBase = base;
    }

    /**
     * @return the base context as set by the constructor or setBaseContext
     */
    public Context getBaseContext() {
        return mBase;
    }

   以上很简单，也就是从父类继承下来的Context mBase属性。

   2.if (WINDOW_SERVICE.equals(name)) {......}elseif(SEARCH_SERVICE.equals(name)){......}。这个
就是判断要取的服务是Window类型还是Search类型的。这个在Activity里有对应的WindowManager类
型和SearchManager类型属性，并且做了对应的实例化。

   3.return super.getSystemService(name)：当要取的服务不是Window类型也不是Search类型的时候，就
调用了父类（ContextThemeWrapper）的getSystemService(String name)帮你找服务。那么就让我们顺便看看
ContextThemeWrapper的源代码：

    @Override public Object getSystemService(String name) {
        if (LAYOUT_INFLATER_SERVICE.equals(name)) {
            if (mInflater == null) {
                mInflater = LayoutInflater.from(getBaseContext()).cloneInContext(this);
            }
            return mInflater;
        }
        return getBaseContext().getSystemService(name);
    }


   4.这里也做了个Activity里类似的判断是否为 layout Inflater 服务，然后返回相应的layout Inflater 服务。

   5.return getBaseContext().getSystemService(name)：这里的话就回到了Context 这个抽象的基类。 我
们就再看看Context的源代码：

    /**
     * Return the handle to a system-level service by name. The class of the
     * returned object varies by the requested name. Currently available names
     * are:
     *
     * <dl>
     *  <dt> {@link #WINDOW_SERVICE} ("window")
     *  <dd> The top-level window manager in which you can place custom
     *  windows.  The returned object is a {@link android.view.WindowManager}.
     *  <dt> {@link #LAYOUT_INFLATER_SERVICE} ("layout_inflater")
     *  <dd> A {@link android.view.LayoutInflater} for inflating layout resources
     *  in this context.
     *  <dt> {@link #ACTIVITY_SERVICE} ("activity")
     *  <dd> A {@link android.app.ActivityManager} for interacting with the
     *  global activity state of the system.
     *  <dt> {@link #POWER_SERVICE} ("power")
     *  <dd> A {@link android.os.PowerManager} for controlling power
     *  management.
     *  <dt> {@link #ALARM_SERVICE} ("alarm")
     *  <dd> A {@link android.app.AlarmManager} for receiving intents at the
     *  time of your choosing.
     *  <dt> {@link #NOTIFICATION_SERVICE} ("notification")
     *  <dd> A {@link android.app.NotificationManager} for informing the user
     *   of background events.
     *  <dt> {@link #KEYGUARD_SERVICE} ("keyguard")
     *  <dd> A {@link android.app.KeyguardManager} for controlling keyguard.
     *  <dt> {@link #LOCATION_SERVICE} ("location")
     *  <dd> A {@link android.location.LocationManager} for controlling location
     *   (e.g., GPS) updates.
     *  <dt> {@link #SEARCH_SERVICE} ("search")
     *  <dd> A {@link android.app.SearchManager} for handling search.
     *  <dt> {@link #VIBRATOR_SERVICE} ("vibrator")
     *  <dd> A {@link android.os.Vibrator} for interacting with the vibrator
     *  hardware.
     *  <dt> {@link #CONNECTIVITY_SERVICE} ("connection")
     *  <dd> A {@link android.net.ConnectivityManager ConnectivityManager} for
     *  handling management of network connections.
     *  <dt> {@link #WIFI_SERVICE} ("wifi")
     *  <dd> A {@link android.net.wifi.WifiManager WifiManager} for management of
     * Wi-Fi connectivity.
     *  <dt> {@link #WIFI_P2P_SERVICE} ("wifip2p")
     *  <dd> A {@link android.net.wifi.p2p.WifiP2pManager WifiP2pManager} for management of
     * Wi-Fi Direct connectivity.
     * <dt> {@link #INPUT_METHOD_SERVICE} ("input_method")
     * <dd> An {@link android.view.inputmethod.InputMethodManager InputMethodManager}
     * for management of input methods.
     * <dt> {@link #UI_MODE_SERVICE} ("uimode")
     * <dd> An {@link android.app.UiModeManager} for controlling UI modes.
     * <dt> {@link #DOWNLOAD_SERVICE} ("download")
     * <dd> A {@link android.app.DownloadManager} for requesting HTTP downloads
     * <dt> {@link #BATTERY_SERVICE} ("batterymanager")
     * <dd> A {@link android.os.BatteryManager} for managing battery state
     * <dt> {@link #JOB_SCHEDULER_SERVICE} ("taskmanager")
     * <dd>  A {@link android.app.job.JobScheduler} for managing scheduled tasks
     * </dl>
     *
     * <p>Note:  System services obtained via this API may be closely associated with
     * the Context in which they are obtained from.  In general, do not share the
     * service objects between various different contexts (Activities, Applications,
     * Services, Providers, etc.)
     *
     * @param name The name of the desired service.
     *
     * @return The service or null if the name does not exist.
     *
     * @see #WINDOW_SERVICE
     * @see android.view.WindowManager
     * @see #LAYOUT_INFLATER_SERVICE
     * @see android.view.LayoutInflater
     * @see #ACTIVITY_SERVICE
     * @see android.app.ActivityManager
     * @see #POWER_SERVICE
     * @see android.os.PowerManager
     * @see #ALARM_SERVICE
     * @see android.app.AlarmManager
     * @see #NOTIFICATION_SERVICE
     * @see android.app.NotificationManager
     * @see #KEYGUARD_SERVICE
     * @see android.app.KeyguardManager
     * @see #LOCATION_SERVICE
     * @see android.location.LocationManager
     * @see #SEARCH_SERVICE
     * @see android.app.SearchManager
     * @see #SENSOR_SERVICE
     * @see android.hardware.SensorManager
     * @see #STORAGE_SERVICE
     * @see android.os.storage.StorageManager
     * @see #VIBRATOR_SERVICE
     * @see android.os.Vibrator
     * @see #CONNECTIVITY_SERVICE
     * @see android.net.ConnectivityManager
     * @see #WIFI_SERVICE
     * @see android.net.wifi.WifiManager
     * @see #AUDIO_SERVICE
     * @see android.media.AudioManager
     * @see #MEDIA_ROUTER_SERVICE
     * @see android.media.MediaRouter
     * @see #TELEPHONY_SERVICE
     * @see android.telephony.TelephonyManager
     * @see #INPUT_METHOD_SERVICE
     * @see android.view.inputmethod.InputMethodManager
     * @see #UI_MODE_SERVICE
     * @see android.app.UiModeManager
     * @see #DOWNLOAD_SERVICE
     * @see android.app.DownloadManager
     * @see #BATTERY_SERVICE
     * @see android.os.BatteryManager
     * @see #JOB_SCHEDULER_SERVICE
     * @see android.app.job.JobScheduler
     */
    public abstract Object getSystemService(@ServiceName @NonNull String name);


    /******************************以下是服务的名称***************************/


    /**
     * Use with {@link #getSystemService} to retrieve a
     * {@link android.os.PowerManager} for controlling power management,
     * including "wake locks," which let you keep the device on while
     * you're running long tasks.
     */
    public static final String POWER_SERVICE = "power";

    /**
     * Use with {@link #getSystemService} to retrieve a
     * {@link android.view.WindowManager} for accessing the system's window
     * manager.
     *
     * @see #getSystemService
     * @see android.view.WindowManager
     */
    public static final String WINDOW_SERVICE = "window";

    /**
     * Use with {@link #getSystemService} to retrieve a
     * {@link android.view.LayoutInflater} for inflating layout resources in this
     * context.
     *
     * @see #getSystemService
     * @see android.view.LayoutInflater
     */
    public static final String LAYOUT_INFLATER_SERVICE = "layout_inflater";

    /**
     * Use with {@link #getSystemService} to retrieve a
     * {@link android.accounts.AccountManager} for receiving intents at a
     * time of your choosing.
     *
     * @see #getSystemService
     * @see android.accounts.AccountManager
     */


     /*******************************下面还有很多很多*************************/


   6.以上可以看出，如果在Activity和ContextThemeWrapper不能帮你取到服务的话，最终会回到Context来一个"大搜查"。





ServiceManager 管理的服务

   ServiceManager是一个独立的进程!这里特别强调，它是一个进程!作用就跟它的名称一样，管理各种系统服务，如下图所示：
![](http://img.blog.csdn.net/20141112182131029?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMTY0MzA3MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)





   其实ServiceManager本身也是一个Service，Framework提供了一个系统函数，可以获取ServiceManager
对应的Binder引用，那就是BinderInternal.getContextObject()。该静态函数返回ServiceManager后，就
可以通过ServiceManger提供的方法获取其他服务Service的Binder引用。这种设计模式就好比：ServiceManager
是一个公司的总机，这个总计的号码是公开的，系统中任何进程都可以使用BinderInternal.getContextObject()
获取该总机的Binder对象，而当前用户想联系公司的其他人（服务）的时候，则要经过总机再获取到分机的号码。
这种设计模式的优点就是：系统中仅仅只暴漏一个全局Binder引用，那就是ServiceManager，而其他服务则可以
隐藏起来，从而有助于系统服务的扩展，以及调用系统服务的安全检查。其他系统服务在启动的时候，首先把自己
的Binder对象传递给ServiceManager，就是所谓的注册（addService）。


BinderInternal.getContextObject() ->ServiceManager ->其它Service的Binder引用

那么我们来看一看ServiceManager.getService()的源码：

	public static IBinder getService(String name){
		try{
			IBinder service = sCache.get(name);
			if(servie != null){
				return service
			}else{
				return getIServiceManager().getService(name);
			}
		}catch (RemoteException e) {
			Log.e(TAG, "error in getService",e);
		}
		return null;
	}


以上片段源码，sCache.get(name) 缓存中查看是否有对应的 Binder 对象，有则返回，没有则调用
return getIServiceManager.getService(name)；第一个getIServiceManager()即用于返回系统中
唯一的ServiceManager对应的Binder。


接着看看getIServiceManager()：

	private static IServiceManager getIServiceManager() {
		if (sServiceManager != null) {
			return sServiceManager;
		}

		// Find the service manager
		sServiceManager = ServiceManagerNative.asInterface(BinderInternal.getContextObject());
		return sServiceManager;
	}




以上源码中，BinderInternal.getContextObject()静态函数用于返回ServiceManager对应的全局Binder
对象，该函数不需要任何参数，因为它的作用所致是固定的。

其他所有通过ServiceManager获取的系统服务的过程与以上基本相似，所不同的就是传递给ServiceManager
的服务名称不同，因为ServiceManager正是按照服务的名称（String）来保存不同的Binder。




Android 中的 Manager

在Android中，Manager的含义应该翻译为“经纪人”，Manager所管理的对象时服务本身，因为每一
个具体的服务一般都会提供多个API接口，而Manager所管理的就是这些API。客户端一般不能直接通过
Binder访问的服务，而是要经过一个Manager，对应的Manager类对客户端是可见的，而远程的服务类
对客户端是隐藏的。

而这些Manager的类内部都会存在一个远程服务Binder的变量，而且在一般情况下，这些Manager的构造
函数中都会包含这个Binder对象。即先通过ServiceManager获取远程服务的Binder引用，然后使用这个Binder
引用构造一个客户端本地可以访问的经纪人，然后客户端就可以通过经纪人间接访问远程的服务了。

这种设计的作用是为了屏蔽客户端直接访问远程服务，从而可以给应用程序提供灵活的、可控的API接口，比
如Ams。系统不希望用户直接去访问Ams，而是经过ActivityManager类去访问，而ActivityManager内部则
提供一些更具可操作性的数据结构，比如RecentTaskInfo数据结构封装了最近访问过的Task列表；MemoryInfo
数据类封装了和内存相关的信息。如下（RecentTaskInfo和MemoryInfo的源代码）：

ActivityManager 中的 RecentTaskInfo 类 源码：

    /**
     * Information you can retrieve about tasks that the user has most recently
     * started or visited.
     */
    public static class RecentTaskInfo implements Parcelable {
        /**
         * If this task is currently running, this is the identifier for it.
         * If it is not running, this will be -1.
         */
        public int id;

        /**
         * The true identifier of this task, valid even if it is not running.
         */
        public int persistentId;

        /**
         * The original Intent used to launch the task.  You can use this
         * Intent to re-launch the task (if it is no longer running) or bring
         * the current task to the front.
         */
        public Intent baseIntent;

        /**
         * If this task was started from an alias, this is the actual
         * activity component that was initially started; the component of
         * the baseIntent in this case is the name of the actual activity
         * implementation that the alias referred to.  Otherwise, this is null.
         */
        public ComponentName origActivity;

        /**
         * Description of the task's last state.
         */
        public CharSequence description;

        /**
         * The id of the ActivityStack this Task was on most recently.
         * @hide
         */
        public int stackId;

        /**
         * The id of the user the task was running as.
         * @hide
         */
        public int userId;

        /**
         * The first time this task was active.
         * @hide
         */
        public long firstActiveTime;

        /**
         * The last time this task was active.
         * @hide
         */
        public long lastActiveTime;

        /**
         * The recent activity values for the highest activity in the stack to have set the values.
         * {@link Activity#setTaskDescription(android.app.ActivityManager.TaskDescription)}.
         */
        public TaskDescription taskDescription;

        /**
         * Task affiliation for grouping with other tasks.
         */
        public int affiliatedTaskId;

        /**
         * Task affiliation color of the source task with the affiliated task id.
         *
         * @hide
         */
        public int affiliatedTaskColor;

        public RecentTaskInfo() {
        }

        @Override
        public int describeContents() {
            return 0;
        }

        @Override
        public void writeToParcel(Parcel dest, int flags) {
            dest.writeInt(id);
            dest.writeInt(persistentId);
            if (baseIntent != null) {
                dest.writeInt(1);
                baseIntent.writeToParcel(dest, 0);
            } else {
                dest.writeInt(0);
            }
            ComponentName.writeToParcel(origActivity, dest);
            TextUtils.writeToParcel(description, dest,
                    Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
            if (taskDescription != null) {
                dest.writeInt(1);
                taskDescription.writeToParcel(dest, 0);
            } else {
                dest.writeInt(0);
            }
            dest.writeInt(stackId);
            dest.writeInt(userId);
            dest.writeLong(firstActiveTime);
            dest.writeLong(lastActiveTime);
            dest.writeInt(affiliatedTaskId);
            dest.writeInt(affiliatedTaskColor);
        }

        public void readFromParcel(Parcel source) {
            id = source.readInt();
            persistentId = source.readInt();
            baseIntent = source.readInt() > 0 ? Intent.CREATOR.createFromParcel(source) : null;
            origActivity = ComponentName.readFromParcel(source);
            description = TextUtils.CHAR_SEQUENCE_CREATOR.createFromParcel(source);
            taskDescription = source.readInt() > 0 ?
                    TaskDescription.CREATOR.createFromParcel(source) : null;
            stackId = source.readInt();
            userId = source.readInt();
            firstActiveTime = source.readLong();
            lastActiveTime = source.readLong();
            affiliatedTaskId = source.readInt();
            affiliatedTaskColor = source.readInt();
        }

        public static final Creator<RecentTaskInfo> CREATOR
                = new Creator<RecentTaskInfo>() {
            public RecentTaskInfo createFromParcel(Parcel source) {
                return new RecentTaskInfo(source);
            }
            public RecentTaskInfo[] newArray(int size) {
                return new RecentTaskInfo[size];
            }
        };

        private RecentTaskInfo(Parcel source) {
            readFromParcel(source);
        }
    }


ActivityManager 的 MemoryInfo类 源代码 ：

    /**
     * Information you can retrieve about the available memory through
     * {@link ActivityManager#getMemoryInfo}.
     */
    public static class MemoryInfo implements Parcelable {
        /**
         * The available memory on the system.  This number should not
         * be considered absolute: due to the nature of the kernel, a significant
         * portion of this memory is actually in use and needed for the overall
         * system to run well.
         */
        public long availMem;

        /**
         * The total memory accessible by the kernel.  This is basically the
         * RAM size of the device, not including below-kernel fixed allocations
         * like DMA buffers, RAM for the baseband CPU, etc.
         */
        public long totalMem;

        /**
         * The threshold of {@link #availMem} at which we consider memory to be
         * low and start killing background services and other non-extraneous
         * processes.
         */
        public long threshold;

        /**
         * Set to true if the system considers itself to currently be in a low
         * memory situation.
         */
        public boolean lowMemory;

        /** @hide */
        public long hiddenAppThreshold;
        /** @hide */
        public long secondaryServerThreshold;
        /** @hide */
        public long visibleAppThreshold;
        /** @hide */
        public long foregroundAppThreshold;

        public MemoryInfo() {
        }

        public int describeContents() {
            return 0;
        }

        public void writeToParcel(Parcel dest, int flags) {
            dest.writeLong(availMem);
            dest.writeLong(totalMem);
            dest.writeLong(threshold);
            dest.writeInt(lowMemory ? 1 : 0);
            dest.writeLong(hiddenAppThreshold);
            dest.writeLong(secondaryServerThreshold);
            dest.writeLong(visibleAppThreshold);
            dest.writeLong(foregroundAppThreshold);
        }

        public void readFromParcel(Parcel source) {
            availMem = source.readLong();
            totalMem = source.readLong();
            threshold = source.readLong();
            lowMemory = source.readInt() != 0;
            hiddenAppThreshold = source.readLong();
            secondaryServerThreshold = source.readLong();
            visibleAppThreshold = source.readLong();
            foregroundAppThreshold = source.readLong();
        }

        public static final Creator<MemoryInfo> CREATOR
                = new Creator<MemoryInfo>() {
            public MemoryInfo createFromParcel(Parcel source) {
                return new MemoryInfo(source);
            }
            public MemoryInfo[] newArray(int size) {
                return new MemoryInfo[size];
            }
        };

        private MemoryInfo(Parcel source) {
            readFromParcel(source);
        }
    }



这种通过Manager 访问远程服务的模型如下图：


![](http://img.blog.csdn.net/20141120163612296?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMTY0MzA3MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
