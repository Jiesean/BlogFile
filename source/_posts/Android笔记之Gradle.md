---
title: Android笔记之Gradle
date: 2017-04-17 22:34:13
tags: [Android]
---


## 前言
Gradle是Android编译应用资源和源代码，然后将它们打包成 APK的一个自动化构建工具，他能够自动化管理资源和执行构建，类似于Linux中的make工具。

Android Studio就是基于Gradle来完成APK的构建的。当然我们也可以单独地使用Gradle来构建我们的Android应用工程。当然我们最常用的还是Android Studio下对他的使用，因此我们主要对Android Studio中的Gradle的使用作出说明。

##### Gradle
Gradle作为一种高级的构建工具，可以帮助开发者实现自定义的构建过程，使用Gradle可以实现的功能包括;
构建类型、产品风味、构建变体、清单条目、依赖项、签署、ProGuard、APK 拆分等

<!-- more -->

##### Android Studio中的Gradle文件
在Android的视图下，我们可以看到Gradle Scripts目录下的文件都是Gradle文件，他是基于Java 虚拟机 (JVM) 的动态语言 [Groovy](http://groovy.codehaus.org/)来编写的。

## 构建文件
##### 1. setting.gradle
```
include ‘:app’
```
该文件的内容十分简单，就是包含构建APK时所包含的module即可，默认该工程下建立了多少module都会被包含进去。
##### 2. build.gradle(Project)
```
/**
 * buildscript中包含的是Gradle工具的仓库和依赖
 */
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.1'
    }
}

/**
 * allprojects 标签内包含的是整个工程，也就是所有的module的 repositories and
 * dependencies，例如第三方插件或者库。
 */
allprojects {
   repositories {
       jcenter()
   }
}
```
##### 3.  build.gradle(Module)
```
/**
 * 应用插件
 */
apply plugin: 'com.android.application'

/**
 * 所有Android特性相关的注册
 */
android {
  /**
   *  compileSdkVersion  Gradle用来编译App的Android Api版本
   *
   *  buildToolsVersion  Gradle使用的build工具的版本
   */
  compileSdkVersion 25
  buildToolsVersion "25.0.2"

  /**
   * defaultConfig  
   */

  defaultConfig {

    /**
     * applicationId 唯一标识
     */
    applicationId 'com.example.myapp'

    // 运行App的最小Api
    minSdkVersion 15

    // 制定测试App的Api
    targetSdkVersion 25

    // 版本号
    versionCode 1

    // 用户友好的版本名称
    versionName "1.0"
  }

  /**
   * 构建类型，包括debug版本和release版本
   */

  buildTypes {
    /**
     * By default, Android Studio configures the release build type to enable code
     * shrinking, using minifyEnabled, and specifies the Proguard settings file.
     */
    release {
        minifyEnabled true // Enables code shrinking for the release build type.
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }

  /**
   * 产品风味代表您可以发布给用户的不同应用版本，例如免费和付费的应用版本
   */
  productFlavors {
    free {
      applicationId 'com.example.myapp.free'
    }
    paid {
      applicationId 'com.example.myapp.paid'
    }
  }

  /**
   * 屏幕适配相关的
   */

  splits {
    // Screen density split settings
    density {

      // Enable or disable the density split mechanism
      enable false

      // Exclude these densities from splits
      exclude "ldpi", "tvdpi", "xxxhdpi", "400dpi", "560dpi"
    }
  }
}

/**
 *  包含这个module所有的依赖项
 */
dependencies {
    compile project(":lib")
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
##### 4. gradle.properties
Gradle相关的配置，例如内存大小等

##### 5. local.properties
为构建系统配置本地环境属性，例如 SDK 安装路径。


