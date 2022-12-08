
[TOC]


### 一、scroller
###### 1、概述
scroller用于视图滚动 滚动依赖于scrollTo
只移动自己的内容(scrollTo的作用)
ViewGroup的scroller 对于子view起作用
View的scroller 对于view中的内容起作用

###### 2、scrollTo原理
`myView.scrollTo(x, y)`会将x/y的值保存在View的mScrollX/mScrollY中
然后调用invalidate 在View.draw中 会调用canvas.translate对mScrollX/mScrollY进行操作

###### 3、scroller原理
3.1 主要类及方法
Scroller类：封装了开始位置/目标位置/插值器等
Scroller.startScroll(int startX, int startY, int dx, int dy, int duration):初始化操作
Scroller.computeScrollOffset:用于计算是否达到目标值
View.computeScroll:用于更新mScrollX/mScrollY值

3.2 使用
```java
public class MyView extends View {
    private Scroller mScroller;

    public MyView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        //初始化
        mScroller = new Scroller(context);
    }

    public void zoom() {
        //初始目标值
        mScroller.startScroll(0, 0, -200, -200);
        invalidate();
    }

    //在View.draw中调用
    @Override
    public void computeScroll() {
        super.computeScroll();
        //判断是否达到目标值 内部会用到插值器 并计算getCurrX/getCurrY值
        if (mScroller.computeScrollOffset()) {
            //对内容进行移动
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
        }
    }
}
```

3.3 原理
在View.draw中会回调computeScroll方法
所以每次invalidate就会调用一次computeScroll

Scroller.computeScrollOffset内部会对开始值/目标值以及时间通过插值器进行计算 结果保存在Scroller中的mCurrX/mCurrY中

这就是scroller整体框架的原理
视图的移动由scrollTo完成

### 二、view.xml解析过程
https://www.jianshu.com/p/eccd8ba87e8b

Activity.setContentView最终也是调用LayoutInflater.inflate

1、LayoutInflater.from(context)
ContextImpl中有SystemServiceRegistry 在其中创建了继承自LayoutInflater的PhoneLayoutInflater
LayoutInflater.from返回的就是PhoneLayoutInflater的实例

2、LayoutInflater.inflate
在其中解析xml 然后调用createViewFromTag 使用反射创建具体view

### 三、View的生命周期
https://www.jianshu.com/p/67bae09479e6

| 方法 | activity |说明 |
|--------|--------|--------|
||onCreate||
|   Constructors     ||   构造方法     |
|  onFinishInflate  || View及其子View从XML文件中加载完成   |
||onStart||
||onResum||
|  onVisibilityChanged      ||   View或其祖先的可见性改变 |
|  onAttachedToWindow      ||    View被附到一个window上     |
|  onWindowVisibilityChanged ||  包含当前View的window可见性改变  |
|  onMeasure      ||         |
|  onSizeChanged      ||     view的大小发生变化    |
|  onLayout      ||         |
|  onDraw      ||         |
||onPause||
|  onWindowFocusChanged      ||    View的window获得或失去焦点    |
||onStop||
||onDestroy||
|  onDetachedFromWindow      ||    View从一个window上分离    |
|  onFocusChanged      |与activity无关|    View获得或失去焦点    |

### 四、view的刷新流程 (UML)
setView内部会调用view.assignParent(this) 设置view的parent

1、View.invalidate 会调用ViewGroup.invalidateChild
2、invalidateChild中最终调用ViewRootImpl.scheduleTraversals
内部使用Choreographer请求vsync
3、最终调用performTraversals 里面会执行performMeasure/performLayout/performDraw

### 五、自定义控件
###### 1、组合控件
多个基础控件进行封装

###### 2、继承自特定控件
如：继承TextView、LinearLayout等

###### 3、继承View/ViewGroup
3.1 需要重写onMeasure/onLayout/onDraw
3.2 封装Scroller实现滚动效果
3.3 attrs.xml自定义属性
在构造方法中用TypedArray接收自定义的属性值
添加xmlns:标记名="http://schemas.android.com/apk/res/包名"
3.4 如果需要销毁对象(线程等) 可以在onDetachedFromWindow中
3.5 继承View或者ViewGroup需要处理padding、margin

### 六、view绘制流程
###### 1、onMeasure
1.1 MeasureSpec 一个32位int值，高2位代表SpecMode，低30位代表SpecSize
```java
SpecMode有三种模式：
MeasureSpec.EXACTLY //确定模式，父View希望子View的大小是确定的，由specSize决定；
MeasureSpec.AT_MOST //最多模式，父View希望子View的大小最多是specSize指定的值；
MeasureSpec.UNSPECIFIED //未指定模式，父View完全依据子View的设计值来决定；
```

1.2 注意事项
1.2.1 DecorView测量时的MeasureSpec是由ViewRootImpl中getRootMeasureSpec方法确定的
1.2.2 LayoutParams宽高参数均为MATCH_PARENT，specMode是EXACTLY，specSize为物理屏幕大小
1.2.3 最终确定控件大小的方法为setMeasuredDimension

###### 2、onLayout
最终确定控件位置的方法为setFrame

###### 3、onDraw
1、绘制背景	drawBackground(canvas);
2、绘制自己	onDraw(canvas)
3、绘制子View	dispatchDraw(canvas) -> drawChild -> child.draw -> 从步骤1开始
4、绘制装饰	onDrawForeground(canvas)	绘制滑动条 前景色之类

#### 七、获取View宽高的方式
1. Activity#onWindowFocusChange()中调用获取
2. view.post(Runnable)将获取的代码投递到消息队列的尾部。
3. ViewTreeObservable.