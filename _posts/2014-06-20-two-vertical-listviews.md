---
layout: post
title: 一个 activity 中垂直排列两个 listview 
tags:
  - android
---

原生 Android 并不支持在一个 activity 中垂直地排列两个 listview，本文章中介绍了一种实现方法。

## 原理

为了实现这种效果，一开始我想到的办法是 listview 中加上一个 footerview，footerview 里面嵌套一个listview，但是实际操作之后发现 footerview 里的 listview 只显示一项，这个问题困扰了半天，一直没有找到合适的解决办法，直到昨天晚上偶然看到一篇博文介绍说，scrollview 里嵌套 listview 也出现了同样的问题，解决办法是动态设置 listview 的高度，于是我用这种方法也试了试，结果还真解决了。

## 关键代码

动态设置 listview 的高度：

```java
public void setListViewHeightBasedOnChildren(ListView listView) {
    ListAdapter adapter = listView.getAdapter();
    if (adapter != null) {
        int totalHeight = 0;
        System.out.println(adapter.getCount());
        for (int i = 0; i < adapter.getCount(); i++) {
            View listItem = adapter.getView(i, null, listView);
            listItem.measure(0, 0);
            totalHeight += listItem.getMeasuredHeight();
        }
        ViewGroup.LayoutParams params = listView.getLayoutParams();
        params.height = totalHeight + (listView.getDividerHeight() * (adapter.getCount() - 1));
        ((MarginLayoutParams) params).setMargins(0, 0, 0, 0);
        listView.setLayoutParams(params);
    }
}
```

footerview.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:id="@+id/linearlayout"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    android:orientation="vertical" >  
  
    <LinearLayout  
        android:layout_width="fill_parent"  
        android:layout_height="fill_parent"  
        android:orientation="horizontal" >  
  
        <TextView  
            android:layout_width="wrap_content"  
            android:layout_height="wrap_content"  
            android:layout_weight="1"  
            android:text="分隔标识=================" />  
  
    </LinearLayout>  
  
    <ListView  
        android:id="@+id/head_listView"  
        android:layout_width="fill_parent"  
        android:layout_height="fill_parent" >  
    </ListView>  
  
</LinearLayout>  
```

## 实现效果

![twoverticallistviews_screenshot](\media\files\2014\06\20\twoverticallistviews_screenshot.png)

## 代码地址

[**TwoVerticalListviews**](https://github.com/Silocean/TwoVerticalListview.git)

