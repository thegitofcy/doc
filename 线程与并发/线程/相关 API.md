# 1.

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



# J.U.C

## CountDownLatch

### 背景

- CountDownLatch 是在 1.5 引入的, 一起引入的工具类还有 CyclicBarrier, Semaphore, ConcurrentHashMap, BlockingQueue.
- 存在于 java.util.concurrent 包下.



### 概念

- 作用: CountDownLatch 是一个多线程控制工具类, 使一个线程等待其他线程各自执行完毕后再执行.
- 实现原理: 通过一个 ==**计数器**== 来实现的. 计数器的初始值就是线程的数量. 每当一个线程执行完毕后, 计数器的值就减 1, 当计数器的值为 0 的时候,就表示所有线程都执行完毕了, 然后在闭锁上等待的线程就可以恢复工作了.



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



## **CyclicBarrier**

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

