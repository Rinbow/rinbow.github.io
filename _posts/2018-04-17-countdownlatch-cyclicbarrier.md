---
title: CountDownLatch 和 CyclicBarrier
tags: [java]
---

这两个工具类都可以简单看作线程计数器，表面上功能相似，其实区别还是不小的。

### CountDownLatch

CountDownLatch 可以理解为一个计数器在初始化时设置初始值，当一个线程需要等待某些操作先完成时，需要调用 await() 方法。这个方法让线程进入休眠状态直到等待的所有线程都执行完成。每调用一次 countDown() 方法内部计数器减 1 ，直到计数器为 0 时唤醒。

示例程序：

```java
import java.util.Random;
import java.util.concurrent.CountDownLatch;

/**
 * 7个人赛跑
 */
public class TCountDownLatch {
    private static int count = 8;
    private static CountDownLatch c = new CountDownLatch(count);

    public static void main(String[] args) {
        for (int i = 1; i <= count; i++) {
            int index = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(new Random().nextInt(3000));
                        System.out.println("第" + index + "个人抵达终点");
                        c.countDown();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
        try {
            c.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("所有人都已到达终点");
    }

}
```

输出结果：

```
第8个人抵达终点
第5个人抵达终点
第2个人抵达终点
第7个人抵达终点
第3个人抵达终点
第6个人抵达终点
第4个人抵达终点
第1个人抵达终点
所有人都已到达终点
```

### CyclicBarrier

CyclicBarrier 允许两个或者多个线程在某个集合点同步。当一个线程到达集合点时，它将调用 await() 方法等待其它的线程。线程调用 await() 方法后，CyclicBarrier 将阻塞这个线程并将它置入休眠状态等待其它线程的到来。直到最后一个线程调用 await() 方法时，CyclicBarrier 将唤醒所有等待的线程然后这些线程将继续执行。CyclicBarrier 还可以传入另一个 Runnable 对象作为初始化参数，当所有的线程都到达集合点后，CyclicBarrier 类将 Runnable 对象作为线程执行。

示例程序：

```java
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

/**
 * 7个人赛跑
 */
public class TCyclicBarrier {
    private static int count = 8;
    private static CyclicBarrier c = new CyclicBarrier(count, new Runnable() {
        @Override
        public void run() {
            System.out.println("所有人都已到达终点");
        }
    });

    public static void main(String[] args) {
        for (int i = 1; i <= count; i++) {
            int index = i;
            new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(new Random().nextInt(3000));
                        System.out.println("第" + index + "个人抵达终点");
                        c.await();
                    } catch (InterruptedException | BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
        }
    }
}
```

输出结果：

```
第6个人抵达终点
第1个人抵达终点
第7个人抵达终点
第8个人抵达终点
第5个人抵达终点
第3个人抵达终点
第4个人抵达终点
第2个人抵达终点
所有人都已到达终点
```

### 区别和联系

从上面的代码可以看出，二者可以实现同一功能，不过 CyclicBarrier 要比 CountDownLatch 更灵活一些，功能更强大一些。具体区别如下：

1. CountDownLatch 采用减计数方式，count 值为0时释放所有等待线程；CyclicBarrier 采用加计数方式，count 值达到指定值是释放所有等待线程。
2. CountDownLatch 的 count 值为 0 时，不可以重置；CyclicBarrier 的count 值到达指定值时，count 自动从0开始。
3. CountDownLatch 不可以重复利用；CyclicBarrier可重复利用，reset() 方法可以使其回到初始状态。
4. CountDownLatch 是阻塞主线程，在任务线程中倒数计数，直到任务线程完成才唤醒主线程继续执行；CyclicBarrier 是阻塞任务线程，直到所有任务线程执行完成才各自继续执行。

---

### 参考资料

- [Java多线程之CountDownLatch和CyclicBarrier同步屏障的使用](http://www.cnblogs.com/ygj0930/p/6558349.html)

- [java并发之同步辅助类（Semphore、CountDownLatch、CyclicBarrier、Phaser）](http://www.cnblogs.com/uodut/p/6830939.html)

- [Java多线程编程-（8）-两种常用的线程计数器CountDownLatch和循环屏障CyclicBarrier](https://blog.csdn.net/bntX2jSQfEHy7/article/details/78237208)

- [CyclicBarrier和CountDownLatch区别](https://blog.csdn.net/tolcf/article/details/50925145)

- [Java并发编程：CountDownLatch、CyclicBarrier和Semaphore](http://www.cnblogs.com/dolphin0520/p/3920397.html)

