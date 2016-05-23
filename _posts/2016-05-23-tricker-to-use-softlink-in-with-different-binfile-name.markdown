---
title: 在小系统上使用符号链接来扩展和简化命令行应用程序功能 
layout: post
guid: urn:uuid:de8d898d-6f35-4f7b-bb23-1951eb7dadfc
tags:
  - MTK 
  - ralink\_init
  - nvram\_get
  - nvram 
  - ln 
  - argc 
  - argv
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
好久没有更新了，主要也是最近的工作比较偏重复劳作。没有太多值得提的。这次的topic也是非常简单轻松的。主要是讲如何使用软链接来扩展命令行的功能命令。
<p />
在MT7628上开发了一段时间，主要是web配置（内容比较繁杂，包括wifi配置，wps配置，wan配置等等），当然还有GPIO操作。不经意间看到一个在linux上比较多的小把戏。想象一下这样一种场景：有一个bin比如ralink\_init，里面做了很多的扩展，比如修改路由器工作模式，获取nvram的key值，设置nvram的key值，等等等等。如果你要通过参数来识别就要增加非常多的子命令，类似于busybox的applet，然后busybox知道该进入那个branch做对应的工作。这个会让人比较烦恼，因为命令行下打字麻烦了很多。所以下一章节讲讲怎么简化这种情况。同样适用于改进busybox的命令。
<p />

### 代码示例 

<p>
创建一个c文件如下, 其中参数校验，以及native\_nv\_printf和native\_nv\_show的实现不贴出来了：
</p>

        void main (int argc, char* argv) {
            char* cmd;
            cmd = argv[0];
            if (!strcmp(cmd, "nvram_print")) {
                native_nv_printf(argc, argv);
            } else if (!strcmp(cmd, "nvram_show")) {
                native_nv_show(argc, argv);
            }
        }

<p />
### 创建软链接 


        ln -sf ralink_init nvram_print         
        ln -sf ralink_init nvram_show 


<p />
### 测试 

        nvram_print WebInit
