# 线程基础知识

## 线程与进程区别

**进程**：当一个程序被运行，从磁盘加载程序代码到内存，这时就是开启了一个进程

**线程**：一个线程就是一个执行流，将指令流中的指令交给 CPU 执行

**区别**：
- 一个进程可以包含多个线程
- 同一个进程下的所有线程可以共享内存空间，不同进程使用不同内存空间
- 线程更轻量，线程上下文切换的成本比进程低


## 并行与并发区别

**并行**：一台机器多个核心在运行，他们在同一时刻同时执行成为并行
**并发**：一台核心上并发执行两个程序，他们以很快的频率切换，在宏观上看是同时执行，但微观上看是分别执行的

## 线程创建的方法有哪些

一共有四种方法，分别是继承 Thread ，实现 Runnable 接口，使用 Callable<> 方法，线程池创建
### 继承 Thread

通过继承 Thread 类，重写 run 方法，然后通过 start() 方法运行

```java
class Thread1 extends Thread{  
    @Override  
    public void run(){  
        for (int i = 0 ; i < 100 ; i++){  
            System.out.println("当前线程1输出:"+i);  
        }  
    }  
}

// 继承 Thread 类  
@Test  
public void testThread(){  
    Thread1 thread1 = new Thread1();
      
    thread1.start();  
}
```

### 实现 Runnable 接口

通过实现 Runnable 接口，实现其中的 run 方法，然后通过创建 Thread 的构造方法运行

```java
class Runnable1 implements Runnable{  
    @Override  
    public void run() {  
        for (int i = 0; i < 50; i++) {  
            System.out.println("当前Runnable线程1输出:"+i);  
        }  
    }  
}

@Test  
public void testRunnable(){  
    Runnable1 runnable1 = new Runnable1();  
  
    // 使用的使用需要通过创建 Thread 的构造方法  
    Thread thread1 = new Thread(new Runnable1());  
  
    thread1.start();  
}
```

### 使用 Callable 接口

Callable 接口可以返回运行结束后的结果，需要先实例化这个类，然后使用这个实例创建一个 FutureTask 类，然后再使用这个 FurureTask 类创建线程 Thread 然后启动线程

```java

// 实现 Callable 接口  
class Callable1 implements Callable<String> {  
    @Override  
    public String call(){  
        try{  
            for (int i = 0; i < 60; i++) {  
                System.out.println("当前Callable线程1输出:"+i);  
            }  
            return "运行正常结束";  
        }catch (Exception e){  
            e.printStackTrace();  
            return "运行错误";  
        }  
    }  
}

public void testCallable(){  

    FutureTask<String> task1 = new FutureTask<String>(new Callable1());  
  
    Thread thread1 = new Thread(task1);  
  
    thread1.start();  
  
    try {  
        System.out.println("---------最后运行结果1"+task1.get());  
    } catch (InterruptedException e) {  
        throw new RuntimeException(e);  
    } catch (ExecutionException e) {  
        throw new RuntimeException(e);  
    }  
}
```

### 线程池创建（项目中常使用）

先实现一个 Runnable 接口，然后创建一个线程池提交

```java

class Executor1 implements Runnable{  
    @Override  
    public void run() {  
        for (int i = 0; i < 50; i++) {  
            System.out.println("当前线程1输出:"+i);  
        }  
    }  
}
    @Test  
    public void testExecutor(){  
  
        ExecutorService executorService = Executors.newFixedThreadPool(10);  
        executorService.submit(new Executor1());  
  
        executorService.shutdown();  
  
    }  
}



```

## Runnable 与 Callable 的区别

1. Runnable 没有返回值，Callable 有返回值
2. Callable 可以配合 FutureTask 异步获取执行的结果
3. Callable 可以抛出异常，但 Runnable 无法抛出异常只能内部消化


## Java 的线程包括哪些状态

```java
public enum State {  
     NEW,  
     RUNNABLE,    
     BLOCKED,  
     WAITING,  
     TIMED_WAITING,  
     TERMINATED;  
}
```

Java 中一共有以上这些状态，分别代表 新建态，运行态，阻塞态，无限等待，超时等待，终止

- **NEW**：新建状态，主要是线程被创建但并没有调用 start()
- **RUNNABLE**：调用 `start()` 后，线程进入就绪或运行状态。
- **BLOCKED**：线程等待进入 `synchronized` 代码块/方法，或 `wait()` 后重新获取锁时锁被占用。
- **WATTING**：调用以下方法之一：
	- `Object.wait()`（不带超时）
	- `Thread.join()`（不带超时）
	- `LockSupport.park()`
	- 需要其他线程干预比如`notify()`、`unpark()`）才能恢复。
- **TIMED_WATTING**：调用带超时参数的方法：
    - `Thread.sleep(long)`
    - `Object.wait(long)`
    - `Thread.join(long)`
    - `LockSupport.parkNanos()`
    - `LockSupport.parkUntil()`
	- **特点**：在指定时间后自动恢复或提前被唤醒。
- **TERMINATED**：线程执行完毕（`run()` 结束）或异常终止。生命周期结束

线程中的状态切换主要可以看以下图

![[Java - 多线程.png]]

## 线程按顺序执行

可以使用 `join()` 方法，在一个线程中调用另一个线程中的 join 方法，就会等待另一个线程执行完再执行当前线程

以下是示例代码
```java
/**  
 * 线程按照顺序执行的方法 join 函数  
 */  
@Test  
public void testJoin(){  
    Thread t1 = new Thread(() -> {  
        System.out.println("t1");  
    });  
    Thread t2 = new Thread(() -> {  
        try {  
            t1.join();  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        }  
        System.out.println("t2");  
    });  
    Thread t3 = new Thread(() -> {  
        try {  
            t2.join();  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        }  
        System.out.println("t3");  
    });  
  
    t3.start();  
    t2.start();  
    t1.start();  
}
```


## notify() 和 notifyall() 有什么区别

当线程休眠时，notify() 可以随机唤醒线程，notifyall() 可以唤醒全部线程

notifyall() 唤醒所有 wati() 中的代码
notify() 随机唤醒一个 wati() 中的代码


## `wait()` 方法 `sleep()` 方法有什么不同

### **共同点**：
wait(),wati(long),sleep(long) 这三个方法都可以暂时让线程放弃 cpu 的使用权

### **不同点**：
**方法归属不同**
sleep(long) 是属于 **静态方法** 是 属于 Thread
wati() 是属于 **实例方法** ,每个对象都有

**醒来时机不同**
wati(long) 可以在时间结束后被唤醒，还可以被我 notify 或 notifyall 唤醒

**锁特性不同**
wati 方法调用钱必须获得 wati 对象锁
wait 方法会释放对象锁，其他线程可以获得
sleep方法如果在 synchronized 中使用则不会释放对象锁

## 如何停止一个正在运行的线程

### 设定标记
设定一个标志位用于退出
```java
/**  
 * 让正在运行的线程退出方法  
 */  
@Test  
public void testInterrupt(){  
     AtomicBoolean flag = new AtomicBoolean(true);  
     // 这个线程会运行2次，然后因为标志位改变而终止  
    new Thread(() -> {  
        while (flag.get()) {  
            System.out.println("正在运行");  
            try {  
                Thread.sleep(3000L);  
            } catch (InterruptedException e) {  
                throw new RuntimeException(e);  
            }  
        }  
    }).start();  
  
    new Thread(() -> {  
        try {  
            Thread.sleep(6000L);  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        }  
        System.out.println("线程被终止");  
        flag.set(false);  
    }).start();  
	// 防止主线程被关闭
    try {  
        Thread.sleep(10000L);  
    } catch (InterruptedException e) {  
        throw new RuntimeException(e);  
    }  
}
```

### 使用 interrupt 方法打断线程

如果打断 wati,sleep,join 的线程，会报错
如果打断 正在运行的线程 会成功终止执行
 

# 线程中的并发安全

## `synchronized` 关键字的底层原理

synchronized 对象锁采用互斥的方式让同一时刻最多只能有一个线程获得对象锁，其他线程想要获取锁就会被阻塞

### `Monitor` 与锁的使用

^0fb5a0

`Monitor` 是一个监视器，由 JVM 提供，C++ 实现

获取锁的时候需要使用对象锁管理 `Monitor`

`Monitor` 结构
	Owner：关联存储当前获取锁的线程，只能有一个个线程可以获取
	EntryList：关联没有抢到锁的线程会处于 Blocked 状态存放在 EntryList
	WatiSet : (调用了 wati())等待的线程放在 WatiSet 中，处于 Waiting 状态
	
	

![[Java - 多线程-1.png|800]]
### 重量级锁(多线程锁竞争)

- `Monitor` 实现的锁属于**重量级锁**，里面涉及到用户态与内核态的切换、进程的上下文切换、成本较高、性能低
- JDK1.6 引入了 **偏向锁** 和 **轻量级锁**，为了解决没有多线程竞争或基本没有竞争场景的性能开销问题

#### 对象内存结构
在 HotSpot 虚拟机(JVM 的一种实现)中，对象在内存中存储分为三部分：对象头、实例数据、对齐填充

- **对象头**：
	- MarkWord ： 描述实例对象与 Monitor 的关联
	- KlassWord ：描述对象实例的具体类型 User 类型或 Order 类型之类
- **实例数据**：存储的对象中的成员变量属性
- **对齐填充**：如果对象头+示例数据不是8的整数倍通过无意义数据填充补齐

**MarkWord** 是关于对象锁的重点知识点
![[Java - 多线程-2.png|750]]

这个表是对象内存中对象头里的 MarkWord 信息，MarkWord 的各个表示代表各个锁的意思，比如重量级锁在 MarkWord 中前30位用于只想 Monitor 指针，然后偏向锁也有各个不同的代表含义

这里就能解释[[#^0fb5a0 | Monitor 原理]] 中对象锁如何关联到 Monitor 监视器，就是通过设置对象内存中对象头的 MarkWord 为指向 Monitor 的指针

### 轻量级锁(不同线程交替持有)

轻量级锁包含 CAS 操作，不可出现竞争否则升级为重量级锁，轻量级锁具有可重入特性

#### lock_record 与 线程
![[Java - 多线程-3.png|775]]
**是什么**：是**锁记录**，每一个线程的**栈帧**都包含一个锁记录(这里的 栈帧 我简单理解就是重入一次锁的"帧")。

**有什么作用**：主要是记录锁对象的状态与指向锁对象的位置，如图
![[Java - 多线程-4.png|500]]

轻量级锁的对象 MarkWord 中指向的是 lock_record 锁地址与 00 标识，这里记为 `lock_record 00` 
而无锁状态下则是 hashcode,age,bliased_lock 00 这些信息，所以记为 `hashcode age 00 1` 


#### 加锁流程
1. 线程中创建一个 Lock Record 锁记录，然后与锁对象进行一次 **CAS** 操作(就相当于原子操作)，将 Object 中的 MarkWord 信息`hashCode age 0 01` 与 Lock Record 的 MarkWord 信息 `lock_record 00` 进行交换，再将使得 Object  设定为轻量级锁，并且 Lock Record 的示例部分也要指向 Object
![[Java - 多线程-5.png|575]]
- 如果失败，说明可能发生了**竞争**，这时会自动升级成重量级锁

1. 如果当前线程已经持有锁，代表这次为**锁重入**，就会在栈帧中再加一个 Lock Record 记录，此时会再与锁对象 Object 进行一次 CAS 操作，但不会存储 MarkWord 数据（因为第一次时已经添加过了）设置为 null,同时这个记录也会指向锁对象 
![[Java - 多线程-6.png|525]]


#### 解锁流程
1. 遍历线程栈，如果有重入记录(obj 字段等于当前锁对象)，直接按退出的顺序删除 Lock Record 记录即可，将  Lock Record指向的 Object设置为 null
2. 如果 Lock Record 的 Mark Word 不为null，要再进行一次 CAS 操作，将锁对象的 mark word 恢复成无锁状态，如果失败则膨胀为重量级锁
![[Java - 多线程-7.png|500]]![[Java - 多线程-8.png|500]]
即可，这时 Object 就是未上锁状态

### 偏向锁(只有一个线程持有)
因为轻量级锁在没有竞争时还会在重入时多次执行 CAS 操作，所以轻量级锁只有第一次使用 CAS 将线程 ID 对象的 Mark Word 头，之后发现这个线程 ID 是自己的没有竞争，就不重新进行 CAS，只要不发生竞争，这个对象就归该线程所有

![[Java - 多线程-9.png|850]]

也就是说，这个锁是对于轻量级锁的优化，主要就是在加锁时，会记录线程 id ，然后设置 `biased_lock` 为 1 说明是轻量级锁，此时如果发生**重入**，那么就先判断是否是同一个线程，如果是同一个线程就添加一个锁记录，不需要 CAS 操作

## JMM ( Java 内存模型）

JMM(Java Memory Model) Java 内存模型，定义了共享内存中**多线程程序的读写操作**的行为规范，通过这些规则对内存的读写操作保证正确性

**分块**：JMM 把内存分为 **工作内存** 和 **主内存** ，工作内存是线程私有，主内存是所有线程共享
![[Java - 多线程-10.png|525]]


线程和线程相互隔离，线程跟线程交互需要通过主内存进行

---
## CAS 是什么

CAS（Compare And Swap 比较再交换）保证了线程在无锁状态下的原子性
### 数据交换流程（自旋锁）

当主内存值为 V=100，线程的工作内存中获取的主内存值A=100，此时需要将 A+1 也就是结果值 B= 101，这时把这个数据B再同步给主内存V
如果这是有其他内存也保存了 V 的值为100，但此时 V的值为101 ，不匹配，这时就会进行**自旋**，也就是这时就会先获取这个101的值，然后再进行其他的操作，

![[Java - 多线程-11.png|675]]

### CAS 原理

CAS 是一个**乐观锁**的原理，在底层调用的是 Unsafe 类中系统实现的方法 

**乐观锁与悲观锁**：
- 乐观锁：最乐观的估计，不怕其他线程修改锁，就算改了就重试就行
- 悲观锁：得防着其他锁修改数据，上锁之后其他的线程不能更改数据

## 谈谈对 volatile 的理解

一旦一个共享变量（类的成员变量，类的静态成员变量），被 volatile 修饰之后，会得到 **保证线程可见性** 和 **禁止指令重排序** 两个特性
### 1. 保证线程间的可见性

用 **volatile** 修饰共享变量，能够防止编译器的优化，让一个线程对共享变量的修改对另一个线程可见 

#### JIT（及时编译器）

这是 JVM 虚拟机中的一个即时编译器，他会对代码进行一个优化，比如下图

![[Java - 多线程-12.png|550]]

这里因为变量判断的次数太多，所以对代码进行了优化。

**解决方法一：** 在程序运行时加入参数 `-Xint` 表示禁即时编译器（不推荐，因为其他可能需要）
**解决方法二：** 在共享的变量（类似于 `stop`）加上 `volatile` 变量告诉 JIT 不要对 `volatile` 修饰的变量进行优化

### 2. 禁止指令重排序

`volatile` 修饰的共享变量会在读写共享变量是加入不同的**屏障**，阻止其他读写操作越过屏障，从而达到阻止**重排序**

![[Java - 多线程-13.png|750]]

这个代码中，写操作加的屏障是为了阻止上方的其他写操作在还没执行前先执行屏障下方的写操作，读操作正好相反，这样就可以解决因为指令重排序出现的读写顺序问题，而这个操作尽量要少因为指令重排序是对 CPU 的运行的优化，所以可以使用以下技巧

- 写变量让 `volatile` 修饰的变量在代码的最后位置
- 读变量让 `volatile` 修饰的变量在代码的开始的位置

---
## AQS 是什么

全称(AbstractQueuedSynchronizer) 抽象队列同步器。他是构建锁或者其他同步组件的**基础框架**

### 与 Synchronized 的区别

| synchronized    | AOS           |
| --------------- | ------------- |
| 关键字，C++ 语言实现    | Java 语言实现     |
| 悲观锁，自动释放锁       | 悲观锁，手动开启和关闭   |
| 锁竞争激烈都是重量级锁，性能差 | 提供了多种解决锁竞争的方案 |
AQS 常见实现类
- `ReentrantLock` 阻塞式锁
- `Semaphore` 信号量
- `CountDownLatch` 倒计时锁

### 工作机制

AQS 内部维护了一个双向的先进先出的队列，用于存储排队的线程
并且内部使用一个 state 的状态属性来控制是否获得锁
![[Java - 多线程-14.png|675]]

在这里，AQS 的工作机制是
1. 线程0 持有锁，然后设定 state 为 1 ，表示当前有锁
2. 线程1 尝试获取锁失败，进入 AQS 的先进先出队列，进行等待
3. 线程2 也尝试获取失败，也进入队列
4. 如果线程0 执行结束释放锁，此时从 FIFO 队列中唤醒 head 也就是等待最久的元素线程 1 获取锁

### 保证原子性

AQS 保证多个线程共同修改时的原子性的方法是使用 CAS 操作

![[Java - 多线程-15.png|400]]
![[Java - 多线程-16.png|450]]

当使用 CAS 操作后，如果线程4没有获取到锁，进入队列

### AQS 是否公平锁

AQS 可以实现公平锁和非公平锁，在不同的实现里公平锁和非公平锁都可以实现

- 当新的线程来和旧的线程争抢资源，就是非公平锁
- 当新的线程直接进入 FIFO 队列中等待，只让线程队列中 head 指向的线程进行获取锁，则是公平锁

---
## ReentrantLock 的实现原理

ReentrantLock是可重入锁，对于 synchronized 的特点有以下
- 可中断
- 可以设置超时时间
- 可以设置公平锁
- 支持多个条件变量
- 与 synchronized 一样都支持重入
### 实现原理

主要利用 CAS + AQS队列 实现。支持公平锁和非公平锁。

#### 构造方法

ReentrantLock 可以选择一个参数用于是否为公平锁，在构造方法中有这两个方法
```java
public ReentrantLock() {  
    sync = new NonfairSync();  
}  
  
 public ReentrantLock(boolean fair) {  
    sync = fair ? new FairSync() : new NonfairSync();  
}
```

源码中表示如果直接创建就是非公平锁的构造方法，如果传入变量 true 则是公平锁的构造方法。

#### 工作流程

![[Java - 多线程-17.png|850]]

非公平锁的工作模式主要是上图
- 先来的线程如果状态 state 为0，进来的多个线程会用 CAS 操作尝试获取锁并修改 state 为1
- 如果成功获取，则 exclusiveOwnerThread 的值设置为成功获得的锁（类似于重量级锁的 Owner ）
- 如果获取失败，则进入队列中的尾部
- 当线程执行完释放锁，exclusiveOwnerThread 为 null 的时候，state 值设置为0，这时会对新进来的线程与队列中的锁进行竞争（这时非公平锁的体现）

---
## synchronized 和 Lock 有什么区别

### 语法层面

`synchronized` 是关键字，源码在 JVM 中由 C++ 实现

`Lock` 是接口，源码在 JDK 中，用 Java 语言

- 使用 synchronized 退出代码块的时候会自动释放，而 Lock 使用时需要手动调用 unlock 方法
### 功能层面

二者都是悲观锁，都具备互斥、同步、重入等功能

Lock 提供比 synchronized 不具备的例如公平锁，可打断，可超时，多条件变量等功能。

Lock 有适合不同的场景实现比如 ReentrantLock、ReentrantReadWriteLock（读写锁）

### 性能层面

在没有竞争时， synchronized 做了很多的优化，如偏向锁和轻量级锁，性能较好

在竞争激烈的时候， Lock 的实现通常有更好的性能

## 死锁的问题

### 死锁的产生条件是什么

一个线程需要同时获取多个锁时容易发生死锁，比如 A 线程需要锁1 ，然后需要锁2，而  B 线程需要锁2，然后释放锁1，这时就会发生死锁

### 如何进行死锁诊断 

可以使用jdk 自带工具 : jps 和 jstack

- jps：输出 JVM 中运行的进程状态信息
- jstack：查看 Java 进程内线程的堆栈信息

![[Java - 多线程-19.png|975]]

### 如何进行死锁解决和避免

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

---
## 对 ConcurrentHashMap 的理解

是一种线程安全的高效的 Map 集合

### 底层数据结构

其底层的数据结构主要分为 jdk1.7 以前和 jdk1.8 以后的区别

**jdk1.7**
底层采用的是分段数组  + 链表实现

![[Java - 多线程-20.png|725]]

在这个图例中，当请求进过哈希函数计算后得到 Segment 数组中的下标是，加上 ReentrantLock 锁，然后通过下标的值定位 hashEntry 数组的下标然后进行读写，在这个过程中
- Segment 数组下标对应一个  hashEntry 数组的下标
- Segment 数组是不可以进行扩容的
- 当多个请求时因为有锁的存在所以性能很低

**jdk1.8**
底层采用的是和 HashMap1.8 一样的 数组+链表/红黑二叉树 的结构，并且采用 CAS 和 Synchronized 来保证并发安全的进行

![[Java - 多线程-21.png|700]]

主要的特点有两点
- 使用 CAS 自旋操作控制数组节点的添加，保障数据安全
- synchronized 只会锁住链表或者红黑树的首节点，只要 hash 不冲突，就不会发生并发问题，效率得到提高

### 加锁方式不同

jdk1.7 采用的是 Segment 分段锁，底层是采用的 ReentrantLock  加锁
jdk1.8 采用的是 CAS 添加新节点，Synchronized 控制链表和红黑树的首节点，比 Segment 分段锁的粒度更低性能更好

---
## 导致并发程序出问题的根本原因是什么

==Java 程序怎么保证多线程的执行安全==

Java 并发编程的三大特征是
- 可见性 - 可以使用 volatile 、synchronized、lock 解决
- 有序性 - 可以使用 volatile 解决
- 原子性 - 可以使用 synchronized、lock 解决


---

# 线程池

## 说一下线程池的核心参数（线程池的执行原理理解）

### 核心参数

以下是构造函数

```java
public ThreadPoolExecutor(int corePoolSize,  
                          int maximumPoolSize,  
                          long keepAliveTime,  
                          TimeUnit unit,  
                          BlockingQueue<Runnable> workQueue,  
                          RejectedExecutionHandler handler) {  
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,Executors.defaultThreadFactory(), handler);  
}
```

`corePoolSize` 核心线程数目
`maximumPoolSize` 最大线程数目 = （核心线程数 + 救济线程最大数目）
`keepAliveTime` 生存时间 - 救急线程生存时间，生存时间内没有新任务，此线程资源就会被释放
`unti` 时间单位 - 救急线程的生存时间单位，如秒、毫秒等
`workQueue`- 当没有空闲的核心线程时，新来的任务就会加入到此队列排队，队列满时会创建救 急线程执行任务
`threadFactory` 线程工厂 - 可以定制线程对象的创建，例如线程名字，是否是守护线程等
`handler` 拒绝策略 - 当所有线程都在繁忙，且 workQueue 也满时，会触发拒绝策略 

### 线程池执行原理

![[Java - 多线程-22.png|750]]

首先提交任务，**判断核心线程**是否满了，如果没满直接执行，满了**判断阻塞队列**是否满，如果没满放入阻塞队列，满了则判断是否达到了**最大线程数**，如果没有达到则创建救急线程执行任务，如果线程数也达到了最大线程数，则开始执行**拒绝策略**。
如果救急线程任务执行完成，会检查队列是否还有其他任务，如果有的话直接继续执行，核心线程也是如此

一共有四种救急策略：
- `AbortPolicy`：（默认）直接抛出异常
- `CallerRunsPolicy`：用调用者所在线程执行
- `DiscardOldestPolicy`：丢失阻塞队列中最靠前的任务，然后执行
- `DiscardPolicy`：直接**丢弃**任务


---
## 线程池中有那些常见的阻塞队列

`workQueue` 常见的阻塞队列有以下这些
1. **`ArrayBlockingQueue`**：基于数组的有界阻塞队列，FIFO
2. **`LinkedBlockingQueue`**：基于链表(单向)结构的有界阻塞队列，FIFO（一般开发中使用这个，但需设置边界）
3. `DelayedWorkQueue`：是一个优先级队列，可以保证每次出队的任务都是当前队列中执行时间最靠前的
4. `SynchronousQueue`：不存储元素的阻塞队列，每个插入操作都必须等待一个移除操作

### **`ArrayBlockingQueue`** 与 **`LinkedBlockingQueue`** 的区别


|       | **`LinkedBlockingQueue`**                                      | **`ArrayBlockingQueue`**                           |
| ----- | -------------------------------------------------------------- | -------------------------------------------------- |
| 关于有界  | 默认无界，支持有界                                                      | 强制必须有界                                             |
| 底层结构  | 底层是链表                                                          | 底层是数组                                              |
| 初始化方式 | 是懒惰的，创建节点时添加数据                                                 | 会提前初始化 Node 数组                                     |
| 添加方式  | 入队时会产生新的 Node                                                  | Node 需要提前创建好                                       |
| 锁     | 会有两把锁（头和尾）<br>![[Java - 多线程-23.png\|400]]<br>所以可以一边入队一边出队，效率较高 | 只有一把锁<br>![[Java - 多线程-24.png\|337]]<br>每次都会锁住整个队列 |

---

## 如何确定核心线程数

要确定核心线程数首先要确定任务的类型，主要分为 IO密集型任务 和 CPU密集型任务

N 为计算机的核心数量

**IO密集型任务**：文件读写、DB读写、网络请求等 
设置核心线程数大小为 2N+1

**CPU密集型任务**：计算型代码、BitMap转换、Gson转换等
设置核心线程数大小为 N+1

可能出现的情况有
1. 高并发、任务执行时间短 -> N+1，减少线程上下文切换
2. 并发不高、任务执行时间长
	- IO密集型任务 -> 2N + 1
	- 计算密集型任务 -> N + 1
3. 并发高、业务执行时间长 -> 因从架构进行分析。
## 线程池的种类有哪些

可以使用 `java.util.concurrent.Executors` 这个类提供了很多创建连接池的静态方法

**1. 创建固定大小线程池** 
可以使用 `newFixedThreadPool` 创建定长大小线程池
可控制线程最大并发数，超出线程在队列中等待

**2. 单线程化的线程池**
`newSingleThreadExecutor` 可以创建一个单线程化线程池
可以按照顺序使用的线程池执行

**3.可缓存线程池**
`newCachedThreadPool` 创建一个可缓存的线程池，如果线程池超过处理需要，就可以灵活回收线程，如果没有可以回收的就创建线程
适用于任务数比较密集但是单个任务执行时间短

**4. 提供延迟和周期执行功能的线程池**
`newScheduledThreadPool` 可以延迟执行任务的线程池
支持定时和周期性的任务执行
## 为什么不建议使用 Executors 创建线程池

因为通过 `Executors` 创建的话，有可能会因为 Interger.MAX_VALUE 这个参数造成 OOM 内存溢出
一般采用 `ThreadPoolExecutor` 方式，这种方式可以明确自己参数内容 

可能得弊端有
![[Java - 多线程-25.png|500]]


---

# 使用场景

## 线程池的使用场景

### CountDownLatch（有依赖关系）
又称为倒计时锁/闭锁，用来进行线程同步协作。
也就是完成的一个任务就进行 countdown() 操作，直到前置的所有任务全部完成，再把 await() 的线程开启即可。保证前置线程的任务全部完成再进行。 

- 其中构造参数用于初始化等待计数值
- await() 用来等待任务结束
- countDown() 用于完成任务后执行
![[Capture_20250508_100949.jpg|425]]
类似于这种，在代码中为
```java
package xkqq.top.thread;  
import java.util.concurrent.*;  
public class CountDownLatchTest {  
  
    public static void main(String[] args) {  
        new CountDownLatchTest().threadTest();  
    }  
  
    public static void threadTest(){  
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(  
                3,  
                5,  
                1000,  
                TimeUnit.MILLISECONDS,  
                new LinkedBlockingQueue<>(3),  
                new ThreadPoolExecutor.AbortPolicy()  
        );  
  
        CountDownLatch yellowCount1 = new CountDownLatch(2);  
        CountDownLatch yellowCount2 = new CountDownLatch(1);  
  
        CountDownLatch redCount = new CountDownLatch(2);  
  
        // 黑1  
        threadPoolExecutor.execute(() -> {  
            try {  
                System.out.println("黑1开始执行");  
                Thread.sleep(1000);  
                System.out.println("黑1执行完成");  
                yellowCount1.countDown(); // 黑1完成，计数器-1  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        });  
  
        // 黑2  
        threadPoolExecutor.execute(()->{  
            try{  
                System.out.println("黑2开始执行");  
                Thread.sleep(1000);  
                System.out.println("黑2执行完成");  
                yellowCount1.countDown();  
            }catch (InterruptedException e){  
                e.printStackTrace();  
            }  
        });  
        // 黑3  
        threadPoolExecutor.execute(()->{  
            try {  
                System.out.println("黑3开始执行");  
                Thread.sleep(1000);  
                System.out.println("黑3执行完成");  
                yellowCount2.countDown();  
            }catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        });  
  
        // 黄4  
        threadPoolExecutor.execute(()->{  
            try {  
                System.out.println("等待黑1和黑2完成");  
                yellowCount1.await();  
                System.out.println("黑1黑2执行完成开始执行黄4");  
                Thread.sleep(1000);  
                System.out.println("黄4 执行完成");  
                redCount.countDown();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        });  
        // 黄5 - 依赖于黑3  
        threadPoolExecutor.execute(() -> {  
            try {  
                System.out.println("黄5等待黑3完成");  
                yellowCount2.await(); // 等待黑3完成  
                System.out.println("黄5开始执行");  
                Thread.sleep(1000);  
                System.out.println("黄5执行完成");  
                redCount.countDown(); // 黄5完成，计数器-1  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        });  
        // 红6 - 依赖于黄4和黄5  
        threadPoolExecutor.execute(() -> {  
            try {  
                System.out.println("红6等待黄4和黄5完成");  
                redCount.await(); // 等待黄4和黄5完成  
                System.out.println("红6开始执行");  
                Thread.sleep(1000);  
                System.out.println("红6执行完成");  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        });  
        // 关闭线程池  
        threadPoolExecutor.shutdown();  
    }  
}
```

最后得到的结果是
```
黑1开始执行
黑2开始执行
黑3开始执行
黑3执行完成
等待黑1和黑2完成
黑2执行完成
黄5等待黑3完成
黄5开始执行
黑1执行完成
红6等待黄4和黄5完成
黑1黑2执行完成开始执行黄4
黄5执行完成
黄4 执行完成
红6开始执行
红6执行完成
```
完成了后一个任务对前一个任务必须完成的依赖问题

### 线程池 + Future 实现并发获取结果汇总

![[Java - 多线程-27.png|825]]

在这个里面，使用第二种线程池的方法会提高非常多的获取速度，以下是示例代码

![[Java - 多线程-28.png|700]]

所以在实际开发中，多个线程中没有依赖关系，就可以使用线程池 + Future获取数据用来提升性能

### 异步调用
当为了避免上一个任务影响下一个任务的正常进行时，可以使用 异步调用的方法。

## 如何控制某个方法允许并发访问线程的数量

可以使用 JUC 包中的一个工具类叫 Semaphore 信号量，他的底层使用的是 AQS，可以通过这个工具类限制线程数量

使用与明确有访问限制的场景，常常用于限流

Semaphore 使用步骤
- 创建 Semaphore 对象，给一个容量
- semaphore.acquire() 请求一个信号量，如果这个信号量为-1，说明没有可用的信号量请求就会被阻塞，直到其他线程释放信号量
- semaphore.release() 释放一个信号量，这时信号量数量 +1 

## 谈谈对 ThreadLocal 的理解

### 基本概念
ThreadLocal 是 Java 提供的一种解决多线程环境下线程安全问题的机制。它通过为每个线程分配**独立的变量副本**，实现线程间的数据隔离，从而避免了共享变量可能带来的并发访问冲突。

### 主要作用 
- **线程隔离**：让每个线程拥有自己独立的变量副本，避免线程安全问题
- **线程内共享**：同一线程内的不同方法可以共享这些变量，无需通过参数传递

主要作用有
- 实现资源对象的线程隔离，让每个资源对象使用各自的资源，避免发生线程安全问题
- 实现线程内的资源共享

### ThreadLocal 基本使用

- set(value) 设置值
- get() 获取值
- remove() 清除值

### 基本使用方法 
```java
// 创建ThreadLocal对象
ThreadLocal<String> threadLocal = new ThreadLocal<>();

// 设置值
threadLocal.set("当前线程的值");

// 获取值
String value = threadLocal.get();

// 清除值（避免内存泄漏）
threadLocal.remove();
```

### ThreadLocal 实现原理

#### 核心数据结构关系

**关键点**：ThreadLocal 的数据并不是存储在 ThreadLocal 实例中，而是存储在当前线程的 ThreadLocalMap 中。

1. 每个**Thread**对象中有一个`ThreadLocalMap`类型的成员变量，命名为`threadLocals`
2. **ThreadLocalMap**是ThreadLocal的内部类，它才是真正存储数据的地方
3. **ThreadLocalMap**内部维护了一个**Entry数组**，每个Entry是一个键值对
4. **Entry**的key是对ThreadLocal的弱引用，value是实际存储的值

所以存储结构是：
```
Thread → threadLocals(ThreadLocalMap) → Entry数组 → Entry(ThreadLocal作为key, 实际值作为value)
```
#### set 方法工作流程

- 首先会获取当前的线程：`Thread t = Thread.currentThread()`
- 获取当前的线程的的 ThreadLocalMap ：`ThreadLocalMap map = getMap(t)`
- 如果map存在，则将当前ThreadLocal对象作为key，值作为value存入map
- 如果map不存在，则创建一个新的ThreadLocalMap，并与线程关联
  ![[Java - 多线程-29.png|825]]
  ![[Java - 多线程-30.png|400]]

#### get方法/remove方法

当调用`threadLocal.get()`时：

1. 首先获取当前线程：`Thread t = Thread.currentThread()`
2. 获取该线程的ThreadLocalMap对象
3. 如果map存在，则以当前ThreadLocal对象为key，获取对应的Entry
4. 如果Entry存在，则返回其value值；否则执行初始化并返回默认值

关键点：**ThreadLocal实例本身作为key**，用于在ThreadLocalMap中查找对应的值。

![[Java - 多线程-31.png|600]]


### ThreadLocal 内存泄露问题

Java 对象中有四种引用类型：强引用、弱引用、软引用、虚引用

- 强引用：最普通的引用，表示一个对象处于 **有用且必须** 的状态，GC 不会回收一个强引用的对象。即使堆中的内存不足，会出现 OOM ，也不会进行回收
- 弱引用：表示一个对象**可能有用但非必须**的状态， GC 在内存区域中发现弱引用就会回收到弱引用相关联的对象，如果内存区域是否足够，发现就会回收


```java
static class ThreadLocalMap {  
    static class Entry extends WeakReference<ThreadLocal<?>> {  
        /** The value associated with this ThreadLocal. */  
        Object value;  
  
        Entry(ThreadLocal<?> k, Object v) {  
            super(k);  
            value = v;  
        }  
    }
```

ThreadLocalMap中的Entry对ThreadLocal的引用是弱引用，但对存储的value是强引用：

因为每一个 Thread 都会维护一个 ThreadLocalMap，在 ThreadLocalMap 中的 Entry 对象继承了一个 WeakReference ，这里面的 key 就是**弱引用**的 ThreadLocal 的实例，而 value 却是一个 **强引用** ，所以久而久之弱引用被回收，但强引用没有被回收，就造成内存泄露问题

当发生以下情况时会导致内存泄漏：
1. ThreadLocal对象被回收（没有强引用指向它）
2. Thread对象仍然存活（如线程池中的线程）
3. Thread持有的ThreadLocalMap中的Entry的key变为null（因为是弱引用）
4. 但Entry中的value依然被强引用，无法被回收

这样就形成了"key为null但value无法回收"的Entry，造成内存泄漏。

**解决方案**
1. **手动清理**：使用完ThreadLocal后，始终调用`remove()`方法
2. **在框架层面**：一些框架（如Spring）会自动处理ThreadLocal的清理工作
 






















































