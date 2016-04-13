---
title: widget用法
date: 2014-09-18 02:24:16
tags: widget
categories: 技术
---


September 23, 2015 8:21 PM

创建AppWidgetProvider的子类

	public class MyAppWidgetProvider extends AppWidgetProvider {
    @Override
    public void onEnabled(Context context) {
        // 第一次创建执行
        // 服务监控进程状态
        Intent service = new Intent(context,TaskWidgetService.class);
        context.startService(service);
        super.onEnabled(context);
    }

    @Override
    public void onDisabled(Context context) {
        //删除最后一个执行
        Intent service = new Intent(context,TaskWidgetService.class);
        context.stopService(service);
        super.onDisabled(context);
    }
	}
<!--more-->
创建xml文件夹，创建info的xml文件

	<?xml version="1.0" encoding="utf-8"?>
	<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
		android:initialLayout="@layout/process_widget"
		android:minHeight="72.0dip"
		android:minWidth="294.0dip"
		android:updatePeriodMillis="0" />
配置清单文件

	<receiver android:name="ExampleAppWidgetProvider" >
		<intent-filter>
		<action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
		</intent-filter>
		<meta-data android:name="android.appwidget.provider"
		android:resource="@xml/example_appwidget_info" />
	</receiver>
广播一定要在清单文件中注册

	 <receiver android:name="com.itheima.mobilesafe13.receiver.WidgetClearTaskReceiver">
		<intent-filter >然后在显示Toast的地方
			<action android:name="widget.clear.task"></action>
			</intent-filter>
	</receiver>
	<receiver android:name="com.itheima.mobilesafe13.receiver.MyAppWidgetProvider" >
		<intent-filter>
			<action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
		</intent-filter>
		<meta-data
			android:name="android.appwidget.provider"
			android:resource="@xml/process_widget_provider" />
		</receiver>
使用widget

	/**
	* @author Administrator
	* @desc 清理进程的widget的服务
	*/
	public class TaskWidgetService extends Service {

		private AppWidgetManager mAWM;

		@Override
		public IBinder onBind(Intent intent) {
			// TODO Auto-generated method stub
			return null;
		}

		@Override
		public void onCreate() {
			mAWM = AppWidgetManager.getInstance(getApplicationContext());
			System.out.println("widget  service create");

			Timer timer = new Timer();
			TimerTask task = new TimerTask() {

				@Override
				public void run() {
					updateWidgetMessage();

				}
			};
			timer.schedule(task, 0 , 1000 * 2);
			super.onCreate();
		}

		protected void updateWidgetMessage() {
			ComponentName provider = new ComponentName(getApplicationContext(), MyAppWidgetProvider.class);
			RemoteViews views = new RemoteViews(getPackageName(), R.layout.process_widget);
			views.setTextViewText(R.id.tv_process_count, "运行中的软件:" + TaskInfoUtils.getAllRunningAppInfos(getApplicationContext()).size());
			views.setTextViewText(R.id.tv_process_memory, "可用内存:" + Formatter.formatFileSize(getApplicationContext(),
					TaskInfoUtils.getAvailMem(getApplicationContext())));

			Intent intent = new Intent();
			intent.setAction("widget.clear.task");
			PendingIntent pendingIntent = PendingIntent.getBroadcast(getApplicationContext(), 0, intent , 0);
			views.setOnClickPendingIntent(R.id.btn_clear, pendingIntent );
			// 更新widget界面
			mAWM.updateAppWidget(provider, views);
		}

		@Override
		public void onDestroy() {
			// TODO Auto-generated method stub
			System.out.println("widget  service stop");
			super.onDestroy();
		}

	}
