### 4. join 详解

#### 1. 作用: ==**谁 join, 就等待谁执行完毕后再执行.**==

==**因为新的线程加入了我们, 所以我们要等待他执行完再出发.**==



#### 2. 用法

通常是在 main 线程中等待子线程 thread 执行完毕. 注意是谁等待谁: ==**主线程等待子线程.**==

通过上面那句话解释就是: ==**子线程**== 加入 ==**main 线程**==,  子线程 `thread.join()`, 此时 main 线程就会等待 thread 线程执行完毕后再执行.



#### 3. 场景举例

启动服务的时候, 需要初始化一些资源, 比如初始化 5 个资源, 那么这里就可以通过 5 个子线程 join, 此时主线程就会等待这 5 个子线程执行完毕后再执行.



#### 4. 演示

```java
/**
 *  演示 main 线程等待子线程执行完毕后, 再执行.
 */

public class Join {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " 执行完毕");
        });

        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " 执行完毕");
        });

        thread1.start();
        thread2.start();
        System.out.println("main 线程将要开始等待子线程执行完毕");

        /**
         * 对于 main 线程来说, 执行到这里后, 就会开始等待子线程执行,等子线程执行完毕后, main 线程才会执行.
         * 所有结果应该是先打印 "main 线程将要开始等待子线程执行完毕", 然后打印两个子线程的内容,
         * 最后打印 "所有子线程执行完毕".
         *
         * 如果将两个 join 注释掉,对于主线程来说就不会再等待子线程执行完毕, 所以在执行完上面那句后,
         * 马上输出 "所有子线程执行完毕".
         */
//        thread1.join();
//        thread2.join();
        System.out.println("所有子线程执行完毕");
    }
}
```



#### 5. join 期间遇到中断

在子线程 join 的时候, 

- 普通中断
- 遇到中断

```java
/**
 *  join 期间 interrupt, 其实是主线程中断. 所以对于子线程的中断应该是在 join 的catch 中进行传递,
 *  也就是使用子线程的对象调用 interrupt
 */
public class JoinWithInterrupt {
    public static void main(String[] args) {
        Thread mainThread = Thread.currentThread();

        Thread thread = new Thread(() -> {
            try {
                mainThread.interrupt();
                Thread.sleep(5000);
                System.out.println("子线程运行完毕.");
            } catch (InterruptedException e) {
                System.out.println("子线程中断. 可以运行子线程的");
            }
        });

        thread.start();
        System.out.println("1. main 线程等待子线程运行完毕");
        try {
            System.out.println("2. 子线程开始进行 join");
            thread.join();
        } catch (InterruptedException e) {
            // 到这里为止其实是 main 线程中断了.
            System.out.println("3. " + Thread.currentThread().getName() + " 线程中断!");
            // 如果不将中断传递给子线程, 那么子线程还是会继续往下运行, 会输出 "子线程运行完毕."
            // 如果将中断传递给子线程, 那么在子线程 sleep 的时候, 就会获取到中断, 然后运行 catch, 打印出 "子线程中断". 其实这里就是中断时,
            // 子线程的善后操作
            thread.interrupt();
        }
        System.out.println("主线程打印的 子线程运行完毕");
    }
}
```



#### 6. join 期间, 主线程的状态.

结论: 





#### CountDownLatch/CyclicBarrier

#### 原理

#### 面试问题





### 5. yield 方法



### 6. 获取当前线程 Thread.currentThread()



### 7. run 和 start 方法



### 8. stop, suspend, resume



### 9. 常见面试问题.







## 6. 线程的各种属性

## 