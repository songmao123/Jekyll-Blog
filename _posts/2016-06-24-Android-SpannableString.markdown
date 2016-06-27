---
layout:     post
title:      "Android TextView文字格式化"
subtitle:   "使用SpannableString对TextView文本进行格式化处理"
date:       2016-06-20 12:11:00
author:     "SQSong"
header-img: "img/post-01-bg.jpg"
header-mask: 0.3
catalog:    true
tags:
    - android
---

## 使用`SpannableString`对字符串样式进行处理
Android中经常遇到在`TextView`中设置的文字中，有部分文字需要设置不同的样式， 例如：部分文字需要设置不同的背景色、文字颜色，不同大小字体，或识别文字中的超链接文本等等，这时就需要使用到`SpannableString`来对文字设置不同的属性了。

#### 设置文字的样式
`TextVeiew`文字样式常见如下：

```java
textView = (TextView) findViewById(R.id.textView);
String text = "蓝色背景，大字体，红色字体，斜体，粗体，粗斜体，下划线";
SpannableString spannableString = new SpannableString(text);

//设置背景色
spannableString.setSpan(new BackgroundColorSpan(Color.BLUE), 0, 4, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置文字大小
spannableString.setSpan(new AbsoluteSizeSpan(30, true), 5, 8, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置前景色
spannableString.setSpan(new ForegroundColorSpan(Color.RED), 9, 13, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置斜体
spannableString.setSpan(new StyleSpan(Typeface.ITALIC), 14, 16, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置粗体
spannableString.setSpan(new StyleSpan(Typeface.BOLD), 17, 19, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置粗斜体
spannableString.setSpan(new StyleSpan(Typeface.BOLD_ITALIC), 20, 23, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
//设置下划线
spannableString.setSpan(new UnderlineSpan(), 24, 27, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
textView.setText(spannableString);
```
以上设置效果图如下：<br> ![](/img/post-img/post-01-01.png)

#### 识别`String`中的链接或特殊标记
由于项目中需要做类似QQ空间动态，需要对动态内容中的url地址链接以及用户名称进行识别，所以对`TextView`中地址匹配做了一些研究，最后参考了开源项目<a href="https://github.com/qii/weiciyuan">微次元</a>的做法。

1. 设置布局<br>先设置一个简单布局：一个`EditText`、一个`Button`、一个`TextView`，当点击`Button`的时候，将`EditText`中输入的文本进行链接识别并显示到`TextView`中。
<br>
2. 对输入框中的文本进行识别转换：

```java
public static SpannableString convertNormalStringToSpannableString(String text) {
    if (TextUtils.isEmpty(text)) {
        return null;
    }

    SpannableString result = SpannableString.valueOf(text);
    //对字符串中的url地址和人命称使用正则表达式进行匹配
    Linkify.addLinks(result, WeiboPatterns.WEB_URL, WeiboPatterns.WEB_SCHEME);
    //对话题进行匹配
    Linkify.addLinks(result, WeiboPatterns.TOPIC_URL, WeiboPatterns.TOPIC_SCHEME);
    //对@进行匹配
    Linkify.addLinks(result, WeiboPatterns.MENTION_URL, WeiboPatterns.MENTION_SCHEME);

    URLSpan[] urlSpans = result.getSpans(0, result.length(), URLSpan.class);
    MyURLSpan mUrlSpan = null;
    //对不同得的span进行标记
    for (URLSpan urlSpan : urlSpans) {
        mUrlSpan = new MyURLSpan(urlSpan.getURL());
        int start = result.getSpanStart(urlSpan);
        int end = result.getSpanEnd(urlSpan);
        result.removeSpan(urlSpan);
        result.setSpan(mUrlSpan, start, end, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
    }
    return result;
}
```

`WeiboPatterns`:

```java
public class WeiboPatterns {
  //url地址匹配表达式
  public static final Pattern WEB_URL = Pattern
          .compile("http://[a-zA-Z0-9+&@#/%?=~_\\-|!:,\\.;]*[a-zA-Z0-9+&@#/%=~_|]");
  //话题匹配表达式
  public static final Pattern TOPIC_URL = Pattern
          .compile("#[\\p{Print}\\p{InCJKUnifiedIdeographs}&&[^#]]+#");
  //@匹配表达式
  public static final Pattern MENTION_URL = Pattern
          .compile("@[\\w\\p{InCJKUnifiedIdeographs}-]{1,26}");

  public static final String WEB_SCHEME = "http://";
  public static final String TOPIC_SCHEME = BuildConfig.APPLICATION_ID + ".topic://";
  public static final String MENTION_SCHEME = BuildConfig.APPLICATION_ID + ".mention://";
}
```

`MyURLSpan`:  

```java
public class MyURLSpan extends ClickableSpan implements ParcelableSpan {

  private final String mURL;

  public MyURLSpan(String url) {
      mURL = url;
  }

  public MyURLSpan(Parcel src) {
      mURL = src.readString();
  }

  @Override
  public int getSpanTypeId() {
      return 11;
  }

  public int describeContents() {
      return 0;
  }

  public void writeToParcel(Parcel dest, int flags) {
      dest.writeString(mURL);
  }

  public String getURL() {
      return mURL;
  }

  public void onClick(View widget) { //识别链接的点击事件
      Uri uri = Uri.parse(getURL());
      Context context = widget.getContext();
      Toast.makeText(context, uri.toString(), Toast.LENGTH_SHORT).show();
  }

  @Override
  public void updateDrawState(TextPaint tp) {  //链接字体的颜色
      tp.setColor(Color.parseColor("#800080"));
//        tp.setUnderlineText(true);
  }
}
```  

3. 在`MainActivity`中处理`Button`的点击事件:

```java
private void recognizeLinks() {
    button.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            String input = inputEdit.getText().toString();
            SpannableString spannableString = StringUtils.convertNormalStringToSpannableString(input);
            textView.setText(spannableString);
        }
    });
}
```

最终效果图如下:
![](/img/post-img/post-01-02.png)

> 在此感谢<a href="https://github.com/qii/weiciyuan">微次元</a>项目提供的参考。
