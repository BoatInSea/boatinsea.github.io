---
title: 在7688上用node来做GPIO控制 
layout: post
guid: urn:uuid:de8a34084-6f35-4f7b-bb23-1951eb7dadfc
tags:
  - LinkIt 
  - MT7688 
  - OpenWrt 
  - GPIO 
  - nodejs 
  - libmraa 
  - mraa
  - iot
  - intel
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
MT7688系列提供了丰富的GPIO pin脚，当我们需要进行GPIO控制的时候有多种方法，MTK SDK提供了gpio接口，能够直接访问GPIO，并设定中断模式。在OpenWrt中也有相应的GPIO控制方式，主要通过/sys/class/gpio这个类来操作，通过echo传入export来选择控制哪个GPIO。但是这种是基于bash层面的操作。在OpenWrt环境中想要通过代码（C或者JavaScript）来控制GPIO该如何做呢？
<p />
Intel提供了一个mraa的接口，请参考:

    https://github.com/intel-iot-devkit/mraa

我们今天讨论的是如何在7688上使用mraa。
<p />

### 7688的MRAA在哪里 

1.我们在intel-iot-devkit上寻找

    Supported Boards
    ----------------------
    X86
    ----------------------
    Galileo Gen 1 - Rev D
    Galileo Gen 2 - Rev H
    Edison
    Intel DE3815
    Minnowboard Max
    NUC 5th generation
    UP
    ----------------------
    ARM
    
    Raspberry Pi
    Bannana Pi
    Beaglebone Black
    ----------------------
    USB
    
    FT4222

7688不在此列

2.在OpenWrt官网寻找

目前没有找到

3.在联发科网站上寻找

    OpenWrt-SDK-ramips-mt7688_gcc-4.8-linaro_uClibc-0.9.33.2.Darwin-x86_64.tar.bz2

下载后发现了两个文件，似乎就是我们想要的，一个是libmraa.so，另一个是node\_modules里面的mraa，内部是一个mraa.node库文件。
似乎问题解决了，我们可以写一个c程序一个JavaScript程序来分别验证。

### 配置环境 

    ssh root@7688host  //或者通过ttyS0登录
    scp -r user@xx.xx.xx.xx:/home/user/mraa/libmraa.so /usr/lib/ 
    scp -r user@xx.xx.xx.xx:/home/user/mraa/node_modules/mraa /lib/node_modules/ 
    scp -r user@xx.xx.xx.xx:/home/user/mraa/mraa_gpio /usr/bin
    scp -r user@xx.xx.xx.xx:/home/user/mraa/mraa_gpio.js /usr/bin


javascript程序代码如下，特别注意的是，这个pin必须被设置成GPIO工作模式，否则无效。
    
    var m = require('mraa'); //require mraa
    console.log('MRAA Version: ' + m.getVersion()); //write the mraa version to the console

    var myDigitalPin = new m.Gpio(5); //setup digital read on pin 5
    myDigitalPin.dir(m.DIR_OUT); //set the gpio direction to output
    myDigitalPin.write(1); //set the digital pin to high (1)

<p />
### 遇上问题

无论是用libmraa.so还是用mraa.node，通通失败了。特别是node返回new m.Gpio(5)报异常为错误的参数。于是进入单步模式

    >node
    >var m = require('mraa')
    > .............//一大串的函数信息。
    >m.getPlatformType()
    >99

知道原因了，板子没有被识别出来，所以gpio count为0，所以传入参数5就报错了。继续查下去，因为没有LinkIt 7688的mraa库的源代码，所以只能使用intel-iot-devkit上的查看，发现有个arm.c是针对ARM芯片，然后还有对于树莓派的c文件。intel-iot-devkit代码不支持mips芯片，参考arm.c其实里面是查看了/proc/cpuinfo里面的machine信息来判断platform type的。打开mraa.node文件可以看到里面也是查看了cpuinfo中的machine字段，内容为MediaTeck Linkit 7688 Smart。现在两个选项，修改machine字段，或者修改mraa.node的源代码改成自己板子的machine。 不过在看到mraa.node中出现了/sys/class/gpio/export, value, unexport等字段后，我相信其实LinkIt 7688的mraa对GPIO的操作是用bash脚本方式控制的。 所以何必绕弯子去通过mraa来操作GPIO，使用nodejs的childprocess的spawn就能实现了。测试后完全能正常工作。

### 结束语 

联发科自己定制化了7688用于物联网系统，所以联发科网站上SDK中一些非通用的东西没法直接是用，这个得记住。

### 提醒 
特别注意的是，这个pin必须被设置成GPIO工作模式，否则无效。

