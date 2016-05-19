---
layout: post
title:  "如何在手机浏览中打开APP（Android、iOS）"
date:   2016-05-19 15:15:32 +0800
categories: front-end
---
在手机浏览器中，如果用户安装了App就打开该App，否则进入App下载页面。

{% highlight javascript %}
var urlSchema = [Schema]://[URL];
var androidDownloadUrl = 'http://***';
var iOSDownloadUrl = 'itms-apps://www.apple.com/****';

// 获取当前浏览器用于 HTTP 请求的用户代理头的值
var ua = navigator.userAgent.toLowerCase();

if(/micromessenger/.test(ua)) {
  // 微信

} else {
  var downloadUrl;
  if(/iphone|ipad|ipod/.test(ua)) {
    // iOS
    downloadUrl = iOSDownloadUrl;
  } else if(/android/.test(ua)) {
    // Android
    downloadUrl = androidDownloadUrl;
  }
  window.location.href = urlSchema;
  setTimeout(() => {
    window.location.href = downloadUrl;
  }, 1500);
}
{% endhighlight %}

1、“urlSchema”包含两部分，Schema和URL，两部分都是自己定义，但是要和App的开发者约定好，App会根据规则来进行解析。Schema可以是产品名称，比如“taobao”，URL则是请求的地址加上参数，比如“product?id=1&type=2”，域名也可以省略，最终形成的urlSchema应该是这样的：

```
taobao://product?id=1&type=2
```

2、微信无法支持，很多App都是通过引导用户用浏览器打开来解决的。

3、iOS的下载地址以“itms-apps://”开头，用来打开iOS设备上的AppStore。

4、setTimeout的时间本来设置的是500ms，在Android设备上没有问题，但是在iOS设备上会直接跳转到AppStore，把时间延长到1500ms，可以解决该问题。