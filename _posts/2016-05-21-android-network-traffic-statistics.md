---
published: true
layout: post
title: Android network traffic statistics
author: Johnny 
category: articles
tags:
- Android Development
- 正则表达式
---
由于手机是android4.4，翻遍酷市场也没有一个满足需求的成品，于是小手一动、脑袋一热决定自己丰衣足食。  
代码已经上传到github:[链接](https://github.com/sikuquanshu123/FlowStatistics)  

<!-- more --> 
主要的需求有两点：
- 实现本月剩余流量的查询
- 实现每日消耗流量的统计  
ok，要实现第一点的思路是给10086发短信，然后利用正则表达式提取回执短信中所需要的信息。这里发短信是调用系统api--smsmanage，然后设一个广播接收器，在接受器中实现正则的读取，然后通过接口的调用，实现数据的更新。  
```
private DecimalFormat df = new DecimalFormat("#.##");
    //尊敬的客户，您好！截至09月23日16时，您本帐单月使用的"赠送1.5G省内通用流量，48个月"套餐移动数据流量共有1536.00M，剩余流量还有1358.83M; "0元50M省内流量包"套餐移动数据流量共有50.00M，剩余流量还有50.00M; "赠送10M省内流量"套餐移动数据流量共有10.00M，剩余流量还有10.00M; "18元包来电显示+100M国内流量，超出0.29元/M"套餐移动数据流量共有100.00M，剩余流量还有100.00M; 普通GPRS"10元包1G省内通用流量"套餐移动数据流量共有1024.00M，剩余流量还有1024.00M; "0元包6GB咪咕省内定向流量，有效期12个月"套餐移动数据流量共有6144.00M，剩余流量还有6144.00M; 仅供参考，具体以月结账单为准。
    private String[] reg = {
            "(?<=剩余流量还有)\\d+\\.\\d\\d",
            "(?<=流量共有)\\d+\\.\\d\\d"
    };

    private String deal(Context context, String data, String reg) {
        SharedPreferences pref = getDefaultSharedPreferences(context);
        Double check = Double.valueOf(pref.getString("check", "0"));
        Double return_data = 0.0;
        Pattern reg_data = Pattern.compile(reg);
        Matcher matcher = reg_data.matcher(data);
        while (matcher.find()) {
            return_data += Double.valueOf(matcher.group());
        }
        return_data -= check;
        DecimalFormat df = new DecimalFormat("#.##");

        if (return_data > 1024.0 && return_data / 1024.0 > 0) {
            return df.format(return_data / 1024.0) + "G";
        }
        else if(return_data < 1024.0 && return_data  > 1){
            return df.format(return_data) + "M";
        }
        else {
            return df.format(return_data*1024.0) + "k";
        }
    }
    String[] calculate(Context context, String data) {
        String[] remain = new String[2];
        for (int t = 0; t < remain.length; t++) {
            remain[t] = deal(context, data, reg[t]);
        }
        return remain;
    }
```
第二点是用到了android的traficstats类中的getmobilerxbytes()读取手机流量信息，对了，这个类是统计此设备自从开机以来产生的数据，所以在每次关机之前都需要将数据保存下来。在这里遇到了一些问题，这个类在android 5以上的系统中，如果数据开关处于关闭状态时，读取的数据为0，当开关打开时，读取数据恢复正常。所以设置一个网络连接改变的广播，只在数据处于活动时启动服务。  
此外，还需要实现数据的定时刷新，每隔3分钟和每隔24小时，每隔3分钟是读取数据库数据，更新每天使用的流量，然后发送新的通知。每隔24小时是发送短信查询流量信息，然后发送新的通知。过程是先设置一个alarmmanager，时间到了就唤醒更新数据的服务，更新完了之后再设置下一个alarm，最后自杀掉服务。所以该应用的后台是不会有任何服务长期滞留在内存中。
```
//0点定时器
    Intent intent1 = new Intent(this, AlarmReceiverManual.class); 
    PendingIntent pendingIntent2 = PendingIntent.getBroadcast(this, 1, intent1, PendingIntent.FLAG_UPDATE_CURRENT);
    long firstTime = SystemClock.elapsedRealtime();    
    // 开机之后到现在的运行时间(包括睡眠时间)
    long systemTime = System.currentTimeMillis();
    Calendar calendar = Calendar.getInstance();
    calendar.setTimeInMillis(System.currentTimeMillis());
    // 这里时区需要设置一下，不然会有8个小时的时间差
    calendar.setTimeZone(TimeZone.getTimeZone("GMT+8"));
    calendar.set(Calendar.MINUTE, 0);
    calendar.set(Calendar.HOUR_OF_DAY, 0);
    calendar.set(Calendar.SECOND, 0);
    calendar.set(Calendar.MILLISECOND, 0);
    // 选择的定时时间
    long selectTime = calendar.getTimeInMillis();
    // 如果当前时间大于设置的时间，那么就从第二天的设定时间开始
    if (systemTime > selectTime) {
        calendar.add(Calendar.DAY_OF_MONTH, 1);
        selectTime = calendar.getTimeInMillis();
    }
    // 计算现在时间到设定时间的时间差
    long time = selectTime - systemTime;
    firstTime += time;
    // 进行闹铃注册
    AlarmManager manager = (AlarmManager) getSystemService(ALARM_SERVICE);
    manager.setRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP,
            firstTime, time, pendingIntent2);
    stopSelf();
```

