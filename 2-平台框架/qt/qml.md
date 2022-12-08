### 
#### 1、属性

```js
//属性
//type可以为var 表示通用类型  也可以为具体类型 如int，long等
[default] property <type> <name> : <value>


//别名
property alias <name> : <reference>
```

属性可以定义信号

##### 1.2、qml的类型
1. qml类型
| 类型 | 说明 |
|--------|--------|
|    bool    |        |
|    double    |        |
|    enumeration    |        |
|    int    |        |
|    list    |        |
|    read    |    实数    |
|    string    |        |
|    url    |        |
|    var    |        |
|    date    |        |
|    point/rect/size    |        |
2. js类型			var/Array等js标准类型
3. 对象类型		属性、信号、方法等


#### 2、基本元素
可视元素:需要展示内容 如Text、Image
不可视元素:MouseArea

#### 3、组件
可以编写自定义的组件MyButton.qml  然后在别的qml文件中直接引用`<MyButton>`组件

https://blog.csdn.net/qq_40194498/article/details/79853776

#### 4、定位器
Row/Column/Grid/Flow/Repeater

anchors属性可以具体定位一个元素 指定left/right/padding/margins等

#### 5、输入
TextInput:单行输入框  支持焦点
TextEdit:富文本编辑区

#### 6、动画
##### 6.1 主要动画
常用动画：
PropertyAnimation（属性动画） - 使用属性值改变播放的动画
NumberAnimation（数字动画） - 使用数字改变播放的动画
ColorAnimation（颜色动画） - 使用颜色改变播放的动画
RotationAnimation（旋转动画） - 使用旋转改变播放的动画

其他动画：
PauseAnimation（停止动画） - 运行暂停一个动画
SequentialAnimation（顺序动画） - 允许动画有序播放
ParallelAnimation（并行动画） - 允许动画同时播放
AnchorAnimation（锚定动画） - 使用锚定改变播放的动画
ParentAnimation（父元素动画） - 使用父对象改变播放的动画
SmotthedAnimation（平滑动画） - 跟踪一个平滑值播放的动画
SpringAnimation（弹簧动画） - 跟踪一个弹簧变换的值播放的动画
PathAnimation（路径动画） - 跟踪一个元素对象的路径的动画
Vector3dAnimation（3D容器动画） - 使用QVector3d值改变播放的动画

动画元素：
PropertyAction（属性动作） - 在播放动画时改变属性
ScriptAction（脚本动作） - 在播放动画时运行脚本

##### 6.2 状态（States）和过渡（Transitions）
qml的元素有state属性 可以自定义状态
transitions属性可以定义from-to 执行对应于state的动画

#### 7、信号和槽
https://blog.csdn.net/gongjianbo1992/article/details/87937804

##### 7.1 系统槽函数

系统信号和槽为`on<Signal>` Signal需要首字母大写

##### 7.2 自定义信号和槽
```js
//定义
signal signalA([参数])

signalA: {
	//处理函数
}
//触发
root.signalA()
```

##### 7.3 属性改变信号
qml属性（property）值改变时，会自动触发信号
`on<Property>Changed`的形式改变

##### 7.4 附加信号处理程序
##### 7.5 连接任意对象的信号
##### 7.6 信号到方法/信号的连接


#### 8、c++与qml交互
https://blog.csdn.net/gongjianbo1992/article/details/87965925

待整理


#### 参考:
很好的入门教程:https://blog.csdn.net/qq_40194498/category_7580030.html

https://zhuanlan.zhihu.com/TaoQt
https://blog.csdn.net/gongjianbo1992/category_8746723.html
