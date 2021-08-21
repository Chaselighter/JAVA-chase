# java内存模型

## 物理模型

- cpu
- cpu寄存器
- cpu高速缓存
- 主内存

## 交互方式

- lock
- unlock
- Read:读取主内存变量
- load：载入主内存变量到本地内存
- use：对变量进行操作
- assign：赋值
- store：存储变量到主内存
- write：写入主内存吧

## 规则

- ### 重排序

  - 编译器重排序
  - 处理器乱序执行
  - 缓存会改变写入提交到主内存变量的次序

- ### 内存可见性

  - jvm内存模型规定了jvm变量对其他线程可见的最小保证
  - 

- ### happens-before

  - 

## 规范

- 所有变量都存在主内存值
- 每个线程都有自己的私有本地内存，存储了该对象的读写变量的副本
- 线程对变量的操作必须在本地内存进行
- 不同线程之间的本地内存变量不共享



# 并发三要素

## 原子性

### 对象类型

- 对象地址原子读写，线程安全
- 并发读不可变状态，线程安全
- 并发读可变状态，线程不安全

### 基本类型

- int，char读写，线程安全
- long，double高低位，非线程安全
- i++组合操作，非线程安全

### 原子操作

- 指令，小对象（8字节以内，int，char）
  - Guaranteed atomic operations。基本内存读写操作，一般来说，读写一个cache line 中的数据原子的
  - bus lock：使用LOCK#信号和指令的lock前缀，进行原子操作的cpu会在bus上assert一个LOCK#信号其他cpu都会被block
  - cache lock：利用缓存一致性协议MSEI来实现。如果要访问的内存区域已经在当前cpu的缓存中就会遵守这个协议来实现原子操作，否则会锁总线
  - CAS（三个值，变量引用；期望变量值，设置变量值）如果期望值等于当前值就set，否则自旋
    - 问题
      - ABA
      - 循环时间长
      - 只能保证一个共享原子变量
  - 

- 大对象
  - 加锁
  - copy on write
    - 指定指向大对象的指针，先把对象copy一份，在对象副本上进行修改，最后更改指针指向
- 多对象，本质上进行一次事务操作
  - 加锁，不过要注意死锁的发生
  - 写日志：innodb的undolog，redolog保证原子性和一致性
  - 事务内存

## 可见性

- ## final

  - 初始化字段确保可见性

- ## volatile

  - 读写volatile字段确保可见性

  - 阻止指令重排

  - 体现DCL

  - 不能用于符合操作，和变量值基于当前值

  - 无原子性，只有有序和可见

  - ### 原理：

    - 观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令。
    - lock前缀指令实际上相当于一个内存屏障
      - 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
      - 它会强制将对缓存的修改操作立即写入主存
      - 如果是写操作，它会导致其他CPU中对应的缓存行无效

- ## synchronized

  - 监视器锁：

    - 使用moniorEnter/monitorExit来同步代码块
    - ACC_SYNCHRONIZED同步方法

  - 同步块内读写字段确保可见性

    

- ## happen before

  - 遵守happen before次序可见性

## 有序性

- ## happen before法则

  - 程序次序法则
    - 如果
  - 监视器法则
    - 监视器锁的解锁happens before于后续对同一监视器的加锁
  - volatile变量法则
    - 对于volatile域的写入操作happens before 于后续对同一变量的读操作
  - 线程启动法则
  - 线程终结法则
  - 中断法则
  - 终结法则
  - 传递性

## 线程安全策略

- 不可变类，如果一个类初始化后，所有属性和类都是final不可变的
- 线程栈内使用
  - 方法内局部变量
  - 线程内参数传递
  - ThreadLocal持有
- 同步锁
  - syn串行化
- CAS
  - 循环设置新值，如果旧值变化，则重设，乐观并发
  - 缺点
    - ABA

# 线程和进程

## 线程：

是cpu调度和指派的基本单元

## 线程间通信

## 线程死锁

### 死锁：

死锁是指两个以上的线程互相等待对方持有的资源，导致无限阻塞

### 死锁条件

- 互斥：资源只允许一个线程持有
- 请求并持有：一个线程持有资源，还要请求其他资源
- 不可剥夺：只能自己释放，不可剥夺
- 循环等待：线程形成资源循环等待

### 死锁避免和处理

避免以上四个条件同时成立

## 线程五大状态

- 新建状态
- 就绪状态
- 运行状态
- 阻塞状态
- 死亡状态

# 线程同步控制

**引用**

[[从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)



## AQS（AbstractQueuedSynchronized）

- 通过对一个原子类的CAS操作争夺锁的控制权，暂未获得锁的线程被维护在一个FIFO队列中
- 每个线程都在自旋询问自己的前序节点是否执行完毕
- 为了减少消耗，会标记前序节点，让前序节点释放资源后通知自己
- 优点：相比于synchronized有更多特性
- 解决多线程访问共享资源的同步管理框架
- 线程同步类：ReentrantLock、ReentrantReadWriteLock、CountDownLatch、CyclicBarrier、Semaphore 
- 线程池：ThreadPoolExecutor、ScheduledThreadPoolExecutor
- 线程安全：ConcurrentHashMap、LinkedBlockingQueue、CopyOnWriteArrayList

## 原理：

<img src="/Users/liujianqiang/Documents/java并发编程/AQS源码.jpg" alt="AQS源码" style="zoom:75%;" />

- getState（）
- setState（）
- compreAndSetState
- 这三个方法搭配 volatile变量int变量state，让每个方法都变为原子操作。
- 线程同步队列的维护工作（入队，出队，唤醒，状态）AQS抽象出来了，实现同步器线程要实现的方法有
  - isHeldExclusively（）:该线程是否独占资源，只有用到condition时候才会去用到它
  - tryAcquire(int)：独占方式，尝试获取资源
  - tryRelease(int)：独占方式，尝试释放资源
  - tryAcquireShared(int)：共享方式。
  - tryReleaseShared(int)：共享方式

## 独占模式源码

### 获取资源

- tryaquire（）
- 失败addworker进入队列
- 进入同步队列，开始等待获取锁
  - 先拿到前驱节点，如果是head，尝试竞争下资源，成功返回
  - 拿到锁资源后执行出队列
  - 获取锁失败后先判断是否可以沉睡，条件就是前序节点为signal,每个节点通过多次自旋找到第一个有效的前序节点（waitstatus<0）并把它状态cas为signal，然后等待前序节点唤醒自己

### 释放资源

## 线程同步类

- **ReentrantLock**

  - 特性

    - |          | ReentrantLock                  | synchronized     |
      | -------- | ------------------------------ | ---------------- |
      | 原理     | AQS                            | 监视器锁         |
      | 灵活性   | 支持响应中断，超时，尝试获取锁 | 不灵活           |
      | 释放形式 | 必须unlock                     | jvm自动释放      |
      | 锁类型   | 公平/非公平                    | 非公平           |
      | 条件队列 | 可关联多个条件队列             | 关联一个条件队列 |
      | 可重入性 | 可重入                         | 可重入           |

      

- ReadAndWriteLock
  - 一个ReentrantReadWriteLock包含有一个读锁一个写锁
  - 获取到写锁的线程可以再次获得读锁
  - 获取到读锁的线程可以再次获得读锁
  - 那么一个资源变量status如何表达读锁和写锁两种资源语义呢？答案就是对这个整型变量按位切割，高16位表示读，低16位表示写，每次读锁进入，高16位加一，每次写锁重入，低16位加一 

- Condition

- Semaphore
  - 使用AQS同步状态来保存信号量的当前计数。tryRelease会增加计数，acquireShared会减少计数。

- CountDownLatch：
  - count为资源数，所有线程都release才能跑通，status全变为0

- CyclicBarrier
  - 所有线程在condition中park，等所有线程到齐，notifyAll

- LocksSupport



# 线程池

**引用：**

[java线程池实现原理及其在美团业务中的实践](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)

## 线程池定义？优劣

- 线程池是指在初始化一个多线程应用的过程中创建一个线程集合，

- 线程池的容量取决于可用内存的数量和应用程序的要求

- 降低资源消耗，然后在**执行任务的时候重用这些线程而不是新创建线程，重复利用已经创建的线程**
- 提高响应速度，不需要创建线程，直接执行任务
- 便于管理，统一分配调控优化‘
- 通过ThreadExcutorPool创建线程池

## 生命周期

- running：接受任务，处理任务状态
- shutdown：不接受任务，处理完任务队列中已接任务
- stop：不接受任务，不处理任务队列，执行中的也中断
- tidying：所有任务都被终止，工作线程为0，马上进入terminated状态
- terminated：执行完毕

### 原理：

线程池内部使用一个**原子**变量维护两个值：

- 运行状态(runState)

- 线程数量 (workerCount)。

- 在具体实现中，线程池将运行状态(runState)、线程数量 (workerCount)两个关键参数的维护放在了一起

- 高3位保存runState，低29位保存workerCount，两个变量之间互不干扰（**为什么这样做？？？**）

  ```java
  private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
  private static int runStateOf(int c)     { return c & ~CAPACITY; } //计算当前运行状态
  private static int workerCountOf(int c)  { return c & CAPACITY; }  //计算当前线程数量
  private static int ctlOf(int rs, int wc) { return rs | wc; }   //通过状态和线程数生成ctl
  
  ```

  

## 线程池如何管理线程，原理

![ThreaExecutorPool原理](/Users/liujianqiang/Documents/java并发编程/ThreaExecutorPool原理.jpg)

### 模型理解：

生产者和消费者模型，将任务和线程分开管理，线程池的任务有两部分

- 管理线程，充当消费者部分

  - ```java
    private final class Worker extends AbstractQueuedSynchronizer implements Runnable{
        final Thread thread;//Worker持有的线程
        Runnable firstTask;//初始化的任务，可以为null
    }
    ```

  - worker持有一个线程，初始化执行的第一个任务，可有可无

  - worker生命周期由线程池使用一张hash表去持有线程的引用

  - worker继承AQS，实现独占锁的功能，不可重入

    - lock方法一旦获取锁，表示线程在执行任务
    - 如果不是独占锁，说明空闲，可以清除
    - 处理线程前先trylock判断线程是否在执行任务

  - addWorker方法有两个参数：firstTask、core。

  - 线程池中线程的回收销毁依赖于jvm，**线程池做的工作是根据线程池的状态维护一定线程的引用，防止被jvm回收**。

  - worker被创建会不断轮训，获取任务去执行，核心线程可以无限等待，非核心线限时等待（keepAliveTime）

  - 当Worker无法获取到任务，也就是获取的任务为空时，循环会结束，Worker会主动消除自身在线程池内的引用。

    ```java
    try {
      while (task != null || (task = getTask()) != null) {
        //执行任务
      }
    } finally {
      processWorkerExit(w, completedAbruptly);//获取不到任务时，主动回收自己
    }
    ```

  - **Worker线程执行任务runWorker方法**

    - while循环不断getTask（）
    - getTask（）从阻塞队列获取任务
    - 如果线程池正在停止，要保证线程是中断在状态，否则保持不中断
    - 如果getTask结果为null则跳出循环，执行processWorkerExit()方法，销毁线程

- 管理任务，充当生产者部分

  - 任务提交，首先检查线程池运行状态，如果不是running直接拒绝
  - 如果workcount<coreSize,创建并启动一个线程来执行任务
  - workercount》=coreSize，且阻塞队列没满，就加入队列
  - 如果workercount》=coreSize&&workerCount < maximumPoolSize，且线程池内的阻塞队列已满，则创建并启动一个线程来执行新提交的任务。
  - 如果workerCount >= maximumPoolSize，并且线程池内的阻塞队列已满, 则根据拒绝策略来处理该任务, 默认的处理方式是直接抛异常。

## ThreadEXecutorPool核心参数

- coreSize：核心线程数/最小存活的工作线程数量
- MaximumPoolSize：最大线程数量
- 任务队列：workQueue
  - ![](/Users/liujianqiang/Documents/java并发编程/线程池阻塞队列.jpg)
  - synchrouousQueue 直接提交队列又叫做无缓存等待队列
  - ArrayBlockingQueue：有界队列/基于数组的先进先出队列
  - LinkeBlockingQueue：无界队列/基于链表
  - DelayQueue：延时队列
  - PriorityBlockingQueue：基于优先级的阻塞队列
- keepAliveTime：核心线程意外的线程空闲存活时间
- RejectExecutionHandler：拒绝策略
  - AbortPolicy：默认策略，抛出RejectException
  - CallerRunsPolicy：在调用者线程执行任务
  - DiscardPolicy：直接抛弃
  - DiscardOldersPolicy：抛弃最老任务

## 线程池的四种模型（java提供的四种线程池模型）

### 这四种线程池怎么选择，优劣，适合业务？

### 参数配置不同，优劣也不同：

- FixedThreadPool和singleThreadPool使用的是基于链表的队列，所以队列无限大，很占内存，可能oom
- CacheThreadPool和ScheduledThreadPool 的最大线程数很大，会创建很多线程，oom

- 总的来说就分为两类
  - 线程数一定，任务来了就积压，反正是无界队列
  - 线程数无限，及时清理任务

### 线程配置

- 特点：不用排队，直接new线程执行，容易oom

- 可缓存线程池

- coreSize：0

- MaxSize：Integer.max

  ```java
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
              60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
  }
  ```

  

- newFixedThreadPool

  - 特点：无界队列，容易积压任务，就n个线程一直工作，压力大的时候不够呀

  - 固定线程池

  - coreSize：n

  - MaxSize：n

    ```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
    ```

    

- singleThreadPool

  - 特点：就一个线程处理任务，基于链表的无界队列，容易积压

  - 单线程线程池，只有一个线程处理任务，保证串行执行

  - coresize：1

  - maxsize：1

  - ```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
    ```

    

- ScheduledThreadPool

  - 特点：延时，容易产生很多线程oom

  - 固定大小线程池

  - 并以延时或者定时的形式执行任务

  - coreSize：n

  - maxSize：Integer.max

    ```java
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE,
              DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
              new DelayedWorkQueue());
    }
    ```

    

## 应用场景/使用事项

- ### 注意事项

  - 一定要使用线程池
  - 任务独立，如果任务依赖于其他任务可能死锁
  - 合理分配阻塞时间长的任务
  - 设置合理的线程池大小 公式：
    - 线程池大小=NCPU *UCPU(1+W/C)
    - 核心线程：2*CPU   最大线程25*cpu
    - 核心线程：tps*time 最大线程：tps*time *（1.7-2）
  - 选择合适的阻塞队列

### 应用场景

- 高并发，任务时间短
  - 线程数设置为cpu加1，减少线程的上下文切换
- 并发低，任务时间长
  - io密集任务，不涉及cpu，不要让cpu闲下来，加大线程数
  - 计算密集任务，线程数少一点，避免线程的上下文切换
- 并发高，任务时间长

## 线程池参数动态化

- 根据时间设置不同的参数，高峰期时间设置大参数，，低峰期设置小参数，实现动态线程数目

# 同步容器

## concurrentHashMap

### 结构分析

- hashEntry
  - next指针final ：在concurrentHashMap中，散列如果发生hash冲突，采用拉链法来解决，由于HashEntry的next是final的所以新节	点只能头插法插入
  - 

```java
static final class HashEntry<K,V> { 
       final K key;                       // 声明 key 为 final 型
       final int hash;                   // 声明 hash 值为 final 型 
       volatile V value;                 // 声明 value 为 volatile 型
       final HashEntry<K,V> next;      // 声明 next 为 final 型 
  		//
 
       HashEntry(K key, int hash, HashEntry<K,V> next, V value) { 
           this.key = key; 
           this.hash = hash; 
           this.next = next; 
           this.value = value; 
       } 
} 
```

- segment类继承于ReentrantLock类

  ```java
  static final class Segment<K,V> extends ReentrantLock implements Serializable { 
         /** 
          * 在本 segment 范围内，包含的 HashEntry 元素的个数
          * 该变量被声明为 volatile 型
          */ 
         transient volatile int count; 
   
         /** 
          * table 被更新的次数
          */ 
         transient int modCount; 
   
         /** 
          * 当 table 中包含的 HashEntry 元素的个数超过本变量值时，触发 table 的再散列
          */ 
         transient int threshold; 
   
         /** 
          * table 是由 HashEntry 对象组成的数组
          * 如果散列时发生碰撞，碰撞的 HashEntry 对象就以链表的形式链接成一个链表
          * table 数组的数组成员代表散列映射表的一个桶
          * 每个 table 守护整个 ConcurrentHashMap 包含桶总数的一部分
          * 如果并发级别为 16，table 则守护 ConcurrentHashMap 包含的桶总数的 1/16 
          */ 
         transient volatile HashEntry<K,V>[] table; 
   
         /** 
          * 装载因子
          */ 
    		 final float loadFactor; 
    		 Segment(int initialCapacity, float lf) { 
             loadFactor = lf; 
             setTable(HashEntry.<K,V>newArray(initialCapacity)); 
         } 
   
         /** 
          * 设置 table 引用到这个新生成的 HashEntry 数组
          * 只能在持有锁或构造函数中调用本方法
          */ 
         void setTable(HashEntry<K,V>[] newTable) { 
             // 计算临界阀值为新数组的长度与装载因子的乘积
             threshold = (int)(newTable.length * loadFactor); 
             table = newTable; 
         } 
   
         /** 
          * 根据 key 的散列值，找到 table 中对应的那个桶（table 数组的某个数组成员）
          */ 
         HashEntry<K,V> getFirst(int hash) { 
             HashEntry<K,V>[] tab = table; 
             // 把散列值与 table 数组长度减 1 的值相“与”，
  // 得到散列值对应的 table 数组的下标
             // 然后返回 table 数组中此下标对应的 HashEntry 元素
             return tab[hash & (tab.length - 1)]; 
         } 
  }
  ```

### 高并发性来源于以下三方面

- ## 用分离锁实现多个线程间的更深层次的共享访问。

  - 通过减小请求同一个锁的频率和尽量减少持有锁的时间，使得 ConcurrentHashMap 的并发性相对于 HashTable 用同步包装器包装的 HashMap有了质的提高。`

- ## 用 HashEntery 对象的不变性来降低读操作对加锁的需求

  - key，hash，next都被声明为final类型。这意味着不能把节点添加到链接的中间或者尾部，也不能在中间或者尾部删除节点这个特性可以保证：在访问某个节点时，这个节点后面的节点不会被改变。
    - clear操作。clear 操作只是把 ConcurrentHashMap 中所有的桶“置空”，每个桶之前引用的链表依然存在，只是桶不再引用到这些链表（所有链表的结构并没有被修改）。**正在遍历某个链表的读线程依然可以正常执行对该链表的遍历。**
    - remove操作
      - remove操作流程：
      - 首先根据散列码找到具体的链表；
      - 然后遍历这个链表找到要删除的节点；
      - 最后把待删除节点之后的所有节点原样保留在新链表中，把待删除节点之前的每个节点克隆到新链表中。下面通过图例来说明 remove 操作。
      - 假设写线程执行 remove 操作，要删除链表的 C 节点，
      - 另一个读线程同时正在遍历这个链表。在执行 remove 操作时，原始链表并没有被修改，也就是说：读线程不会受同时执行 remove 操作的并发写线程的干扰。
    - put操作。put 操作如果需要插入一个新节点到链表中时 , 会在链表头部插入这个新节点。**此时，链表中的原有节点的链接并没有被修改。**也就是说：插入新健 / 值对到链表中的操作不会影响读线程正常遍历这个链表。
  - value被声明为volatile

- ## 用 Volatile 变量协调读写线程间的内存可见性

  - Volatile value
  - Volatile count
    - 在 ConcurrentHashMap 中，所有执行写操作的方法（put, remove, clear），在对链表做结构性修改之后，在退出写方法前都会去写这个 count 变量。所有未加锁的读操作（get, contains, containsKey）在读方法中，都会首先去读取这个 count 变量。





## 原子类

- CAS实现

## BlockedQueue

## CopyOnWriteArrayList

## ThreadLocal

参考资料：

[ThreadLocal面试攻略：吃透它的每一个细节和设计原理](https://www.jianshu.com/p/dc9be75b8efd)

[ThreadLocal-hash冲突与内存泄漏](https://blog.csdn.net/Summer_And_Opencv/article/details/104632272)

[Java-ThreadLocal三种使用场景](https://cloud.tencent.com/developer/article/1636025)

### 定义：

threadLocal时Thread类的一个局部变量，每个线程都有一个ThreadLocal他们是线程隔离的，只能访问自己的

### 结构：

<img src="/Users/liujianqiang/Documents/java并发编程/threadlocal str.jpeg" alt="threadlocal str" style="zoom:50%;" />

- 每个线程中有一个ThreadLocalMap
  - key：是ThreadLocal的弱引用（内存泄漏的问题）
  - value是值

- 通过hash确定在哪一个entry，如果发生hash冲突就采用开放地法

  - 为什么采用开放地址法，因为数据少，key又是弱引用，一下子就被回收了
  - ThreadLocal 中看到一个属性 HASH_INCREMENT = 0x61c88647 ，0x61c88647 是一个神奇的数字，让哈希码能均匀的分布在2的N次方的数组里, 即 Entry[] table，

- ### 使用场景：

  - RPC框架中的traceId，多个模块有同样的traceid，放在线程的threadLocal中

  - 代替参数的显示传递

    - 当我们在写API接口的时候，通常Controller层会接受来自前端的入参，当这个接口功能比较复杂的时候，可能我们调用的Service层内部还调用了 很多其他的很多方法，通常情况下，我们会在每个调用的方法上加上需要传递的参数。

      但是如果我们将参数存入ThreadLocal中，那么就不用显式的传递参数了，而是只需要ThreadLocal中获取即可。

  - 全局存储用户信息

    - 我们会选择在拦截器的业务中， 获取到保存的用户信息，然后存入ThreadLocal，那么当前线程在任何地方如果需要拿到用户信息都可以使用ThreadLocal的get()方法 
    - 当用户登录后，会将用户信息存入Token中返回前端，当用户调用需要授权的接口时，需要在header中携带 Token，然后拦截器中解析Token，获取用户信息，调用自定义的类(AuthNHolder)存入ThreadLocal中，当请求结束的时候，将ThreadLocal存储数据清空， 中间的过程无需在关注如何获取用户信息，只需要使用工具类的get方法即可。

  - 解决线程安全问题

    - 在Spring的Web项目中，我们通常会将业务分为Controller层，Service层，Dao层， 我们都知道@Autowired注解默认使用单例模式，那么不同请求线程进来之后，由于Dao层使用单例，那么负责[数据库](https://cloud.tencent.com/solution/database?from=10680)连接的Connection也只有一个， 如果每个请求线程都去连接数据库，那么就会造成线程不安全的问题，Spring是如何解决这个问题的呢？

      在Spring项目中Dao层中装配的Connection肯定是线程安全的，其解决方案就是采用ThreadLocal方法，当每个请求线程使用Connection的时候， 都会从ThreadLocal获取一次，如果为null，说明没有进行过数据库连接，连接后存入ThreadLocal中，如此一来，每一个请求线程都保存有一份 自己的Connection。于是便解决了线程安全问题

      ThreadLocal在设计之初就是为解决并发问题而提供一种方案，每个线程维护一份自己的数据，达到线程隔离的效果。

### 慎用场景

- 1.线程池中线程调用使用ThreadLocal 由于线程池中对线程管理都是採用线程复用的方法。在线程池中线程非常难结束甚至于永远不会结束。这将意味着线程持续的时间将不可预測，甚至与JVM的生命周期一致
- 2.异步程序中，ThreadLocal的參数传递是不靠谱的， 由于线程将请求发送后。就不再等待远程返回结果继续向下运行了，真正的返回结果得到后，处理的线程可能是其他的线程。Java8中的并发流也要考虑这种情况
- 3.使用完ThreadLocal ，**最好手动调用 remove() 方法，防止出现内存溢出，因为中使用的key为ThreadLocal的弱引用， 如果ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，但是如果value是强引用，不会被清理， 这样一来就会出现 key 为 null 的 value。**

## vector/HashTable/StringBuffer

- synchronized实现
- HashTable 
  - 初始化10
  - resize 2*capacity+1
  - key 和value不能为null
- vector
  - 扩容原来的2倍
  - todo：vector一定是线程安全的吗



# 锁

[[不可不说的Java“锁”事](https://tech.meituan.com/2018/11/15/java-lock.html)](https://tech.meituan.com/2018/11/15/java-lock.html)

### 公平锁和非公平锁

- 公平锁

  - 原理：

    - lock方法最终调用FairSync重写的tryAcquire方法

      ```java
      protected final boolean tryAcquire(int acquires) {
                  //获取当前线程和状态值
                  final Thread current = Thread.currentThread();
                  int c = getState();
                 //状态为0说明该锁未被任何线程持有
                  if (c == 0) {
                   //为了实现公平，首先看队列里面是否有节点，有的话再看节点所属线程是不是当前线程，是的话hasQueuedPredecessors返回false,然后使用原子操作compareAndSetState保证一个线程更新状态为1，设置排他锁归属与当前线程。其他线程通过cass则返回false.
                      if (!hasQueuedPredecessors() &&
                          compareAndSetState(0, acquires)) {
                          setExclusiveOwnerThread(current);
                          return true;
                      }
                  }
      //状态不为0说明该锁已经被线程持有，则看是否是当前线程持有，是则重入锁次数+1.
                  else if (current == getExclusiveOwnerThread()) {
                      int nextc = c + acquires;
                      if (nextc < 0)
                          throw new Error("Maximum lock count exceeded");
      
                      setState(nextc);
                      return true;
                  }
                  return false;
              }
          }
      ```

      

    - unLock方法，最终调用了Sync的tryRelease方法

      ```java
      protected final boolean tryRelease(int releases) {
                 //如果不是锁持有者调用UNlock则抛出异常。
                  int c = getState() - releases;
                  if (Thread.currentThread() != getExclusiveOwnerThread())
                      throw new IllegalMonitorStateException();
                  boolean free = false;
                 //如果当前可重入次数为0，则清空锁持有线程
                  if (c == 0) {
                      free = true;
                      setExclusiveOwnerThread(null);
                  }
                  //设置可重入次数为原始值-1
                  setState(c);
                  return free;
              }
      ```

      

- 非公平锁

  

  

- 为什么默认非公平？公平锁需要资源调度

### 乐观锁和悲观锁

- 乐观锁

  - 定义：默认不冲突，数据提交更新的时候再通过检测是否发生了冲突，如果发生了，就重试。
  - 检测：
    - 版本号
    - 时间戳
    - 实现：CAS

  - 缺点：ABA问题
    - AtomicStampedReference
    - 版本号

  - 

- 悲观锁

  - 定义：默认并发，操作前先拿锁，事务执行完释放
  - 适用于竞争激烈的场景
  - 上下文切换开销大，排他

自旋锁

### 共享锁和排他锁

- 共享锁
- 排他锁

### 数据库锁

- 间隙锁
- 行锁
- 表锁

### 锁升级

#### 原理：

##### 	对象头：

对象头中有两类信息：markword和类型指针

- mark word

  - 当对象状态是偏向锁的时候，存储的是偏向线程的id
  - 当状态是轻量级锁的时候，存储的是指向线程栈中的lock record指针
  - 当为重量级锁的时候，存储的是指向堆中monitor对象的指针

  - hashcode

  - Gc分代年龄

  - 锁状态

    ![对象头锁状态](/Users/liujianqiang/Documents/java并发编程/对象头锁状态.jpg)

  - 无锁

  - 偏向锁

    - 第一次获取，CAS thread ID，成功就加锁
    - 再次进入，发现锁的对象的thread id相同，检查继续就行
    - 其他线程进来，发现有偏向锁状态，就会撤销状态，持有锁的线程执行完就会锁升级
    - 

  - 轻量级锁

    加锁过程：

    - 在线程栈中创建一个lock record，将其obj字段指向锁对象
    - 直接通过cas指令将lock record的地址存储在对象头的mark word 中，拿锁成功
    - 如果当前线程已经持有锁,设置lock record的第一部分为null，起到一个重入计数器的作用

    解锁过程：

    - 遍历线程栈，找到所有obj字段等于当前锁对象的lock record
    - 如果lock record 的displaced mark word为null，代表这是一次重入，将obj设置为null后continue
    - 如果displaced mark word不为null，则利用cas指针将对象头的mark word

  - 重量级锁

  

### 可重入锁

# unSafe

[unsafe](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)

[Java CAS 原理剖析](https://juejin.cn/post/6844903558937051144)

## CAS

CAS操作包含三个操作数——内存位置、预期原值及新值。执行CAS操作的时候，将内存位置的值与预期原值比较，如果相匹配，那么处理器会自动将该位置值更新为新值，否则，处理器不做任何操作。

### 原理：

getAndAddInt()->compareAndSwapInt()->底层CPU指令comxchg

### 应用：

- CAS在java.util.concurrent.atomic相关类、Java AQS、CurrentHashMap等实现上有非常广泛的应用。
- AtomicInteger的实现中，**静态字段valueOffse**t即为字段value的内存偏移地址，valueOffset的值在AtomicInteger初始化时，在静态代码块中通过Unsafe的objectFieldOffset方法获取。在AtomicInteger中提供的线程安全方法中，**通过字段valueOffset的值可以定位到AtomicInteger对象中value的内存地址，从而可以根据CAS实现对value字段的原子操作。**
- getAndAddInt()循环获取给定对象o中的偏移量处的值v，然后判断内存值是否等于v。如果相等则将内存值设置为 v + delta，否则返回false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。
- 整个“比较+更新”操作封装在compareAndSwapInt()中，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。
- 后续JDK通过CPU的cmpxchg指令，去比较寄存器中的 A 和 内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。**如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的while循环再次调用cmpxchg指令进行重试，直到设置成功为止。**

### 缺点：

- ABA
- 自旋消耗cpu
- 只能保证一个原子变量的共享操作

## 内存屏障

```java
//内存屏障，禁止load操作重排序。屏障前的load操作不能被重排序到屏障后，屏障后的load操作不能被重排序到屏障前
public native void loadFence();
//内存屏障，禁止store操作重排序。屏障前的store操作不能被重排序到屏障后，屏障后的store操作不能被重排序到屏障前
public native void storeFence();
//内存屏障，禁止load、store操作重排序
public native void fullFence();
```



## 线程调度

- park（LockSuppor中的unpark，park实现就是基于这个）
- unpark
- monitorstart
- monitorexit
- tryMonitorExist

## 对象操作

## 数组相关

## 内存操作

包含堆外内存的分配、拷贝、释放、给定地址值操作等方法。

# 面试经典题目

## double check lock

## 生产者消费者

### ReentrantLock

```java
package producerAndConsumer;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author: liujianqiang
 * @Date: 2021/8/21 11:39 下午
 */
public class AQSsolution {
    public static void main(String[] args) {
        Producer p1 = new Producer();
        Producer p2 = new Producer();
        Producer p3 = new Producer();
        Producer p4 = new Producer();
        Consumer c1 = new Consumer();
        Consumer c2 = new Consumer();
        Consumer c3 = new Consumer();
        new Thread(p1).start();
        new Thread(p2).start();
        new Thread(p3).start();
        new Thread(p4).start();
        new Thread(c1).start();
        new Thread(c2).start();
        new Thread(c3).start();
    }
    static int size;
    static int capacity=10;
    static ReentrantLock reentrantLock = new ReentrantLock();
    static Condition c1 = reentrantLock.newCondition();
    static Condition c2 = reentrantLock.newCondition();
    static class Consumer implements Runnable{
        @Override
        public void run() {
            while (true){
                reentrantLock.lock();
                try {
                    while (size==capacity){
                        c2.await();
                        System.out.println(Thread.currentThread().getName()+"消息队列空了消费者停止消费了size："+size);
                    }
                    size++;
                    c1.signalAll();
                    System.out.println(Thread.currentThread().getName()+"消息队列消费者消费了size："+size);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    reentrantLock.unlock();
                }
            }


        }
    }
    static class Producer implements Runnable{
        @Override
        public void run() {
            while (true){
                reentrantLock.lock();
                try {
                    while (size==0){
                        c1.await();
                        System.out.println(Thread.currentThread().getName()+"消息队列空了消费者者停止消费size："+size);
                    }
                    size--;
                    c2.signalAll();
                    System.out.println(Thread.currentThread().getName()+"消息队列消费者消费了size："+size);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    reentrantLock.unlock();
                }
            }


        }
    }
}

```



### Semaphore

```java
package producerAndConsumer;

import java.util.concurrent.Semaphore;

/**
 * @Author: liujianqiang
 * @Date: 2021/8/21 11:46 下午
 */
public class SemaphoneSolution {
    public static void main(String[] args) {
        Producer p1 = new Producer();
        Producer p2 = new Producer();
        Producer p3 = new Producer();
        Producer p4 = new Producer();
        Consumer c1 = new Consumer();
        Consumer c2 = new Consumer();
        Consumer c3 = new Consumer();
        new Thread(p1).start();
        new Thread(p2).start();
        new Thread(p3).start();
        new Thread(p4).start();
        new Thread(c1).start();
        new Thread(c2).start();
        new Thread(c3).start();
    }
    static int size;
    static int capacity=10;
    static Semaphore notFull = new Semaphore(10);
    static Semaphore notEmpty = new Semaphore(0);
    static Semaphore mutex = new Semaphore(1);
    static class Consumer implements Runnable{
        @Override
        public void run() {
            while (true){
                try {
                    while (size==0){
                        notEmpty.acquire();
                        mutex.acquire();
                        System.out.println(Thread.currentThread().getName()+"消息队列空了消费者者停止消费size："+size);
                    }
                    size--;
                    System.out.println(Thread.currentThread().getName()+"消息队列消费者消费了size："+size);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    mutex.release();
                    notFull.release();
                }
            }
        }
    }
    static class Producer implements Runnable{
        @Override
        public void run() {
            while (true){
                try {
                    while (size==capacity){
                        notFull.acquire();
                        mutex.acquire();
                        System.out.println(Thread.currentThread().getName()+"消息队列满了生产者者停止生产size："+size);
                    }
                    size++;
                    System.out.println(Thread.currentThread().getName()+"消息队列生产者生产了size："+size);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    mutex.release();
                    notEmpty.release();
                }
            }
        }
    }
}

```



### Syn

```java
package producerAndConsumer;

/**
 * @Author: liujianqiang
 * @Date: 2021/8/21 11:29 下午
 */
public class Solutino {
    public static void main(String[] args) {
        Producer p1 = new Producer();
        Producer p2 = new Producer();
        Producer p3 = new Producer();
        Producer p4 = new Producer();
        Consumer c1 = new Consumer();
        Consumer c2 = new Consumer();
        Consumer c3 = new Consumer();
        new Thread(p1).start();
        new Thread(p2).start();
        new Thread(p3).start();
        new Thread(p4).start();
        new Thread(c1).start();
        new Thread(c2).start();
        new Thread(c3).start();
    }
    private static final Object lock = new Object();
    static int capacity=10;
    static int size=0;
    static class Producer implements Runnable{
        @Override
        public void run() {
            while (true){
                synchronized (lock){
                    while (size>=capacity){
                        try {
                            lock.wait();
                            System.out.println(Thread.currentThread().getName()+":"+"消息队列已经满了"+"生产者生产了"+size);
                        }catch (Exception e){
                            e.printStackTrace();
                        }
                    }
                    size++;
                    System.out.println(Thread.currentThread().getName()+":"+"消息队列生产者生产了"+size);
                    lock.notifyAll();
                }
            }
        }
    }
    static class Consumer implements Runnable{
        @Override
        public void run() {
            while (true){
                synchronized (lock){
                    while (size==0){
                        try {
                            lock.wait();
                            System.out.println(Thread.currentThread().getName()+":"+"消息队列已经满了size"+size+"消费者停止");
                        }catch (Exception e){
                            e.printStackTrace();
                        }
                    }
                    size--;
                    System.out.println(Thread.currentThread().getName()+":"+"消息队列 消费者消费了"+size);
                    lock.notifyAll();
                }
            }
        }
    }

}

```



### 三个线程轮流执行

#### AQS

```java
package producerAndConsumer;

/**
 * @Author: liujianqiang
 * @Date: 2021/8/22 12:23 上午
 */
public class ABC {
    public static void main(String[] args) {
        A a1 = new A();
        B b1 = new B();
        C c1 = new C();
        Thread a = new Thread(a1);
        Thread b = new Thread(b1);
        Thread c = new Thread(c1);
        a.start();
        b.start();
        c.start();
    }
    static int time=1;
    static Object o = new Object();

    static class A implements Runnable{
        @Override
        public void run() {
            while (true){
               synchronized (o){
                   while (time%3!=1){
                       try {
                            o.wait();
                       }catch (Exception e){
                           e.printStackTrace();
                       }
                   }
                   if (time<31){
                       System.out.println(Thread.currentThread().getName()+"A-------"+time);

                       time++;
                       o.notifyAll();
                   }else {
                       break;

                   }
               }
           }
        }
    }
    static class B implements Runnable{
        @Override
        public void run() {
            while (time<31){
                synchronized (o){
                    while (time%3!=2){
                        try {
                            o.wait();
                        }catch (Exception e){
                            e.printStackTrace();
                        }
                    }
                    if (time<31){
                        System.out.println(Thread.currentThread().getName()+"B-------"+time);

                        time++;
                        o.notifyAll();
                    }else {
                        break;

                    }
                }
            }
        }
    }
    static class C implements Runnable{
        @Override
        public void run() {
            while (time<31){
                synchronized (o){
                    while (time%3!=0){
                        try {
                            o.wait();
                        }catch (Exception e){
                            e.printStackTrace();
                        }
                    }
                    if (time<31){
                        System.out.println(Thread.currentThread().getName()+"C-------"+time);

                        time++;
                        o.notifyAll();
                    }else {
                        break;

                    }
                }
            }
        }
    }
}

```

#### AQS

