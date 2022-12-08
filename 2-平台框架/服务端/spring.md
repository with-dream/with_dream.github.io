##### 官网

https://spring.io

https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/spring-framework-reference/core.html#beans-basics

### 一、bean

**导maven包:**
https://mvnrepository.com 搜索spring  导入Spring Web MVC

```gradle
<dependencies>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.5</version>
    </dependency>
</dependencies>
```

##### 1、bean配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

- 注入带参数的构造方法`<constructor-arg>`

- 可注入的类型:
bean | ref | idref | list | set | map | props | value | null
props专用于`java.util.Properties` 继承自Hashtable

- 自动装配Bean
使用ref引用别的类时 可以使用autowire自动引用 不用显示配置

##### 2、xml命名空间
- p命名空间

```xml
导入
xmlns:p="http://www.springframework.org/schema/p"

<bean id="hello" class="com.example.HelloModel" p:Hello类的属性="值">
```

- c命名空间 用于有参构造

```xml

xmlns:c="http://www.springframework.org/schema/c"

<bean id="hello" class="com.example.HelloModel" c:属性="值">
```

##### 3、bean的作用域
单例模式   (默认)
原型模式   每次获取 都会创建一个新对象
request		一次请求 创建一次 结束释放
session
application
websocket


#####  4、注解的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

bean标签有name属性

@Autowired(request=[true|false])  默认通过byName
@Qualifier	指定唯一的bean对象

@Resource	通过byName 如果找不到 通过byType
@Nullable

#### 5、扫描注解
由框架自动扫描需要注解的类 只需要配置包名

```xml

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
	//自动扫描
	<context:component-scan base-package="包名"/>
    <context:annotation-config/>

</beans>
```

@Component
@Value		属性的默认值
@Repository		DAO
@Service		Service
@Controller		controller

#### 6、纯java注解
@Configuration

### 二、AOP
| 方式 | 说明 |
|--------|--------|
|    前置通知    |    目标方法执行前添加方法    |
|    后置通知    |    目标方法执行后添加方法    |
|    环绕通知    |    目标方法执行前、后添加方法    |
|    异常通知    |    目标方法抛异常添加方法    |
|    引介通知    |    目标方法添加新方法、属性添加方法    |

**导maven包:**
```xml
<dependencies>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.3</version>
    </dependency>
</dependencies>
```
bean配置
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       //aop配置
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        //aop配置
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

</beans>
```

#### 1、bean配置
1. 编写继承自Advice的通知
2. 配置xml

`execution(返回类型 包名.方法(参数))`

```xml
<bean id="user" class="com.example.UserImpl"/>
<bean id="logBefore" class="com.example.test1.LogBefore"/>

<aop:config>
	//切入点
    <aop:pointcut id="pointcut" expression="execution(* com.example.UserImpl.*(..))"/>
    //添加通知
    <aop:advisor advice-ref="logBefore" pointcut-ref="pointcut"/>
</aop:config>
```

#### 2、自定义通知
1. 编写普通类 用于通知
2. 配置aop:aspect  将编写的方法与切入点连接

```xml
<bean id="advice" class="com.example.test2.MyAdvice"/>
<aop:config>
    <aop:aspect ref="advice">
        <aop:pointcut id="pointcut" expression="execution(* com.example.UserImpl.*(..))"/>
        <aop:before method="before" pointcut-ref="pointcut"/>
    </aop:aspect>
</aop:config>
```

#### 3、使用注解
1. 编写注解类
```java
@Aspect
public class AnnotationPointCut {
    @Before("execution(* com.example.UserImpl.*(..))")
    public void before() {
        System.out.println("==>AnnotationPointCut  before");
    }
}
```

2. 配置xml
```xml
<bean id="ann" class="com.example.test3.AnnotationPointCut"/>
//配置自动扫描
<aop:aspectj-autoproxy/>
```