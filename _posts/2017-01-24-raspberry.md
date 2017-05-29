---
published: true
layout: post
title: raspberry pi上手
author: johnny 
category: articles
tags:
- raspberry
---
树莓派是一台电脑，尺寸只有银行卡大小，树莓派优秀的可扩展性给了人们极大的发挥空间，你可以在树莓派上部署任何你能想到的项目、各种geek天马行空的玩法。  
<!-- more -->
## pi的基本介绍
一个树莓派启动所必须的配件包括：一个8G的TF卡用于装系统、一个5V2.5A的电源、再加一个亚克力外壳，最好再配个散热片，亲测加上散热片日常开机运行不会超过55°。  
![](/images/raspberry_2.jpg)
树莓派3B采用了BCM2837 1.2GHz四核A53 64位处理器，1GB RAM和VideoCore IV GPU，内置Wi-Fi和蓝牙。  
![](/images/raspberry_3.jpg)
## 初试树莓派
树莓派拥有强大的社区资源，因此可供选择的系统不少，包括官方raspbian、arch、ridora、以及win10 IOT、甚至包括android4.4原生。我选的是官方推荐的[raspbian镜像](https://www.raspberrypi.org/downloads/)，如果选raspbian，官方提供了两个版本 WITH PIXEL和LITE，前者是带图形界面的。
![](/images/raspberry_4.jpg)
将镜像下载下来之后，使用官方推荐的[Win32DiskImager](https://sourceforge.net/projects/win32diskimager/)刻到sd卡。然后插上电源等待开机。接下来，找到一根网线，连接到路由器，从路由器的管理界面获取到pi的ip。  
接着打开[PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) ，输入ip，端口填22，选ssh，确认进入。（对了，新版jessie是默认关闭ssh的，所以如果连接不上pi，那么拔掉电源，给pi的sd卡上新建一个名为SSH没有后缀的文件）。  
ok,连接上以后树莓派的默认用户名是pi，密码是raspberry。登录上以后，首先我们得先更改软件源，官方的源慢的跟蜗牛一样。
```
sudo nano /etc/apt/sources.list

deb http://mirrors.aliyun.com/raspbian/raspbian/ jessie main non-free contrib rpi
deb-src http://mirrors.aliyun.com/raspbian/raspbian/ jessie main non-free contrib rpi

//ctrl+o保存 直接按enter不更改文件名 ctrl+x退出

sudo apt-get update //更新软件索引清单
```
接下来我们要做的工作是启用wifi，这样就可以愉快的拔掉网线玩啦。
```
sudo iwlist wlan0 scan      //扫描wifi

sudo nano /etc/wpa_supplicant/wpa_supplicant.conf //修改wifi配置 

network={  
    ssid="XXXX"  
    psk="XXXX" 
    scan_ssid=1 如果连接的wifi是隐藏的，就得写上这个

} 
//添加wifi的用户名和密码
```
ok,接下来我们重启树莓派就可以拔掉网线了```sudo reboot```  
连接好wifi之后呢，我们只要装上xrdp可以更好地调戏，啊呸，调试树莓派。  
```
sudo apt-get install xrdp
sudo apt-get install vnc4server tightvncserver
```
装好以后，打开windows自带的远程桌面连接，输入pi的ip，输入用户名和密码，点击确认。接下来就可以看到pi的庐山真面目了。
![](/images/raspberry_5.jpg)
## 上手树莓派
趁热打铁，在pi上部署一个小应用---起床闹钟。需求：一个小音响  
思路是用python的requests库爬取墨迹天气当天的天气状况，然后利用百度大脑的语音合成生成音频，然后在树莓派上采用定时任务，播放音频。同时也可以顺手来几首music  
```
# encoding=utf-8
import requests
import os
from bs4 import BeautifulSoup
import re
import datetime
import time


def gethtml(url):
    s = requests.get(url)
    return s.text


def retest(html):
    aqilevelpatten = 'src="https://h5tq.moji.com/tianqi/assets/images/aqi/(\d)'
    aqilevel = re.findall(aqilevelpatten, html)[0]
    print(aqilevel)

    aqipatten = '/aqi/' + aqilevel + '\.png"\s*alt="(\d* \w*)'
    aqi = re.findall(aqipatten, html)[0]
    print(aqi)

    aboutpatten = 'wea_about clearfix">\s*&lt;span>(\w* \w*%)&lt;/span>\s*&lt;em>(\w*)'
    about = re.findall(aboutpatten, html)  # [('湿度 8%', '北风3级')]
    humidity = about[0][0]
    windy = about[0][1]
    print(humidity)
    print(windy)

    tippatten = ' &lt;div class="wea_tips clearfix">\s*&lt;span>\w*&lt;/span>\s*&lt;em>(\w*，?\w*)'
    tip = re.findall(tippatten, html)[0]
    print(tip)

    list_weather = [aqi, humidity, windy, tip]
    today = datetime.date.today()
    print(today)
    with open("weather.txt", 'a', encoding='utf-8')as f:
        f.write(str(today) + ':')
        f.write(str(list_weather)+"\n")

    return list_weather

def tts(weather):
    #+" 湿度"weather[1]
    text="强哥 早上好 该起床啦"+"今天空气质量指数 是"+str(weather[0])+" 风力 为"+str(weather[2])+" 肚子提醒你，"+str(weather[3])
    headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit'
                          '/537.36 (KHTML, like Gecko) Chrome/53.0.2785.143 Safar'
                          'i/537.36',
        }
    url_2='http://tts.baidu.com/text2audio?idx=1&amp;tex=(%s)&amp;cuid=baidu_speech_demo&amp;cod=2&amp;lan=zh&amp;ctp=1&amp;pdt=1&amp;spd=5&amp;per=4&amp;vol=5&amp;pit=5'%text
    url = 'http://tts.baidu.com/text2audio?idx=1&amp;tex=%E5%BC%BA%E5%93%A5%20%E6%97%A9%E4%B8%8A%E5%A5%BD%20%E8%AF%A5%E8%B5%B7%E5%BA%8A%E5%95%A6&amp;cuid=baidu_speech&amp;cod=2&amp;lan=zh&amp;ctp=1&amp;pdt=1&amp;spd=5&amp;per=4&amp;vol=4&amp;pit=5'
    res = requests.get(url_2, headers=headers)
    with open('1.mp3', 'wb') as f:
        #print(res.content)
        f.write(res.content)
if __name__ == '__main__':
    #os.chdir('d:\\')
    url = 'http://tianqi.moji.com/'
    html = gethtml(url)
    weather = retest(html)
    tts(weather)
```
接下来我们需要在树莓派里设置定时任务。  
```
crontab -e

00 06 * * * python3 /root/weather/weather.py
01 06 * * * mplayer /root/weather/morning.mp3 /root/weather/1.mp3
03 06 * * * mplayer -shuffle /root/Misic/*.mp3
30 06 * * * pkill mplayer
```
哦，差点忘了装播放器```sudo apt-get install mplayer```  
接下来，就可以坐下来静静的等着黎明的到来，伴着清晨的微光，满怀期望的等待树莓派喊你起床. 0_0