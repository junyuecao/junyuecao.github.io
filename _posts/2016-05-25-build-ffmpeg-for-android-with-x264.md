---
layout: post
title: Mac下为Android编译FFMPEG和x264 （一）
description: "NDK提供了很好的交叉编译支持，让我们来学习一下如何在Mac下面通过NDK编译工具来编译一个Android上可以运行的ffmpeg"
category: Android
tags: ["ffmpeg", "Android", "Mac"]
---


#### 前言

ffmpeg是一个超级牛逼的音视频软件，它几乎可以处理市面上常见的所有音视频格式，许多著名的软件都是基于ffmpeg的，比如kmplayer之类的播放器，格式工厂之类的格式转换软件等。并且ffmpeg可以运行在多种平台上，其中就包括了我们的Android平台。

#### 1. 下载ffmpeg和x264

下载自然不必多说，到这个链接[http://ffmpeg.org/download.html](http://ffmpeg.org/download.html)，在右下角的`Get the Sources`中，有`Download Snapshot`和`git snapshot`，如果你想通过git来更新代码，那就点击`git snapshot`，如果不需要则选择`Download Snapshot`。当然你也可以选择直接git clone的方式下载代码，带git的代码库有100多M，还是比较大的。

然后到这里[http://www.videolan.org/developers/x264.html](http://www.videolan.org/developers/x264.html)下载最新的x264代码，同样可以下载压缩包或者直接git clone。

#### 2. 配置

首先，确保你已经安装了ndk，如果没有安装，到这里下载最新版：[https://developer.android.com/ndk/downloads/index.html](https://developer.android.com/ndk/downloads/index.html)。下载完后解压到一个任意目录下。

由于ffmpeg默认的编译产物so文件后面会加上版本号，我们可以首先把编译产物命名修改成libxxxxx-<version>.so以及libxxxxx.a

打开ffmpeg目录下的`configure`文件,找到如下一段：

{% highlight shell%}
SLIBNAME_WITH_VERSION='$(SLIBNAME).$(LIBVERSION)'
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
{% endhighlight %}

改成如下（注意我注释掉的行）：

{% highlight shell%}
SLIBNAME_WITH_VERSION='$(SLIBNAME).$(LIBVERSION)'
# SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'  
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
# SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
# SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
{% endhighlight %}


接下来就是配置x264和ffmpeg了,这里我把x264放入了ffmpeg的目录，目录结构如下

{% highlight shell%}
├── ffmpeg
    ├── x264
    └── others....
{% endhighlight %}

在ffmpeg下新建一个文件 `build_x264.sh`，内容如下：

{% highlight shell%}
cd x264
export PLATFORM_VERSION=android-24
export ANDROID_NDK=/Users/junyuecao/Work/adt-bundle-mac-x86_64-20140702/sdk/ndk-bundle #ndk 目录根据你的安装目录
export TOOLCHAIN=../../fftoolchain #toolchain 安装目录
export SYSROOT=$TOOLCHAIN/sysroot/
export PLATFORM=$ANDROID_NDK/platforms/$PLATFORM_VERSION/arch-arm
export PREFIX=../android-lib #编译结果的目录

#生成toolchain目录，这一段可以在第一次运行后注释掉
$ANDROID_NDK/build/tools/make-standalone-toolchain.sh \
    --toolchain=arm-linux-androideabi-4.9 \
    --platform=$PLATFORM_VERSION --install-dir=$TOOLCHAIN 

#
./configure \
    --prefix=$PREFIX \
    --enable-static \
    --enable-shared \
    --enable-pic \
    --disable-asm \
    --disable-cli \
    --host=arm-linux \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --sysroot=$PLATFORM

make -j8
make install

cd ..
{% endhighlight %}

在ffmpeg下新建一个文件 `build_ffmpeg_with_x264.sh`，内容如下：

{% highlight shell%}
cd x264
export ANDROID_NDK=/Users/junyuecao/Work/adt-bundle-mac-x86_64-20140702/ndk #ndk 目录根据你的安装目录
export TOOLCHAIN=../fftoolchain #toolchain 安装目录 和上面的不同，因为x264在ffmpeg的下级
export SYSROOT=$TOOLCHAIN/sysroot/
export PLATFORM=$NDK/platforms/android-8/arch-arm
export PREFIX=../ffandroid #编译结果的目录 最终生成的编译结果

#生成toolchain目录，这一段可以在第一次运行后注释掉
$ANDROID_NDK/build/tools/make-standalone-toolchain.sh \
    --toolchain=arm-linux-androideabi-4.9 \
    --platform=android-8 --install-dir=$TOOLCHAIN 

#
./configure \
    --prefix=$PREFIX \
    --enable-static \
    --enable-pic \
    --disable-asm \
    --disable-cli \
    --host=arm-linux \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --sysroot=$PLATFORM

make -j8
make install

# 加入x264编译库
EXTRA_CFLAGS="-I./android-lib/include" 
EXTRA_LDFLAGS="-L./android-lib/lib"


./configure \
    --target-os=linux \
    --prefix=$PREFIX \
    --enable-cross-compile \
    --enable-runtime-cpudetect \
    --disable-asm \
    --disable-doc \
    --arch=arm \
    --cc=$TOOLCHAIN/bin/arm-linux-androideabi-gcc \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --disable-stripping \
    --nm=$TOOLCHAIN/bin/arm-linux-androideabi-nm \
    --sysroot=$PLATFORM \
    --enable-gpl \
    --enable-static \
    --disable-shared \
    --enable-version3 \
    --enable-small \
    --disable-vda \
    --disable-iconv \
    --disable-encoders \
    --enable-libx264 \
    --enable-encoder=libx264 \
    --disable-muxers \
    --enable-muxer=mov \
    --enable-muxer=ipod \
    --enable-muxer=psp \
    --enable-muxer=mp4 \
    --enable-muxer=avi \
    --disable-decoders \
    --enable-decoder=aac \
    --enable-decoder=aac_latm \
    --enable-decoder=h264 \
    --enable-decoder=mpeg4 \
    --disable-demuxers \
    --enable-demuxer=h264 \
    --enable-demuxer=mov \
    --disable-parsers \
    --enable-parser=aac \
    --enable-parser=ac3 \
    --enable-parser=h264 \
    --disable-protocols \
    --enable-protocol=file \
    --enable-protocol=rtmp \
    --disable-bsfs \
    --enable-bsf=aac_adtstoasc \
    --enable-bsf=h264_mp4toannexb \
    --disable-indevs \
    --enable-zlib \
    --disable-outdevs \
    --disable-ffprobe \
    --disable-ffplay \
    --disable-ffmpeg \
    --disable-ffserver \
    --disable-debug \
    --extra-cflags=$EXTRA_CFLAGS \
    --extra-ldflags=$EXTRA_LDFLAGS


make clean 
make -j8
make install

# 这段解释见后文
$TOOLCHAIN/bin/arm-linux-androideabi-ld -rpath-link=$PLATFORM/usr/lib -L$PLATFORM/usr/lib -L$PREFIX/lib -soname libffmpeg.so -shared -nostdlib -Bsymbolic --whole-archive --no-undefined -o $PREFIX/libffmpeg.so \
    android-lib/lib/libx264.a \
    libavcodec/libavcodec.a \
    libavfilter/libavfilter.a \
    libswresample/libswresample.a \
    libavformat/libavformat.a \
    libavutil/libavutil.a \
    libswscale/libswscale.a \
    libpostproc/libpostproc.a \
    libavdevice/libavdevice.a \
    -lc -lm -lz -ldl -llog --dynamic-linker=/system/bin/linker $TOOLCHAIN/lib/gcc/arm-linux-androideabi/4.9/libgcc.a  
{% endhighlight %}

这里有两种选择：

 1. 打包出一个ffmpeg.so文件，上面的配置就是这样的结果
 2. 每个模块打包出单独的so,那么需要对上面的配置文件做两件事：1.去掉最后一段配置（make install 后面的），2.将configure选项--enable-static  --disable-shared 反过来--enable-shared  --disable-static

*另外在这段配置中我去掉了很多编码解码组件，只保留了我需要的组件，大家可以视情况而定，增减配置。*

#### 编译结果
最终编译结果在ffmpeg的编译脚本里的`PREFIX=../ffandroid`所定义的路径，对于上面说的两种方式产生的编译产物分别如下。
第一种：

{% highlight shell%}
.
├── include
│   ├── libavcodec
│   ├── libavdevice
│   ├── libavfilter
│   ├── libavformat
│   ├── libavutil
│   ├── libpostproc
│   ├── libswresample
│   └── libswscale
├── lib
│   ├── libavcodec.a
│   ├── libavdevice.a
│   ├── libavfilter.a
│   ├── libavformat.a
│   ├── libavutil.a
│   ├── libpostproc.a
│   ├── libswresample.a
│   ├── libswscale.a
│   └── pkgconfig
└── libffmpeg.so
{% endhighlight %}

只需要在Android中引入libffmpeg.so就可以使用了。

第二种：

{% highlight shell%}
.
├── include
│   ├── libavcodec
│   ├── libavdevice
│   ├── libavfilter
│   ├── libavformat
│   ├── libavutil
│   ├── libpostproc
│   ├── libswresample
│   └── libswscale
└── lib
    ├── libavcodec-57.so
    ├── libavcodec.so -> libavcodec-57.so
    ├── libavdevice-57.so
    ├── libavdevice.so -> libavdevice-57.so
    ├── libavfilter-6.so
    ├── libavfilter.so -> libavfilter-6.so
    ├── libavformat-57.so
    ├── libavformat.so -> libavformat-57.so
    ├── libavutil-55.so
    ├── libavutil.so -> libavutil-55.so
    ├── libpostproc-54.so
    ├── libpostproc.so -> libpostproc-54.so
    ├── libswresample-2.so
    ├── libswresample.so -> libswresample-2.so
    ├── libswscale-4.so
    ├── libswscale.so -> libswscale-4.so
    └── pkgconfig
{% endhighlight %}

这个方式出来的话，就需要把每一个so文件都引用一遍了。

#### 备注

我在编译单个so过程中遇到一个出线`undefined reference to 'x264_picture_init'`的错误，是因为上面配置文件最后一段没有加入`android-lib/lib/libx264.a` 静态文件的引用导致找不到x264库的函数，这还是由于我对C语言的编译链接过程以及动态库和静态库之间的区别理解太浅。这里将静态库合并成一个so文件的过程中需要保证每一个函数都能够找到，不像动态链接库，可以在需要时再去加载库文件。


#### 接下来

[后一篇文章](/android/2016/05/26/build-ffmpeg-for-android-with-x2642/)
