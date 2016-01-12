---
layout: post
title:  "Nginx中请求大小的限制的设置"
date:   2016-01-11 13:46:53 +0800
categories: java
tags: nginx
---
Nginx对客户端请求缓冲区大小有个默认限制，如果超过了该值（比如在上传大文件时），会报500错误。

只需要设置三个值，就可以解决该问题：

1、 client_body_buffer_size： 指定客户端请求体缓冲区大小，如果请求大于该值，会报“500 Internal Server Error”错误。

The directive specifies the client request body buffer size.

If the request body is more than the buffer, then the entire request body or some part is written in a temporary file.

The default size is equal to two pages size, depending on platform it is either 8K or 16K.

2、 client_body_temp_path： 指定请求体临时文件的存放目录。

The directive assigns the directory for storing the temporary files in it with the body of the request.

3、 client_max_body_size： 允许客户端请求的最大单文件字节数，如果请求体大于该值，会报“413 Request Entity Too Large”错误。

Directive assigns the maximum accepted body size of client request.

<!-- more -->

更多请参考：http://nginx.org/en/docs/http/ngx_http_core_module.html