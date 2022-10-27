### 一、基础
##### 1、关键字
let	声明常量
var	声明变量

swift支持元组与解构
`let (a, b, c) = ("123", 456, false)`

合并空值运算符
`a ?? b` 相当于 `a != nil ? a! : b`

区间运算符及半开区间/单侧区间

支持直接打印变量
`print("\(变量名)")`

lazy延迟加载

属性观察器willSet/didSet

final可以防止重写

支持泛型

控制级别 open，public，internal，fileprivate，private

支持自定义运算符`postfix operator{}`
左prefix	右postfix	中infix
_ _ _


集合支持[]操作符  有Array Set Dictionary
支持for in
支持差集/合集等操作

_ _ _


swift中没有do while 有repeat while
switch支持元组  并且支持where子句

_ _ _


函数：
支持元组
可以使用默认值
可以指定标签(啰嗦)
支持函数类型 即函数指针
可以在函数内部定义函数 即内联函数
```swift
func test(param:String) -> (x:Int, y:String) {
    return (1, "abc");
}

//返回函数类型 类型为(Int)->Int
func test(param:String) -> (Int)->Int {
}
```

_ _ _


闭包
https://blog.csdn.net/longshihua/article/details/51384113
逃逸闭包：闭包作为一个实际参数传递给一个函数 逃逸闭包需要@escaping标记 可以调用self 非逃逸则不能

自动闭包:不接收参数的闭包 简化代码

```swift
{ (parameters) -> (return type) in
    statements
  }

//闭包可以简略 通过$0 , $1 , $2代表实参
names.sorted(by: { $0 > $1 } )

//尾随闭包 闭包为最后一个参数时 可以省略  类似groovy
names.sorted{ $0 > $1 }

```

_ _ _


枚举：
https://www.jianshu.com/p/9c7a07163e5b
枚举可以像类一样操作
```swift
enum Movement:Int {
    case left = 0
    case right = 1
    case top = 2
    case bottom = 3
}
```

_ _ _

方法：
对象方法 有隐含的self
类方法：http://www.hangge.com/blog/cache/detail_520.html
静态方法也有隐含的self 指向类本身
_ _ _

垃圾回收
swift的垃圾回收用的是引用计数 会有循环引用的问题
弱引用weak  无主引用unowned
闭包循环引用的解决https://www.cnblogs.com/WJJ-Dream/p/5830147.html

_ _ _

异常处理：
try、try?、try!、catch、throw、throws
try配合do/catch使用
try? 如果try后面崩溃 会将nil赋值给左值
try! 如果try后面崩溃 直接停止运行
defer语句相当于java的finally
```swift
do {
	try expression
    statements
} catch pattern 1 {
    statements
} catch pattern 2 where condition {
    statements
}
```

_ _ _

类型转换：
is	类型检查
as/as?	向下转型
Any	任何类型 相当于id	AnyObject类类型的实例

_ _ _

扩展：
类似oc的类别
可以添加新的构造方法
```swift
extension 已有类 {
    // 为 SomeType 添加的新功能写到这里
}

extension SomeType: SomeProtocol, AnotherProctocol {
    // 协议实现写到这里
}
```
_ _ _
协议：
协议可以提供默认的实现
```swift
protocol SomeProtocol {
    // 这里是协议的定义部分
}
```
_ _ _


参考：
https://github.com/qq20539350/swift4
