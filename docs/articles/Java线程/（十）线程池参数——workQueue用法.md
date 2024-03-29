线程池参数的 `workQueue` 决定了缓存任务的排队策略，对于不同的业务场景，我们可以使用不同的排队策略。

我们只需要实现`BlockingQueue` 这个接口即可。

![图片出处找不到了](https://images-1253198264.cos.ap-guangzhou.myqcloud.com/20181026185652119.png)



JDK7 提供了 7 个阻塞队列。分别是：

ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。

LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。

PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。

DelayQueue：一个使用优先级队列实现的无界阻塞队列。

SynchronousQueue：一个不存储元素的阻塞队列。

LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/Home/image-20210525232155784.png)

介绍一下常用的有三种`workQueue` 

## 1. SynchronousQueue

SynchronousQueue**没有容量**，是无缓冲等待队列，是一个**不存储元素的阻塞队列**，会直接将任务交给消费者（即丢给空闲的线程去执行），必须等队列中的添加元素被消费后才能继续添加新的元素，否则会走拒绝策略，所以使用SynchronousQueue阻塞队列一般要求maximumPoolSizes为无界，避免线程拒绝执行操作。

插入元素到队列的线程被阻塞，直到另一个线程从队列中获取了队列中存储的元素。同样，如果线程尝试获取元素并且当前不存在任何元素，则该线程将被阻塞，直到线程将元素插入队列。



## 2. LinkedBlockingQueue

LinkedBlockingQueue如果不指定大小，默认值是 `Integer.MAX_VALUE`

源码在此：

```java
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}
```

```java
@Native public static final int   MAX_VALUE = 0x7fffffff;
```

就是说这个队列里面可以放 2^31-1 = 2147483647 个 任务，也就是无界队列。

所以为了避免队列过大造成机器负载或者内存爆满的情况出现，我们在使用的时候建议手动传一个队列的大小。

与ArrayBlockingQueue不同的是，LinkedBlockingQueue内部分别使用了takeLock 和 putLock 对并发进行控制，也就是说，添加和删除操作并不是互斥操作，可以同时进行，这样也就可以大大提高吞吐量。

与之类似的是 LinkedBlockingDeque。

LinkedBlockingDeque： 使用双向队列实现的双端阻塞队列，双端意味着可以像普通队列一样FIFO(先进先出)，可以像栈一样FILO(先进后出)

## 3. DelayQueue

DelayQueue是一个延迟队列，无界，队列中每个元素都有过期时间，当从队列获取元素时，只有过期元素才会出队列，而队列头部的元素是过期最快的元素。

能够准确的把握任务的执行时间，通常可以使用在：

1. 定时任务调度，比如订单过期未支付自动取消

2. 缓存

   

如何使用，可以参考这篇文章：[https://blog.csdn.net/zhu_tianwei/article/details/53549653](https://blog.csdn.net/zhu_tianwei/article/details/53549653)



---

参考：

- 【细谈Java并发】谈谈LinkedBlockingQueue：[https://blog.csdn.net/tonywu1992/article/details/83419448](https://blog.csdn.net/tonywu1992/article/details/83419448)