---
layout: post
title: Java 中 Integer 的各种比较
categories: [Java]
tags:
  - program
  - java
---

Java 中 Integer 的比较问题一直比较蛋疼，稍不注意就会出错，这里总结一下主要比较方式及区别。


- 基本型和基本型封装型进行“==”运算符的比较，基本型封装型将会自动拆箱变为基本型后再进行比较。


```java
int a = 100;
Integer b = new Integer(100);
System.out.println(a==b); // true
```

`Integer(100)` 自动拆箱变为 int 类型，所以返回 true。

- 两个Integer类型进行“==”比较，如果其值在-128至127，那么返回 true，否则返回 false。

```java
Integer c = 100;
Integer d = 100;
System.out.println(c==d) // true
Integer e = 1000;
Integer f = 1000;
System.out.println(e==f) // false
```
`Integer c = 100`，它的内部是调用 `Integer.valueOf(100)`，其内部源码是这样的：

```java
    /**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
由于 100 在 [-128, 127] 之间，所以 c 和 d 直接取自缓存；而 1000 不在此区间内，所以 e 和 f 是 new 的 Integer 对象。

**注意：Float, Double 不适用此规则。**

- 两个基本型的封装型进行 `equals()` 比较，首先 `equals()` 会比较类型，如果类型相同，则继续比较值，如果值也相同，返回 true。

```java
Integer g = new Integer(12);
Integer h = new Integer(12);
Long i = new Long(12);
System.out.println(g.equals(h)); // true
System.out.println(g.equals(i)); // false
```

- 对封装类型调用 `equals()`,但是参数是基本类型，这时候，先会进行自动装箱，基本型转换为其封装类型，再进行上一步中的比较。

```java
Integer j = new Integer(34);
int k = 34;
double m = 34.0;
System.out.println(j.equals(k)); // true
System.out.println(j.equals(m)); // false
```

**注意：Long 类型同 Integer 比较规则一样。**