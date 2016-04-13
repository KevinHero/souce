---
title: Android 开发规范
date: 2014-02-08 15:13:06
tags: 开发规范
categories: 技术
---

## 命名规范
### 包命名规范

    采用反域名命名规则，
    包名全部小写，
    连续的单词只是简单地连接起来，
    不使用下划线，
    一级包名为com，
    二级包名为xxx（可以是公司域名或者个人命名），
    三级包名根据应用进行命名，
    四级包名为模块名或层级名。如：
	com.weicheche.android.activity | com.weicheche.android.adapter

<!--more-->
### JAVA类命名规范

* 采用大驼峰式命名法，尽量避免缩写，除非该缩写是众所周知的，比如HTML，URL,如果类名称包含单词缩写，则单词缩写的每个字母均应大写。如：


    Product | ProductManager | ProductListActivity | ProductListAdapter | JsonHTTPSRequest


### 接口命名规范

* 命名规则与类一样采用大驼峰命名法，多以able或ible结尾。例如：


    interface Runable | interface Accessible



###  成员变量命名规范

* 采用小驼峰命名法。


### 临时变量命名

* 使用标准的Java命名方法，不推荐使用Google的m命名法。例如：


    private String userName; 而不推荐使用 private String mUserName;


###   常量命名

* 常量使用全大写字母加下划线的方式命名。例如：


    public static final String TAG = "tag";


### 控件实例命名

* 类中控件名称必须与xml布局id保持一致(可以去掉{module_name})。例如:

在布局文件中 Button 的id为:


    android:id="@+id/btn_pay" -->
    private Button btn_pay;


### 方法命名规范

* 动词或动名词，采用小驼峰命名法。例如：


    run(); | onCreate(); | syncProducts();


### 布局文件(Layout)命名规范

* 全部小写，采用下划线命名法。其中{module_name}为业务模块或是功能模块等模块化的名称或简称。


	activity layout： {module_name}_activity_{名称} 例如：
	crm_activity_main.xml | crm_activity_shopping.xml

	fragment layout:{module_name}_fragment_{名称} 例如：
	crm_fragment_main.xml | crm_fragment_shopping.xml

	Dialog layout: {module_name}_dialog_{名称} 例如：
	crm_dialog_loading.xml

	列表项布局命名：{module_name}_list_item_{名称} 例如：
	crm_listitem_customer.xml

	包含项布局命名：include_{名称} 例如：
	include_head.xml

	adapter的子布局： {module_name}_item_{名称} 例如：
	qz_item_order.xml

	widget layout： {module_name}_widget_{名称} 例如：
	crm_widget_shopping_detail.xml


### 资源id命名规范

* 命名模式为：{view缩写}_{module_name}_{view的逻辑名称}，如：


顾客管理CRM模块布局 LinearLayout 的布局id –>ll_crm_content模块简称为qz的 ImageView 的布局id –> iv_qz_photo


常见控件View与其缩写对照参考表如下：

<img src="http://img.blog.csdn.net/20150408172729880"/>

### 图片资源文件命名规范

* 图标命名：


    {module_name}_ic_{名称} 例如：
    crm_ic_app.png



* 背景图片命名：


    {module_name}_bg_{名称} 例如：
    crm_bg_navbar_highlight_normal.9.png


* 按钮Button命名：


    {module_name}_btn_{名称} 例如：
    crm_btn_login_normal.9.png


* 按钮checkbox图片命名：


    {module_name}_checkbox_{名称} 例如：
    crm_checkbox_cart_true.png


* 其他图片命名：


    {module_name}_icon_{名称} 例如：
    qz_icon_blue_circle.png


## 代码风格

### 大括号问题

* 风格一


    if (hasMoney())
    {
    }
    else
    {
    }


* 风格二


    if (hasMoney()) {
    } else {
    }


### 空格问题

    if else | while | 运算符两端 等后面需用空格隔开。例如：

* 规范的编写方式：


    if (hasMoney()) {
    } else {
    }
    for (int i = 0; i &lt; 10; i++) {
    }


* 不规范的编写方式：


    if(hasMoney()){
    }else{
    }
    for(int i=0; i&lt;10;i++){
    }


### 方法参数

* 当方法参数数量过多时，需进行换行处理.

注释

* 必须要对所有实例变量、类常量进行注释说明 例如：


  	// 用户姓名
  	private String userName


* 必须对所有的类、接口进行注释说明 例如：


      /**
    * Activity基类
    *
    */
    public class BaseActivity extends Activity
    {

    }


* 必须对所有的方法进行注释说明 例如:


    /**
    * 请求
    *
    * @param path 路径
    * @param generalParams 基本参数
    * @param businessParams 业务参数
    * @return 请求结果
    * @throws ApiException 请求错误则返回该异常
    */
    public Map<String, Object> request (String path,
                  Map<String, Object> generalParams,
                  Map<String, Object> businessParams) throws ApiException {

       return null;
    }


### 资源文件 Resources

* 命名 遵循前缀表明类型的习惯，形如type_foo_bar.xml。例如：


    fragment_contact_details.xml,
    view_primary_button.xml,
    activity_main.xml.


组织布局文件 若果你不确定如何排版一个布局文件，遵循一下规则可能会有帮助。


每一个属性一行，缩进4个空格
android:id 总是作为第一个属性
android:layout_**** 属性在上边
style 属性在底部
关闭标签/>单独起一行，有助于调整和添加新的属性
考虑使用Designtime attributes 设计时布局属性，Android Studio已经提供支持，而不是硬编码android:text
(译者注：墙内也可以参考stormzhang的这篇博客链接)。




    <?xml version="1.0" encoding="utf-8"?>

    <LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        >

        <TextView
            android:id="@+id/name"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_alignParentRight="true"
            android:text="@string/name"
            style="@style/FancyText"
            />
        <include layout="@layout/reusable_part" />

    </LinearLayout>


作为一个经验法则,android:layout_****属性应该在 layout XML 中定义,同时其它属性android:**** 应放在 styler XML中。此规则也有例外，不过大体工作

的很好。这个思想整体是保持layout属性(positioning, margin, sizing) 和content属性在布局文件中，同时将所有的外观细节属性（colors, padding, font）放
在style文件中。

例外有以下这些:


    android:id 明显应该在layout文件中
    layout文件中android:orientation对于一个LinearLayout布局通常更有意义
    android:text 由于是定义内容，应该放在layout文件中
    有时候将android:layout_width 和 android:layout_height属性放到一个style中作为一个通用的风格中更有意义，但是默认情况下这些应该放到layout文件中。


* 使用styles

几乎每个项目都需要适当的使用style文件，因为对于一个视图来说有一个重复的外观是很常见的。

在应用中对于大多数文本内容，最起码你应该有一个通用的style文件，例如：


    <style name="ContentText">
        <item name="android:textSize">@dimen/font_normal</item>
        <item name="android:textColor">@color/basic_black</item>
    </style>

    应用到TextView 中:

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/price"
        style="@style/ContentText"
        />


 你或许需要为按钮控件做同样的事情，不要停止在那里。将一组相关的和重复android:****的属性放到一个通用的style中。
将一个大的style文件分割成多个文件 你可以有多个styles.xml 文件。

    AndroidSDK支持其它文件，styles这个文件名称并没有作用，起作用的是在文件里xml的<style>标签。因此你可以有多个style文件styles.xml,style_home.xml,style_item_details.xml,styles_forms.xml。不用于资源文件路径需要为系统构建起的有意义，在res/values目录下的文件可以任意命名。

* colors.xml是一个调色板 在你的colors.xml文件中应该只是映射颜色的名称一个RGBA值，而没有其它的。不要使用它为不同的按钮来定义RGBA值。

* 不要这样做


    <resources>
        <color name="button_foreground">#FFFFFF</color>
        <color name="button_background">#2A91BD</color>
        <color name="comment_background_inactive">#5F5F5F</color>
        <color name="comment_background_active">#939393</color>
        <color name="comment_foreground">#FFFFFF</color>
        <color name="comment_foreground_important">#FF9D2F</color>
        ...

        <color name="comment_shadow">#323232</color>


使用这种格式，你会非常容易的开始重复定义RGBA值，这使如果需要改变基本色变的很复杂。同时，这些定义是跟一些环境关联起来的，如button或者comment,

应该放到一个按钮风格中，而不是在color.xml文件中。

* 相反，这样做:



      <resources>
          <!-- grayscale -->
          <color name="white"     >#FFFFFF</color>
          <color name="gray_light">#DBDBDB</color>
          <color name="gray"      >#939393</color>
          <color name="gray_dark" >#5F5F5F</color>
          <color name="black"     >#323232</color>
          <!-- basic colors -->
          <color name="green">#27D34D</color>
          <color name="blue">#2A91BD</color>
          <color name="orange">#FF9D2F</color>
          <color name="red">#FF432F</color>
      </resources>


* 向应用设计者那里要这个调色板，名称不需要跟"green", "blue", 等等相同。

"brand_primary", `"brand_secondary", "brand_negative" 这样的名字也是完全可以接受的。
像这样规范的颜色很容易修改或重构，会使应用一共使用了多少种不同的颜色变得非常清晰。
通常一个具有审美价值的UI来说，减少使用颜色的种类是非常重要的。

* 像对待colors.xml一样对待dimens.xml文件 与定义颜色调色板一样，你同时也应该定义一个空隙间隔和字体大小的“调色板”。

一个好的例子，如下所示：






    <resources>
        <!-- font sizes -->
        <dimen name="font_larger">22sp</dimen>
        <dimen name="font_large">18sp</dimen>
        <dimen name="font_normal">15sp</dimen>
        <dimen name="font_small">12sp</dimen>
        <!-- typical spacing between two views -->
        <dimen name="spacing_huge">40dp</dimen>
        <dimen name="spacing_large">24dp</dimen>
        <dimen name="spacing_normal">14dp</dimen>
        <dimen name="spacing_small">10dp</dimen>
        <dimen name="spacing_tiny">4dp</dimen>
        <!-- typical sizes of views -->
        <dimen name="button_height_tall">60dp</dimen>
        <dimen name="button_height_normal">40dp</dimen>
        <dimen name="button_height_short">32dp</dimen>

    </resources>


* 布局时在写 margins 和 paddings 时，你应该使用spacing_****尺寸格式来布局，而不是像对待String字符串一样直接写值。
    这样写会非常有感觉，会使组织和改变风格或布局是非常容易。



* 避免深层次的视图结构 有时候为了摆放一个视图，你可能尝试添加另一个LinearLayout。你可能使用这种方法解决：





    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        >
        <RelativeLayout
            ...
            >
            <LinearLayout
                ...
                >
                <LinearLayout
                   ...
                    >
                    <LinearLayout
                        ...
                        >
                   </LinearLayout>
                </LinearLayout>
            </LinearLayout>
        </RelativeLayout>
    </LinearLayout>



* 即使你没有非常明确的在一个layout布局文件中这样使用，如果你在Java文件中从一个view inflate（这个inflate翻译不过去，大家理解就行） 到其他views当中，也是可能会发生的。


* 可能会导致一系列的问题。你可能会遇到性能问题，因为处理起需要处理一个复杂的UI树结构。

还可能会导致以下更严重的问题StackOverflowError.


* 因此尽量保持你的视图tree：学习如何使用RelativeLayout,

如何 optimize 你的布局 和如何使用```<merge>``` 标签.


* 小心关于WebViews的问题. 如果你必须显示一个web视图，

比如说对于一个新闻文章，避免做客户端处理HTML的工作，

最好让后端工程师协助，让他返回一个 "纯" HTML。

WebViews 也能导致内存泄露

当保持引他们的Activity，而不是被绑定到ApplicationContext中的时候。

当使用简单的文字或按钮时，避免使用WebView，这时使用TextView或Buttons更好。




### `Toast`


* 使用 工具类 `ToastUtils`,上下使用 `getApplicationContext()`;

### 新建`activity`

* 布局文件中,在根布局使用` android:fitsSystemWindows="true"` 属性,保证`Toast`, `Dialog`的中的内容不会移位  这个使用了 沉浸式状态栏的副作用!




### 异常的捕获处理

* 使用友盟 ` MobclickAgent.setCatchUncaughtExceptions(true);`开启错误统计,如果是为`false` 就不会上传错误


	Android统计SDK从V4.6版本开始内建错误统计，不需要开发者再手动集成。
	SDK通过`Thread.UncaughtExceptionHandler`  捕获程序崩溃日志，并在程序下次启动时发送到服务器。 如不需要错误统计功能，可通过此方法关闭

	MobclickAgent.setCatchUncaughtExceptions(false);

	如果开发者自己捕获了错误，需要上传到友盟服务器可以调用下面方法：

	public static void reportError(Context context, String error)
	//或
	public static void reportError(Context context, Throwable e)

	使用自定义错误，查看时请在错误列表页面选择【自定义错误】
