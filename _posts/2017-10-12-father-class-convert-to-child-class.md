---
title: Java 中父类对象能不能强制转换为子类对象
tags: [java]
---

昨天遇见一道笔试题，大意是给定如下代码，问编译运行有没有错：

```java
public class Solution {
    public static void main(String[] args) {
        Father father = new Father();
        Child child = (Child) father;
    }
}
class Father {}
class Child extends Father {}
```

没仔细看直接选了「编译运行正常」。

交卷后测试了一下发现结果却是：

```
Exception in thread "main" java.lang.ClassCastException: Father cannot be cast to Child
```

其实细想一下，子类的内容是大于等于父类的，把父类转成子类，将来需要调用子类特有的方法，那不是逻辑冲突了吗。所以以上代码「编译正常，运行出错」。

反过来说，子类转父类是完全可以，不过这是一种损失精度的转变。看下面这段代代码：

```java
public class Solution {
    public static void main(String[] args) {
        Child child = new Child();
        Father father = (Father) child;
        father.fMethod();
        father.cMethod(); // compile error
    }
}
class Father {
    public void fMethod(){
        System.out.println("father's method");
    }
}
class Child extends Father {
    public void cMethod(){
        System.out.println("child's method");
    }
}
```

子类转成父类之后，损失了自己特有的方法 `cMethod()`，只能调用父类的方法 `fMethod()`。

那么是不是父类对象一定不能转成子类对象，看看下面这种特殊的形式：

```java
public class Solution {
    public static void main(String[] args) {
        Father father = new Child();
        Child child = (Child) father;
        child.cMethod();
        child.fMethod();
    }
}
class Father {
    public void fMethod() {
        System.out.println("father's method");
    }
}
class Child extends Father {
    public void cMethod() {
        System.out.println("child's method");
    }
}
```

在这个示例代码中，`father` 实际上父类引用指向子类对象，它实际上还是一个 `Child` 对象 ，所以这里转换是完全没有问题的。不会报错，也不会损失精度。

其实说白了，在判断父类对象能不能转化为子类对象的时候，只要判断父类对象是不是一个子类对象就可以了，反映到代码上就是 `father instanceof Child`，如果是，可以转；不是，不可以转。