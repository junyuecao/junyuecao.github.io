---
layout: post
title: Mac下为Android编译FFMPEG和x264（二）
description: "如何在Android Studio里导入上一篇文章中编译好的ffmpeg"
category: Android
tags: ["ffmpeg", "Android", "Mac"]
---


#### 前言

[前一篇文章](/android/2016/05/25/build-ffmpeg-for-android-with-x264/)我们编译好了一个so文件（或者一个个独立的so，为了方便，我合并成一个so文件）。那么如何在Android Studio（后面简称AS）中导入ffmpeg以及使用呢？

#### 1. AS的jni支持

AS从1.3开始支持jni，可以在src/main/jni目录下新建C或者C++的文件，AS会自动编译并打包至apk中，但是as目前对于更加复杂的C模块支持还不是很好，在gradle的实验版本里正在加入完善的jni支持，可以看这里[http://tools.android.com/tech-docs/new-build-system/gradle-experimental](http://tools.android.com/tech-docs/new-build-system/gradle-experimental)。我暂时禁用了AS的jni功能，自行通过ndk-build生成native模块。然后在AS中调用。

#### 2. jni目录配置

首先我们在项目的app/main目录下新建一个jni目录，并将编译好的libffmpeg.so 和include目录拷贝到jni目录下, 然后新建一个`Android.mk`文件。

{% highlight shell%}
.
├── Android.mk
├── include
│   ├── libavcodec
│   ├── libavdevice
│   ├── libavfilter
│   ├── libavformat
│   ├── libavutil
│   ├── libpostproc
│   ├── libswresample
│   └── libswscale
└── libffmpeg.so
{% endhighlight %}

`Android.mk`文件内容如下：

{% highlight shell%}
LOCAL_PATH := $(call my-dir)

# FFmpeg库
include $(CLEAR_VARS)
LOCAL_MODULE := ffmpeg
LOCAL_SRC_FILES := libffmpeg.so
include $(PREBUILT_SHARED_LIBRARY)

# Program
include $(CLEAR_VARS)
LOCAL_MODULE := hello-android-jni
LOCAL_SRC_FILES := hello-android-jni.c
LOCAL_C_INCLUDES += $(LOCAL_PATH)/include
LOCAL_LDLIBS := -llog -lz
LOCAL_SHARED_LIBRARIES := ffmpeg
include $(BUILD_SHARED_LIBRARY)
{% endhighlight %}

上面的代码里的`LOCAL_MODULE := hello-android-jni`是我们编译后的ndk模块名，`LOCAL_SRC_FILES := hello-android-jni.c`是模块源文件。我们在jni目录下新建`hello-android-jni.c`，填入以下内容：


{% highlight c%}
#include <jni.h>
#include "include/libavcodec/avcodec.h"

JNIEXPORT jstring JNICALL
Java_me_zheteng_android_ffmpeg_MainActivity_getMsgFromJni(JNIEnv *env, jobject instance) {

    char info[10000] = {0};
    sprintf(info, "%s\n", avcodec_configuration());
    return (*env)->NewStringUTF(env, info);
}

{% endhighlight %}

在jni目录下运行`ndk-build`来编译（ndk-build在/path/to/ndk/ndk-build下），我们会发现自动生成了和jni同级的目录libs和obj：

{% highlight shell%}
├── jni
├── libs
│   └── armeabi
│       ├── libffmpeg.so
│       └── libhello-android-jni.so
└── obj
    └── local
        └── armeabi
{% endhighlight %}

#### gradle配置

由于gradle默认的jniLibs并没有这个路径下的目录，并且我们要防止gradle默认的jni行为，所以我们修改gradle文件，加入以下内容：

{% highlight shell%}
sourceSets {
    main {
        jniLibs.srcDirs = ['src/main/libs'] 
        jni.srcDirs = []
    }
}
{% endhighlight %}

完整的build.gradle看起来应该是这样的（可能略有差别

{% highlight shell%}
apply plugin: 'com.android.application'
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"

    defaultConfig {
        applicationId "me.zheteng.android.ffmpeg"
        minSdkVersion 15
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles.add(file('proguard-android.txt'))
        }
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['src/main/libs']
            jni.srcDirs = []
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
}
{% endhighlight %}


#### 在Java类里调用jni

要在Java程序里调用native的函数，我们必须加载so库，加载方式很简单，

{% highlight java%}
static {
    System.loadLibrary("hello-android-jni");
    System.loadLibrary("ffmpeg");
}
{% endhighlight %}

回到我们刚才创建的C程序里的函数名`Java_me_zheteng_android_ffmpeg_MainActivity_getMsgFromJni`，jni命名规则是这样的`Java_包名(点变下划线)_类名_函数名` ，所以我们的类名是`me.zheteng.android.ffmepg.MainActivity`，在MainActivity中声明这个函数：

{% highlight java%}
public native String getMsgFromJni();
{% endhighlight %}

调用这个函数就能获取到我们编译ffmpeg时所用的参数。

完整的Activity代码如下：

{% highlight java%}
public class MainActivity extends AppCompatActivity {

    static {
        System.loadLibrary("hello-android-jni");
        System.loadLibrary("ffmpeg");
    }

    public native String getMsgFromJni();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ((TextView) findViewById(R.id.text)).setText(getMsgFromJni());
    }
}
{% endhighlight %}

#### 运行结果

运行结果如下图所示：


<img style="height:400px; width:auto;" src="https://github.com/junyuecao/junyuecao.github.io/blob/master/assets/static/20160527.png?raw=true" alt="截图">


