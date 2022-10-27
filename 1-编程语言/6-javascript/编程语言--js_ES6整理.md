
[TOC]


### 一、ES6
##### 1、var/let/const
1. let只能在当前作用域中使用
2. 如果是全局变量不会添加到全局window中
3. 暂时性死区 即声明了才能使用
4. 不允许重复声明

https://blog.csdn.net/ibliplus/article/details/81047913

const与let类似 只是不能改变且必须初始化
const指向的内存不能改变，对于引用变量，可以改变引用变量指向的内容

##### 2、解构赋值
解构赋值 允许对象和数组
解构赋值允许默认值

解构可以用于返回值、参数、for循环中

##### 3、箭头函数
类似lambda表达式
箭头函数没有自己的this this是从外围的作用域继承过来的

##### 4、this绑定
```javascript
foo::bar(...argument);
等同于
foo.apply(foo, argument)
```

##### 5、扩展运算符
```javascript
var more = [1, 2, 3];
console.log([9, 8, ...more]);//打印9,8,1,2,3
```

##### 6、Null传导运算符
const a = b?.c?.d || 'default'

##### 7、Symbol类型
ES6新增的基本类型 表示独一无二的值
内部有一个[Description]的属性 使用toString()方法时才可以读取这个属性

全局共享
`Symbol.for('字符串')`创建全局共享的Symbol 即多次调用该方法 字符串相同 Symbol相同 全局唯一
`Symbol.keyFor(Symbol)` 返回Symbol的字符串


##### 8、Set/Map
set	类似数组 没有重复数据
WeekSet 只能保存对象 与java类似
Map	键值对
WeekMap	键只能为对象 与java类似

##### 9、Promise
```js
//相当于异步回调 Promise中操作耗时任务 完成后调用resolve/reject回调
function readFile(filename) {
    return new Promise(function(resolve, reject) {
        // 触发异步操作
        fs.readFile(filename, { encoding: "utf8" }, function(err, contents) {
            // 检查错误
            if (err) {
                reject(err);
                return;
            }
            // 读取成功
            resolve(contents);
        });
    });
}
let promise = readFile("example.txt");

promise.then((contents) => {//对应resolve方法
    // 完成
    console.log(contents);
}).catch((err) => {	//对应reject方法
    // 拒绝
    console.error(err.message);
});
```

##### 9.1、async
阻塞任务 简化了生成器的操作
```js
//声明异步任务
async function read() {
	//添加await标记 执行完readFile才继续执行下面的操作
    let contents = await readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
}

read()
```

##### 10、class
```javascript
class Point {
	#x;	//私有属性
    _test() {}	//私有方法

	static y = 1;	//静态属性
    static fun() {}	//静态方法

    constructor(x, y) {}

    toString() {

    }
}

var p = new class {}
//实例化 new不可去除
let point = new Point(1, 2);

//继承  必须重写super 作用是将this指针从父类指向当前类
class PointA extends Point {
    constructor() {
        super(x, y);
    }
}
```

##### 11、修饰器
在编译期 对类和类方法添加@方法  简化代码  类似java注解


##### 12、模块化
import export

##### 13、数组遍历
```javascript
let m = [5, 4, 3, 2, 1];
//1 for in     v为string类型  也可以遍历对象
for (let v in m) {
    console.log((typeof v) + "==>" + v);
}
//2 forEach   不能使用return/break/continue
m.forEach((v, index) => {
    console.log((typeof v) + "==>" + v + "--" + (typeof index) + "==>" + index);
})
//3 for of     v为数组元素的实际类型  推荐
for (let v of m) {
    console.log((typeof v) + "==>" + v);
}
```

##### 14、生成器
```javascript
//function后面需要加*号
let go = function*(x) {
    let a = yield x
    let b = yield x + 1
}
go(10)

let g = go(10)
//调用一次next 会执行一次yield
//next函数返回value和done(是否执行完成boolean)
console.log(g.next())
console.log(g.next().value)
```

##### 15、代理(Proxy)和反射(Reflection)


### 二、函数式编程
##### 1、纯函数
函数不依赖外部变量 只靠传入的参数在函数内部计算 且不能改变外部的状态
可以不受外部干扰以及并行计算

##### 2、科里化及链式调用
2.1 科里化 将多个参数分为多个单参数的函数
```javascript
//将n/m分为两个函数传参 每次传参只有一个实参
function test(n) {
    return function (m) {
        return n/m;
    }
}
let v = test(10);
console.log("==>"+v(3));
```

2.2 链式调用 即方法返回this 然后通过返回的this继续调用函数

##### 3、递归
js的递归属于底层操作，且没有做相关的优化，所以应该避免使用。可以使用高阶函数做相关操作
递归操作有栈限制
js对于尾递归有优化 将尾递归优化为for循环

##### 4、组合
将每个功能使用函数封装 再进行各种组合 完成相关的功能
```javascript
let toUpperCase = (x) => {
    return x.toUpperCase();
};
let exclaim = (x) => {
    return x + '!'
};
//将功能组装在一起
let shout = function (x) {
    return exclaim(toUpperCase(x));
};

console.log("==>" + shout("send in the clowns"))
//=> "SEND IN THE CLOWNS!"
```

##### 5、函子
任何具有map方法的数据结构，都可以当作函子的实现。
函子是一个范畴/容器 如例子中的Container
它包含了值和值的变形关系 并且一般有map方法 将容器里面的每个值映射到另一个容器
一般使用of来生成新容器
```javascript
var Container = function (x) {
    this.__value = x;
}
// 函数式编程一般约定，函子有一个of方法
Container.of = x => new Container(x);
// Container.of(‘abcd’);
// 一般约定，函子的标志就是容器具有map方法。该方法将容器
// 里面的每一个值， 映射到另一个容器。
Container.prototype.map = function (f) {
    return Container.of(f(this.__value))
}
Container.of(3)
    .map(x => x + 1) //=> Container(4)
    .map(x => 'Result is ' + x); //=> Container('Result is 4')
```
Container为一个函子 它有map方法
通过`f(this.__value)`将一个容器**3**转为另一个容器**Container('Result is 4')**

##### 6、例子
```javascript
function lazy() {
//存储的是方法 为局部变量
var calls = [];
//返回对象
return {
    //invoke为闭包 name值可以一直保存
    invoke:function (name) {
        calls.push(function () {
            return name;
        });
        //返回this 形成链式调用
        return this;
    },
    p:function () {
        //惰性方法 执行才会调用
        calls.forEach(value => console.log(value()))
    }
}
};

var l = lazy();
l.invoke('concat', [4, 2, 1, 3])
.invoke('sort')
.p()
```

##### 10、补充
```javascript
let res = function () {
    return function () {
        return 1;
    }
}
console.log("==>" + (typeof res())	//function
console.log("==>" + (typeof res()())	//number
```

### 三、性能
##### 1、运行和加载
https://www.cnblogs.com/ymwangel/p/5980373.html
1.1 将js加载放在最后
浏览器在遇到`<script>`标签就会下载js文件并解析 这两者都会阻塞界面
浏览器在遇到`<body>`标签之前不会进行渲染
如果js文件过多且放在`<head><body>`之间 会影响页面的的响应速度 所有最好放在`</body>`前面

1.2 http请求
下载100kb文件的速度比下载四个25kb文件的速度要快 所以最好合并小文件

。。。

##### 20、参考：
ES6标准入门

js函数式编程
http://www.ruanyifeng.com/blog/2017/02/fp-tutorial.html
https://www.cnblogs.com/tjyoung/p/8976013.html