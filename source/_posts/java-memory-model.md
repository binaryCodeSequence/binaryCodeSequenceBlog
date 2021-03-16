---
title: Java 内存模型
categories:
  - Java VM
tags:
  - Java
toc: true
date: 2021-03-16 10:46:05
---

内存模型最好被认为是决定内存不同部分(即堆栈和堆)如何交互的体系结构。Java虚拟机(JVM)将内存划分为两个逻辑单元：

* 线程栈
* 堆

线程堆栈是内存的一部分，用于存储特定于线程的数据，每个线程都有自己的线程堆栈。
线程堆栈存储每个方法的局部变量(包括原始变量和引用变量)，无论该方法是否属于一个对象。
当两个独立的线程调用同一个方法时，每个线程都创建在该方法中声明的局部变量的自己的副本;一个线程不能直接访问另一个线程的变量。


整个Java应用程序只有一个共享堆;它存储创建的对象及其成员变量(包括原语和引用)
任何对堆上对象有引用的线程都可以访问其成员变量，而且，堆上的单个对象可以被不同线程的局部变量引用。






## 参考资料

[Java Memory Model](http://tutorials.jenkov.com/java-concurrency/java-memory-model.html)

[What is the Java memory model?](https://www.educative.io/edpresso/what-is-the-java-memory-model)

