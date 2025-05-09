## 一、基础概念

### 1. 线程与进程
- **进程（Process）**：操作系统分配资源的基本单位，拥有独立的内存空间
- **线程（Thread）**：CPU调度的基本单位，共享所属进程的内存空间
- **关系**：一个进程可以包含多个线程，线程是进程内的执行单元

### 2. 线程的生命周期
- **新建（New）**：创建线程对象
- **就绪（Runnable）**：调用start()方法后等待CPU调度
- **运行（Running）**：获得CPU时间片开始执行
- **阻塞（Blocked）**：等待获取锁
- **等待（Waiting）**：调用wait()、join()等方法进入无限期等待
- **超时等待（Timed_Waiting）**：sleep(time)、wait(time)等方法导致的有限期等待
- **终止（Terminated）**：run()方法执行完毕

## 二、线程创建方式

### 1. 继承Thread类
```java
class MyThread extends Thread {
    @Override
    public void run() {
        // 线程执行代码
    }
}
// 使用
MyThread t = new MyThread();
t.start();
```

### 2. 实现Runnable接口
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        // 线程执行代码
    }
}
// 使用
Thread t = new Thread(new MyRunnable());
t.start();
```

### 3. 实现Callable接口（可返回结果）
比 Runnable 接口要功能多一些，可以返回线程执行结果
```java
class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        // 线程执行代码
        return "线程执行结果";
    }
}
// 使用
FutureTask<String> task = new FutureTask<>(new MyCallable());
Thread t = new Thread(task);
t.start();
String result = task.get(); // 获取返回结果
```

### 4. 使用线程池
```java
ExecutorService executor = Executors.newFixedThreadPool(10);
executor.execute(new Runnable() {
    @Override
    public void run() {
        // 线程执行代码
    }
});
// 或者提交任务
Future<String> future = executor.submit(new MyCallable());
```

## 三、线程控制

### 1. 基本方法
- `start()`：启动线程
- `run()`：线程执行体
- `sleep(long millis)`：使线程休眠指定时间
- `join()`：等待线程终止
- `yield()`：让出CPU执行权，但不保证让出后不会再次获取
- `interrupt()`：中断线程
- `isInterrupted()`：检查线程是否被中断
- `interrupted()`：检查当前线程是否被中断，并清除中断状态

### 2. 线程优先级
- 范围：1(最低)~10(最高)，默认为5
- 设置方法：`thread.setPriority(int priority)`
- 常量：`Thread.MIN_PRIORITY`、`Thread.NORM_PRIORITY`、`Thread.MAX_PRIORITY`

### 3. 守护线程
- 特点：随主线程结束而结束
- 设置：`thread.setDaemon(true)` (必须在start前设置)
- 应用：垃圾回收线程、日志记录线程等

## 四、线程同步与锁

### 1. synchronized关键字
- **同步代码块**：
```java
synchronized (对象) {
    // 需要同步的代码
}
```
- **同步方法**：
```java
public synchronized void method() {
    // 需要同步的代码
}
```
- **静态同步方法**：
```java
public static synchronized void method() {
    // 需要同步的代码
}
```

### 2. volatile关键字
- 保证变量对所有线程的可见性
- 禁止指令重排序
- 不能保证原子性
```java
private volatile boolean flag = false;
```

### 3. Lock接口
- **ReentrantLock**：可重入锁，功能类似synchronized但更灵活
```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 需要同步的代码
} finally {
    lock.unlock();  // 一定要在finally中释放锁
}
```

### 4. ReadWriteLock接口
- **ReentrantReadWriteLock**：读写锁，允许多个线程同时读，但只允许一个线程写
```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
// 读锁
rwLock.readLock().lock();
try {
    // 读取操作
} finally {
    rwLock.readLock().unlock();
}
// 写锁
rwLock.writeLock().lock();
try {
    // 写入操作
} finally {
    rwLock.writeLock().unlock();
}
```

### 5. 线程间通信
- `wait()`：让线程等待并释放锁
- `notify()`：唤醒等待队列中的一个线程
- `notifyAll()`：唤醒等待队列中的所有线程

[[Java 多线程 线程同步和锁]]

## 五、并发工具类

### 1. CountDownLatch
- 用途：让一个或多个线程等待其他线程完成操作
```java
CountDownLatch latch = new CountDownLatch(3);
// 其他线程调用
latch.countDown();
// 主线程等待
latch.await();
```

### 2. CyclicBarrier
- 用途：让一组线程互相等待，直到所有线程都到达某个屏障点
```java
CyclicBarrier barrier = new CyclicBarrier(3);
// 线程调用
barrier.await();
```

### 3. Semaphore
- 用途：控制同时访问特定资源的线程数量
```java
Semaphore semaphore = new Semaphore(3);
semaphore.acquire();  // 获取许可
try {
    // 访问资源
} finally {
    semaphore.release();  // 释放许可
}
```

### 4. Exchanger
- 用途：两个线程之间交换数据
```java
Exchanger<String> exchanger = new Exchanger<>();
// 线程1
String data1 = exchanger.exchange("from thread1");
// 线程2
String data2 = exchanger.exchange("from thread2");
```

## 六、线程池

### 1. 线程池核心参数
- **corePoolSize**：核心线程数
- **maximumPoolSize**：最大线程数
- **keepAliveTime**：空闲线程存活时间
- **workQueue**：工作队列
- **threadFactory**：线程工厂
- **handler**：拒绝策略

### 2. 常用线程池
- **FixedThreadPool**：固定大小线程池
```java
ExecutorService fixedPool = Executors.newFixedThreadPool(10);
```

- **CachedThreadPool**：缓存线程池，按需创建线程
```java
ExecutorService cachedPool = Executors.newCachedThreadPool();
```

- **SingleThreadExecutor**：单线程执行器
```java
ExecutorService singlePool = Executors.newSingleThreadExecutor();
```

- **ScheduledThreadPool**：定时任务线程池
```java
ScheduledExecutorService scheduledPool = Executors.newScheduledThreadPool(5);
```

### 3. 自定义线程池
```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                      // 核心线程数
    10,                     // 最大线程数
    60, TimeUnit.SECONDS,   // 空闲线程存活时间
    new LinkedBlockingQueue<>(100),  // 工作队列
    Executors.defaultThreadFactory(), // 线程工厂
    new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
);
```

### 4. 拒绝策略
- **AbortPolicy**：抛出RejectedExecutionException异常（默认）
- **CallerRunsPolicy**：由调用线程执行任务
- **DiscardPolicy**：直接丢弃任务
- **DiscardOldestPolicy**：丢弃队列最前面的任务，然后重新尝试执行任务

## 七、并发容器

### 1. ConcurrentHashMap
- 线程安全的HashMap，替代Hashtable
```java
ConcurrentHashMap<String, String> map = new ConcurrentHashMap<>();
```

### 2. CopyOnWriteArrayList
- 线程安全的ArrayList，适用于读多写少的场景
```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
```

### 3. CopyOnWriteArraySet
- 线程安全的Set，基于CopyOnWriteArrayList实现
```java
CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();
```

### 4. BlockingQueue
- **LinkedBlockingQueue**：基于链表的FIFO阻塞队列
- **ArrayBlockingQueue**：基于数组的FIFO有界阻塞队列
- **PriorityBlockingQueue**：支持优先级排序的无界阻塞队列
- **DelayQueue**：延迟获取元素的无界阻塞队列
- **SynchronousQueue**：不存储元素的阻塞队列

## 八、原子类

### 1. 基本类型原子类
- AtomicInteger
- AtomicLong
- AtomicBoolean

### 2. 数组类型原子类
- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray

### 3. 引用类型原子类
- AtomicReference
- AtomicStampedReference
- AtomicMarkableReference

### 4. 字段更新器
- AtomicIntegerFieldUpdater
- AtomicLongFieldUpdater
- AtomicReferenceFieldUpdater

## 九、并发编程最佳实践

### 1. 线程安全策略
- 不可变对象（Immutable Objects）
- 线程封闭（Thread Confinement）
- 同步容器（Synchronized Collections）
- 并发容器（Concurrent Collections）
- 显式锁（Explicit Locks）

### 2. 性能优化
- 减少锁的粒度
- 减少锁的持有时间
- 使用读写锁分离读写操作
- 使用并发容器代替同步容器
- 合理使用线程池

### 3. 避免死锁的策略
- 固定加锁顺序
- 使用定时锁
- 使用锁的层次结构
- 及时释放锁资源

## 十、Java内存模型（JMM）

### 1. 主要内容
- 原子性：保证指令不会受到线程上下文切换的影响
- 可见性：保证指令不会受到CPU缓存的影响
- 有序性：保证指令不会受到CPU指令重排序的影响

### 2. happens-before规则
- 程序顺序规则：同一个线程中，前面的操作先行发生于后面的操作
- 监视器锁规则：unlock操作先行发生于后续对同一个锁的lock操作
- volatile变量规则：对volatile变量的写操作先行发生于后续对该变量的读操作
- 线程启动规则：线程的start()方法先行发生于线程中的每一个操作
- 线程终止规则：线程中的所有操作都先行发生于线程的终止检测
- 线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- 对象终结规则：对象的构造函数执行完成先行发生于finalizer的开始
- 传递性：如果A先行发生于B，且B先行发生于C，那么A先行发生于C