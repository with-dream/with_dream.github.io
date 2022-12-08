### 一、init进程
1、解析init.rc文件
通过fork启动守护进程 adb进程、serviceManager进程、surfaceflinger进程 比较重要的是启动了Zygote进程

2、解析系统属性文件 (类似注册表 保存在磁盘的属性)

3、最后进入死循环 变为守护进程 用于重启服务等

### 二、zygote
在AndroidRuntime中注册JNI(Canvas/Bitmap/)并启动虚拟机

然后调用ZygoteInit.java进入java环境:
1. 启动socket 用于AMS请求新建进程
2. 预加载资源、类、公共so文件等
3. 通过fork 启动SystemServer
4. 进入死循环 监听socket 如果收到则在Zygote中调用native的fork创建新进程 最终会调用执行ActivityThread.main 启动新的app

### 三、SystemServer
主要启动系统服务(很多服务 主要是Installer  AMS)
然后创建ActivityThread运行framework-res.apk(系统弹框等公共UI)

启动完服务后 会调用AMS.systemReady 调用SystemUI(状态栏等) 启动Launcher 并发送USER_STARTED广播

