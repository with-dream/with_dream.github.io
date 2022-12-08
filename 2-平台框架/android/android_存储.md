### 一、SharedPreferences
###### 1、基本使用
获取：
```java
context.getSharedPreferences(key, value)
```
存入：
```java
SharedPreferences.Editor editor = sp.edit();
editor.putInt(key, value);
editor.commit();
editor.apply();
```

###### 2、源码
https://www.jianshu.com/p/f150ed3f7a81
SharedPreferences为接口 实现类SharedPreferencesImpl 第一次使用会创建新的xml文件 位置为/data/data/package_name/shared_prefs下

edit()方法返回EditorImpl

SharedPreferencesImpl中有HashMap[String, Object] 保存有文件中的数据
get/put操作都是操作mMap

commit()时 会对mMap深复制 然后文件的写操作 且为同步
apply()时 为异步操作

参考：https://www.jianshu.com/p/f150ed3f7a81

###### 3、跨进程
1 支持多进程 MODE_MULTI_PROCESS 但是已经废弃
会多次读取文件 耗时
commit提交完成 其他进程才能感知到

2 推荐使用ContentProvider

