---
title: Webview上传文件的那些坑
date: 2016-02-22 21:01:14
tags: Webview
categories: 技术
---

>要说Android中最厉害的组件莫过于Webview 了，夸张点说把这个组件放在屏幕上就可以算作一个简单地浏览器应用了。但你若认为这就万事大吉了，可太小看Webview这个磨人的妖精了，下面单就上传文件的这个坑来做展开。

<!--more-->
### 支持上传文件

Webview执行上传操作的逻辑是这样的：首先准备上传时会回调WebChromeClient类下的`openFileChooser`方法，在这个方法中给我们机会发起Intent来打开支持提供文件的第三方应用，最后在`onActivityResult`回调中将第三方应用提供的内容通过一个叫做`ValueCallback`的参数返回给`Webview`（详细点来说：ValueCallback是在openFileChooser 方法里由webview提供给我们的，里面包裹一个Uri，我们在`onActivityResult` 里将选中的Uri反馈给`ValueCallback`，这时候相当于Webview就知道我们选择了什么文件），因此，我们需要为`Webview`设置一个提供`openFileChooser`方法的`WebChromeClient`，这个方法在不同版本的`Android`中参数是不同的，为此我们一般需要写三个重载函数，大致像这个样子：

    private ValueCallback<Uri> mUploadMessage;
    	//设置`WebChromeClient`:
    webview.setWebChromeClient(new WebChromeClient(){
         public void openFileChooser(ValueCallback<Uri> uploadMsg) {
                Log.d(TAG, "openFileChoose(ValueCallback<Uri> uploadMsg)");
                mUploadMessage = uploadMsg;
                Intent i = new Intent(Intent.ACTION_GET_CONTENT);
                i.addCategory(Intent.CATEGORY_OPENABLE);
                i.setType("*/*");
                MainActivity.this.startActivityForResult(Intent.createChooser(i, "File Chooser"), FILECHOOSER_RESULTCODE);
          }
          public void openFileChooser( ValueCallback uploadMsg, String acceptType ) {
                Log.d(TAG, "openFileChoose( ValueCallback uploadMsg, String acceptType )");
                mUploadMessage = uploadMsg;
                Intent i = new Intent(Intent.ACTION_GET_CONTENT);
                i.addCategory(Intent.CATEGORY_OPENABLE);
                i.setType("*/*");
                MainActivity.this.startActivityForResult(
                        Intent.createChooser(i, "File Browser"),
                        FILECHOOSER_RESULTCODE);
          }
          public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture){
                Log.d(TAG, "openFileChoose(ValueCallback<Uri> uploadMsg, String acceptType, String capture)");
                mUploadMessage = uploadMsg;
                Intent i = new Intent(Intent.ACTION_GET_CONTENT);
                i.addCategory(Intent.CATEGORY_OPENABLE);
                i.setType("*/*");
                MainActivity.this.startActivityForResult( Intent.createChooser( i, "File Browser" ), MainActivity.FILECHOOSER_RESULTCODE );
            }
    });

    //onActivityResult回调   
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    	    super.onActivityResult(requestCode, resultCode, data);
    	    if(requestCode==FILECHOOSER_RESULTCODE)
    	     {
    	            if (null == mUploadMessage && null == mUploadCallbackAboveL) return;
    	             Uri result = data == null || resultCode != RESULT_OK ? null : data.getData();
    				 if (mUploadMessage != null) {
    	                mUploadMessage.onReceiveValue(result);
    	                mUploadMessage = null;
    	           }
    	      }
       	}


还有重要的一点：如果这个上传操作涉及到JS操作，别忘记对Webview开启对JS的支持：

      WebSettings settings = webview.getSettings();
      settings.setJavaScriptEnabled(true);

### 代码混淆

    -keepclassmembers class * extends android.webkit.WebChromeClient{
        public void openFileChooser(...);
    }

### 兼容5.0



    webview.setWebChromeClient(new WebChromeClient(){
    public void openFileChooser(ValueCallback<Uri> uploadMsg) {
         ...
    }
    public void openFileChooser( ValueCallback uploadMsg, String acceptType ) {
    	   ...
    }
    public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture){
    				...
    }

    // For Android 5.0+
    public boolean onShowFileChooser (WebView webView, ValueCallback<Uri[]> filePathCallback, WebChromeClient.FileChooserParams fileChooserParams) {
             mUploadCallbackAboveL = filePathCallback;
             Intent i = new Intent(Intent.ACTION_GET_CONTENT);
             i.addCategory(Intent.CATEGORY_OPENABLE);
             i.setType("*/*");
             MainActivity.this.startActivityForResult(
                        Intent.createChooser(i, "File Browser"),
                        FILECHOOSER_RESULTCODE);
             return true;
            }
    });

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if(requestCode==FILECHOOSER_RESULTCODE)
        {
            if (null == mUploadMessage && null == mUploadCallbackAboveL) return;
            Uri result = data == null || resultCode != RESULT_OK ? null : data.getData();
            if (mUploadCallbackAboveL != null) {
                onActivityResultAboveL(requestCode, resultCode, data);
            }
            else  if (mUploadMessage != null) {
                mUploadMessage.onReceiveValue(result);
                mUploadMessage = null;
            }
        }
    }
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    private void onActivityResultAboveL(int requestCode, int resultCode, Intent data) {
        if (requestCode != FILECHOOSER_RESULTCODE
                || mUploadCallbackAboveL == null) {
            return;
        }
        Uri[] results = null;
        if (resultCode == Activity.RESULT_OK) {
            if (data == null) {
            } else {
                String dataString = data.getDataString();
                ClipData clipData = data.getClipData();
                if (clipData != null) {
                    results = new Uri[clipData.getItemCount()];
                    for (int i = 0; i < clipData.getItemCount(); i++) {
                        ClipData.Item item = clipData.getItemAt(i);
                        results[i] = item.getUri();
                    }
                }
                if (dataString != null)
                    results = new Uri[]{Uri.parse(dataString)};
            }
        }
        mUploadCallbackAboveL.onReceiveValue(results);
        mUploadCallbackAboveL = null;
        return;
    }

  > 代码转自: http://blog.saymagic.cn/2015/11/08/webview-upload.html

  > 源码地址: https://gitcafe.com/saymagic/Webviewdemo

      * 说明:实际上我使用了该段代码只能够 完成5.0+的支持,对于5.0以下的机器支持的并不完美,比如说上传完成之后,并不能显示图片到目标位置.

  * 没办法继续爬文 终于发现了


    package com.fuiou.webviewupload;
	import java.io.File;
	import android.app.Activity;
	import android.app.AlertDialog;
	import android.content.ContentValues;
	import android.content.Context;
	import android.content.DialogInterface;
	import android.content.Intent;
	import android.database.Cursor;
	import android.graphics.Bitmap;
	import android.net.Uri;
	import android.os.Bundle;
	import android.os.Environment;
	import android.provider.MediaStore;
	import android.util.Log;
	import android.view.KeyEvent;
	import android.webkit.ValueCallback;
	import android.webkit.WebChromeClient;
	import android.webkit.WebView;
	import android.webkit.WebViewClient;
	import android.widget.Toast;

	public class MainActivity extends Activity {
		public static final String TAG = "MainActivity";
		ValueCallback<Uri> mUploadMessage;
		private WebView mWebView;

		@Override
		protected void onCreate(Bundle savedInstanceState) {
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			initView();
		}

		private void initView() {
			mWebView = (WebView) findViewById(R.id.web_view);
			mWebView.setWebChromeClient(new MyWebChromeClient());

			mWebView.setWebViewClient(new MyWebViewClient(this));
	//		webView.loadUrl("file:///android_asset/upload_image.html");
			mWebView.loadUrl("http://192.168.72.62:8080/fileUpload");
		}


		private class MyWebViewClient extends WebViewClient{
			private Context mContext;
			public MyWebViewClient(Context context){
				super();
				mContext = context;
			}

			@Override
			public void onPageStarted(WebView view, String url, Bitmap favicon) {
				Log.d(TAG,"URL地址:" + url);
				super.onPageStarted(view, url, favicon);
			}

			@Override
			public void onPageFinished(WebView view, String url) {
				Log.i(TAG, "onPageFinished");
				super.onPageFinished(view, url);
			}
		}

		public static final int FILECHOOSER_RESULTCODE = 1;
		private static final int REQ_CAMERA = FILECHOOSER_RESULTCODE+1;
		private static final int REQ_CHOOSE = REQ_CAMERA+1;

		private class MyWebChromeClient extends WebChromeClient {

			// For Android 3.0+
			   public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType) {  
				   if (mUploadMessage != null) return;
				   mUploadMessage = uploadMsg;   
				   selectImage();
	//               Intent i = new Intent(Intent.ACTION_GET_CONTENT);
	//               i.addCategory(Intent.CATEGORY_OPENABLE);
	//               i.setType("*/*");
	//                   startActivityForResult( Intent.createChooser( i, "File Chooser" ), FILECHOOSER_RESULTCODE );
			   }
				// For Android < 3.0
				public void openFileChooser(ValueCallback<Uri> uploadMsg) {
					   openFileChooser( uploadMsg, "" );
				}
				// For Android  > 4.1.1
			  public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
					  openFileChooser(uploadMsg, acceptType);
			  }

		}

		/**
		 * 检查SD卡是否存在
		 *
		 * @return
		 */
		public final boolean checkSDcard() {
			boolean flag = Environment.getExternalStorageState().equals(
					Environment.MEDIA_MOUNTED);
			if (!flag) {
				Toast.makeText(this, "请插入手机存储卡再使用本功能",Toast.LENGTH_SHORT).show();
			}
			return flag;
		}
		String compressPath = "";

		protected final void selectImage() {
			if (!checkSDcard())
				return;
			String[] selectPicTypeStr = { "camera","photo" };
			new AlertDialog.Builder(this)
					.setItems(selectPicTypeStr,
							new DialogInterface.OnClickListener() {
								@Override
								public void onClick(DialogInterface dialog,
										int which) {
									switch (which) {
									// 相机拍摄
									case 0:
										openCarcme();
										break;
									// 手机相册
									case 1:
										chosePic();
										break;
									default:
										break;
									}
									compressPath = Environment
											.getExternalStorageDirectory()
											.getPath()
											+ "/fuiou_wmp/temp";
									new File(compressPath).mkdirs();
									compressPath = compressPath + File.separator
											+ "compress.jpg";
								}
							}).show();
		}

		String imagePaths;
		Uri  cameraUri;
		/**
		 * 打开照相机
		 */
		private void openCarcme() {
			Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);

			imagePaths = Environment.getExternalStorageDirectory().getPath()
					+ "/fuiou_wmp/temp/"
					+ (System.currentTimeMillis() + ".jpg");
			// 必须确保文件夹路径存在，否则拍照后无法完成回调
			File vFile = new File(imagePaths);
			if (!vFile.exists()) {
				File vDirPath = vFile.getParentFile();
				vDirPath.mkdirs();
			} else {
				if (vFile.exists()) {
					vFile.delete();
				}
			}
			cameraUri = Uri.fromFile(vFile);
			intent.putExtra(MediaStore.EXTRA_OUTPUT, cameraUri);
			startActivityForResult(intent, REQ_CAMERA);
		}

		/**
		 * 拍照结束后
		 */
		private void afterOpenCamera() {
			File f = new File(imagePaths);
			addImageGallery(f);
			File newFile = FileUtils.compressFile(f.getPath(), compressPath);
		}

		/** 解决拍照后在相册中找不到的问题 */
		private void addImageGallery(File file) {
			ContentValues values = new ContentValues();
			values.put(MediaStore.Images.Media.DATA, file.getAbsolutePath());
			values.put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg");
			getContentResolver().insert(
					MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
		}

		/**
		 * 本地相册选择图片
		 */
		private void chosePic() {
			FileUtils.delFile(compressPath);
			Intent innerIntent = new Intent(Intent.ACTION_GET_CONTENT); // "android.intent.action.GET_CONTENT"
			String IMAGE_UNSPECIFIED = "image/*";
			innerIntent.setType(IMAGE_UNSPECIFIED); // 查看类型
			Intent wrapperIntent = Intent.createChooser(innerIntent, null);
			startActivityForResult(wrapperIntent, REQ_CHOOSE);
		}

		/**
		 * 选择照片后结束
		 *
		 * @param data
		 */
		private Uri afterChosePic(Intent data) {

			// 获取图片的路径：
			String[] proj = { MediaStore.Images.Media.DATA };
			// 好像是android多媒体数据库的封装接口，具体的看Android文档
			Cursor cursor = managedQuery(data.getData(), proj, null, null, null);
			if(cursor == null ){
				Toast.makeText(this, "上传的图片仅支持png或jpg格式",Toast.LENGTH_SHORT).show();
				return null;
			}
			// 按我个人理解 这个是获得用户选择的图片的索引值
			int column_index = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);
			// 将光标移至开头 ，这个很重要，不小心很容易引起越界
			cursor.moveToFirst();
			// 最后根据索引值获取图片路径
			String path = cursor.getString(column_index);
			if(path != null && (path.endsWith(".png")||path.endsWith(".PNG")||path.endsWith(".jpg")||path.endsWith(".JPG"))){
				File newFile = FileUtils.compressFile(path, compressPath);
				return Uri.fromFile(newFile);
			}else{
				Toast.makeText(this, "上传的图片仅支持png或jpg格式",Toast.LENGTH_SHORT).show();
			}
			return null;
		}



		/**
		 * 返回文件选择
		 */
		@Override
		protected void onActivityResult(int requestCode, int resultCode,
				Intent intent) {
		//		if (requestCode == FILECHOOSER_RESULTCODE) {
		//			if (null == mUploadMessage)
		//				return;
		//			Uri result = intent == null || resultCode != RESULT_OK ? null
		//					: intent.getData();
		//			mUploadMessage.onReceiveValue(result);
		//			mUploadMessage = null;
		//		}

			if (null == mUploadMessage)
				return;
			Uri uri = null;
			if(requestCode == REQ_CAMERA ){
				afterOpenCamera();
				uri = cameraUri;
			}else if(requestCode == REQ_CHOOSE){
				uri = afterChosePic(intent);
			}
			mUploadMessage.onReceiveValue(uri);
			mUploadMessage = null;
			super.onActivityResult(requestCode, resultCode, intent);
		}

		public boolean onKeyDown(int keyCode, KeyEvent event) {
			if ((keyCode == KeyEvent.KEYCODE_BACK) && mWebView.canGoBack()) {  
				mWebView.goBack();  
				return true;  
			}else{
					finish();
			}
			return super.onKeyDown(keyCode, event);  
			}
	}

根据这个哥们的代码进行精简,符合自己的需求,而且发现了这个代码的一个问题,没有能够进行判空操作,如果没有选择图片,直接返回APP会奔溃!

下面是我的开发源码:

	private class MyWebChromeClient extends WebChromeClient {


			// For Android 5.0+
			public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, WebChromeClient.FileChooserParams fileChooserParams) {
				mUploadCallbackAboveL = filePathCallback;
	//            Intent i = new Intent(Intent.ACTION_GET_CONTENT);
	//            i.addCategory(Intent.CATEGORY_OPENABLE);
	//            i.setType("*/*");
	//            HomePageViewActivity.this.startActivityForResult(
	//                    Intent.createChooser(i, "File Browser"),
	//                    FILECHOOSER_RESULTCODE);
				selectImage();
				return true;
			}


			//For Android 3.0+
			public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType) {
				if (mUploadMessage != null) return;
				mUploadMessage = uploadMsg;
				selectImage();
			}

			// For Android < 3.0
			public void openFileChooser(ValueCallback<Uri> uploadMsg) {
				if (mUploadMessage != null) return;
				mUploadMessage = uploadMsg;
				selectImage();
			}

			// For Android  > 4.1.1
			public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
				if (mUploadMessage != null) return;
				mUploadMessage = uploadMsg;
				selectImage();
			}


		}


		@Override
		protected void onActivityResult(int requestCode, int resultCode, Intent data) {
			super.onActivityResult(requestCode, resultCode, data);
			if (requestCode == FILECHOOSER_RESULTCODE) {
				if (null == mUploadMessage && null == mUploadCallbackAboveL) return;
				Uri result = data == null || resultCode != RESULT_OK ? null : data.getData();
				if (mUploadCallbackAboveL != null) {
					onActivityResultAboveL(requestCode, resultCode, data);
				} else if (mUploadMessage != null) {
					mUploadMessage.onReceiveValue(result);
					mUploadMessage = null;
				}
			}


			if (null == mUploadMessage)
				return;
			Uri uri = null;
			if (requestCode == REQ_CHOOSE) {
				uri = afterChosePic(data);
			}
			mUploadMessage.onReceiveValue(uri);
			mUploadMessage = null;
			super.onActivityResult(requestCode, resultCode, data);
		}


		@TargetApi(Build.VERSION_CODES.LOLLIPOP)
		private void onActivityResultAboveL(int requestCode, int resultCode, Intent data) {
			if (requestCode != FILECHOOSER_RESULTCODE
					|| mUploadCallbackAboveL == null) {
				return;
			}

			Uri[] results = null;
			if (resultCode == Activity.RESULT_OK) {
				if (data == null) {

				} else {
					String dataString = data.getDataString();
					ClipData clipData = data.getClipData();

					if (clipData != null) {
						results = new Uri[clipData.getItemCount()];
						for (int i = 0; i < clipData.getItemCount(); i++) {
							ClipData.Item item = clipData.getItemAt(i);
							results[i] = item.getUri();
						}
					}

					if (dataString != null)
						results = new Uri[]{Uri.parse(dataString)};
				}
			}
			mUploadCallbackAboveL.onReceiveValue(results);
			mUploadCallbackAboveL = null;
			return;
		}


		/**
		 * 检查SD卡是否存在
		 *
		 * @return
		 */
		public final boolean checkSDcard() {
			boolean flag = Environment.getExternalStorageState().equals(
					Environment.MEDIA_MOUNTED);
			if (!flag) {
				Toast.makeText(this, "请插入手机存储卡再使用本功能", Toast.LENGTH_SHORT).show();
			}
			return flag;
		}

		String compressPath = "";

		protected final void selectImage() {
			if (!checkSDcard())
				return;

			chosePic();
			compressPath = Environment
					.getExternalStorageDirectory()
					.getPath()
					+ "/fuiou_wmp/temp";
			new File(compressPath).mkdirs();
			compressPath = compressPath + File.separator
					+ "compress.jpg";
		}


		/**
		 * 本地相册选择图片
		 */
		private void chosePic() {
			FileUtils.delFile(compressPath);
			Intent innerIntent = new Intent(Intent.ACTION_GET_CONTENT); // "android.intent.action.GET_CONTENT"
			String IMAGE_UNSPECIFIED = "image/*";
			innerIntent.setType(IMAGE_UNSPECIFIED); // 查看类型
			Intent wrapperIntent = Intent.createChooser(innerIntent, null);
			startActivityForResult(wrapperIntent, REQ_CHOOSE);
		}

		/**
		 * 选择照片后结束
		 *
		 * @param data
		 */
		private Uri afterChosePic(Intent data) {

			if (data != null) {
				// 获取图片的路径：
				String[] proj = {MediaStore.Images.Media.DATA};
				// 好像是android多媒体数据库的封装接口，具体的看Android文档
				Cursor cursor = managedQuery(data.getData(), proj, null, null, null);
	//        if (cursor == null) {
	//            Toast.makeText(this, "上传的图片仅支持png或jpg格式", Toast.LENGTH_SHORT).show();
	//            return null;
	//        }
				// 按我个人理解 这个是获得用户选择的图片的索引值
				int column_index = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA);
				// 将光标移至开头 ，这个很重要，不小心很容易引起越界
				cursor.moveToFirst();
				// 最后根据索引值获取图片路径
				String path = cursor.getString(column_index);
	//            if (path != null && (path.endsWith(".png") || path.endsWith(".PNG") || path.endsWith(".jpg") || path.endsWith(".JPG"))) {
	//            } else {
	//                Toast.makeText(this, "上传的图片仅支持png或jpg格式", Toast.LENGTH_SHORT).show();
	//            }
				File newFile = FileUtils.compressFile(path, compressPath);
				return Uri.fromFile(newFile);
			}
			return null;
		}

都目前位置 问题得到完美的解决!!

参考:http://blog.isming.me/2015/12/21/android-webview-upload-file/
http://www.huochai.mobi/p/d/900504/?share_tid=846ea82e2685&fmid=0
http://www.lai18.com/content/1191983.html
