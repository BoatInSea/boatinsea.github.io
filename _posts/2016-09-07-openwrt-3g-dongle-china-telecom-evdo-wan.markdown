---
title: 在mt7688上使用电信3G dongle上网
layout: post
guid: urn:uuid:de8a34084-6f35-4f7b-bb2d-197b7dadffff
tags:
  - mt7688
  - 3G
  - dongle 
  - 电信 
  - evdo
  - network
  - wan
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
以前在android环境上bring up过联通的3G，需要自己写一些连接与断开的脚本，需要判断是否usb设备转换成功，需要守护进程rild来进行断线重连，自己干的事情不少。现在到手的一块Widora非常不错，已经在上面玩了不少东西，也借机了解了openwrt的基础，比如编译，添加自定义文件，升级，opkg的使用。这次手上有个电信的3G dongle正好温顾一下自己对3G dongle的知识。

此文使用的openwrt版本为`chaos_calmer 15.05.1`，不同版本的op会有不同的操作，至少usb的mode switch我看到就不一样。

### Kernel准备

为了方便建议使用kernel重编译增加module的方式，因为后期opkg安装会比较麻烦，下次重烧image就得再来一遍。而且flash够大，无需担心跟usb转串口相关的那些kernel module的size的。一次编译终身使用。
在[Openwrt wiki的这篇文章](https://wiki.openwrt.org/zh-cn/doc/howtobuild/wireless-router-with-a-3g-dongle)里，详细描述了如何编译出一个支持3g&4g dongle的linux kernel。
如果完全按照这篇文章操作现在应该已经可以得到一个新的image了。使用uboot命令烧入好准备下一步测试。

<p />

### 插入设备识别

1.接上OTG USB转接线（因为板子上是miniusb口）

2.插入之前的联通的3G dongle，查看得到如下串口信息

        [  154.530000] usb 2-1: new full-speed USB device number 4 using ohci-platform
        [  154.750000] usb-storage 2-1:1.0: USB Mass Storage device detected
        [  154.770000] scsi host2: usb-storage 2-1:1.0
        [  155.200000] usb 2-1: USB disconnect, device number 4
        [  156.760000] usb 2-1: new full-speed USB device number 5 using ohci-platform
        [  156.980000] usb-storage 2-1:1.0: USB Mass Storage device detected
        [  157.000000] option 2-1:1.0: GSM modem (1-port) converter detected
        [  157.000000] usb 2-1: GSM modem (1-port) converter now attached to ttyUSB0
        [  157.010000] usb-storage 2-1:1.1: USB Mass Storage device detected
        [  157.020000] option 2-1:1.1: GSM modem (1-port) converter detected
        [  157.030000] usb 2-1: GSM modem (1-port) converter now attached to ttyUSB1
        [  157.040000] usb-storage 2-1:1.2: USB Mass Storage device detected
        [  157.040000] option 2-1:1.2: GSM modem (1-port) converter detected
        [  157.050000] usb 2-1: GSM modem (1-port) converter now attached to ttyUSB2
        [  157.070000] usb-storage 2-1:1.3: USB Mass Storage device detected
        [  157.070000] scsi host6: usb-storage 2-1:1.3
        [  158.080000] scsi 6:0:0:0: Direct-Access     WCDMA    MMC Storage      2.31 PQ: 0 ANSI: 2
        [  158.100000] sd 6:0:0:0: [sda] Attached SCSI removable disk

3.插入电信的3G dongle，得到如下串口信息

        [   14.160000] usb 2-1: new full-speed USB device number 6 using ohci-platform
        [   14.380000] usb-storage 2-1:1.0: USB Mass Storage device detected
        [   14.400000] scsi host7: usb-storage 2-1:1.0
        [   14.840000] usb 2-1: USB disconnect, device number 6
        [   16.400000] usb 2-1: new full-speed USB device number 7 using ohci-platform
        [   16.610000] usb 2-1: config 1 has an invalid interface number: 5 but max is 4
        [   16.620000] usb 2-1: config 1 has no interface number 4
        [   21.680000] option 2-1:1.0: GSM modem (1-port) converter detected
        [   21.680000] usb 2-1: GSM modem (1-port) converter now attached to ttyUSB0
        [   21.690000] option 2-1:1.1: GSM modem (1-port) converter detected
        [   21.700000] usb 2-1: GSM modem (1-port) converter now attached to ttyUSB1
        [   21.710000] option 2-1:1.2: GSM modem (1-port) converter detected
        [   21.710000] usb 2-1: GSM modem (1-port) converter now attached to ttyUSB2
        [   21.750000] option 2-1:1.3: GSM modem (1-port) converter detected
        [   21.750000] usb 2-1: GSM modem (1-port) converter now attached to ttyUSB3
        [   21.760000] usb-storage 2-1:1.5: USB Mass Storage device detected
        [   21.770000] scsi host1: usb-storage 2-1:1.5
        [   23.110000] scsi 1:0:0:0: Direct-Access     OEM      Qualcom EVDORevA 2.31 PQ: 0 ANSI: 0
        [   23.140000] sd 1:0:0:0: [sda] Attached SCSI removable disk

<p />
神奇的是，根本没有使用mode switch，都成功转成dongle设备了。老版的OP可能是需要手工介入的。
<p />
### 编写脚本
我们需要把dongle的interface作为wan口，所以会有如下的脚本，原型是芒果的`eth_set_wan`和`connect2ap`，感谢这些工作，等于有了学习的范本。对于加快openwrt开发非常重要。脚本首先设置网口为lan，如果之前设置成wan了话最好如此，如果之前作为apclient，那就先禁用这个功能，剩下就是把`3g-wan`接口设置成wan，然后设置wan的其他属性，协议为3g，服务为evdo，设备是ttyUSB0，apn是'ctnet'，启动自动拨号连接。提交后network restart。

        #!/bin/sh
        clear
        eth_set_lan
        sleep 2
        killall ap_client
        uci set wireless.sta.disabled='1'
        uci set network.wan=interface
        uci set network.wan.ifname='3g-wan'
        uci set network.wan.proto='3g'
        uci set network.wan.service='evdo'
        uci set network.wan.device='/dev/ttyUSB0'
        uci set network.wan.apn='ctnet'
        uci set network.wan.username='card'
        uci set network.wan.password='card'
        uci set network.wan.auto='1'
        uci commit
        nr

因为手头没有4G卡，我看到LTE可能proto是用的'qmi'的，然后设备也可能叫不同于tty打头的，比如'/dev/cdc-wdm0'。以后有机会再试。
<p />
### 测试
万事具备，调用3g\_set\_wan,等待拨号成功，大概10几秒后，发现增加了3g-wan接口，并且获得了相应的ip地址。测试：

        ping www.baidu.com
        PING www.baidu.com (115.239.210.27): 56 data bytes
        64 bytes from 115.239.210.27: seq=0 ttl=56 time=90.375 ms
        64 bytes from 115.239.210.27: seq=1 ttl=56 time=128.098 ms
        64 bytes from 115.239.210.27: seq=2 ttl=56 time=142.851 ms
        64 bytes from 115.239.210.27: seq=3 ttl=56 time=125.608 ms
        64 bytes from 115.239.210.27: seq=4 ttl=56 time=142.354 ms
        64 bytes from 115.239.210.27: seq=5 ttl=56 time=129.102 ms
        --- www.baidu.com ping statistics ---
        6 packets transmitted, 6 packets received, 0% packet loss
        round-trip min/avg/max = 90.375/126.398/142.851 ms2
成功。
<p/>





