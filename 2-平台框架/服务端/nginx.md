### 一、基础
#### 1、概述
- 反向代理
1、正向代理 即客户和服务间加一个中转服务器
2、反向代理 在服务器前加一个中转服务器

正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端.
反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端

- 负载均衡
反向代理服务器 将大量请求 平均分发给多台服务器 减少单个服务器的压力

- 动静分离
将静态资源(html等)放在单独的服务器  动态资源(servlet等)放在另外的服务器
减少单个服务器的压力

#### 2、安装
brew install nginx
安装完成 需要关闭linux的防火墙

#### 3、常用命令
```shell
ps -ef | grep nginx
nginx -v
nginx  //启动
nginx -s stop
nginx -s reload
```

#### 4、配置文件

使用nginx -V(大写的V) 可以查看详细信息 包括`--conf-path`

##### 4.1、全局块
##### 4.2、events块
##### 4.3、http块

##### 4.4、日志配置

#### 5、语法规则
https://www.jianshu.com/p/df1a9a68b72c
https://blog.51cto.com/5001660/2130506


#### 6、配合memcache  以及一致性hash
memcache集群时 同一个ip连接到固定的memcache 需要用到一致性hash

参考:
https://www.bilibili.com/video/BV1zJ411w7SV?p=8