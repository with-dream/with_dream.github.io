
[TOC]



### 一、基本概念
##### 1、基本类型
typeof 检测变量的类型 主要用于基本类型 类类型检查用instanceof
undefined  声明变量但未初始化
null
boolean
string
number	超出范围Infinity / 如果不是数据返回NaN

object
function

##### 2、语句
with 方便定义object类型的变量

##### 3、引用类型
3.1 object类型
两种初始化方式：
```javascript
var obj = new Object()
var obj = {}		对象字面量初始化 不会调用构造方法
```
3.2 Array类型
数组是object的特殊形式 也是一种object类型
数组类型很灵活，可以同时存放多种类型
初始化：
```javascript
var arr = new Array();
var arr = new Array(n);  n为常数 表示数组的长度
var arr = new Array("a", "b"...);
//可以进行简化
var arr = Array();
var arr = ["a", "b"...];

var arr[99]="a";  arr.length为100
```
数组可以模拟栈: Array.push/Array.pop

模拟队列：Array.shift移除第一个元素并返回  Array.unshift在数组前端添加元素

迭代方法:every(fun{}) fun返回true则返回true
some(fun{}) fun有一个返回true则返回true
filter(fun{}) 返回fun为true的数组
map(fun{}) 返回计算的结果数组
forEach没返回

3.3 function类型
函数名可以理解为指向函数的指针，函数名也是一个变量

### 二、面向对象
##### 1、Object常用方法
https://segmentfault.com/a/1190000011294519

| 方法 | 说明 |
|--------|--------|
|    Object.defineProperty <br/> Object.defineProperties  |   修改对象的数据属性和访问属性     |
| isPrototypeOf | 判断指定对象object1是否存在于另一个对象object2的原型链中 |
| hasOwnProperty | 对象是否有指定属性 仅此对象|
|in| 对象的所有属性，包括对象实例及其原型的属性 |

##### 2、this
https://www.cnblogs.com/pssp/p/5216085.html
原理：http://www.ruanyifeng.com/blog/2018/06/javascript-this.html
this没有作用域限制 嵌套函数不会从调用它的函数中继承this
如果嵌套函数作为方法调用 this的值指向调用它的对象
如果嵌套函数函数作为函数调用 this不是全局对象(非严格模式)就是undefined(严格模式)
```javascript
var o = {
    m:function () {
        var self = this;
        console.log("outer this==>"+(this == o));
        f();

        function f() {
        	//this不是全局对象(非严格模式)就是undefined(严格模式)
            console.log("this==>"+(this == window));
            console.log("self==>"+(self == o));
        }
    }
}
o.m();

打印：
outer this==>true
this==>true
self==>true
```
##### 3、函数
如果函数为一个对象的属性 则为方法

3.3.1 函数的创建
```javascript
//1、函数声明
alert(sum(10, 10))
function sum(num1, num2){}

//2、函数表达式
alert(sum1(10, 10))
var sum1 = function (num1, num2){}

//函数表达式
//此时 sum2可以当做函数调用 但是fun只能在函数内部使用
var sum2 = function fun(num1, num2){}
fun();		//报错 函数未定义

//3、Function 构造函数
var ff = Function('x', 'y', 'z', 'var m = x + y; m+=z; console.log("==>" + m)');
ff(1, 2, 3);

//函数的覆盖 当有两个同名函数时，只会执行最后一个
var cover = function (){
    console.log("==>cover1");
}
function cover() {
    console.log("==>cover2");
}
//解析器会先解析function cover() 再解析var cover
//因为var cover会覆盖 所以打印cover2
cover();	//打印cover2

```
> sum会正常运行，因为解析器会先读取函数声明，并使其在执行任何代码之前可用
>
> sum1为函数表达式，必须等到解析器执行到才会真正被解释
>
> Function构造对象只能使用字符且最后一个参数为函数表达式前面为参数
> Function构造为匿名函数 且每次调用都会new一次 如果多次调用会影响性能
> Function构造不使用词法作用域 函数体代码的编译会在顶层函数执行
> Function构造允许动态编译/执行代码

```javascript
var variate = "gobal"
function test() {
    var variate = "local"
    return Function('return variate');
}
console.log(test()());	//打印 gobal
```

3.3.2 function内部属性
arguments 类数组对象，保存着所有传进来的参数 arguments.callee则指向该函数
this 在全局作用域中 this为window  具体的对象调用方法时，this指向具体的对象


3.1 函数的嵌套与闭包
函数嵌套 可以使内部函数调用外部函数的属性 使变量的作用域发生改变
函数的嵌套尤其是闭包延长内存的占用时间 因为会持有对象不释放

3.3 函数的属性

| 属性名 | 作用 |
|--------|--------|
|    length |    函数形参个数    |
|    prototype    |    指向原型对象    |
|    call/apply    |     间接调用   |
|    bind    |    将函数绑定至对象 与apply类似   |


3.4 实参对象
函数的参数保存在arguments中 它有一个length属性 保存参数的个数 实参对象不是数组
如果实参对象个数不足 会用undefined填充
callee(标准)/caller(非标准) 指向当前函数

3.5 自定义函数属性
函数即是对象 函数可以添加自己的属性

3.6 函数可以作为命名空间使用

##### 4、类和类型

| 方法 | 作用 |
|--------|--------|
|    instanceof    |    是否为实例    |
|    isPrototypeOf    |    是否为类的成员    |
|    constructor    |    --    |

##### 5、高阶函数
高阶函数：将程序拆分并抽象成多个函数再组装回去
1. 以一个函数作为参数
2. 返回一个函数作为结果

高阶函数为Array的方法 用于快速计算

| 函数 | 说明 |
|--------|--------|
|    map(fun(a)    |    对Array中的每个元素使用fun函数 并返回新的数组    |
|    reduce(fun(a, b))    |   结果继续和序列的下一个元素 并返回结果     |
|    filter(fun(a))    |   过滤     |
|    sort(fun)    |    排序    |
|    every(fun)    |    遍历元素 都为true时返回true    |
|    some(fun)    |    遍历元素 有一个为true时返回true    |
|    forEach(fun)    |    遍历    |

##### 6、原型及原型链

6.1 对象与函数对象
js中一切皆对象 函数也是一种对象
```javascript
//对象
var o1 = {};
var o2 =new Object();
var o3 = new f1();
//函数对象
function f1(){};
var f2 = function(){};
var f3 = new Function('str','console.log(str)');
```


![js原型链](./js原型链.jpg)
注：对象也有constructor  图中未画出 因为对象是函数对象的实例继承而来???<br/>且`f1.constructor === Foo.prototype.constructor`

参考: https://www.jianshu.com/p/dee9f8b14771

1. 对象和函数对象都有`__proto__`属性 原型链的真正实现者 <br/> 对象的`__proto__`指向对象构造函数.prototype  <br/> 函数对象的`__proto__`都指向Function.prototype  <br/> `函数对象.prototype.__proto__`都指向Object.prototype <br/> `Object.prototype.__proto__`指向null
2. Function.prototype为函数对象 其他.prototype的为对象
3. 函数对象才有prototype的为对象属性(Function.prototype除外)
4. typeof Function.prototype为function  https://stackoverflow.com/questions/39698919/why-typeoffunction-prototype-is-function
6. typeof null 为object  但是没有`__proto__`和prototype 属于遗留bug  Object.create(null)也没有
```javascript
//参考https://www.cnblogs.com/94pm/p/9113434.html
Object.create =  function (o) {
    var F = function () {};
    F.prototype = o;
    return new F();
};
```

对象属性/方法的访问规则:
> 先在自身属性中查找，找到则返回
如果没有，再沿着`__prop__`这条链向上查找找到返回
如果依然没有找到，返回undefined

Function.prototype继承Object.prototype 其他函数对象继承Function.prototype

1、不使用prototype属性定义的对象方法，是静态方法，只能直接用类名进行调用！另外，此静态方法中无法使用this变量来调用对象其他的属性！
2、使用prototype属性定义的对象方法，是非静态方法，只有在实例化后才能使用！其方法内部可以this来引用对象自身中的其他属性！

### 四、js客户端主要对象
##### 1、window
客户端js的全局类

| 方法 | 说明 |
|--------|--------|
|    setTimeout<br/>setInterval    |    设置定时器    |
|location| 当前页面的url |
|history| 浏览历史 |
|alert|弹框|
|onerror| 捕获为处理的异常 |

##### 2、尺寸
https://www.cnblogs.com/wyx424/p/6713062.html

##### 5、Element
https://www.cnblogs.com/zhaoran/p/3132360.html
选择器：

| 方法 | 说明 |
|--------|--------|
|    getElementById    |    返回指定id的对象   id属性 |
| getElementsByName|返回带有指定名称的对象的集合 name属性|
|getElementsByTagName| 返回带有指定标签的对象集合 标签名 |
|getElementsByClassName| 返回带有指定class的对象集合 class属性 |
|querySelector<br/>querySelectorAll| 返回指定指定css选择器的集合 前一个只返回一个 后一个返回多个<br/> 支持id class 标签等 |

节点、属性操作：

| 方法 | 说明 |
|--------|--------|
|    childNodes<br/>children    |    获取子节点    |
|    firstChild <br/>lastChild<br/> firstElementChild <br/>lastElementChild    |    获取第一/最后一个子节点    |
|    nextSibling <br/>previousSibling<br/> nextElementSibling<br/> previousElementSibling    |    获取上一个/下一个节点    |
|    parentNode    |    获取父节点    |
|    nodeName <br/> nodeValue <br/> tagName    |    获取标签名    |
|    nodeType    |    获取节点类型    |
|    innerHTML <br/> outerHTML    |    内容操作    |
|    parentNode    |    获取父节点    |
|insertBefore<br/>appendChild<br/>removeChild<br/>replaceChild<br/>insertAdjacentHTML<br/>hasChildNodes<br/>cloneNode| 节点操作 |
|attributes<br/>hasAttribute <br/>getAttribute<br/> setAttribute<br/> removeAttribute<br/> hasAttributes|属性操作|

### 补充

##### 1、垃圾收集
https://www.cnblogs.com/zhwl/p/4664604.html
1.1、标记清除 主流的方式
垃圾收集器会在运行时会给所有变量加上标记，当变量及被用到的变量进入环境(即作用域)时会去除标记。有标记的变量将被视为准备删除的变量

1.2、引用计数
记录变量被引用的次数，为0时清除 会有循环引用的隐患

##### 3、Global对象
不属于任何其他对象的属性和方法都是Global的，如isNaN/parseInt
还有encodeURI/eval

##### 4、命名空间
https://blog.csdn.net/bater2356/article/details/80608841

##### 6、yield
使生成器函数暂停执行 用处不大
https://www.jianshu.com/p/36c74e4ca9eb

##### 7、数组推导语法
非标准 尽量不用
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Array_comprehensions

##### 8、事件流
https://www.cnblogs.com/jingwhale/p/4656869.html
W3C规定 事件分为三个阶段
1. 捕获阶段：事件从根节点流向目标阶段 (使用较少)
2. 目标阶段：触发目标
3. 冒泡阶段：事件回溯到根节点 (大多数事件)
事件可以阻止或取消

##### 9、异常
```javascript
//1、异常捕获
try {

} catch(err) {

} finally {

}

//2、抛出异常
throw {}/Error("")

//3、错误事件
window.onerror = function() {}
```