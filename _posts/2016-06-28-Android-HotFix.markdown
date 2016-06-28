---
layout:     post
title:      "Android apk热补丁修复"
subtitle:   "AndFix热补丁修复线上apk bug"
date:       2016-06-27 16:38:00
author:     "SQSong"
header-img: "img/post-01-bg.jpg"
header-mask: 0.4
catalog:    true
tags:
    - android
    - hotfix
---

## Android 热补丁修复介绍
Android应用的每次版本更新是比较麻烦的，每次发布新的版本都要上新包到各大应用市场，流程都比较繁琐。当我们的应用遇到一些紧急的bug时，发布一次新版本的代价是比较大的而且用户也不能够及时获得更新，所以一些比较大的公司提出了热补丁动态修复技术来解决以上问题。目前应用的比较多的就是阿里巴巴的开源项目[AndoFix](https://github.com/alibaba/AndFix)。

## AndFix简介及使用
AndFix以在线修复bug的方式来替代重新发布新版本app，它以[库](https://sites.google.com/a/android.com/tools/tech-docs/new-build-system/aar-format)的形式来发布。<br>
AndFix支持的版本号从2.3到6.0，同时支持ARM架构和X86机构CUP，以及Dalvik、ART虚拟机机型。它以.apatch文件形式从服务器端拉取到手机端进行bug修复。

#### 实现原理
AndFix的实现原理是**方法体替换**， 原理图如下所示：<br>
![AndFix Principle image](https://github.com/alibaba/AndFix/raw/master/images/principle.png)<br>
修复流程图：<br>
![Fix Process image](https://github.com/alibaba/AndFix/raw/master/images/process.png)
<!-- <img src="https://github.com/alibaba/AndFix/raw/master/images/process.png" width="800px" height="400px" /> -->

#### 集成方式
1. 在gradle文件中引入AndFix依赖库: <br>
`dependencies {
    compile 'com.alipay.euler:andfix:0.4.0@aar'
}`


2. 自定义`Application`类，并在`Application`的`onCreate`方法中加载.apatch文件即可。

```java
private void loadPatchs() { //在Application的onCreate方法中调用
  try {
      mPatchManager = new PatchManager(this);
      String appVersion= getPackageManager().getPackageInfo(getPackageName(), 0).versionName;
      mPatchManager.init(appVersion); //初始化版本信息
      mPatchManager.loadPatch();
      String patchFileString = Environment.getExternalStorageDirectory().getAbsolutePath()
       + APATCH_PATH;  //补丁文件放在根目录下
      File file = new File(patchFileString);
      if (!file.exists()) {
          return;
      }
      mPatchManager.addPatch(patchFileString); //加载补丁文件
  } catch (Exception e) {
      e.printStackTrace();
  }
}
```

#### 补丁文件(即.apatch)生成方式
1. 到该[链接](https://github.com/alibaba/AndFix/raw/master/tools/apkpatch-1.0.3.zip)下载生产补丁的工具包。
2. 准备两个已签名安装包:  即线上出现bug的安装包和bug修复完成的安装包。
3. 通过如下命令生产.apatch补丁文件:
  ![](/img/post-img/post-02-01.png)
例如：`apkpatch -f 2.apk -t 1.apk -o F:\apkpatch -k debug.keystore -p 12345
6 -a android -e 123456` <br>
按照上面代码逻辑，将生成的.apatch补丁文件放到sd卡的根目录，当启动应用的时候就会去加载该文件，如果加载到该文件，就会将修复的逻辑替换有bug的逻辑，从而达到不用升级apk而修复bug的效果。

> 更多细节可以参考开源项目[AndoFix](https://github.com/alibaba/AndFix)。
<br>
