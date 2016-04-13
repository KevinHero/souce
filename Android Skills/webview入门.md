---
title: WebView入门
date: 2014-08-08 03:01:56
tags: WebView
categories: 技术
---


>WebView控件可以实现一个浏览器的功能，直接在控件中显示指定的网页

第一种方式

第一种方式是不需要，在布局文件中，使用WebView控件的

步骤：
<!--more-->

1、创建WebView实例

            WebView web= new WebView(Context context);

2、webkit浏览器是支持JavaScript的所以，添加支持

            web.getSetting.setJavaScriptEnabled（true）;

3、添加需要加载的网页地址Uri

            web.loadUri("http://www.google.com");

4、为保证能点击加载页面中的超链接是在当前打开，而不是在系统默认的浏览器中打开 ，需要添加支持

            web.setWebViewClient(shouldOverrideUrlLoad(view,url){
                            view.loadUrl(url);
                            return super.shouldOverrideUrlLoad(view,url);   
            });

5、显示页面

            setContentView(web);

6、为了在按系统返回键，出现直接退出app的情况，重写activity的 onkeydown（）方法

7、添加权限

            <uses-permission android:name="android.permission.INTERNET" />

代码实现

布局文件

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <TextView
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="社会新闻" />

    </LinearLayout>
实现代码

    public class MainActivity extends Activity {

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
           WebView webView= new WebView(this);
            //支持javascript
            webView.getSettings().setJavaScriptEnabled(true);
            //触摸焦点
            webView.requestFocus();
            //滚动条取消
            webView.setScrollBarStyle(View.SCROLLBARS_OUTSIDE_OVERLAY);
           //指定网页
            webView.loadUrl("http://www.qiushibaike.com");
            //显示网页
            setContentView(webView);

           //保证超链接跳转在当前页面
            webView.setWebViewClient(new WebViewClient(){
                @Override
                public boolean shouldOverrideUrlLoading(WebView view, String url) {
                    view.loadUrl(url);
                    return super.shouldOverrideUrlLoading(view, url);
                }
            } );
        }
           /**
             * 保证，在按系统返回键的时候，不是退出程序，而是，返回上一个页面
             * @param keyCode
             * @param event
             * @return
             */
            @Override
            public boolean onKeyDown(int keyCode, KeyEvent event) {
                if((keyCode==KeyEvent.KEYCODE_BACK))
                {
                    webView.goBack(); //返回上一个页面
                    return true;
                }
                return  false;
            }
    }
清单文件

            <manifest xmlns:android="http://schemas.android.com/apk/res/android"
                      package="com.kevin.fragmentdemo" >
                    <uses-permission android:name="android.permission.INTERNET" />
            </manifest>
第二种方式

基本上与第一种方式相同，只是在布局文件中定义了WebView的控件

步骤

1、创建WebView实例,通过的是布局文件中的id创建 XX是WebView组件的id

             WebView webView= (WebView) findViewById(R.id.XX);
2、webkit浏览器是支持JavaScript的所以，添加支持

            webView.getSetting.setJavaScriptEnabled（true）;  

3、为保证能点击加载页面中的超链接是在当前打开，而不是在系统默认的浏览器中打开 ，需要添加支持

            web.setWebViewClient(shouldOverrideUrlLoad(view,url){
                            view.loadUrl(url);
                            return super.shouldOverrideUrlLoad(view,url);   
            });
4、添加需要加载的网页地址Uri

            webView.loadUri("http://www.google.com");

5、为了在按系统返回键，出现直接退出app的情况，重写activity的 onkeydown（）方法（详见代码）

6、添加权限、添加权限

            <uses-permission android:name="android.permission.INTERNET" />

代码实现

布局文件

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
		xmlns:tools="http://schemas.android.com/tools"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		tools:context=".MainActivity">
		<WebView
			android:id="@+id/wv"
			android:layout_width="fill_parent"
			android:layout_height="fill_parent">
		</WebView>
	</RelativeLayout>

实现代码

	public class MainActivity extends Activity {
    private WebView webView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        webView = (WebView) findViewById(R.id.web_view);
        webView.getSettings().setJavaScriptEnabled(true);
        webView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String
            url) {
            view.loadUrl(url); // 根据传入的参数再去加载新的网页
            return true; // 表示当前WebView可以处理打开新网页的请求,不用借助
            系统浏览器
            }
        });
        webView.loadUrl("http://www.baidu.com");
    }
     /**
      * 保证，在按系统返回键的时候，不是退出程序，而是，返回上一个页面
      * @param keyCode
      * @param event
      * @return
      */
     @Override
     public boolean onKeyDown(int keyCode, KeyEvent event) {
     if((keyCode==KeyEvent.KEYCODE_BACK))
      {
            webView.goBack(); //返回上一个页面
             return true;
      }
 	           return  false;
  	}
	}
清单文件

    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.kevin.fragmentdemo" >
            <uses-permission android:name="android.permission.INTERNET" />
    </manifest>
