---
layout: post
title:  "如何自定义滚动条的样式（Chrome有效）"
date:   2016-06-21 16:18:31 +0800
categories: front-end
---
目前各种浏览器自带的滚动条的样式很不美观，而Google Chrome浏览器为webkit内核，提供了对滚动条样式的自定义功能，虽然对其他内核的浏览器没有效果，但是聊胜于无，况且现在Chrome的用户还是非常高的。

示例：

<style>
#scroll-bar::-webkit-scrollbar {
  width: 10px;
  height: 10px;
  background-color: #f5f5f5;
}
#scroll-bar::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px rgba(0, 102, 153, 0.3);
    border-radius: 10px;
    background-color: #f5f5f5;
}
#scroll-bar::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 6px rgba(0, 102, 153, 1.0);
    background-color: #006699;
}
</style>
<div id="scroll-bar" style="overflow-y: auto; height: 200px; border: 1px solid #cccccc; margin-bottom: 2em;">
  <div style="height: 800px;"></div>
</div>

下面是简单的实现代码：

{% highlight css %}
/* 定义滚动条基本样式 */
::-webkit-scrollbar {
  width: 10px; /* 垂直滚动条的宽度 */
  height: 10px; /* 水平滚动条的高度 */
  background-color: #f5f5f5; /* 滚动条的背景色 */
}

/* 定义滚动条轨道 */
::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px rgba(0, 102, 153, 0.3); /* 内阴影 */
    border-radius: 10px;
    background-color: #f5f5f5;
}

/* 定义滑块 */
::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 6px rgba(0, 102, 153, 1.0);
    background-color: #006699;
}
{% endhighlight %}

代码释义：

> ::-webkit-scrollbar：定义滚动条的整体样式，垂直滚动条的宽度，水平滚动条的高度，背景等。

> ::-webkit-scrollbar-track：定义滚动条的轨道样式。

> ::-webkit-scrollbar-thumb：定义滚动条滑块的样式。

> ::-webkit-scrollbar-button：定义滚动条轨道两端按钮的样式。

> ::-webkit-scrollbar-track-piece：定义滚动条滚动中未被滑块覆盖的地方的样式。

> ::-webkit-scrollbar-corner：定义两个滚动条交汇处的角落的样式。

> ::-webkit-resizer：定义可拖动元素的样式，比如“textarea”右下角可拖动的地方。

更多详细设置：


> :horizontal – The horizontal pseudo-class applies to any scrollbar pieces that have a horizontal orientation.

> :vertical – The vertical pseudo-class applies to any scrollbar pieces that have a vertical orientation.

> :decrement – The decrement pseudo-class applies to buttons and track pieces. It indicates whether or not the button or track piece will decrement the view’s position when used (e.g., up on a vertical scrollbar, left on a horizontal scrollbar).

> :increment – The increment pseudo-class applies to buttons and track pieces. It indicates whether or not a button or track piece will increment the view’s position when used (e.g., down on a vertical scrollbar, right on a horizontal scrollbar).

> :start – The start pseudo-class applies to buttons and track pieces. It indicates whether the object is placed before the thumb.

> :end – The end pseudo-class applies to buttons and track pieces. It indicates whether the object is placed after the thumb.

> :double-button – The double-button pseudo-class applies to buttons and track pieces. It is used to detect whether a button is part of a pair of buttons that are together at the same end of a scrollbar. For track pieces it indicates whether the track piece abuts a pair of buttons.

> :single-button – The single-button pseudo-class applies to buttons and track pieces. It is used to detect whether a button is by itself at the end of a scrollbar. For track pieces it indicates whether the track piece abuts a singleton button.

> :no-button – Applies to track pieces and indicates whether or not the track piece runs to the edge of the scrollbar, i.e., there is no button at that end of the track.

> :corner-present – Applies to all scrollbar pieces and indicates whether or not a scrollbar corner is present.

> :window-inactive – Applies to all scrollbar pieces and indicates whether or not the window containing the scrollbar is currently active. (In recent nightlies, this pseudo-class now applies to ::selection as well. We plan to extend it to work with any content and to propose it as a new standard pseudo-class.)
