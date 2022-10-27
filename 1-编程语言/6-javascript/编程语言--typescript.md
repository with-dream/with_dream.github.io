
[TOC]

### 一、基本类型

| 类型 | 备注 |
|--------|--------|
|    boolean    |        |
|    number    |    都是浮点类型    |
|    string    |    嵌入字符 `${ expr }`    |
|    T[]/Array<T>    |    数组    |
|    let x: [T, T]    |   元组定义     |
|    enum    |        |
|    any    |     任意值   |
| null/undefined | 所有类型的子类  |
| never | |

备注:
--strictNullChecks标记 可以使实例不能为null/undefined

类型断言:
<T>param    类似java强转
param as T  将param强转为T类型

### 二、类与接口
```js
//定义接口
interface Person{
    name: string,
    age: number,  // 必选属性
    job?: string, //可选属性，表示不是必须的参数，
    readonly salary: id,  //表示是只读的属性,但是在初始化之后不能重新赋值，否则会报错
    [ propName : string ] : any,  // 任意类型
    (souce:string):boolean; 	//函数 参数为string 返回bool
}

//定义一个变量，它的类型时接口Person，这样就约束了接口的内容
let person: Person = {
    name: 'jack',
    age: 28,
    job: 'IT dog',
    id: 9872,
    salary: 9999,
}
```

类可继承接口
private修饰的属性，类的外部无法访问
支持static属性
构造函数为constructor
支持默认参数

```js
let myAdd: (x:number, y:number) => number =
    function(x: number, y: number): number { return x + y; };

##### 2、函数类型
let myAdd: (x:number, y:number) => number
//函数实现
function(x: number, y: number): number { return x + y; };
```

##### 3、泛型
```js
function identity<T>(arg: T): T {
    return arg;
}
```

泛型支持约束 extends

### 三、高级类型
##### 1、交叉类型
`let x:Person & Serializable`
x的类型为Person和Serializable
用于属性的赋值

##### 2、联合类型
```js
fun(x:T | U){}
x只能为T或者U类型
```
用于函数的参数和返回值

### 四、symbol
原生类型 唯一且不可改变

### 五、迭代
```js
for..of  	//迭代对象的值
for..in		//迭代对象的键列表 即对象属性
```

### 六、模块
```js
//导出
export	T
export { T };
export { T as 别名 };
export default 默认导出

导入
import { T } from "./文件名";
```