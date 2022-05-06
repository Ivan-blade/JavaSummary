### 多线程并发实战

#### 线程基础见base笔记多线程部分，以下内容来自多线程并发实战

#### 并发编程的挑战

##### 上下文切换

 + 简述
   + 即使单核处理器也能支持多线程执行代码，cpu通过给每个线程分配cpu时间片来实现这个机制。时间片是cpu分配给各个线程的时间，因为时间片非常短，所以cpu通过不停的切换线程执行，让我们感觉有多个线程同时执行，时间片一般是几十毫秒。
   + cpu通过时间片分配算法来循环执行任务，当前任务执行一个时间片之后会切换到下一个任务，但是切换前会保存上一个任务的状态，一边下次切换回这个任务时，可以再加载这个任务的状态。所以任务从保存到加载的过程就是一次上下文切换
   + 这个过程就像我们看书时，发现某个字不认识，打开字典去查一样，但是去查之前必须先记住目前看的书浏览到了多少页多少行，等登查完字典后能够返回阅读这本书。这样的切换是会影响读书效率的，同样上下文切换也会影响多线程的执行效率。

###### 多线程一定快吗

 + 代码及测试结果

   ```java
   package ClassTest.thread.test;
   
   /**
    * @author hylu.ivan
    * @date 2022/5/4 下午1:56
    * @description
    */
   public class ConcurrencyTest {
   
       /**
        * 更改循环次数测试并行和串行的效率，以下用例对应结果
        * 并行消耗时间 串行消耗时间
        *    1          0
        *    2          2
        *    6          8
        *    10         15
        *    62         114
        */
   //    private static final long count = 10001;
   //    private static final long count = 100001;
   //    private static final long count = 1000001;
   //    private static final long count = 10000001;
       private static final long count = 100000001;
   
       public static void main(String[] args) {
           concurrency();
           serial();
       }
   
       /**
        * 并发
        */
       public static void concurrency() {
           long start = System.currentTimeMillis();
           Thread thread = new Thread(new Runnable() {
               @Override
               public void run() {
                   int a  = 0;
                   for (int i = 0; i < count; i++) {
                       a += 5;
                   }
               }
           });
           thread.start();
           int b  = 0;
           for (int i = 0; i < count; i++) {
               b--;
           }
   
           long time = System.currentTimeMillis() - start;
           try {
               thread.join();
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           System.out.println("concurrent: " + time + "ms,b = " +b);
       }
   
   
       /**
        * 串行
        */
       public static void serial() {
           long start = System.currentTimeMillis();
           int a = 0;
           for (int i = 0; i < count; i++) {
               a += 5;
           }
           int b = 0;
           for (int i = 0; i < count; i++) {
               b--;
           }
           long time = System.currentTimeMillis() - start;
           System.out.println("serival: " + time + "ms , b = "+b+",a = " + a);
       }
   }
   
   ```

+ 结果分析

  + 根据测试结果，并发执行累加低于百万时，速度甚至会比串行操作慢，这就是并发操作时上下文切换对于效率产生了一定影响

###### 测试上限文切换次数和时长

+ 测试工具
  + lmbench3
  + vmstat命令
+ Vmstat测试结果
  + 其中cs（Content Switch）是指上下文，每秒切换一千多次（书中情况，本地测试就9次。。。）

###### 如何减少上线文切换

+ 简述
  + 减少上下文切换的方法：无锁并发编程，CAS算法，使用最少线程和使用协程
  + 无锁并发编程：多线程竞争锁时会引起上下文切换，所以多线程处理数据，可以用一些方法来避免使用锁，如将数据的id按照hash算法取模分段，不同线程处理不同端的数据
  + CAS算法：java的Atomic包使用CAS算法更新数据，而不需要加锁
  + 使用最少的线程：避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，会导致大量线程处于等待状态
  + 协程：在单线程里实现多任务调度，并在单线程里维持多个任务间的切换

###### 减少上下文切换实战

+ 策略以及原理：

  + 减少waiting线程可以减少上下文切换次数，因为每一次waiting到runnable都会进行一次上下文切换。

+ 实战步骤（书中实例）

  + 使用jstack命令dump线程信息，查看pid为31177的进程中线程都在做什么

    ```shell
    sudu -u admin jstack 31177 > /home/tengfei.fangtf/dump17
    ```

  + 统计所有线程处于什么状态，发现300多个线程处于waiting状态（onobject-monitor）状态

    ```shell
    grep java.lang.Thread.State dump17 | awk '{print $2$3$4$5}' | sort | uniq -c
    ```

  + 打开dump文件查看处于waiting的线程在做什么，发现都是在JBOSS的工作线程，在await。说明jboss线程池接收到的任务太少，大量线程闲置

  + 打开jboss线程池配置信息，将maxThreads降低，重启JBoss

##### 死锁

+ 基本概念

  + 产生死锁的四个必要条件：
    （1） 互斥条件：一个资源每次只能被一个进程使用。
    （2） 占有且等待：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
    （3）不可强行占有:进程已获得的资源，在末使用完之前，不能强行剥夺。
    （4） 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

  + 这四个条件是死锁的必要条件，只要系统发生死锁，这些条件必然成立，而只要上述条件之一不满足，就不会发生死锁。

  + 示例代码

    ```java
    package ClassTest.thread.test;
    
    /**
     * @author hylu.ivan
     * @date 2022/5/4 下午2:59
     * @description
     */
    public class DeadLockDemo {
    
        private static String A = "A";
        private static String B = "B";
    
        public static void main(String[] args) {
            new DeadLockDemo().deadLock();
        }
    
        private void deadLock() {
            Thread thread1 = new Thread(new Runnable() {
                @Override
                public void run() {
                    synchronized (A) {
                        try {
                            Thread.currentThread().sleep(2000);
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                        synchronized(B) {
                            System.out.println("1");
                        }
                    }
                }
            });
    
            Thread thread2 = new Thread(new Runnable() {
                @Override
                public void run() {
                    synchronized(B) {
                        synchronized(A) {
                            System.out.println("2");
                        }
                    }
                }
            });
            thread1.start();
            thread2.start();
        }
    }
    ```

  + 上述代码只是演示了一个场景，现实业务中更容易遇到的情况是t1拿到锁之后由于异常情况没有能够释放锁，导致业务停止。如果碰到这种情况只能通过dump线程查看哪个线程出现问题。本地执行以上代码后查看死锁方式

    ```shell
    # 查看java程序pid
    jps
    # jstack pid 信息如下，显示28行和39行引起了死锁
    Java stack information for the threads listed above:
    ===================================================
    "Thread-0":
            at ClassTest.thread.test.DeadLockDemo$1.run(DeadLockDemo.java:28)
            - waiting to lock <0x000000070fd64c68> (a java.lang.String)
            - locked <0x000000070fd64c38> (a java.lang.String)
            at java.lang.Thread.run(java.base@11.0.12/Thread.java:834)
    "Thread-1":
            at ClassTest.thread.test.DeadLockDemo$2.run(DeadLockDemo.java:39)
            - waiting to lock <0x000000070fd64c38> (a java.lang.String)
            - locked <0x000000070fd64c68> (a java.lang.String)
            at java.lang.Thread.run(java.base@11.0.12/Thread.java:834)
    
    Found 1 deadlock.
    ```

  + 避免死锁的几个方法

    + 避免一个线程同时获取多个锁
    + 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源
    + 尝试使用定时锁，使用lock.tryLock(timeout)来替代使用内部锁机制
    + 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况

##### 资源限制的挑战

+ 什么是资源限制
  + 资源限制是指在进行并发编程时，程序的执行速度受限于计算机硬件资源或软件资源。例如，服务器的带宽只有2Mb/s,某个资源的下载速度是1Mb/s，系统启动十个线程下载资源，下载速度并不会变成10Mb/s，所以在进行并发编程时，要考虑这些资源的限制，硬件资源限制有带宽的上传/下载速度，硬盘读写速度和CPU的处理速度，软件资源的限制有数据库连接数和socket连接数等等。
+ 资源限制引发的问题
  + 在并发编程中，将代码执行速度加快的原则是将代码中的串行部分编程并发执行，但是如果将某段代码并发执行，因为受限于资源，仍然在串行执行，这时候程序不仅不会加快执行，反而会更慢，因为增加了上下文切换和资源调度时间。例如一个使用多线程在办公网络并发下载和数据数据的程序让cou利用率达到了100%，几小时都不能完成任务，改为单线程一小时就完成了处理任务
+ 如何解决资源限制问题
  + 对于硬件资源限制，可以使用集群并行执行程序，例如使用odps，hadoop或者自己搭建服务器集群，不同的机器处理不同的数据，可以通过数据ID%机器数，计算获取一个机器编号，然后由对应编号的机器处理不同的数据。对于软件资源的限制，可以考虑使用资源池将资源复用，比如使用连接池将数据库和socket连接复用，或者调用对象webservice接口获取数据时，只建立一个连接。
+ 在资源限制情况下进行并发编程
  + 在资源限制的情况下想要进行并发编程让程序更快需要根据资源限制跳帧程序的并发度。比如涉及下载文件程序时，需要考虑带宽和硬盘读写速度；涉及数据库操作时，需要考虑数据库连接数，如果线程数远大于数据库连接数，某些线程会被阻塞，等待数据库连接。

##### 小结

​	本章中介绍了进行并发编程时可能会碰到的挑战和对应解决方案，对于java开发工程师而言，尽量使用jdk并发包提供的并发容器和工具类解决问题，因为这些类以及经过充分的测试和优化，本章提到的问题都已经被充分解决。

#### java并发机制的底层实现

+ 简述
  + java代码在编译之后会变成java字节码，字节码被类加载器加载到jvm里，jvm执行字节码，最终需要转化为汇编指令在cpu上执行，java中所使用的并发机制依赖与jvm的实现和cpu指令。本章我们将深入底层一起探索java并发机制的底层实现原理。

##### volatile的应用

+ 简述

  + 在多线程并发编程中synchronized和volatile都扮演着重要的角色，volatile是轻量级的synchronized，它在多处理器开发中保证了共享变量的可见性。可见性的意思是当一个线程修改一个共享变量时，另一个线程能读到这个修改的值。如果voatile变量修饰符使用恰当的话，它比synchronized的使用和执行成本更低，因为它不会引起线程上下文的切换和调度。

+ volatile的定义与实现原理

  + java编程语言允许线程间访问共享变量，为了确保共享变量能被准确和一致地更新，线程应该确保通过排他锁单独获得这个变量。java语言提供volatile,在某些情况下比锁要更方便，如果一个字段被声明成volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。

  + 前置cpu术语

    | 术语       | 英文单词               | 术语描述                                                     |
    | ---------- | ---------------------- | ------------------------------------------------------------ |
    | 内存屏障   | memory barriers        | 是一组处理器指令，用于实现对于内存操作的顺序限制             |
    | 缓冲行     | cache line             | 缓存中可以分配的最小存储单位，处理器填写缓存线时会加载整个缓存线，需要使用多个主内存读周期 |
    | 原子操作   | atomic operations      | 不可中断的一个或者一系列操作                                 |
    | 缓存行填充 | cache line fill        | 当处理器识别到从内存中读取操作数是可缓存的，处理器读取整个缓存行到适当的缓存（L1,L2,L3的或所有） |
    | 缓存命中   | cache hit              | 如果进行高速缓存行填充操作的内存位置仍然是下次处理器访问的地址时，处理器从缓存中读取操作数，而不是从内存读取 |
    | 写命中     | write hit              | 当处理器将操作数写回到一个内存缓存的区域时，它首先会检查这个缓存的内存地址是否在缓存行中，如果存在一个有效的缓存行，则处理器将这个操作数写回到缓存，而不是写回到内存，这个操作被成为写命中 |
    | 写缺失     | write misses the cache | 一个有效的缓存行被写入到不存在的内存区域                     |

    + volatile是如何保证可见性的？

      + 以x86处理器下通过工具获取JIT编译器生成的汇编指令来查看对volatile进行写操作时，CPU操作为例

      + java代码

        ```java
        instance = new SingleTon(); // instance是volatile变量	
        ```

      + 转变为汇编代码

        ```txt
        0x01a3deld: movb $0x0,0x1104800(%esi);0x01a3de24:lock addl $0x0,(%esp)
        ```

      + 有volatile变量修饰的共享变量进行写操作时会多出第二行汇编代码，即带有Lock的编码

      + 通过查阅IA-32架构软件开发者手册可知，Lock前缀的指令在多核处理器下会引发两件事。

        + 将当前处理器缓存行的数据写回到系统内存
        + 这个写回内存的操作会在其他CPU里缓存了该地址的数据无效

      + 为了提高处理速度，处理器不直接和内存进行通信，而是将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，JVM就会向处理器发送一个Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存式一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置为无效状态，当处理器对这个数据进行修改操作时，会重新从系统内存中把数据读到处理器缓存里。

    + volatile的两条实现原则

      + **lock前缀指令会引起处理器缓存回写到内存。**Lock前缀指令导致在执行指令期间，盛宴处理器的Lock#信号。在多处理器环境中，Lock#信号确保在声言该信号期间，处理器可以独占任何共享内存。但是在最近的处理器里，LOCK#信号一般不锁总线，而是锁缓存，毕竟锁总线的开销比较大。在8.1.4中会详细说明锁定操作对处理器缓存的影响，对于Intel486和Pentium处理器，在锁操作时，总是在总线上声言Lock#信号。但在P6和目前的处理器中，如果访问的内存区域已经缓存在处理器内部，则不会声言Lock#信号。相反，它会锁定这块区域的缓存并回写到内存，并使用缓存一致性机制来确保修改的原子性，此操作被成为缓存锁定，缓存一致性机制会组织同时修改由两个以上处理器缓存的内存区域数据。
      + **一个处理器的缓存回写到内存会导致其他处理器的缓存无效**。

##### synchronized的实现原理与应用

+ java对象头
+ 锁的升级与对比

##### 原子操作的实现原理与应用
##### 小结

#### java内存模型

##### 重排序

##### 顺序一致性

##### volatile的内存语义

##### 锁的内存语义

##### final域的内存语义

##### happens-before

##### 双重检查锁定与延迟初始化

##### java内存模型综述

#### java并发编程基础

##### 线程简介

##### 启动和终止线程

##### 线程间通信

##### 线程应用实例

#### java中的锁

#### java并发容器与框架

#### java中的13个原子操作类

#### java中的并发工具类

#### java中的线程池

#### Executor框架

#### java并发编程实践

