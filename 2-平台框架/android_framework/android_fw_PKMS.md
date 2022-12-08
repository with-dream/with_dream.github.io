#### 一、启动
1、SystemServer.startBootstrapServices将创建Installer 用于创建socket 和Installd建立连接

2、接着PKMS.main中new PKMS 并SM.addService(PKMS)
主要为PKMS的构造函数中：
1. 初始化Settings(权限管理相关) 构造时 创建了packages.xml 用于保存package信息
2. 创建PackageHandler 用于apk的安装和卸载
3. SystemConfig会读取系统权限xml以及系统配置
4. 扫描目录scanDirLI 最终调用PackageParser.parseBaseApk解析manifest文件 创建package对象 其内包括Application及四大组件数据结构 还会将apk的资源加载到AssetManager 最终将四大组件信息保存在PKMS全局(manifest解析的)
扫描时 会依次扫描系统目录 所以预装apk越多 启动越慢
扫描时 会在/data/data根据包名创建相应的目录
5. 最后创建PackageInstallerService服务

#### 二、安装流程
adb install命令实际会调用install_app方法
install_app中会将apk文件复制到/data/local/temp或者sd卡
然后使用pm命令进行安装 pm命令最终会获取PKMS的代理 并调用installPackageAsUser

在PKMS.installPackageAsUser中 创建InstallParams并发送INIT_COPY到PackageHandler中
PackageHandler.INIT_COPY首先会启动并绑定DefaultContainerService(用于复制apk文件的服务) 然后发送MCS_BOUND
MCS_BOUND主要调用
1. handleStartCopy 解析apk 找到合适的安装位置(内部还是sd卡) 然后createInstallArgs创建具体的安装策略(内部存储FileInstallArgs 外部AsecInstallArgs) 并调用copyApk 通过DefaultContainerService将/data/local/temp中的apk拷贝/data/app目录下
2. handleReturnCode 使用scanPackageLI解析apk 然后发送PACKAGE_ADDED，ACTION_PACKAGE_REPLACED广播

#### 三、Intent查询
显示Intent 携带component信息(四大组件类名) 直接获取符合的组件
半隐式		指明包名 会查找该包名所包含的组件
隐式		 会查找PKMS保存的所有组件 主要是IntentFilter的Action/Category/Data

#### 四、Installd
Installd为native守护进行 主要负责apk的安装/优化/移除等操作 因为操作/data/data需要root权限  DefaultContainerService只是负责复制apk文件而已


参考:
https://blog.csdn.net/huilin9960/column/info/23864

