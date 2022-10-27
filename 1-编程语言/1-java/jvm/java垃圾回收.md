#### 一、引用

##### 1 强

##### 2 软

##### 3 弱

##### 4 虚



###### 6 ReferenceQueue

Reference为软弱虚引用的基类 内部有ReferenceQueue

如果Reference不指定Queue 则被gc时直接回收 如果指定Queue 则对象被gc时 Reference会被添加到Queue中(此时指向的对象已经被回收了) Reference对象本身需要手动清理

Queue中 Reference以链表形式存储 以栈的形式操作头节点



###### 7 Reference 

Reference属性pending 即将要被gc处理的对象 为静态对象



属性discovered 下一个要被处理的对象 当pending被处理完后 会将discovered指向的对象赋给pending处理 即待处理的对象以链表的形式存储



Reference对象状态:

> 1 active 对象可达  如果变为不可达 (如果指定了ReferenceQueue 则进入pending状态 否则inactive状态)
>
> 2 pending 对象不可达 等待入队
>
> 3 Enqueued 对象已经被回收 且Reference对象已经进入ReferenceQueue中 从队列移除后进入inactive状态
>
> 4 inactive 对象被回收 且Reference对象也被处理 对象已经无法再恢复



https://www.jianshu.com/p/c6a4aa09f530  对象的状态

https://www.jianshu.com/p/f86d3a43eec5  ReferenceQueue与Reference





##### 二、垃圾回收



https://www.jianshu.com/p/80fd72f48267 垃圾回收