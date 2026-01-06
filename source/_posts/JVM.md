---
title: JVM相关
tags: ['JAVA']
date: 2020/11/1 20:46:25
categories: 'JAVA'
---

## JVM

Java内存区域：

- 本地方法栈(native)
- 虚拟机栈
- 程序计数器
- 方法区（包括运行时常量池）
- 堆

![JVM各版本](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/95SQkNJiHuToDwE.png)

JDK1.8与JDK1.7最大的区别是：**JDK1.8将永久代取消，取而代之的是元空间，在JDK1.8中方法区是由元空间来实现**，所以原来属于方法区的运行时常量池就属于元空间了。

### 本地方法栈

本地方法栈是线程私有的，它内部包括局部变量表，操作数栈、动态链接、出口信息等，他和虚拟机栈的区别是，虚拟机栈是执行JAVA方法的，当时本地方法栈是执行本地方法的，通常不是Java语言编写的方法。 `在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一`

会出现**StackOverFlowError**和**OutOfMemoryError**： Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出OutOfMemoryError异常。

### 虚拟机栈

虚拟机栈也是线程私有的，它内部包括方法的栈帧，每个栈帧由局部变量表，操作数栈、动态链接、出口信息等构成，当执行方法的时候将栈帧压入，等待方法执行完成后将栈帧进行弹出。

### 程序计数器

程序计数器也是线程私有的，，我的理解是，就是指示当前程序执行到了哪里，其具体的作用是：

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

### 方法区

**方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的应该是与 Java 堆区分开来。**

**方法区存着类的信息，常量和静态变量，即类被编译后的数据。这个说法其实是没问题的，只是太笼统了。更加详细一点的说法是方法区里存放着类的版本，字段，方法，接口和常量池。常量池里存储着字面量和符号引用。**

### 运行时常量池

运行时常量池是方法区的一部分。`Class 文件中除了有类的版本、字段、方法、接口等描述信息外`，还有`常量池信息`**（用于存放编译期生成的各种字面量和符号引用）**

**JDK1.7及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。**

![](https://camo.githubusercontent.com/17620721a9f326a235aeec8956949cec03f3f125/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d31342f32363033383433332e6a7067)



### 堆

堆是Java里面最大的一块，堆是由所有线程共享的区域，他里面主要存储的类生成的对象的实例信息**几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是垃圾收集器管理的主要区域，因此也被称作**GC堆（Garbage Collected Heap）**.从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以Java堆还可以细分为：新生代和老年代：再细致一点有：Eden空间、From Survivor、To Survivor空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

![](https://cdn.jsdelivr.net/gh/kklll/Resources@master/files/20201004203612.jpg)

**在 JDK 1.8中移除整个永久代，取而代之的是一个叫元空间（Metaspace）的区域（永久代使用的是JVM的堆内存空间，而元空间使用的是物理内存，直接受到本机的物理内存限制）。**

### 垃圾回收算法

- 标记-清除
- 标记-整理
- 复制算法
- 分代算法

#### 类加载机制

类加载的阶段 加载=》验证=》准备=》解析=》初始化=》使用=》卸载

**加载：**加载类的全限定类名所指的class文件=》生成java.lang.Class的类到内存中

**验证**：主要是对（class文件或者其他的二进制流）进行验证，验证的即是比如cafebabe，版本号是否支持，元数据验证（是否有父类，是否继承自其他类等）、语法语义验证（比如在static 方法中使用了非static值）

**准备**：内存的分配，其中类变量会进入方法区，常量进入静态常量池并进行类变量的初始化

**解析**：虚拟机将常量池内的符号引用替换为直接引用的过程

**什么是字面量和符号引用？直接引用？**

**初始化**：主要包括`<cinit>`和其中类初始化会初始化静态代码块和静态初始化语句。

#### 对象创建的过程

1. 加载Class文件（如果没加载，加载了的话就直接跳到第二步）
2. **分配内存**在堆内存中分配内存
3. 初始化，执行\<init>指令进行初始化
4. 将对象的地址传递给引用。

#### 双亲委派策略

![pic](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/16be8e7a9b686137)



**双亲委派为什么要出现？**

因为它使得类有了层次的划分。就拿`java.lang.Object`来说，你加载它经过一层层委托最终是由`Bootstrap ClassLoader`来加载的，也就是最终都是由`Bootstrap ClassLoader`去找`<JAVA_HOME>\lib`中rt.jar里面的`java.lang.Object`加载到JVM中。

**怎么破坏？**

JAVA开发者开发了ThreadContextClassLoader(线程上下文类加载器)，这个类加载器可以通过java.lang.Thread类中的setContextClassLoader()方法进行设置，如果创建线程的时候没有设置，那么就是默认的应用程序类加载器。

## 锁

**在介绍synchronized关键词之前首先要介绍一下对象头**

### 对象头

Java对象保存在内存中时，由以下三部分组成：

1. **对象头**
2. **实例数据**
3. **填充数据**

2和3我们都很好理解，其中2代表的实例对象的一些值，3是因为JVM要求java的对象占的内存大小应该是8bit的倍数，所以后面有几个字节用于把对象的大小补齐至8bit的倍数。



那么对象头是啥呢？

**对象头由如下组成：**

1. **Mark Word**
2. **指向类的指针**
3. **数组长度（只有数组对象才有）**

Mark Word记录了对象和锁有关的信息，当这个对象被synchronized关键字当成同步锁时，围绕这个锁的一系列操作都和Mark Word有关。

**Mark Word在32位JVM中的长度是32bit，在64位JVM中长度是64bit。**

Mark Word在不同的锁状态下存储的内容不同，在64位JVM中是这么存的：

![image-20201107164941313](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/image-20201107164941313.png)

指向类的指针，这个就不用说了，因为有时候需要用到类的信息，那么通过这个对象指向类的指针就能够访问到类的属性。

需要注意的是数组对象还会额外的多出一个数组长度的字段，用来存储数组对象的长度。

### synchronized关键词

锁的状态总共有四种：无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁（但是锁的升级是单向的，也就是说只能从低到高升级，不会出现锁的降级）。

#### 偏向锁

**1、偏向锁获取过程：**

　　（1）访问Mark Word中偏向锁的标识是否设置成1，锁标志位是否为01——确认为可偏向状态。

　　（2）如果为可偏向状态，则测试线程ID是否指向当前线程，如果是，进入步骤（5），否则进入步骤（3）。

　　（3）如果线程ID并未指向当前线程，则通过CAS操作竞争锁。如果竞争成功，则将Mark Word中线程ID设置为当前线程ID，然后执行（5）；如果竞争失败，执行（4）。

　　（4）如果CAS获取偏向锁失败，则表示有竞争。当到达全局安全点（safepoint）时获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码。

　　（5）执行同步代码。

**2、偏向锁的释放：**

　　偏向锁的撤销在上述第四步骤中有提到**。**偏向锁只有遇到其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁，线程不会主动去释放偏向锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，判断锁对象是否处于被锁定状态，撤销偏向锁后恢复到未锁定（标志位为“01”）或轻量级锁（标志位为“00”）的状态。

**转换关系**：

![img](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/820406-20160424163618101-624122079.png)

#### 轻量级锁

1、**轻量级锁的加锁过程**

　　（1）在代码进入同步块的时候，如果同步对象锁状态为无锁状态（锁标志位为“01”状态，是否为偏向锁为“0”），虚拟机首先将在当前线程的栈帧中建立一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的Mark Word的拷贝，官方称之为 Displaced Mark Word。

　　（2）拷贝对象头中的Mark Word复制到锁记录中。

　　（3）拷贝成功后，虚拟机将使用CAS操作尝试将对象的Mark Word更新为指向Lock Record的指针，并将Lock record里的owner指针指向object mark word。如果更新成功，则执行步骤（3），否则执行步骤（4）。

　　（4）如果这个更新动作成功了，那么这个线程就拥有了该对象的锁，并且对象Mark Word的锁标志位设置为“00”，即表示此对象处于轻量级锁定状态，这时候线程堆栈与对象头的状态如图2.2所示。

　　（5）如果这个更新操作失败了，虚拟机首先会检查对象的Mark Word是否指向当前线程的栈帧，如果是就说明当前线程已经拥有了这个对象的锁，那就可以直接进入同步块继续执行。否则说明多个线程竞争锁，轻量级锁就要膨胀为重量级锁，锁标志的状态值变为“10”，Mark Word中存储的就是指向重量级锁（互斥量）的指针，后面等待锁的线程也要进入阻塞状态。 而当前线程便尝试使用自旋来获取锁，自旋就是为了不让线程阻塞，而采用循环去获取锁的过程。

![锁膨胀的过程](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/image-20201024215715251.png)

**2、轻量级锁的解锁过程：**

　　（1）通过CAS操作尝试把线程中复制的Displaced Mark Word对象替换当前的Mark Word。

　　（2）如果替换成功，整个同步过程就完成了。

　　（3）如果替换失败，说明有其他线程尝试过获取该锁（此时锁已膨胀），那就要在释放锁的同时，将监视器中的owner指向null，之后唤醒被挂起的线程。

引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次CAS原子指令，而偏向锁只需要在置换ThreadID的时候依赖一次CAS原子指令（由于一旦出现多线程竞争的情况就必须撤销偏向锁，所以偏向锁的撤销操作的性能损耗必须小于节省下来的CAS原子指令的性能消耗）。上面说过，轻量级锁是为了在线程交替执行同步块时提高性能，而偏向锁则是在只有一个线程执行同步块时进一步提高性能。

#### 重量级锁

重量级锁是我们常说的传统意义上的锁，其利用操作系统底层的同步机制去实现Java中的线程同步。

重量级锁的状态下，对象的`mark word`为指向一个堆中monitor对象的指针。

一个monitor对象包括这么几个关键字段：cxq（下图中的ContentionList），EntryList ，WaitSet，owner。

其中cxq ，EntryList ，WaitSet都是由ObjectWaiter的链表结构，owner指向持有锁的线程。



![1517900250327](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/16759dd1b0ad4fbe)



当一个线程尝试获得锁时，如果该锁已经被占用，则会将该线程封装成一个ObjectWaiter对象插入到ContentionList的队列尾部，然后暂停当前线程。当持有锁的线程释放锁前，会将ContentionList中的所有元素移动到EntryList中去，并唤醒EntryList的队首线程。

如果一个线程在同步块中调用了`Object#wait`方法，会将该线程对应的ObjectWaiter从EntryList移除并加入到WaitSet中，然后释放锁。当wait的线程被notify之后，会将对应的ObjectWaiter从WaitSet移动到EntryList中。



|    锁    |                             优点                             |                       缺点                       |               适用场景               |
| :------: | :----------------------------------------------------------: | :----------------------------------------------: | :----------------------------------: |
|  偏向锁  | 加锁和解锁不需要额外的消耗，和执行非同步方法比仅存在纳秒级的差距。 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗。 |  适用于只有一个线程访问同步块场景。  |
| 轻量级锁 |          竞争的线程不会阻塞，提高了程序的响应速度。          |  如果始终得不到锁竞争的线程使用自旋会消耗CPU。   | 追求响应时间。同步块执行速度非常快。 |
| 重量级锁 |              线程竞争不使用自旋，不会消耗CPU。               |             线程阻塞，响应时间缓慢。             |   追求吞吐量。同步块执行速度较长。   |



自己实现一个ThreadPool(简单的)

```java
package com.kklll.leetcode.face.meituan;

import lombok.extern.slf4j.Slf4j;

import java.util.ArrayDeque;
import java.util.Deque;
import java.util.HashSet;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author DeepBlue
 * @Date 2020/10/27 10:59
 */

@Slf4j(topic = "test")
class TestMyThreadPool {
    public static void main(String[] args) {
        MyThreadPool myThreadPool = new MyThreadPool(2, 1000, TimeUnit.MILLISECONDS, 10);
        for (int i = 0; i < 5; i++) {
            int j = i;
            myThreadPool.exec(() -> {
                log.debug("{}", j);
            });
        }
    }
}

@Slf4j(topic = "test")
public class MyThreadPool {
    //任务队列
    private BlockingQueue<Runnable> taskQueue;
    //线程集合
    private HashSet<Worker> workers = new HashSet<>();

    //核心线程数
    private int coreSize;

    //获取任务的超时时间
    private long timeout;

    private TimeUnit timeUnit;

    public MyThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueCapacity) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue<>(queueCapacity);
    }


    public void exec(Runnable task) {
        //当任务没有超过coreSize那么worker执行
        //否则放入BlockingQueue
        synchronized (this) {
            if (workers.size() < coreSize) {
                Worker worker = new Worker(task);
                log.debug("新增worker{}，{}", worker, task);
                workers.add(worker);
                worker.start();
            } else {
                log.debug("加入任务队列{}", task);
                taskQueue.put(task);
            }
        }
    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            //当task没任务，那么执行任务
            //当没任务的时候去找阻塞队列的任务执行
//            while (task != null || (task = taskQueue.take()) != null) {
            while (task != null || (task = taskQueue.poll(timeout,timeUnit)) != null) {
                try {
                    log.debug("正在执行{}", task);
                    task.run();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    task = null;
                }
            }
            synchronized (workers) {
                log.debug("worker被移除{}", this);
                workers.remove(this);
            }
        }
    }
}

class BlockingQueue<T> {
    //任务队列
    private Deque<T> queue = new ArrayDeque<>();

    //锁
    Lock lock = new ReentrantLock();

    //容量
    private int capacity;

    //生产者条件，消费者条件
    private Condition fullWait = lock.newCondition();
    private Condition emptyWait = lock.newCondition();

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    //带timeout的阻塞获取
    public T poll(long timeout, TimeUnit timeUnit) {
        lock.lock();
        try {
            //转换成ns
            long l = timeUnit.toNanos(timeout);
            while (queue.isEmpty()) {
                try {
                    //返回剩余时间
                    if (l <= 0) {
                        return null;
                    }
                    l = emptyWait.awaitNanos(l);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeLast();
            fullWait.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    //获取
    public T take() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                try {
                    emptyWait.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeLast();
            fullWait.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    //添加
    public void put(T t) {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                try {
                    fullWait.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            queue.addFirst(t);
            emptyWait.signal();
        } finally {
            lock.unlock();
        }
    }

    public int size() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }
}
```

增加拒绝策略后

```java
package com.kklll.leetcode.face.meituan;

import lombok.extern.slf4j.Slf4j;

import java.util.ArrayDeque;
import java.util.Deque;
import java.util.HashSet;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author DeepBlue
 * @Date 2020/10/27 10:59
 */

@Slf4j(topic = "test")
class TestMyThreadPool {

    public static void main(String[] args) {
        MyThreadPool myThreadPool = new MyThreadPool(1, 1000, TimeUnit.MILLISECONDS, 1,
                (queue, task) -> {
//            1.等待超时
//            queue.offer(task,500,TimeUnit.MILLISECONDS);
//            2.直接放弃
//            log.debug("放弃任务{}",task);
//            3.抛出异常
//            throw new RuntimeException("任务失败了");
//            4.自己去执行吧
//            task.run();
                });

        for (int i = 0; i < 3; i++) {
            int j = i;
            myThreadPool.exec(() -> {
                log.debug("{}", j);
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
    }
}

@FunctionalInterface
interface RejectPolicy<T> {
    void reject(BlockingQueue<T> blockingQueue, T t);
}

@Slf4j(topic = "test")
public class MyThreadPool {
    //任务队列
    private BlockingQueue<Runnable> taskQueue;
    //线程集合
    private HashSet<Worker> workers = new HashSet<>();

    //核心线程数
    private int coreSize;

    //获取任务的超时时间
    private long timeout;

    private TimeUnit timeUnit;

    //拒绝策略
    private RejectPolicy<Runnable> rejectPolicy;

    public MyThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueCapacity, RejectPolicy<Runnable> rejectPolicy) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue<>(queueCapacity);
        this.rejectPolicy = rejectPolicy;
    }


    public void exec(Runnable task) {
        //当任务没有超过coreSize那么worker执行
        //否则放入BlockingQueue
        synchronized (this) {
            if (workers.size() < coreSize) {
                Worker worker = new Worker(task);
                log.debug("新增worker{}，{}", worker, task);
                workers.add(worker);
                worker.start();
            } else {
                log.debug("加入任务队列{}", task);
                /*
                1.死等
                2.超时等待
                3.放弃任务执行
                4.抛出异常
                5.让调用者自己执行
                 */
//                taskQueue.put(task);
                taskQueue.tryPut(rejectPolicy, task);
            }
        }
    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            //当task没任务，那么执行任务
            //当没任务的时候去找阻塞队列的任务执行
//            while (task != null || (task = taskQueue.take()) != null) {
            while (task != null || (task = taskQueue.poll(timeout, timeUnit)) != null) {
                try {
                    log.debug("正在执行{}", task);
                    task.run();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    task = null;
                }
            }
            synchronized (workers) {
                log.debug("worker被移除{}", this);
                workers.remove(this);
            }
        }
    }
}

@Slf4j
class BlockingQueue<T> {
    //任务队列
    private Deque<T> queue = new ArrayDeque<>();

    //锁
    Lock lock = new ReentrantLock();

    //容量
    private int capacity;

    //生产者条件，消费者条件
    private Condition fullWait = lock.newCondition();
    private Condition emptyWait = lock.newCondition();

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }

    //带timeout的阻塞获取
    public T poll(long timeout, TimeUnit timeUnit) {
        lock.lock();
        try {
            //转换成ns
            long l = timeUnit.toNanos(timeout);
            while (queue.isEmpty()) {
                try {
                    //返回剩余时间
                    if (l <= 0) {
                        return null;
                    }
                    l = emptyWait.awaitNanos(l);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeLast();
            fullWait.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    public void tryPut(RejectPolicy<T> rejectPolicy, T task) {
        lock.lock();
        try {
            //如果队列满了
            if (queue.size() == capacity) {
                rejectPolicy.reject(this, task);
            } else {
                log.debug("加入任务队列{}", task);
                queue.addFirst(task);
                emptyWait.signal();
            }
        } finally {
            lock.unlock();
        }
    }

    //获取
    public T take() {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                try {
                    emptyWait.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeLast();
            fullWait.signal();
            return t;
        } finally {
            lock.unlock();
        }
    }

    //添加
    public void put(T t) {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                try {
                    log.debug("任务队列满了等待------------");
                    fullWait.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            log.debug("加入任务队列{}", t);
            queue.addFirst(t);
            emptyWait.signal();
        } finally {
            lock.unlock();
        }
    }

    //带超时添加的方法
    public boolean offer(T t, long timeout, TimeUnit timeUnit) {
        lock.lock();
        try {
            long l = timeUnit.toNanos(timeout);
            while (queue.size() == capacity) {
                try {
                    log.debug("任务队列满了等待------------");
                    if (l <= 0) {
                        log.debug("进行拒绝{}", t);
                        return false;
                    }
                    l = fullWait.awaitNanos(l);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            log.debug("加入任务队列{}", t);
            queue.addFirst(t);
            emptyWait.signal();
            return true;
        } finally {
            lock.unlock();
        }
    }

    public int size() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }
}
```

#### 阻塞队列的种类

##### **ArrayBlockingQueue**

ArrayBlockingQueue 是一个有界的阻塞队列，其内部实现是将对象放到一个数组里。有界也就意味着，它不能够存储无限多数量的元素。它有一个同一时间能够存储元素数量的上限。你可以在对其初始化的时候设定这个上限，但之后就无法对这个上限进行修改了

##### **DelayQueue**

DelayQueue 对元素进行持有直到一个特定的延迟到期。注入其中的元素必须实现 java.util.concurrent.Delayed 接口。

##### **LinkedBlockingQueue**

LinkedBlockingQueue 内部以一个链式结构(链接节点)对其元素进行存储。如果需要的话，这一链式结构可以选择一个上限。如果没有定义上限，将使用 Integer.MAX_VALUE 作为上限。

**PriorityBlockingQueue**

PriorityBlockingQueue 是一个无界的并发队列。它使用了和类 java.util.PriorityQueue 一样的排序规则。你无法向这个队列中插入 null 值。所有插入到 PriorityBlockingQueue 的元素必须实现 java.lang.Comparable 接口。因此该队列中元素的排序就取决于你自己的 Comparable 实现。

##### **SynchronousQueue**

SynchronousQueue 是一个特殊的队列，它的内部同时只能够容纳单个元素。如果该队列已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走。同样，如果该队列为空，试图向队列中抽取一个元素的线程将会阻塞，直到另一个线程向队列中插入了一条新的元素。据此，把这个类称作一个队列显然是夸大其词了。它更多像是一个汇合点。

#### 线程池的四种拒绝策略

1、直接丢弃（DiscardPolicy）

2、丢弃队列中最老的任务(DiscardOldestPolicy)。

3、抛异常(AbortPolicy)

4、将任务分给调用线程来执行(CallerRunsPolicy)

#### 线程池的种类

**1.newFixedThreadPool**，固定大小的线程池，没有应急线程吗，全部为核心线程，拒绝策略为AbortPolicy，阻塞队列为LinkedBlockingQueue，且没有指定阻塞队列的最大值，所以为2^31-1，阻塞队列可能会OOM。

**2.newCachedThreadPool**，缓存线程池，没有核心线程，全部为应急线程，拒绝策略是AbortPolicy。60S内没有新的任务，应急线程会被放掉，阻塞队列为SynchronousQueue，这个阻塞队列它的内部同时只能够容纳单个元素，已有一元素的话，试图向队列中插入一个新元素的线程将会阻塞，直到另一个线程将该元素从队列中抽走**一手交钱，一手交货**

**3.newSingleThreadExecutor**,单例线程池，只有一个核心线程，没有应急线程，阻塞队列为LinkedBlockingQueue，没有指定大小，界限为Integer.MAX_VALUE，拒绝策略为AbortPolicy。

#### AQS

**`AbstractQueuedSynchronizer`抽象队列同步器**，简称为`AQS`，可用于构建**阻塞锁**或者其他相关**同步器**的基础框，是Java并发包的基础工具类。通过`AQS`这个框架可以对**同步状态原子性管理、线程的阻塞和解除阻塞、队列的管理**进行统一管理。
`AQS`是抽象类，并不能直接实例化，当需要使用`AQS`的时候需要继承`AQS`抽象类并且重写指定的方法，这些重写方法包括**线程获取资源和释放资源的方式**(如ReentractLock通过分别重写线程获取和释放资源的方式实现了公平锁和非公平锁)，**同时子类还需要负责共享变量state的维护，如当state为0时表示该锁没有被占，大于0时候代表该锁被一个或多个线程占领(重入锁)**，而队列的维护(获取资源失败入队、线程唤醒、线程的状态等)不需要我们考虑，`AQS`已经帮我们实现好了。`AQS`的这种设计模式采用的正是**模板方法模式**。
**总结起来子类的任务有：**

1. 通过`CAS`操作维护共享变量`state`。
2. 重写资源的获取方式。
3. 重写资源释放的方式。

**AQS的数据结构：**

1. state状态，表示当前同步器是否有线程占用。

2. Node，即队列中的元素。（有如下属性：）

   | 方法和属性值 | 含义                              |
   | ------------ | --------------------------------- |
   | waitStatus   | 当前节点在队列中的状态            |
   | thread       | 表示处于该节点的线程              |
   | prev         | 前驱指针                          |
   | predecessor  | 返回前驱节点，没有的话抛出npe     |
   | nextWaiter   | 指向下一个处于CONDITION状态的节点 |
   | next         | 后继指针                          |

   其中waitState的取值及含义：

   | 取值      | 含义                                           |
   | :-------- | :--------------------------------------------- |
   | 0         | 当一个Node被初始化的时候的默认值               |
   | CANCELLED | 为1，表示线程获取锁的请求已经取消了            |
   | CONDITION | 为-2，表示节点在等待队列中，节点线程等待唤醒   |
   | PROPAGATE | 为-3，当前线程处在SHARED情况下，该字段才会使用 |
   | SIGNAL    | 为-1，表示线程已经准备好了，就等资源释放了     |

3. 条件队列（单向链表）等待某个特定条件发生的队列。

4. 同步队列（双向链表）等待当前资源释放后运行的线程组成的队列。

`AQS`内部数据结构为一个**双向链表**和一个**单向链表**，双链表为同步队列，队列中的每个节点对应一个`Node`内部类，`AQS`通过控制链表的节点而达到阻塞、同步的目的，单链表为条件队列，**可以把同步队列和条件队列理解成储存等待状态的线程的队列**，但是条件队列中的线程并不能直接去获取资源，而要先从条件队列转到同步队列中排队获取，同步队列的唤醒结果是线程去尝试获取锁，而条件队列的唤醒结果是把线程从条件队列移到同步队列中，一个线程要么是在同步队列中，要么是在条件队列中，不可能同时存在这两个队列里面。

![](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/image-20201029112731331.png)



![image-20201029113032248](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/image-20201029113032248.png)

**ReentrantLock的lock方法流程**

**公平锁：**

![img](https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/b8b53a70984668bc68653efe9531573e78636.png)	

1. 首先CAS更改同步状态state变量，如果成功，那么将当前同步器的独占线程拥有者换成自己，否则执行同步器的acquire方法。
2. 进入同步器的acquire方法后会尝试进行加锁（tryacquire方法），如果加锁成功，将当前同步器的独占线程拥有者换成自己。否则进行下一步。
3. 调用nofiretryacquire方法，此时还会进行CAS尝试获取锁，如果获取成功的话，也是将当前线程设置成独占线程的持有者，否则进行下一步。
4. 将当前线程包装成一个Node，然后使用CAS将其加入到同步队列中（其实创建了两个节点，第一个节点叫dummy节点，无实际作用，这其中还要把dummy节点的state置成-1，表示其有义务唤醒后继节点的义务）。

**非公平锁：**

可以看出公平锁和非公平锁之间的区别在于：**公平锁不会直接进去就使用CAS改变同步器的state状态**而是调用公平锁的tryAcquire方法，且方法内会判断当前的同步队列中是不是有正在等待资源的线程，如果有的话将当前线程添加到同步队列的末尾。

**ReentrantLock的unlock方法流程**

1. unlock方法调用同步器的release方法，可以看出其中的大概逻辑，首先先获取同步器的state，将其减去1（一般情况下，让然你也可以自己实现），然后判断state是不是等于0，如果等于零的话，证明该锁要被释放掉。否则，返回一个false

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

### 读写锁

读写锁的特性：

1. **读读可以并发执行，读写不能并发。**
2. **读锁不支持条件变量**
3. **重入不能升级（即同一线程先获得读锁，后获得写锁是不支持的）**
4. **重入时可以降级（即同一线程先获取写锁，再获取读锁是支持的）**

**读写锁的底层实现：**

**读写锁使用的是一个同步器而不是两个**，其中同步器内部的state标志位有所不同，其中的state分为第十六位和高十六位，其中读锁在高十六位，写锁占用的是低十六位。（0表示为加读锁，1表示加了锁，大于1为重入状态的计数），这种设计能够大大提高效率，因为只去访问一次内存就能拿到读写锁的所有mark word，并且使用CAS的时候只用一次就行了。

<img src="https://cdn.jsdelivr.net/gh/kklll/Resources@master/pics/16e3e72e34ef7d2f" alt="读写锁" style="zoom:80%;" />

#### 写锁的lock方法

```java
//首先调用了同步器的acquire方法
public void lock() {
    sync.acquire(1);
}
//acquire中的逻辑和reentrantLock基本相同。
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
protected final boolean tryAcquire(int acquires) {
    /*
             * Walkthrough:
             * 1. If read count nonzero or write count nonzero
             *    and owner is a different thread, fail.
             * 2. If count would saturate, fail. (This can only
             *    happen if count is already nonzero.)
             * 3. Otherwise, this thread is eligible for lock if
             *    it is either a reentrant acquire or
             *    queue policy allows it. If so, update state
             *    and set owner.
             */
    Thread current = Thread.currentThread();
    //获取state值
    int c = getState();
    //查看写锁部分的值
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        //判断写锁是不是等于0，如果等于0说明读锁不为零，那么需要阻塞。或者是当前线程不是当前AQS的持有者
        //那么也需要阻塞
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        //如果上面不满足，判断是不是重入，如果是冲入而且超过最大冲入次数，那么报错
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

- 在tryAcquire()方法中，先通过`exclusiveCount()`方法来计算写锁的数量，怎么计算的呢？就是将state和`0x0000FFFF`进行`与运算`。

- 然后判断state是否等于0，如果等于0，就表示读锁和写锁都没有被获取，当前线程就调用`writerShouldBlock()`方法判断线程是否需要等待，如果需要等待，tryAcquire()方法就返回false，表示获取锁失败，那么就会回到AQS的acquire()方法中，后面的逻辑与排他锁的逻辑一样。如果不需要等待，就尝试去修改state的值，如果修改成功，就表示获取锁成功，否则失败。

- 如果state不等于0，那么就表示存在读锁或者写锁，那么究竟是读锁还是写锁呢？就需要根据w的值进行判断了。

- 如果w为0，说明写锁的数量为0，而此时又因为c不等于0，说明锁被占用，但是不是写锁，那么此时锁的状态一定是读锁，既然是读锁状态，那么写锁此时来获取锁时，就肯定失败，因为读锁存在时，是不能去获取写锁的。因此当w等于0时，tryAcquire()方法返回false。

- 如果w不为0，说明此时锁的状态是写锁，接着进行`current != getExclusiveOwnerThread()`判断，判断持有锁的线程是否是当前线程。如果不是当前线程，那么tryAcquire()返回false；如果是当前线程，那么就进行后面的逻辑。为什么是当前线程持有锁，就能执行后面的逻辑呢？ 因为读写锁是支持重入的。

- 如果是当前线程获取的写锁，接着就判断，再次对写锁进行重入时，会不会超出写锁的最大重入次数，如果是，就抛出异常。（因为state的低16位表示写锁，所以写锁最大可被重入的次数是![2^{16}](https://juejin.im/equation?tex=2%5E%7B16%7D)-1）。

#### 读锁的lock方法

```java
public void lock() {
    //调用同步器的acquireShared方法
    sync.acquireShared(1);
}
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
protected final int tryAcquireShared(int unused) {
    /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
    Thread current = Thread.currentThread();
    int c = getState();
    //如果有人加了写锁，且当前线程不是加写锁的线程，返回-1
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    //获取锁的数值
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        // 读锁数量为0时，就将当前线程设置为firstReader，firstReaderHoldCount=1
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        // 读锁数量不为0且firstReader(第一次获取读的线程)为当前线程，就将firstReaderHoldCount累加
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            // 读锁数量不为0，且第一个获取到读锁的线程不是当前线程
            // 下面这一段逻辑就是保存当前线程获取读锁的次数，如何保存的呢？
            // 通过ThreadLocal来实现的，readHolds就是一个ThreadLocal的实例
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}
```

tryAcquireShared方法，返回值是int类型的，**如果返回-1，表示获取读锁失败，返回0表示加锁成功，且不需要唤醒后续节点，如果返回正数表明，需要唤醒几个后继节点，在读写锁中只会出现前两种情况**

- 在tryAcquireShared()方法中，会先通过`exclusiveCount()`方法来计算写锁的数量，如果写锁存在，再判断持有写锁的线程是不是当前线程，如果不是当前线程，就表示写锁被其他线程给占用，此时当前线程不能获取读锁。tryAcquireShared()方法返回-1，表示获取读锁失败。如果写锁不存在或者持有写锁的线程是当前线程，那么就表示当前线程有机会获取到读锁。

- 接下里会判断当前线程获取读锁是否不需要排队，读锁数量是否会超过最大值，以及通过CAS修改读锁的状态是否成功（将state的值加 1<<16）。如果这三个条件成立，就进入if语句块中，这一块的代码比较繁琐，但是功能比较单一，就是统计读锁的数量以及当前线程对读锁的重入次数，底层原理就是`ThreadLocal`。因为在读写锁中提供了`getReadLockCount()、getReadHoldCount()`等方法，这几个方法的数据就来自这儿。

#### 写锁的unlock方法

```java
public void unlock() {
    sync.release(1);
}
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    //写锁减一
    int nextc = getState() - releases;
    //如果减成0了，那么证明可以释放锁了
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```

写锁的释放与排他锁的释放逻辑也几乎一样。当调用writeLock.unlock()时，先调用到AQS的release()方法，在release()方法中会先调用子类的tryRelease()方法。在这里调用的是ReentrantReadWriteLock的内部类Sync的tryRelease()方法。

#### 读锁的unlock方法

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
        // 将修改同步变量的值（读锁状态减去1<<16）
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```

在tryReleaseShared()方法中，会先修改和读锁计数有关的数据，然后在for的死循环中，通过CAS操作将state的值减去1<<16。如果CAS操作成功，才会从for循环中退出。当读锁数量为0时，tryReleaseShared()返回true，表示锁被完全释放。

