# java线程

### 死锁

死锁：不同线程分别占用对方需要的同步资源不放弃，都在等待对方放弃自己需要的同步资源，就形成了线程的死锁。说明

1、出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续

2、我们使用同步时，要避免出现死锁
解决方法：1、专门的算法，原则2、尽量减少同步资源的定义3、尽量避免嵌套同步

### synchronized于lock的异同？

相同：二者都可以解决线程安全问题

不同：

1. synchronized机制在执行完相应的同步代码以后，自动的释放同步监视器；lock需要手动启动同步（lock()），同时结束同步也需要手动的实现（unlock()）
2. synchronized是Java语言的关键字，因此是内置特性，Lock不是Java语言内置的，Lock是一个接口，通过实现类可以实现同步访问。
3. 在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态。

### 线程的通信

涉及到的三个方法：

wait():一旦执行此方法，当前线程就进入阻塞状态，并释放同步监视器。

notify();一旦执行此方法，就唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个。

notifyAll():

说明：

1、wait(),notify(),notifyAll()三个方法必须使用在同步代码块或同步方法中。

2、wait(),notify(),notifyAll()三个方法的调用者必须是同步代码块或同步方法中的同步监视器，否则会出现IllegalMonitorStateException

3、wait(),notify(),notifyAll()三个方法是定义在java.lang.Object类中

### sleep()和wait()的异同

1、相同点：一旦执行方法，都可以使得当前的线程进入阻塞状态。

2、不同点：

1）两个方法声明的位置不一样：Thread类中声明sleep()，Object类中声明wait()

2）调用的要求不用：sleep()可以在任何的场景下调用。wait()必须在同步代码块中调用

3）如果两个方法都使用在同步代码块或同步方法中，sleep()不会释放同步监视器，wait()会释放同步监视器

### Callable接口

说明（相较于实现Runnable创建多线程的强大之处）：

1. 相比run方法，可以有返回值方法

2. 可以抛出异常

3. 支持泛型返回值

创建方式：

1. 创建一个实现Callable的实现类
2. 实现call方法，将此线程需要执行的操作声明在call中
3. 创建Callable接口的实现类对象
4. 将此Callable接口实现类对象作为参数传递到FutureTask构造器中，创建FutureTask对象
5. 将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象，并调用start方法
6. 获取Callable中的call方法的返回值

## 线程池

线程池的使用流程：

```java
// 创建线程池
ThreadPoolExecutor threadPool = new ThreadPoolExecutor(CORE_POOL_SIZE,
                                             MAXIMUM_POOL_SIZE,
                                             KEEP_ALIVE,
                                             TimeUnit.SECONDS,
                                             sPoolWorkQueue,
                                             sThreadFactory);
// 向线程池提交任务
threadPool.execute(new Runnable() {
    @Override
    public void run() {
        ... // 线程执行的任务
    }
});
// 关闭线程池
threadPool.shutdown(); // 设置线程池的状态为SHUTDOWN，然后中断所有没有正在执行任务的线程
threadPool.shutdownNow(); // 设置线程池的状态为 STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表
```

构造器需要用的参数：

- **corePoolSize**（必需）：核心线程数。默认情况下，核心线程会一直存活，但是当将**allowCoreThreadTimeout**设置为true时，核心线程也会超时回收。
- **maximumPoolSize**（必需）：线程池所能容纳的最大线程数。当活跃线程数达到该数值后，后续的新任务将会阻塞。
- **keepAliveTime**（必需）：线程闲置超时时长。如果超过该时长，非核心线程就会被回收。如果将**allowCoreThreadTimeout**设置为true时，核心线程也会超时回收。
- **unit**（必需）：指定keepAliveTime参数的时间单位。常用的有：**TimeUnit.MILLISECONDS**（毫秒）、**TimeUnit.SECONDS**（秒）、**TimeUnit.MINUTES**（分）。
- **workQueue**（必需）：任务队列。通过线程池的execute()方法提交的Runnable对象将存储在该参数中。其采用阻塞队列实现。
- **threadFactory**（可选）：线程工厂。用于指定为线程池创建新线程的方式。
- **handler**（可选）：拒绝策略。当达到最大线程数时需要执行的饱和策略。

### 线程池的工作原理

![](D:\ZhongmingNote\Fig\线程池工作原理.png)

​		任务进来时，首先执行判断，判断**核心线程是否处于空闲状态**，如果不是，核心线程就先就执行任务，如果核心线程已满，则判断**任务队列是否有地方存放该任务**，若果有，就将任务保存在任务队列中，等待执行，如果满了，在判断**最大可容纳的线程数**，如果没有超出这个数量，就开创非核心线程执行任务，如果超出了，就调用handler实现拒绝策略。

### 拒绝策略

​		当线程池的线程数达到最大线程数时，需要执行拒绝策略。拒绝策略需要实现**RejectedExecutionHandler**接口，并实现**rejectedExecution(Runnable r, ThreadPoolExecutor executor)**方法。不过Executors框架已经为我们实现了4种拒绝策略：

1. **AbortPolicy（默认）**：丢弃任务并抛出**RejectedExecutionException**异常。
2. **CallerRunsPolicy**：由调用线程处理该任务。
3. **DiscardPolicy**：丢弃任务，但是不抛出异常。可以配合这种模式进行自定义的处理方式。
4. **DiscardOldestPolicy**：丢弃队列最早的未处理任务，然后重新尝试执行任务。

