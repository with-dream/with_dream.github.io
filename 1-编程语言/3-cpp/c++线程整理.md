### c++线程整理

| 头 | 说明 |
|--------|--------|
|    atomic    |    原子变量    |
|    thread    |    线程相关    |
|    mutex    |    互斥量    |
|    condition_variable    |    条件变量    |
|    future    |        |
|        |        |

#### 一、thread
##### 1、创建
```cpp
void f1(int n);
//普通创建
//如果需要传递引用时 可以使用std::ref
template <class Fn, class... Args>
explicit thread(Fn&& fn, Args&&... args);

//拷贝构造函数不可用
thread(const thread&) = delete;

//可以使用move语义 x获取当前线程的全部属性 当前线程销毁
thread(thread&& x) noexcept;
```

| api | 说明 |
|--------|--------|
|   get_id()     |    获取线程id    |
|   joinable()     |    检查线程是否可被 join()    |
|   join()     |    阻塞当前线程 直到启动的线程执行完成   |
|   detach()     |    线程交由系统托管，用于回收资源    |
|   swap()     |    交换两个线程对象所代表的句柄    |
|   native_handle     |    返回相应平台的句柄(如Linux的pthread句柄)    |
|    this_thread    |    当前线程空间的实例    |
|    yield()    |    当前线程放弃执行    |
|    sleep_until    |    线程睡眠 直到某个时刻    |
|    sleep_for    |     线程睡眠一定时间    |
|    hardware_concurrency()    |    获取cpu核心数    |
|        |        |

#### 二、同步
##### 1、mutex
###### 1.1 mutex
互斥锁

| 方法 | 说明 |
|--------|--------|
|    lock()    |        |
|    unlock()    |        |
|    try_lock()    |    非阻塞 尝试获取锁  如果已被锁住 则死锁    |
|        |        |

###### 1.2 recursive_mutex
可重入互斥锁

用法同  1.1 mutex

###### 1.3 timed_mutex
超时锁 不可重入
一定时间内未获取锁 则超时返回false

| 方法 | 说明 |
|--------|--------|
|    try_lock_for    |        |
|    try_lock_until    |        |


###### 1.4 recursive_timed_mutex
可重入超时锁
用法类似1.3 timed_mutex

##### 2、Lock模板类
###### 2.1、lock_guard
`template <class Mutex> class lock_guard`

相当于为整个方法加锁 自动管理加锁、解锁
类似java的synchronized锁方法

###### 2.2、unique_lock
`template <class Mutex> class unique_lock`

类似lock_guard 但是提供了粒度更细的方法

| 方法 |
|--------|
|    lock/try_lock/try_lock_for/try_lock_until/unlock    |
|    swap/release(释放mutex)    |
|    owns_lock(锁住为真)/lock/mutex    |

lock_guard/unique_lock构造时的标记

| 标记 | 说明 |
|--------|--------|
|    once_flag    |    多个线程执行一个方法 该方法只会执行一次 需要配合call_once使用   |
|    adopt_lock_t    |        |
|    defer_lock_t    |    初始化时 不锁住mutex    |
|    try_to_lock_t    |    调用mutex.try_lock() 失败不阻塞    |



| 模板方法 | 说明 |
|--------|--------|
|    `Mutex1& a, Mutex2& b, Mutexes&... cde`    |  都成功 返回true 有一个失败 则解锁并返回false      |
|    `lock (Mutex1& a, Mutex2& b, Mutexes&... cde)`    |    锁定所有，如果有一个未成功 则解锁已成功的    |
|    `call_once (once_flag& flag, Fn&& fn, Args&&... args)`    |    多个线程有一个成功 则方法不再被调用    |
|        |        |

参考:
https://blog.csdn.net/xiaominggunchuqu/article/details/90674710

##### 3、condition_variable
###### 3.1、condition_variable
提供了std::unique_lock 相关联的条件变量

| 方法 | 说明 |
|--------|--------|
|    wait(unique_lock)    |    阻塞线程    |
|    wait (unique_lock, Predicate)    |   Predicate:false阻塞     |
|    notify_all()    |    唤醒所有等待线程    |
|    notify_one()    |    唤醒任意一个线程    |
|    wait_for    |    一定时间未唤醒 则超时解除阻塞    |
|    wait_until    |        |
|        |        |

###### 3.2、condition_variable_any
提供与任何锁类型相关联的条件变量 是condition_variable的泛化

###### 3.3、cv_status
由wait_for和wait_until返回

cv_status::no_timeout :  该函数返回没有超时
cv_status::timeout : 该函数由于达到其时间限制（timeout）而返回。

###### 3.4、notify_all_at_thread_exit
唤醒所有线程

##### 4、atomic
原子变量 不常用
https://www.jianshu.com/p/8c1bb012d5f8

##### 5、future
###### 5.1、promise
一个共享内存 用于线程间传递数据

| 方法 | 说明 |
|--------|--------|
|    get_future()    |    返回一个关联的future对象    |
|    setValue()    |    将值传入promise    |
|    set_exception    |    设置一个异常 future().get()将抛异常    |
|    set_value_at_thread_exit    |    将值传入promise，future.get()将阻塞 直到线程结束    |
|    set_exception_at_thread_exit     |    同上 抛异常用    |
|        |        |

###### 5.2、packaged_task
用于线程间传递方法

方法同promise

###### 5.3、future
用于线程间检索值
获取方式:
async
promise::get_future
packaged_task::get_future

| 方法 | 说明 |
|--------|--------|
|    get()    |    就绪 则获取值 且之后不可用  否则阻塞    |
|    valid    |    当前future对象是否与共享状态相关联    |
|     wait()   |    等待共享状态就绪    |
|     wait_until   |    等待超时    |
|     wait_for   |        |
|        |        |

###### 5.4、shared_future
与future类似
get()可以获取多次

###### 5.5、future_error
抛异常时 catch 的对象

###### 5.6、future_errc
枚举类 定义错误条件

###### 5.7、future_status
wait_for和wait_until的可能返回值

###### 5.8、launch
定义了对异步的调用的启动策略 配合async

launch::async :异步:该函数被一个新线程异步调用，并与共享状态的访问点同步它的返回。
launch::deferred :递延:函数在访问共享状态时调用。

###### 5.9、async
异步调用函数 无需等待fn的执行完成

可以使用 5.8、launch策略

###### 5.10、future_cate
与异常有关

#### 三、



参考:
https://www.jianshu.com/u/9456fecb5f96