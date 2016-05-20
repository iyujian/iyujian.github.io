---
layout: post
title:  "如何将URL参数转成对象（JS）"
date:   2016-05-20 11:40:26 +0800
categories: front-end
---
比如URL是http://www.***.com/product.html?id=1&type=2，那么如何使用Javascript将URL参数转成对象？

方式一：正则实现将URL参数转成对象：

{% highlight javascript %}
console.log(getParameters());
console.log(getParameterValueByName("type"));
function getParameterValueByName(name) {
  return getParameters()[name];
}
function getParameters() {
  var parameters = {};
  var search = window.location.search.substring(1);
  var pattern = /(\w+)=(\w+)/ig;
  search.replace(pattern, function(pair, name, value){
    parameters[name] = value;
  });
  return parameters;
}
{% endhighlight %}

方式二：split实现将URL参数转成对象：

{% highlight javascript %}
console.log(getParameters());
console.log(getParameterValueByName("type"));
function getParameterValueByName(name) {
  return getParameters()[name];
}
function getParameters() {
  var parameters = {};
  var search = window.location.search.substring(1);
  var _temp = search.split('&');
  for(var i in _temp) {
    var __temp = _temp[i].split('=');
    parameters[__temp[0]] = __temp[1];
  }
  return parameters;
}
{% endhighlight %}