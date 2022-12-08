
[TOC]



#### 一、配置项目

官网:https://mybatis.org/mybatis-3/getting-started.html

```shell
mysql.server start
mysql.server stop
```

##### 1、maven包
```xml
<dependencies>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.4</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.19</version>
    </dependency>
</dependencies>

```

##### 2、pom.xml配置  如果还是出问题 重启idea
```xml
<build>
    <resources>
        <!-- mapper.xml文件在java目录下 -->
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

##### 3、创建mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/example/dao/UserMapper.xml"/>
    </mappers>
</configuration>
```

##### 4、编写dao
```java
public interface UserMapper {
    List<User> getUser();
}
```

##### 5、编写映射xml   映射的类名 需要使用全路径
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.dao.UserMapper">
    <!--  namespace:接口类全路径  id对应相应的方法   resultType:返回类的全路径 -->
    <select id="getUser" resultType="com.example.pojo.User">
        select * from mybatis.user
    </select>
</mapper>
```

##### 6、查询
```java
String resource = "mybatis-config.xml";
try {
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory =
            new SqlSessionFactoryBuilder().build(inputStream);

	//如果是增删改 则需要使用session.commit()提交事务
    SqlSession session = sqlSessionFactory.openSession();
    UserMapper userMapper = session.getMapper(UserMapper.class);
    List<com.example.pojo.User> users = userMapper.getUser();

    for (com.example.pojo.User user : users)
        System.out.println("user==>" + user.toString());

    session.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

##### 7、万能Map

增删改 如果参数过多时 可以传map 不用传递所有的参数

#### 二、mybatis-config.xml配置
##### 1、environments
environments可以配置多个环境  使用default指定具体的配置

```xml
//指定使用的配置
<environments default="development">
	//开发环境
    <environment id="development">
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        </dataSource>
    </environment>
    //测试环境
    <environment id="test">
        <dataSource type="POOLED">
            <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        </dataSource>
    </environment>
</environments>
```

##### 2、properties
可以将配置放入外部properties键值对中 方便配置
mybatis-config.xml中 每种配置都有既定的顺序 乱序会出错
```xml
<properties resource="db/config.properties">
  <!-- ... -->
  <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
</properties>
```

##### 3、别名  typeAliases
使用别名代替类的全路径 方便使用 主要用在编写的映射xml中

3.1 使用xml配置
```xml
<typeAliases>
    <typeAlias type="com.example.pojo.User" alias="User"/>
</typeAliases>
```

3.2 使用注解
```xml
<typeAliases>
    <package name="com.example.pojo"/>
</typeAliases>
```
com.example.pojo包名下的类 添加`@Alias()`注解

##### 4、设置  setting

##### 5、结果集映射  resultMap
可以将数据库中的字段关联到模型类的字段 防止不统一

`association`	多对一
`collection`	一对多

#### 三、日志配置
```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

#### 四、使用注解

resultMap对应的注解为@Param()

```java
public interface UserMapper2 {
	//sql语句
    @Select("select * from mybatis.user")
    List<User> getUser();
}
```

配置mapper
```xml
<mappers>
    <mapper class="example.dao.UserMapper2"/>
</mappers>
```

#### 五、动态sql
mapper.xml配置sql时 根据不同条件 生成不同的sql
```xml
select * from user where 1=1
<if test="name != null">
	and name = #{name}
</if>
```

#### 六、缓存
##### 1、一级缓存
SqlSession即是一级缓存 关闭之前有效

##### 2、二级缓存

`cacheEnable = true`

Mapper.xml中添加`<cache/>`标签

##### 3、自定义缓存  ehcache
