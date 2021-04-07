## Queue介绍

Queue 是我们常说的队列，队列是一种**先进先出**的数据结构，元素在队列末尾添加，在队列头部删除。

Queue接口扩展自Collection，并提供插入、提取、检验等操作。



## Queue 接口的API

![](https://cdn.jsdelivr.net/gh/DogerRain/image@main/img-20210401/image-20210407113029806.png)



add()、offer()表示向队列添加一个元素

poll()与remove()方法都是移除队列头部的元素并返回

> 两者的区别在于如果队列为空，那么poll()返回的是null，而remove()会抛出一个NoSuchElementException异常。

element()与peek()主要是获取头部元素，不删除。

> 可以看到同一个功能有两个方法

## Queue 接口的实现类

Queue 接口的实现类的实现类比较多。常见的有：

- LinkedList
- PriorityQueue
- CustomLinkedQueue
- DelayQueue

LinkedList支持在两端插入和删除元素，因为LinkedList类实现了Deque接口，所以通常我们可以使用LinkedList来创建一个队列。

> LinkedList底层是双向链表的结构，得益于实现了Deque

PriorityQueue类实现了一个优先队列，优先队列中元素被赋予优先级，拥有高优先级的先被删除。

```java
public class QueueTest {
    public static void main(String[] args) {
        Queue<String> queue = new LinkedList<String>();
        queue.add("1");
        queue.add("2");
        queue.add("3");
        queue.add("3");
        queue.offer("0");
        while (queue.size()>0){
            System.out.print(queue.remove()+" ");
        }
    }
}
```

控制台打印：

```
1 2 3 3 0 
```

