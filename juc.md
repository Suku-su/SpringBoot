# JUC

## 什么是juc

Java.Util.Concurrent包简称JUC，它主要是负责==处理线程==，==实现多[线程通信](https://so.csdn.net/so/search?q=线程通信&spm=1001.2101.3001.7020)==、==线程安全==、==线程间高并发==的工具包

## 进程

进程即正在运行的程序，可以理解为一个==程序的实例对象==，它是==资源分配的最小单位==。在操作系统中，进程由==代码块、数据块、程序控制块PCB==三部分组成。进程的创建也能理解为PCB的创建

```java
进程状态：新建态、就绪态、运行态、阻塞态、终止态、（阻塞挂起、就绪挂起）	
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/2d3be9a46d890bfccdd131fe2310ff59.png)

##  线程

线程是轻量级的进程，它是==cpu进行调度的最小单位==，在Java中一个线程对应一个具体物理线程实体

```txt
线程状态：
          NEW：新建态，创建线程还未启动
     RUNNABLE：可运行态，包括就绪态和运行态
   TERMINATED：终止态，线程结束并回收线程资源
      BLOCKED：阻塞态，线程等待某项资源而主动放弃cpu进行阻塞态
      WAITING：无限等待，调用wait()方法线程会进入无限等待状态，等待其他线程唤醒
TIMED_WAITING：有限等待，调用sleep()方法或带参wait(t)方法，线程进入等待状态直到设置时间才被唤醒。

```

## 两者异同

关系：一个进程至少包含一个线程。
切换：进程切换消耗较多资源、线程切换消耗较少资源。
资源：进程中的资源能被其线程共享访问。



## 并发与并行

并发：一段时间内，多个进程或线程交替运行。

并行：一段时间内，多个进程或线程同时运行。

串行：一段时间内，只允许一个进程或线程运行。



## sleep() 和 wait()等方法

1）==sleep()与wait()==：
调用两个方法都能让线程进入等待状态，==前者有限等待、后者无限等待==。可以从以下4点进行区分：
来源：sleep()是==Thread==类方法、wait()是==Object==类方法。
解锁：==sleep()不会释放锁==、==wait()释放锁==。
唤醒：==sleep()等待时间到了会自动唤醒、wait()需要被其他线程调用notify()或notifyAll()唤醒==。
位置：sleep()可以在任意位置被调用，wait()只能在同步块中使用，因为这里存在一个同步关系：wait()方法必须要在notify()、notifyAll()方法之前执行，通过一个同步代码块来实现，否在该进程会错过通知永远被阻塞。
2）join()与yield()
join()可以理解为插队线程，在当前线程A内其他线程B的join()方法会让当前线程A进行阻塞态，直到线程B运行结束才会被其唤醒。
yield()可以理解为礼让线程，当前线程A调用yield()方法会让其从运行态退出，重新与其他线程争抢cpu资源。

|      | sleep                    | wait                                                         |
| ---- | ------------------------ | ------------------------------------------------------------ |
| 来源 | Thread的类方法           | Object的类方法                                               |
| 解锁 | 不会释放锁               | 会释放锁                                                     |
| 唤醒 | 时间结束，自动唤醒       | 需要notify或者notifyAll来唤醒，否则一直等待                  |
| 位置 | 可以在任意一个位置被调用 | 只能在一个同步关系里面调用，否则接受不到这个通知，就会一直等待 |



## 锁

### 1.Synchronized

Synchronized是Java提供的关键字，是一种同步锁（对方法或者代码块中存在共享数据的操作），同步锁可以是==任意对象==，主要用于==实现线程同步操作，保证线程安全==。

修饰代码块：被修饰的代码块被叫为==同步代码块==，用Synchronized() {}进行定义，作用范围为==大括号区间内==。
修饰普通方法：普通成员方法添加Synchronized关键字修饰可以变为==同步方法==，其作用范围为==同一对象==。只能有一个线程调用同一对象的同步方法，其余想要调用此同步方法的线程将会被阻塞。即==同一对象的同步方法争抢一把锁，不同对象则有多把锁==。
修饰静态方法：静态方法添加Synchronized关键字修饰可以变为==静态同步方法==，其作用范围为==整个类==。所有该类的实例变量争抢同一把锁。
同步方法只能被显示的设置、子类继承父类的同步方法，在默认情况下不是同步的。

### 2.lock

Lock接口是JUC提供的一种更加灵活，功能更为强大的同步锁框架。其有多个功能强大的接口和实现类，例如Future（未来任务接口）、Callable（具有返回值的线程接口）、*Executor（线程池接口）、ReentrantLock类（可重入锁）等等。

```java

		//基本用法
		Lock() lock = new ReentrantLock();
		lock.lock();
        try {
        	// 处理逻辑
        } catch(Exception e) {
        	// 处理异常
        } finally {
        	lock.unlock();
        }

```





3.异同

|          | Synchronized                                                 | lock                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 实现     | Java提供的内置关键字                                         | JDK的接口                                                    |
| 性能     | 旧版本中Synchronized同步锁性能很低，新版在进行相关锁优化后与ReentrantLock性能大致相同 |                                                              |
| 中断     | 当持有锁线程长期不释放锁的情况下，等待Synchronized同步锁的线程不能放弃等待，也不能响应中断。 | ReentrantLock可以响应中断，放弃等待，去处理其他事情。调用interrupt()方法可以设置线程中断标记，并且中断该线程。如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException。如果该线程没有抛出异常语句，通过interrupted()也可以中断线程。 |
| 公平     | 非公平锁                                                     | 可以通过带参构造使其成为公平锁                               |
| 等待条件 | 而Synchronized同步锁只能绑定一个等待条件，通过wait()方法和notify()/notifyAll()方法实现等待与通知。 | Lock实例对象可以通过newCondition()获取与当前锁绑定的条件对象，一个ReentrantLock对象可以绑定多个条件。通过await()方法和signal()/signalAll方法实现等待和通知， |
| 释放     | 自动释放锁                                                   | 手动释放锁                                                   |

## 常见队列

1. 生产者消费者问题
    1）生产者：生产资源供消费者使用，如果资源区已满，则需等待消费者消费。满则等待
    2）消费者：消费生产者生产的资源，若资源区为空，则需等待生产者生产。空则等待
    3）资源区/临界区：存放有限资源的区域。

2. 读者写者问题
    1）读者：读操作、多个读者线程可以共享访问，且读线程与写线程之间需互斥访问。
    2）写者：写操作、多个写线程需要互斥访问，且读线程与写线程之间需互斥访问。
    3）分析：读读共享、读写互斥、写写互斥。
    4）ReadWriteLock读写锁，通过readLock() 和 writeLock() 来实现读写分离。

3. 哲学家问题
    1）死锁形成4个条件：资源共享、不可剥夺、请求与保持、循环等待。
    2）预防死锁：打破死锁形成四条件。
    3）避免死锁：银行家算法（Allocated、Need、Max、Left）
    4）死锁判断与解决：对资源分配图进行简化分析。

  ## 死锁的四个必要条件

  1. **互斥条件（Mutual Exclusion）**
     - 资源一次只能被一个线程占用。
     - 例如：Java 中的 `synchronized` 锁或 `ReentrantLock` 具有排他性。
  2. **请求与保持条件（Hold and Wait）**
     - 线程在持有至少一个资源的同时，请求其他线程持有的资源。
     - 例如：线程 A 持有锁 X 后，尝试获取锁 Y，但不释放 X。
  3. **不可剥夺条件（No Preemption）**
     - 资源只能由持有它的线程主动释放，不能被强制抢占。
     - 例如：Java 的锁机制不支持外部线程强制释放锁。
  4. **循环等待条件（Circular Wait）**
     - 存在一组等待线程，每个线程都在等待下一个线程占有的资源，形成环路。
     - 例如：线程 A 等待线程 B 的资源，线程 B 等待线程 A 的资源。

解决死锁的方法；

1、破坏循环等待的顺序

ex：统一加锁的顺序

```java
// ✅ 解决方案：统一锁的获取顺序（先 lock1 后 lock2）
public class FixedLockOrder {
    private static final Object lock1 = new Object();
    private static final Object lock2 = new Object();

    public static void main(String[] args) {
        Thread threadA = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("ThreadA: Holding lock1...");
                synchronized (lock2) {  // 固定顺序：先 lock1 后 lock2
                    System.out.println("ThreadA: Acquired lock2!");
                }
            }
        });

        Thread threadB = new Thread(() -> {
            synchronized (lock1) {  // 同样先获取 lock1
                System.out.println("ThreadB: Holding lock1...");
                synchronized (lock2) {
                    System.out.println("ThreadB: Acquired lock2!");
                }
            }
        });

        threadA.start();
        threadB.start();
    }
}
```

2、破坏请求与保持的条件

ex：一次性申请所有需要的资源，避免持有部分资源时再申请其他资源。

```java
// ✅ 解决方案：通过外层锁保证原子性
public class AtomicLocks {
    private static final Object lock1 = new Object();
    private static final Object lock2 = new Object();
    private static final Object globalLock = new Object(); // 全局锁

    public static void main(String[] args) {
        Thread threadA = new Thread(() -> {
            synchronized (globalLock) {
                synchronized (lock1) {
                    System.out.println("ThreadA: Holding lock1...");
                    synchronized (lock2) {
                        System.out.println("ThreadA: Acquired both locks!");
                    }
                }
            }
        });

        // ThreadB 使用相同的全局锁顺序
    }
}
```

3、破坏不可剥夺条件

ex：使用可超时的锁，当获取锁失败时主动释放已持有的锁

```java
import java.util.concurrent.locks.*;

// ✅ 解决方案：使用 ReentrantLock.tryLock()
public class TimeoutLocks {
    private static final Lock lock1 = new ReentrantLock();
    private static final Lock lock2 = new ReentrantLock();

    public static void main(String[] args) {
        Runnable task = () -> {
            while (true) {
                if (lock1.tryLock()) {       // 尝试获取锁1
                    try {
                        if (lock2.tryLock(500, TimeUnit.MILLISECONDS)) {  // 带超时
                            try {
                                System.out.println(Thread.currentThread().getName() + ": Got both locks!");
                                return;  // 成功获取，退出循环
                            } finally {
                                lock2.unlock();
                            }
                        }
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    } finally {
                        lock1.unlock();  // 释放锁1让其他线程有机会
                    }
                }
                // 短暂休眠避免活锁
                try { Thread.sleep(100); } catch (InterruptedException e) {}
            }
        };

        new Thread(task, "Thread-A").start();
        new Thread(task, "Thread-B").start();
    }
}
```

## 集合的线程安全

### 1.并发集合ArrayList

ArrayList底层采用可调整大小的数组来实现，支持随机有序访问，可重复添加元素。其add() 方法没有添加同步关键字，在进行多线程操作时会出现并发修改异常。ConcurrentModificationException



### 2.Vector

Vector列表中的所有方法都有synchronized同步关键字修饰，是线程安全的，不支持并发操作，效率较低。



```java
public synchronized boolean add(E e){}

```



### 3.Collections

可以通过Collections工具类将ArrayList<>() 、HashSet<>() 转换为线程安全的集合。

```java
Collections.synchronizedList(new ArrayList<T> ());
Collections.synchronizedSet(new HashSet<T> ());
Collections.synchronizedMap(new HashMap<K,V> ());

```



### 4.JUC.CopyOnWriteArrayList

JUC提供的线程安全的集合，底层原理涉及写时复制技术。即：在对集合进行写操作时，会进行copy，复制一个集合副本对象，并执行写操作，完成之后再将原引用指向此对象。而再进行读操作则支持并发。	

```java
List<String> list = new CopyOnWriteArrayList<>();
Set<String> set = new CopyOnWriteArraySet<>();
Map<String,String> map = new ConcurrentHashMap<>();
// 没有CopyOnWriteMap()对象;

```



## 线程池

用于创建线程并对线程生命周期进行管理，减少线程开销、增加线程复用性



```java
public ThreadPoolExecutor(int corePoolSize,	// 核心线程数
                         int maximumPoolSize, // 最大线程数
                         long keepAliveTime, // 临时线程存活时间
                         TimeUnit unit, // 时间单位
                         BlockingQueue<Runnable> workQueue, // 阻塞队列
                         ThreadFactory threadFactory, // 线程创建工厂
                         RejectedExecutionHandler handler //拒绝策略 
                         ){
    if (corePoolSize < 0 ||
      maximumPoolSize <= 0 ||
      maximumPoolSize < corePoolSize ||
     keepAliveTime < 0)
      throw new IllegalArgumentException();
      if (workQueue == null || threadFactory == null || handler == null)
         throw new NullPointerException();
       this.corePoolSize = corePoolSize;
       this.maximumPoolSize = maximumPoolSize;
       this.workQueue = workQueue;
       this.keepAliveTime = unit.toNanos(keepAliveTime);
       this.threadFactory = threadFactory;
       this.handler = handler;
}

```



## 1. 公平锁与非公平锁

1）公平锁：FIFO即线程根据先后顺序依次获取CPU时间片。当一个线程拥有公平锁时，其余需要获取该锁的线程必须依次等待。
2）非公平锁：线程之间可以相互争抢CPU时间片，没有先来后到之分。通常可以设置线程优先级来建议操作系统调用优先级更高的线程。（但是具体如何调用还得根据操作系统的具体实现。）
3）ReentrantLock()对象可以通过参数修改锁为公平锁，而Sync则不能。

## 2.乐观锁与悲观锁

1）悲观锁：悲观并发策略总认为如果不采取同步措施、任务总会出现问题；因此无论共享数据的竞争是否发生，都会进行加锁操作。用于线程调度需要操作系统从用户态切换为核心态，频繁地切换、调度和唤醒线程会带来很大的开销。
2）乐观锁：基于冲突检验的乐观并发策略，即线程在访问共享资源时，乐观地认为不会有其他线程来争抢该资源。即先不上锁进行访问，如果操作成功，则直接返回结果；如果操作失败，表示有其他线程想要访问该共享资源，此时应该采取补救措施来保证线程安全。例如通过不断重试、自旋（CAS）操作避免进入阻塞态，减少系统线程切换开销。其中，保证冲突检测操作的原子性非常重要，但随着硬件的发展也得到了解决（CAS等指令）。



## 3. 可重入锁与不可重入锁

1）可重入锁：一个线程可以重复获取已拥有的锁，而不会发生死锁现象。这一功能主要是通过锁计数器来实现的，即获取一次锁，计数器+1，释放一次锁，计数器-1，直到计数器为0。Sync和ReentrantLock都属于可重入锁。
2）不可重入锁：一个线程不能重复获取已拥有的锁，必须释放该锁之后才能继续获取该锁，使用不可重入锁可能会引发死锁问题。



## 4.共享锁与独占锁

1）共享锁：又称读锁（S锁），表示多个线程可以同时访问共享资源而不会出现线程安全问题。如果只有读操作，无论是单线程还是多线程环境下都是安全的。读读共享
2）排他锁：又称独占锁、写锁（X锁），一次只允许一个线程进行写操作。写写互斥、读写互斥
3）读写锁：ReadWriteLock 管理了读锁和写锁。读写锁比互斥锁允许对于共享数据更大程度的并发。每次只能有一个写线程，但是同时可以有多个线程并发地读数据。ReadWriteLock 适用于读多写少的并发情况。读写锁一般适用于读多写少的场景

```java
	public interface ReadWriteLock {
		//返回读锁
	    Lock readLock();
	    //返回写锁
	    Lock writeLock();
	}
	ReadWriteLock rwl = new ReentrantReadWriteLock();
	// 获取读锁
	Lock readLock = rwl.readLock();
	readLock.lock();
	readLock.unlock();
	// 获取写锁
	Lock writeLock = rwl.writeLock();
	writeLock.lock();
	writeLock.unlock();

```

## 5.锁升级与锁降级

锁升级是指读锁升级为写锁。

锁降级是指写锁降级为读锁。

在Java中只允许写锁降级为读锁，而不允许读锁升级为写锁。即写线程在写的过程中，可以降级为读线程与其他线程一起进行读操作。



## 6. 锁的4种状态









# 网站

```
https://blog.csdn.net/weixin_67796933/article/details/142352410?ops_request_misc=elastic_search_misc&request_id=9ac4f224830df99e75d346a8cd6c8ddb&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-142352410-null-null.142^v102^pc_search_result_base3&utm_term=juc&spm=1018.2226.3001.4187
```

