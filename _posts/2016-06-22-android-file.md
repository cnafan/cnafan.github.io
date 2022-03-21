---
published: true
layout: post
title: android文件读取和写入
author: johnny 
category: articles
tags:
- android
---
利用FileOutputStream将文件写入到/data/data/目录下
<!-- more --> 
```
try {
    FileOutputStream fout = context.openFileOutput(fileName, Context.MODE_APPEND);
    byte[] bytes = writestr.getBytes();
    fout.write(bytes);
    fout.close();
    } catch (Exception e) {
    	e.printStackTrace();
    }
```
利用FileInputStream从/data/data/目录下读取文件
```
String res = "";
try {
    FileInputStream fin = context.openFileInput(fileName);
    int length = fin.available();
    byte[] buffer = new byte[length];
    fin.read(buffer);
    res = EncodingUtils.getString(buffer, "UTF-8");
    fin.close();
} catch (Exception e) {
    e.printStackTrace();
}
return res;
```


