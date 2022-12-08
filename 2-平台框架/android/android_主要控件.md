
[TOC]


### 一、ListView
ListView -> AbsListView -> AdapterView -> ViewGroup

前置：forceLayout：设置标志位
https://blog.csdn.net/fyfcauc/article/details/41895509
LinearLayout A中有View B
A.requestLayout() 不会调用B.onMeasure/B.onLayout
B.forceLayout A.requestLayout则会调用B.onMeasure/B.onLayout

###### 1、复用机制 AbsListView.RecycleBin
属性：
View[] mActiveViews:当前屏幕显示的items
ArrayList<View>[] mScrapViews:准备被复用的items 可以存储多种类型 如果只有一种类型 使用mCurrentScrap
方法：
fillActiveViews:将所有子view保存在mActiveViews中
addScrapView:将超出屏幕的子view保存在mScrapViews/mCurrentScrap中

###### 2、ListView流程
2.1 初始填充过程
AbsListView.onLayout -> layoutChildren -> fillFromTop -> fillDown -> makeAndAddView -> obtainView -> Adapter.getView -> setupChild(填充view)

开始时 因为没有可复用的view 所以每个item需要调用Adapter.getView构建
然后调用setupChild将item填充到ListView中

如果有可复用的view 则会在obtainView中获取mScrapViews 传入Adapter.getView中进行复用

2.2 滑动过程
onTouchEvent -> MotionEvent.ACTION_MOVE -> onTouchMove -> startScrollIfNeeded -> trackMotionScroll -> fillGap -> fillDown

trackMotionScroll:对滑动的处理 如果item超出了屏幕 则放入mScrapViews中
最终调用fillDown -> obtainView 对view进行复用

###### 3、notifyDataSetChanged
内部使用观察者模式 调用后会requestLayout()重绘

参考：https://blog.csdn.net/u012954720/article/details/80664943

### 二、Recyclerview
https://www.jianshu.com/p/c52b947fe064

###### 2.1 绘制流程
主要方法：
dispatchLayoutStep1	处理Adapter的更新/决定动画/保存view信息
dispatchLayoutStep2	主要的处理逻辑(创建view 缓存策略)
dispatchLayoutStep3	重置状态

在onMeasure中 如果设置Recyclerview具体宽高 则略过
否则执行dispatchLayoutStep1/dispatchLayoutStep2

onLayout中 如果执行了onMeasure 则只执行dispatchLayoutStep3 否则执行1~3步
Recyclerview内部会管理这个状态

dispatchLayoutStep2的执行：
1. 首先调用getItemCount 获取item个数
2. 然后多次调用onLayoutChildren
onLayoutChildren中会调用LayoutManager的具体实现(以LinearLayoutManager为例)：
2_1. 调用resolveShouldLayoutReverse判断正反向绘制
2_2. 调用updateAnchorInfoForLayout确定锚点信息
2_3. 调用updateLayoutStateToFillEnd(Start)根据2_1判断绘制顺序
2_4. 调用fill进行绘制
2_4_1. fill内部会依次去Recycler缓存的view进行复用 如果没有则使用Adapter.createViewHolder进行创建

###### 2.2 复用
Recyclerview有四级缓存
1. 一级缓存：mAttachedScrap 当前屏幕显示的item
2. 二级缓存：mCacheViews	  缓存item 不需要bind
3. 三级缓存：mViewCacheExtension	自定义缓存
4. 四级缓存：mRecyclerPool	缓存池 默认创建

mAttachedScrap是对view的复用 使用detachView方法
其他是复用ViewHolder 使用removeView方法
RecyclerPool使用SparseArray存储 key:ViewType value:ArrayList<ViewHolder>
RecyclerPool可多个Recyclerview共用

流程：
onTouchEvent -> scrollByInternal -> LayoutManager.scrollVerticallyBy -> scrollBy -> fill
在tryGetViewHolderForPositionByDeadline中进行相关处理

规则：
1. 如果item被detach 则加入mAttachedScrap
2. 如果item被remoe 则加入mCacheViews/mRecyclerPool
3. 如果mCacheViews满则加入mRecyclerPool中

###### 2.3 ChildHelper
https://blog.csdn.net/fyfcauc/article/details/54175072
RecyclerView在处理消失动画时 动画完成才remove掉
会造成数据被remove掉 但是item还在的情况
ChildHelper主要用来处理这种差异

###### 2.4 添加头部
使用viewtype

### 三、Fragment
###### 1、主要类
Fragment 更像一个容器 存放view及父view
FragmentController 用于分发生命周期
FragmentManagerImpl	根据生命周期做处理以及管理动画
BackStackRecord	管理Fragment添加/删除等操作状态
BackStackRecord.Op用于存储Fragment以及cmd(ADD/REMOVE) 用List管理

Fragment有两种状态：
BackStackRecord用于管理Op类的状态(add/remove/replace)
FragmentManager用于生命周期管理的状态(INIT/CREATE)

FragmentTransaction为接口 实现类为BackStackRecord

###### 2、添加流程
getFragmentManager会返回FragmentManager的实现类FragmentManagerImpl
beginTransaction会返回FragmentTransaction的实现类BackStackRecord

add时 会调用BackStackRecord.doAddOp 将传入的fragment保存在Op类中 并为cmd赋值为OP_ADD

commit时 根据指令操作Op类(ADD/REMOVE)等
然后调用moveToState() 根据生命周期进行处理以及分发

###### 3、生命周期分发
在FragmentActivity中 会创建FragmentController
在activity的各生命周期中 会进行分发 最终会传到FragmentManager中的moveToState进行处理

### 四、webview
内核 4.4以前使用webkit	之后使用Chrome
###### 1、主要类
WebView	主要生命周期(onResume/onPause) url设置等
WebSettings	状态、功能的设置 (缩放、js)
WebViewClient	页面状态及请求管理
WebChromeClient	处理对话框 图标 进度等

###### 2、js交互
@JavascriptInterface注解  4.2版本
addJavascriptInterface		4.2版本以前 注入漏洞(执行任意java方法)

java调js  loadUrl	4.4以前版本
evaluateJavascript	4.4版本

###### 3、问题
webview白板  关闭硬件加速
内存泄漏		动态创建webView 传入context的弱引用
			   将web独立进程 不用则结束进程
js后台执行(动画等)	使用onStop
兼容性			使用第三方内核 如腾讯X5



### 五、SurfaceView/GLSurfaceView/TextureView/SurfaceTexture
https://blog.csdn.net/jinzhuojun/article/details/44062175
###### 1、SurfaceView
SurfaceView虽然在view树中 但是自带独立Surface 可以在单独的线程中进行渲染 主要用于游戏和视频
不能进行变换操作(平移/旋转等)

###### 2、GLSurfaceView
在SurfaceView的基础上添加了EGL的管理 并自带渲染线程

###### 3、TextureView
https://www.jianshu.com/p/a14955e52216
使用硬件加速绘制的view 绘制的内容为纹理
内部有SurfaceTexture

###### 4、SurfaceTexture
不是一个view 主要作用是调用updateTexImage将图像流转换为GL纹理
内部维护一个单独的BufferQueue

### 六、基础布局
###### 1、RelativeLayout和LinearLayout
https://www.jianshu.com/p/8a7d059da746
RelativeLayout调用两次Measure
LinearLayout不是用weight调用一次 使用则调用两次

onLayout onDraw只调用一次

###### 2、ConstraintLayout (约束布局)
扁平视图层次 只有一层嵌套 减少了测量时间已经层次 提高效率
可以使用百分比布局 做到完美适配
https://blog.csdn.net/qq_38520096/article/details/80813994


