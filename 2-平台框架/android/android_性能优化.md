
[TOC]

### 一、检测工具
###### 1、MAT
android studio可以导出hprof文件 使用hprof-conv转换 然后使用MAT分析

###### 2、leakcanary
原理：
https://www.jianshu.com/p/7d6946a1bbb6
https://www.jianshu.com/p/261e70f3083f

使用弱引用包装activity 然后监听onDestory
1. 如果onDestory activity将被放入ReferenceQueue
2. 然后判断弱引用 如果被回收 则直接返回
如果没有被回收 则调用gc强制回收
3. 如果还是没有被回收 说明存在内存泄漏
4. 然后dump出内存文件进行分析

###### 3、Lint
检查多余的文件

###### 4、Hierarchy ['haɪərɑːkɪ] Viewer
查看布局

###### 5、Battery Historian
电量测试 5.0以后

###### 6、GT
腾讯开发的测试工具 为单独的apk 也可以继承sdk

###### 7、严苛模式

### 二、优化
##### 1、代码规范
Crash即时上传
合理的架构
统一的代码规范
找到项目瓶颈 针对优化

##### 2、内存
2.1 容易出现问题的地方：
显示图片的界面
网络传输数据的类
缓存的类

2.2 优化建议：
2.2.1 不要在循环中创建过多的临时对象(临时变量需要gc)
2.2.2 减少gc

2.2.3 IO的buffer

2.2.4内存泄漏:需要被回收的对象被引用
单例中引用Activity
资源文件未关闭(file等) 会有缓存
静态属性持有大对象

2.2.5 OOM
对象池 线程池

2.2.6 Bitmap相关操作：
像素宽 x 像素高 x 位数(RGB565 = 2) /1024 = kb
Bitmap.Options  inSampleSize inJustDecodeBounds
2.3之前图片放在java堆中 之后在native中 需要bitmap.recycle 然后置null

2.2.7 强软弱虚

2.2.8 ArrayMap SparseArray

##### 3、UI
使用硬件加速
不要在onDraw中创建对象(调用很频繁)
去除多余的背景颜色 减少绘制次数
减少层级
RecycleView
使用ConstraintLayout布局  (约束布局)
merge/include
避免在onCreate中做耗时操作 避免Application/Activity启动延迟

ANR:输入5s  广播10s  service20s  /data/anr/traces.txt

##### 6、网络
预先获取数据
1.1 对于必定要加载的数据 可以预先加载
1.2 对于有关联的数据 也可以预先加载

批量处理传送和连接

网络链路复用

本地缓存

根据网络做具体操作
可以通过app获取当前网络延迟
如果当前网络状况良好 可以适当预加载数据
如果网络状态很差 可以将一些文件上传、大图片的加载等耗时操作进行优化   如:先将任务缓存 待到网络环境好时 再进行操作

##### 7、电量
Job Scheduler
gps
wakeLock
4G比wifi更耗电

##### 8、apk大小
使用混淆 减少代码体积
插件化
减少枚举的使用
使用系统的资源(颜色/style等)
png优于jpg  可以考虑svg 或者压缩(webp)
