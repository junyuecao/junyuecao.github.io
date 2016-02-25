---
layout: post
title: Gradle基本配置以某项目为例
description: "Gradle基本配置-以某项目为例, Gradle是随着Android Studio一起在Android领域的新工具之一, 它可以极大地减轻打包的配置工作,并且能够非常灵活地控制打包的过程. 抛弃古老的ant,投入Gradle的怀抱吧"
category: Android
tags: ["Android", "Gradle"]
---



## 0.为什么用gradle
> http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Why-Gradle-
 
Gradle is an advanced build system as well as an advanced build toolkit allowing to create custom build logic through plugins.

Here are some of its features that made us choose Gradle:

- Domain Specific Language (DSL) to describe and manipulate the build logic
    - 用DSL(领域特定语言)来表示和操作构建逻辑
- Build files are Groovy based and allow mixing of declarative elements through the DSL and using code to manipulate the DSL elements to provide custom logic.
    - 构建配置文件是基于Groovy的,  可以通过DSL和陈述性的元素混合, 通过代码来操作DSL元素来提供自定义逻辑
- Built-in dependency management through Maven and/or Ivy.
    - 通过Maven和/或Ivy提供内建依赖管理
- Very flexible. Allows using best practices but doesn’t force its own way of doing things.
    - 非常灵活, 允许使用一些最佳实践方式, 但是不强制使用   
- Plugins can expose their own DSL and their own API for build files to use.
    - 插件可以暴露他们自己的DSL和他们的API提供给构建文件使用
- Good Tooling API allowing IDE integration
    - 为IDE整合提供很好的API(Android Studio已经整合,貌似也有Eclipse插件)

http://tech.meituan.com/gradle-practice.html

Gradle的语言基于Groovy(半小时学会),Groovy兼容Java ,推荐这个教程: http://blog.csdn.net/zxhoo/article/details/29570685

 > Android Studio 原生支持gradle, 一个gradle项目导入到Android Studio时如果gradle版本适合,那么可以实现零配置导入项目.
 
## 1.目录结构

<img src="https://raw.githubusercontent.com/junyuecao/private-static/master/20141230a.png">

**与gradle有关的文件:**

#### `settings.gradle` (项目配置文件,配置项目路径,默认分隔符是冒号: )

-----

    include  ':HttpAPI',
            ':ImageCache', ':Loader',
            ':Pulltorefresh',
            ':ThreadPool',
            ':xwalk_core_library',
            ':FunGame'

-----

如果对语法有疑惑,可以看看参考资料里的语法学习,就可以看懂上面的include的语法了:`include([':HttpAPI', '', '', ....])`

#### `local.properties` (sdk 路径配置)

-----

    sdk.dir=/Users/junyuecao/Work/adt-bundle-mac-x86_64-20140702/sdk

-----

#### `build.gradle`

这是模块主要配置文件,根目录的`settings.gradle`中配置的每个模块目录下也都有这个文件

-----

    // Top-level build file where you can add configuration options common to all sub-projects/modules.
    buildscript { 
        //构建的脚本配置,基本上不需要动
        repositories {
            jcenter() //也可以是maven的仓库,Android Studio默认用这个仓库,没什么差别
        }
        dependencies {
            //gradle插件版本配置
            classpath 'com.android.tools.build:gradle:1.0.0'
        }
    }
    allprojects {
        repositories {
            //如果这里配置了jcenter,在各个模块下面就不再需要配置repositories了
            jcenter()
        }
    }

-----

### 每个模块下面的`build.gradle`

    buildscript {
        repositories {
            mavenCentral() //如果主项目里配置了allprojects,这里可不配
        }
        dependencies {
            //gradle插件
            classpath 'com.android.tools.build:gradle:1.0.0+'
        }
    }
    //应用android插件 等价于:apply({plugin:'com.android.library'});
    apply plugin: 'com.android.library'
    dependencies {
        //编译依赖包:现在很多开源库都支持这种方式引入,不需要下载jar包或者工程
        //导入之后项目中就可以直接用这个库里的方法了
        compile 'com.android.support:support-v4:20.0.+'
    }
    //Android主要配置的地方
    android {
        //编译的sdk版本,需要sdk中已经下载
        compileSdkVersion 19
        //buildTools版本,同样需要sdk中已经下载
        buildToolsVersion '19.1.0'
        //源代码设置
        //gradle默认的源代码目录如下面的图
        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java {
                    srcDirs = ['src']//srcDir-加入源码目录,srcDirs-直接指定目录
                }
                resources.srcDirs = ['src']//java资源,如app.properties文件
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
                res {
                    srcDirs = ['res']//资源文件
                }
                assets.srcDirs = ['assets']//静态文件
            }
        }
        lintOptions {
            abortOnError false //出错时不取消编译(gradle错误之类的会被忽略)
        }
    }

<img src="https://raw.githubusercontent.com/junyuecao/private-static/master/20141230a.png">

## 2. Android基本配置

以下2 - 5都是FunGame模块的`build.gradle`配置,因为这是我们的最主要模块


    android {
        defaultConfig {
            applicationId "com.fungame.mobile.client" //可以自定包名
            minSdkVersion 14 //这里的配置会覆盖AndroidManifest.xml文件的配置
            targetSdkVersion 19 //这里的配置会覆盖AndroidManifest.xml文件的配置
            versionCode 2 //这里的配置会覆盖AndroidManifest.xml文件的配置
            versionName "1.1" //这里的配置会覆盖AndroidManifest.xml文件的配置
        }
    }


## 3.签名配置和构建类型配置

签名只需要在我们的真正的app模块里面配置,gradle默认有两种构建类型,一个是debug,一个是release


    android {
        signingConfigs {
            myConfig {
                //需要在build.gradle同目录下的gradle.properties中配置以下信息
                //RELEASE_STORE_FILE=/path/to/key/
                //RELEASE_STORE_PASSWORD=***********
                //RELEASE_KEY_ALIAS=*****
                //RELEASE_KEY_PASSWORD=*******
                storeFile file(RELEASE_STORE_FILE)
                storePassword RELEASE_STORE_PASSWORD
                keyAlias RELEASE_KEY_ALIAS
                keyPassword RELEASE_KEY_PASSWORD
            }
        }
        
        buildTypes {
            debug {
                //配置在debug包中也签上名,好处是debug包换release包不会清数据
                signingConfig signingConfigs.myConfig 
            }
            release {
                //下面的minifyEnabled true可以执行proguard程序.但是fungame里用到了crosswalk的so库,会导致应用崩溃.暂时没有使用
                signingConfig signingConfigs.myConfig
                zipAlignEnabled true
                //minifyEnabled true
                //proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-project.txt'
            }
        }
    }


## 4. 渠道配置

渠道配置非常灵活,渠道少的时候可以通过为每个渠道指定规则,多的时候则可以动态生成,目前我们只有三个渠道,每个渠道两个平台,我们的渠道标识码是写在src目录下的`app.properties`中的, 因此每个渠道配置了单独的`resources`目录.
 
配置如下

    def armVersionCode = 3  //必须是奇数
    def x86VersionCode = armVersionCode + 1 //比arm多1,这样不用担心重复
    productFlavors {
        googleplay_x86 {
            flavorDimension "abi"
            ndk {
                abiFilter "x86"
            }
            versionCode x86VersionCode
        }
        googleplay_arm {
            flavorDimension "abi"
            ndk {
                abiFilter "armeabi-v7a"
            }
            versionCode armVersionCode
        }
        android_mobomarket_x86 {
            flavorDimension "abi"
            ndk {
                abiFilter "x86"
            }
            versionCode x86VersionCode
        }
        android_mobomarket_arm {
            flavorDimension "abi"
            ndk {
                abiFilter "armeabi-v7a"
            }
            versionCode armVersionCode
        }
        android_mobostore_x86 {
            flavorDimension "abi"
            ndk {
                abiFilter "x86"
            }
            versionCode x86VersionCode
        }
        android_mobostore_arm {
            flavorDimension "abi"
            ndk {
                abiFilter "armeabi-v7a"
            }
            versionCode armVersionCode
        }
    }
    sourceSets.googleplay_x86 {
        resources.srcDirs = ['resources_play']
    }
    sourceSets.googleplay_arm {
        resources.srcDirs = ['resources_play']
    }
    sourceSets.android_mobomarket_x86 {
        resources.srcDirs = ['resources_mobomarket']
    }
    sourceSets.android_mobomarket_arm {
        resources.srcDirs = ['resources_mobomarket']
    }
    sourceSets.android_mobostore_x86 {
        resources.srcDirs = ['resources_mobostore']
    }
    sourceSets.android_mobostore_arm {
        resources.srcDirs = ['resources_mobostore']
    }

## 5.完整的FunGame下的`build.gradle`

    buildscript {
        repositories {
            jcenter()
            maven {
                url 'https://maven.fabric.io/public' // twitter的fabric仓库
            }
        }
        dependencies {
            classpath 'com.android.tools.build:gradle:1.0.0'
            //我们用到了twitter库,这是官方提供的插件
            classpath 'io.fabric.tools:gradle:1.14.4' 
        }
    }
    apply plugin: 'com.android.application'
    apply plugin: 'io.fabric'
    repositories {
        jcenter()
        maven { url 'https://maven.fabric.io/public' }
    }
    configurations {
    //    all*.exclude group: 'com.android.support', module: 'support-v4'
    }
    dependencies {
        compile fileTree(dir: 'libs', include: '*.jar')
        compile 'com.android.support:support-v4:21.0.2'
    //    compile project(':FacebookSDK')
        compile 'com.facebook.android:facebook-android-sdk:3.20.0'
        compile project(':HttpAPI')
        compile project(':ImageCache')
        compile project(':Loader')
        compile project(':Pulltorefresh')
        compile project(':ThreadPool')
        compile project(':xwalk_core_library')
        compile('com.twitter.sdk.android:twitter:1.0.0@aar') {
            transitive = true;
        }
    }
    def armVersionCode = 3
    def x86VersionCode = 4
    android {
        compileSdkVersion 19
        buildToolsVersion '19.1.0'
        defaultConfig {
            applicationId "com.fungame.mobile.client"
            minSdkVersion 14
            targetSdkVersion 19
            versionCode 2
            versionName "1.1"
        }
        signingConfigs {
            myConfig {
    //            需要在build.gradle同目录下的gradle.properties中配置以下信息
    //            RELEASE_STORE_FILE=/path/to/key/
    //            RELEASE_STORE_PASSWORD=***********
    //            RELEASE_KEY_ALIAS=*****
    //            RELEASE_KEY_PASSWORD=*******
                storeFile file(RELEASE_STORE_FILE)
                storePassword RELEASE_STORE_PASSWORD
                keyAlias RELEASE_KEY_ALIAS
                keyPassword RELEASE_KEY_PASSWORD
            }
        }
        buildTypes {
            debug {
                signingConfig signingConfigs.myConfig
            }
            release {
                signingConfig signingConfigs.myConfig
                zipAlignEnabled true
    //            minifyEnabled true
    //            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-project.txt'
            }
        }
        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java.srcDirs = ['src']
                resources.srcDirs = ['src']
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
                res.srcDirs = ['res']
                assets.srcDirs = ['assets']
            }
        }
        flavorDimensions "abi"
        productFlavors {
            googleplay_x86 {
                flavorDimension "abi"
                ndk {
                    abiFilter "x86"
                }
                versionCode x86VersionCode
            }
            googleplay_arm {
                flavorDimension "abi"
                ndk {
                    abiFilter "armeabi-v7a"
                }
                versionCode armVersionCode
            }
            android_mobomarket_x86 {
                flavorDimension "abi"
                ndk {
                    abiFilter "x86"
                }
                versionCode x86VersionCode
            }
            android_mobomarket_arm {
                flavorDimension "abi"
                ndk {
                    abiFilter "armeabi-v7a"
                }
                versionCode armVersionCode
            }
            android_mobostore_x86 {
                flavorDimension "abi"
                ndk {
                    abiFilter "x86"
                }
                versionCode x86VersionCode
            }
            android_mobostore_arm {
                flavorDimension "abi"
                ndk {
                    abiFilter "armeabi-v7a"
                }
                versionCode armVersionCode
            }
        }
        sourceSets.googleplay_x86 {
            resources.srcDirs = ['resources_play']
        }
        sourceSets.googleplay_arm {
            resources.srcDirs = ['resources_play']
        }
        sourceSets.android_mobomarket_x86 {
            resources.srcDirs = ['resources_mobomarket']
        }
        sourceSets.android_mobomarket_arm {
            resources.srcDirs = ['resources_mobomarket']
        }
        sourceSets.android_mobostore_x86 {
            resources.srcDirs = ['resources_mobostore']
        }
        sourceSets.android_mobostore_arm {
            resources.srcDirs = ['resources_mobostore']
        }
        lintOptions {
            abortOnError false
        }
    }

## 6. 迁移步骤

### 6.0 安装Gradle: 请Google

### 6.1 根目录配置

在根目录下执行`gradle init wrapper` (这里的根目录不是指app模块,最好是包含所有依赖工程的一个目录)

这样，就会自动生成相关的gradlew，build.gradle，settings.gradle等文件和相关目录，并会自动下载对应版本的gradle binary包（所以以后不需要安装）。Gradle会自动识别Maven里的配置，并相应的导入进来，有少量部分配置可能需要修改。

 > 注：在已有的gradle项目里，尽量使用生成的gradlew这个wrapper，因为它会自动下载对应版本的Gradle，也就是说团队合作的其他人开发机上是不需要手动安装Gradle的，并且wrapper也让大家的Gradle版本一致，避免问题。

### 6.2 gradle脚本修改

要做的就是告诉gradle，要多项目构建，编辑settings.gradle，增加项目配置：

    include ":submodule1", ":submodule2"

根据项目目录分布,配置`build.gradle`,这里针对每个模块非常灵活.

### 6.3 如果子项目的目录结构不符合约定,则需要配置模块的build.gradle

参照上面的代码示例

### 6.4 配置buildType和productFlavor(渠道)

参照上面的代码示例

### 6.5 常用命令

构建

`gradle build`或者`./gradlew.bat build`(用wrapper的话,以下省略)

清理输出目录

`gradle clean`

打包release的apk

`gradle aR` 或者 `gradle assembleRelease`

## 7. 其他

### 7.1. gradle 项目依赖.SO文件时应该怎么配置

**gradle的Android插新版已经支持自定义jni库的位置了:**
 
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }

**以下是约定的路径,如果按照这个路径,就不需要配上面的位置了**

"gradle 1.11 com.android.tools.build:gradle:0.9.+" 以上支持预编译的ndk文件,只需要吧*.so文件放入src/main/jniLibs 目录中,当gradle编译时,会自动把这些ndk文件打包到正确的位置.

文件目录如下

    Project:
    |--src
    |--|--main
    |--|--|--java
    |--|--|--jniLibs
    |--|--|--|--armeabi
    |--|--|--|--|--.so files
    |--libs
    |--|--other.jar

## 8. 常见问题解决

### 8.1 gradle版本不对

gradle插件目前版本为1.0.0, 要求gradle版本2.0, 如果gradle版本过低,会提示错误

建议新项目直接上最新稳定版.

建议使用gradle wrapper

### 8.2 proguard使用

proguard可以压缩混合生成的类, 减少体积, 在FunGame中使用到了crosswalk内核,他提供了.so库,使用proguard会导致生成的包崩溃,目前还没有找到解决方法,暂时未使用proguard

### 8.3 lint出错

在模块的build.gradle中加入:

    android {
       lintOptions {
            abortOnError false
        }
    }

## 9. 参考资料

- 看懂gradle语法: http://blog.csdn.net/zxhoo/article/details/29570685
- 官方使用文档: http://tools.android.com/tech-docs/new-build-system/user-guide 鸟文版,即便是官方文档也稍微有一点点过时
- 官方使用文档: http://avatarqing.gitbooks.io/gradlepluginuserguidechineseverision/content/ 翻译版
- 美团的一些使用心得:
    - http://tech.meituan.com/gradle-practice.html
    - http://tech.meituan.com/mt-apk-packaging.html
    - http://tech.meituan.com/mt-apk-adaptation.html
        