先来个死锁的例子：

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author HaC
 * @date 2021/2/5
 * @Description  ReentrantLock 死锁的例子
 */
public class ReentrantLockDeadLock {
    static Lock lock1 = new ReentrantLock();
    static Lock lock2 = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new DeadLockDemo(lock1, lock2), "Thread1");
        Thread thread2 = new Thread(new DeadLockDemo(lock2, lock1), "Thread2");
        thread1.start();
        thread2.start();
    }

    static class DeadLockDemo implements Runnable {
        Lock lockA;
        Lock lockB;

        public DeadLockDemo(Lock lockA, Lock lockB) {
            this.lockA = lockA;
            this.lockB = lockB;
        }

        @Override
        public void run() {
            try {
                lockA.lock();
                System.out.println(Thread.currentThread().getName() + "\t 自己持有：" + lockA + "\t 尝试获得：" + lockB);
                TimeUnit.SECONDS.sleep(2);
                lockB.lock();
                System.out.println(Thread.currentThread().getName() + "\t 自己持有：" + lockB + "\t 尝试获得：" + lockA);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lockA.unlock();
                lockB.unlock();
                System.out.println(Thread.currentThread().getName() + "正常结束!");
            }
        }
    }
}
```



## 1、jstack

首先使用 `jps` 命令列出当前的Java进程：

![](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824103325691.png)

找到疑似死锁的例子，找到PID，执行 `jstack -l 20148`

往下找，会显示一段 deadlock 的关键字：

![](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824112216025.png)

即可定位到死锁的类和行数。

## 2、jconsole

jconsole 位于 JDK 的 bin 目录，双击即可运行。

如下，选择需要建立连接的进程。

![](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824101138022.png)

切换到 **线程**，再点击下方的 **检测死锁** ，即可查看死锁的情况：

![](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824112452610.png)

除此之外，jconsole 还可以查看堆内存、CPU、线程数 等其他信息。

## 3、jvisualvm

jvisualvm 也在 JDK 的 bin 目录。

选择本地的进程，上方切换至 **线程** ，再点击一下 **线程Dump** 即可。

![](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824113514128.png)

点击后可以看到线程的状态日志，可以看到死锁的信息：

![](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824113643717.png)

## 4、jmc

同样位于 JDK 的 bin 目录。

打开你需要监测的进程：

![](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824114429643.png)

![image-20210824114540025](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824114540025.png)

下方切换到 **线程**

![](F:\笔记\PureJavaCoderRoad（Java基础教程）\docs\articles\开发辅助工具\picture\image-20210824135124607.png)