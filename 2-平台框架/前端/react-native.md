### 一、基础

##### 1. 命令

 ```
 react-native init 项目名  创建项目
 react-native start				启动项目
 ```

##### 2. 常用组件

##### 3 高级特性

###### 3.1 ScrollView

同android端scrollview

`pagingEnabled=true` 底层使用viewpager 支持水平分页

 

###### 3.2 FlatList

- 动态列表 类似android端recycleView 且已集成上拉刷新 下拉加载功能

- 需要实现data(数据源)和renderItem(item样式)

- keyExtractor用于唯一标识item 增减一条数据时 不需要刷新所有的列表
- `extraData` 除`data`以外的数据用在列表中 [如果需要改变内容 需要先改变地址 再改变值]

###### 3.3 触摸事件

- View
  - onTouchStart()  onTouchMove()  onTouchEnd()  onPress()  onLongPress()  pointerEvents

- TouchableNativeFeedback
  - onPressIn()  onPressOut()  onPress()  onLongPress()

- 

###### 3.4 动画



##### 4 原生桥接



##### 4 优化





参考:

https://www.jianshu.com/u/b1704a5943fe