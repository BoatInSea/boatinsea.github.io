---
title: Github 开篇-编译MT7628 image并且bring up开发板
layout: post
guid: urn:uuid:de8d598d-6f35-4c7b-ab23-1951062dadfc
tags:
  - router 
  - AP 
  - MT7628
  - compile
  - SDK 
---
#  Head1

##  Head2

![Alone](/media/files/2016/3/alone.jpg)
<p />

###  1. MT7628 SDK 4.3.1.0 编译

<p>
由于有SDK的User Manual参考，所以主要还是参考用户手册，此文只是提一下需要注意的点。
</p>

####  环境

1.对于MT7628, gcc-4.6.3需要被安装到/opt/下面。

2.务必安装toolchain中的lzma到gcc-4.6.3/usr/bin下面。

3.make squash fs似乎gcc的bin目录下已经有了，如果不放心可以编译出来后替换或者拷贝过去。

4.Uboot用的是gcc-3.4.2。

####  芯片

1.在source目录下进行make menuconfig可以调出配置菜单。

2.选中默认设置与内核定制，确保linux2.6.36.x中选择了Wifi，并且使用AP模式。

####  DDR Flash

Product中选择MT7628,Flash和DDR选择上首先关注的是DDR的大小，菜单中是MB单位，所以可能是64MB，对于Flash,如果不是Dual Image模式就不用在意大小，如果是Dual Image，在后面配置的时候会有选择Flash大小的选项。

####  WLLLL

确保配置是与你所用的开发板的WAN，LAN设置一致。

1.理论上配置好后调用make dep;make就能编译出两个image: zImage.lzma, user\_uImage(此处也得确保使用lzma压缩而不是gzip压缩).

2.烧入Flash切记使用User\_uImage。

###  2. 坑与解决

####  编译错误

<p>
编译出错主要是wifi驱动加入内核后编译失败，要使编译通过只需要改一个Kconfig就行，也就是位于linux.2.6.36.x/ralink/下面。打开注释
</p>
        source /driver/net/wireless/mt_wifi/embedded/Kconfig
<p />

####  SSID找不到

<p>
如果烧入好之后发现找不到默认的MT7628\_AP,那么需要确认如下几个情况：
</p>

1.ifconfig能否看到ra0，如果没有使用ifconfig ra0 up。

2.如果能看到ra0，但是依然看不到SSID，那么可以查看'source/vendor/Ralink/RT2860AP/RT2860\_default\_vlan'文件.把'Channel'与'AutoChannelSelect'改成1。然后重新编译烧入再看。

<p />

###  3. 连上MT7628\_AP Enjoy 

<p />
编后话，切记编译MT7628不要使用SDK 4.3.0.0，会有各种编译问题，特别是uclibc不能修改成0.9.33.2,还有不少user application不能编译成功。
