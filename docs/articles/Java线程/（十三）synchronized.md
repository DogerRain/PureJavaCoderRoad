本文篇幅较长，醋酸菌本人例举了多个demo的代码例子，相信各位静下心看完会有不错的收获。



《深入理解Java虚拟机》一句话：

>当多个线程访问同一个对象时，如果不用考虑这些线程在运行时环境下的调度和交替运行，也不需要进行额外的同步，或者在调用方进行任何其他的协调操作，调用这个对象的行为都可以获取正确的结果，那这个对象是线程安全的。



## 1. 絮

内存分为主内存和工作内存，每个线程都有自己的工作内存，如何和主内存的数据同步，产生的数据不一致性，就是我们常说的线程安全，这就需要我们去了解Java内存模型了。

借用一张图：

![JMM内存模型的抽象结构示意图](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/image-20200722173317229.png)

如图为JMM抽象示意图，线程A和线程B之间要完成通信的话，要经历如下两步：

1. 线程A从主内存中将共享变量读入线程A的工作内存后并进行操作，之后将数据重新写回到主内存中；
2. 线程B从主存中读取最新的共享变量。

## 2. 作用

synchronized 的字面意思，是同步的意思。

在多线程访问某行代码的时候，synchronized可以用来控制线程的同步，简单的说就是控制synchronized代码段不被多个线程同时执行，使其有序执行。

eg：

```java
public class SynchronizedCount extends Thread {
    public static Counter counter = new Counter();
    public Integer flag =1;

    public static class Counter {
        private int count = 0;

        public void addCount() {
            count++;
        }

        public int getCount() {
            return count;
        }
    }

    @Override
    public void run() {
//        synchronized (flag){
            for (int i = 0; i < 100; i++) {
                counter.addCount();
            }
//        }
    }

    public static void main(String[] args) {
        SynchronizedCount[] synchronizedCounts = new SynchronizedCount[100];
        for (int i = 0; i < 100; i++) {
            synchronizedCounts[i] = new SynchronizedCount();
        }
        for (int i = 0; i < 100; i++) {
            synchronizedCounts[i].start();
        }
        try {
            Thread.sleep(1*1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("最后的count值：" + counter.getCount());
    }
}
```

启动100个线程，100个线程同时调用 `counter.addCount();`，也就是一个线程让count自增100，但是我们发现最后的结果是：

```
最后的count值：9753
```

每次的结果基本都不一样。

这就是因为线程对资源的竞争，无法保证数据的一致性。

如果把上面的注释放开，加上`synchronized (flag)`，就能保证这段只能允许一个线程访问：

```
for (int i = 0; i < 100; i++) {
	counter.addCount();
}
```

其他线程只能等待，从而输出最后正确的值10000。



---



下面这部分写的比较复杂。

因为synchronized的用法有好几种。

## 3. 用法

synchronized是Java中的关键字，是一种同步锁。它修饰的对象有以下几种：

| 修饰范围 | 作用范围                                                     |
| -------- | ------------------------------------------------------------ |
| 代码块   | 大括号{}括起来的代码，作用的对象是调用这个代码块的对象       |
| 方法     | 整个方法，作用的对象是调用这个方法的对象                     |
| 静态方法 | 整个静态方法，作用的对象是这个类的所有对象                   |
| 类       | synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象 |

借用一张图更好的理解：

![img](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/2615789-08f16aeac7e0977d.webp)





 为了彻底理解synchronized作用的范围，下面我写了了几个demo，如果你理解了，应该就能彻底明白synchronized该怎么用了。

### 3.1、demo1 —synchronized 普通用法



```java
public class TestDemoSynchronized implements Runnable {
    private Integer y = 0;

    private void setNumber() {
        y++;
    }

    private int getNumber() {
        return y;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            synchronized (y) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                setNumber();
                System.out.println(Thread.currentThread().getName() + " : " + i + "  --->>>" + getNumber());
            }
        }
    }
}

class TestDemo extends Thread {
    TestDemo(Runnable runnable, String name) {
        super(runnable, name);
    }

    public static void main(String[] args) {
        TestDemoSynchronized testDemoSynchronizedfor = new TestDemoSynchronized();
        for (int i = 1; i <= 2; i++) {
            TestDemo testDemofor = new TestDemo(testDemoSynchronizedfor, "Thread" + i);
            testDemofor.start();
        }
    }
}
```


```
Thread1 : 1  --->>>1
Thread2 : 1  --->>>2
Thread1 : 2  --->>>2
Thread2 : 2  --->>>3
Thread1 : 3  --->>>4
Thread2 : 3  --->>>4
Thread1 : 4  --->>>5
Thread2 : 4  --->>>6
Thread1 : 5  --->>>7
Thread2 : 5  --->>>8
```

这里起了两个线程去对`y`进行自增，但是锁住了`y`。

虽然`对象y` 被**synchronized**锁住了，但是 `对象y` (**自增后不是同一个对象了，第一次锁住0，第二次锁住1....**)本身发生了改变。

简而言之，两个线程在进行不同的操作时锁定的不是同一个对象 。（这里并不是原子性问题）所以无法保持数据的一致性。



### 3.2、demo2—加个volatile 

把 上面  demo1 的例子中的

```java
private Integer y = 0; 
```

改成：

```java
private volatile static Integer y = 0;
```
结果：


```
Thread1 : 1  --->>>1
Thread2 : 1  --->>>2
Thread1 : 2  --->>>2
Thread2 : 2  --->>>3
Thread1 : 3  --->>>4
Thread2 : 3  --->>>5
Thread2 : 4  --->>>7
Thread1 : 4  --->>>7
Thread2 : 5  --->>>8
Thread1 : 5  --->>>9
```
因为`volatile`**不能保证原子性**。

导致最后的结果还是错误的。

> 下一篇文章再详细讲一下这个volatile的用法。
>
> 这里先不讲。



### 3.3、demo3 —synchronized（this）和 synchronized（class）
synchronized (this) 锁住this对象，即Object

```java
public class TestDemoSynchronized implements Runnable {
    private Integer y = 0;

    private void setNumber() {
        y++;
    }

    private int getNumber() {
        return y;
    }

    @Override
    public void run() {
        synchronized (this) {
             for (int i = 1; i <= 5; i++) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                setNumber();
                System.out.println(Thread.currentThread().getName() + " : " + i + "  --->>>" + getNumber());
            }
        }
    }
}

class TestDemo extends Thread {
    TestDemo(Runnable runnable, String name) {
        super(runnable, name);
    }

    public static void main(String[] args) {
        TestDemoSynchronized testDemoSynchronizedfor = new TestDemoSynchronized();
        for (int i = 1; i <= 2; i++) {
            TestDemo testDemofor = new TestDemo(testDemoSynchronizedfor, "Thread" + i);
            testDemofor.start();
        }

    }
}
```


结果：

```
Thread2 : 1  --->>>1
Thread2 : 2  --->>>2
Thread2 : 3  --->>>3
Thread2 : 4  --->>>4
Thread2 : 5  --->>>5
Thread1 : 1  --->>>6
Thread1 : 2  --->>>7
Thread1 : 3  --->>>8
Thread1 : 4  --->>>9
Thread1 : 5  --->>>10
```

因为第一个线程进入的时候，会拿到整个对象的锁，执行完5次循环才会释放锁。

Thread2先进入，拿到对象锁 `testDemoSynchronizedfor`，Thread1 发现自己也是 `testDemoSynchronizedfor`，但是被Thread2先进入锁住了，只能等待。

看看`synchronized (TestDemoSynchronized.class)` 会怎么样：
```java
public class TestDemoSynchronized implements Runnable {
    private Integer y = 0;

    private void setNumber() {
        y++;
    }

    private int getNumber() {
        return y;
    }

    @Override
    public void run() {
        synchronized (TestDemoSynchronized.class) {
            for (int i = 1; i <= 5; i++) {
                setNumber();
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " : " + i + "  --->>>" + getNumber());
            }
        }
    }
}

class TestDemo extends Thread {
    TestDemo(Runnable runnable, String name) {
        super(runnable, name);
    }

    public static void main(String[] args) {
        TestDemoSynchronized testDemoSynchronizedfor = new TestDemoSynchronized();
        for (int i = 1; i <= 2; i++) {
            TestDemo testDemofor = new TestDemo(testDemoSynchronizedfor, "Thread" + i);
            testDemofor.start();
        }
    }
}
```
结果：

```
Thread1 : 1  --->>>1
Thread1 : 2  --->>>2
Thread1 : 3  --->>>3
Thread1 : 4  --->>>4
Thread1 : 5  --->>>5
Thread2 : 1  --->>>6
Thread2 : 2  --->>>7
Thread2 : 3  --->>>8
Thread2 : 4  --->>>9
Thread2 : 5  --->>>10
```

拿到的是类**所有对象的锁**，自然也锁住了。
类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。

有人可能会问，为什么y的自增又正确了呢？ 因为线程拿到的是整个对象，setNumber 也在synchronized里面，而且最重要的一点是：**synchronize是 能保证原子性**。（即setNumber()方法的 y++）



注意：这种方法和 `synchronized static` 修饰的方法一样的效果。

 当`synchronized`修饰一个`static`方法时，多线程下，获取的是类锁（即Class本身，注意：不是实例），作用范围是整个静态方法，作用的对象是这个类的所有对象。



接着再看看下面的例子：

### 3.4、demo4—只锁一部分，不锁原子部分


```java
public class TestDemoSynchronized implements Runnable {
    private volatile static Integer y = 0; //volatile也没有用

    private void setNumber() {
        y++;
    }

    private int getNumber() {
        return y;
    }

    @Override
    public void run() {
        setNumber();  //把这个方法放外面
        synchronized (this) {
            for (int i = 1; i <= 5; i++) {
            try {
                Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " : " + i + "  --->>>" + getNumber());
            }
        }
    }
}

class TestDemo extends Thread {
    TestDemo(Runnable runnable, String name) {
        super(runnable, name);
    }

    public static void main(String[] args) {
        TestDemoSynchronized testDemoSynchronizedfor = new TestDemoSynchronized();
        for (int i = 1; i <= 2; i++) {
            TestDemo testDemofor = new TestDemo(testDemoSynchronizedfor, "Thread" + i);
            testDemofor.start();
        }

    }
}

```
结果：

```
Thread2 : 1  --->>>2
Thread2 : 2  --->>>3
Thread2 : 3  --->>>4
Thread2 : 4  --->>>5
Thread2 : 5  --->>>6
Thread1 : 1  --->>>6
Thread1 : 2  --->>>7
Thread1 : 3  --->>>8
Thread1 : 4  --->>>9
Thread1 : 5  --->>>10
```
`setNumber()`不使用锁，当两个线程进入的时候就会异步执行这个方法，**导致y错误**；但是for循环还是加锁的，2个线程只能同步执行。

### 3.5、demo5—synchronized换位置

```java
public class TestDemoSynchronized implements Runnable {
    private volatile static Integer y = 0;

    private void setNumber() {
        y++;
    }

    private int getNumber() {
        return y;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) { //位置换了
            //可能都进入到了这里
            synchronized (this) { //位置换了
                setNumber();
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " : " + i + "  --->>>" + getNumber());
            }
        }
    }
}

class TestDemo extends Thread {
    TestDemo(Runnable runnable, String name) {
        super(runnable, name);
    }

    public static void main(String[] args) {
        TestDemoSynchronized testDemoSynchronizedfor = new TestDemoSynchronized();
        for (int i = 1; i <= 2; i++) {
            TestDemo testDemofor = new TestDemo(testDemoSynchronizedfor, "Thread" + i);
            testDemofor.start();
        }
    }
}
```
结果

```java
Thread1 : 1  --->>>1
Thread1 : 2  --->>>2
Thread1 : 3  --->>>3
Thread1 : 4  --->>>4
Thread2 : 1  --->>>5
Thread2 : 2  --->>>6
Thread2 : 3  --->>>7
Thread2 : 4  --->>>8
Thread2 : 5  --->>>9
Thread1 : 5  --->>>10  //注意
```
同步进入for里面的循环体 { }，但是`synchronized`并没有锁住for，所以在运行的时候，线程是交替执行的，但是y 是正确的。（因为 `setNumber()` 被锁住了）



### 3.6、demo6—综合例子

我们来试一下 `synchronized（this）`、`synchronized（class）`、`synchronized（object）` 这三种情况：

```java
public class TestDemoSynchronized implements Runnable {
    public Integer y = 0;
    public Integer x = new Integer(1);
    public Integer z = 200;
    public Integer k = 100;

    public void setNumber() {
        y++;
    }

    public int getNumber() {
        return y;
    }

    @Override
    public void run() {
        synchronized (TestDemoSynchronized.class) { //能
//        synchronized (this) { //不能
//        synchronized (x) {  //不能，x 在堆， 1 在常量池，两个对象拥有的x 不一样，可以进入
//        synchronized (z) {  //不能 //-128-127 之间，还是同一个对象，否则会Intger创建一个新对象
//        synchronized (k) { //能，x 此时在常量池，常量池是线程共享的，
            for (int i = 1; i <= 5; i++) {
                setNumber();
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " : " + i + "  --->>> " + getNumber());
            }
        }
    }
}

class TestDemo extends Thread {
    TestDemo(Runnable runnable, String name) {
        super(runnable, name);
    }

    public static void main(String[] args) {
        TestDemoSynchronized testDemoSynchronizedfor = new TestDemoSynchronized();
        TestDemoSynchronized testDemoSynchronizedfor2 = new TestDemoSynchronized();
    
        TestDemo testDemo =new TestDemo(testDemoSynchronizedfor,"Thread1");
        TestDemo testDemo2 =new TestDemo(testDemoSynchronizedfor2,"Thread2");

        testDemo.start();
        testDemo2.start();

    }
}
```
输出：

```
Thread1 : 1  --->>> 1
Thread1 : 2  --->>> 2
Thread1 : 3  --->>> 3
Thread1 : 4  --->>> 4
Thread1 : 5  --->>> 5
Thread2 : 1  --->>> 1
Thread2 : 2  --->>> 2
Thread2 : 3  --->>> 3
Thread2 : 4  --->>> 4
Thread2 : 5  --->>> 5
```

注意这个demo是创建 两个**不同的对象**：

- testDemoSynchronizedfor

- testDemoSynchronizedfor2

可以使用demo6 的  ` synchronized (TestDemoSynchronized.class)` 和demo3 比较一下。

如果把 **同步块** 换成    synchronized (this)、synchronized (z)  就不能同步了。

> synchronized (this) 参考 demo3

对于 synchronized (x) 、synchronized (z) 、synchronized (k) 这三者：

 因为这个 Integer在   [-128, 127] 之间时，会拆箱放在常量池，常量池是线程共享的，所以两个不同的**TestDemoSynchronized** 对象去创建 Integer(100) 会先判断常量池是否有100，有就不会创建，直接返回该对象；

如果不在  [-128, 127]，Integer会直接在堆创建一个对象，这时候两个**TestDemoSynchronized**对象就互不影响了。

> 建议大家尝试一下这个demo6，在本地运行测试一下。



## 4、synchronized底层原理

以上并没有讲到`synchronized`的底层原理，因为这个太复杂了，简单的说就是 **基于进入和退出管程(Monitor)对象实现**。

JVM对于**同步方法和同步代码块的处理方式不同**。

- 对于同步方法，JVM采用`ACC_SYNCHRONIZED`标记符来实现同步。 

- 对于同步代码块。JVM采用`monitorenter`、`monitorexit`两个指令来实现同步。

  

在 JDK 1.5 时，synchronized 需要调用监视器锁（Monitor）来实现，监视器锁本质上又是依赖于**底层的操作系统的 Mutex Lock（互斥锁）**实现的，互斥锁在进行释放和获取的时候，需要从**用户态**转换到**内核态**，切换是操作系统做的，这个过程会消耗较长的时间（IO也是如此），这种依赖于操作系统 Mutex Lock 实现的锁我们称之为**重量级锁**。

随着JDK的升级和迭代，synchronized 的优化做的越来越好，其中最大的一次优化就是在`jdk6`的时候，新增了两个锁状态，通过**锁消除、锁粗化、自旋锁**等方法使用各种场景，给synchronized性能带来了很大的提升。

自此，synchronized就有四个状态了：

- 无锁
- 偏向锁（1.6新加）
- 轻量级锁（1.6新加）
- 重量级锁

锁的升级是单向的：**无锁 -> 偏向锁 -> 轻量锁锁 -> 重量级锁**，这个过程称为**锁的膨胀**，不会出现**锁降级**的情况。

锁的膨胀过程就是 synchronized 性能优化的过程，这种策略是为了提高获得锁和释放锁的效率。



无锁大家都知道了，讲一下虚拟机在执行代码的时候，它是如何知道一个方法、方法块 是加了锁的：

**HotSpot 的对象头（Object Header）分为两部分**：

- Mark Word 区域，用于存储对象自身的运行时数据，如哈希码（HashCode）、GC 分带年龄等；
- Klass Word（类指针）用于存储指向方法区对象类型数据的指针，JVM通过这个指针确定对象是哪个类的实例。
- 数组长度， 如果对象是一个数组，那么对象头还需要有额外的空间用于存储数组的长度。

Mark Word 在 32 位系统和在64位的系统中不一样的。

64位的 Mark Word 标记位：

![](https://img-blog.csdnimg.cn/20190111091608949.jpg)

### 偏向锁：

偏向锁（Biased Locking）指的是，它会偏向于第一个访问锁的线程，如果在运行过程中，同步锁只有一个线程访问，不存在多线程争用的情况，**则线程是不需要触发同步的，这种情况下会给线程加一个偏向锁。如果在运行过程中，遇到了其他线程抢占锁，则持有偏向锁的线程会被挂起，JVM会消除它身上的偏向锁，将锁恢复到标准的轻量级锁。**

 JDK 1.6 中默认是开启偏向锁的。如果同一个线程中又对该对象进行加锁操作，我们只要对比对象的线程id是否与线程id相同，如果相同即为线程锁重入问题。加锁和解锁不需要额外的消耗。



### 轻量级锁：

当有另外一个线程，假如为 线程2 竞争获取这个锁时，由于该锁已经是偏向锁，当发现对象头 Mark Word 中的线程 ID 不是自己的线程 ID，就会进行 CAS 操作获取锁，如果获取成功，直接替换 Mark Word 中的线程 ID 为自己的 ID，该锁会保持偏向锁状态；**如果获取锁失败，代表当前锁有一定的竞争，偏向锁将升级为轻量级锁。**



### 重量级锁：

互斥锁(重量级锁)也称为阻塞同步、悲观锁

在轻量级锁的情况下，线程3 也想申请资源，它发现已经是轻量级锁了，获取失败后，它就会自旋，自旋一定的次数还是获取不到，就会膨胀称为重量级锁。

意味着 线程3 只能被挂起阻塞来等待唤醒了。每个对象中都有一个Monitor监视器，而Monitor依赖操作系统的MuteLock（互斥锁）来实现，线程3 被阻塞后便进入内核（Linux）调度状态，这个会导致系统在用户态和内核态来回切换，严重影响锁的性能。

> synchronized关键字并非一开始就该对象加上重量级锁，也是从偏向锁，轻量级锁，再到重量级锁的过程。这个过程也告诉我们，假如我们一开始就知道某个同步代码块的竞争很激烈、很慢的话，那么我们一开始就应该使用重量级锁了，从而省掉一些锁转换的开销。



### 锁优化：

jdk1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁的操作的开销。

这些统统称为 **锁优化** 

### 锁消除：

锁消除可以节省毫无意义的请求锁的时间。有时候我们程序员明明没有使用锁，程序也能保持同步，比如说我们使用了StringBuffer、Vector，但是外层又加了一层 synchronized 同步，就毫无意义了，这时候虚拟机会自动**消除锁**

### 锁粗化：

锁消除是Java虚拟机在JIT编译是，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过锁消除，可以节省毫无意义的请求锁时间

比如说：

```java
for(int i=0;i<size;i++){
    synchronized(lock){
    }
}
```

会粗化成：

```java
synchronized(lock){
    for(int i=0;i<size;i++){
    }
}
```

通过锁消除，可以节省毫无意义的请求锁时间。



---

参考：

- [https://blog.csdn.net/mockingbirds/article/details/51336105](https://blog.csdn.net/mockingbirds/article/details/51336105)

- [https://blog.csdn.net/ya_1249463314/article/details/52571505](https://blog.csdn.net/ya_1249463314/article/details/52571505)

- [https://www.cnblogs.com/hgnulb/p/9942486.html](https://www.cnblogs.com/hgnulb/p/9942486.html)

- [https://juejin.im/post/594a24defe88c2006aa01f1c#heading-0](https://juejin.im/post/594a24defe88c2006aa01f1c#heading-0)

想深入了解`synchronized`原理的可以看一下这里：

- [https://blog.csdn.net/javazejian/article/details/72828483](https://blog.csdn.net/javazejian/article/details/72828483)

- [https://cloud.tencent.com/developer/article/1465413](https://cloud.tencent.com/developer/article/1465413)

Mark Word：

- [https://www.jianshu.com/p/993628f0f4bd](https://www.jianshu.com/p/993628f0f4bd)
- [https://blog.csdn.net/qq_26542493/article/details/90938070](https://blog.csdn.net/qq_26542493/article/details/90938070)