---
layout: post
title: Android硬编码的坑
description: "记录Android硬编码使用过程中的坑"
category: 随笔
tags: ["Android","编解码"]
---

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

### 1. 像素16对齐
### 2. 一台手机上同时使用的硬编码数量有限，如果多于手机的限制，就会使用兼容的软编码实现，而在Android5.x上，这个软编码基本属于不可用状态（限制非常多）
如果使用MediaCodec.createEncoderByType("video/avc"); 在5.0的手机上，可能会回退到『OMX.google.h264.encoder』，此时能够编码的视频参数极其有限，可以通过MediaCodec.getCodecInfo来判断是否回退到了Google的软编，如果是的话，最好不要再5.x手机上使用，可以提示错误之类的。
因为底层代码会匹配所有该类型的编码器，并且一个个尝试：https://android.googlesource.com/platform/frameworks/av/+/69113e8ccc30fbdb8733ca2070bd3491b21e656d/media/libstagefright/ACodec.cpp ： L4527
而Google的软编CODEC在5.x上基本属于不可用的状态：https://stackoverflow.com/questions/30505875/how-to-use-software-codec-in-android-using-mediacodec


[OMX.qcom.video.encoder.avc] storeMetaDataInBuffers (output) failed w/ err -2147483648 似乎不会出什么问题
https://android.googlesource.com/platform/frameworks/av/+/e2d617f5ba7fb90f27b03e2593666b2c927e4dc9/media/libstagefright/omx/OMXNodeInstance.cpp
https://android.googlesource.com/platform/frameworks/native/+/jb-mr1-dev/include/media/hardware/HardwareAPI.h

编码器两种使用方式：buffer输入，Surface输入
解码器两种输出方式：buffer输出，Surface输出

编码时，如果输入的帧率大于给编码器设置的帧率，编码器将会以高帧率去匹配编码器设置的码率，导致视频模糊，此时可以通过给InputSurface设置时间戳的方式解决此问题

