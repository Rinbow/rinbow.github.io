---
title: CMS 的两大问题
tags: [java, jvm]
---

#### CMS 的两大问题

##### Promotion Failed

```
2018-11-03T16:06:56.472+0800: 70589.121: [GC (Allocation Failure) 2018-11-03T16:06:56.472+0800: 70589.121: [ParNew (promotion failed): 7100051K->7116478K(7549760K), 1.4566873 secs]2018-11-03T16:06:57.929+0800: 70590.578: [CMS: 21753651K->8203385K(33554432K), 15.3593321 secs] 28840583K->8203385K(41104192K), [Metaspace: 44827K->44827K(47104K)], 16.8164584 secs] [Times: user=18.22 sys=0.10, real=16.81 secs]
```

由于 CMS 是基于『标记-清除』算法实现的，这就意味着垃圾回收后会产生大量空间碎片，一旦空间碎片过多，同时新生代要晋升的对象又过大时，就会导致老年代放不下从而触发 Full GC（采用Serial Old，很慢）。

**应对方式：**

- CMS 提供了一个 -XX:+UseCMSComapctAtFullCollection 开关参数（默认就是开启的），用于在 CMS 收集器顶不住要进行 Full GC 的时候对内存碎片合并整理，但这个过程无法并发执行，所以停顿时间不得不变长。CMS 还提供了一个参数 -XX:CMSFullGCsBeforeComapction，这个参数指定 CMS 在多少次『标记-清除』的垃圾收集后，跟着来一次『标记-整理』的垃圾收集，以此来腾出更多可用的老年代空间。
- 增大 survivor 空间大小。

##### Concurrent Mode Failure

```
2018-11-02T21:58:23.511+0800: 5141.978: [GC (CMS Initial Mark) [1 CMS-initial-mark: 32408183K(33554432K)] 32540937K(41104192K), 0.0041429 secs] [Times: user=0.05 sys=0.00, real=0.00 secs] 
2018-11-02T21:58:23.515+0800: 5141.982: [CMS-concurrent-mark-start]
2018-11-02T21:58:26.712+0800: 5145.178: [GC (Allocation Failure) 2018-11-02T21:58:26.712+0800: 5145.179: [ParNew: 6841791K->6841791K(7549760K), 0.0000304 secs]2018-11-02T21:58:26.712+0800: 5145.179: [CMS2018-11-02T21:58:30.128+0800: 5148.595: [CMS-concurrent-mark: 6.600/6.613 secs] [Times: user=50.97 sys=1.96, real=6.61 secs] 
 (concurrent mode failure): 32408183K->9422134K(33554432K), 21.5011902 secs] 39249974K->9422134K(41104192K), [Metaspace: 44456K->44456K(47104K)], 21.5016087 secs] [Times: user=32.14 sys=2.26, real=21.50 secs] 
```

这是因为 CMS 不能像其他收集器那样等到老年代机会完全被填满了再进行收集，需要预留一部分空间提供并发收集时的程序运作使用。当 CMS 在回收老年代垃圾的过程中，新生代有对象要晋升到老年代，而老年代有没有足够空间，就会导致这个问题。

**应对方式：**调节 -XX:CMSInitiatingOccupancyFraction 参数，这个参数表示触发 CMS 垃圾回收的阈值，JDK1.5 中默认68%，JDK1.6 中默认为 92%。如果频繁出现 Concurrent Mode Failure 的情况，可以适当调低这个阈值，虽然这会加快老年代回收的频率，但可以有效减少 Concurrent Mode Failure 问题的出现。