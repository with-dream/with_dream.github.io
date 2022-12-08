
[TOC]

#### 待整理
https://www.jianshu.com/p/e5de9538effa

### 一、概述

###### 1、整体结构

```groovy
/*
apply plugin表明当前module是一个app
如果是library则写成 apply plugin: 'com.android.library'；
* */
apply plugin: 'com.android.application'
//android {...} 配置了android相关的构建选项：
android {
    compileSdkVersion 26//编译的sdk版本
    buildToolsVersion "26.0.2"//构建工具版本
    defaultConfig {//在构建时会覆盖AndroidManifest
        applicationId "com.test"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {//buildTypes 配置了如何构建app，默认有debug和release两种
        release {
        	zipAlignEnabled true //自动压缩
        	shrinkResources true //去除无用的资源文件
            minifyEnabled false//是否进行混淆
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'//混淆文件的位置
        }
    }

	//指定编译版本
	compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    /*
    * java插件引入了一个概念叫做SourceSets，通过修改SourceSets中的属性，
    * 可以指定哪些源文件（或文件夹下的源文件）要被编译，
    * 哪些源文件要被排除。
    * */
    sourceSets {
        main {
            if (Boolean.valueOf(rootProject.ext.isModule_North)) {//apk
                manifest.srcFile 'src/main/manifest/AndroidManifest.xml'
            } else {
                manifest.srcFile 'src/main/AndroidManifest.xml'
                java {
                    //library模式下，排除java/debug文件夹下的所有文件
                    exclude '*module'
                }
            }
        }
    }
}
/*
dependencies 表明了当前module依赖关系,一个module可以用三种方式依赖。
* */
dependencies {
	//方式1.依赖统一工程下的另一个module
    //compile project(":lib")
    //方式2.依赖libs目录下的jar文件
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    //方式3.依赖远程库
    compile 'com.android.support:appcompat-v7:26.0.0-alpha1'
    testCompile 'junit:junit:4.12'
}
```

###### 2、依赖关键字

| 关键字 | 功能 |
|--------|--------|
|    api    |    依赖项目 但所依赖的项目外部可以访问    |
|    implementation    |    依赖项目 但所依赖的项目外部无法访问    |
|    provided    |    只编译 但不写入apk    |
|    compile    |    已经废弃 同api    |

annotationProcessor  注解处理器
Android plugin 3.0.0之前使用compile会自动添加到注解处理器 现在需要手动添加

buildTypes 打包配置
一般会有debug、release  可以指定不同的签名方式、混淆逻辑等

signingConfigs 签名配置
变量可以在项目根目录的 gradle.properties 文件定义，gradle 脚本直接按变量名使用即可

productFlavors 签名配置
在flavor里指定signingConfig 就可以指定release包的签名配置
可以生成渠道包

buildTypes和productFlavors支持
```groovy
pplicationVariants.all {
            variant ->
                variant.outputs.all {
                    outputFileName = "DEMO_${variant.name}_${variant.versionName}.apk"
                }
        }
```

参考: https://www.jianshu.com/p/52493f919e68

###### 3、BuildConfig类与特定全局变量

```groovy
defaultConfig {
    buildConfigField "String", "TOKEN", '"XXX"'
}
buildTypes {
    debug {
      buildConfigField "String", "COMMON_URL", '"http://api.dev.com/"'
      buildConfigField "boolean", "FLAG", "true"
    }
    release {
      buildConfigField "String", "COMMON_URL", '"http://api.prod.com/"'
      buildConfigField "boolean", "FLAG", "false"
    }
}
```

在build.gradle中定义buildConfigField关键字 会自动生成BuildConfig类
在项目中可以直接使用`BuildConfig.TOKEN`

###### 4、自定义task
```groovy
task autoSign {
  //依赖于其他Task执行完，在执行本Task
      dependsOn ''
      //先执行
      doFirst{
      }
      // 后执行
      doLast{
      }
}
```
https://blog.csdn.net/Utzw0p0985/article/details/87942434

###### 5、全局设置
```groovy
allprojects {
  repositories {
    jcenter()
  }
  tasks.withType(JavaCompile) {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
  }
}
```

###### 6、定义全局变量

###### 7、生成自定义apk名称
```groovy
//生成自定义App名称
def generateAppName(variant) {
    variant.outputs.each { output ->
        def file = output.outputFile
        output.outputFile = new File(file.parent, "Gank.IO-V" +           android.defaultConfig.versionName + ".apk")
    }
}
```

###### 8、多渠道打包


### 补充
###### 1、查看gradle依赖树
https://blog.csdn.net/ouyang_peng/article/details/82590820

###### 2、gradle中可以直接引用gradle.properties文件中的常量