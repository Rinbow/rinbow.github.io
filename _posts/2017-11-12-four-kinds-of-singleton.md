---
title: 线程安全的单例模式的几种实现方式
tags: [java, design pattern]
---

 单例模式是23种设计模式应用最广的一种，也是最简单的一种设计模式。它确保一个类只有一个实例对象。

实现方式有很多种，具体如下：

一、饿汉式（不使用同步方法）

```java
public class Singleton {

    private static Singleton singleton = new Singleton(); // 直接初始化一个实例对象

    private Singleton() { // 私有构造方法，保证该类对象不能直接被new出来

    }

    public static Singleton getInstance() { // 公有静态方法对外提供获取对象实例的途径
        return singleton;
    }

}
```

顾名思义，这种方法直接把对象实例 new 出来了，不管该对象是否用到，所以造成了一定程度的资源浪费，不推荐。

二、懒汉式（使用同步方法）

```java
public class Singleton {

    private static Singleton singleton;

    private Singleton() {

    }

    public static synchronized Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }

}
```

这种方法是在需要获取实例的时候才去创建对象，getInstance() 方法加锁，保证了线程安全，但同时由于是对整个对象上锁，所以也造成了性能的下降。

三、双重检测（DCL）

```java
public class Singleton {

    private static volatile Singleton singleton;

    private Singleton() {

    }

    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }

}
```

作为第二种方法的改进版，DCL 缩小了加锁的粒度，保证线程安全的同时提高了性能。volatile 保证有序性（禁止指令重排序）和可见性。

四、枚举类型

```java
class Resource {

}
enum Singleton {
    INSTANCE;

    private Resource resource = new Resource();

    public Resource getInstance() {
        return resource;
    }

}
```

实际上 enum 继承自 Enum 类，而且是 final 的，有且仅有 private 构造方法，每个枚举默认是 static final 的，也就保证了 INSTANCE 只能被实例化一次。 只需要通过调用 Singleton.INSTANCE.getInstance() 方法即可获取实例。

五、静态内部类

```java
public class Singleton {

    private Singleton() {

    }

    private static class Inner {
        private static Singleton singleton = new Singleton();
    }

    public static Singleton getInstance() {
        return Inner.singleton;
    }

}
```

采用这种方式，第一次加载 Singleton 类时，并不会实例化单例对象，只有第一次调用 getInstance() 方法时会导致虚拟机加载 Inner 类，这种方式不仅能保证对象的单一性，还避免加锁带来的性能问题，同时使用了延迟加载，保证了线程安全性，所以是实现单例模式的最佳方式。

---

### 参考资料

- [设计模式之单例模式(线程安全)](http://www.cnblogs.com/xudong-bupt/p/3433643.html)

- [单例模式的五种实现方式](http://blog.csdn.net/soul_code/article/details/50183765)

- [Java 利用枚举实现单例模式](http://blog.csdn.net/yy254117440/article/details/52305175)

- [Java枚举enum以及应用：枚举实现单例模式](http://www.cnblogs.com/cielosun/p/6596475.html)


