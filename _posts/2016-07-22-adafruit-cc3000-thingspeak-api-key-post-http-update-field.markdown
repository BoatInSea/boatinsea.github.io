---
title: arduino使用CC3000模块访问并更新物联网thingspeak云 
layout: post
guid: urn:uuid:de8a34084-6f35-4f7b-bb2d-1977eb7dafff
tags:
  - arduino
  - CC3000
  - thingspeak 
  - update
  - POST
  - field
  - channel
  - Iot 
  - api
  - key
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
arduino做为杰出的开源硬件电路已经可以连接各种扩展板了，除了之前提到的ESP8266，还有今天这块CC3000 shield。当然arduino有他自己的WIFI shield，本文不做讨论。CC3000提供了通用的tcp，udp连接接口，使用adafruit库能够方便的进行tcp，udp连接，或者组成http请求。
<p />
thingspeak是一个免费的使用RESTful API的Iot公共云，通过API key可以获取和更新数据，每个API key对应一个channel，需要人为去thingspeak网站进行申请https://api.thingspeak.com。如果需要查看详细的参考请访问https://cn.mathworks.com/help/thingspeak/channels-and-charts.html。
<p />

### thingspeak channels 
数据channel需要通过一个POST API创建，如下:

        POST https://api.thingspeak.com/channels
             api_key=XXXXXXXXXXXXXXXX
             name=My New Channel

此处api\_key是thingspeak上账户的key，而不是channel的key，每个channel有自己的key。
http返回默认网页，可以看到对应channel的属性。当然也可以返回json或者xml格式的response，如:

        POST https://api.thingspeak.com/channels.json
             api_key=XXXXXXXXXXXXXXXX
                  name=My New Channel


将会返回类似下面的json数据

        {
            "id": 4,
                "name": "My New Channel",
                "description": null,
                "metadata": null,
                "latitude": null,
                "longitude": null,
                "created_at": "2014-03-25T13:12:50-04:00",
                "elevation": null,
                "last_entry_id": null,
                "ranking": 15,
                "username": "hans",
                "tags": [],
                "api_keys":
                    [
                    {
                        "api_key": "XXXXXXXXXXXXXXXX",
                        "write_flag": true
                    }
            ]
        }

channel id等于4，channel的api key也在其中，在上传或者查询channel数据的时候用得着。还有其他一些默认属性。
如果想要创建field则需要在POST的时候在body中添加其他属性，thingspeak的所有合法参数请看下面:

        api_key (string) - User's API key. Note that this is different from a channel API key, and can be found in your account details. (required).
        description (string) - Description of the channel (optional)
        elevation (integer) - Elevation in meters (optional)
        field1 (string) - Field1 name (optional)
        field2 (string) - Field2 name (optional)
        field3 (string) - Field3 name (optional)
        field4 (string) - Field4 name (optional)
        field5 (string) - Field5 name (optional)
        field6 (string) - Field6 name (optional)
        field7 (string) - Field7 name (optional)
        field8 (string) - Field8 name (optional)
        latitude (decimal) - Latitude in degrees (optional)
        longitude (decimal) - Longitude in degrees (optional)
        metadata (text) - Metadata for the channel, which can include JSON, XML, or any other data (optional)
        name (string) - Name of the channel (optional)
        public_flag (true/false) - Whether the channel should be public, default false (optional)
        tags (string) - Comma-separated list of tags (optional)
        url (string) - Webpage URL for the channel (optional)


<p />
### thingspeak channel fields

如果想在channel中使用field域名那么需要使用fieldX在POST请求，其中X代表1-8的数字。给域加上名字可以方便进行在终端进行presentation。在创建好channel后就可以对其中的field进行操作，查看或者更新，其中更新的数据将会进入云数据库，后台提供了matlab分析结果，并会以图表形式呈现在web上面。

这里要介绍两个API，一个是view的，一个是update的。

        GET https://api.thingspeak.com/channels/1417
        GET https://api.thingspeak.com/channels/1417.json

上一个请求以网页形式呈现，下一个以json数据呈现:

        {
            "id": 4,
                "name": "My New Channel",
                "description": null,
                "metadata": null,
                "latitude": null,
                "longitude": null,
                "created_at": "2014-03-25T13:12:50-04:00",
                "elevation": null,
                "last_entry_id": null,
                "ranking": 15,
                "username": "hans",
                "tags": []
        }

如果在请求中加入用户的api key那么返回信息中还将包含channel的api key。

        PUT https://api.thingspeak.com/channels/4.json
            api_key=XXXXXXXXXXXXXXXX
                name=Updated Channel


上面的api key也是账号的api key。

        {
            "id": 4,
                "name": "Updated Channel",
                "description": null,
                "metadata": null,
                "latitude": null,
                "longitude": null,
                "created_at": "2014-03-25T13:12:50-04:00",
                "elevation": null,
                "last_entry_id": null,
                "ranking": 15,
                "username": "hans",
                "tags": [],
                "api_keys":
                    [
                    {
                        "api_key": "XXXXXXXXXXXXXXXX",
                        "write_flag": true
                    }
            ]
        }

返回的api key是channel的api key。那么到底channel的api key在哪里使用呢？
<p />
### 在arduino上初始化CC3000

        #include <Adafruit_CC3000.h>
        #include <ccspi.h>
        #include <SPI.h>
        #include <string.h>
        #include "utility/debug.h"
        
        // These are the interrupt and control pins
        #define ADAFRUIT_CC3000_IRQ   3  // MUST be an interrupt pin!
        // These can be any two pins
        #define ADAFRUIT_CC3000_VBAT  5
        #define ADAFRUIT_CC3000_CS    10

        // Use hardware SPI for the remaining pins
        // On an UNO, SCK = 13, MISO = 12, and MOSI = 11
        Adafruit_CC3000 cc3000 = Adafruit_CC3000(ADAFRUIT_CC3000_CS, ADAFRUIT_CC3000_IRQ, ADAFRUIT_CC3000_VBAT,
             42         SPI_CLOCK_DIVIDER); // you can change this clock speed
        
        #define WLAN_SSID       "ap_ssid"           // cannot be longer than 32 characters!
        #define WLAN_PASS       "password"
        // Security can be WLAN_SEC_UNSEC, WLAN_SEC_WEP, WLAN_SEC_WPA or WLAN_SEC_WPA2
        #define WLAN_SECURITY   WLAN_SEC_WPA2
        
        // What page to grab!
        #define WEBSITE      "api.thingspeak.com" // See? No 'http://' in front of it
        #define WEBPAGE      "/update?key=JZS17WY5DUGK8J3E&"

上面就是久违的channel api key了。


        Adafruit_CC3000_Client www;

        在setup()中初始化

        if (!cc3000.begin())
        {
            Serial.println(F("Couldn't begin()! Check your wiring?"));
            while(1);
        }
        
        
        if (!cc3000.connectToAP(WLAN_SSID, WLAN_PASS, WLAN_SECURITY)) {
            Serial.println(F("Failed!"));
            while(1);
        }
        
        
        while (!cc3000.checkDHCP())
        {
            delay(100); // ToDo: Insert a DHCP timeout!
        }
        
        /* Display the IP address DNS, Gateway, etc. */
        while (! displayConnectionDetails()) {
            delay(1000);
        }
        
        ip = 0;
        // Try looking up the website's IP address
        Serial.print(WEBSITE); Serial.print(F(" -> "));
        while (ip == 0) {
            if (! cc3000.getHostByName(WEBSITE, &ip)) {
                Serial.println(F("Couldn't resolve!"));
            }
            delay(500);
        }
        
        cc3000.printIPdotsRev(ip);

        www = cc3000.connectTCP(ip, 80);
        if (www.connected()) {
            // do something
        }

<p />
### 定义连接格式

一次普通POST

        www.fastrprint(F("POST "));
        www.fastrprint(F("/"));
        www.fastrprint(F(" HTTP/1.1\r\n"));
        www.fastrprint(F("Host: "));
        www.fastrprint(WEBSITE);
        www.fastrprint(F("\r\n"));
        www.fastrprint(F("\r\n"));
        www.println();


一次update field post


        update("88", "field1");//更新 channel的第一个field的数据
        cc3000.disconnect();

        //update 函数定义如下
        void update(String value, String field){
           String cmd = field+"="+value;
           Serial.println("in the update loop");
           www = cc3000.connectTCP(ip, 80);
           if (www.connected()) {
                www.fastrprint(F("POST "));
                www.fastrprint(WEBPAGE);
                www.print(cmd);
                www.fastrprint(F(" HTTP/1.1\r\n"));
                www.fastrprint(F("Host: "));
                www.fastrprint(WEBSITE);
                www.fastrprint(F("\r\n"));
                www.fastrprint(F("\r\n"));
                www.println();
            } else {
               Serial.println(F("Connection failed"));
               return;
           }
           www.close();
        }

编辑好ino文件后用arduino IDE编译烧入就可以调用程序了。提醒一点是thingspeak每次update需要中间延迟一段时间，大概15s以上，保险起见可以delay 17-20s。
<p />
### 上api.thingspeak.com检查结果
在thingspeak上分成private channel和public channel，如果是public只要用浏览器打开https://api.thingspeak.com/channels/channel\_id就可以了。
如果是private channel则需要在GET api中使用用户的APIKey，才能返回一个json或者xml答复。或者通过登录api.thingspeak.com来查看了。

