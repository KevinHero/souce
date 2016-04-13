---
title: Android OkHttp使用教程
date: 2015-07-19 13:33:17
tags: OkHttp
categories: 技术
---
 


Android系统提供了两种HTTP通信类，HttpURLConnection和HttpClient。

尽管Google在大部分安卓版本中推荐使用HttpURLConnection，但是这个类相比HttpClient实在是太难用，太弱爆了。

OkHttp是一个相对成熟的解决方案，据说Android4.4的源码中可以看到HttpURLConnection已经替换成OkHttp实现了。所以我们更有理由相信OkHttp的强大。
<!--more-->
OkHttp 处理了很多网络疑难杂症：会从很多常用的连接问题中自动恢复。如果您的服务器配置了多个IP地址，当第一个IP连接失败的时候，OkHttp会自动尝试下一个IP。OkHttp还处理了代理服务器问题和SSL握手失败问题。

使用 OkHttp 无需重写您程序中的网络代码。OkHttp实现了几乎和java.net.HttpURLConnection一样的API。如果你用了 Apache HttpClient，则OkHttp也提供了一个对应的okhttp-apache 模块。

	注：在国内使用OkHttp会因为这个问题导致部分酷派手机用户无法联网，所以对于大众app来说，需要等待这个bug修复后再使用。或者尝试使用OkHttp的老版本。
	截止到目前，OkHttp一直没有修复，并把修复计划延迟到了OkHttp2.3中。不是所有设备都能重现，仅少量设备会出现这个问题。（如果问题这么明显，OkHttp早就修复了）

### 入门

	官方资料
	官方介绍
	github源码
	使用范围
	OkHttp支持Android 2.3及其以上版本。
	对于Java, JDK1.7以上。
	jar包准备
	官方介绍页面有链接位置。这里把下载链接也写在下面。
	OkHttp
	Okio

### 基本使用
#### HTTP GET

	1	OkHttpClient client = new OkHttpClient();
	2	 
	3	String run(String url) throws IOException {
	4	    Request request = new Request.Builder().url(url).build();
	5	    Response response = client.newCall(request).execute();    if (response.isSuccessful()) {        return response.body().string();
	6	    } else {        throw new IOException("Unexpected code " + response);
	7	    }
	8	}
	Request是OkHttp中访问的请求，Builder是辅助类。Response即OkHttp中的响应。
	Response类：
	1	public boolean isSuccessful()
	2	Returns true if the code is in [200..300),
	3	 which means the request was successfully received, understood, and accepted.
	response.body()返回ResponseBody类

可以方便的获取string

	1	public final String string() throws IOException
	2	Returns the response as a string decoded with the charset of the Content-Type header. If that header is either absent or lacks a charset,
	3	 this will attempt to decode the response body as UTF-8.Throws:
	4	IOException

当然也能获取到流的形式：

	1	public final InputStream byteStream()

#### HTTP POST

POST提交Json数据

	1	public static final MediaType JSON = MediaType.parse("application/json; charset=utf-8");
	2	OkHttpClient client = new OkHttpClient();
	3	String post(String url, String json) throws IOException {
	4	     RequestBody body = RequestBody.create(JSON, json);
	5	      Request request = new Request.Builder()
	6	      .url(url)
	7	      .post(body)
	8	      .build();
	9	      Response response = client.newCall(request).execute();
	10	    f (response.isSuccessful()) {
	11	        return response.body().string();
	12	    } else {
	13	        throw new IOException("Unexpected code " + response);
	14	    }
	15	}

使用Request的post方法来提交请求体RequestBody
POST提交键值对
很多时候我们会需要通过POST方式把键值对数据传送到服务器。 OkHttp提供了很方便的方式来做这件事情。

	1	OkHttpClient client = new OkHttpClient();
	2	String post(String url, String json) throws IOException {
	3	 
	4	     RequestBody formBody = new FormEncodingBuilder()
	5	    .add("platform", "android")
	6	    .add("name", "bug")
	7	    .add("subject", "XXXXXXXXXXXXXXX")
	8	    .build();
	9	 
	10	      Request request = new Request.Builder()
	11	      .url(url)
	12	      .post(body)
	13	      .build();
	14	 
	15	      Response response = client.newCall(request).execute();
	16	    if (response.isSuccessful()) {
	17	        return response.body().string();
	18	    } else {
	19	        throw new IOException("Unexpected code " + response);
	20	    }
	21	}

#### 总结
通过上面的例子我们可以发现，OkHttp在很多时候使用都是很方便的，而且很多代码也有重复，因此特地整理了下面的工具类。
注意：

	• OkHttp官方文档并不建议我们创建多个OkHttpClient，因此全局使用一个。 如果有需要，可以使用clone方法，再进行自定义。这点在后面的高级教程里会提到。
	• enqueue为OkHttp提供的异步方法，入门教程中并没有提到，后面的高级教程里会有解释。
	1	import java.io.IOException;
	2	import java.util.List;
	3	import java.util.concurrent.TimeUnit;
	4	import org.apache.http.client.utils.URLEncodedUtils;
	5	import org.apache.http.message.BasicNameValuePair;
	6	import cn.wiz.sdk.constant.WizConstant;
	7	import com.squareup.okhttp.Callback;
	8	import com.squareup.okhttp.OkHttpClient;
	9	import com.squareup.okhttp.Request;
	10	import com.squareup.okhttp.Response; 
	11	  
	12	public class OkHttpUtil {
	13	    private static final OkHttpClient mOkHttpClient = new OkHttpClient();
	14	    static{
	15	        mOkHttpClient.setConnectTimeout(30, TimeUnit.SECONDS);
	16	    }
	17	    /**
	18	     * 该不会开启异步线程。
	19	     * @param request
	20	     * @return
	21	     * @throws IOException
	22	     */
	23	    public static Response execute(Request request) throws IOException{
	24	        return mOkHttpClient.newCall(request).execute();
	25	    }
	26	    /**
	27	     * 开启异步线程访问网络
	28	     * @param request
	29	     * @param responseCallback
	30	     */
	31	    public static void enqueue(Request request, Callback responseCallback){
	32	        mOkHttpClient.newCall(request).enqueue(responseCallback);
	33	    }
	34	    /**
	35	     * 开启异步线程访问网络, 且不在意返回结果（实现空callback）
	36	     * @param request
	37	     */
	38	    public static void enqueue(Request request){
	39	        mOkHttpClient.newCall(request).enqueue(new Callback() {
	40	             
	41	            @Override
	42	            public void onResponse(Response arg0) throws IOException {
	43	                 
	44	            }
	45	             
	46	            @Override
	47	            public void onFailure(Request arg0, IOException arg1) {
	48	                 
	49	            }
	50	        });
	51	    }
	52	    public static String getStringFromServer(String url) throws IOException{
	53	        Request request = new Request.Builder().url(url).build();
	54	        Response response = execute(request);
	55	        if (response.isSuccessful()) {
	56	            String responseUrl = response.body().string();
	57	            return responseUrl;
	58	        } else {
	59	            throw new IOException("Unexpected code " + response);
	60	        }
	61	    }
	62	    private static final String CHARSET_NAME = "UTF-8";
	63	    /**
	64	     * 这里使用了HttpClinet的API。只是为了方便
	65	     * @param params
	66	     * @return
	67	     */
	68	    public static String formatParams(List<BasicNameValuePair> params){
	69	        return URLEncodedUtils.format(params, CHARSET_NAME);
	70	    }
	71	    /**
	72	     * 为HttpGet 的 url 方便的添加多个name value 参数。
	73	     * @param url
	74	     * @param params
	75	     * @return
	76	     */
	77	    public static String attachHttpGetParams(String url, List<BasicNameValuePair> params){
	78	        return url + "?" + formatParams(params);
	79	    }
	80	    /**
	81	     * 为HttpGet 的 url 方便的添加1个name value 参数。
	82	     * @param url
	83	     * @param name
	84	     * @param value
	85	     * @return
	86	     */
	87	    public static String attachHttpGetParam(String url, String name, String value){
	88	        return url + "?" + name + "=" + value;
	89	    }
	90	}

### 高级
高级属性其实用的不多，这里主要是对OkHttp github官方教程进行了翻译。
#### 同步get
下载一个文件，打印他的响应头，以string形式打印响应体。
响应体的 string() 方法对于小文档来说十分方便、高效。但是如果响应体太大（超过1MB），应避免适应 string()方法 ，因为他会将把整个文档加载到内存中。
对于超过1MB的响应body，应使用流的方式来处理body。

	1	private final OkHttpClient client = new OkHttpClient();
	2	 
	3	public void run() throws Exception {
	4	    Request request = new Request.Builder()
	5	        .url("http://publicobject.com/helloworld.txt")
	6	        .build();
	7	 
	8	    Response response = client.newCall(request).execute();
	9	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	10	 
	11	    Headers responseHeaders = response.headers();
	12	    for (int i = 0; i < responseHeaders.size(); i++) {
	13	      System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
	14	    }
	15	 
	16	    System.out.println(response.body().string());
	17	}

#### 异步get
在一个工作线程中下载文件，当响应可读时回调Callback接口。读取响应时会阻塞当前线程。OkHttp现阶段不提供异步api来接收响应体。

	1	private final OkHttpClient client = new OkHttpClient();
	2	 
	3	public void run() throws Exception {
	4	    Request request = new Request.Builder()
	5	        .url("http://publicobject.com/helloworld.txt")
	6	        .build();
	7	 
	8	    client.newCall(request).enqueue(new Callback() {
	9	      @Override public void onFailure(Request request, Throwable throwable) {
	10	        throwable.printStackTrace();
	11	      }
	12	 
	13	      @Override public void onResponse(Response response) throws IOException {
	14	        if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	15	 
	16	        Headers responseHeaders = response.headers();
	17	        for (int i = 0; i < responseHeaders.size(); i++) {
	18	          System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
	19	        }
	20	 
	21	        System.out.println(response.body().string());
	22	      }
	23	    });
	24	}

#### 提取响应头
典型的HTTP头 像是一个 Map<String, String> :每个字段都有一个或没有值。但是一些头允许多个值，像Guava的Multimap。例如：HTTP响应里面提供的Vary响应头，就是多值的。OkHttp的api试图让这些情况都适用。
当写请求头的时候，使用header(name, value)可以设置唯一的name、value。如果已经有值，旧的将被移除，然后添加新的。使用addHeader(name, value)可以添加多值（添加，不移除已有的）。
当读取响应头时，使用header(name)返回最后出现的name、value。通常情况这也是唯一的name、value。如果没有值，那么header(name)将返回null。如果想读取字段对应的所有值，使用headers(name)会返回一个list。
为了获取所有的Header，Headers类支持按index访问。

	1	private final OkHttpClient client = new OkHttpClient();
	2	 
	3	public void run() throws Exception {
	4	    Request request = new Request.Builder()
	5	        .url("https://api.github.com/repos/square/okhttp/issues")
	6	        .header("User-Agent", "OkHttp Headers.java")
	7	        .addHeader("Accept", "application/json; q=0.5")
	8	        .addHeader("Accept", "application/vnd.github.v3+json")
	9	        .build();
	10	 
	11	    Response response = client.newCall(request).execute();
	12	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	13	 
	14	    System.out.println("Server: " + response.header("Server"));
	15	    System.out.println("Date: " + response.header("Date"));
	16	    System.out.println("Vary: " + response.headers("Vary"));
	17	}

#### Post方式提交String
使用HTTP POST提交请求到服务。这个例子提交了一个markdown文档到web服务，以HTML方式渲染markdown。因为整个请求体都在内存中，因此避免使用此api提交大文档（大于1MB）。

	1	public static final MediaType MEDIA_TYPE_MARKDOWN
	2	  = MediaType.parse("text/x-markdown; charset=utf-8");
	3	 
	4	private final OkHttpClient client = new OkHttpClient();
	5	 
	6	public void run() throws Exception {
	7	    String postBody = ""
	8	        + "Releases\n"
	9	        + "--------\n"
	10	        + "\n"
	11	        + " * _1.0_ May 6, 2013\n"
	12	        + " * _1.1_ June 15, 2013\n"
	13	        + " * _1.2_ August 11, 2013\n";
	14	 
	15	    Request request = new Request.Builder()
	16	        .url("https://api.github.com/markdown/raw")
	17	        .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, postBody))
	18	        .build();
	19	 
	20	    Response response = client.newCall(request).execute();
	21	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	22	 
	23	    System.out.println(response.body().string());
	24	}

#### Post方式提交流
以流的方式POST提交请求体。请求体的内容由流写入产生。这个例子是流直接写入Okio的BufferedSink。你的程序可能会使用OutputStream，你可以使用BufferedSink.outputStream()来获取。

	1	public static final MediaType MEDIA_TYPE_MARKDOWN
	2	      = MediaType.parse("text/x-markdown; charset=utf-8");
	3	 
	4	private final OkHttpClient client = new OkHttpClient();
	5	 
	6	public void run() throws Exception {
	7	    RequestBody requestBody = new RequestBody() {
	8	      @Override public MediaType contentType() {
	9	        return MEDIA_TYPE_MARKDOWN;
	10	      }
	11	 
	12	      @Override public void writeTo(BufferedSink sink) throws IOException {
	13	        sink.writeUtf8("Numbers\n");
	14	        sink.writeUtf8("-------\n");
	15	        for (int i = 2; i <= 997; i++) {
	16	          sink.writeUtf8(String.format(" * %s = %s\n", i, factor(i)));
	17	        }
	18	      }
	19	 
	20	      private String factor(int n) {
	21	        for (int i = 2; i < n; i++) {
	22	          int x = n / i;
	23	          if (x * i == n) return factor(x) + " × " + i;
	24	        }
	25	        return Integer.toString(n);
	26	      }
	27	    };
	28	 
	29	    Request request = new Request.Builder()
	30	        .url("https://api.github.com/markdown/raw")
	31	        .post(requestBody)
	32	        .build();
	33	 
	34	    Response response = client.newCall(request).execute();
	35	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	36	 
	37	    System.out.println(response.body().string());
	38	}

#### Post方式提交文件
以文件作为请求体是十分简单的。

	1	public static final MediaType MEDIA_TYPE_MARKDOWN
	2	  = MediaType.parse("text/x-markdown; charset=utf-8");
	3	 
	4	private final OkHttpClient client = new OkHttpClient();
	5	 
	6	public void run() throws Exception {
	7	    File file = new File("README.md");
	8	 
	9	    Request request = new Request.Builder()
	10	        .url("https://api.github.com/markdown/raw")
	11	        .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, file))
	12	        .build();
	13	 
	14	    Response response = client.newCall(request).execute();
	15	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	16	 
	17	    System.out.println(response.body().string());
	18	}

#### Post方式提交表单
使用FormEncodingBuilder来构建和HTML<form>标签相同效果的请求体。键值对将使用一种HTML兼容形式的URL编码来进行编码。

	1	private final OkHttpClient client = new OkHttpClient();
	2	 
	3	public void run() throws Exception {
	4	    RequestBody formBody = new FormEncodingBuilder()
	5	        .add("search", "Jurassic Park")
	6	        .build();
	7	    Request request = new Request.Builder()
	8	        .url("https://en.wikipedia.org/w/index.php")
	9	        .post(formBody)
	10	        .build();
	11	 
	12	    Response response = client.newCall(request).execute();
	13	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	14	 
	15	    System.out.println(response.body().string());
	16	}

#### Post方式提交分块请求
MultipartBuilder可以构建复杂的请求体，与HTML文件上传形式兼容。多块请求体中每块请求都是一个请求体，可以定义自己的请求头。这些请求头可以用来描述这块请求，例如他的Content-Disposition。如果Content-Length和Content-Type可用的话，他们会被自动添加到请求头中。

	1	private static final String IMGUR_CLIENT_ID = "...";
	2	private static final MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");
	3	 
	4	private final OkHttpClient client = new OkHttpClient();
	5	 
	6	public void run() throws Exception {
	7	    // Use the imgur image upload API as documented at https://api.imgur.com/endpoints/image
	8	    RequestBody requestBody = new MultipartBuilder()
	9	        .type(MultipartBuilder.FORM)
	10	        .addPart(
	11	            Headers.of("Content-Disposition", "form-data; name=\"title\""),
	12	            RequestBody.create(null, "Square Logo"))
	13	        .addPart(
	14	            Headers.of("Content-Disposition", "form-data; name=\"image\""),
	15	            RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
	16	        .build();
	17	 
	18	    Request request = new Request.Builder()
	19	        .header("Authorization", "Client-ID " + IMGUR_CLIENT_ID)
	20	        .url("https://api.imgur.com/3/image")
	21	        .post(requestBody)
	22	        .build();
	23	 
	24	    Response response = client.newCall(request).execute();
	25	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	26	 
	27	    System.out.println(response.body().string());
	28	}

#### 使用Gson来解析JSON响应
Gson是一个在JSON和Java对象之间转换非常方便的api。这里我们用Gson来解析Github API的JSON响应。
注意：ResponseBody.charStream()使用响应头Content-Type指定的字符集来解析响应体。默认是UTF-8。

	1	private final OkHttpClient client = new OkHttpClient();
	2	private final Gson gson = new Gson();
	3	 
	4	public void run() throws Exception {
	5	    Request request = new Request.Builder()
	6	        .url("https://api.github.com/gists/c2a7c39532239ff261be")
	7	        .build();
	8	    Response response = client.newCall(request).execute();
	9	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	10	 
	11	    Gist gist = gson.fromJson(response.body().charStream(), Gist.class);
	12	    for (Map.Entry<String, GistFile> entry : gist.files.entrySet()) {
	13	      System.out.println(entry.getKey());
	14	      System.out.println(entry.getValue().content);
	15	    }
	16	}
	17	 
	18	static class Gist {
	19	    Map<String, GistFile> files;
	20	}
	21	 
	22	static class GistFile {
	23	    String content;
	24	}

#### 响应缓存
为了缓存响应，你需要一个你可以读写的缓存目录，和缓存大小的限制。这个缓存目录应该是私有的，不信任的程序应不能读取缓存内容。
一个缓存目录同时拥有多个缓存访问是错误的。大多数程序只需要调用一次new OkHttp()，在第一次调用时配置好缓存，然后其他地方只需要调用这个实例就可以了。否则两个缓存示例互相干扰，破坏响应缓存，而且有可能会导致程序崩溃。
响应缓存使用HTTP头作为配置。你可以在请求头中添加Cache-Control: max-stale=3600 ,OkHttp缓存会支持。你的服务通过响应头确定响应缓存多长时间，例如使用Cache-Control: max-age=9600。

	1	private final OkHttpClient client;
	2	 
	3	public CacheResponse(File cacheDirectory) throws Exception {
	4	    int cacheSize = 10 * 1024 * 1024; // 10 MiB
	5	    Cache cache = new Cache(cacheDirectory, cacheSize);
	6	 
	7	    client = new OkHttpClient();
	8	    client.setCache(cache);
	9	}
	10	 
	11	public void run() throws Exception {
	12	    Request request = new Request.Builder()
	13	        .url("http://publicobject.com/helloworld.txt")
	14	        .build();
	15	 
	16	    Response response1 = client.newCall(request).execute();
	17	    if (!response1.isSuccessful()) throw new IOException("Unexpected code " + response1);
	18	 
	19	    String response1Body = response1.body().string();
	20	    System.out.println("Response 1 response:          " + response1);
	21	    System.out.println("Response 1 cache response:    " + response1.cacheResponse());
	22	    System.out.println("Response 1 network response:  " + response1.networkResponse());
	23	 
	24	    Response response2 = client.newCall(request).execute();
	25	    if (!response2.isSuccessful()) throw new IOException("Unexpected code " + response2);
	26	 
	27	    String response2Body = response2.body().string();
	28	    System.out.println("Response 2 response:          " + response2);
	29	    System.out.println("Response 2 cache response:    " + response2.cacheResponse());
	30	    System.out.println("Response 2 network response:  " + response2.networkResponse());
	31	 
	32	    System.out.println("Response 2 equals Response 1? " + response1Body.equals(response2Body));
	33	}

### 扩展

在这一节还提到了下面一句：
There are cache headers to force a cached response, force a network response, or force the network response to be validated with a conditional GET.
我不是很懂cache，平时用到的也不多，所以把Google在Android Developers一段相关的解析放到这里吧。
Force a Network Response
In some situations, such as after a user clicks a 'refresh' button, it may be necessary to skip the cache, and fetch data directly from the server. To force a full refresh, add the no-cache directive:
connection.addRequestProperty("Cache-Control", "no-cache");
If it is only necessary to force a cached response to be validated by the server, use the more efficient max-age=0 instead:
connection.addRequestProperty("Cache-Control", "max-age=0");
Force a Cache Response
Sometimes you'll want to show resources if they are available immediately, but not otherwise. This can be used so your application can show something while waiting for the latest data to be downloaded. To restrict a request to locally-cached resources, add the only-if-cached directive:

	1	try {
	2	     connection.addRequestProperty("Cache-Control", "only-if-cached");
	3	     InputStream cached = connection.getInputStream();
	4	     // the resource was cached! show it
	5	  catch (FileNotFoundException e) {
	6	     // the resource was not cached
	7	 }
	8	}

This technique works even better in situations where a stale response is better than no response. To permit stale cached responses, use the max-stale directive with the maximum staleness in seconds:

	1	int maxStale = 60 * 60 * 24 * 28; // tolerate 4-weeks staleconnection.addRequestProperty("Cache-Control", "max-stale=" + maxStale);

以上信息来自：`HttpResponseCache - Android SDK | Android Developers`
取消一个Call
使用Call.cancel()可以立即停止掉一个正在执行的call。如果一个线程正在写请求或者读响应，将会引发IOException。当call没有必要的时候，使用这个api可以节约网络资源。例如当用户离开一个应用时。不管同步还是异步的call都可以取消。
你可以通过tags来同时取消多个请求。当你构建一请求时，使用RequestBuilder.tag(tag)来分配一个标签。之后你就可以用OkHttpClient.cancel(tag)来取消所有带有这个tag的call。

	1	private final ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
	2	private final OkHttpClient client = new OkHttpClient();
	3	 
	4	public void run() throws Exception {
	5	    Request request = new Request.Builder()
	6	        .url("http://httpbin.org/delay/2") // This URL is served with a 2 second delay.
	7	        .build();
	8	 
	9	    final long startNanos = System.nanoTime();
	10	    final Call call = client.newCall(request);
	11	 
	12	    // Schedule a job to cancel the call in 1 second.
	13	    executor.schedule(new Runnable() {
	14	      @Override public void run() {
	15	        System.out.printf("%.2f Canceling call.%n", (System.nanoTime() - startNanos) / 1e9f);
	16	        call.cancel();
	17	        System.out.printf("%.2f Canceled call.%n", (System.nanoTime() - startNanos) / 1e9f);
	18	      }
	19	    }, 1, TimeUnit.SECONDS);
	20	 
	21	    try {
	22	      System.out.printf("%.2f Executing call.%n", (System.nanoTime() - startNanos) / 1e9f);
	23	      Response response = call.execute();
	24	      System.out.printf("%.2f Call was expected to fail, but completed: %s%n",
	25	          (System.nanoTime() - startNanos) / 1e9f, response);
	26	    } catch (IOException e) {
	27	      System.out.printf("%.2f Call failed as expected: %s%n",
	28	          (System.nanoTime() - startNanos) / 1e9f, e);
	29	    }
	30	}

#### 超时
没有响应时使用超时结束call。没有响应的原因可能是客户点链接问题、服务器可用性问题或者这之间的其他东西。OkHttp支持连接，读取和写入超时。

	1	private final OkHttpClient client;
	2	 
	3	public ConfigureTimeouts() throws Exception {
	4	    client = new OkHttpClient();
	5	    client.setConnectTimeout(10, TimeUnit.SECONDS);
	6	    client.setWriteTimeout(10, TimeUnit.SECONDS);
	7	    client.setReadTimeout(30, TimeUnit.SECONDS);
	8	}
	9	 
	10	public void run() throws Exception {
	11	    Request request = new Request.Builder()
	12	        .url("http://httpbin.org/delay/2") // This URL is served with a 2 second delay.
	13	        .build();
	14	 
	15	    Response response = client.newCall(request).execute();
	16	    System.out.println("Response completed: " + response);
	17	}

#### 每个call的配置
使用OkHttpClient，所有的HTTP Client配置包括代理设置、超时设置、缓存设置。当你需要为单个call改变配置的时候，clone 一个 OkHttpClient。这个api将会返回一个浅拷贝（shallow copy），你可以用来单独自定义。下面的例子中，我们让一个请求是500ms的超时、另一个是3000ms的超时。

	1	private final OkHttpClient client = new OkHttpClient();
	2	 
	3	public void run() throws Exception {
	4	    Request request = new Request.Builder()
	5	        .url("http://httpbin.org/delay/1") // This URL is served with a 1 second delay.
	6	        .build();
	7	 
	8	    try {
	9	      Response response = client.clone() // Clone to make a customized OkHttp for this request.
	10	          .setReadTimeout(500, TimeUnit.MILLISECONDS)
	11	          .newCall(request)
	12	          .execute();
	13	      System.out.println("Response 1 succeeded: " + response);
	14	    } catch (IOException e) {
	15	      System.out.println("Response 1 failed: " + e);
	16	    }
	17	 
	18	    try {
	19	      Response response = client.clone() // Clone to make a customized OkHttp for this request.
	20	          .setReadTimeout(3000, TimeUnit.MILLISECONDS)
	21	          .newCall(request)
	22	          .execute();
	23	      System.out.println("Response 2 succeeded: " + response);
	24	    } catch (IOException e) {
	25	      System.out.println("Response 2 failed: " + e);
	26	    }
	27	}

#### 处理验证
这部分和HTTP AUTH有关。
相关资料：HTTP AUTH 那些事 - 王绍全的博客 - 博客频道 - CSDN.NET
OkHttp会自动重试未验证的请求。当响应是401 Not Authorized时，Authenticator会被要求提供证书。Authenticator的实现中需要建立一个新的包含证书的请求。如果没有证书可用，返回null来跳过尝试。

	1	public List<Challenge> challenges()
	2	Returns the authorization challenges appropriate for this response's code. 
	3	If the response code is 401 unauthorized, 
	4	this returns the "WWW-Authenticate" challenges.
	5	If the response code is 407 proxy unauthorized, this returns the "Proxy-Authenticate" challenges.
	6	Otherwise this returns an empty list of challenges.

当需要实现一个Basic challenge， 使用Credentials.basic(username, password)来编码请求头。

	1	private final OkHttpClient client = new OkHttpClient();
	2	 
	3	public void run() throws Exception {
	4	    client.setAuthenticator(new Authenticator() {
	5	      @Override public Request authenticate(Proxy proxy, Response response) {
	6	        System.out.println("Authenticating for response: " + response);
	7	        System.out.println("Challenges: " + response.challenges());
	8	        String credential = Credentials.basic("jesse", "password1");
	9	        return response.request().newBuilder()
	10	            .header("Authorization", credential)
	11	            .build();
	12	      }
	13	 
	14	      @Override public Request authenticateProxy(Proxy proxy, Response response) {
	15	        return null; // Null indicates no attempt to authenticate.
	16	      }
	17	    });
	18	 
	19	    Request request = new Request.Builder()
	20	        .url("http://publicobject.com/secrets/hellosecret.txt")
	21	        .build();
	22	 
	23	    Response response = client.newCall(request).execute();
	24	    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);
	25	 
	26	    System.out.println(response.body().string());
	27	}


转自  <http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0106/2275.html>
