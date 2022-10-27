
[TOC]


### 一、线程
##### 1、线程基础
###### 1.1 线程状态
新建：new Thread 但是为运行run中的代码
可运行:调用了start方法 可能在运行(获得时间片) 也可能未运行
被中止:run运行完/run中异常退出
可转换状态:
被阻塞:未获得锁
等待:调用了Object.wait
计时等待:Thread.sleep/Lock.tryLock

###### 1.2 使用
继承Thread重写run方法 每个run中的属性相互独立
重写Runnable 当做参数传入Thread的构造 run中属性共用 需要处理同步

###### 1.3 interrupt
https://www.cnblogs.com/jenkov/p/juc_interrupt.html
interrupt:向另一个线程发送中断状态 由另一个线程做具体处理 也可以让处于阻塞(io阻塞等)状态的线程抛异常唤醒
isInterrupted:判断是否设置了中断
interrupted:判断是否设置了中断 并清除中断标记(需要注意的是 在mainThread中调用thread.interrupted 不会去判断thread的状态 而是mainThread的 所有要在thread中用)
https://blog.csdn.net/qq_39682377/article/details/81449451

###### 1.4 异常处理
run无法捕获异常 崩溃也不会影响其他线程 可以做到线程的隔离
如果想捕获 可以重写Thread.UncaughtExceptionHandler

###### 1.5 Callable/Runnable/Future/FutureTask
用于封装异步任务
run没有参数和返回值 无法抛异常
call有返回值 可以抛异常
Future有返回值 抛异常 且可以设置阻塞超时
FutureTask继承Runnable, Future

###### 1.6 实现
在android中 Thread的实现为Linux的系统调用pthread 在调用Thread.start时才会正式创建 并启动底层线程 然后使用jni找到run方法 并执行
android中所有的线程由vm统一管理

###### 1.7 优缺点
有点：
方便高效的内存共享 共享堆内存	比进程轻便
较小的上下文开销

缺点：
线程占用一定的内存
线程越多 线程切换开销越大
同步通信复杂

###### 1.8 wait/sleep

| wait | sleep |
|--------|--------|
|   Object的对象方法     |    Thread的静态方法    |
| 需要配合synchronized | 任何地方使用 |
| 会释放锁 | 不会释放锁 |
| 不需要捕获异常 | 捕获异常 |
| 都是锁当前线程 |
| 都会响应interrupt |

Object.wait也可以设置超时

###### 1.9 yield/join
yield 会停止当前线程 让同等或更高优先级的线程优先执行 否则不起作用
join 在某一个线程中执行另一个线程 当前线程阻塞直到另一个线程执行完

##### 2、线程池
###### 2.1 优势
线程重复利用 避免了创建、销毁
统一管理 方便控制

###### 2.2 线程池定制
```java
/**
* corePoolSize 核心线程数 一直存活
* workQueue 任务队列容量（阻塞队列）
* handler 拒绝策略
*/
ThreadPoolExecutor(int corePoolSize,
                  int maximumPoolSize,
                  long keepAliveTime,
                  TimeUnit unit,
                  BlockingQueue<Runnable> workQueue
                  hreadFactory threadFactory,
                  RejectedExecutionHandler handler
                  )

beforeExecute/afterExecute
```



线程数选择
1. 密集型计算:cpu核心数+1
每个cpu负责处理一个任务
过多会造成内存浪费 且有额外的线程切换
2. io型计算:根据具体的业务处理 (推荐2*cpu核心数)

###### 2.3 原理
主要实现类为ThreadPoolExecutor 在execute时 将任务封装为FutureTask 再封装在继承自AQS的Worker中 Worker会新建线程
线程池开启时
1. 如果当前线程数小于核心线程数 则通过Worker新建线程(Worker的构造中) 并执行 即使有空线程
2. 如果当前线程数量大于核心线程 且小于最大线程时 继续创建
3. 如果任务数量大于最大线程数 则加入缓冲队列
4. 如果缓冲满 则进入拒绝策略
5. 每个线程中有一个while死循环 通过getTask获取BlockingQueue中的任务
6. 当任务执行完 任务数量小于最大线程数时 空闲一段时间后会销毁多余线程
7. 当没有任务时 线程会通过BlockingQueue.take进入阻塞状态 直到有新任务唤醒

Worker继承AQS不是为了互斥(因为worker内部的线程时独立的) 而是为了防止interrupt中断当前正在执行的任务(lock会忽略interrupt)
https://blog.csdn.net/xxcupid/article/details/51983582

###### 2.4 拒绝策略
AbortPolicy() 抛异常
CallerRunsPolicy() 交由调用者线程处理
DiscardOldestPolicy() 抛弃最老的任务
DiscardPolicy() 不接受新任务

### 二、同步
##### 1、前置
###### 1.1、Unsafe
提供了直接操作内存和线程的低层次操作
官方不建议使用 且以后有可能会关闭

常用的功能有：
1. 通过变量的偏移地址 直接修改对象值
2. unsafe.park/unsafe.unpark
3. compareAndSwapObject(obj, offset, expect, update):boolean
obj和offset共同确定内存中的值 如果内存中的值和expect的值相同 则替换为update 成功返回true

###### 1.2、CAS
基于Unsafe.compareAndSwapObject 完成自旋锁
缺点:如果自旋次数多开销大 只能保证变量 ABA问题

ABA问题:
变量初始为A T1获取锁 改为B 又再次改为A
T2获取锁 但是发现值没有改变(无法察觉T1改为B的过程) 会造成隐患

https://www.cnblogs.com/549294286/p/3766717.html

解决：
带标记的原子引用类:AtomicStampedReference
使用互斥

###### 1.3、LockSupport
基于Unsafe 的park/unpark
不可重入 可以响应interrupt
可以先unpark 再park

##### 2、同步语义
###### 1、synchronized
锁对象(需要保证对象的唯一性)
锁方法
锁静态方法(全局锁 无论有多少个对象 共用锁)

需要尽量缩小锁的粒度

非公平 可重入锁

原理:
synchronized有对应的字节码指令 每个对象都有一个监视器 然后通过指令获取监视器 如果获取成功则执行 否则阻塞
所以需要保证对象的唯一性



锁升级:

1 无锁

2 偏向锁 只有一个线程访问代码块 且可重入

3 轻量级锁 有其他线程通过自旋尝试获取锁 如果自旋一定次数未获取到锁 则升级为重量级锁

4 重量级锁 其他线程获取锁失败且进入阻塞状态



https://blog.csdn.net/xiangwanpeng/article/details/57096586

锁升级: https://www.jianshu.com/p/d61f294ac1a6

###### 2、wait与notify
会操作对象的监视器(只能在synchronized中使用)
都可以通过interrupt打断暂停状态 抛出IntterruptedException

###### 3、volatile
https://www.cnblogs.com/chenssy/p/6379280.html
每个线程获取的volatile变量都是最新值 但不是原子性

原理：
主内存中为运行时数据
每个线程占用一个cpu 每个cpu有独立的存储 运行时会将主内存数据复制到线程缓存 提高cpu效率 但是有一致性问题

volatile有两层语义
1. 保证可见性:当cpu写volatile变量时 会通知其他线程变量失效 需要重新从主内存读
2. 禁止指令重排序:底层添加内存屏障 防止处理器重排序(如果不存在数据依赖 cpu可以改变执行顺序)

###### 4、ThreadLocal
https://www.jianshu.com/p/0ba78fe61c40
ThreadLocal中有一个ThreadLocalMap 继承Map 
ThreadLocalMap中有Entry 继承自弱引用 键值对结构 key为ThreadLocal的弱引用 value为保存的数据

每个Thread中ThreadLocalMap是唯一的 用于实际的存储 ThreadLocal只是一个key而已 所以保证了每个线程只有一份数据

如果需要自己完成 可以使用HashMap key为Thread的弱引用即可

优势:
当多线程数据不需要共享时 使用锁会造成线程等待 降低效率 而TL可以避免线程阻塞

###### 5、原子变量
原子变量基于CAS和volatile

###### 6、纯函数
方法是线程相关的 如果不依赖外部变量 就不存在同步了 与ThreadLocal思想类似

##### 3、锁
###### 1 AQS (AbstractQueuedSynchronizer)
https://www.cnblogs.com/waterystone/p/4920797.html
AQS可以实现共享/独占锁 内部将线程维护为一个双向链表 主要基于LockSupport/CAS
AQS.state支持线程的状态(等待 取消等) 也支持可重入的次数为32位
tryAcquire:独占模式 抽象方法 尝试获取资源 成功返回true
tryAcquireShared:共享模式 尝试获取资源 负数:失败 0:成功但没资源 正数:成功

独占锁原理:
1. 通过实现tryAcquire 可以尝试直接获取锁
2. 如果失败 则将当前线程加入AQS的双向链表中 等待获取锁
3. 因为是独占锁 只有第二个节点的线程可以获取资源(头结点为空节点)
4. 如果没能得到锁 则使用park进入等待 直到前一个锁被释放 通过unpark唤醒当前节点

共享锁原理:
与独占锁类似 只是在第四步 前一个节点被唤醒时 如果有多余的资源 会唤醒后续的多个节点

问题:等待线程interrupt时的处理
interrupt的线程会被唤醒 但是在循环中 没有得到锁是不会出来 唤醒的线程将被再次park  当得到锁时会添加中断标记
另外有lockInterruptibly可以使等待的线程响应中断而返回

1.2 Condition
https://www.cnblogs.com/vielat/p/15022895.html
用来替代传统Object的wait()、notify()
主要有await/signal 只可以在lock()/unlock()
Condition的实现为内部类ConditionObject

ConditionObject维护一个双向队列 firstWaiter/lastWaiter

AQS内部也维护一个双向队列 head/tail

原理:
ConditionObject内部复用AQS的Node 实现了一个双向队列

```java
public final void await() {
	  //1 将当前线程加入Coud的队列
    Node node = addConditionWaiter();
    //2 移除AQS中当前线程占用的锁
    int savedState = fullyRelease(node);
    //3 遍历AQS中的节点 如果没有当前线程 则说明没有资格获取锁 进入睡眠 直到signal将线程加入AQS中
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
    }
    //4 将当前线程加入AQS队列 重新获取锁
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
}
```
```java
public final void signal() {
    doSignal(first) {
    	//将Cond中的节点放入AQS中的节点 并唤醒对应的线程 对应await中的第三步isOnSyncQueue方法
    	transferForSignal() {
        }
    }
}
```

总结:
在当前线程获取锁的情况下 await将当前线程移除AQS队列 并park阻塞(会释放锁)
调用signal时 将当前线程加入AQS队列 并调用unpark
await解除阻塞 并将当前线程加入AQS队列 重新尝试获取锁

###### 2、ReentrantLock 可重入锁
2.1 ReentrantLock为可重入独占锁 可以实现公平锁/非公平锁
基于AQS 同时可以使用Condition

公平锁原理:
重写了AQS的tryAcquire

1. AQS.state==0 则表示没有线程持有锁 然后AQS.hasQueuedPredecessors判断是否有线程等待的比当前线程时间长 如果有 则当前线程进入AQS队列(公平锁的体现)
2. AQS.hasQueuedPredecessors=false 则当前线程可以直接获取 将AQS.state设置为1 并将AQS.exclusiveOwnerThread设置为当前线程(当前持有锁的线程)
3. AQS.state!=0则判断AQS.exclusiveOwnerThread是否为当前线程 如果是 则将AQS.state+1 即重入锁
4. 如果不是当前线程 则进入AQS的双向队列等待

非公平锁原理:
不添加AQS.hasQueuedPredecessors即可 有线程lock即判断AQS.state==0 则当前线程持有锁

非公平锁体现在 当前线程释放锁的一瞬间 不在队列中的线程和队列中第二个元素的线程都有机会获取锁(非公平的主要体现)  但是队列中非第二个元素还是需要等待

> lock 一直阻塞到获得锁
> tryLock 尝试获取锁 如果不能则立即返回
> lockInterruptibly 一直阻塞到获得锁 但是接受中断信号interrupt

###### 3、ReadWriteLock 读写锁
https://blog.csdn.net/fxkcsdn/article/details/82217760
可以是公平/非公平  可重入 基于AQS
写锁为独占模式且支持Condition 读锁为共享模式
AQS.state的32位可重入次数分别给读锁(高16位表示持有读锁的线程数)/写锁(低16位写锁的重入次数)
读写线程共同竞争资源 有可能写线程一直得不到资源

写写/读写 互斥
读读 共享

当没有线程获取写锁且AQS中没有等待获取写锁的线程 才能获取读锁

###### 4、StampedLock 控制锁
有三种状态 读、写、乐观读
对于读写锁 如果读线程过多 写线程可能一直获取不到锁 控制锁是读写锁的改进
乐观读：读操作不会阻塞写操作 当写时 会使读操作重读

###### 5、CountDownLatch
基于AQS的共享锁 等待其他线程执行完再执行本线程
1. 初始时 设置最大等待数 new CountDownLatch(MAX)
2. 启动其他线程 然后CountDownLatch.await使本线程进入等待
3. 每有一个其他线程执行完 调用CountDownLatch.countDown()使MAX-1 当MAX条线程执行完 本线程会被唤醒

AQS.state相当于计数器
第一步会设置AQS.state=MAX
第三步每次countDown()会调用tryReleaseShared使AQS.state-1
当AQS.state==0 tryAcquireShared返回1 会使当前线程获取锁

###### 6、CyclicBarrier 循环屏障锁
基于ReentrantLock和Condition
1. 初始new CyclicBarrier(MAX)
2. 每个执行的线程会调用CyclicBarrier.await() 当有MAX个线程await()时 所有被阻塞的线程被唤醒

CyclicBarrier内部有一个计数器count 每await一次 count--
当count!=0 调用Condition.await()
当count==0 调用Condition.notifyAll()

###### 7、Semaphore 计数信号量
http://www.cnblogs.com/skywang12345/p/3534050.html
基于AQS的共享锁 初始化时 设置资源总数 new Semaphore(MAX)
每次调用会消耗资源数sem.acquire(count[可以不是1]); 当不足时等待

release(n)释放资源数



###### 1、生产消费者模式
```java
class Service{
    public int index;
    public LinkedList<Integer> list = new LinkedList<Integer>();
    public final static int MAX = 100;

    public void produce() {
        synchronized (list) {
            try {
                //如果货物已满 则等待消费
                while (list.size() >= MAX) {
                    System.out.println(Thread.currentThread().getName()+"工厂==>仓库有  " + list.size()+"  个商品  已经无法生产");
                    list.wait();
                }

                this.list.offer(++this.index);
                System.out.println(Thread.currentThread().getName()+"工厂==>仓库有  " + this.list.size()+"  个商品");
                Thread.currentThread().sleep(50);

                list.notify();
            }catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void custom() {
        synchronized (list) {
            try {
                //如果没有物品 需要等待生产
                while (this.list.size() <= 0) {
                    System.out.println(Thread.currentThread().getName()+"   消费==>仓库有  " + this.list.size()+"  个商品  等待工厂生产");
                    list.wait();
                }
                this.list.pop();
                System.out.println(Thread.currentThread().getName()+"消费==>仓库有  " + this.list.size()+"  个商品");
                Thread.currentThread().sleep(80);

                list.notify();
            }catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}

class Producer extends Thread{
    private Service service;

    public Producer(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        while(true)
            this.service.produce();
    }
}

class Customer extends Thread{
    private Service service;

    public Customer(Service service) {
        this.service = service;
    }

    @Override
    public void run() {
        while(true)
            this.service.custom();
    }
}

public class Test{
    public static void main(String[] args) {
        Service service = new Service();
        new Producer(service).start();
        new Producer(service).start();
        new Producer(service).start();
        new Customer(service).start();
        new Customer(service).start();
    }
}
```

### 三、内存模式
##### 1、基础
1.1 线程间的通信
1. 共享内存 需要显式同步 即互斥
2. 消息传递	隐式同步(消息的发送需要在接收之前)

1.2 内存模型抽象
实例域、静态域、数组元素在堆内存 是线程共享的
局部变量、方法参数、异常处理参数 线程的私有内存 不需要同步

java内存模型(JMM 不存在/抽象的) 线程对共享变量的写入何时对另一个线程可见
包括：缓存、写缓冲区、寄存器等
两个线程正确的通信步骤：
1. A线程修改本地的共享变量
2. 将本地共享变量刷到主内存
3. 线程B向主内存读取共享变量

1.3 重排序
1. 编译器优化重排序：不改变单线程语义的前提下 重新安排语句的执行顺序
2. 指令级并行重排序：不存在数据依赖的情况下 处理器改变机器指令的顺序
3. 内存系统重排序：处理器使用缓存和读写缓冲区 加载和存储操作有可能乱序

1为编译器重排序 2、3为处理器重排序
处理器重排序可以插入内存屏障禁止重排序

##### 2、重排序
单线程中 如果数据存在依赖关系 则不会重排序 否则也会有重排序 但是不会影响结果(在不影响结果的情况下 会尽量并行执行 提高效率)

##### 3、顺序一致性
一个线程中的所有操作必须按照程序的顺序来执行。
（不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。
在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。 但是效率低

##### 4、volatile
可见性：对于一个volatile变量的读 总能看到这个变量最后的写入
原子性：单个volatile变量具有读写原子性(long double)

写操作：当写入一个volatile变量时 会将本地内存中的共享变量刷到主内存
读操作：JMM会使当前线程对应的共享内存无效 然后从主内存读取变量

volatile会禁止编译器、处理器重排序

内存屏障 为编译器插入 用于防止处理器重排序
