---
layout: post
title:  "VIP视频解析，VIP视频在线观看"
date:   2018-11-17 13:46:53 +0800
categories: others
---
<div class="row">
  <div class="col-lg-12">
    <div class="input-group">
      <input type="text" id="target" class="form-control" placeholder="请输入视频地址...">
      <span class="input-group-btn">
        <button class="btn btn-default" type="button" onclick="play()">播放!</button>
      </span>
    </div>
  </div>
</div>
<div class="row">
  <div class="col-lg-12">
  <div class="embed-responsive embed-responsive-16by9">
    <iframe class="embed-responsive-item" src="" id="iframe"></iframe>
  </div>
  </div>
</div>  
<div class="row">
  <div class="col-lg-12">
      <p>VIP视频解析站：提供优酷VIP解析，爱奇艺VIP解析，腾讯VIP解析，乐视VIP解析，芒果VIP解析等解析服务</p>
  </div>
</div>

<script>
    var api = 'http://api.smq1.com/?url=';
    function play() {
        var target = document.getElementById('target').value;
        if(target) {
            document.getElementById("iframe").src = api + target;
        }
    }
</script>