---
title: Synchronization and Locks
categories:
  - Java 并发
tags:
  - Java
toc: true
date: 2021-02-24 20:44:25
---

如果你刚接触到并发和多线程，那么可以先看一下下面这两篇文章：
- [进程和线程](/2021/02/20/processes-and-threads/)
- [【番外篇】并发&并行](/2021/02/20/concurrency-and-parallel/)

> 本篇笔记只会说一些基础知识和使用，并不涉及到源码分析。


线程之间的通讯非常有效的方式是使用共享对象，但是可能会导致两种错误：**thread interference** 和 **memory consistency errors**。为了防止这些错误可以使用`synchronization`。

然而，`synchronization`又引入线程竞争，当两个或多个线程试图同时访问同一资源时就会出现，并导致Java运行时更缓慢地执行一个或多个线程，甚至暂停其执行。


## Thread Interference

```java
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }
}
```

假设，`线程A`调用`increment()`方法使变量`c`自加1，线程`线程B`调用`decrement()`方法使变量`c`自减1。

但是，表达式`c++`可以分解为三个步骤，`c--`和`c++`一样：

1. 获取`c`的值。
2. 将这个值增加1。
3. 将加1后的值存储到`c`中。

当这两个线程使用的都是同一个`Counter`对象并对`c`进行操作时就会产生**交错**，这种情况就是**线程干扰**，并会导致数据出错。

下面列出了交错执行时的一种顺序：

1. 线程A：获取`c`的值。
2. 线程B：获取`c`的值。
3. 线程A：将这个值增加1。
4. 线程B：将这个值递减1。
5. 线程A：将结果存储在`c`中；`c`现在是`1`。
6. 线程B：将结果存储在`c`中；`c`现在为`-1`。

但是这种交错顺序不是唯一的，没有固定顺序，甚至有时候会出现操作执行正确的情况。

> 简单理解就是：多个线程访问相同数据，操作数据的时候产生了交错；这种情况就是线程干扰。


## Memory Consistency Errors

线程干扰指的是操作数据的时候，而**内存一致性错误**指的是访问数据的时候。

假设，`线程A`调用`increment()`方法使变量`c`自加1，线程`线程B`调用`value()`方法获取变量`c`的值。

`线程B`很有可能获取的值是`0`，这是因为`线程A`修改了值，不能保证对`线程B`是立即可见的。

除了使用同步解决这个问题外，还可以使用`Thread#join`方法。


## 同步方法

Java 编程语言提供了两种基本的同步用法：同步方法和同步代码块。

要使方法同步，只需将`synchronized`关键字添加到其声明中：

```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

如果将`SynchronizedCounter`代替`Counter`，则使这些方法同步具有两个效果：

- 首先，当一个线程正在执行某个同步方法时，会挂起所有调用了这个同步方法的其它线程，直到第一个线程执行完这个方法。这样就解决了代码的**交错**执行。
- 其次，当同步方法退出时，后续调用相同对象的同步方法时，它会自动建立一个`happens-before`关系。这保证了对象状态的更改对所有线程都是可见的。


> 注意，不能将`synchronized`关键字与构造方法一起使用。同步构造方法没有任何意义，因为只有在创建对象时，创建对象的线程才可以访问它，其它情况下只是使用该对象的方法。


同步方法提供了一种防止线程干扰和内存一致性错误的简单策略：如果一个对象对一个以上线程可见，则对该对象变量的所有读取或写入均通过`synchronized`方法完成。


## 同步语句

创建同步代码的另一种方法是使用**同步语句**。与同步方法不同，同步语句必须指定提供**内部锁**的对象：

```java
public void addName(String name) {
    synchronized(this) {
        lastName = name;
        nameCount++;
    }
    nameList.add(name);
}
```

在本例中，`addName`方法需要同步对`lastName`和`nameCount`做修改，但是要尽量避免在同步语句中调用其他对象的方法；因为这有可能会导致死锁。

如果想要在一个静态方法中使用同步语句，则可以使用：

```java
public static void performStaticSyncTask() {
    synchronized(SynchronisedBlocks.class) {
        setStaticCount(getStaticCount() + 1);
    }
}
```


## 重进入

在 Java 中，内部锁本质上都是可重入的。重入就是为了方式线程自身阻塞。


## ReentrantLock(可重入锁)

`ReentrantLock`直接实现了`Lock`接口，和`synchronized`关键字一样都有线程同步机制；但是它们两个的实现方法不一样，并且前者更强大。

个人感觉`ReentrantLock`类是最接近`Lock`接口的实现。

> 如果需要它的高级功能，可以使用，否则还是建议使用`synchronized`。因为这两个的性能一样，`synchronized`可以自动释放锁，还有就是`ReentrantLock`实现类没有自适应旋转。









## 扩展点 Lock 接口

`java.util.concurrent.locks.Lock`接口的用作类似于`synchronized`关键字，它们两个都有线程同步的机制。但是实现方式不一样，并且和`synchronized`相比，`Lock`更加灵活，主要区别如下：

- 非块结构：`synchronized`强制所有获取和释放锁在一个结构块中，而`lock()`和`unlock()`可以在不同的结构块中调用。可以将结构快认为是个代码块。

- 顺序保证：`synchronized`并不会保证让这些等待线程的顺序执行，因为是随机一个线程来获取锁并执行，也就是说它没有**公平性**。但是要注意：公平锁不能保证线程调度的公平性，因为线程调度是操作系统负责。

- 可超时获取锁：没有超时获取锁的功能。`Lock`接口可以通过`tryLock(long timeout，TimeUnit timeUnit)`指定超时时间。

- 可中断获取锁：假设线程A 中调用了`lock.lockInterruptibly()`方法获取锁，当某个线程调用了`线程A.interrupt()`方法时，则`lockInterruptibly()`方法会立即抛出`InterruptedException`异常。`interrupt()`方法就是`Thread#interrupt`方法。

除此之外，`Lock`接口可以通过`newCondition()`创建`Condition`实例。

> 值得注意的是：该接口提供了三种形式的锁获取可中断、不可中断和定时。












## 讨论

**1. 为什么`synchronized`关键字作用在方法上，就不需要提供内部锁对象？**

无论是静态方法还是实例方法，都可以使用`synchronized`关键字。当作用在方法上时，它就会自动获取内部锁对象。

- 静态方法：当前`类的名称.class`
- 实例方法：`this`























## 参考资料

[Synchronization](https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html)

[Java 8 Concurrency Tutorial: Synchronization and Locks](https://winterbe.com/posts/2015/04/30/java8-concurrency-tutorial-synchronized-locks-examples/)

[Guide to the Synchronized Keyword in Java](https://www.baeldung.com/java-synchronized)

[Java Concurrency - Lock Interface](https://www.tutorialspoint.com/java_concurrency/concurrency_lock.htm)

[Synchronized Vs ReentrantLock in Java](https://knpcode.com/java/concurrency/difference-between-synchronized-and-reentrantlock-in-java/)

[Choosing Between Synchronized and ReentrantLock](https://flylib.com/books/en/2.558.1/choosing_between_synchronized_and_reentrantlock.html)

[ReentrantLock可中断锁](https://blog.csdn.net/dongyuxu342719/article/details/94395877)




## 扩展阅读

