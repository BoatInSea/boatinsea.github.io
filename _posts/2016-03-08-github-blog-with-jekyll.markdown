---
title: 使用Jekyll建立静态博客 
layout: post
guid: urn:uuid:de8d898d-6f35-4c7b-ab23-1951062dadfc
tags:
  - jekyll 
  - gem 
  - pages
  - bundler 
  - blog 
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />

此文用来记录博主自己的搭建过程，以作备案，并供网络分享。
<p />

###  1. 安装Ruby 

<p>
由于使用Mac，系统再带了ruby，所以跳过此步。
</p>

###  2. 安装bundle 

<p>
bundle 是一个包工具，可以使用bundle来把所有Jekyll依赖的模块都安装进一个特定的目录下面。有了bundle整个过程就易如反掌了。
</p>

1.调用gem install bundle

2.在工作目录下创建一个Gemfile文件，也可以使用bundle init来生成一个默认的Gemfile文件。 

3.修改Gemfile内容为:
    
        source 'https://ruby.taobao.org/'
        gem 'jekyll'
        gem 'github-pages'

4.运行bundle install,成功后会把所有依赖包都下载并安装好。

### 3. 运行Jekyll

1.cd Jekyll 工作目录.

2.运行 bundle exec jekyll serve.

3.访问http://127.0.0.1:4000,假如jekyll目录已经有正确的jekyll博客模板,就能看的你想看的内容.

4.如果运行出错，可以尝试把之前的Gemfile拷贝到jekyll工作目录。 

### 4. pages.github.com 

1.在github工作根目录下创建一个新的工程为username.github.io。

2.模仿pages.github.com的步骤测试一个index.html，然后访问https://username.github.io

3.看的index.html能正常显示说明github工作目录完成。username应该全小写。

### 5. 启用Jekyll模板

1.搜索github上面中意的jekyll模板fork到本地jekyll工作目录

2.在本地4000端口测试好博客内容后push到username.github.io。

### 6. 访问https://username.github.io

### 7. 注意事项

<p>
在markdown语法上jekyll模拟环境与github上会有些不同，就目前看到的现象，要注意在github上面标题，列表之间一定要加上空行，否则显示一定会出问题。
</p>



