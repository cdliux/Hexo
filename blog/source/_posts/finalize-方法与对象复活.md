---
title: finalize 方法与对象复活
date: 2020-06-04 16:55:15
tags: Java
---
### finalize

finalize 是Object的 protected 方法，子类可以覆盖该方法以实现资源清理工作，GC在回收对象之前调用该方法。

**finalize()在什么时候被调用?**

- 所有对象被Garbage Collection时自动调用,比如运行 System.gc() 的时候。
- 程序退出时为每个对象调用一次finalize方法。
- 显式的调用finalize方法。
<escape><!-- more --></escape>
不建议用finalize方法完成“非内存资源”的清理工作，因为 Java 语言规范并不保证 finalize 方法会被及时地执行、而且根本不会保证它们会被执行，而且 finalize 方法可能会带来性能问题。因为JVM通常在单独的低优先级线程中完成 finalize 的执行，finalize 方法中，可将待回收对象赋值给 GC Roots 可达的对象引用，从而达到对象再生的目的。finalize 方法至多由 GC 执行一次(用户当然可以手动调用对象的 finalize 方法，但并不影响GC对finalize的行为)

但建议用于：

- 清理本地对象(通过JNI创建的对象)；
- 作为确保某些非内存资源(如Socket、文件等)释放的一个补充：在finalize方法中显式调用其他资源释放方法。

一个对象复活的实例。

```java
public class ObjectRelive {
    public static ObjectRelive obj;

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method excuted.");
        obj = this;
    }

    @Override
    public String toString() {
        return "I am CanReliveObj.";
    }

    public static void invokeObjectRealive() throws InterruptedException {
        obj = new ObjectRelive();
        obj = null; //复活对象

        System.gc();//主动发起一次 GC
        Thread.sleep(500); //finalize 方法会由低优先级线程执行，此处进行等待

        if (obj == null) {
            System.out.println("obj is null.");
        } else {
            System.out.println("obj is not null.");
        }

        obj = null;
        System.out.println("第二次 GC.");
        System.gc();
        Thread.sleep(500);

        if (obj == null) {
            System.out.println("obj is null.");
        } else {
            System.out.println("obj is not null.");
        }
    }
}

--output
finalize method excuted.
obj is not null.
第二次 GC.
obj is null.
```

