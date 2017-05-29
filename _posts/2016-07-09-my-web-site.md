---
published: true
layout: post
title: 网站的完整搭建流程
author: johnny 
category: articles
tags:
- web
- 腾讯云
---
整个网站的开始可以从3月17号算起，因为这一天就是购买腾讯云主机的日子。从购买主机加上购买域名还有装系统这些步骤就不赘述了，主要记录记录网站的环境配置。
<!-- more -->  
整个配置大体上分的话，可以分成3个方面:
- 第一步即各模块的安装,自己的服务器搭建选用的是apache+mod_wsgi+python(flask)；其中apache服务器当时选用的版本是Win64-2.4.20 VC10,mod_wsgi的版本是VC10，在加上VC10编译的python34.这里需要强调的这三个模块必须是统一编译版本，要么都是VC10,要么都是VC11，不要问我为什么知道,都是泪0.0。ok,版本选用完后，开始正式安装。
- 第二步，先说apache的配置，进入Apache24的安装目录下的conf目录，打开httpd.conf文件，找到```LoadModule wsgi_module modules/mod_wsgi.so``` ```LoadModule include_module modules/mod_include.so```这两行，去掉前面的‘#’，接下来进入Apache24/conf/extra目录,打开httpd-vhosts.conf文件，添加
```
&lt;VirtualHost *:80 >
    ServerAdmin example@company.com
    DocumentRoot c:\网站的本地文件夹
    ServerName ‘自己服务器的ip’
    WSGIScriptAlias / c:\网站的本地文件夹\myweb.wsgi
   &lt;Directory "c:\网站的本地文件夹">
        Options -Indexes
        AllowOverride All
        Require all granted
    &lt;/Directory>
&lt;/VirtualHost>
```
- ok，继续，第三步我们在本地创建一个文件夹，在里面新建一个*.wsgi文件
```
import sys
sys.path.insert(0, "c:/网站的本地文件夹")
from test import app

#Put logging code (and imports) here ...

#Initialize WSGI app object
application = app
```
然后我们打开window服务，启动apache24服务。ok，接下来我们cmd，用pip安装flask，我装的是Flask-0.10.1,装完之后，就可以正式开始了，在刚才自己建的网站文件夹中创建一个myweb.py文件
```
from flask import Flask, request,render_template,redirect, url_for, abort, session
from flask.ext.admin import Admin, BaseView, expose
app = Flask(__name__)

@app.route("/")
def welcome():
    return render_template('welcome.html')
```
整个流程大概就是这样，因为是第一次碰web方面的知识，所有的问题都只能借助于搜索引擎和一些博客,于是自己就想把自己的经历记录下来，说不定还能帮助到曾经像我一样的少年.
