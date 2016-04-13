---
title: Android开发代码片段
date: 2016-03-21 20:33:02
tags: 写作
categories: 技术
---

### 1、获取电源锁


	public static void acquireWakeLock() {
		unlockKeyBoard();
		try {
			if (null == mWakeLock) {
				PowerManager pm = (PowerManager) BaseApplication.getInstance()
						.getSystemService(Context.POWER_SERVICE);
				try {
					mWakeLock = pm.newWakeLock(PowerManager.FULL_WAKE_LOCK
							| PowerManager.ACQUIRE_CAUSES_WAKEUP
							| PowerManager.ON_AFTER_RELEASE, "PushMessageReceiver"); // |
																						// PowerManager.ON_AFTER_RELEASE
				} catch (Exception e) {
					DbUtils.exceptionHandler(e);
				}
			}
			if (null != mWakeLock) {
				mWakeLock.acquire();
			}
		} catch (Exception e) {
			DbUtils.exceptionHandler(e);
		}
	}

<!--more-->
### 2、释放设备电源锁


	public static void releaseWakeLock() {
		try {
			if (null != mWakeLock) {
                mWakeLock.release();
                mWakeLock = null;
            }
		} catch (Exception e) {
			DbUtils.exceptionHandler(e);
		}
	}


### 3、解锁键盘

     
	public static void unlockKeyBoard () {
		KeyguardManager km = (KeyguardManager)ApplicationContext.getInstance().getContext().getSystemService(Context.KEYGUARD_SERVICE);
		km.newKeyguardLock("Tag For Debug").disableKeyguard();
	}
    
    
    
4、获取当前程序版本名

    /**  
    * 返回当前程序版本名  
    */    
    public static String getAppVersionName(Context context) {    
        String versionName = "";    
        try {    
            // ---get the package info---    
            PackageManager pm = context.getPackageManager();    
    
            PackageInfo pi = pm.getPackageInfo(context.getPackageName(), 0);    
            versionName = pi.versionName;    //版本名称
            versioncode = pi.versionCode;  //版本号
            if (versionName == null || versionName.length() <= 0) {    
                return "";    
            }    
        } catch (Exception e) {    
            Log.e("VersionInfo", "Exception", e);    
        }    
        return versionName;    
    } 
   
### 5、获取当前应用的版本号

    public static String getVersionName() throws Exception  
    {  
            // 获取packagemanager的实例  
            PackageManager packageManager = getPackageManager();  
            // getPackageName()是你当前类的包名，0代表是获取版本信息  
            PackageInfo packInfo = packageManager.getPackageInfo(getPackageName(),0);  
            String version = packInfo.versionName;  
            return version;  
    }
    
### 6、获取当前系统的版本号

       /** 
        * 手机系统版本 
        */  
        public static String getSdkVersion() {  
            return android.os.Build.VERSION.RELEASE;  
        }  
        
### 7、当我们点击某个话题的选项卡，会弹出一个popupwindow，里面有诸如 评论、回复的选项，你点击这个选项的时候，需要定位到EditText编辑框，并且自动弹出输入法。可以考虑如下方法：

    // 获取编辑框焦点
    editText.setFocusable(true);
    //打开软键盘
    InputMethodManager imm = (InputMethodManager) ctx
    .getSystemService(Context.INPUT_METHOD_SERVICE);
    imm.toggleSoftInput(0, InputMethodManager.HIDE_NOT_ALWAYS);

    //关闭软键盘
    imm.hideSoftInputFromWindow(editText.getWindowToken(), 0);
    
### 8、EditText软键盘
       
        //打开软键盘
        et_feedback_content.setFocusable(true);
        getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_STATE_ALWAYS_VISIBLE);
        //关闭软键盘
        ((InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE)).hideSoftInputFromWindow(input.getWindowToken(), 0); 
        
### 9、设置全屏的Dialog

    
   
    //在代码里设置Dialog的Theme
    Dialog dialog = new Dialog(this, R.style.Dialog_Fullscreen);  
    dialog.setContentView(R.layout.main);  
    dialog.show(); 
    
    //设置style
     <style name="Dialog_Fullscreen"> 
       <item name="android:windowFullscreen">true</item> 
       <item name="android:windowNoTitle">true</item>  
    </style>   
  
  
  
  
### 10、设置全屏的Dialog（二）
  
我们也可以自定义Dialog，首先继承Dialig，然后再构造函数中添加

    super(context, android.R.style.Theme); 
    setOwnerActivity((Activity)context);  
   
  
    
### 11、设置全屏Dialog（三）

首先介绍一个方法：getDecorView()

decorView是window中的最顶层view，可以从window中获取到decorView，然后decorView有个getWindowVisibleDisplayFrame方法可以获取到程序显示的区域，包括标题栏，但不包括状态栏。 于是，我们就可以算出状态栏的高度了。

    Rect frame = new Rect();
    getWindow().getDecorView().getWindowVisibleDisplayFrame(frame);
    int statusBarHeight = frame.top;
    
同样我们获取标题栏的高度

    getWindow().findViewById(Window.ID_ANDROID_CONTENT)这个方法获取到的view就是程序不包括标题栏的部分，然后就可以知道标题栏的高度了，代码如下：
    int contentTop = getWindow().findViewById(Window.ID_ANDROID_CONTENT).getTop();         
    int titleBarHeight = contentTop - statusBarHeight; //statusBarHeight是上面所求的状态栏的高度

最后：知道上述原理，我们就可以设置我们的Dialog和activity一样大了，Java代码如下：

    final Dialog dialog = new Dialog(WenDetailActivity.this, R.style.popupDialog);
            dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
            dialog.setContentView(R.layout.wen_cover_pager);
            dialog.setCanceledOnTouchOutside(false);
            dialog.setCancelable(false);
            WindowManager.LayoutParams lay = dialog.getWindow().getAttributes();
            DisplayMetrics dm = new DisplayMetrics();
            getWindowManager().getDefaultDisplay().getMetrics(dm);
            Rect rect = new Rect();
            View view = getWindow().getDecorView();//decorView是window中的最顶层view，可以从window中获取到decorView
            view.getWindowVisibleDisplayFrame(rect);
            lay.height = dm.heightPixels - rect.top;
            lay.width = dm.widthPixels;
            
style.xml如下：

    <style name="popupDialog" parent="@android:style/Theme.Dialog">
            <item name="android:windowBackground">@drawable/filled_activity_bg</item>
            <item name="android:backgroundDimEnabled">false</item>
            <item name="android:windowIsFloating">true</item>
            <item name="android:windowIsTranslucent">true</item>
            <item name="android:windowNoTitle">true</item>
            <item name="android:windowContentOverlay">@null</item>
            <!--<item name="android:windowAnimationStyle">@style/dialog_animation</item>-->
            <item name="android:colorBackgroundCacheHint">@null</item>
            <item name="android:backgroundDimAmount">0.6</item><!-- 灰度 -->
            <!--<item name="android:windowFullscreen">true</item>-->
     </style>
     
 ### 12、利用代码清除App的数据 
    /** 
    * 利用代码清除App的数据 
    * 平常我们在清除App的数据时,多半在设置中找到对应的App 
    * 然后选择其清除数据.下面给出代码实现. 
    *  
    * 注意事项: 
    * 1 设备需要root 
    * 2 该示例中删除的是系统级应用 
    * 2 注意在命令的末尾需要加上换行\n 
    *   这就相当于我们平时在Dos中输入命令后再换行一样. 
    *   否则命令不会被执行. 
    */  
    private void cleanData(String packageName){  
        try {  
            Process su= Runtime.getRuntime().exec("su");  
            String cmd = "cd /data/data/"+packageName+";"+"rm -r `ls|grep -v lib`";  
            System.out.println("------cmd="+cmd);  
            cmd = cmd + "\n exit\n";  
            su.getOutputStream().write(cmd.getBytes());  
            if ((su.waitFor() != 0)) {  
                throw new SecurityException();  
            }  
        } catch (Exception e) {  
            System.out.println("---> 9527 清除数据时 e="+e.toString());  
        }  
          
    } 
    
    
 ### 13、清除内/外缓存，清除数据库，清除sharedPreference，清除files和清除自定义目录

    /** 文 件 名:  DataCleanManager.java  * 描    述:  主要功能有清除内/外缓存，清除数据库，清除sharedPreference，清除files和清除自定义目录  */
    import java.io.File;
    import android.content.Context;
    import android.os.Environment;

    /** * 本应用数据清除管理器 */
    public class DataCleanManager {
    /** * 清除本应用内部缓存(/data/data/com.xxx.xxx/cache) * * @param context */
    public static void cleanInternalCache(Context context) {
        deleteFilesByDirectory(context.getCacheDir());
    }

    /** * 清除本应用所有数据库(/data/data/com.xxx.xxx/databases) * * @param context */
    public static void cleanDatabases(Context context) {
        deleteFilesByDirectory(new File("/data/data/"
                + context.getPackageName() + "/databases"));
    }

    /**
     * * 清除本应用SharedPreference(/data/data/com.xxx.xxx/shared_prefs) * * @param
     * context
     */
    public static void cleanSharedPreference(Context context) {
        deleteFilesByDirectory(new File("/data/data/"
                + context.getPackageName() + "/shared_prefs"));
    }

    /** * 按名字清除本应用数据库 * * @param context * @param dbName */
    public static void cleanDatabaseByName(Context context, String dbName) {
        context.deleteDatabase(dbName);
    }

    /** * 清除/data/data/com.xxx.xxx/files下的内容 * * @param context */
    public static void cleanFiles(Context context) {
        deleteFilesByDirectory(context.getFilesDir());
    }

    /**
     * * 清除外部cache下的内容(/mnt/sdcard/android/data/com.xxx.xxx/cache) * * @param
     * context
     */
    public static void cleanExternalCache(Context context) {
        if (Environment.getExternalStorageState().equals(
                Environment.MEDIA_MOUNTED)) {
            deleteFilesByDirectory(context.getExternalCacheDir());
        }
    }

    /** * 清除自定义路径下的文件，使用需小心，请不要误删。而且只支持目录下的文件删除 * * @param filePath */
    public static void cleanCustomCache(String filePath) {
        deleteFilesByDirectory(new File(filePath));
    }

    /** * 清除本应用所有的数据 * * @param context * @param filepath */
    public static void cleanApplicationData(Context context, String... filepath) {
        cleanInternalCache(context);
        cleanExternalCache(context);
        cleanDatabases(context);
        cleanSharedPreference(context);
        cleanFiles(context);
        for (String filePath : filepath) {
            cleanCustomCache(filePath);
        }
    }

    /** * 删除方法 这里只会删除某个文件夹下的文件，如果传入的directory是个文件，将不做处理 * * @param directory */
    private static void deleteFilesByDirectory(File directory) {
        if (directory != null && directory.exists() && directory.isDirectory()) {
            for (File item : directory.listFiles()) {
                item.delete();
            }
        }
    }
    }


### 14、Listview测量高度 （放在设置完适配器之后进行测量，放在之前，没有效果）

       /**
        * 设置高度
        */
        private void setHeight(ListView listView) {
            // 获取listView的适配器
            ListAdapter adapter = listView.getAdapter();
            // 获取视图的个数
            int count = adapter.getCount();
            // 总高度
            int totalHeight = 0;
            // 循环获取视图
            for (int i = 0; i < count; i++) {
                // 通过i获取每个视图
                View view = adapter.getView(i, null, listView);
                // 重新测量view的高度
                view.measure(MeasureSpec.UNSPECIFIED, MeasureSpec.UNSPECIFIED);
                // 获取测量后的高度添加到总高度
                totalHeight += view.getMeasuredHeight();
            }
            // 总高度加上所有分割线的总高度
            totalHeight += listView.getDividerHeight() * (count - 1);
            // 获取listView的布局属性
            LayoutParams params = listView.getLayoutParams();
            // 设置高度
            params.height = totalHeight;
            // 重新设置布局属性
            listView.setLayoutParams(params);
        }

