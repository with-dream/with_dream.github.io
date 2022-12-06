
[TOC]


### 一、关键字
##### 1、static
1 修饰内部类:
不持有外部类的引用
只能访问外部类的静态成员

2 修饰方法:
类的方法，不属于任何实例，不能调用非静态变量，方法。不能使用this，super
父子类有static相同的方法时，为隐藏

3 修饰变量:
只在方法区中的静态变量区分配一次内存

4 代码块:
第一个被执行

##### 2、final
1 修饰类：
类不能被继承

2 修饰方法
修饰的方法不能被重写
如果方法被private final修饰，则子类定义的同名同参数方法是子类的方法，与本类无关

3 修饰变量
必须显示初始化，变量不可更改
如果修饰基本变量，则值不能更改。如果修饰的是对象或数组，指向的对象和数组不能修改，但内容可以修改
final空白：声明变量，使用时才赋值

##### 3、this
代表当前对象的引用
引用构造方法

##### 4、super
代表父类的引用
引用父类构造，必须放在第一行

### 二、泛型
##### 1、基本使用
泛型是将类型当做参数 使用时传入具体的类型
1. 泛型可以用于接口、类、方法、静态方法的参数和返回
2. 泛型支持super/extends 以及通配符`?`
3. 类型擦除即编译时进行泛型类型的检测 运行时删除泛型 如果想绕过 可以使用反射
4. 用于静态方法在返回类型前需要额外的泛型类型
`public static <T> void t(T t)}{}`

```java
class A<T1, T> {}
//继承时 必须继承所有的泛型
class B<T1, T> extends A<T1, T> {}
//或者具体化部分类型
class B<T1> extends A<T1, Integer> {}
//不支持多态 即要么同时用A 要么同时用B类型
List<A> list = new ArrayList<B>();
//不支持数组  ERROR
List<A> list[] = new ArrayList<B>[];
```

5. 继承

   ```java
   //全部继承
   class Child<T1,T2,T2> extends Father<T1,T2>{} 
   
   //部分继承
   calss Child<T1,A,B> extends Father<T1,String>{}
   
   //实现父类泛型 此时子类的泛型与父类无关
   class Child<A,B> extends Father<Integer,String>{}
   
   //忽略父类泛型 此时泛型默认为Object
   class Child extends Father{}
   ```

6.获取泛型类型

限制: 

1、反射无法操作局部变量 所以无法获取局部变量的类型

2、必须具有真实类型的存在

3、泛型的类型是明确的如(List<User>是明确的，List<T>是不明确的)

```java
		Map<String, Integer> map = new HashMap<>();	

		public static class D<T> {
    }
    
    public static class Demo<T> extends D<T> {
    }

    public static class SubDemo extends Demo<TTT> {
    }

    public static void main(String[] args) {
        //获取类的泛型类型
        SubDemo subDemo = new SubDemo();
        Type[] type = ((ParameterizedType) subDemo.getClass()
                .getGenericSuperclass()).getActualTypeArguments();
        for (Type t : type)
            System.out.println("subdemo==>" + t.getTypeName());

        //获取局部变量的泛型类型
      	//需要注意new对象时需要加大括号 用于创建子类 调用getGenericSuperclass才能获取当前类
        Demo<TTT> demo = new Demo<>(){};
        Type[] typeDemo = ((ParameterizedType) demo.getClass()
                .getGenericSuperclass()).getActualTypeArguments();
        for (Type t : typeDemo)
            System.out.println("demo==>" + t.getTypeName());
      
      	//获取属性泛型类型
      	Type t = Test.class.getDeclaredField("map").getGenericType(); //Filed的方法
        if (ParameterizedType.class.isAssignableFrom(t.getClass())) {
            for (Type t1 : ((ParameterizedType) t).getActualTypeArguments()) {
                System.out.print(t1 + ",");
            }
            System.out.println();
        }
    }
```



### 三、反射

https://www.jianshu.com/p/516228c2acfd
###### 1、反射原理
**1.1 获取属性、方法**
Class.getField/getMethod时 在底层会获取一个*软引用*缓存对象ReflectionData 内部有volatile修饰的成员变量、方法等信息 方法/属性为懒加载的方式且每个线程的Method对象是独立的。因为是缓存 多线程调用时 可以直接获取
    
**1.2 反射调用方法**
invoke方法在前15次调用使用native执行(初始化快但是性能略差)  超过之后jvm会生成字节码并加载到内存中执行(初始化慢但是性能好)  生成字节码并获取类加载器 使用unsafe.defineClass加载到内存中执行

### 四、注解
https://blog.csdn.net/with_dream/article/details/77888670
###### 1、标准注解
@Override :保证重写方法时的正确性
@Deprecated：对不应该使用的方法添加的标记
@SuppressWarnings:关闭不当的编译器警告。

###### 2、元注解
@Target 作用的级别 CONSTRUCTOR构造 METHOD方法 PACKAGE包
@Retention 注解级别 SOURCE源文件有效 RUNTIME运行时有效
@Documented 生成文档
@Inherited 允许子类继承父类的注解

### 五、jni
基础:https://blog.csdn.net/with_dream/article/category/7259627

```cpp
//A和V为传入参数的方式不同
void    (*CallVoidMethod)(JNIEnv*, jobject, jmethodID, ...);
void    (*CallVoidMethodV)(JNIEnv*, jobject, jmethodID, va_list);
void    (*CallVoidMethodA)(JNIEnv*, jobject, jmethodID, jvalue*);
```
###### 1、注册流程
https://blog.csdn.net/with_dream/article/details/78396590
System.loadLibrary会查找so文件的路径
最终通过JavaVMExt::LoadNativeLibrary实际加载so文件 并查找so文件中的JNI_OnLoad方法
如果找到JNI_OnLoad 则使用动态注册 否则使用静态注册
如果是动态注册 将使用registerNativeMethods注册本地方法

###### 2、内存泄漏
https://www.jianshu.com/p/ed891bf9ade3
native内存不足 会导致jvm崩溃并退出
2.1 native语言本身的内存泄漏问题 如malloc没有free
2.2 Global Reference 在创建后一直有效 需要手动删除
2.3 LocalReference会在函数结束时自动释放 但是有些情况需要手动处理
> jni函数中循环创建大量对象 需要手动释放

java程序切换到native时 会创建本地引用表 每当本地对象应用一个java对象时 就会在引用表中创建一个本地引用
调用DeleteLocalRef会释放某个引用 如果函数切换到java环境 则会删除引用表
本地引用不是局部变量

###### 3、检测、优化
使用工具 valgraind

###### 4、缓存
将native对象地址保存在java中
https://www.jianshu.com/p/0753c8ea06ec

native缓存FieldId MethodId
https://blog.csdn.net/m0_37312601/article/details/81637880

native调用java
https://www.jianshu.com/p/290fbbfac841

### 六、Lambda
https://blog.csdn.net/jiankunking/article/details/79825928

编译时 会将lambda表达式生成一个实现接口的静态内部类和一个静态方法
然后将内部类的实例传入调用方法即可

即 编译时 Lambda表达式生成额外的方法

### 七、jvm参数
https://baijiahao.baidu.com/s?id=1663410856124616488&wfr=spider&for=pc

jvm参数的调整只是锦上添花 主要的还是应用的性能

#### 1、跟踪参数
-verbos:gc / -XX:+printGC   跟踪gc
-XX:+printGCDetails			gc的详细信息
-Xloggc:文件名				  将gc信息存到文件中

### 、补充
##### 1、动态代理及原理
https://blog.csdn.net/with_dream/article/details/78438305

##### 2、异常

https://www.jianshu.com/p/d82fe75ee01d

异常类Throwable 分为
1. Error(运行时内部错误 无法处理)
2. Exception
2. 1. RuntimeException 代码异常 需要处理的
2. 2. IOException	非代码异常

##### 3、动态修改代码
###### 3.1、ASM
直接产生二进制class文件 可以在类被加载入Java虚拟机之前动态改变类行为 效率较高

###### 3.2、Javassist
编写容易 效率低于ASM 但是高于反射

###### 3.3、AndroidStudio APT
编译时 将注解生成java文件

##### 4、概念
###### 4.1、多态
运行时多态 重写
编译时多态 重载
屏蔽不同子类的差异 写出通用的代码 做到可维护 可扩展 可重用

##### 5、小补充
###### 1、hashCode与equals
基本类型的hashCode是其本身
需要保证同一个对象多次调用 返回的是同一个值

两个对象equals相同 hashCode一定相同
两个对象equals不同 hashCode不确定相不相同
两个对象hashCode相同 equals不一定相同
两个对象hashCode不同 equals一定不同

重写equals时 最好重写hashCode
因为如果对象放入集合 不会比较equals 而是hashCode

###### 2、三次握手 四次挥手 以及不可以两次握手
不可以两次握手：https://www.cnblogs.com/hoobey/p/6854020.html
客户端发送握手消息 若网络延迟 服务端很久之后才收到
此时客户端以为超时 放弃连接
但是服务端收到连接请求进行了连接 造成资源浪费

###### 3、Comparable/Comparator
Comparable为排序接口 内部比较器 如果实现则该类支持排序
Comparator为比较器 外部比较器 控制某个类的次序

###### 4、Integer
https://blog.csdn.net/u011541946/article/details/79978325
Integer内部有一个byte的缓存 即-128~127

需要特别注意的是
```java
Integer i1 = new Integer(97);   
Integer i2 = new Integer(97); 
System.out.println(i1 == i2);   //false  因为i1和i2都是new出来的
Integer i5 = 97;   
Integer i6 = 97;
System.out.println(i5 == i6);   //true  使用的是同一份缓存 地址相同
```
###### 4、属性作为参数与修改的影响 (重要)
https://blog.csdn.net/fenglllle/article/details/81389286

基本变量作为参数时 传的是值 方法内部修改与外部无关
引用变量作为参数时 传的是地址 地址指向的堆内存是相同的 方法内修改时 外部也会修改
引用变量作为参数时 如果在方法中重新为参数赋值 因为参数是副本 无法关联到外部变量 所以不会影响外部变量 如果想传出去 需要return

###### 5、对象的实例化顺序
1．父类静态成员和静态初始化块 ，按在代码中出现的顺序依次执行
2．子类静态成员和静态初始化块 ，按在代码中出现的顺序依次执行
3．父类实例成员和实例初始化块 ，按在代码中出现的顺序依次执行
4．父类构造方法
5．子类实例成员和实例初始化块 ，按在代码中出现的顺序依次执行
6．子类构造方法

顺序是：当创建类对象时，先初始化静态变量和静态块，然后是非静态变量和非静态代码块，然后是构造器。由于静态成员只会被初始化一次，所以如果静态成员已经被初始化过，将不会被再次初始化



6、双亲委派

加载一个类时 先在查看当前加载器缓存是否有，如果没有则委托给父类加载器去执行，直到最顶层的启动类加载器

如果启动类加载器缓存没有，则去从相应的库去加载，如果没有找到，则退回给子加载器去加载，直到最底层



目的:保证类的安全性和唯一性



##### 6、奇淫巧技

6.1、new对象时加{} 

自动创建匿名子类 并返回该子类的对象

可以对变量赋值操作、重写方法

```java
class A{}
//类似实现匿名接口
A a = new A() {}
```



参考: https://blog.csdn.net/wangzizhou7610/article/details/107845242?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-107845242-blog-65436773.pc_relevant_multi_platform_whitelistv4&spm=1001.2101.3001.4242.1&utm_relevant_index=3
