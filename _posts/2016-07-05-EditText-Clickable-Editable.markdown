---
layout:     post
title:      "EditText中的链接可点击、可编辑"
subtitle:   "设置EditText中的链接可点击、同时可以继续编辑"
date:       2016-07-05 09:12:00
author:     "SQSong"
header-img: "img/post-06-bg.jpg"
header-mask: 0.3
catalog:    true
tags:
    - android
    - EditText
---

##### 场景
在项目开发中，有可能会遇到这样的场景: `EditText`包含一个链接，点击该链接部分可以打开该链接，但是点击该链接以外的区域，需要继续对EditText中的内容进行编辑。
![demo img](/img/post-img/post-04-01.png)

##### 解决方案
我们可以使用`LinkMovementMethod`类来处理文本的链接，但是整个文本中的最后一个字符还是属于链接内容，那么点击链接以外的部分(即图中可编辑区域)也会打开链接，显然这不是我们想要的。<br>
**解决方案:** 在链接字符串的末尾加上一个特殊字符用以区分链接与可编辑区域的边界:

```java
mBinding.editText.setMovementMethod(LinkMovementMethod.getInstance());
Spannable spannable = new SpannableString("http://sqsong.me");
Linkify.addLinks(spannable, Linkify.WEB_URLS);
//如果去掉该行，则无法区分链接区域和可编辑区域，当点击EditText的编辑区域时也会进行链接的跳转
CharSequence text = TextUtils.concat(spannable, "\u200B"); //​​"\u200B"表示一个宽度为零的空字符
mBinding.editText.setText(text);
```
OK！ 现在点击链接部分时就可以打开链接，当点击`EditText`的空白区域时就可以进行继续编辑了。

> 参考来源：[http://blog.danlew.net/2015/12/14/making-edittexts-with-links-both-clickable-and-editable](http://blog.danlew.net/2015/12/14/making-edittexts-with-links-both-clickable-and-editable/)
