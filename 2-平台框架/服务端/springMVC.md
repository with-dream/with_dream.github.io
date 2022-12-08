### 一、基础

#### 1、概述

#### 2、环境搭建
https://segmentfault.com/a/1190000017248622

**报错**

通配符的匹配很全面, 但无法找到元素 'context:component-scan' 的声明

补全dispatcher-servlet.xml的xsi:schemaLocation
```xml
xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/aop
                            http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
                            http://www.springframework.org/schema/context
                            http://www.springframework.org/schema/context/spring-context.xsd"
```

#### 3、注解关键字
##### 3.1 @Controller
用于配置控制器

##### 3.2 @RequestMapping
用于url映射

#### 4、Restful风格
使用`@RequestMapping`注解添加url路径 可以指定请求方式

另外还有:
@GetMapping
@PostMapping
