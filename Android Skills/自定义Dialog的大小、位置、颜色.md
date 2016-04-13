---
title: Android自定义对话框(Dialog)位置,大小
date: 2015-06-21 22:32:17
tags: Dialog
categories: 技术
---




> 代码:

    package angel.devil;
    import android.app.Activity;
    import android.app.Dialog;
    import android.os.Bundle;
    import android.view.Gravity;
    import android.view.Window;
    import android.view.WindowManager;

    public class DialogDemoActivity extends Activity {
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);
    Dialog dialog = new Dialog(this);

    // setContentView可以设置为一个View也可以简单地指定资源ID
    // LayoutInflater
    // li=(LayoutInflater)getSystemService(LAYOUT_INFLATER_SERVICE);
    // View v=li.inflate(R.layout.dialog_layout, null);
    // dialog.setContentView(v);
    dialog.setContentView(R.layout.dialog_layout);

    dialog.setTitle("Custom Dialog");

---
<!--more-->

    /*
    * 获取圣诞框的窗口对象及参数对象以修改对话框的布局设置,
    * 可以直接调用getWindow(),表示获得这个Activity的Window
    * 对象,这样这可以以同样的方式改变这个Activity的属性.
    */
    Window dialogWindow = dialog.getWindow();
    WindowManager.LayoutParams lp = dialogWindow.getAttributes();
    dialogWindow.setGravity(Gravity.LEFT | Gravity.TOP);

    /*
    * lp.x与lp.y表示相对于原始位置的偏移.
    * 当参数值包含Gravity.LEFT时,对话框出现在左边,所以lp.x就表示相对左边的偏移,负值忽略.
    * 当参数值包含Gravity.RIGHT时,对话框出现在右边,所以lp.x就表示相对右边的偏移,负值忽略.
    * 当参数值包含Gravity.TOP时,对话框出现在上边,所以lp.y就表示相对上边的偏移,负值忽略.
    * 当参数值包含Gravity.BOTTOM时,对话框出现在下边,所以lp.y就表示相对下边的偏移,负值忽略.
    * 当参数值包含Gravity.CENTER_HORIZONTAL时
    * ,对话框水平居中,所以lp.x就表示在水平居中的位置移动lp.x像素,正值向右移动,负值向左移动.
    * 当参数值包含Gravity.CENTER_VERTICAL时
    * ,对话框垂直居中,所以lp.y就表示在垂直居中的位置移动lp.y像素,正值向右移动,负值向左移动.
    * gravity的默认值为Gravity.CENTER,即Gravity.CENTER_HORIZONTAL |
    * Gravity.CENTER_VERTICAL.
    *
    * 本来setGravity的参数值为Gravity.LEFT | Gravity.TOP时对话框应出现在程序的左上角,但在
    * 我手机上测试时发现距左边与上边都有一小段距离,而且垂直坐标把程序标题栏也计算在内了,
    * Gravity.LEFT, Gravity.TOP, Gravity.BOTTOM与Gravity.RIGHT都是如此,据边界有一小段距离
    */
    lp.x = 100; // 新位置X坐标
    lp.y = 100; // 新位置Y坐标
    lp.width = 300; // 宽度
    lp.height = 300; // 高度
    lp.alpha = 0.7f; // 透明度

    // 当Window的Attributes改变时系统会调用此函数,可以直接调用以应用上面对窗口参数的更改,也可以用setAttributes
    // dialog.onWindowAttributesChanged(lp);
    dialogWindow.setAttributes(lp);

    /*
    * 将对话框的大小按屏幕大小的百分比设置
    */
    //        WindowManager m = getWindowManager();
    //        Display d = m.getDefaultDisplay(); // 获取屏幕宽、高用
    //        WindowManager.LayoutParams p = dialogWindow.getAttributes(); // 获取对话框当前的参数值
    //        p.height = (int) (d.getHeight() * ); // 高度设置为屏幕的0.6
    //        p.width = (int) (d.getWidth() * 0.65); // 宽度设置为屏幕的0.65
    //        dialogWindow.setAttributes(p);

    dialog.show();

    }
    }


>布局文件:
main.xml

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:background="#00FF00"
    android:orientation="vertical" >

    <TextView
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:text="@string/hello" />

    </LinearLayout>

    dialog_layout.xml

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout_root"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal"
    android:padding="10dp" >

    <ImageView
    android:id="@+id/image"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginRight="10dp"
    android:src="@drawable/ic_launcher" />

    <TextView
    android:id="@+id/text"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="A Dialog"
    android:textColor="#FFF" />

    </LinearLayout>



> 取消自定义dialog的标题栏

Android中取消自定义dialog的标题栏, 只需在

    dialog.setContentView(R.layout.popwin_chooseversion);
前面加一句:

    dialog.requestWindowFeature(Window.FEATURE_NO_TITLE);
加在这句之后会产生异常.

还要导入

    import android.view.Window;
包.




### android 如何让自定义dialog的宽度充满整个屏幕

#### 方案：
  通过设置Dialog的样式实现

##### 步骤：
> 1、添加style

    <style name="Dialog_FS">
    <item name="android:windowFullscreen">true</item> //设置填充父窗体
    <item name="android:windowNoTitle">true</item> //设置隐藏标题栏
    </style>

> 2、代码里面设置dialog的样式

    Dialog dialog = new Dialog(this,R.style.Dialog_FS); //设置全屏样式
    dialog.setContentView(R.layout.main); //设置dialog的布局
    dialog.show();//显示dialog界面

> 3.style文件具体

      <style name="iphone_progress_dialog" parent="@android:style/Theme.Dialog">
         <item name="android:windowFrame">@null</item> <!--Dialog的windowFrame框为无--> 
         <item name="android:windowIsFloating">true</item><!-- 是否漂现在activity上-->
         <item name="android:windowIsTranslucent">true</item><!-- 是否半透明 -->
         <item name="android:windowNoTitle">true</item>
         <item name="android:backgroundDimEnabled">false</item><!-- dim:模糊的 阴影效果 -->
         <item name="android:windowBackground">@drawable/load_bg</item><!-- 背景图片的大小也影响窗口的大小 -->
    </style>
