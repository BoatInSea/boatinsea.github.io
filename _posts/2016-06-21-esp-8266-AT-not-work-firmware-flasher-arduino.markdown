---
title: 在ESP8266使用中遇到的问题及使用arduino刷新固件 
layout: post
guid: urn:uuid:de8a34084-6f35-4f7b-bb23-1951ef12adfc
tags:
  - ESP8266 
  - arduino
  - AT 
  - UART 
  - GPIO 
  - firmware 
  - Flasher
  - USB
  - TTL
  - COM
  - Passthrough
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
目前市场上已经有了成熟的WiFi低功耗Iot解决方案，手痒买了一块ESP8266玩一下。配合arduino的话一般使用的是arduino的软件串口+ESP8266的串口WiFi(AT指令方式)功能。于是根据ESP8266的说明进行了连线。写了个测试程序，结果不用问，失败。ESP8266在使用中有些讲究，其中串口的波特率就是第一个要关心的问题，随后是AT指令的支持。知道这些后首先进行了一些验证来确认问题。
<p />

### ESP8266的8个pin脚的连线 

1.pin脚定义

![Alone](/media/files/2016/6/esp8266pcb.jpg)

2.正确的供电与连接

        VCC     ----------->    3.3v
        GND     ----------->    GND
        CH_PD   ----------->    3.3v 可以与VCC用线直连,高电压表示进入UART AT指令模式
        GPIO0   ----------->    悬空，如果为低电平则芯片进入烧入固件模式，这个后面章节会再降到
        GPIO16  ----------->    悬空，reset脚，给一个高->低->高电平就会重启芯片 
        GPIO2   ----------->    悬空，可以进行GPIO操作，作为input或者output
        UTXD    ----------->    连接host的RX, esp8266串口发送线
        RTXD    ----------->    连接host的TX, esp8266串口接收线 

注意：千万不可以接5v电压，另外与外部设备接串口的时候3.3v可以分别供电，但是GND一定要连在一起，否则会导致数据错误。


### 检验ESP8266串口

1.使用MT7688连接ESP8266检测串口情况，首先两边把地连在一起，arduino UNO有三个GND，随便选一个，然后ESP8266的TX接7688的RX，ESP8266的RX接7688的TX。

2.按照前一节，连接好esp的pin脚，如果使用arduino供电，请使用3.3v的供电，arduino的地有三个，可以连一个到7688，另一个连esp的地。还能富裕一个。

3.设置7688的串口波特率为115200，使用cat /dev/ttyS1 &来接收数据，使用echo AT > /dev/ttyS1来写指令。

4.插拔一下esp的VCC，这个时候7688的串口打印了很多的乱码，但是最后可以看到Ai think ........, ready. 这说明启动成功了，前面的乱码是启动过程的波特率不是115200造成的。期间你也可以看到esp8266的蓝灯闪烁，蓝灯连接的是esp8266的TX，说明在往串口发送数据。

5.这个时候使用命令echo AT > /dev/ttyS1会发现总是返回ERROR(由cat /dev/ttyS1抓到)。只能进行到这一步了。

6.这个时候可以看到有个AITHINK开头的SSID，并且能连上，也能ping通，应该可以证明固件是好的，就是AT指令有问题。可以考虑重新刷firmware了。

<p />
### 刷Firmware的尝试

我有一个usb转串口工具，有四个针脚，分别是3.3, TX, RX, GND,我尝试接上ESP8266后发现115200部分的字符串也经常出现乱码，不确定原因所以担心刷firmware的时候出现误码导致升级失败(事后我再试过，似乎刷成功了)。在网上帖子看到可以使用arduino做USB串口透传觉得可能更靠谱一些。于是进行了连接，并且参考别人的程序给arduino做了初始化。
<p />
连接图如下:

![Alone](/media/files/2016/6/arduino-esp.jpg)

参考程序如下:

        int ch_pd = 3;
        int gpio0 = 2;
        int rst = 4;
        
        void setup() {
            pinMode(ch_pd, OUTPUT);
            pinMode(gpio0, OUTPUT);
            pinMode(rst, OUTPUT);
            digitalWrite(ch_pd, HIGH);
            digitalWrite(gpio0,LOW);
            //RESET ESP8266
            digitalWrite(rst,HIGH);
            delay(50);
            digitalWrite(rst,LOW);
            delay(200);
            digitalWrite(rst,HIGH);       
        }
        
        void loop()
        {
        }

在arduino上通过给数字pin输出高电压也能提供3.3v供电, 或者输出低电平等于提供了一个GND。

### 注意: 在使用arduino做USB转串口passthrough功能时候必须是ESP8266的TX接arduino的TX，RX接RX才行。

我的ESP8266板子并没有连接ch\_pd这个D3，因为我自己把ESP8266的VCC跟CH\_PD直接用线焊接在一起了。程序中把GPIO0拉低进入烧写模式，重启ESP8266后就可以使用工具烧入firmware了。

在网上可以找到各种版本的firmware，不管版本号，主要目前是两种烧入，单文件，多文件。单文件只要把文件写入起始0x00000位置即可，多文件则需要写入多个偏移地址。全部烧写完毕才行。第一次找的v1.1版本烧入成功，但是同样AT不工作，所以在我买ESP8266-12的淘宝店提供的百度盘链接上面找了个最新版，0.9.25，波特率为9600，分成两个bin文件烧入。百度盘地址如下:

        http://pan.baidu.com/s/1nt0rVIT

下载解压后里面自带烧入步骤说明。我没有自己抓图，所以用网上找的烧入图示意一下

![Alone](/media/files/2016/6/advance.png)

![Alone](/media/files/2016/6/config.png)
我下载的firmware使用了两个bin，分别烧到0x00000和0x40000。右边齿轮图标可以浏览目录，左边打叉表示要进行烧入操作。

![Alone](/media/files/2016/6/flasher.png)
选择好COM口，我的arduino一直是COM28，这里有两个点要注意:

1.在烧入进行前先插拔一下USB线，这个时候不要连接两根串口线，等到打开烧入工具能找到COM28后再把串口线插上。

2.确保电脑能上网，似乎需要上网获取MAC地址才能进行烧入。如果不能获取到MAC地址，包括左边的二维码，那么就会一直等待。这是我测试多次失败后找到的问题。

3.点击Flash(F)按钮, 等到进度条满格后，右下角会出现Ready字样，表示成功。我相信这个工具有烧入验证功能，如果能成功就应该是firmware升级成功了。


### 最终结果

进行正常使用请先关闭烧入工具，然后把GPIO0连线断开。用串口调试工具打开串口28，并设置波特率为9600（我用SSCOM），插拔一下ESP8266的VCC线，可以看到,打印信息了，最后Ready出现了，随后输入"AT"，然后"OK"出现了，然后你就知道该怎么玩了。


