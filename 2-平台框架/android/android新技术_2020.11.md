
[TOC]


# android新技术汇总
### 一、LifeCycle
https://www.cnblogs.com/best-hym/p/12235446.html

ComponentActivity实现LifecycleOwner接口
LifeCycle主要是用于管理Activity生命周期的回调

```java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        getLifecycle().addObserver(new LifecycleEventObserver() {
            @Override
            public void onStateChanged(LifecycleOwner source,  Lifecycle.Event event) {
            	//source为Activity
                //event为具体的生命周期事件
                Log.e("=====", "event==>" + event.toString());
            }
        });
    }
```


### 二、LiveData
类似Eventbus 但是有生命周期的限制(页面可见时有回调) 内部使用的是LifeCycle
回调总是在主线程

https://www.jianshu.com/p/b04abbb766e0?utm_campaign=haruki

```java
//继承ViewModel类
public class TestViewModel extends ViewModel {
    private MutableLiveData<String> mNameEvent = new MutableLiveData<>();
    public MutableLiveData<String> getNameEvent() {
        return mNameEvent;
    }
}
```
```java
//创建TestViewModel 并绑定LifecycleOwner
//绑定有固定的流程
TestViewModel mTestViewModel = ViewModelProviders.of(this).get(TestViewModel.class);
MutableLiveData<String> nameEvent = mTestViewModel.getNameEvent();
nameEvent.observe(this, new Observer<String>() {
    @Override
    public void onChanged(@Nullable String s) {
        //回调处理
    }
});

//设置值 会回调到onChanged
mTestViewModel.getNameEvent().setValue("");
```

### 三、ViewModel
https://www.jianshu.com/p/2e511a9e46ec

```java
TestViewModel model = ViewModelProviders.of(this).get(TestViewModel.class)
```
内部由key/value方式保存通过反射创建的TestViewModel实例
相当于单例 且线程不安全
在Lifecycle的onDestroy状态销毁(如果因配置状态[横竖屏切换] 不会销毁)

### 四、DataBindding
https://developer.android.google.cn/topic/libraries/data-binding

```gradle
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

1、xml中可以使用简单的表达式
2、可以包含集合、资源、事件处理函数
3、可以在布局中使用`<include layout="@layout/name"
               bind:user="@{user}"/>`
4、可以导入类 再使用时可以不加全路径直接引用


```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
	//数据
    <data>

        <variable
            name="model"
            type="cn.sharesdk.test.Model" />
    </data>
	//布局
    <RelativeLayout xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            android:id="@+id/text"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:padding="10dp"
            android:text="@{model.name}" />

    </RelativeLayout>
</layout>
```

1、通过binding.setModel(user)同步xml数据
2、数据可以通过可观察数据(基本类型、集合、对象 ObservableXXX) 自动同步xml数据

```java
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //绑定布局
        final ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
        //初始化模型类
        final Model user = new Model();

        mTvName = findViewById(R.id.text);
        mTvName.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
            	//为模型变量赋值
                user.name = count++ + "--";
                //更新view的数据
                binding.setModel(user);
            }
        });
    }
```
