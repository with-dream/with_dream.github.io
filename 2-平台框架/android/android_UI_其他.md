
[TOC]



### 一、点击事件分发
###### 1、主要方法
收到DOWN/UP两个事件才是一个完整的
如果只有DOWN UP事件不在控件上 会触发ACTION_CANCEL

dispatchTouchEvent	activity/ViewGroup/View都有
onTouchEvent		activity/View
onInterceptTouchEvent	ViewGroup
superDispatchTouchEvent	activity 用于自定义window 如dialog
OnTouchListener		View 优先于onTouchEvent方法

###### 2、GestureDetector
工具类 内部测量速度使用了VelocityTracker
实现了滑动/滚动/单击等回调

###### 3、VelocityTracker
测量手指移动速度

### 二、适配
###### 1、屏幕适配
1.1 主流屏幕尺寸
720 x 1280
768 x 1280
800 x 1280
1080 x 1920
1440 x 2560

###### 2、基础
屏幕尺寸  1英寸=2.54cm   屏幕的对角线长度
屏幕分辨率	像素个数 px
像素密度	dpi 每英寸上的像素点数
dp/sp	一般字体用sp 长度用dp  sp会根据系统字体大小变化而变化

###### 3、适配方案
图片会根据像素密度确定 如xxhdi等 考虑包大小 一般用xx即可
尽量使用wrap_content、match_parent、weight（权重）
.9图片  svg图片
最小宽度限定符	layout-sw600dp, values-sw600dp  sw=Small-Width(设备上最小的一边)
屏幕方向限定符	values-land		values-port

###### 2、版本适配
Build.VERSION.SDK_INT获取当前系统版本

1. android6.0 出现了动态权限 需要动态设置
2. android8.0	手动取消App启动白屏或者黑屏的时候，将Splash界面设为了透明，然后这个时候又设置了方向为垂直，从而导致了这个问题。  设置theme
3. 广播 android8.0 部分静态广播不能使用 可以使用动态广播或者发送时指定包名

###### 3、ROM适配
从相机/相册获取图片旋转270/90度 三星手机  强制旋转

### 三、沉浸式
https://www.jianshu.com/p/752f4551e134

android4.4才出现的全屏沉浸式
1. android4.4以下 可以设置全屏 隐藏状态栏
2. android4.4 使用FLAG_TRANSLUCENT_STATUS设置状态栏为透明且全屏模式 添加一个与StatusBar 一样大小的View实现沉浸式
3. android5.0 getWindow().setStatusBarColor即可实现 对于图片 可以设置颜色为透明
4. android6.0 可以改变状态栏的字体颜色 因为状态栏字体为白色 如果设置背景为白色看不到文字 可以设置SYSTEM_UI_FLAG_LIGHT_STATUS_BAR
5. 有些厂商会定制statusbar 如魅族 小米等

android:fitsSystemWindows 绘制时会考虑系统控件 避免重叠

可以使用StatusBarUtil库 已经做好了兼容

### 四、图像处理
###### 1、图片占用内存
宽像素 x 高像素 x 字节(ARGB8888 为4) = Byte = /1024 = kb = /1024 = Mb

常用格式ARGB_8888、ARGB_4444、ARGB_565

###### 2、Bitmap相关处理
BitmapFactory.Options 工具类
options.inJustDecodeBounds=true 只加载图片的信息到内存 可以获取图片的宽高信息
options.inSampleSize 压缩比例 可以减少内存占用

###### 3、图片压缩
质量压缩：Bitmap.compress
采样率压缩:BitmapFactory.Options.inSampleSize
矩阵压缩：使用Matrix
RGB:降低RGB

###### 4、常用API
BitmapRegionDecoder 可以只加载图片的一部分
Xfermode			图片混合

### 五、相机

