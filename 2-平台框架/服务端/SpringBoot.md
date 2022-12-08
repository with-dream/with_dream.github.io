#### yaml
springboot使用yaml进行配置

可以定义对象、数组、key-value等
也可以将yaml的值映射到java中  使用`@ConfigurationProperties(prefix = "属性")

#### 配置文件  application.yml
boot会自动加载`@ConfigurationProperties`注解的XXXProperties类中的属性
在application.yml中可以修改这些配置