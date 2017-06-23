---
published: true
layout: post
title: raspberry pi car
author: johnny
category: articles
tags:
  - GPIO
  - raspberry pi
---

从小就幻想着能拥有一辆自己的车（最好是特斯拉-。-），虽然目前只向着梦想前进了一小步，也是激动万分。接下来就是见证这梦想之路的一小步。  
<!-- more -->

---------------------------------------  


#### 准备工作  

- 小车底座
- 5v树莓派供电
- 12v电机驱动电源 (我用的是双输出口的充电宝 一个给pi 一个驱动)
- 四个电机
- l298n驱动一个
- 树莓派3b一只
- 三种杜邦线各20根
- 红外、超声波、小灯等模块(可选)
- 树莓派摄像头一枚

---------------------------------------  

#### 线路连接

由于一个l298n驱动只能控制两个电机，但是想做成四轮小车于是采用两侧的电机并联，即两侧的电机同步转动。树莓派和驱动的接线按照下图(这个图是3b的，其他型号pi的gpio针脚会略有不同)需要注意的一点是图上那个EN_A和EN_B那个是需要拔掉那个短接帽，因为硬件小白以前从来没接触过，结果半天没看懂怎么插  
![](/images/raspberry_car_gpio.png)

---------------------------------------  

#### 程序测试  

![](/images/raspberry_car_l298n.png)  
从真值表可以看出给IN1 IN2和EN通不同的电平即可产生不同的效果。接下来我们在树莓派中编写程序测试一下。
```  
import RPi.GPIO as GPIO

def forward():
	GPIO.output(2,True)
	GPIO.output(3,True)
	GPIO.output(4,False)
def backward():
	GPIO.output(2,True)
	GPIO.output(3,False)
	GPIO.output(4,True)
def stop():
	GPIO.output(2,True)
	GPIO.output(3,True)
	GPIO.output(4,True)

if __name__ == '__main__':
	GPIO.setmode(GPIO.BCM) #BCM编号即上图中针脚旁的数字
	GPIO.setup(2,GPIO.OUT)
	GPIO.setup(3,GPIO.OUT)
	GPIO.setup(4,GPIO.OUT)
	forward()  
```  
--------------------------------------- 

#### 超声波模块试用  

我买的超声波模块型号是HY-SRF05，接线方案为：ucc接5v，trig接23，echo接24，gnd接23紧挨着的gnd。还有需要注意的一点是，超声波模块似乎需要方向，即当倒置模块时测距误差会变得特别大。
```  
import RPi.GPIO as GPIO
import time
#trig 23
#echo 24
Trig_Pin=23
Echo_Pin=24
def init_hy():
	GPIO.setup(Trig_Pin, GPIO.OUT, initial = GPIO.LOW)
	GPIO.setup(Echo_Pin, GPIO.IN)

def checkdist():
	GPIO.output(Trig_Pin, GPIO.HIGH)
    time.sleep(0.00015)
    GPIO.output(Trig_Pin, GPIO.LOW)
    while not GPIO.input(Echo_Pin):
        pass
    t1 = time.time()
    while GPIO.input(Echo_Pin):
        pass
    t2 = time.time()
    return (t2-t1)*340*100/2

if __name__ == '__main__':
	try:
   		while True:
        print('Distance:'+checkdist()+'cm')
        time.sleep(1)
	except KeyboardInterrupt:
    	GPIO.cleanup()
```  
---------------------------------------  

#### 摄像头使用  

ssh连接到pi(使用非root用户登录)，输入指令``` sudo raspivid -o - -t 0 -w 640 -h 360 -fps 25|cvlc -vvv stream:///dev/stdin --sout '#standard {access=http,mux=ts,dst=:8080}' :demux=h264  ```接着用vlc的串流播放地址：```http://你的树莓派ip:8080```，这个方案相比之前用mjpg-streamer而言，会有3秒的延迟。

---------------------------------------  

#### 呼吸灯  

小灯这个只需要两个GPIO口，一个接3.3v，另一个我接了18
```  

import RPi.GPIO
import time

RPi.GPIO.setmode(RPi.GPIO.BCM)
RPi.GPIO.setup(18, RPi.GPIO.OUT)

pwm = RPi.GPIO.PWM(18, 50)
pwm.start(0)
try:
    while True:
        for i in range(0, 101, 1):
            pwm.ChangeDutyCycle(i)
            time.sleep(.01)
        for i in range(100, -1, -1):
            pwm.ChangeDutyCycle(i)
            time.sleep(.01)
except KeyboardInterrupt:
    pass
pwm.stop()
RPi.GPIO.cleanup()
```
---------------------------------------  

#### 大功告成  

剩下的就是完善整个小车，包括整合超声波避障、转向灯、摄像头等模块以及可视化操控。最后附上完工照。  
![](/images/raspberry_car_fin.jpg)
