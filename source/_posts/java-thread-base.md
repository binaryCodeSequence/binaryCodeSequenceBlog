---
title: Java 线程基础
categories:
  - Java 并发
tags:
  - Java
toc: true
date: 2021-02-20 21:40:01
---

Java 中线程的生命周期和线程的状态可以理解为：在其生命周期中，线程会经历各种状态。

类似于下面这张图是最常见的：

![Thread status](./Life_cycle_of_a_Thread_in_Java.jpg)

使用 `new Thread` 创建出来的线程就是 `NEW` 状态，这个时候线程并没有启动；当调用 `start()` 方法时，线程状态就会变成 `RUNNABLE` 状态。

当 t1 获得锁，t2 会等待 t1 释放锁，这个时候 t2 线程的状态就是 `BLOCKED` 阻塞状态。

当调用了线程的 `Object#wait()`、`Thread#join()` 或 `LockSupport#park()` 方法时，就会变成 `WAITTING` 状态；如果调用的是带有超时参数的就会变成 `TIMED_WAITTING` 状态。

线程执行结束就是 `TERMINATED` 状态。

在 Java 中线程一共就分为 6 种状态并且可以通过 `state()` 方法查看线程状态。


## 创建线程

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Thread run");
    }
});
```

上面是 Java 创建一个线程的例子，下面代码展示了这个线程具体是怎么创建的，`Thread` 构造方法最终都会调用 `init` 方法。

```java
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc,
                  boolean inheritThreadLocals) {
    /* 每一个线程必须有一个名字，默认的线程名称是：Thread- 加上线程ID。 */
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }
    
    this.name = name;

    /* 假设在线程 A 中，调用了 new Thread 创建了线程 B。
       这里获取线程 A 是为了，将线程 A 的一些配置传递给线程 B。*/
    Thread parent = currentThread();
    SecurityManager security = System.getSecurityManager();
    
    /* 每一个线程必须有一个线程组，如果没有指定则会先获取安全管理器中的线程组，否则再回去父级(线程A)线程组。 */
    if (g == null) {

        if (security != null) {
            g = security.getThreadGroup();
        }

        if (g == null) {
            g = parent.getThreadGroup();
        }
    }

    /* 判断当前线程(线程A) 有没有操作线程组的权限，如果没有就会抛出 SecurityException。
       因为 Thread#start() 方法会操作线程组。*/
    g.checkAccess();
    
    if (security != null) {
        /* 主要用来判断子类有没有重写 getContextClassLoader 和 setContextClassLoader 方法，如果重写了则返回 true。*/
        if (isCCLOverridden(getClass())) {
            /* 既然重写了那么就判断一下有没有重写权限(enableContextClassLoaderOverride)，如果没有则抛出 SecurityException。*/
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }
    
    /* 将线程组中的 未启动线程 变量加 1。*/
    g.addUnstarted();
    
    this.group = g;
    this.daemon = parent.isDaemon();
    this.priority = parent.getPriority();
    
    /* 如果重写了就调用子类的 getContextClassLoader() 方法，否则就直接获取父类的 ClassLoader 就可以了。*/
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    
    /* 我也不清楚为什么要获取这个。*/
    this.inheritedAccessControlContext = acc != null ? acc : AccessController.getContext();
    
    this.target = target;
    
    /* 设置线程优先级。*/
    setPriority(priority);
    
    /* 继承父级的 ThreadLocal.ThreadLocalMap。
       当然自己的线程数据可以放到 ThreadLocal.ThreadLocalMap threadLocals 中。*/
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals = ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    
    this.stackSize = stackSize;

    tid = nextThreadID();
}
```

### init 方法中的相关方法

`currentThread()` 是一个本地方法，下面是 JVM 的源码。

```c++

```



这里先挖坑。




























































































## 拓展知识点 Runnable 接口

`Runnable` 这个接口可以理解成线程的任务接口，它里面只有一个无参的 `run` 方法，线程启动后就会执行这个方法中的代码。







































## 讨论

**1. Thread 为什么要实现 Runnable 接口？**

当线程启动后就会执行 `Runnable#run` 方法，这里实现该接口是为了，当没有传递接口实例的时候，也可以保证线程正常运行。

只不过这里的运行只是线程空转一次。



**2. 为什么要传递线程配置？**



**3. 当线程状态是 NEW 的时候，操作系统有没有启动线程？**

首先 Java 线程就是操作系统线程，



**4. 为什么要弃用 stop()、suspend()、resume() 方法？**

当调用 `Thread#stop()` 方法时，就会将线程持有的锁立即释放掉，其它线程所看到的状态就会不一样。



**5. 为什么 wait 在 Object 中？**





## 参考资料

[Life Cycle of a Thread in Java](https://www.baeldung.com/java-thread-lifecycle)


