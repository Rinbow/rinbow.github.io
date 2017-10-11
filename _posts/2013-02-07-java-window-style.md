---
layout: post
title: 调整 Java 中窗口界面风格
categories: [Java]
tags:
  - program
  - java
---

Java 的 Swing 套件默认的界面窗口太丑了，不过还好 Java 提供了其他几种风格可供替换。

以下代码输出操作系统所支持的界面风格的名字：

```java
UIManager.LookAndFeelInfo[] looks = UIManager.getInstalledLookAndFeels();
for(UIManager.LookAndFeelInfo look : looks) {
    System.out.println(look.getClassName());
}
```

输出结果如下：

```
javax.swing.plaf.metal.MetalLookAndFeel
javax.swing.plaf.nimbus.NimbusLookAndFeel
com.sun.java.swing.plaf.motif.MotifLookAndFeel
com.sun.java.swing.plaf.windows.WindowsLookAndFeel
com.sun.java.swing.plaf.windows.WindowsClassicLookAndFeel
```

以下代码更改界面默认字体：

```java
private static void InitGlobalFont(Font font) {
    FontUIResource fontRes = new FontUIResource(font);
    for (Enumeration<Object> keys = UIManager.getDefaults().keys(); keys.hasMoreElements();) {
        Object key = keys.nextElement();
        Object value = UIManager.get(key);
        if (value instanceof FontUIResource) {
            UIManager.put(key, fontRes);
        }
    }
}
```

