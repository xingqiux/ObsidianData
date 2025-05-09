## **一、线程同步是什么？**

**线程同步**是控制多个线程按照特定顺序执行、访问共享资源的机制，目的是解决多线程环境中因并发操作导致的**数据不一致性**问题。核心是要确保：**同一时刻只有一个线程能访问临界区资源**。

## **二、为什么需要线程同步？**

在并发环境中有两大典型问题：

1. **数据竞争（Race Condition）**  
    当多个线程同时修改共享变量时，可能导致不可预知的结果。
    ```java
// 示例：两个线程同时递增同一个计数器
private int count = 0;

public void increment() {
    count++; // 非原子操作，可能丢失更新
}

    ```
2. **内存可见性问题（Memory Visibility）**  
    线程对共享变量的修改可能对其他线程不可见，导致读取到过期数据（由 CPU 缓存机制引发）。

## **三、线程同步的核心概念**

### 1. **临界区（Critical Section）**

涉及共享资源访问的代码块，必须互斥执行。

```java
synchronized (lock) {    // 临界区代码（访问共享资源）}
```

### 2. **原子性（Atomicity）**

保证一系列操作要么全部成功，要么全部失败。例如：`count++` 并非原子操作，需转为原子操作。

### 3. **可见性（Visibility）**

一个线程对共享变量的修改，其他线程能立即读到新值。

### 4. **有序性（Ordering）**

程序执行的顺序需符合代码的编写顺序。

### 5. **互斥（Mutual Exclusion）**

同一时间只允许一个线程访问临界资源。

### 6. **死锁（Deadlock）**

多个线程互相持有对方需要的锁，无法继续执行。


## **四、Java 中如何进行线程同步？**

线程同步是多线程编程中的核心概念，主要解决多线程访问共享资源时可能产生的数据不一致问题。下面详细介绍Java中实现线程同步的几种主要方式。

## 1. synchronized关键字

`synchronized`是Java中最基本的同步机制，它通过独占锁的方式确保同一时间只有一个线程可以执行被保护的代码。

### 同步代码块

```java
synchronized (锁对象) {
    // 需要同步的代码
}
```

- **原理**：线程在执行同步代码块前，必须先获取锁对象的监视器（monitor）。如果监视器已被其他线程占用，则当前线程会被阻塞等待。
- **锁对象**：可以是任何对象，但通常使用共享资源本身或其所在类的实例（`this`）作为锁对象。
- **示例**：
  ```java
public class OrderService {

    private final Object lock = new Object(); // 专门用于锁的对象
    private int orderCount = 0;

    // 处理订单（使用同步代码块）
    public void processOrder() {
        // ... 非同步的逻辑（如数据校验）可放在外围
        
        synchronized (lock) {  // 锁住指定对象
            // 临界区代码（如修改共享变量）
            orderCount++;
            System.out.println("当前订单数量: " + orderCount);
        }
        
        // ... 同步后的其他操作
    }
}

  ```

### 同步方法

```java
public synchronized void method() {
    // 需要同步的代码
}
```

- **原理**：同步方法相当于整个方法体都被`synchronized(this)`包围，实例方法使用对象本身作为锁。
- **适用场景**：整个方法都需要同步时使用，简洁明了。
- **示例**：
  ```java
  public synchronized void withdraw(double amount) {
      if (balance >= amount) {
          balance -= amount;
      } else {
          throw new InsufficientFundsException();
      }
  }
  ```

### 静态同步方法

```java
public static synchronized void method() {
    // 需要同步的代码
}
```

- **原理**：静态同步方法使用类对象（`Class`对象）作为锁，相当于`synchronized(类名.class)`。
- **适用场景**：需要同步静态资源或方法时使用。
- **示例**：
  ```java
  public static synchronized void updateGlobalCounter() {
      globalCounter++;
  }
  ```

## 2. volatile关键字

`volatile`关键字主要解决的是多线程间的可见性问题，但不能保证操作的原子性。

```java
private volatile boolean flag = false;
```

- **可见性**：当一个线程修改了`volatile`变量的值，其他线程能立即看到最新值，因为修改会被立即写回主内存，而读取总是从主内存获取最新值。
- **禁止指令重排序**：编译器和处理器通常会对指令进行重排序以优化性能，`volatile`禁止这种重排序，保证程序执行顺序与代码书写顺序一致。
- **不保证原子性**：例如`count++`这样的复合操作（读取、增加、写回）不是原子的，即使`count`是`volatile`的。
- **适用场景**：
  - 状态标志：如线程停止标志
  - 双重检查锁定（Double-Checked Locking）中的单例模式
  ```java
  // 线程停止标志示例
  public class TaskRunner {
      private volatile boolean running = false;
    
      public void start() {
          running = true;
          new Thread(() -> {
              while (running) {
                  // 执行任务...
              }
          }).start();
      }
    
      public void stop() {
          running = false; // 所有线程立即可见
      }
  }
  ```

## 3. Lock接口

`Lock`接口提供了比`synchronized`更灵活的锁操作，`ReentrantLock`是其最常用的实现。

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 需要同步的代码
} finally {
    lock.unlock();  // 一定要在finally中释放锁
}
```

- **特点**：
  - 可中断获取锁：支持等待锁的线程被中断
  - 可超时获取锁：支持等待锁一段时间后放弃
  - 可实现非阻塞获取锁：尝试获取锁，获取不到立即返回
  - 可实现公平锁：按请求顺序获取锁

- **重要方法**：
  - `lock()`：获取锁，如果锁被占用则阻塞
  - `unlock()`：释放锁
  - `tryLock()`：尝试获取锁，获取成功返回true，失败返回false
  - `tryLock(long timeout, TimeUnit unit)`：在指定时间内尝试获取锁
  - `lockInterruptibly()`：获取锁，但允许当前线程被中断

- **示例**：实现有超时限制的资源访问
  ```java
public class Counter {
    private int count = 0;
    private Lock lock = new ReentrantLock();  // 创建锁对象

    // 尝试在指定时间内增加计数器
    public boolean safeIncrement(long timeout, TimeUnit unit) {
        try {
            // 尝试获取锁，最多等待指定时间
            if (lock.tryLock(timeout, unit)) {
                try {
                    count++;  // 安全地操作共享数据
                    return true;  // 表示操作成功
                } finally {
                    lock.unlock();  // 无论是否异常，必须释放锁
                }
            }
            return false;  // 超时未获取锁，操作失败
        } catch (InterruptedException e) {
            return false;  // 线程被中断，操作失败
        }
    }

    // 获取当前计数值（仅示例，省略同步细节）
    public int getCount() {
        return count;
    }
}

  ```

### 普通锁和公平锁

如果在实例化 Lock 锁的时候，设置 `new ReentrantLock(true);` 则将锁设置为公平锁，如果不带参数或者设置值为 false 则为普通锁


#### **普通锁与公平锁的区别**

**普通锁特点**：

1. **抢占式**：新请求的线程可以"插队"，无需考虑等待队列中已有的线程
2. **高吞吐量**：减少了线程切换的开销，整体性能更高
3. **饥饿风险**：某些线程可能长时间得不到执行机会

**公平锁特点**：

4. **排队机制**：严格按照线程请求锁的顺序分配锁
5. **防止饥饿**：确保每个线程都有机会获得锁
6. **性能开销**：频繁的线程切换会降低整体吞吐量


## 4. ReadWriteLock接口

读写锁允许多个线程同时读取共享资源，但在写入时需要独占访问，`ReentrantReadWriteLock`是其标准实现。

```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();

// 读操作使用读锁
rwLock.readLock().lock();
try {
    // 读取操作
} finally {
    rwLock.readLock().unlock();
}

// 写操作使用写锁
rwLock.writeLock().lock();
try {
    // 写入操作
} finally {
    rwLock.writeLock().unlock();
}
```

- **读锁特点**：共享的，多个线程可以同时持有读锁
- **写锁特点**：独占的，一个线程持有写锁时，其他线程无法获取读锁或写锁
- **适用场景**：读多写少的场景，如缓存、配置数据等

- **示例**：缓存实现
  ```java
  public class CacheData {
      private Map<String, Object> cache = new HashMap<>();
      private ReadWriteLock rwLock = new ReentrantReadWriteLock();
    
      public Object get(String key) {
          rwLock.readLock().lock(); // 使用读锁，允许并发读取
          try {
              return cache.get(key);
          } finally {
              rwLock.readLock().unlock();
          }
      }
    
      public void put(String key, Object value) {
          rwLock.writeLock().lock(); // 使用写锁，独占访问
          try {
              cache.put(key, value);
          } finally {
              rwLock.writeLock().unlock();
          }
      }
  }
  ```

## 5. 线程间通信

线程间通信允许多个线程协调工作，Java 提供了基于对象监视器的线程通信机制。

- **wait()**：使当前线程进入等待状态并释放锁，直到被其他线程通过`notify()`或`notifyAll()`唤醒
- **notify()**：唤醒在当前对象监视器上等待的单个线程
- **notifyAll()**：唤醒在当前对象监视器上等待的所有线程

这些方法**必须在同步块或同步方法中调用**，且当前线程必须持有对象的监视器。

- **经典示例**：生产者-消费者模式
  ```java
  public class Buffer {
      private List<Integer> data = new ArrayList<>();
      private int maxSize = 10;
    
      public synchronized void produce(int value) throws InterruptedException {
          while (data.size() == maxSize) {
              // 缓冲区已满，生产者等待
              wait();
          }
          data.add(value);
          // 通知可能等待的消费者
          notify();
      }
    
      public synchronized int consume() throws InterruptedException {
          while (data.isEmpty()) {
              // 缓冲区为空，消费者等待
              wait();
          }
          int value = data.remove(0);
          // 通知可能等待的生产者
          notify();
          return value;
      }
  }
  ```

**注意事项**：
- `wait()`释放锁，而`Thread.sleep()`不释放锁
- 使用`notifyAll()`比`notify()`更安全，因为`notify()`只唤醒一个线程，如果唤醒的不是我们期望的线程，可能会导致死锁
- 总是在循环中检查等待条件，而不是if语句，这样可以防止虚假唤醒
- 线程通信方法必须在持有锁的情况下调用，否则会抛出`IllegalMonitorStateException`异常

通过这些同步机制，Java提供了丰富而灵活的线程安全编程方式，开发者可以根据具体场景选择合适的同步策略。