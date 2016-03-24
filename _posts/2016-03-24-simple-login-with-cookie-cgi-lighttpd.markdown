---
title: 在lighttpd中使用cgi和cookie实现简单登录 
layout: post
guid: urn:uuid:de8d898d-6f35-4f7b-bb23-1951062dadfc
tags:
  - lighttpd 
  - cookie
  - redirect
  - login 
  - cgi
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
最近在用lighttpd加cgi做路由器的网页管理，一开始使用mod\_auth来做登录认证，发现欠缺美观与灵活。由于该模块使用的是浏览器自带的对话框来做登录框非常原始，不方便定制化。而且虽然支持多种验证方式但是针对nvram的存储方式缺乏支持（至少目前了解如此）。基于此，想着利用cookie来做简易的认证结果存储，于是便有此文。
<p />

###  1. 环境 

<p>
操作系统为Ubuntu 14，lighttpd使用1.4.20（为了与7628上面的版本一致）。制作一个index.html文件，包含一个简单的post form，提交给cgi\_bin目录下的login.cgi。再建立一个whatever.html如果登录成功就跳转到此页面，否则返回index.html。
</p>

###  2. 前端cookie处理 

<p>
包含了index部分的cookie重置以及whatever.html里面的cookie教研两部分。
</p>

1.index.html中添加JavaScript代码

        document.cookie = "user=0";
        //保证回到index.html后cookie被reset

2.whatever.html中需要检验收到的cookie是不是包含了正常的用户名等信息。此时我只放置了用户名，不包含密码等信息。

        var one, end;
        one = document.cookie.indexOf("user");
        end = (document.cookie.indexOf(';',one)!=-1) ? document.cookie.indexOf(';',one) : document.cookie.length;
        var login = unescape(document.cookie.substring(one+5,end));
        if(login=="0" || document.cookie.length == 0) {
            parent.location.href="/index.shtml";
        }
        //此段代码用来防止未经index.html认证就访问此页面

### 3. cgi cookie处理 

1.本文关注cookie所以此处不讲用户验证逻辑

2.认证成功后的跳转逻辑如下代码所示

        printf("HTTP/1.0 302 Redirect\r\n");
        printf("Location: %s\r\n", /whatever.html);
        printf("Set-Cookie: user=administrator; path=/\r\n\r\n");// !!!两次换行

### 4. 小问题 

<p>
Set-Cookie:必须包含path否则一切都是浮云。目前在FF, chrome上面都能工作。
</p>

