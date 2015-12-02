---
layout: post
title:  "Bootstrap Modal 模态框的简单封装"
date:   2015-12-02 13:31:01 +0800
categories: front-end
tags: Bootstrap
---
把 Bootstrap 的模态框简单的封装了一下，在这里做个记录，以备不时之需。

<!-- more -->

1、HTML

{% highlight html linenos %}
<div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        <h4 class="modal-title" id="myModalLabel">模态框</h4>
      </div>
      <div class="modal-body">
      </div>
      <div class="modal-footer">
      </div>
    </div>
  </div>
</div>
{% endhighlight %}

2、JS

{% highlight javascript linenos %}
jQuery.fn.extend({
    showModal: function(options) {
        return this.each(function() {
            var title = options.title || ''; //模态框的名称，显示在 #myModalLabel 处
            var width = options.width || ''; //模态框的宽度
            var buttons = options.buttons || []; //模态框的按钮
            var content = options.content || ''; //模态框的内容，显示在 .modal-body 处
            var contentUrl = options.contentUrl; //以Jquery.load(url)的方式去加载模态框的内容，若设置了这个选项，content 选项将无效
            var modal = $(this);
            modal.find('.modal-title').text(title);
            modal.find('.modal-dialog').width(width);
            if(contentUrl) {
                modal.find('.modal-body').load(contentUrl);
            } else {
                modal.find('.modal-body').html(content);
            }

            var footer = '';
            for(var i in buttons) {
                var btnClass = buttons[i].class || 'btn btn-default';
                footer += '<button class="'+btnClass+'">'+buttons[i].text+'</button>';
            }
            modal.find('.modal-footer').html(footer);
            modal.find('.modal-footer button').each(function(i, btn) {
                $(btn).on('click', buttons[i].click);

            });
            $(this).modal("show");
        });
    }
});
{% endhighlight %}

3、调用
{% highlight javascript linenos %}
$('#myModal').showModal({
    title: '模态框的名称',
    contentUrl: 'view.html',
    buttons: [{
        text: '提交',
        class: 'btn btn-primary',
        click: function() {
            alert('提交了');
        }
    }, {
        text: '取消',
        class: 'btn btn-danger',
        click: function() {
            alert('取消了');
            $('#myModal').modal('hide');
        }
    }]
});
{% endhighlight %}