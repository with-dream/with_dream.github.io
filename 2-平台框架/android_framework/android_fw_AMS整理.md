
[TOC]


### 一、Activity启动整体流程

###### 1、启动流程
AMS在SystemServer.run中启动

1. new ActivityManagerService  <br/> 主要做初始化操作 主要为: <br/> **初始化ServiceThread(HandlerThread用于处理消息)**  <br/> 初始化cpu、电量监测等服务   <br/> 栈管理相关的类 <br/> 初始化前后台广播队列(https://juejin.im/post/5c13a61c6fb9a049f746167e)
2. mService.start  <br/> 开启线程 监测cpu

###### 2、主要类

| 类 | 作用 |
|--------|--------|
|    ActivityThread    |   app进程的主线程 用于管理actviity 以及执行AMS的命令    |
|	ApplicationThread	|	BnBinder 用于和AMS进行通信 然后通过handler分发给ActivityThread执行	|
|	ActivityClientRecord	|	ActivityThread中用于保存Activity信息的类 <br/> 其中token为Binder用于唯一标识	|
|	ActivityInfo	|	ActivityClientRecord属性 保存了Manifest信息	|
|	H	 |	将ApplicationThread的消息异步发送给ActivityThread执行	|
|	ActivityStack	|	管理TaskRecord和activity	|
|	TaskRecord	|	用于管理Activity	|
|	ActivityRecord	|	栈的节点 代表一个activity	|
|	ActivityStackSupervisor	|	用于管理ActivityStack	|
|	Instrumentation	|	用于activity的测试类	|
|	LoadedApk	|	保存已加载的apk的信息	|
|		|		|

###### 3、启动activity大体流程
1. Launcher组件向AMS发送一个启动Activity的请求
2. AMS保存要启动的Activity的信息 向Launcher组件发送进入中止状态的请求
3. Launcher组件进入中止状态后向AMS发送已进入中止状态
4. AMS发现Activity组件的进程不存在 则启动新的应用进程
5. 新应用进程启动完成后 向AMS发送启动完成消息
6. 将第二步保存的Activity信息发送给第四步启动的进程 完成activity的启动

###### 4、启动activity流程  (ActivityA启动ActivityB)
https://www.jianshu.com/p/e1153fed5b23

1. A.startActivityForResult会通过Instrumentation获取AMP(ActivityManagerService的客户端实例ActivityManagerProxy)
2. 通过Binder调用AMS.startActivityAsUser  <br/> 最终调用ActivityStackSupervisor.startActivityMayWait
3. ActivityStackSupervisor.startActivityMayWait中会根据Intent在PKMS中查找匹配的Activity 然后创建ActivityRecord 并关联TaskRecord和ActivityStack <br/> 然后将新的ActivityRecord放入栈顶 向ApplicationThread发送暂停事件
4. 调用ApplicationThread.schedulePauseActivity 主要回调ActivityA的onUserInteraction/onUserLeaveHint/<br/>onSaveInstanceState/onPause方法
5. 向AMS发送activityA暂停完毕 接着ActivityStack.startSpecificActivityLocked启动ActivityB 如果进程未启动 则fork一个进程 然后通过反射执行ActivityThread.main 并将B的ApplicationThread与AMS关联
6. 关联完成后 AMS向activityB的ApplicationThread发送启动命令
7. ApplicationThread依次创建Context/Application并关联ActivityB 接着回调onCreate 然后执行handleResumeActivity

AMP用于activity向AMS发送消息
ApplicationThreadProxy用于AMS向activity发送消息(ApplicationThread)

activity的生命周期主要由ApplicationThread进行分发(4/7步)
生命周期的一部分onUserInteraction(手动离开页面)/onUserLeaveHint(分发点击事件)

步骤1-4主要用于暂停activityA  其中步骤3会进行栈的管理
步骤5-7主要启动activityB

###### 4、appToken 唯一标识activity
https://www.jianshu.com/p/5e2efbaa2949
4.1 ActivityRecord中的token
第一次启动activity在创建ActivityRecord的构造中创建Token的实例 继承自IApplicationToken

4.2 WMS中的token
将ActivityRecord中的Token封装为AppWindowToken 用于WMS向AMS发送关于window的消息

4.3 App中的token
ActivityThread中的ActivityClientRecord节点保存了Token 来自ActivityRecord.token

4.4 WindowManager.LayoutParams里的token
传递给WMS 做一致性判断 还是ActivityRecord中的token

4.5 WindowState中的token
WindowState中的token是从WindowManager.LayoutParams传来的 并将其保存在WindowState中
用来作为WindowState的key  还可以发送一些消息 如windowsVisible等

总结：
startActivity时 调用ActivityStackSupervisor.startActivityLocked创建ActivityRecord时也会创建Token
token用于唯一标识一个activity 另外还是一个Bnbinder 实现了WMS向AMS发送消息

###### 5、补充
**5.1 activity生命周期**
actvityA.onPause之后activityB.onCreate
被透明activity或dialog覆盖时 被覆盖的activity.onPause 不会onStop

特殊方法:

| 方法 | 说明 |
|--------|--------|
|    onUserInteraction    |    事件分发到当前actvity时调用 只能捕获按下事件    |
|onUserLeaveHint|activity放到后台时被调用 在onPause之前 <br> 但是来电界面时不会调用|
|onSaveInstanceState|不是主动退出activity时 home键、切屏幕方向 <br/> 在onPause前后执行|
|onRestoreInstanceState|onCreate中执行|
onUserInteraction/onUserLeaveHint主要用于取消通知栏

横竖屏切换:
https://www.cnblogs.com/xiaoQLu/p/3324503.html
常规的横竖屏切换会销毁activity并重新创建 走相应的生命周期
configChanges属性可以控制是否重新创建activity

###### 6、参考：
整体架构:
https://www.cnblogs.com/cynchanpin/p/6786323.html

AMS启动流程:
https://blog.csdn.net/love000520/article/details/70230544
https://blog.csdn.net/yangzhihuiguming/article/details/51700157

### 二、Service
https://blog.csdn.net/zwjemperor/article/details/82949913
###### 1、主要类
ServiceConnection	客户端 监听service连接状态 用于bindService
ConnectionRecord	服务端 Client端与Service端绑定的抽象描述
ActiveServices	用于操作Service
ServiceRecord	对应一个Service
ServiceMap		一个app对应的所有service

###### 2、生命周期
*后台启动* onCreate(执行一次) -> onStartCommand(调用次数对应startService次数) -> onDestory
*绑定方式* onCreate(执行一次) -> onBind(执行一次) -> onUnBind -> onDestory

多次调用bindService 会多次调用ServiceConnection.onServiceConnected

###### 3、主要流程
*3.1 startService*
会在ActiveServices中创建对应的ServiceRecord
如果service已经启动 则onStartCommand
如果进程已启动service未启动 则AMS会向ActivityThread发送消息 通过反射实例化Service 并调用onCreate/onStartCommand
如果进程未启动 则先启动进程 待启动再启动service

*3.2 bindService*
bindService会在ActiveServices中创建ServiceRecord、ConnectionRecord
如果没绑定过 则调用onBinder
如果绑定过 则调用onServiceConnected

如果没绑定过 调用完onBinder之后会调用publishService 将Service中的Bind实例存入ServiceDispatcher中 然后回调ServiceConnection.onServiceConnected完成绑定

###### 4、onStartCommand返回值

| 标记 | 作用 |
|--------|--------|
|    START_NOT_STICKY    |    被杀死之后 接收到新的Intent才启动    |
| START_STICKY | 被杀死之后 会重启 但是Intent为null |
| START_REDELIVER_INTENT | 重启 并使用最后传入的Intent |

###### 5、保活
1、设置onStartCommand返回值
2、提升Service优先级  1000最高值
3、提升Service进程优先级
> 前台进程
> 可见进程	绑定的activity在onPause状态
> 服务进程	startService开启的
> 后台进程	绑定的activity在onStop状态
> 空进程 	 没有数据在运行

4、onDestory中重启Service
5、AlarmManager定时任务
6、绑定一个像素的activity
7、监听系统广播判断Service状态

### 三、广播
###### 1、广播种类
普通广播
有序广播
粘性广播
本地广播
> https://blog.csdn.net/ylyg050518/article/details/71194324
> 本地广播只能在应用内使用 效率更高
> 内部有ReceiverRecord 用于封装注册的本地广播
> 发送时 匹配ReceiverRecord的action/data等 然后调用回调onReceive

###### 2、注册方式
静态	不受生命周期影响 优先级低
动态	跟随生命周期 优先级高

###### 3、注册流程
*3.1 动态广播注册*
调用ContextImpl.registerReceiverInternal 内部会通过ReceiverDispatcher创建一个Binder
然后通过远程调用AMS的registerReceiver 将广播封装为BroadcastFilter并以Binder为键保存起来
如果有粘滞广播 则使用BroadcastQueue进行发送

*3.2 静态广播注册*
PKMS解析Manifast.xml文件时 会将广播解析并保存在PackageParser的内部类Package中

###### 4、发送流程
如果是静态广播 则将广播放入mStickyBroadcasts(用户id,intent)的map中
然后从PKMS查找静态广播接收者  查找动态接收者
对接收者根据优先级进行排序 如果优先级相同 动态接收者优先级高于静态
根据Binder调用ActtivityThread.scheduleRegisteredReceiver发送出去
如果广播是静态注册且进程没启动 则先启动进程再发广播

### 四、ANR
###### 1、Service ANR
ActiveServices在调用Service各生命周期方法之前 会调用bumpServiceExecutingLocked通过handler设置一个超时
如果ActivityThread没有调用AMS.serviceDoneExecuting移除超时 则报ANR

