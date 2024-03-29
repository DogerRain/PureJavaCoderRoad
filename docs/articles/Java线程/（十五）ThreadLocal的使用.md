## 什么是ThreadLocal变量

ThreadLocal称为线程本地变量，其为变量在每个线程中都创建了一个副本，每个线程都访问和修改本线程中变量的副本,但每个线程之间的变量是不能相互访问的，ThreadLocal不是一个Thread。

ThreadLocal 有四个方法：

![ThreadLocal方法](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-202109/image-20200727101151586.png)

## ThreadLocal作用

1. ThreadLocal可以让线程独占资源，存储于线程内部，避免线程堵塞造成CPU吞吐下降。

2. 使用同一个threadLocal ，但每个线程的变量是独立都，对其他线程不可见，不需要每个线程都 new 一个对象，减少了内存的开销。

> **在`set`,`get`,`remove`的时候都调用了`expungeStaleEntry`来将所有失效的`Entity`移除**

## ThreadLocal方法详解

demo：

```java
public class ThreadLocalTest {

    public static void main(String[] args) {

        ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

        new Thread(new Runnable() {
            @Override
            public void run() {
                try{
                    threadLocal.set(1);
                    System.out.println(threadLocal.get());
                }finally {
                    threadLocal.remove();
                }
            }
        }, "Thread1").start();


        new Thread(new Runnable() {
            @Override
            public void run() {
                threadLocal.set(2);
                System.out.println(threadLocal.get());
            }
        }, "Thread2").start();


        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(threadLocal.get());
            }
        }, "Thread3").start();
    }

}
```

输出：

```
1
2
null
```

可以看到，每个线程使用都是对同一个threadLocal的引用，但是线程之间的变量是不能互相访问的。



## ThreadLocal如何创建副本的？（如何维护变量的）

在每个Thread中包含一个ThreadLocalMap，ThreadLocalMap的key是ThreadLocal的对象，value是独享数据。

每个Thread 维护一个 `ThreadLocalMap` 映射表，这个映射表的 key 是 `ThreadLocal`实例本身，value 是真正需要存储的 Object。

也就是说 ThreadLocal 本身并不存储值，它只是作为一个 key 来让线程从 ThreadLocalMap 获取 value。

总结：

1、每个Thread维护着一个ThreadLocalMap的引用

2、ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储

3、调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap设置值，key是ThreadLocal对象，值是传递进来的对象

4、调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象

5、**ThreadLocal本身并不存储值**，它只是**作为一个key来让线程从ThreadLocalMap获取value**。





## ThreadLocal内存泄漏问题

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k); //继续深扒，key 是一个弱引用，以一个弱引用指向ThreadLcoal对象k
        value = v;
    }
}
```


ThreadLocalMap 是使用 ThreadLocal 的弱引用作为 Key 的，弱引用的对象在 GC 时会被回收。

ThreadLocalMap使用ThreadLocal的弱引用作为key，如果一个ThreadLocal没有外部强引用来引用它，那么系统 GC 的时候，这个ThreadLocal势必会被回收（为什么会被回收下面讲到），然后ThreadLocalMap中就会出现key为null的Entry，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value 永远无法回收，造成内存泄漏。

以上总结就是，key为null被回收了，但是value还在，需要在set、get的时候判断才能回收。

**那为什么value不能被设置成弱引用呢？**

如果vaule设计为弱引用，你可能获取到的是null ，毫无意义。



## 为什么要使用弱引用而不是强引用？

#### 强引用：

```java
Object object = new Object();
```

强引用用的最多，无论任何情况下，**只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。**

```java
object = null
```

对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式地将相应（强）引用赋值为null，才可以当做垃圾被收集，当然具体回收时机还是要看垃圾收集策略。

而强引用是造成Java内存泄漏的主要原因之一，内存空间不足时，Java虚拟机会抛出OutOfMemoryError ，使程序异常终止，但是注意，抛出错误也不会回收这些强引用的对象。

#### 弱引用：

```java
String str = new String("123");
WeakReference<String> weakReference = new WeakReference<>(str);
str = null;
```

在垃圾回收的一个周期内，**jvm发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。**

所以，弱引用相对强引用来说，生命周期更短。

参考四种引用：[https://www.cnblogs.com/yanl55555/p/13365397.html](https://www.cnblogs.com/yanl55555/p/13365397.html)



| 引用类型 | 被垃圾回收时间 | 用途               | 生存时间          |
| -------- | -------------- | ------------------ | ----------------- |
| 强引用   | 从来不会       | 对象的一般状态     | JVM停止运行时终止 |
| 软引用   | 当内存不足时   | 对象缓存           | 内存不足时终止    |
| 弱引用   | 正常垃圾回收时 | 对象缓存           | 垃圾回收后终止    |
| 虚引用   | 正常垃圾回收时 | 跟踪对象的垃圾回收 | 垃圾回收后终止    |



上个图理解一下：

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-202109/image-20200727183451141.png)

看图，当主线程结束，栈帧销毁，强引用ThreadLocal没有了。再看一下**红色部分**，

如果是**强引用**， 线程的ThreadLocalMap里某个entry的 k 引用还指向这个ThreadLocal对象，这样会导致k指向的ThreadLocal对象以及 v 指向的对象都不能被jvm虚拟机gc回收，造成内存泄漏。

如果是弱引用，就可以是ThreadLocal对象在执行完毕的时候被回收了，因为此时只有entry的k弱引用指向它，ThreadLocal回收后，k 就指向为null了，但是v还是有值，也会有内存泄漏的风险，并不能保证不会内存泄漏。



**那ThreadLocal为什么要使用弱引用而不是强引用呢？**

总结 就是是减少严重内存泄漏的风险。

1. 上面提到，key为弱引用，key为null时，value不为null，导致value无法被回收，引发内存泄漏。
2. 弱引用尚且有内存泄漏的风险，强引用更加。使用线程池的时候，自定义的线程数不规范，若使用强引用，内存泄漏的风险更高。



引自：[谈谈ThreadLocal为什么被设计为弱引用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/304240519)

> 个人也觉得没必要让创建的ThreadLocal对象生命周期过短，ThreadLocal被设计出来本身就是用来跨**方法栈**获取当前线程set的数据或者无锁的获取线程安全的数据，空间换时间。只要让ThreadLocal具有线程的生命周期，就完全没必要使用remove方法，也完全不用担心内存泄漏的问题。

## 如何防止内存泄漏？

上面提到entry的value还会有内存泄漏的风险。

ThreadLocal有通过方法：调用get,set或remove方法时，就会尝试删除key为null的entry，可以释放value对象所占用的内存。

上面demo的正确例子应该是：

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        try{
            threadLocal.set(1);
            System.out.println(threadLocal.get());
        }finally {
            threadLocal.remove(); //用完要remove，防止内存泄漏
        }
    }
}, "Thread1").start();
```

remove方法核心：

```java
 private void remove(ThreadLocal<?> key) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);
            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                if (e.get() == key) {
                    e.clear();
                    expungeStaleEntry(i); //调用消除方法
                    return;
                }
            }
        }
        
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // expunge entry at staleSlot
    tab[staleSlot].value = null;   //把vaule赋值null
    tab[staleSlot] = null; 
    size--;
```

还有set方法：

```java
// If key not found, put new entry in stale slot
tab[staleSlot].value = null;
tab[staleSlot] = new Entry(key, value);
```



---

参考：

- [https://blog.csdn.net/qq_33404395/article/details/82356344](https://blog.csdn.net/qq_33404395/article/details/82356344)

- [https://www.cnblogs.com/shen-qian/p/12108655.html](https://www.cnblogs.com/shen-qian/p/12108655.html)