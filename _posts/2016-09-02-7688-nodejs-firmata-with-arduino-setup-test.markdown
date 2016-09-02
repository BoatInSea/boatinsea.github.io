---
title: 在MT7688上使用firmata协议与arduino通信 
layout: post
guid: urn:uuid:de8a34084-6f35-4f7b-bb2d-197b7dadfcee
tags:
  - firmata
  - arduino
  - node 
  - LinkIt 
  - ttyS
  - connect
  - sysex
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
在[Smart Linkit的Github链接](https://iamblue.gitbooks.io/linkit-smart-nodejs/content/zh-TW/basic/firmata.html)上，我们可以看到如何使用node与firmata与arduino进行通信。
但是实际上有比这个更方便的方案，我们只要安装[https://downloads.openwrt.org/chaos\_calmer/15.05.1/ramips/mt7688/packages/packages/](https://downloads.openwrt.org/chaos_calmer/15.05.1/ramips/mt7688/packages/packages/)目录下的serialport以及arduino-firmata就可以正常进行通信了。
<p />

### 硬件连接

1.各自上电

2.连接7688的GND---arduino的GND， 7688的UART1的RX-----arduino的TX, 7688的TX-----arduino的RX

>此处arduino的RX和TX可以是硬件串口或者软件串口的

<p />
### 测试用例
本测试用例使用了sysex消息格式，包括了发送和接收，但是未做处理。arduino需要能处理USER\_DEFINED\_COMMAND消息。

        var ArduinoFirmata = require('arduino-firmata');
        var board = new ArduinoFirmata().connect('/dev/ttyS1', {baudrate: 57600}); //可以自定义波特率
        board.on('connect', function(){
          console.log("connect!! "+board.serialport_name);//connect消息意味着arduino发送了版本信息上来
        });
        board.on('sysex', function(e){
          console.log("command : " + e.command);
          console.log("data    : " + e.data);
          handler(e.data);
        });
        var sendCommand = function(data) {
           var ddata = new Buffer (firmata.Board.encode(data));
           board.sysex(USER_DEFINED_COMMAND, ddata);
        }
<p />
### Done

`node app.js`可查看结果.
<p/>
### 参考
1.https://github.com/shokai/node-arduino-firmata/blob/master/samples/sysex/StandardFirmataWithLedBlink/StandardFirmataWithLedBlink.ino
2.http://firmata.org/wiki/V2.1ProtocolDetails#Sysex\_Message\_Format
3.https://github.com/shokai/node-arduino-firmata


