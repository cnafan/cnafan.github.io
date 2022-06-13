---
published: true
layout: post
title: NextCloud自建网盘备份照片
author: johnny
category: articles
tags:
  - NextCloud
  - raspberry pi
---

失踪人口回归，时隔5年又来折腾吃灰的树莓派了。这次的需求是摆脱手机厂商的云服务，实现跨平台的照片备份。
<!-- more -->  

---------------------------------------

## 准备工作

- 树莓派3b一只  
考虑到能快速搭建且尽可能少踩坑，所以使用现成的Docker进行部署。

---------------------------------------

## 架设环境

首先我们在官方的Raspberry OS上安装Docker。  

### 安装docker图形化界面
```shell
sudo docker pull cr.portainer.io/portainer/portainer-ce

sudo docker volume create portainer_data

sudo docker run -d -p 9000 : 9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data cr.portainer.io/portainer/portainer-ce
```

### 通过Docker Compose生成
编辑 `docker-compose.yml`
```
version: '2'

services:
db:
 image: ibex/debian-mysql-server-5.7  # mysql 镜像文件
 restart: always
 volumes:

   - /mnt/sda1/cloud/db:/var/lib/mysql
     vironment:
        - MYSQL_ROOT_PASSWORD=******  # root 账号密码，根据需要更改 
          MYSQL_PASSWORD=****** # 普通账号密码，根据需要更改 
             - MYSQL_DATABASE=nextcloud  # 数据库名
               MYSQL_USER=nextcloud  # 普通用户名
                 app:
                   image: nextcloud  # nextcloud 镜像文件
                   privileged: true  # 拥 有root 权限
                   ports:
                  - 8888:80  # 端口映射，将 Docker 的80端口，映射成主机的 8888 端口，根据需要更改
                    nks:
                       - db  # db 是别名，使用该别名访问 前面定义的 db。nextcloud 启动时数据库的链接填 db。
                         lumes:
                            - /mnt/sda1/cloud/config:/var/www/html/config
                              /mnt/sda1/cloud/data:/var/www/html/data 
                                 - /mnt/sda1/cloud/apps:/var/www/html/apps
                                   start: always

  redis:
    image: redis  
    container_name: redis-d  # 别名
    privileged: true  # 拥有 root 权限
    restart: always
    command: --appendonly yes  --requirepass "******"  # appendonly 持久化的模式, requirepass 设置密码 password
    ports:

   - 16379:6379  # redis 6379 映射到主机的 16379 端口
     - /mnt/sda1/cloud/redis/data:/data  # 数据目录映射点
       da1/cloud/redis/config/redis.conf:/usr/local/etc/redis/redis.conf  # 配置文件映射点
```
执行 `docker-compose up -d`



---------------------------------------

## 启用服务  

登录NextCloud的后台  

![微信截图_20220609200702](https://md-images-1251991865.cos.ap-chengdu.myqcloud.com/img/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220609200702.png)

---------------------------------------

## 额外  

为了提高网页访问速度，我们配置redis对资源进行缓存  
修改`config.php`
```
  'memcache.local' => '\OC\Memcache\Redis',
  'memcache.distributed' => '\OC\Memcache\Redis',
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' => array(
    'host' => 'redis',
    'port' => 6379,
    'password' => '******'
  ),
```

同时我们可以为图片、视频等开启缩略图  

首先记得安装ffmpeg `apt install ffmpeg -y`

修改`config.php`

```
  'enable_previews' => true,
  'enabledPreviewProviders' =>
  array (
    0 => 'OC\\Preview\\Image',
    1 => 'OC\\Preview\\Movie',
    2 => 'OC\\Preview\\TXT',
  ),
```

---------------------------------------
## 总结

总的体验下来，NextCloud的备份可以满足需求，上传速度最高能达到10M，基本上可以跑满3B的百兆小水管。不过看到Nextcloud的同步逻辑是以云端为主，删除云端本地会同步删除，本地删除后会自动同步到本地，这一点还未测试。等后续再体验几天，再来更新一波。

<img src="https://md-images-1251991865.cos.ap-chengdu.myqcloud.com/img/image-20220609201103981.png" style="zoom: 67%;" />
