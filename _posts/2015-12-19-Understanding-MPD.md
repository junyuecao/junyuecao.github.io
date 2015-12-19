---
layout: post
title: MPD-DASH文件的结构
description: The Structure of an MPEG-DASH MPD:https://www.brendanlong.com/the-structure-of-an-mpeg-dash-mpd.html
category: Android
tags: ["Android"]
---

{% include JB/setup %}

### Periods

Periods包含在MPD元素的第一层，它用开始时间和持续时间来描述内容的一部分。一个场景或章节里可以有多个Periods， 也可以用来分离广告和正文内容。

### Adaptation Sets

Adaptation Set 包含了一个媒体流或者一组媒体流，最简单的情况下，一个Period用一个Adaptation Set包含所有的视频和音频，但是为了减少带宽每个流可以分到独立的Adaptation Set里。常见做法就是一个视频流，多个音频流（多语言）。Adaptation Set也可以包含字母或者Arbitrary元信息。

### Representation

Representation可以让一个Adaptation Set 通过不同的方式包含同一个内容。大部分情况下Representation用来提供不同屏幕尺寸和带宽，这样就可以让客户端在不缓冲的情况下选择最高的画质，并且不会浪费带宽来获取不需要的分辨率（720P的屏幕不需要1080P的画质）。Representation还可以用不同的编码，允许不同的客户端选择其支持的编码格式（比如浏览器中，有些支持MPEG－4AVC/H.264，另一些支持VP8），或者提供更高的质量的Representation给更新客户端，并且同时还支持老的（比如同时提供H264和H265）。多个编码格式在需要省电的设备上也很有用，有些设备可能会选择老的编码，因为它可以硬解（即便它可以软解更新的编码格式）。

Representation一般都是自动选择的，但是一些播放器允许用户指定他们的选择（特别是分辨率）。如果用户不想浪费带宽在某个视频上，可以选择他们想要的Representation（可能他只关注音频），或者他希望暂停视频来缓冲更高画质的视频。

### SubRepresentation

SubRepresentation包含只应用到Representation中一个媒体流上的的信息。例如，如果一个Representation又有音频又有视频，那么它可以包含一个SubRepresentation来指定额外的只应用到音频上的信息。这些额外的信息可以是特定编码，采样率，嵌入字幕等。SubRepresentation也可以提供从多个容器里抽取一个流的必要信息，或者抽取出一个流的低质量版（比如只在快进模式下有用的I帧）

### Media Segment

Media Segment是DASH客户端实际播放的媒体文件， 通常是一个接一个地播放，就好像他们是同一个文件一样（当然在切换Representation时要复杂的多）。格式问题会在我的另一篇关于profiles的文章中讲到，但是MPEG中描述的两个容器：ISO Base Media File Format（ISOBMFF）是和MPEG－4类似的容器格式；MPEG－TS。Dash里的WebM在这里描述http://wiki.webmproject.org/adaptive-streaming/webm-dash-specification

Media Segment的地址可以用BaseURL来描述一个单段Representation，Segment的列表或者模版（SegmentTemplate）。SegmentBase里的信息会应用到所有的segment上。Segment开始时间和持续时间可以用SegmentTimeline来描述（对于直播流特别重要，使客户端可以快速决定最新的一段）。这个信息也可以在MPD的更高层出现，这种情况下，这个信息主要提供的是默认的信息，除非XML结构的底层覆盖了这个信息。这一点是SegmentTemplate最有用的地方。

Segment可以在不同的文件里（直播中常用），也可以在一个文件里的字节范围（静态的”on-demand“常用）

### Index Segment

Index Segment有两种类型：一个Representation用一个Presentation Index Segment，或者一个Media Segment配一个Single Index Segment。Presentation Index Segment永远是一个单独的文件，而Single Index Segment可以是同和Media Segment同一个文件的字节范围。

### 例子：

结束之间给一个注释过的MPD的例子，看看这些部分是怎么合在一起的：

{% highlight xml %}

<?xml version="1.0"?>
<MPD xmlns="urn:mpeg:dash:schema:mpd:2011" profiles="urn:mpeg:dash:profile:full:2011" minBufferTime="PT1.5S">
    <!-- Ad -->
    <Period duration="PT30S">
        <BaseURL>ad/</BaseURL>
        <!-- Everything in one Adaptation Set -->
        <AdaptationSet mimeType="video/mp2t">
            <!-- 720p Representation at 3.2 Mbps -->
            <Representation id="720p" bandwidth="3200000" width="1280" height="720">
                <!-- Just use one segment, since the ad is only 30 seconds long -->
                <BaseURL>720p.ts</BaseURL>
                <SegmentBase>
                    <RepresentationIndex sourceURL="720p.sidx"/>
                </SegmentBase>
            </Representation>
            <!-- 1080p Representation at 6.8 Mbps -->
            <Representation id="1080p" bandwidth="6800000" width="1920" height="1080">
                <BaseURL>1080p.ts</BaseURL>
                <SegmentBase>
                    <RepresentationIndex sourceURL="1080p.sidx"/>
                </SegmentBase>
            </Representation>
        </AdaptationSet>
    </Period>
    <!-- Normal Content -->
    <Period duration="PT5M">
        <BaseURL>main/</BaseURL>
        <!-- Just the video -->
        <AdaptationSet mimeType="video/mp2t">
            <BaseURL>video/</BaseURL>
            <!-- 720p Representation at 3.2 Mbps -->
            <Representation id="720p" bandwidth="3200000" width="1280" height="720">
                <BaseURL>720p/</BaseURL>
                <!-- First, we'll just list all of the segments -->
                <!-- Timescale is "ticks per second", so each segment is 1 minute long -->
                <SegmentList timescale="90000" duration="5400000">
                    <RepresentationIndex sourceURL="representation-index.sidx"/>
                    <SegmentURL media="segment-1.ts"/>
                    <SegmentURL media="segment-2.ts"/>
                    <SegmentURL media="segment-3.ts"/>
                    <SegmentURL media="segment-4.ts"/>
                    <SegmentURL media="segment-5.ts"/>
                    <SegmentURL media="segment-6.ts"/>
                    <SegmentURL media="segment-7.ts"/>
                    <SegmentURL media="segment-8.ts"/>
                    <SegmentURL media="segment-9.ts"/>
                    <SegmentURL media="segment-10.ts"/>
                </SegmentList>
            </Representation>
            <!-- 1080p Representation at 6.8 Mbps -->
            <Representation id="1080p" bandwidth="6800000" width="1920" height="1080">
                <BaseURL>1080/</BaseURL>
                <!-- Since all of our segments have similar names, this time we'll use a SegmentTemplate -->
                <SegmentTemplate media="segment-$Number$.ts" timescale="90000">
                    <RepresentationIndex sourceURL="representation-index.sidx"/>
                    <!-- Let's add a SegmentTimeline so the client can easily see how many segments there are
                         -->
                    <SegmentTimeline>
                        <!-- This reads: Starting from time 0, there are 10 segments with a duration of
                             (5400000 / @timescale) seconds -->
                        <S t="0" r="10" d="5400000"/>
                    </SegmentTimeline>
                </SegmentTemplate>
            </Representation>
        </AdaptationSet>
        <!-- Just the audio -->
        <AdaptationSet mimeType="audio/mp2t">
            <BaseURL>audio/</BaseURL>
            <!-- We're just going to offer one audio representation, since audio bandwidth isn't very
                 important. -->
            <Representation id="audio" bandwidth="128000">
                <SegmentTemplate media="segment-$Number$.ts" timescale="90000">
                    <RepresentationIndex sourceURL="representation-index.sidx"/>
                    <SegmentTimeline>
                        <S t="0" r="10" d="5400000"/>
                    </SegmentTimeline>
                </SegmentTemplate>
            </Representation>
        </AdaptationSet>
    </Period>

</MPD>

{% endhighlight %}

### 总结

本文提供了足够的信息让你理解一个MPD文件的结构，以及一个DASH客户端工作的基本思想。下次，我会讨论一些额外的元数据，可以让客户端更加智能，并提供更好的用户体验。