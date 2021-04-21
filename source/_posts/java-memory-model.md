---
title: Java 内存模型
categories:
  - Java VM
tags:
  - Java
toc: true
date: 2021-03-16 10:46:05
---

Java 虚拟机内部使用 Java 内存模型将内存划分为两个逻辑单元，**线程栈**和**堆**。

![](./java-memory-model-1.png)

在 Java 虚拟机中，每一个线程都有属于自己的线程栈，而线程栈会存储被调用的方法和这些方法的局部变量(基本类型和引用类型)，假设有下面这样一个方法：

```java
public void method() {
    // 基本类型
    int i = 0;
    // 引用类型
    Test test = new Test();
}
```

当两个独立的线程调用同一个方法时(不管是不是同一个对象的)，**每个线程都会在各自的线程栈中创建局部变量的副本**，对于基本数据类型(boolean、byte、short、char、int、long、float、double) 都是直接创建副本到线程栈中，而对于引用类型( Byte、Integer、Long 等) 它的引用(就是变量名`test`) 是存储在线程栈中，对象本身(就是`Test`这个对象) 是存储在堆中的(就是说对象都是存储在堆上)；并且每个线程只能访问自己的线程栈(就是说对其它线程不可见)。注意：这里说的是每个方法，方法 A 调用方法 B 时，当前线程栈会保存这两个被调用的方法以及这些方法的局部变量。

但是，对于对象的成员变量不管是基本类型还是引用类型，这些成员变量和对象本身(包括静态类)都是存储在堆上的；只要一个线程有对该对象的引用，那么这个线程就能访问堆上的对象，并且也能够访问该对象的成员变量，但每个线程都有自己的局部变量副本。

个人理解是：方法中使用了成员变量，那么就会创建这成员变量的副本存储到线程栈中。

下面这张图说明了上述几点：

![](./java-memory-model-3.png)


有两个线程，它们分别有一组局部变量，其中局部变量 Local Variable 2 都指向堆上的同一个共享对象(Object 3)；这两个线程各自对同一个对象有一个不同的引用，它们的引用都是局部变量，因此会存储在每个线程的线程栈中；并且都可以通过 Object 3 中的成员变量的引用，使这两个线程可以访问 Object 2 和 Object 4。

上图中还展示了一个局部变量，它们分别指向堆上的两个不同对象(Object 1 和 Object 5)，而不是同一个对象；所以，一个线程不能访问 Object 1，另一个线程不能访问 Object 5。

下面的代码描述了上述内存图：

```java
public class MyRunnable implements Runnable() {

    public void run() {
        methodOne();
    }

    public void methodOne() {
        int localVariable1 = 45;

        MySharedObject localVariable2 = MySharedObject.sharedInstance;

        //... do more with local variables.

        methodTwo();
    }

    public void methodTwo() {
        Integer localVariable1 = new Integer(99);

        //... do more with local variable.
    }
}

public class MySharedObject {

    //static variable pointing to instance of MySharedObject

    public static final MySharedObject sharedInstance = new MySharedObject();


    //member variables pointing to two objects on the heap

    public Integer object2 = new Integer(22);
    public Integer object4 = new Integer(44);

    public long member1 = 12345;
    public long member2 = 67890;
}
```

每个执行 methodOne() 的线程都会在各自的线程栈上创建自己的 localVariable1 和 localVariable2 的副本；localVariable1 副本之间将完全分离，只活在每个线程的线程栈上；**一个线程不能看到另一个线程对 localVariable1 的做了什么改变**。

localVariable2 之间也是完全分离的，但是这两个不同副本最终都指向静态变量所引用的对象(堆上的同一个对象)。**静态变量只有一个副本**，这个副本存储在堆上；因此，localVariable2 的两个副本最终都指向静态变量指向的 MySharedObject。MySharedObject 实例也存储在堆上。它对应于上图中的 Object 3。

注意 MySharedObject 类也包含了四个成员变量，这些成员变量和对象一起存储在堆上，其中两个成员变量指向另外两个 Integer 对象，这些 Integer 对象对应上图中的 Object 2 和 Object 4。

在 methodTwo() 方法中创建了一个名为 localVariable1 的局部变量，这个局部变量是对 Integer 对象的引用，localVariable1 引用将在每个执行 methodTwo() 的线程中存储一个副本；但是由于该方法每次执行时都会创建一个新的 Integer 对象，所以 localVariable1 引用指向一个新的 Integer 实例，对应上图中的 Object 1 和 Object 5。


## 硬件内存架构

现代硬件内存架构与 Java 内部的内存模型有一定的区别。了解硬件内存架构也是很重要的，要了解 Java 内存模型是如何与之合作的。

![](./java-memory-model-4.png)

现在的计算机中往往有 2 个或更多的 CPU，这些 CPU 都有多个核心，每个 CPU 在任何时候都能够运行一个线程或同时运行多个线程；这意味着，如果你的 Java 应用程序是多线程的，那么每个 CPU 所执行的线程，可能在你的 Java 应用程序中同时(并发) 运行。


每个 CPU 都包含一组寄存器，这些寄存器基本上是 CPU 内的内存。CPU 在这些寄存器上执行操作的速度比在主内存中的变量上执行操作的速度快得多。这是因为 CPU 访问这些寄存器的速度比访问主内存的速度快得多。

> 计算机中都一个或多个内存(RAM) 也就是主内存区，所有的 CPU 都可以访问主内存，主内存区域通常比 CPU 的缓存存储器大得多。



每个CPU还可能有一个CPU高速缓存存储器层。事实上，大多数现代CPU都有一定规模的高速缓存存储器层。CPU访问其缓存存储器的速度比主存储器快得多，但通常没有访问其内部寄存器的速度快。所以，CPU的高速缓存存储器的速度介于内部寄存器和主存储器的速度之间。有些CPU可能有多个缓存层（Level 1和Level 2），但这并不是那么重要，要了解Java内存模型如何与内存交互。重要的是要知道CPU可以有某种形式的缓存内存层。



通常情况下，当一个 CPU 需要访问主内存时，它将把主内存的一部分数据读到它的 CPU 缓存中。它甚至可能将缓存的一部分读入其内部寄存器中，然后对其进行操作。当 CPU 需要将结果写回主内存时，它会将其内部寄存器中的值冲到缓存中，并在某一时刻将该值冲回主内存。

当CPU需要在缓存存储器中存储其他东西时，缓存存储器中存储的值通常会被冲回主存储器。CPU缓存一次可以有数据写入部分内存，一次可以有部分内存被刷新。它不必每次更新时都要读/写全部缓存。通常，缓存是以较小的内存块更新的，称为 "缓存线"。一条或多条缓存线可以被读入缓存内存，一条或多条缓存线可以再次被刷新回主内存。














## 参考资料

[Java Memory Model](http://tutorials.jenkov.com/java-concurrency/java-memory-model.html)

[What is the Java memory model?](https://www.educative.io/edpresso/what-is-the-java-memory-model)

[Java堆和栈看这篇就够](https://iamjohnnyzhuang.github.io/java/2016/07/12/Java%E5%A0%86%E5%92%8C%E6%A0%88%E7%9C%8B%E8%BF%99%E7%AF%87%E5%B0%B1%E5%A4%9F.html)

