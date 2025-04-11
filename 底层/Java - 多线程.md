### **多线程与并发**

## 1. 线程状态与生命周期

**Java线程的6种状态（Thread.State）**:

1. NEW - 新创建但未启动的线程
2. RUNNABLE - 可运行状态，包括就绪和运行中
3. BLOCKED - 被阻塞等待监视器锁
4. WAITING - 无限期等待另一线程执行特定操作
5. TIMED_WAITING - 指定等待时间的等待状态
6. TERMINATED - 已执行完毕的线程


    - `wait()` 和 `sleep()` 的区别？
    1. wait 是线程自己进入等待时刻，而 sleep 是线程进入休眠状态
7. **线程安全性问题**
    
    - synchronized 的实现原理？锁膨胀过程（偏向锁→轻量级锁→重量级锁）？
    - volatile 关键字的作用（可见性、禁止指令重排）？
8. **JUC 工具类**
    
    - ReentrantLock 与 synchronized 的区别？AQS（AbstractQueuedSynchronizer）原理？
    - CountDownLatch 和 CyclicBarrier 的应用场景？
9. **线程池原理**
    
    - 核心参数（corePoolSize、maximumPoolSize、workQueue）的作用？
    - 线程池拒绝策略？如何合理配置线程池大小？
10. **ThreadLocal**
    
    - ThreadLocal 实现原理？为什么可能引发内存泄漏？
11. **CAS 与原子类**
    
    - CAS 的 ABA 问题如何解决？AtomicStampedReference 的作用？


### 一、什么是线程安全（Thread Safety）？

**线程安全**是指：在多线程环境下，多个线程同时访问同一个共享资源（如对象、变量、数据结构）时，程序仍能保持**正确性**和**一致性**，不会因线程的交替执行导致数据错误或逻辑混乱。

#### 举个生活化的例子：
假设有一个银行账户（共享资源），两个线程同时操作这个账户：
- **线程 A**：存入 100 元。
- **线程 B**：取出 50 元。

如果账户余额初始为 0，**线程不安全**的情况可能是：
1. 线程 A 读取余额为 0，准备存入 100。
2. 线程 B 读取余额也为 0，准备取出 50。
3. 线程 A 存入后余额变为 100。
4. 线程 B 取出后余额变为 -50（错误结果）。

**线程安全**的设计会保证：  
两个操作按顺序执行，最终余额为 50（正确结果）。

---

### 二、`synchronized` 是什么？

`synch­ronized` 是 Java 中实现线程安全的核心关键字，它通过**互斥锁（Lock）** 确保同一时刻只有一个线程能访问被保护的代码块或方法。

#### `synchronized` 的两种用法：
1. **修饰方法**：锁住整个方法。
   ```java
   public synchronized void add(int value) {
       count += value; // 同一时间只有一个线程能执行此方法
   }
   ```

2. **修饰代码块**：锁住指定对象。
   ```java
   public void add(int value) {
       synchronized (this) { // 锁住当前对象
           count += value;
       }
   }
   ```

---

### 三、为什么 `StringBuffer` 用 `synchronized`，而 `StringBuilder` 不用？

#### 1. `StringBuffer` 的线程安全实现
- **所有方法都加了 `synchronized`**：  
  例如 `append()` 方法：
  ```java
  public synchronized StringBuffer append(String str) {
      super.append(str);
      return this;
  }
  ```
  同一时间只有一个线程能调用 `append()`，避免多线程并发修改导致数据错乱。

#### 2. `StringBuilder` 的非线程安全设计
- **没有 `synchronized` 修饰**：  
  例如 `append()` 方法：
  ```java
  public StringBuilder append(String str) {
      super.append(str); // 无锁，多线程并发修改可能出错
      return this;
  }
  ```
  性能更高，但多线程下需自行处理同步问题。

---

### 四、线程安全的直观对比（示例）

#### 场景：两个线程同时向同一个对象追加字符 10000 次。
```java
// 使用 StringBuilder（非线程安全）
StringBuilder sb = new StringBuilder();
Runnable task = () -> {
    for (int i = 0; i < 10000; i++) {
        sb.append("a");
    }
};

Thread t1 = new Thread(task);
Thread t2 = new Thread(task);
t1.start();
t2.start();
t1.join();
t2.join();

System.out.println(sb.length()); // 结果可能小于 20000（数据丢失）！
```

```java
// 使用 StringBuffer（线程安全）
StringBuffer sb = new StringBuffer();
Runnable task = () -> {
    for (int i = 0; i < 10000; i++) {
        sb.append("a");
    }
};

Thread t1 = new Thread(task);
Thread t2 = new Thread(task);
t1.start();
t2.start();
t1.join();
t2.join();

System.out.println(sb.length()); // 结果一定是 20000
```

---

### 五、总结

| 概念               | 说明                                                                 |
|--------------------|--------------------------------------------------------------------|
| **线程安全**        | 多线程并发访问共享资源时，程序仍能正确运行。                                  |
| **`synchronized`** | 通过锁机制实现线程安全，但会带来性能开销。                                    |
| **`StringBuffer`** | 用 `synchronized` 保证线程安全，适合多线程环境。                             |
| **`StringBuilder`**| 无锁，性能更高，但仅限单线程使用。                                         |

#### 选择建议：
- 单线程操作字符串：**`StringBuilder`**（更快）。
- 多线程操作字符串：**`StringBuffer`**（更安全）。
- 无需修改的字符串：**`String`**（不可变，天然线程安全）。



# Java多线程与并发知识点整理

针对您提供的面试题，我将给出解答，并在后面补充一些更常见且相对简单的多线程问题。

## 1. 线程状态与生命周期

**Java线程的6种状态（Thread.State）**:
1. NEW - 新创建但未启动的线程
2. RUNNABLE - 可运行状态，包括就绪和运行中
3. BLOCKED - 被阻塞等待监视器锁
4. WAITING - 无限期等待另一线程执行特定操作
5. TIMED_WAITING - 指定等待时间的等待状态
6. TERMINATED - 已执行完毕的线程

**wait()和sleep()的区别**:
- `wait()`是Object类的方法，而`sleep()`是Thread类的静态方法
- `wait()`会释放锁，`sleep()`不会释放锁
- `wait()`必须在同步块/方法中调用，`sleep()`可以在任何地方调用
- `wait()`需要通过`notify()/notifyAll()`唤醒，`sleep()`时间到会自动恢复

## 2. 线程安全性问题

**synchronized实现原理**:
- 基于Monitor机制实现，在字节码层面通过monitorenter和monitorexit指令
- 锁膨胀过程：
  - 偏向锁：只有一个线程访问时，标记为偏向该线程，避免CAS操作
  - 轻量级锁：多线程竞争不激烈时，通过CAS操作尝试获取锁
  - 重量级锁：竞争激烈时，线程进入等待队列，由操作系统调度

**volatile关键字作用**:
- 保证可见性：一个线程修改变量值，对其他线程立即可见
- 禁止指令重排：通过内存屏障确保指令执行顺序
- 不保证原子性：复合操作下仍可能出现线程安全问题

## 3. JUC工具类

**ReentrantLock与synchronized区别**:
- ReentrantLock是API层面实现，synchronized是JVM层面实现
- ReentrantLock提供更多功能：可中断锁、公平锁、尝试获取锁等
- ReentrantLock需要手动释放锁（try-finally块），synchronized自动释放

**AQS原理**:
- AbstractQueuedSynchronizer，同步器框架
- 基于FIFO队列管理线程
- 通过state整型变量维护同步状态
- 支持独占模式和共享模式的资源访问

## 4. 线程池原理

**核心参数作用**:
- corePoolSize：核心线程数，长期保持的线程数
- maximumPoolSize：最大线程数，任务增多时可创建的最大线程数
- workQueue：等待队列，存放等待执行的任务

**线程池执行流程**:
1. 线程数小于corePoolSize，创建新线程执行任务
2. 线程数达到corePoolSize，任务放入队列等待
3. 队列已满，且线程数小于maximumPoolSize，创建新线程
4. 队列已满，且线程数达到maximumPoolSize，触发拒绝策略

## 更简单且常见的多线程面试题

### 1. 创建线程的方式有哪些？
- 继承Thread类
- 实现Runnable接口
- 实现Callable接口（可返回结果）
- 使用线程池ExecutorService

```java
// 方式1：继承Thread类
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}
// 使用：new MyThread().start();

// 方式2：实现Runnable接口
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Runnable running");
    }
}
// 使用：new Thread(new MyRunnable()).start();

// 方式3：实现Callable接口
class MyCallable implements Callable<String> {
    public String call() {
        return "Callable result";
    }
}
// 使用：
// FutureTask<String> task = new FutureTask<>(new MyCallable());
// new Thread(task).start();
// String result = task.get();

// 方式4：线程池
// ExecutorService executor = Executors.newFixedThreadPool(5);
// executor.submit(new MyRunnable());
```

### 2. 线程的生命周期和状态切换
![线程状态图]

- NEW → RUNNABLE：调用start()方法
- RUNNABLE → BLOCKED：等待进入synchronized块/方法
- RUNNABLE → WAITING：调用wait()、join()、LockSupport.park()
- RUNNABLE → TIMED_WAITING：调用sleep(time)、wait(time)
- BLOCKED/WAITING/TIMED_WAITING → RUNNABLE：获得锁、被通知、超时、被中断
- RUNNABLE → TERMINATED：线程执行完成

### 3. 如何停止一个线程？
- 使用中断机制（推荐）：调用thread.interrupt()，并在线程内检查中断状态
- 使用标志位：设置一个volatile布尔变量，线程检查该变量决定是否继续执行

```java
// 使用中断
public class StopThread implements Runnable {
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            // 执行任务
            System.out.println("Thread is running");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                // 重新设置中断状态
                Thread.currentThread().interrupt();
                break;
            }
        }
        System.out.println("Thread stopped");
    }
}

// 使用标志位
public class StopThread2 implements Runnable {
    private volatile boolean stopped = false;
    
    public void stop() {
        stopped = true;
    }
    
    public void run() {
        while (!stopped) {
            // 执行任务
            System.out.println("Thread is running");
        }
        System.out.println("Thread stopped");
    }
}
```

### 4. 什么是线程安全？如何实现线程安全？
**线程安全**指多个线程同时访问共享资源时，不会导致数据不一致或操作失败。

**实现线程安全的方法**：
- 使用synchronized关键字
- 使用Lock接口实现类（ReentrantLock等）
- 使用原子类（AtomicInteger等）
- 使用线程安全的集合类（ConcurrentHashMap等）
- 使用ThreadLocal实现线程本地存储

### 5. 什么是死锁？如何避免？
**死锁**：两个或多个线程互相等待对方持有的锁，导致永久阻塞。

**死锁发生的四个必要条件**：
- 互斥：资源不能同时被多个线程使用
- 持有并等待：线程持有资源的同时等待其他资源
- 不可剥夺：线程获得的资源在未使用完前不能被其他线程强制剥夺
- 循环等待：多个线程形成一个环形等待链

**避免死锁的方法**：
- 固定锁的获取顺序
- 使用tryLock()方法带超时机制
- 使用开放调用（避免在持有锁时调用外部方法）
- 使用线程池管理并发数量

```java
// 死锁示例
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized (lock1) {
            System.out.println("Holding lock1...");
            try { Thread.sleep(100); } catch (Exception e) {}
            synchronized (lock2) {
                System.out.println("Holding lock1 and lock2");
            }
        }
    }
    
    public void method2() {
        synchronized (lock2) {  // 这里改变了获取锁的顺序，可能导致死锁
            System.out.println("Holding lock2...");
            try { Thread.sleep(100); } catch (Exception e) {}
            synchronized (lock1) {
                System.out.println("Holding lock2 and lock1");
            }
        }
    }
}
```

### 6. synchronized和volatile的区别
- synchronized保证原子性、可见性和有序性
- volatile只保证可见性和有序性，不保证原子性
- synchronized会阻塞线程，volatile不会
- synchronized可以修饰方法和代码块，volatile只能修饰变量

### 7. ThreadLocal的使用场景
- 线程隔离：为每个线程创建独立的变量副本
- 常见场景：存储用户会话信息、数据库连接、事务上下文等

```java
public class ThreadLocalExample {
    // 创建ThreadLocal变量
    private static ThreadLocal<String> userContext = new ThreadLocal<>();
    
    public static void setUser(String user) {
        userContext.set(user);
    }
    
    public static String getUser() {
        return userContext.get();
    }
    
    public static void clear() {
        userContext.remove();  // 防止内存泄漏
    }
}
```

这些基础问题更常见，掌握好它们对面试会有很大帮助。希望对您的学习有所帮助！您还有其他关于多线程的问题吗？