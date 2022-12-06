[TOC]

#### 一、基本

###### 1、基本概念
groovy同时支持静态类型和动态类型
groovy在使用元编程和动态特性时 相比静态类型会降低10%的性能

def定义的变量可以自动识别类型
支持类似kotlin的 ?. 运算符
支持 .& 运算符 将方法指针赋值给变量
<=>运算符相当于调用compareTo方法
支持解构
循环支持upto/times/step



1.2 范围  用于for/switch等操作

```
1..10 - 包含范围的示例
1 .. <10 - 独占范围的示例
'a'..'x' - 范围也可以由字符组成
10..1 - 范围也可以按降序排列
'x'..'a' - 范围也可以由字符组成并按降序排列。
```

1.3 数据结构

```goovy
def list1 = [1, 2] //类似java ArrayList
def map = ["k1":"v1", "k2":"v2"] //类似java HashMap
```



###### 2、闭包
2.1 闭包写法
```groovy
def tell(closure) {
    closure()
}

//三者作用相同
tell({println("==>tell 111")});

tell(){println("==>tell 222")}

tell{
    println("==>tell 333")
}
//下面两者作用相同
def test(str, closure) {
    closure(str)
}
//闭包为最后一个参数时 可以使用这种写法
test("==>test 111",{println(it)});
test("==>test 222"){str -> println(str)}
```

2.2 科里化
主要使用curry方法 curry传的参数为固定传参
```groovy
def tell(closure) {
    postFortune = closure.curry("arguments");
    postFortune("arg1")
    postFortune "arg2"
}
//普通写法
tell {
    date, arg1, arg2 -> println("==>${date}  ${arg1} ${arg2}")
}

postFortune "arg3", "arg4"
//科里化时，传几个参数 则在闭包中接收几个参数
tell {
    date, arg1, arg2 -> println("==>${date}  ${arg1} ${arg2}")
}
```

2.3 闭包委托
this/owner/delegate关系
https://www.jianshu.com/p/ae10f75b37cf

1. this 指的是闭包定义所在的类
2. owner 指的是闭包定义所在的对象，这个对象可能是类也可能是另一个闭包。【这是因为闭包是可以嵌套定义的。】
3. delegate 指的是一个第三方的（委托）对象，当在闭包中调用的方法或属性没有定义时，就会去委托对象上寻找。

```groovy
class A {
    def f1() { println "A ==> f1"; println("==>"+this)}

    def f2() { println "A ==> f2"; println("==>"+this)}
}

class Ex {
    def f1() { println "Ex ==> f1" }

    def f2() { println "Ex ==> f2" }

    def foo(closue) {
        closue.delegate = new A();
        closue();
    }
}

def f1() { println "global==> f1"; println("==>"+this)}

println("==>"+this)

new Ex().foo {
    f1();
    f2()
}

//打印：
==>Test@759d26fb
global==> f1
==>Test@759d26fb
A ==> f2
==>A@1e04fa0a
```
new Ex().foo中闭包的this为全局上下文  会优先查找全局的f1/f2方法
在Ex的foo方法中 `closue.delegate = new A()` 委托给对象A 因为没找到全局f2方法 所有调用A.f1()方法

2.4 delegate

用于处理闭包属性/方法调用的第三方对象，可以修改

四种代理模式:

OWNER_FIRST : 所有者中的方法优先 ;

DELEGATE_FIRST : 代理优先策略 , 代理中的方法优先 ;

OWNER_ONLY : 只执行所有者中的方法 ;

DELEGATE_ONLY : 只执行代理中的方法 ;

TO_SELF : 只在自身查找 ;



用法：

```groov
class MyDelegate {
    def func = {
        println('hello')
    }
}
def c = {
    func()
}
//闭包使用对象的方法
c.delegate = new MyDelegate()
c.call()
```

使用java实现类似效果:

```java
//反射实现
static boolean callMethod(Object o, String method, Object... args) {
    try {
        Method func = o.getClass().getDeclaredMethod(method);
        if (func != null) {
            func.invoke(o, args);
            return true;
        }
    } catch (Exception ignored) {
    }
    return false;
}
class MyDelegate {
    void func() {
        System.out.println("func");
    }
}
abstract class MyClosure {
    Object delegate;
    abstract void call();
}
MyClosure c = new MyClosure() {
    @Override
    void call() {
        if (!callMethod(this, "func")) {
            callMethod(delegate, "func");
        }
    }
};
//使用反射实现对象的调用
c.delegate = new MyDelegate();
c.call();
```



2.5 尾递归
groovy的尾递归优化为了循环

#### 二、java与groovy互调
java为静态语言 groovy为动态语言
###### 1、联合编译
groovy可以使用groovyc编译为.class文件或者jar文件 也可以编译.java文件

###### 2、groovy调用java
java较好的兼容groovy  可以直接调用

###### 3、java调用groovy的闭包
```java
//groovy闭包
class GroovyClass {
    def defClosure(closure) {
        println "==>use defClosure"
        closure();
    }

    def paramClosure(int value, closure) {
        println "==>use paramClosure"
        closure(value);
    }
}
//java调用
public class JavaTest {
    public static void main(String[] args) {
        GroovyClass instance = new GroovyClass();
        //无参闭包
        Object resDef = instance.defClosure(new Object() {
            //call的返回类型可以随意定
            public String call() {
                return "resDef";
            }
        });
        System.out.println("defClosure==>" + resDef);
        //一个参数的闭包
        Object resParam1 = instance.paramClosure(1, new Object() {
            public int call(int value) {
                return value;
            }
        });
        System.out.println("paramClosure==>" + resParam1);
    }
}
```

###### 4、java调用groovy的动态方法
```java
class GroovyClass {
    //调用不存在的方法时 将被默认调用
    def methodMissing(String name, args) {
        println "call groovy method $name with args:${args.join(', ')}"
        //返回实参个数
        args.size()
    }
}

//java调用
public class JavaTest {
    public static void main(String[] args) {
        //将对象向上转为GroovyObject  GroovyObject有默认的invokeMethod方法 可以调用动态方法
        GroovyObject instance = new GroovyClass();
        Object res1 = instance.invokeMethod("squeak", new Object[]{});
        System.out.println("res1==>"+res1);
        Object res2 = instance.invokeMethod("squeak", new Object[]{"111", "222", "333"});
        System.out.println("res2==>"+res2);
    }
}
```

###### 4、调用groovy脚本
java/groovy都可使用这种方式
```groovy
GroovyShell shell = new GroovyShell();
//主要使用evaluate方法
shell.evaluate(new File("groovy脚本路径"))
```

###### 5、java调用groovy脚本
```java
public class JavaTest {
    public static void main(String[] args) {
        ScriptEngineManager manager = new ScriptEngineManager();
        //使用groovy的脚本引擎
        ScriptEngine engine = manager.getEngineByName("groovy");
        try {
            engine.eval("println '==>hello'");
        } catch (ScriptException e) {
            e.printStackTrace();
        }
    }
}

```

#### 三、元编程
元方法 对类或对象的各个元素，如名称、方法、属性等等，在运行期进行实时变化，如修改方法名、属性名，动态增加方法、属性等等的一类编程
类似java的反射

groovy的类在编译为class文件时 默认实现了GroovyObject

##### 1、groovy的语言模型
了解即可：https://blog.csdn.net/GroovyObject/article/details/5518077

##### 2、类方法的动态处理
invokeMethod：分派以实现和未实现的方法  如果实现了GroovyInterceptable接口 则都会走invokeMethod方法

methodMissing：分派未实现方法

##### 3、对象的动态处理
GroovyObject类中有个getMetaClass返回MetaClass
MetaClass实现了MetaObjectProtocol 可以实现对象的动态处理

| 方法 | 作用 |
|--------|--------|
|    getProperty <br/> setProperty   |    获取/设置对象的属性    |
|  invokeMissingProperty <br/> invokeMissingMethod |    执行未实现的属性/方法    |
|  getMetaMethod <br/>  getStaticMetaMethod    |  获取元方法      |
|  respondsTo <br/> hasProperty |    判断是否有方法/属性    |
|getAttribute <br/> setAttribute| 获取/设置属性 |

##### 4、方法拦截
4.1 GroovyInterceptable
类实现GroovyInterceptable方法 对象调用的方法都会由invokeMethod分发 可以实现环绕通知

```groovy
class Test implements GroovyInterceptable {
    def check() {
        System.out.println("check called")
    }

    def start() {
        System.out.println("start called");
    }

    def test() {
        System.out.println("test called");
    }

    def invokeMethod(String name, args) {
        System.out.println("invokeMethod called");

        //拦截方法
        if(name != 'check') {
            System.out.println("filter check called");
            Test.metaClass.getMetaMethod('check').invoke(this, null);
        }

        //通过字符串方法名获取方法
        def validMethod = Test.metaClass.getMetaMethod(name, args);
        if(validMethod != null) {
        	//分发方法
            validMethod.invoke(this, args);
        } else {
            System.err.println("can not find the method ==>${name}");
        }
    }
}

def test = new Test();
test.start();
test.test();
test.abc()

打印:
invokeMethod called
filter check called
check called
start called

invokeMethod called
filter check called
check called
test called

invokeMethod called
filter check called
check called
can not find the method ==>abc
```

4.2 使用MetaClass进行拦截
```groovy
Test.metaClass.invokeMethod = { String name, args ->
    System.out.println("invokeMethod called");

    //拦截方法
    if(name != 'check') {
        System.out.println("filter check called");
        Test.metaClass.getMetaMethod('check').invoke(delegate, null);
    }

    //通过字符串方法名获取方法
    def validMethod = Test.metaClass.getMetaMethod(name, args);
    if(validMethod != null) {
        validMethod.invoke(delegate, args);
    } else {
        System.err.println("can not find the method ==>${name}");
    }
}
```

##### 5、方法注入
5.1 分类注入方法
使用use关键字 在use的作用域内注入方法 只能是静态方法且只能在use作用域内有效
groovy提供了语法糖 使用@Category()注解替代static
use接收多个分类 如果方法冲突 最后一个参数优先级最高

限制：每次进入use代码块 会检查静态方法并加入方法列表 退出需要清除该作用域 不适合频繁操作

5.2 ExpandoMetaClass注入方法
可以注入属性、方法、静态方法以及构造方法 且全局有效
可以作用于类和实例
```groovy
//注入静态方法
Integer.metaClass.'static'.isEven = { val -> val % 2 == 0 }
//注入对象方法
Integer.metaClass.isEven = { val -> val % 2 == 0 }
//替换构造方法
Integer.metaClass.constructor = { val -> new Integer(val) }
//添加构造方法  这种方式替换构造方法会报错
Integer.metaClass.constructor << { val, tag -> new Integer(val) }
//添加属性
Integer.metaClass.getAge = { -> 20 }
def age = 5;
println "Integer age==>" + 5.age;
//EMC DSL方式添加多个方法
Integer.metaClass {
    toString = {
        val -> val.toString()
    }

    'static' {
        isEven = { val -> val % 2 == 0 }
    }

    constructor = {
        val -> new Integer(val);
    }
}

```

##### 6、 方法合成
6.1 动态的为类添加方法
```groovy
class Test {

}
String methodName = "play";
//使用字符串 动态为类添加方法
Test.metaClass."$methodName" = {
    println "==>add method $methodName success"
}

def test = new Test()
test.play();
//为实例动态添加方法
test.metaClass."work" = {
    println "==>add method work success"
}

test.work();
```

6.2 动态创建类
主要依靠Expando类 可以随意添加属性/方法
```groovy
def car = new Expando();

println "before  "+car.name
car.name = "==>name";
println "after  "+car.name

car.method = {
    println "==>add method success"
}
car.method();
```

##### 7、编译时元编程
在编译时操作属性/方法节点
https://blog.csdn.net/a568478312/article/details/80019128
《groovy程序设计》第16章

