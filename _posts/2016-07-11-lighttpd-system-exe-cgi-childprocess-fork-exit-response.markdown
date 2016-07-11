---
title: 浏览器向lighttpd提交表单response延迟过长问题
layout: post
guid: urn:uuid:de8a34084-6f35-4f7b-bb2d-1977eb7dadfc
tags:
  - lighttpd 
  - form 
  - system 
  - redirect 
  - response
  - delay
  - fork
  - childprocess
  - exit
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
在路由器上，lighttpd是比较常用的轻量级web服务器。今天只是对遇到一个现象进行记录，非常简单的一个case。
<p />
通常我们需要通过POST或者GET来向web服务器提交请求，服务器会设置路由来处理各种路径的POST或者GET请求。在路由器上通过是cgi或者fastcgi来处理,在收到请求后会fork一个xxx.cgi程序来处理。这个cgi与一个可执行程序没有本质区别，这里不做进一步阐述。网上资料一大把。在处理完后返回response给浏览器。浏览器通过判断response里面的信息在web页面呈现结果。
<p />

### 情景描述 

1.创建一个cgi处理程序，收到某个请求后执行system("xxxx");然后返回

2.在xxx这个程序里面做一个延时，可以是几秒也可以是更久

3.通过浏览器发送POST给这个cgi处理程序，查看等待时间。

4.通常无论何种浏览器都会在xxxx执行完成后收到response。然后刷新

<p />
### 扩展

1.将system("xxxx");改成system("xxxx&");然后返回

2.这时候所有的浏览器debug信息中显示本次POST是pending状态，或者服务器内部错误。

3.在路由器进程里面可以看到一个cgi程序处于僵死状态，等待system出来的childprocess退出。xxxx完全结束后这个cgi进程也才会结束

4.但是这个情况在不同的浏览器上表现有所不同，chrome会立刻返回，虽然response code是错误的，但是能进行下去。但是Mozilla却要一直等到cgi进程退出才能进行下去。

5.这里的请求必须是同步请求，否则看不到这个现象。

<p />
### 基本分析

通常以为加上&符号可以让程序运行在后台，而不用等待结束，但是在lighttpd的应用场景中，似乎不是简单的这么做的。比如Mozilla需要cgi进程退出才能进行下去，在login的时候是无法接受的。

<p />
### 结束语 

今天提到的情况是非常简单的一种用例，但是不知情下以为system("xxxx&");会让浏览器立刻得到200OK是一厢情愿的。


