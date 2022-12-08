
[TOC]

### 一、Choreographer  [,kɒrɪ'ɒɡrəfə]
###### 1、作用
动画主要通过Choreographer驱动 (android4.1)
整体 https://www.jianshu.com/p/dd32ec35db1d
底层源码：https://www.jianshu.com/p/47c866f6fb67

###### 2、概述
android底层每隔16ms产生一个vsync的同步信号
app通过binder向SurfaceFlinger请求 会收到一个信号
Choreographer请求一次 只会回复一次 所以需要不断请求
然后通过vsync信号统一操作input/Animation/Traversal(控制布局和draw)

###### 3、实现
Choreographer通过ThreadLocal实现单例
事件(Runnable或者FrameCallback)封装于CallbackRecord(单向链表结构)
Choreographer内部有一个CallbackQueue 用于维护事件

Choreographer的postCallback向链表添加事件 并通过DisplayEventReceiver.nativeScheduleVsync请求vsync信号

底层会回调DisplayEventReceiver.dispatchVsync分发vsync信号
最终遍历并调用CallbackRecord中的事件(Runnable或者FrameCallback)完成信号的同步
整体是一个生产-消费者模式


### 二、属性动画 (uml)
```java
ValueAnimator animator = ValueAnimator.ofInt(0, 100);
animator.setDuration(5000);
animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator animation) {
        /**
         * 通过这样一个监听事件，我们就可以获取
         * 到ValueAnimator每一步所产生的值。
         * 通过调用getAnimatedValue()获取到每个时间因子所产生的Value。
         * */
        Integer value = (Integer) animation.getAnimatedValue();
    }
});
animator.start();
```

###### 1、概述
1、Interpolator 插值器:控制动画的完成进度变化快慢
2、TypeEvaluator 估值器:将动画进度转换为具体的值(如颜色、宽高等)
3、Keyframe 关键帧:主要属性 开始时间/结束时间/阶段时间 值(颜色等)
4、KeyframeSet 保存关键帧用
5、ValueAnimator 通过AnimatorUpdateListener回调改变控件
6、ObjectAnimator内部在AnimatorUpdateListener的基础上通过反射改变控件的属性
7、可以用Keyframe定制动画

###### 2、主要流程
2.1 ValueAnimator.ofInt 最终调用KeyframeSet的ofInt 并将传入的开始时间/结束时间/阶段时间等保存在多个Keyframe中

2.2 animator.start 会通过Choreographer不断进行调用 在KeyframeSet中通过插值器和估值器计算当前的值 且保存在PropertyValuesHolder中 然后回调onAnimationUpdate

2.3 animation.getAnimatedValue会获取PropertyValuesHolder的值

2.4 在属性动画中 通过AnimationFrameCallback做回调 主要有两个方法
doAnimationFrame做计算 并回调AnimatorUpdateListener
commitAnimationFrame做时间累加

主要看整体框架：
https://www.jianshu.com/p/489abbc15241
主要看时间管理方面：
https://www.jianshu.com/p/46f48f1b98a9

###### 3、补充
Keyframe为抽象方法 可以设置插值器/估值器 有intKeyFrame/floatKeyFrame
keyframe的个数为ofInt(...)参数的个数 最少为2个

### 三、补间动画
###### 1、概述
1.1 补间动画主要有：平移、缩放、旋转、透明度
1.2 补间动画不会实际改变控件的属性 而是通过改变canvas的matrix
1.3 Animation为模板模式 暴露了applyTransformation(time, Transformation)方法供子类实现
1.4 Transformation类中封装了alpha、matrix等属性

###### 2、主要流程
2.1 startAnimation只是调用invalidate 使view进行刷新
2.2 在View.draw中会调用applyLegacyAnimation 进而调用Animation.applyTransformation 进行计算并将结果保存在Transformation中
后续操作Transformatio即可

### 四、帧动画
需要注意OOM

### 五、转场动画
普通转场动画
overridePendingTransition

android5.0新出的动画
主要效果有共享元素(ActivityOptions)、分解、滑动、淡出效果

共享元素内容：https://www.jianshu.com/p/fa1c8deeaa57

### 六、5.0新动画
##### 1、Ripple Effect 水波纹扩散动画
https://www.cnblogs.com/wjtaigwh/p/5282368.html

##### 2、Reveal Effect 揭露动画
以某一点为中心 扩散的隐藏或者显示view
https://www.jianshu.com/p/14d9026c65c9

##### 3、矢量动画
https://www.jianshu.com/p/00f1f6bb96b3

矢量图使用SVG格式 android中有VectorDrawable专门处理
android5.0有AnimatedVector 效果更炫酷









