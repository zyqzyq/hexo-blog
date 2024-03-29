---
title: Toolbar Kotlin 简单使用
urlname: 320456e39492553e4d43a6b5b8db930b
date: '2017-08-14 16:25:06 +0800'
layout: post
categories: kotlin
tags:
  - kotlin
---

## 准备工作

1.在 build.gradle 添加依赖

```
ext.anko_version = '0.10.1'
```

```
apply plugin: 'kotlin-android-extensions'
```

```
compile "org.jetbrains.anko:anko-commons:$anko_version"
```

2.将 AppTheme 改为继承 noActionBar

```
<style name="AppTheme" parent="Theme.AppCompat.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```

## 创建 toolbar

编辑 activity_main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context="com.zyqzyq.myapplication.MainActivity">
    <android.support.v7.widget.Toolbar
		    android:id="@+id/mToolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:minHeight="?attr/actionBarSize"
            android:background="@color/colorPrimary"
            app:contentInsetStart="0dp">
        <TextView
                android:id="@+id/mTitle"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:text="标题"
                android:textSize="36sp"
                android:gravity="center"/>
    </android.support.v7.widget.Toolbar>
    <TextView
            android:id="@+id/mTextView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Hello World!"
            android:layout_centerInParent="true"/>
</RelativeLayout>
```

修改 MainAcitivity.kt
在 MainActivity.kt 中添加如下代码

```
//将mtoolbar设置为actionbar
setSupportActionBar(mToolbar)
//APP 图标
mToolbar.setLogo(R.drawable.ic_launcher)
//APP 标题
mToolbar.title = ""
mTitle.text = "标题居中"
//APP 正文内容
mTextView.text = "正文内容"
```

## 添加 menu

新建 /res/menu/menu_toolbar.xml

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:app="http://schemas.android.com/apk/res-auto" xmlns:android="http://schemas.android.com/apk/res/android">
    <item
            android:id="@+id/app_bar_search"
            android:icon="@drawable/ic_search_black_24dp"
            android:title="Search"
            android:actionViewClass="android.widget.SearchView"
            app:showAsAction="ifRoom"/>
    <item android:title="Item" app:showAsAction="never"/>
    <item android:title="Item" app:showAsAction="never"/>
</menu>
```

将 menu 加入 toolbar 并在 MainActivity.kt 中添加监听。

```
//APP 菜单栏
mToolbar.inflateMenu(R.menu.menu_toolbar)
//菜单栏监听
mToolbar.setOnMenuItemClickListener {
    when(it.itemId){
        R.id.app_bar_search -> toast("search")
        else -> toast("click something else")
    }
    true
}
```

菜单栏未正常显示，注释 setSupportActionBar(mToolbar)后可以正常显示（暂未找到原因，有懂的希望告知下，谢谢）
运行代码后成功界面
![](http://img.blog.csdn.net/20170814100716932?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzI3ODMzNTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast#crop=0&crop=0&crop=1&crop=1&id=NWghO&originHeight=722&originWidth=406&originalType=binary∶=1&rotation=0&showTitle=false&status=done&style=none&title=)
（莫名的发现标题不居中了，注销了 setSupportActionBar 以后居中正常显示，感觉有毒，还需在研究研究）
