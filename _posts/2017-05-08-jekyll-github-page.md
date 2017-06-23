---
published: true
layout: post
title: github page+jekyll
author: johnny
category: articles
tags:
  - github page
  - jekyll
---  

之前于csdn上记录过在腾讯云上通过flask建站的流程，随着临近毕业，腾讯云的学生计划也到期了，于是就看上了github page这块风水地，接下来一步步记录下在github上搭建自己博客的流程。  
<!-- more -->

---------------------------------------  

#### 准备工作
- 拥有一个github账户
- 在本地安装git
- 在本地安装jekyll
- 在万网买个域名（可选）  

---------------------------------------  

#### 第一步
- 首先我们打开命令行，找到一个建立项目的地方并cd到该处  
``` cd desktop ```  
- 接下来创建一个本地的jekyll项目  
``` jekyll new blog ```  
本地会生成一个叫blog的文件夹1，打开可以看到一下文件  
![](/images/github_page_1.png)  

---------------------------------------  

#### 第二步
- 打开github，新建一个仓库，起名为 "你的名字.github.io"
![](/images/github_page_2.png)
- 创建完之后，打开这个仓库，然后复制git地址
![](/images/github_page_3.png)
- 接下来在我们第一步建立项目的地方，打开bash
``` git clone "刚才复制的git地址" ```
- 在本地打开 "你的名字.github.io"的文件夹2，然后将第一步blog中的东西复制到里面。    
- 然后进入到文件夹2中，打开bash同步到github  
```
git init
git add .
git commit -m "first commit"
git push origin master
```  
- 同步成功以后可以直接访问 "https://你的名字.github.io/"

---------------------------------------  

#### 绑定域名（可选）
- 进入仓库的设置页面，在"Custom domain"这一项填入在万网买的域名
![](/images/github_page_4.png)
- 进入万网的域名管理页面，添加CNAME域名解析
![](/images/github_page_5.png)
等待10分钟后，即可使用域名访问github个人网站。









