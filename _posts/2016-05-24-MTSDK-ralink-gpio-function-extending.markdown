---
title: MTSDK上ralink gpio的中断信号的局限及扩展 
layout: post
guid: urn:uuid:de8d598d-6f35-4c7b-ab53-19510645adfc
tags:
  - MTK 
  - SDK 
  - ralink 
  - gpio 
  - signal
  - handler
  - click
  - hold
---
![Alone](/media/files/2016/3/alone.jpg)
<p />

###  1. MT SDK的gpio操作方法 

<p>
MTK的用户态的GPIO操作是通过/dev/gpio设备来进行的。作为外围设备主要是button和LED灯两种。所以本文主要就是围绕着button来展开。
</p>

1.初始化gpio

        fd = open(GPIO_DEV, O_RDONLY);
        info.pid = getpid();
        info.irq = gpio_num;
        ioctl(fd, RALINK_GPIO_SET_DIR_IN, info.irq);//注意在Ralink平台需要判断gpio号
        //enable 中断
        ioctl(fd, RALINK_GPIO_ENABLE_INTP);

2.注册gpio信息

        //注册gpio信息到kernel，包括号和pid
        ioctl(fd, RALINK_GPIO_REG_IRQ, &info);
        close(fd);

        //kernel会得到中断，然后判断长按（hold）还是短按（click）。
        //调用
        ralink_gpio_notify_user(usr);
        //如果user等于1，发送SIGUSR1信号给pid对应process，如果user等于2，发送SIGUSR2信号给pid对应process。

3.处理kernel层signal

        signal(SIGUSR1, doWPSHandler);
        signal(SIGUSR2, doResetHandler);


###  存在的问题 

1.根据上述实现，我们发现一共就两种signal可以利用，在更多应用场景下不够用。

2.只能区分长按短按，无法区分是哪个按键按下的，同一个process只能处理一个button。


###  解决方法 

1.使用不同的process处理不同的按键中断。（太麻烦）

2.扩展gpio info结构，增加一个变量用来告诉kernel发送不同类型signal。如下:


        typedef struct {
            unsigned int irq;   //request irq pin number
            pid_t pid;          //process id to notify
            int can_hold;       // 0 both; 1 only click; 2 only hold; 3 SIGTMIN signal for wilan
        }ralink_gpio_reg_info;
    
        //初始化gpio pin的时候设定好can_hold标记。由kernel来判断发送何种signal。
        can_hold = ralink_gpio_info[ralink_gpio_irqnum].can_hold & 0x3;
        if (usr == 1 && can_hold == 3) {
            printk(KERN_NOTICE NAME ": sending a SIGRTMIN to process %d\n",
                    ralink_gpio_info[ralink_gpio_irqnum].pid);
            send_sig(SIGRTMIN+1, p, 0);
        }
        else if (usr == 1 && can_hold != 2) {
            printk(KERN_NOTICE NAME ": sending a SIGUSR1 to process %d\n",
                    ralink_gpio_info[ralink_gpio_irqnum].pid);
            send_sig(SIGUSR1, p, 0);
        }
        else if (usr == 1 && can_hold != 1) {
            printk(KERN_NOTICE NAME ": sending a SIGUSR2 to process %d\n",
                    ralink_gpio_info[ralink_gpio_irqnum].pid);
            send_sig(SIGUSR2, p, 0);
        }


### 优化

1.上述方法只提供了三种信号，可能依然不够使用，但是却提供了一个途径解决signal不够用的问题。

2.为了灵活使用signal，可以给每个gpio button注册一个signal，或者组合，这样can\_hold可以不只是4种状态，而是直接由signal得值来替代。

3.这样可以把尽可能多的gpio 中断处理放置在一个process中。

4.也许MTK有更加完善的方法扩展GPIO功能，本方法也许不是种很好的方法，但是目前看还是挺方便的。

