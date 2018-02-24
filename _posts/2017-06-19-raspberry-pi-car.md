---
published: true
layout: post
title: raspberry pi car
author: johnny
category: articles
tags:
  - GPIO
  - raspberry pi
  - Android
---

<!-- 从小就幻想着能拥有一辆自己的车（最好是特斯拉-。-），虽然目前只向着梦想前进了一小步，也是激动万分。  --> Android 代码已上传至github:[链接](https://github.com/sikuquanshu123/rocker)  
<!-- more -->  

---------------------------------------  

### 准备工作  

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
### 线路连接

由于一个l298n驱动只能控制两个电机，但是想做成四轮小车于是采用两侧的电机并联，即两侧的电机同步转动。树莓派和驱动的接线按照下图(这个图是3b的，其他型号pi的gpio针脚会略有不同)需要注意的一点是图上那个EN_A和EN_B那个是需要拔掉那个短接帽. 
![](/images/raspberry_car_gpio.png)

---------------------------------------  

### 程序测试  

![](/images/raspberry_car_l298n.png){:height="50%" width="50%"}  
从真值表可以看出给IN1 IN2和EN通不同的电平即可产生不同的效果。
```  
import RPi.GPIO as GPIO 
#采用BCM编码
left_en = 14 
left_in1 = 15
left_in2 = 18
right_en = 25
right_in3 = 8
right_in4 = 7

def up():
    GPIO.output(right_en, True)
    GPIO.output(right_in1, True)
    GPIO.output(right_in2, False)
    GPIO.output(left_en, True)
    GPIO.output(left_in1, True)
    GPIO.output(left_in2, False)
```  

--------------------------------------- 

### 红外避障模块试用  
这款模块只有3个接口 VCC可以接3.3或者5v GND接地 OUT接GPIO，由于需要在左右两侧都考虑避障，所以得最少买两个模块。红外避障模块使用起来很简单，直接读取OUT接口的数据判断即可。
```  
red_left=16
red_right=12
GPIO.setup(red_left,GPIO.IN)
GPIO.setup(red_right,GPIO.IN)
if GPIO.input(red_left) && GPIO.input(red_right):
	t_up()
```
![](/images/raspberry_car_red.png){:height="30%" width="30%"}

--------------------------------------- 

### 超声波模块试用  

我买的超声波模块型号是HY-SRF05，接线方案为：ucc接5v，trig接23，echo接24，gnd接23紧挨着的gnd。还有需要注意的一点是，超声波模块似乎需要方向，即当倒置模块时测距误差会变得特别大。
```  
import RPi.GPIO as GPIO
import time
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
![](/images/raspberry_car_hy.png){:height="30%" width="30%"}

---------------------------------------  

### 摄像头使用  

ssh连接到pi(使用非root用户登录)，输入指令``` sudo raspivid -o - -t 0 -w 640 -h 360 -fps 25|cvlc -vvv stream:///dev/stdin --sout '#standard {access=http,mux=ts,dst=:8080}' :demux=h264  ```接着用vlc的串流播放地址：```http://你的树莓派ip:8080```，这个方案相比之前用mjpg-streamer而言，会有3秒的延迟。  

---------------------------------------  

### 呼吸灯  

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

### 操控  

操控采用的是树莓派作为一个socket服务器，然后通过android客户端向树莓派发送指令，树莓派接受到指令后执行任务。  android客户端中轨迹球的部分是fork一个在github上看到的仓库，其中视频流采用的是之前用的mjpg-stream。  
- socket服务器端  

```  
import socketserver
import gpio
class ThreadedTCPRequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        while True:
            data = (self.request.recv(1024).decode('utf-8'))
            if not data:
                break
            data=data.replace('\n','').replace(' ','')
            response = gpio.ctrl_id(data)+"\n"
            self.request.sendall(response.encode('utf-8'))
if __name__ == "__main__":
    # Port 0 means to select an arbitrary unused port
    HOST, PORT = "0.0.0.0", 20000
    server = socketserver.TCPServer((HOST, PORT), ThreadedTCPRequestHandler)
    server.serve_forever()  
```  
代码里面有换行符的修改是因为android在测试的时候发现java的socket发送会自动以换行符结尾，同样读取也是以换行符为终点。此外发现每次java的发送过程中会首先发一个空的数据包，暂时还不清楚这个是为啥。  
- android socket client  

```  
// 获取 Client 端的输出/输入流
PrintWriter out = null;
try {
    out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(socket.getOutputStream(), "UTF-8")), true);
} catch (IOException e) {
    e.printStackTrace();
}
// 填充信息
assert out != null;
out.println(info);
BufferedReader br;
try {
    br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
    msg = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}  
```  
![](/images/raspberry_car_android.png){:height="50%" width="50%"}

---------------------------------------  

### 大功告成  

剩下的就是完善整个小车，包括整合超声波避障、转向灯、摄像头等模块以及可视化操控。最后附上完工照。  
![](/images/raspberry_car_fin.jpg){:height="50%" width="50%"}
