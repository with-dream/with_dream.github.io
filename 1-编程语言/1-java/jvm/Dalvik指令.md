
[TOC]

### 一、
##### 1、文件格式
```smali
.class <访问权限> [修饰关键字] <类名>	//指令指定了当前类的类名
.super <父类名>	//指令指定了当前类的父类
.source [源文件名]	//指令指定了当前类的源文件名

//成员属性
.field <访问权限> [static] [修饰关键字] <字段名>:<字段类型>

//直接方法
.method <访问权限> [修饰关键字] <方法原型>
    <.locals>	//指定了使用的局部变量的个数
 [.parameter]	//指定了方法的参数
 [.prologue]	//指定了代码的开始处，混淆过的代码可能去掉了该指令
 [.line]		//指定了该处指令在源代码中的行号
<代码体>
.end method

//接口
.implements <接口名>        //接口关键字

//注解 可以作用于类、方法、属性
.annotation [ 注解属性] <注解类名>
    [ 注解字段 =  值]
.end annotation
```





参考:
https://blog.csdn.net/qq_32113133/article/details/85163277
https://blog.csdn.net/qq_32113133/article/details/85246142