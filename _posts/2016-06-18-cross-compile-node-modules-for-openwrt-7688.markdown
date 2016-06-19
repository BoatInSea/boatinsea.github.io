---
title: 为MT7688 OpenWrt交叉编译node modules 
layout: post
guid: urn:uuid:de8d8084-6f35-4f7b-bb23-1951eb7dadfc
tags:
  - MTK 
  - MT7688 
  - OpenWrt 
  - Cross compile
  - nodejs 
  - modules 
  - toolchain
---

![Alone](/media/files/2016/3/shangrila.jpg)
<p />
在使用OpenWrt的时候，opkg给我们很多软件支持，常用软件几乎都能找到。对于喜欢用nodejs的人来说现在可以在MT7688的trunk上找到node\_v0.12.7-1\_ramips\_24kec.ipk(印象中v4.4.x也已经ready了), 对于一般的web应用已经足够。唯一遗憾的是可供node调用的module还是太少，目前看到也就cylon，firmata，serial模块能找到。其他都没有。对于有特殊需要apps肯定是不够用的。本篇来讲述如何给OpenWrt编译node modules。
<p />
本篇参考的是:

    https://github.com/netbeast/docs/wiki/Cross-Compile-NPM-modules#cc

在链接中能找到的内容我就不做描述，只讲述我在编译中遇到的问题和处理经过。希望能给人帮助
<p />

### 前期准备 

1.站点选择 

    对于7688的Openwrt有两个地方可以找到对应得开发环境，一个是

    https://labs.mediatek.com/site/znch/developer_tools/mediatek_linkit_smart_7688/sdt_intro/index.gsp

    这是联发科自己的网站。另一个就是Openwrt社区。

    https://downloads.openwrt.org/chaos_calmer/15.05.1/ramips/mt7688/

    这是目前两个源头。我们可以任选一个下载toolchain。

2.下载7688 Openwrt Toolchain

    由于我系统是OSX，所以我下载了
    OpenWrt-SDK-ramips-mt7688_gcc-4.8-linaro_uClibc-0.9.33.2.Darwin-x86_64.tar.bz2



### 配置环境 
不是所有node module都需要交叉编译，只有需要native代码实现的module才需要交叉编译

    export STAGING_DIR=/Users/user/WorkSpace/OpenWrt-SDK-ramips-mt7688_gcc-4.8-linaro_uClibc-0.9.33.2.Darwin-x86_64/staging_dir
    export CC=/Users/user/WorkSpace/OpenWrt-SDK-ramips-mt7688_gcc-4.8-linaro_uClibc-0.9.33.2.Darwin-x86_64/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/bin/mipsel-openwrt-linux-gcc
    export CXX=/Users/user/WorkSpace/OpenWrt-SDK-ramips-mt7688_gcc-4.8-linaro_uClibc-0.9.33.2.Darwin-x86_64/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/bin/mipsel-openwrt-linux-g++
<p />
### 遇上问题
在这里我们测试一下websocket这个module。    

    npm install --arch=mipsel websocket
    > websocket@1.0.23 install /Users/user/WorkSpace/websocket_test/node_modules/websocket
    > (node-gyp rebuild 2> builderror.log) || (exit 0)

      CXX(target) Release/obj.target/bufferutil/src/bufferutil.o
      websocket@1.0.23 node_modules/websocket
      ├── yaeti@0.0.4
      ├── nan@2.3.5
      ├── typedarray-to-buffer@3.1.2 (is-typedarray@1.0.0)
    └── debug@2.2.0 (ms@0.7.1)

查看一下builderror.log

        1 mipsel-openwrt-linux-g++: error: x86_64: No such file or directory
        2 mipsel-openwrt-linux-g++: error: unrecognized command line option '-mmacosx-version-min=10.5'
        3 mipsel-openwrt-linux-g++: error: unrecognized command line option '-arch'
        4 make: *** [Release/obj.target/bufferutil/src/bufferutil.o] Error 1
        5 gyp ERR! build error
        6 gyp ERR! stack Error: `make` failed with exit code: 2
        7 gyp ERR! stack     at ChildProcess.onExit (/Users/user/.nvm/versions/node/v4.0.0/lib/node_modules/npm/node_modules/node-gyp    /lib/build.js:270:23)
        8 gyp ERR! stack     at emitTwo (events.js:87:13)
        9 gyp ERR! stack     at ChildProcess.emit (events.js:172:7)
       10 gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:200:12)
       11 gyp ERR! System Darwin 14.5.0
       12 gyp ERR! command "/Users/user/.nvm/versions/node/v4.0.0/bin/node" "/Users/user/.nvm/versions/node/v4.0.0/lib/node_modules/n    pm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
       13 gyp ERR! cwd /Users/user/WorkSpace/websocket_test/node_modules/websocket
       14 gyp ERR! node -v v4.0.0
       15 gyp ERR! node-gyp -v v3.0.1
       16 gyp ERR! not ok

编译bufferutil.cc的时候就出错了，编译参数不识别。想了很多办法，都宣告失败后我转战到Ubuntu上，同样的过程builderror.log中没有内容。查看到bufferutil.node和validation.node都生成了。

### 结束语 

我认为OSX上做交叉编译有诸多兼容性问题，因为毕竟跟标准linux的结构不同，甚至头文件位置，名字也未必相同，为了顺利编译目标板的库或者module最佳选择还是拥有标准linux结构的系统如Ubuntu。








