---
layout:     post
title:      "Android DataBinding"
subtitle:   "Android 布局绑定数据源"
date:       2016-06-28 10:57:00
author:     "SQSong"
header-img: "img/post-06-bg.jpg"
header-mask: 0.3
catalog:    true
tags:
    - android
    - DataBinding
---

## Data Binding Library介绍
Android开发中经常使用`findViewById`方法来查找布局中的控件，在稍微复杂点的布局中，就会需要写一大堆重复但又不得不写的代码，看着着实让人糟心。不过好消息来啦，最近Android官方提供了Data Binding库，它可以将我们的布局和数据源进行绑定，从而减少重复编写`findViewById`的代码啦！<br>
官方Data Binding Library支持Android 2.1 (API level 7+)及以上版本，同时Gradle插件也需要1.5.0-alpha1及以上版本。

## Data Binding的配置及使用

#### 环境配置
使用DataBinding需要我们在工程的build.gradle文件中配置如下代码:

```
android {
    ....
    dataBinding {
        enabled = true
    }
}
```

#### 设置布局文件
布局文件的根布局需要使用`<layout>`标签来替换，其他的用法与之前相同。在布局里面，使用`<data>`标签来引入需要使用到的数据源JavaBean对象，`<variable>`标签中的`name`属性是使用该数据源对象的名称，`type`属性是该对象的引用路径。布局文件中的控件就可以使用`@{}`来引用数据源中的数据。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="user"
            type="com.example.hotfixpatchdemo.bean.User" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="horizontal"
        tools:context="com.example.hotfixpatchdemo.DataBindingActivity">

        <TextView
            android:id="@+id/tv_firstName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{user.firstName}"
            android:textColor="@android:color/black" />

        <TextView
            android:id="@+id/tv_lastName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="10dp"
            android:text="@{user.lastName}"
            android:textColor="@android:color/black" />

    </LinearLayout>
</layout>
```

数据源JavaBean对象需要提供`Getter`方法来获取对象中的字段：

```java
public class User {
   private final String firstName;
   private final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
   public String getFirstName() {
       return this.firstName;
   }
   public String getLastName() {
       return this.lastName;
   }
}
```

#### 绑定数据
默认情况下，设置完布局后会根据你的布局文件名称自动生成一个以`Binding`结尾的绑定类。例如：我的布局文件名称为`activity_data_binding.xml`，布局文件设置完成后会生成一个名称为`ActivityDataBindingBinding`的类。在代码中通过`DataBindingUtil`工具类类设置布局，设置完成后会返回一个`ActivityDataBindingBinding`类型对象，我们再可以通过该对象来绑定数据。这样就可以将数据源设置到布局文件对应的控件中。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //setContentView(R.layout.activity_data_binding);
    ActivityDataBindingBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_data_binding);
    User user = new User("Jhon", "Sandy");
    binding.setUser(user);

    TextView tvFirstName = binding.tvFirstName;
    TextView tvLastName = binding.tvLastName;
}
```
如上面代码所示，我们可以通过`binding`对象来获取布局中对应的控件。

#### 控件的事件处理
Data Binding允许我们书写表达式来处理一些`view`的分发事件，例如：`onClickListener`、`onLongClickListener`，这些监听在布局文件中都有对ing的属性名称`android:onClick`、`android:onLongClick`，所以我们可以在布局文件中使用表达式来处理该事件。

1. 自定义一个处理事件的类，该类中包含处理事件的方法。

```java
public class ViewClickHandler {
   public void onViewClicked(View view, String name) {
       Toast.makeText(context, "You Clicked " + name, Toast.LENGTH_SHORT).show();
   }
}
```


2. 在布局文件中引入数据源以及事件处理的方法。<br>
  首先需要在`<data>`标签中引入事件处理类，再通过表达式`@{(theView) -> clickHandler.onViewClicked(theView, user.lastName)}`来进行参数的传递。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="user"
            type="com.example.hotfixpatchdemo.bean.User" />

        <variable
            name="clickHandler"
            type="com.example.hotfixpatchdemo.adapter.ListItemAdapter.ViewClickHandler" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?android:attr/selectableItemBackground"
            android:onClick="@{(theView) -> clickHandler.onViewClicked(theView, user.lastName)}"
            android:padding="10dp"
            android:text="@{user.firstName}"
            android:textColor="@android:color/black"
            android:textSize="18sp" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:background="?android:attr/selectableItemBackground"
            android:onClick="@{(theView) -> clickHandler.onViewClicked(theView, user.firstName)}"
            android:padding="10dp"
            android:text="@{user.lastName}"
            android:textColor="@android:color/black" />
    </LinearLayout>

</layout>
```

## ListView使用Data Binding
ListView使用DataBinding也需要使用`DataBindingUtil`来填充布局文件:<br>
`ItemListBinding binding = DataBindingUtil.inflate(inflater, R.layout.item_list, parent, false);`
我们可以通过`binding`类中的`getRoot()`方法来获取`convertView`并返回给`getView`方法, 具体逻辑可以参看以下代码:

```java
public class ListItemAdapter extends BaseAdapter {

    private Context context;
    private List<User> mUsers;
    private LayoutInflater inflater;

    public ListItemAdapter(Context context, List<User> mUsers) {
        this.context = context;
        this.mUsers = mUsers;
        this.inflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return mUsers.size();
    }

    @Override
    public Object getItem(int position) {
        return mUsers.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    public int getPosition(User user) {
        return mUsers.indexOf(user);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ItemListBinding binding;
        if (convertView == null) {
            binding = DataBindingUtil.inflate(inflater, R.layout.item_list, parent, false);
            convertView = binding.getRoot();
            convertView.setTag(binding);
        } else {
            binding = (ItemListBinding) convertView.getTag();
        }
        User user = mUsers.get(position);
        binding.setUser(user);
        binding.setClickHandler(new ViewClickHandler());
        return convertView;
    }

    public class ViewClickHandler {
        public void onViewClicked(View view, String name) {
            Toast.makeText(context, "You Clicked " + name, Toast.LENGTH_SHORT).show();
        }
    }
}
```

item布局见上面的xml文件，通过以上方式就可以实现ListView的数据源和布局的绑定，并可以处理控件的相关事件逻辑。<br>
实现效果图如下所示：<br>
![demo gif](/img/post-img/post-03-img.gif)

> 本文参考自Android官网 [Data Binding介绍](https://developer.android.com/topic/libraries/data-binding/index.html#method_references)，更多细节请参看官网介绍！
