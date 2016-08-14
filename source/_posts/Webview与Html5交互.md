---
layout: android
title: Webview与Html5交互
date: 2015-08-14 15:01:24
categories: H5
tags:
	- H5
  	- Android
---
<br/>
### 一、WebView.setWebViewClient(new MyWebViewClient());

Java

		public boolean shouldOverrideUrlLoading(WebView 			view, String url) { 
       		onWebPageShouldLoad(view, url);  //通过
       		return true;
  		}

在点击请求的是链接是才会调用，重写此方法返回true表明点击网页里面的链接还是在当前的webview里跳转，不跳到浏览器那边。<br>
<!--more-->
坑爹之处1：Android 2.3.x WebView中的两个搞笑的bug : http://blog.csdn.net/thestoryoftony/article/details/7844287

解决办法： 将逻辑加在onPageStarted中处理。

Java

		public void onPageStarted(WebView view, String 		url, Bitmap favicon) {

		}
在页面加载开始时调用。

shouldOverrideUrlLoading与onPageStarted区别：

当点击页面中的链接的时候他们俩都会执行，但是返回到上一个页面的时候onPageStarted会执行，但是shouldOverrideUrlLoading就不执行了，就是onPageStarted什么时候都执行的

Java

	public void onPageFinished(WebView view, String url) 	{
	       onWebPageLoaded(view, url);
	 }
	public void onPageFinished ( WebView view , String 	url ) {
	
	onWebPageLoaded ( view , url ) ;
	
	}

在页面加载结束时调用。

### 二、WebView.setWebChromeClient(new MyWebChromeClient());

Java

	public void onReceivedTitle(WebView view, String 	title) {
	//设置Actionbar的Title
	}
	
	public void onProgressChanged(WebView view, int 	progress) {
	//设置页面加载进度
	}
	
	@Override
	public boolean onJsConfirm(WebView view, String url, 	String message, final JsResult result) {
	//弹出框处理（alert，confirm）
	}



### 三、WebView.addJavascriptInterface(jsObject, “jsObj”);

#### 1.先写一个接口类

Java

	public class JsInteface{
	//分享相关的内容
	private String mTitle;
	private String mDes;
	private String mLink;
	private String mImgUrl;
	private String mBigImgUrl;
	
	@JavascriptInterface
	public void setShareContent(String Title,String Des,String Link,String ImgUrl,String BigImgUrl)
	{
	     mTitle=Title;
	     mDes=Des;
	     mLink=Link;
	     mImgUrl=ImgUrl;
	     mBigImgUrl=BigImgUrl;
	     Log.i("OUTPUT", "11title:"+mTitle+"  desc:"+mDes+"   mLink"+mLink+"    mImgUrl"+mImgUrl+"    mBigImgUrl"+mBigImgUrl);
	}
	}

#### 2.向webview中注入接口类的对象 `WebView.addJavascriptInterface(jsObject, “jsObj”);`

#### 3.调用注入对象的js

Java

	mWebView.loadUrl("javascript:window.jsObj.setShareContent(document.getElementById('app_title').innerHTML,"
	    + "document.getElementById('app_desc').innerHTML,"
	    + "document.getElementById('app_link').innerHTML,"
	    + "document.getElementById('app_img_url').src,"
	    + "document.getElementById('app_big_img_url').src)");
	mWebView . loadUrl ( "javascript:window.jsObj.setShareContent(document.getElementById('app_title').innerHTML,"
	
	+ "document.getElementById('app_desc').innerHTML,"
	
	+ "document.getElementById('app_link').innerHTML,"
	
	+ "document.getElementById('app_img_url').src,"
	
	+ "document.getElementById('app_big_img_url').src)" ) ;

坑爹之处2：Webview.addJavascriptInterface() does not work on API 17

http://stackoverflow.com/questions/16353430/appview-addjavascriptinterface-does-not-work-on-api-17
解决办法： 在接口方法前加上 @JavascriptInterface，并且引入该类，import android.webkit.JavascriptInterface;

### 四、<b>WebView.setOnKeyListener(new View.OnKeyListener())</b>

Java

	mWebView.setOnKeyListener(new View.OnKeyListener() {  
	            @Override  
	            public boolean onKey(View v, int keyCode, KeyEvent event) {  
	                if (event.getAction() == KeyEvent.ACTION_DOWN) {  
	                    if (keyCode == KeyEvent.KEYCODE_BACK && mWebView.canGoBack()) {  //表示按返回键
	                        mWebView.goBack();   //后退  
	                         return true;    //已处理  
	                    }  
	                }  
	                return false;  
	            }  
	        });

这段代码是监听back按键使webview后退一个页面而不是退出webview，类似于浏览器中的后退按键。

### 五、<b>WebView.setDownloadListener(new MyDownloadListener());</b>

这个API可以做下载方面的处理，自己在项目中没有使用到，这里就不做解释了。
<br/>
[示例代码](https://github.com/janker0718/WebViewDemo)
