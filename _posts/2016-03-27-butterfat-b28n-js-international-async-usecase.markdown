---
title: 在网页上使用b28n.js实现多语言支持 
layout: post
guid: urn:uuid:de8d8abd-6f35-4f7b-bb23-1951062dadfc
tags:
  - b28n 
  - js
  - async
  - international 
  - lang
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
b28n.js是一个多语言支持模板，全名叫做butterfat internationalization。是一个通过http GET来获取web界面元素语言描述xml来进行字典式翻译的js工具。下面通过一个简单例程来演示如何使用：

         <html>
            <head>
            <title>I18N Test</title>
            <script type="text/javascript" src="b28n.js"></script>
              <script type="text/javascript">
                Butterlate.setTextDomain("messages","http://butterfat.net/~bmuller/i18n");
                function startUp() {
                  var test = document.getElementById("test");
                  test.innerHTML = _("this is english");
                }
            </script>
            </head>
            <body onload="startUp();">
               <div id="test">
            </body>
         </html>

理论上在http服务器上放好多语言的xml文件，格式为

        <message msgid="this is english" msgstr="this is spanish" />

就能实现翻译了。不过这个js工具时间可能比较早不支持异步的http GET。对于如今大多数浏览器，在主线程中不能进行同步的http GET。所以发现翻译不起作用。于是要做的就是b28n.js的异步处理。
<p />

###  1. 修改request为异步并添加回调函数 

        request.open("GET",this.po,true);//true意味是异步请求
        request.onload = function (e) {
          if (request.readyState === 4) {
            if (request.status === 200) {
              console.log(request.responseXML);
              var pos = request.responseXML.documentElement.getElementsByTagName("message");
              for(var i=0; i<pos.length; i++) {
                //setter 是setTextDomain传入的参数，指代this.dict
                setter.set(pos[i].getAttribute("msgid"),pos[i].getAttribute("msgstr"));
                console.log("pos[i]"+pos[i].getAttribute("msgid")+"|"+pos[i].getAttribute("msgstr"));
              }
              func();//给外部用的回调，用来做翻译赋值
            } else {
              console.error(request.statusText);
            }
          }
        };
        request.onerror = function (e) {
          console.error(request.statusText);
        };

上面并没贴出所有代码，但是核心代码都已经有了。JavaScript还可以优化，功能上基本完成。

###  2. setTextDomain提供翻译回调和b28n的字典变量（onload函数访问不到b28n的字典变量） 

        this.setTextDomain = function(domain, func, setter) {.....}

修改setTextDomain参数来实现。
<p>
</p>

### 3. 放置setTextDomain到html的body onload里面 

        <body onLoad="initValue()">

把显示初始化的内容放入onLoad就可以了，一旦完成http GET，翻译赋值也就完成了，web会被刷新成新的语言。

