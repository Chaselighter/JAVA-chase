# java内存模型

## 物理模型

- cpu
- cpu寄存器
- cpu高速缓存
- 主内存

## 执行操作

- 处理器优化
- 指令重排
- 内存系统重排

## 交互方式

- lock
- unlock
- Read:读取主内存变量
- load：载入主内存变量到本地内存
- use：对变量进行操作
- assign：赋值
- store：存储变量到主内存
- write：写入主内存吧

## 问题

- ### 可见性

- ### 原子性

- ### 有序性

## 规范

- 所有变量都存在主内存值
- 每个线程都有自己的私有本地内存，存储了该对象的读写变量的副本
- 线程对变量的操作必须在本地内存进行
- 不同线程之间的本地内存变量不共享



# 并发三要素

## 原子性

- ### 对象类型

  - 对象地址原子读写，线程安全
  - 并发读不可变状态，线程安全
  - 并发读可变状态，线程不安全

- ### 基本类型

  - int，char读写，线程安全
  - long，double高低位，非线程安全
  - i++组合操作，非线程安全

- 

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
  - 监视器法则
  - volatile变量法则
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
  - AQS

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



# 线程池

# 锁

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

  - 缺点：ABA问题
    - AtomicStampedReference
    - 版本号

  - 

- 悲观锁

  - 定义：默认并发，操作前先拿锁，事务执行完释放
  - 适用于竞争激烈的场景
  - 上下文切换开销大，排他

### 共享锁和排他锁

- 共享锁
- 排他锁

### 数据库锁

- 间隙锁
- 行锁
- 表锁

### 锁升级

#### 原理：

#####  对象头：

对象头中有两类信息：markword和类型指针

- mark word

  - 当对象状态是偏向锁的时候，存储的是偏向线程的id
  - 当状态是轻量级锁的时候，存储的是指向线程栈中的lock record指针
  - 当为重量级锁的时候，存储的是指向堆中monitor对象的指针

  - hashcode

  - Gc分代年龄

  - 锁状态

    ![对象头锁状态](/Users/liujianqiang/Documents/对象头锁状态.jpg)

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

# 同步容器