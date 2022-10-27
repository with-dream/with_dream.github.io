
[TOC]


#### 1、ArrayList
ArrayList 实现了RandmoAccess接口，即提供了随机访问功能

内部使用elementData(Object数组 初始为0 添加一个元素后为10 每次默认增长1.5倍) 和 size实现

每次重新分配内存时 会用Arrays.copyOf[System.arraycopy]高效复制

foreach实现: 语法糖 底层class文件会编译成Iterator实现



注意事项:
1. 快速迭代失败:如果系统出现错误 立即停止操作 即快速失败

  目的是防止多线程下操作List导致数据不一致
  原因是迭代器内部会保存一个全局的modCount值 foreach时会复制expectedModCount ArrayList.remove方法修改的和Iterator.remove修改的expectedModCount不同步

  https://blog.csdn.net/dengxiansuo5846/article/details/102362438

2. for循环遍历删除
5.2节 https://blog.csdn.net/with_dream/article/details/77609072

#### 2、HashMap
http://www.importnew.com/28263.html
###### 2.1 概述
hashmap存储结构为散列表结构(数组是连续的 可以根据O(1)随机访问元素 把数组的下标设置为hash的键) 如果碰撞 会构建为单链表的形式 在java8中 如果单链表长度大于8且散列表容量大于64(小于64会扩容) 会将单链表转化为红黑树结构

每次扩容会size*2 期间会重新计算散列并赋值 很耗性能 所有一开始就选择一个合适的容量

table数组的下标为int 最多盛放2^32 个数据 内存放不下 所以使用(n - 1) & hash  n为数组总长度

数组长度n总是2的次方 因为(n-1)可以保证二进制除了最高位都为1 保证了所有的index都在0~(n-1)的范围

###### 2.2 java7
散列表与单向链表的结构

###### 2.3 java8
散列表与单向链表O(n)和红黑树O(logN)的结构

#### 3、LinkedHashMap
LinkedHashMap中的元素有两种模式 插入顺序/访问顺序(最近访问的放在尾部)

LinkedHashMap继承自HashMap 内部有Entry继承HashMap.Node 并实现了双向循环链表结构(1.8不循环了) 但是存储依然是hashmap的散列表结构

#### 4、CopyOnWriteArrayList
与ArrayList结构类似 但是是线程安全的 使用写时复制

写时复制 写操作时加锁 将旧数组复制然后操作 此时其他线程读的是旧数组
操作完之后将旧数组丢弃 然后让array对象指向新数组 解锁
读数据时 可以一直读 不会有任何影响

array使用volatile修饰

适合读远大于修改操作 因为修改操作会创建新数组 并将旧数组丢弃 很消耗性能

#### 5、ConcurrentHashMap
###### 5.1 概述
结构与HashMap 但是添加了同步操作
另外 插入需要扩容时HashMap先扩容再插入 ConcurrentHashMap先插入再扩容

###### 5.2 java7
结构与java7的HashMap类似 内部实现了分段锁
因为需要支持并发 内部会分配固定大小为16的Segment Segment继承自ReentrantLock Segment内部维护一个桶 用于存放具体的key/value 可扩容

1. 初始化:会创建Segment数组 并创建一个Segment放入Segment[0]
2. put:通过key的hash找到段锁 在段锁中使用tryLock尝试获取锁 获取到则进行插入 否则会自旋一定次数获取锁(单核1次 多核64次) 还是失败的话使用lock
如果段锁没有被初始化 则使用CAS创建
3. get:因为读不存在并发问题 所以没有加锁

###### 5.3 java8
并发时 使用CAS和synchronized 只锁桶中的一个对象 使锁更细致

1. put:
使用CAS获取桶中的第一个节点 如果为空 则直接使用CAS插入
如果不为空 则使用synchronized锁住节点 然后插入链表或树中
如果链表长度大于8 如果容量大于64 则转为红黑树 小于64则扩容

2. get:
与hashmap类似 只是如果是树结构 Node.hash为-2 在继承自Node的TreeBin中进行查找

3. 扩容:
扩容利用多个线程同时进行
新建nextTable为table的2倍 此为单线程操作
然后为每个线程分配16个桶进行操作 进入死循环进行操作
如果旧表的桶为NULL 则使用ForwardingNode进行占位 表示桶已经或者正在被处理
如果为链表/树 则synchronized锁住头节点 然后进行迁移 迁移完成使用ForwardingNod占位 然后获取下一个桶进行操作
如果都完成 则跳出循环 新表nextTable替换旧表

3.1 链表和树在迁移时 会拆成两份 计算桶位置(n-1)&hash 其实就是取余操作
如一个值得hash为17 另一个值hash为9
容量(n)为16时
17hash  1 0 0 0 1 结果(n-1)&hash = 1
9hash   0 1 0 0 1 结果 = 9
(n-1)   0 1 1 1 1

容量(n)为32时
17hash  1 0 0 0 1 结果 = 17
9hash   0 1 0 0 1 结果 = 9
(n-1)   1 1 1 1 1

所以拆分时 如果hash值小于容量 还是放在旧表的位置
如果hash值大于容量 会放在旧表+旧表容量的位置
与HashMap的resize类似
https://www.cnblogs.com/stateis0/p/9062089.html
https://www.jianshu.com/p/2829fe36a8dd

#### 6、BlockingQueue
主要作用是添加了空或者满时阻塞功能 使用ReentrantLock/Condition
ArrayBlockingQueue 连续内存
LinkedBlockingQueue 单链表形式
PriorityBlockingQueue 基于优先级二叉堆 无限大小 为空时会阻塞

#### 7、ArrayMap
https://www.jianshu.com/p/1fb660978b14
继承Map 内部结构:一个数组mHashes保存hash值 一个Object数组mArray保存key(2*index)/value(2n*index+1)

ArrayMap在插入/移除/收缩操作会频繁使用System.arraycopy 如果过于频繁或者数据量过大 效率会降低 但是内存效率高

如果出现了hash冲突，则从需要从目标点向两头遍历，找到正确的index

扩容时 直接复制数组 不用重新计算hash

1. put:通过key计算hash值 然后使用二分查找在mHashes中查找hash值
如果找到 则返回index 然后用新value替换旧value
如果没找到 则是新插入 通过System.arraycopy腾出位置 将hash保存在mHashes中(放在通过二分查找返回的index)  然后key(2*index)/value(2n*index+1)保存在mArray中
如果hash有冲突 会将最新的key/value放在index位置 老的会向后移动

2. get:将key/hash通过二分查找获取index 比对key 如果相同返回 不同则向后查找 直到找到或者一直到结束

#### 8、SparseArray
内部key为int数组mKeys(免去了HashMap键位Integer的拆装箱) value为Object数组mValues

1. put:将key通过二分查找获取index 如果返回正数 表示已经存在 直接覆盖
如果返回负数 且这个index被标记为DELETED 直接复用
如果不能复用 则插入key/value 可能需要扩容

2. remove:删除时 会保存key/value 但是value会赋为DELETED 可以进行复用
在下次gc(一个普通的方法 会压缩数组 并将DELETE的值删除)时回收

#### 9、TreeMap
红黑树结构

#### 10、ConcurrentLinkedQueue