---
layout:     post
title:      "Android 实用小技巧"
subtitle:   "Android 常用小技巧"
date:       2016-08-02 09:12:00
author:     "SQSong"
header-img: "img/post-06-bg.jpg"
header-mask: 0.3
catalog:    true
tags:
    - android
    - 使用技巧
---

## Android实用小技巧

#### 手动重启应用

```java
Intent mStartActivity = new Intent(StartPageActivity.this, StartPageActivity.class);
int mPendingIntentId = 123456;
//定义一个PendingIntent，并设置要启动的Activity
PendingIntent mPendingIntent = PendingIntent.getActivity(StartPageActivity.this, mPendingIntentId,
mStartActivity, PendingIntent.FLAG_CANCEL_CURRENT);
//设置闹钟
AlarmManager mgr = (AlarmManager)getSystemService(Context.ALARM_SERVICE);
mgr.set(AlarmManager.RTC, System.currentTimeMillis() + 100, mPendingIntent);
System.exit(0);
```

#### 重命名文件
```java
String folderName = StartPageActivity.getStorageDirectory(this, 2);
File folder = new File(folderName);
if (folder.exists() && folder.isDirectory()) {
  File origin = new File(folder, patchFileName);
  File newFileName = new File(folder, Constant.USED_PATCH_PREFIX + patchFileName);
  origin.renameTo(newFileName);
}
```
