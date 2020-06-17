---
title: JVM 参数学习
date: 2020-06-05 15:38:41
tags: Java
---

## JVM 参数学习

### 常用参数

| 参数名                  | 解释                                |
| ----------------------- | ----------------------------------- |
| -Xms                    | 初始堆大小                          |
| -Xmx                    | 堆最大值                            |
| -Xmn                    | 堆年轻代大小                        |
| -Xss                    | 线程栈大小                          |
| -XX:MaxMetaspaceSize    | 修改元空间大小                      |
| -XX:MaxDirectMemorySize | 修改直接内存大小                    |
| -XX:NewRatio            | 修改新生代与老年代比例，默认比例为2 |
| -XX:SurvivorRatio       | 改eden与survivor的比例，默认是8     |
| -XX:+PrintGC            | 打印GC信息，默认禁止                |
| -XX:+PrintGCDetails     | 打印GC详细信息                      |
| -XX:+PrintHeapAtGC      | GC 前后打印堆详细信息               |


<escape><!-- more --></escape>
### Java 堆相关参数设置

- -Xms

  -Xms20M,表示JVM Heap(堆内存)最小尺寸20MB，**最开始只有 -Xms 的参数，表示 `初始` memory size(m表示memory，s表示size)**，属于初始分配20M，-Xms表示的 `初始` 内存也有一个 `最小` 内存的概念（其实常用的做法中初始内存采用的也就是最小内存）。

- -Xmx

  -Xmx128M,表示JVM Heap(堆内存)**最大允许**的尺寸128MB，按需分配。如果 -Xmx 不指定或者指定偏小，也许出现java.lang.OutOfMemory错误，此错误来自JVM不是Throwable的，无法用try...catch捕捉

当使用如下配置时 -Xms20M -Xmx128M ，JVM 内存情况如下：

```java
System.out.print("Xmx =");
System.out.println(Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");

System.out.print("free mem =");
System.out.println(Runtime.getRuntime().freeMemory() / 1024.0 / 1024 + "M");

System.out.print("total mem =");
System.out.println(Runtime.getRuntime().totalMemory() / 1024.0 / 1024 + "M");

--output
Xmx =114.0M
free mem =17.08477020263672M
total mem =19.5M

```

**Java会尽量的维持在最小堆运行，即使设置的最大值很大，只有当GC之后也无法满足最小堆，才会去扩容。**

- Xmn

  新生代堆内存大小

- -XX:NewRatio

  新生代和老年代的比值，如 XX:NewRatio4 表示（eden+s0+s1) : 老年代 = 1：4

- XX:SurvivorRatio

  两个 Survivor 区和 Eden 区的比值，比如-XX:SurvivorRatio8表示两个 Survivor : eden=2:8，即一个Survivor占年轻代的1/10。

- OOM时导出堆到文件进行内存分析和问题排查

  -XX:+HeapDumpOnOutOfMemoryError，-XX:+HeapDumpPath 导出OOM的路径，比如：

  -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/a.dump

- 栈大小分配 -Xss

  通常只有几百K，一般很少调大，它的大小决定了函数调用的深度。如果想跑更多的线程，需要把栈用 -Xss 尽量调小，而不是变大！因为线程越多，每个线程都要分配内存空间，这样每个栈的空间越大，占据的内存越多。但是也得预防很深的函数调用可能导致栈内存溢出问题，比如不合适的递归调用（斐波拉契数列计算）。

#### 堆的分配参数总结

- 根据实际事情调整新生代和幸存代的大小
- 官方推荐新生代占堆的3/8
- 幸存代占新生代的1/10
- 在OOM时，记得Dump出堆，确保可以排查现场问题