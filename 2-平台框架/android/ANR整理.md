### 一、导出日志
##### 1、查看日志详情
```cpp
adb shell
cd data/anr
ls -a      //列出所有文件

```

##### 2、导出
adb pull data/anr/anr_XXX

如果出现`remote open failed: Permission denied`
则使用adb bugreport
最后使用`adb pull <手机目录/文件>  <电脑目录>`

一般放在压缩文件中

https://blog.csdn.net/denglusha737/article/details/86706909

### 二、日志分析