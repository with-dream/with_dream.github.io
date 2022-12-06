
[TOC]

##### 一、object关键字的使用
###### 1.1、对象表达式
https://blog.csdn.net/IO_Field/article/details/52937646

```kotlin
//普通形式
val a = 10
val listener = object : Info("submit"),IClickListener {
    override fun doClick() {
        println("a:$a")
    }
}
listener.doClick() // 打印 a:10

//只保留对象的形式
val adHoc = object {
    var x: Int = 0
    var y: Int = 0
}
print(adHoc.x + adHoc.y)
```

###### 1.2、对象声明
object之后指定了一个名称, 那么它就不再是“对象表达式”，而是一个对“对象声明”,是类的变种 且是单例

```kotlin
object MyInfo: Info("submit"),IClickListener {
    override fun doClick() {
        println("MyInfo do click, $text") // Log: MyInfo do click, , submit
    }
}

fun main(args: Array<String>) {
    MyInfo.doClick()
}
```

###### 1.3、伴随对象
java静态全局对象在kotlin中的一种实现  主要是关键字companion
```kotlin
class M {
    companion object{
    	//var 表示全局静态变量
        var a: Int = 10;
        //var 表示全局final静态变量
        val b: Int = 100;
    }
}
fun main(args: Array<String>) {
    M.a = 100;

    println(M.a)
    println(M.b)
}
```

###### 1.4、对象表达式与对象声明
对象表达式在使用的地方被立即执行
对象声明是延迟加载的， 在第一次使用的时候被初始化



###### 1.5、class

```kotlin
class A [constructor] (_id:Int, _name:String) {//主构造函数
  init {
    //主构造函数
  }
  
  constructor(_age:Int) {	//次构造函数
    
  }
}

主构造函数声明属性
类级别的属性赋值
初始化块里属性赋值
次构造函数里属性赋值
```



##### 二、扩展
###### 2.1、扩展函数
```kotlin
//1、扩展方法 形如
fun receiverType.functionName(params){
    body
}

//例：
open class Person {
}
fun Person.doFly() {
    println("Person do fly")
}
//使用
Person().doFly()

//2、扩展属性
val List.lastIndex: Int
get() = 100

//3、伴随对象的扩展
fun Person.Companion.doSwim() {
    println("伴随对象的扩展函数")
}
val Person.Companion.no: Int
get() = 10
```
> 扩展出来的是普通方法 不是静态方法
>
> 扩展属性 只能指定val   且需要使用后端属性赋值
>
> 当扩展方法与类的方法相同时 优先使用类的方法
>
> 当多个继承关系的类扩展相同的方法 具体调用由调用方法实例的类型决定 A类型调用A的扩展方法 B类型调用B的扩展方法
>
> 可以在另一个类定义扩展，但是作用域仅在定义的类中。当扩展方法与另一个类的方法相同时 优先调用扩展方法 如果想指定另一个类的方法 可以使用this@label指定

##### 三、函数
###### 3.1、不定参数  使用vararg修饰
```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts 是一个 Array
        result.add(t)
    return result
}
```

###### 3.2、高阶函数
将函数作为参数  函数也有类型



如果函数的最后一个参数是函数或lamda 则可以将参数函数提到函数外面

```java
定义: fun test(block: () -> Unit) {}
使用: test() { block函数的实现 }
```



###### 3.3、Lambda表达式
```kotlin
/**
	Lambda 表达式用大括号括起,
	它的参数(如果存在的话)定义在 -> 之前 (参数类型可以省略),
	(如果存在 -> 的话)函数体定义在 -> 之后.
    如果只有一个参数 则省略唯一一个参数的定义
*/
val sum = { x: Int, y: Int -> x + y }
```

###### 3.4、kotlin支持匿名函数

###### 3.5、闭包
https://www.jianshu.com/p/1afdb506e766
> 1.闭包指的是函数的运行环境
 2.闭包可以持有函数的运行环境
 3.函数内部可以定义函数
 4.函数内部也可以定义类
 5.在函数中返回一个函数，被返回的函数可以调用主函数的属性
 6.闭包会大量消耗内存
>
 原理：闭包会持有函数或类的引用，导致不会被回收 从而可以保持闭包内的变量可以继续使用 ???

```kotlin
fun makeFun(): () -> Unit { //返回值为一个无参无返回值的函数
    var count = 0
    return fun() { //返回一个匿名函数
        println(++count)
    }
}
//使用
val x = makeFun()
x()
```

##### 四、委托
https://www.jianshu.com/p/a70ba6436e75
https://blog.csdn.net/IO_Field/article/details/53374809

###### 4.1、方法委托
简化java中的代理模式(也叫委托模式) 主要通过by字段
```kotlin
// 创建接口
interface Base {
    fun print()
}

// 实现此接口的被委托的类
class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

// 通过关键字 by 建立委托类
class Derived(b: Base) : Base by b

//print即是代理方法
fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).print() // 输出 10
}
```

###### 4.2、属性委托
> 延迟加载属性(lazy property): 属性值只在初次访问时才会计算,
  可观察属性(observable property): 属性发生变化时, 可以向监听器发送通知,
  将多个属性保存在一个 map 内, 而不是保存在多个独立的域内.

###### 4.3、延迟属性
lazy() 是一个函数, 接受一个 Lambda 表达式作为参数, 返回一个 Lazy <T> 实例的函数，返回的实例可以作为实现延迟属性的委托：
第一次调用 get() 会执行已传递给 lazy() 的 lamda 表达式并记录结果， 
后续调用 get() 只是返回记录的结果。

```kotlin
val lazyValue: String by lazy {
    println("computed!")     // 第一次调用输出，第二次调用不执行
    "Hello"
}

fun main(args: Array<String>) {
    println(lazyValue)   // 第一次执行，执行两次输出表达式
    println("--------")
    println(lazyValue)   // 第二次执行，只输出返回值
}
执行输出结果：
computed!
Hello
--------
Hello
```

##### 五、集合
https://www.jianshu.com/p/d627bf3d5d72
https://blog.csdn.net/IO_Field/article/details/53407829
https://blog.csdn.net/IO_Field/article/details/53446740

###### 5.1、常用集合
| 可变集合 | 只读集合 |创建| 说明|
|--------|--------|--------| --------|
|    MutableCollection    |    Collection    ||
|    MutableSet    |    Set    |emptySet <br/>  setOf <br/> mutableSetOf|hashSetOf <br/> linkedSetOf <br/> sortedSetOf|
|    MutableList    |    List    |listOf <br/> mutableListOf||
|    MutableMap    |    Map    |mapOf(Pair(k,v),...) <br/> mutableMapOf  <br/>  emptyMap | hashMapOf(k to v) <br/> linkedMapOf |

> kotlin中的Collection/List/Map/Set只是与java兼容 两者并不是一个
  kotlin的mapOf和mutableMapOf()是基于Java的LinkedHashMap。
  kotlin不支持TreeMap 、WeakHashMap 、IdentifyHashMap

###### 5.2、集合的操作

| 方法 | 例子 | 说明 |
| --- | --- | --- |
|    any    | listBook.any { it.page > 22 } |   集合中至少有一个元素与指定条件相符，则返回true    |
| all | listBook.all { it.page >= 20 } | 集合中所有元素与指定条件相符，则返回true |
|fold|listBook.fold(0, 累加函数)|集合从第一个到最后一个元素的操作结果进行累加，并加上初始值|
|foldRight|listBook.foldRight(0, 累加函数)|与fold类似，不同的是从最后一个元素到第一个元素|
|reduce|listBook.reduce (::pageReduce)|与fold类似，不同的是其结果不包括初始值，只是将对集合从第一个元素到最后一个元素的操作结果进行累加。|
|reduceRight||同reduce，不同的是从最后一个元素与其前面的元素按照某个条件相加|
|forEach|||
|forEachIndexed|listBook.forEachIndexed{index, book ->{}|可以获取元素索引|
|count|listBook.count {it.page > 23}|集合与指定条件相符的元素个数|
|maxBy|listBook.maxBy (::maxPage)|返回使指定函数产生最大值的第一个元素|
|minBy|||
|none|listBook.none { it.page > 50 }|如果没有元素与指定条件相符，则返回true|
|sumBy|listBook.sumBy(Book::page)|返回集合中元素进转换函数产生值的总和|
|==>筛选方法|||
|drop|listBook.drop(2)|返回集合中除去前N项的所有元素。|
|dropLast||返回集合中除去最后N项的所有元素|
|dropWhile||除去第一个不满足条件元素的以前的所有元素。<br/>如果所有元素都满足条件，将返回一个空的list.|
|dropLastWhile|||
|filter|predicate: (T) -> Boolean|返回所有与指定条件相符的元素列表|
|filterNot|||
|filterNotNull|listBook.filterNotNull()|返回所有元素列表，但不包括null元素|
|slice||返回指定索引的元素列表|
|take||返回前N个元素列表|
|takeLast||返回最后N个元素列表|
|takeWhile||返回满足指定条件第一个元素列表|
|takeLastWhile|||
|==>映射操作|||
|flatMap||将每一个元素转换为一个新的对象，并创建一个新集合|
|groupBy||返回一个映射表，该表包括经指定函数对原始集合中元素进行分组后的元素|
|map||返回一个列表，该列表包含对原始集合中每个元素进行转换后结果|
|mapIndexed||返回一个列表 带索引|
|mapNotNull||去除null元素|
|==>元素操作|||
|elementAt<br/>elementAtOrElse<br/>elementAtOrNull|||
|first<br/>firstOrNull||返回与指定条件相符的第一个元素|
|indexOf<br/>indexOfFirst<br/>last<br/>lastIndexOf<br/>lastOrNull|||
|single<br/>singleOrNull||返回与指定条件相符的单一元素。如果没有或有多个相符的元素，则抛出异常|
|==>生成操作|||
|partition||将原始集合拆分两个集合 一个为true 一个为false条件的集合|
|plus||相当于addAll操作|
|zip||两个集合中相同索引元素建立的元素对 长度由最短的集合决定|
|==>排序操作|||
|reverse<br/>sorted<br/>sortedBy<br/>sortedDescending<br/>sortedByDescending||带by的可以根据条件|

##### 六、高阶
###### 1、kotlin lambda表达式原理及优化
编译时 通过创建FunctionN类 并将lambda表达式重写于invoke中实现 可用inline优化
https://blog.csdn.net/u013064109/article/details/80139556

###### 2、kotlin支持注解 且与java兼容
https://www.jianshu.com/p/7ec60cc61c7b

###### 3、DSL 很好用的语法糖
讲解：https://www.jianshu.com/p/f5f0d38e3e44
例子：https://www.jianshu.com/p/f931a5621ebe  需要注意dialog.apply的用法

```kotlin
class Dependencies {

    fun compile(coordinate: String) {
        println("add $coordinate")
    }

    operator fun invoke(block: Dependencies.() -> Unit) {
        block()
    }
}

fun main(args: Array<String>) {
    val dependencies = Dependencies()
    dependencies {
        compile("==>test")
    }
    //等价
    dependencies.compile("==>test")
}
```

###### 4、kotlin内置作用域函数
https://blog.csdn.net/u013064109/article/details/78786646

| 函数名 | 作用 |
|--------|--------|
|   with |    同一个对象调用多个方法 可以省去对象名    |
|   let  |    返回结果 带参数 类似lamda表达式    |
|   run     |    返回结果 不带参数 使用this指代调用对象    |
|   apply     |    与run类似 不带参数 返回对象本身 支持链式操作    |
|   also     |    与let类似 有参数 返回对象本身 支持链式操作    |
| takeIf | 作用域内部为true 返回对象本身 false 返回null |



##### 5、kotlin协程

https://blog.csdn.net/yechaoa/article/details/105294162  入门理解

https://blog.csdn.net/qq_33394088/article/details/100636425 总结



##### 补充

1、kotlin默认可见性一般为 public

2、后端变量/后端属性  感觉没用 会使代码混乱
https://blog.csdn.net/IO_Field/article/details/53207125

3、kotlin支持运算符重载

4、kotlin 数据类   `data class A(val name:String, var age:Int);`

5、inner定义内部类

6、支持枚举 用法与java类似   `enum class enumName{}`

7、解构 `var (name, age) = person`

> 只有data class声明的类支持结构 普通类不支持
>
> 对于不需要的变量 可以使用"_"替代或者直接忽略

8、使用 :: 操作符来实现函数的引用 this::可以简化为::

9、inline  将频繁调用且代码量不大的代码块在编译时替换 用空间换时间

10.下划线  替代解构、lambda函数的参数名称

11、类型别名 https://www.jianshu.com/p/4968ed493d2d

12、kotlin/java互调

@JvmName 定义类名  @JvmField 访问变量  @JvmOverloads 方法重载 

@JvmStatic 定义静态方法  @Throw 捕获kotlin异常

13、中缀调用 (鸡肋)

> infix修饰的方法
>
> 限制: 参数只能有一个
>
> ```kot
> //声明 使用infix修饰
> infix fun <T> T.into(other: Collection<T>): Boolean = other.contains(this)
> //使用
> boolean res = "a" into "b"
> ```

14、顶层函数

类中的static方法使用方便 但是过多会导致类膨胀

kotlin可以直接在文件中定义方法 使用时直接导入方法即可 解决类膨胀的问题 其实编译后会自动添加一个类

15、range (CharRange、LongRange、IntRange)

```kotlin
val aRange: IntRange = 1..1024 // 闭区间 [0,1024]
val bRange: IntRange = 1 until 1024 // 半闭区间 [1,1024)
//判断是否包含
val isContains: Boolean = bRange.contains(1024) // false
val isIn: Boolean = 1024 in bRange // false
//迭代
for (i in aRange step 10) {
    print("$i, ")
}
```





##### 参考

https://blog.csdn.net/io_field/article/category/6461077







object

by



协程 实现

kotlin-io

ktx扩展库