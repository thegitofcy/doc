# 1. Atomic

## AtomicInteger

高并发的情况下, `i++` 无法保证原子性, 往往会出现问题, 所以引入 `AtomicInteger`.



### 代码演示

```java
public class AtomicIntegerExample {

    private static final int THREADS_COUNT = 2;

    private static int count = 0;
    private static volatile int volatileCount = 0;
    private static AtomicInteger atomicInteger = new AtomicInteger(0);
    private static CountDownLatch countDownLatch = new CountDownLatch(2);

    private static void increase() {
        count ++;
        volatileCount ++;
        atomicInteger.incrementAndGet();
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newFixedThreadPool(THREADS_COUNT);
        for (int i = 0; i < 2; i++) {
            executorService.execute(new Runnable() {
                public void run() {
                    for (int j = 0; j < 1000; j++) {
                        increase();
                    }
                    countDownLatch.countDown();
                }
            });
        }
        countDownLatch.await();
        executorService.shutdown();

        System.out.println("count: " + count);
        System.out.println("volatileCount: " + volatileCount);
        System.out.println("atomicInteger: " + atomicInteger);
    }
}
```



## AtomicLong/LongAdder

```java
@Slf4j
@ThreadSafe
public class AtomicLongExample {

    private static final int clientTotal = 1000;
    private static final int threadTotal = 50;
    private static AtomicLong count = new AtomicLong(0l);

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                } catch (InterruptedException e) {
                    log.error("exception: {}", e);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        log.info("count: {}", count.get());
    }

    private static void add() {
        count.incrementAndGet();
    }
}
```



```java
@Slf4j
@ThreadSafe
public class LongAdderExample {

    private static final int clientTotal = 1000;
    private static final int threadTotal = 50;
    private static LongAdder longAdder = new LongAdder();

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                } catch (InterruptedException e) {
                    log.error("exception: {}", e);
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        log.info("count: {}", longAdder);
    }

    private static void add() {
        longAdder.increment();
    }
}
```

### AtomicLong 和 LongAdder 的不同:

大致意思如下: (不太准确, 后期需要补充.)

首先, AtomicLong 由于 CAS 的原因, 在一个 while 中完成的操作, 在高并发的时候, 效率会比较低, LongAdder 在 AtomicLong 的基础上, 通过数组, 对数据进行分散, 在高并发的时候提高效率.



## AtomicReference<>

```java
@Slf4j
@ThreadSafe
public class AtomicReferenceExample {

    private static AtomicReference<Integer> count = new AtomicReference<>(0);

    public static void main(String[] args) {
      	// 当值是第一个参数的时候, 就将值更新为第二个参数
        count.compareAndSet(0, 1);
        count.compareAndSet(1, 2);
        count.compareAndSet(2, 3);
        count.compareAndSet(3, 4);
        log.info("count: {}", count);   // 4
    }
}
```



## AtomicXXXFieldUpdater

```java
/**
 * @program: thread-part1
 * @description: AtomicXXXFieldUpdater 原子性更新某一个类的一个实例
 *  当前类是使用  AtomicIntegerFieldUpdater 更新 AtomicXXXFieldUpdaterExample 的 count 属性.
 * @author: cy
 */
@Slf4j
@ThreadSafe
public class AtomicXXXFieldUpdaterExample {

    private static AtomicIntegerFieldUpdater<AtomicXXXFieldUpdaterExample> updater = AtomicIntegerFieldUpdater.newUpdater(AtomicXXXFieldUpdaterExample.class, "count");

    /** count 必须使用 volatile 进行修饰, 并且不是 static 修饰的.*/
    @Getter
    public volatile int count = 100;

    private static AtomicXXXFieldUpdaterExample atomicXXXFieldUpdaterExample = new AtomicXXXFieldUpdaterExample();

    public static void main(String[] args) {
        if (updater.compareAndSet(atomicXXXFieldUpdaterExample, 100, 120)) {
            log.info("update success 1, count: {}", atomicXXXFieldUpdaterExample.getCount());
        } else {
            log.info("update success 1, count: {}", atomicXXXFieldUpdaterExample.getCount());
        }

        if (updater.compareAndSet(atomicXXXFieldUpdaterExample, 100, 150)) {
            log.info("update success 2, count: {}", atomicXXXFieldUpdaterExample.getCount());
        } else {
            log.info("update success 2, count: {}", atomicXXXFieldUpdaterExample.getCount());
        }
    }
}
```





# CountDownLatch

### 背景

- CountDownLatch 是在 1.5 引入的, 一起引入的工具类还有 CyclicBarrier, Semaphore, ConcurrentHashMap, BlockingQueue.
- 存在于 java.util.concurrent 包下.



### 概念

- 作用: CountDownLatch 是一个多线程控制工具类, 使一个线程等待其他线程各自执行完毕后再执行. 换句话说就是使一个线程进入阻塞状态(调用 `countDownLatch.await()`的线程), 达到一定条件后(调用 `countDownLatch.countDown()` 使计数器减为 0)才可以继续执行.
- 实现原理: 通过一个 ==**计数器**== 来实现的. 计数器的初始值就是线程的数量. 每当一个线程执行完毕后, 计数器的值就减 1, 当计数器的值为 0 的时候,就表示所有线程都执行完毕了, 然后在闭锁上等待的线程就可以恢复工作了.
- 通常和 Semaphore 一起和线程池一起使用

### 源码

- CountDownLatch 只提供了一个构造方法.

```java
// count 为线程数量
public CountDownLatch(int count) {}
```

- 有三个比较重要的方法

```java
// 调用 await() 的线程会被挂起, 它会等待到 count 的值为 0 才继续运行
public void await() throws InterruptedException {}

// 和 await() 类似, 只不过等待 timeout 时间后, count 值还没变为 0 的话, 线程也会继续运行
public boolean await(long timeout, TimeUnit unit) throws InterruptedException {}

// 将 count 值减 1
public void countDown() {}
```



### 代码演示

```java
public class CountDownLatchExample implements Runnable {

    private static final int THREAD_COUNT = 100;
    private CountDownLatch countDownLatch;

    public CountDownLatchExample(CountDownLatch countDownLatch) {
        this.countDownLatch = countDownLatch;
    }

    public void run() {
        try {
            synchronized (countDownLatch) {
                countDownLatch.countDown();
                System.out.println("thread counts = " + countDownLatch.getCount());
            }
            countDownLatch.await();
            System.out.println("concurrency counts = " + (100 - countDownLatch.getCount()));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        CountDownLatch countDownLatch = new CountDownLatch(THREAD_COUNT);
        for (int i = 0; i < 100; i++) {
            executorService.execute(new CountDownLatchExample(countDownLatch));
        }
    }
}
```



# **CyclicBarrier**

### 背景

- CyclicBarrier 是在 1.5 引入的, 一起引入的工具类还有 CountDownLatch, Semaphore, ConcurrentHashMap, BlockingQueue.
- 存在于 java.util.concurrent 包下.



### 概念

- 字面意思是 ==**循环栅栏**==, 大概的意思是一个可循环利用的屏障
- 作用: 是线程的一种同步机制, 相当于一个栅栏, 所有线程必须等待, 直到指定数量线程到达, 线程才会继续执行下去. 也就是说会让所有线程都等待完成后才进行下一步行动.
- 例子: 比如朋友们一起去吃饭, 有的会早到, 有的会晚到, 但是餐厅规定只有等待所有人都到齐后才会让人们一起进去. 这里的朋友们就是各个线程, 餐厅就是 CyclicBarrier.



### 源码

- 提供了两个构造方法

```java
// parties 是参与线程的个数
public CyclicBarrier(int parties) { this(parties, null); }

// barrierAction是最后一个达到线程要做的任务
public CyclicBarrier(int parties, Runnable barrierAction) {}
```



- 比较重要的方法

```java
// 线程调用 await() 表示自己已经达到栅栏, 进入等待状态. 
public int await() throws InterruptedException, BrokenBarrierException

// 除了指定线程个数之外, 还可以通过指定超时时间. 当超时时间达到时, 无论是否有指定个数的线程, 都会释放.
public int await(long timeout, TimeUnit unit) throws InterruptedException, BrokenBarrierException, TimeoutException
  
// 重置 CyclicBarrier
public void reset() {}
```



### 等待线程释放的条件

- 最后一个线程到达
- 线程被其他线程中断.
- 等待的其他线程被中断.
- 其中一个等待线程超时时间到.
- CyclicBarrier 被重置.

### 代码演示

```java
public class CyclicBarrierExample implements Runnable {

    private static final int THREAD_COUNT = 5;

    private CyclicBarrier cyclicBarrier;

    public CyclicBarrierExample(CyclicBarrier cyclicBarrier) {
        this.cyclicBarrier = cyclicBarrier;
    }

    public void run() {
        try {
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " 达到栅栏 A");
            cyclicBarrier.await();  // 运行到这里后, 会进行等待, 等待 THREAD_COUNT 个线程都达到后, 才会继续往下执行
            System.out.println(Thread.currentThread().getName() + " 冲破栅栏 A");
            Thread.sleep(2000);
            System.out.println(Thread.currentThread().getName() + " 达到栅栏 B");
            cyclicBarrier.await();  // 运行到这里后, 会进行等待, 等待 THREAD_COUNT 个线程都达到后, 才会继续往下执行
            System.out.println(Thread.currentThread().getName() + " 冲破栅栏 B");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(THREAD_COUNT, new Runnable() {
            // 在 await()后, 最后一个到达的线程会执行这个方法
            public void run() {
                System.out.println(Thread.currentThread().getName() + " 完成最后任务");
            }
        });

        for (int i = 0; i < THREAD_COUNT; i++) {
            new Thread(new CyclicBarrierExample(cyclicBarrier)).start();
        }
    }
}
```



# Semaphore

### 背景

### 概念

- 通常和 CountDownLatch 一起和线程池一起使用





# BlockingQueue: 阻塞队列

<img src="/Users/cy/develop/doc/local/picture/JUC/image-20200908150454244.png" alt="image-20200908150454244" style="zoom:33%;" />



# CAS

## 1. 什么是 CAS

- CAS: compare and swap, 即比较再转换.

- juc 包下的类使用 CAS 算法实现了区别于 Synchronized 同步锁的一种==**乐观锁**==. 

## 2. CAS 算法理解

CAS 是一种==**无锁算法**==,有 3 个操作数, ==**内存值 V, 旧的预期值 A, 要修改的新值 B**==, ==**当且仅当预期值 A 和内存值 V 相等时, 将内存值修改为 B, 否则什么也不做**==.



==**就是对主内存的数据和工作内存数据副本进行比较, 如果相等, 说明没有被修改, 那么就进行更新, 如果不相等, 则说明已经被修改了, 就不作操作了!**==



==**CAS 伪代码:**==

```java 
do{
  备份旧数据;
  基于旧数据构建新数据
} while(内存地址, 备份的旧数据, 新数据)
```

<img src="/Users/cy/develop/doc/local/picture/并发与高并发/image-20200907210112135.png" alt="image-20200907210112135" style="zoom:50%;" />

如上图:

1. t1 和 t2线程同时去访问同一个变量 56, 他们会把主内存中的值拷贝一份到工作内存, 所以 t1 和 t2 的预期值都是 56.
2. 加入 t1 先抢到了资源, 那么 t1 就能去更新值, 其他线程都抢占失败(抢占失败的线程并没有挂起, 而是尝试继续抢占), 此时 t1 会拿预期值和内存中的值比较, 发现都是 56, ==相等, 就更新值==, 并且会主内存, 此时主内存中的值是 57.
3. 这时候 t2 抢到了, 此时 t2 的预期值是 56, 但是内存中的值是 57, ==不相等, 就操作失败==了, 就啥也并不做了.

用比较通俗的话说: ==CPU 去更新一个值, 如果想修改的值不是原来的值, 则操作失败.==

就是说两者比较, 如果相等,说明没有被修改过, 则替换成新值, 然后继续运行, 如果不相等, 说明共享数据已经被修改了, 那么就放弃已做的操作.



## 3. CAS 的 ABA 问题.

### CAS 的 ABA 问题解释

ABA 问题是指在 CAS 操作的时候, 其他线程将变量的值修改为了 B, 但是因为一些原因, 有修改为了 A. 于是, CAS就将变量的值做了交换操作, 结果和预期肯定不符合.

例如:

1. 小明去取钱, 当前卡上有 200 块钱. 比如他要取 100
2. 取款机出了点问题, 小明点了 2 次, 发起了 2 个线程, 这两个线程都是期望获取 200, 然后更新为 100.
3. 加入线程 1 首先执行成功,将余额更新为了 100, 线程 2 由于某些原因阻塞了, 在线程 2阻塞的过程中, 小明的爸爸给小明又汇了 100, 这时卡上就有 200 的余额.
4. 这个时候线程 2 恢复运行, 阻塞之前获取到的是 200, 线程 2 就拿着这个 200和当前实际的值就行比较, 然后就把余额更新为 100 了.



### CAS 的 ABA 问题解决方案:

使用==**版本号**==, 比如时间戳, 就是说在 CAS 操作的时候, compare 不仅要比较期望的值和地址中的值是否一致,还要比较变量的版本号是否一致. 每次成功更新的时候, 版本号加 1.



### AtomicStampedReference

AtomicStampedReference 类就实现了用版本号







# AQS

PS: AQS 通过==**模板模式**==实现.

## 1. AQS 同步器

AQS(AbstractQueuedSynchronizer), 队列同步器, 也叫同步器, 是用来==构建其他锁或者同步组件的基础框架==. 是 Java提供的一个底层同步工具类, 用一个 int 类型的变量表示同步状态, 并通过一系列的 CAS 操作来管理这个同步状态. AQS 的主要作用是为 java 中的并发同步组件提供统一的底层支持, 例如 ReentrantLock, CountDownLathc 等都是基于 AQS 实现的, 用法是通过继承 AQS 来实现其模板方法, 然后将其子类作为其内部类.

### 1. AQS 有两大核心

1. volatile 修饰的 int 类型的 ==state==, 用来表示同步状态.
2. ==FIFO== 等待队列, 是一个双向链表. 当线程争抢资源被阻塞的时候, 这个线程就会被添加到链表的尾部. 首节点是成功获取锁的线程节点.



图示如下:

![image-20200908205925529](/Users/cy/develop/doc/local/picture/JUC/image-20200908205925529.png)



- FIFO 主要用来存放在锁上阻塞的线程.

- 当一个线程尝试获取锁的时候, 如果已经被占用, 则此线程会进入阻塞状态, 然后 AQS 会把当前线程以及等待状态等构建一个 Node 节点, 添加到 FIFO 的尾部.

- FIFO 的首节点是成功获取锁的节点, 当首节点释放锁的时候, 会唤醒后边的 FIFO 中的阻塞 Node,

  



### 2. AQS 资源共享方式

AQS 定义了两种资源共享方式:

- Exclusive: 独占式, 每次只能有一个线程持有锁, 其他线程都要等着. 比如 ==ReentrantLock==
- Share: 共享式,允许多个线程同时持有锁, 并发访问共享资源. 比如 ==ReentrantReadWriteLock==



### 3. AQS 的提供的方法

`boolean isHeldExclusively(): 该线程是否正在独占资源`

`boolean tryAcquire(int arg): 尝试以独占的方式获取资源`

`boolean tryRelease(int arg): 尝试以独占的方式释放资源`

`int tryAcquireShared(int arg): 尝试以共享的方式获取资源`

`int tryReleaseShared(int arg): 尝试以共享的方式释放资源`

## 2. 基于 AQS 的锁



### 1. ReentrantLock(重入锁)

ReentrantLock: 可重入锁, 获得锁后, 再次获取该锁不需要阻塞, 而是直接关联一次计数器增加重入次数



原理剖析:

```properties
1: state 初始化为 0.
2: A 线程 lock()时, 会调用 tryAcquire() 独占该锁, 如果成功, 就会将 state+1.
3: A 线程独占后, 其他线程再调用 tryAcquire()就会失败, 直到 A 线程 unLock()到 state=0 为止.其他线程才有机会获取到锁.(也即是说, lock() 和 unLock() 必须成对出现.)
4: 释放锁之前, A 线程是可以重复的获取此锁的, 每次重新获取锁, state 都会加 1, 这就是重入锁的概念. 但是, 重复多少次就要 unLock() 多少次, 一直到 state = 0 才能释放锁.
```



```java
@Slf4j
@ThreadSafe
public class ReentrantLockExample {
    private static int count = 0;

  	/** 声明 ReentrantLock.*/
    static Lock lock = new ReentrantLock();

    private static final int clientTotal = 1000;
    private static final int threadTotal = 50;

    private static void inc() {
        lock.lock();
        lock.lock();
        lock.lock();
        lock.lock();
        lock.lock();

        try {
            count++;
        } catch (Exception e) {
            log.error("example: {}", e);
        } finally {
            lock.unlock();
            lock.unlock();
            lock.unlock();
            lock.unlock();
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService = Executors.newCachedThreadPool();
        final Semaphore semaphore = new Semaphore(threadTotal);
        final CountDownLatch countDownLatch = new CountDownLatch(clientTotal);
        for (int i = 0; i < clientTotal; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    inc();
                    semaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        log.info("count: {}", count);
    }
}
```



### 2. LockSupport

LockSupport 是基于线程的锁, 比 ReentrantLock等更加底层. 是可中断的锁.(Thread.interrupt)



重要的方法:

`static void park(): 静态方法, 让当前线程进入阻塞状态`

`static void unpark(Thread): 静态方法, 唤醒指定的线程.如果在unpark 的时候, 线程未被阻塞, 那么 unpark 后的第一个 park 将不会阻塞线程.`



**演示: 通过其他线程唤醒阻塞的线程**

```java
@Slf4j
public class LockSupportExample {

    public static void main(String[] args) throws Exception {
//        bySynchronized();
        byLockSupport();
    }

    /** 基于 wait/notify, 使用 synchronized 实现.*/
    private static void bySynchronized() throws InterruptedException {
        Object lock = new Object();
        Thread thread = new Thread(() -> {
            int sum = 0;
            synchronized (lock) {
                for (int i = 0; i < 100; i++) {
                    sum += i;
                }
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.info("sum: {}", sum);
            }
        });

        thread.start();
        TimeUnit.SECONDS.sleep(2);
        log.info("其他线程唤醒计数线程...");
        synchronized (lock) {
            lock.notify();
        }
    }

    /** 基于 LockSupport 实现. */
    private static void byLockSupport() throws InterruptedException {
        Thread thread = new Thread(() -> {
            int sum = 0;
            for (int i = 0; i < 100; i++) {
                sum += i;
            }
            LockSupport.park();
            log.info("sum: {}", sum);
        });
        thread.start();
        TimeUnit.SECONDS.sleep(2);
        log.info("其他线程唤醒计数线程...");
        LockSupport.unpark(thread);
    }
}
```





