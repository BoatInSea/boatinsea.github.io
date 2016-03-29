---
title: opkg安装ipk遇上overlay空间不足问题的解决方案 
layout: post
guid: urn:uuid:de8d898d-6f35-4f7b-bb23-1951eb7dadfc
tags:
  - openwrt 
  - opkg
  - overlay
  - opkg.conf 
  - mount
  - SD card
  - ram
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
openwrt有个非常好的安装包管理软件opkg，只要选定好source后就跟其他unix系统中的管理软件一样好用，而且opkg还能选择安装路径,相当灵活。但是openwrt得存储空间是有限的，于是就会遇上overlay安装空间不足的问题了。
<p />
岔开一下，overlay顾名思义是覆盖的意思，overlay在分区上占用了一个可读写的独立分区，在mount的时候覆盖到/根目录下面。等于是在该独立分区的文件与根目录下的文件系统是共用了一个路径。安装到overlay中的bin在/bin下面也会看到。这样用户数据就存放在overlay分区，如果要恢复出厂设置，只需要清空overlay目录就行了。非常方便。
<p />
但是openwrt系统中的存储空间是有限的，比如Flash为16MB，那么实际可供读写的overlay分区可能只有9MB。在这种情况下安装大型一点的ipk安装包可能就不行了。但是幸好opkg提供了灵活的安装路径选择，通过修改opkg.conf我们可以添加安装路径把ipk安装到其他地方，比如SD卡。
<p />

###  添加安装路径解决安装空间不够的问题 

<p>
打开/etc/opkg.conf文件，我们可以看到如下的结构。
</p>

        dest root /
        dest ram /tmp
        lists_dir ext /var/opkg-lists
        option overlay_root /overlay
        option check_signature 1

前两行就是安装路径，第一行是默认路径，第二行需要在opkg命令后面加上-d ram才能起作用。为了安装ipk到SD卡，我们需要添加一行

        dest usb /tmp/run/mounted/mmcblk0p1/system

假设/tmp/run/mounted/mmcblk0p1为SD卡的挂载点，安装在SD卡的system目录内。于是我们可以这样安装一个ipk到SD卡。

        opkg install ffmpeg -d usb

如果遇上SD卡不能安装怎么处理。其实方法已经提到了，就是使用-d ram。安装到tmp目录下。只要设置好PATH环境变量一样可以运行安装的软件。如果ipk只是一个库，那么环境变量是LD_LIBRARY_PATH。

###  opkg安装ipk的几种常用方法

1.通过官方网站安装ipk包

        opkg update 
        opkg install xxxx 

2.通过文件系统本地安装ipk

        opkg install /tmp/xxxx-xxx.ipk

3.通过http或者ftp来安装ipk

        opkg install http://192.168.1.222:8080/xxxx-xxx.ipk
        opkg install ftp://192.168.1.222/xxxx-xxx.ipk

