---
published: true
layout: post
title: 树莓派上部署qq的mipush消息推送服务
author: johnny
category: articles
tags:
  - mipush
  - raspberry pi
---

因为以前的手机只有2GRAM，所以一直以来对于腾讯毒瘤可谓深恶痛绝，恰好github看到有人写了基于smartqq的框架-[Mojo-Webqq](https://github.com/sjdy521/Mojo-Webqq),正好酷安上发布了android客户端[GcmForMojo](https://www.coolapk.com/apk/com.swjtu.gcmformojo)，于是只需要在树莓派上部署服务端即可，实现的效果即无需登录qq的app，消息采用mipush推送。采用系统推送的好处显而易见，不但不占用ram同时也能更省电。 
<!-- more -->  

---------------------------------------  

#### 准备工作

- 树莓派3b一只  
当然由于是在树莓派上部署，因为我的树莓派没有公网ip，所以不能实现快捷回复，只能实现接收消息，点击通知跳转到APP回复。

---------------------------------------  

#### 架设环境

树莓派的系统为基于debian的官方系统Raspbian，首先安装依赖
```  
sudo apt install openssl libssl-dev g++ gcc make 
cpan -i App::cpanminus
```
接下来安装webqq框架
```  
cpanm --mirror http://mirrors.163.com/cpan/ Mojo::Webqq
```

---------------------------------------  

#### 启用服务  
首先我们打开GCMForMojo app获取到设备码
接着我们创建一个perl脚本
```  
nano GCM_qq.pl
```
然后把下面的粘贴进去，以及修改registration_ids为获取到的设备码
```  
use Mojo::Webqq;
my $client = Mojo::Webqq->new(log_encoding=>"utf-8");
$client->load("ShowMsg");
#MiPush 推送
$client->load("MiPush",data=>{
    registration_ids=>["输入你自己从 GCMForMojo APP中获取到的令牌"],
    allow_group=>["接受群消息的号码，如需要推送全部群消息可删除这一行，每个群号码之间使用 "", 分隔"],
    ban_group=>[],
    allow_discuss=>[],
    ban_discuss=>[],
});
$client->load("Openqq",data=>{
    listen => [{host=>"0.0.0.0",port=>5000}, ] ,
});
$client->run(); 
```
然后Ctrl+O Ctrl+X保存退出
由于我们要保证ssh连接树莓派断开后perl脚本依然在执行,因此需要开一个screen
```  
screen -S qq
```
然后执行perl脚本
```  
perl GCM_qq.pl
```
这时候你的 GCMForMojo APP应该会弹出一条检测到二维码事件的通知，点击它，使用手机端 QQ 扫描这个二维码，你的 GCMForMojo 就跑起来了

--------------------------------------- 

#### 额外  

为了安全起见,我们需要在服务端与客户端同时配置盐值
首先在GCMForMojo app中设置盐值,然后修改GCM_qq.pl为如下
```  
use Mojo::Webqq;
use Digest::MD5  qw(md5 md5_hex md5_base64);

my $client = Mojo::Webqq->new(log_encoding=>"utf-8");
$client->load("ShowMsg");
$client->load("MiPush",data=>{
    registration_ids=>["输入你自己从 GCMForMojo APP中获取到的令牌"],
    allow_group=>["接受群消息的号码，如需要推送全部群消息可删除这一行，每个群号码之间使用 "", 分隔"],
    ban_group=>[],
    allow_discuss=>[],
    ban_discuss=>[],
});

$client->load("Openqq",data=>{
    listen => [{host=>"0.0.0.0",port=>5000}, ] ,
    auth => sub {
        my($param,$controller) = @_;
        my $secret = 'salt';
        #请将该行salt改为你自定义的盐值，并在 Android 端内设定好你所自定义的盐值
        my $text='';
        foreach $key (sort keys %$param){
            if($key ne 'sign'){
                $value =$param->{$key};
                $text.=$value;
            }
        }
        if($param->{sign} eq md5_hex($text.$secret) ){
            return 1;
        }
        else{
            return 0;
        }
    }
});
$client->run();
```  

--------------------------------------- 

#### 此外  

同样该框架也支持微信,然而由于微信本身的机制导致开启此服务后会与电脑版或者pad版的微信冲突,所以最终放弃.  

