---
layout: post
title: FFmpeg 主要组件学习笔记 （一） libavformat
description: "记录FFmpeg的一些知识点"
category: ffmpeg
tags: ["ffmpeg"]
---

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

### 1. Libavformat该要

#### 1.1 简要介绍

Libavformat（lavf）是用来处理音视频媒体文件封装格式的库。它主要的作用包括两种：

 1. 将一个媒体文件抽取出不同的流来，比如视频流，音频流
 2. 将指定的数据写入到指定的封装格式，比如将视频流音频流写入到mp4格式里。

<br/>
它还包含了一个I/O模块，可以支持一系列协议来访问数据（比如：文件，tcp，http等等），在使用lavf之前，你需要现调用`av_register_all()`来注册所有编译好的封装器（muxers），解封装器（demuxers）和协议。除非你十分确定你不会使用网络功能，否则你还需要调用一下`avformat_network_init()`。

lavf支持的输入格式可以由数据结构`AVInputFormat`来描述，对应的输出格式由`AVOutputFormat`来描述。如果你想遍历所有的已注册的输入输出格式，你可以调用`av_iformat_next()`和`av_oformat_next()`。协议层不属于公共API，所以你只能通过`avio_enum_protocols()`来查看所有支持协议的名字。

无论是封装还是解封装，lavf的主要结构都由`AVFormatContext`来描述，通过它可以输出关于我们要读和写的文件的所有信息。正如大部分的Libavformat结构体一样，它的大小也不是公共API，所以不能通过栈上分配或者直接`av_malloc()`的方式来分配。要创建一个`AVFormatContext`你需要调用`avformat_alloc_context()`（类似于`avformat_open_input`之类的函数也是一样的道理）。

`AVFormatContext`包涵的最重要的一些信息如下：

 - 输入或者输出格式，可以自动监测或者由用户自行设置；输出格式由用户自行设置。 
 - 一个`AVStreams`数组，描述了这个文件里存储的所有的基础流。`AVStream`通常由他们在这个数组的索引来引用。
 - 一个I/O context。一般由lavf来打开，或者由用户设置输入，输出的都由用户自行设置（除非你用的是`AVFMT_NOFILE`格式）。

#### 1.2 传递选项到封装器或解封装器

可以通过使用`AVOptions`来配置lavf的封装器和解封装器。通用（格式无关）的libavformat的配置信息会由`AVFormatContext`来提供，用户可以通过调用`av_opt_next()`或者`av_opt_find()`，传入`AVFormatContext`（或者通过`avformat_get_class()`获取到的它的`AVClass`）来获得。私有（格式特定）的配置信息由`AVFormatContext.priv_data`提供，但是只有对应格式数据结构里的`AVInputFormat.priv_class`/`AVOutputFormat.priv_class`非空时有效。更多的配置项可能由`I/O context`（当它的AVClass非空时才有效）和协议层来提供。查看内嵌在`AVOptions`文档里的讨论来学习如何访问这些配置。

#### 1.3 url

libavformat里的URL串由协议名，一个冒号‘:’和协议特定字符串，不含协议名和冒号的URL一般用于本地文件，也可以支持，但是已经弃用了。本地文件使用`file:`来访问。

重要提示：不要使用未经检查的未知来源的协议。

注意一些协议非常（过于）强大，可以访问本地和远程的文件，文件内部部分，或者串联文件，访问本地音视频设备等等。

### 2. 解封装

解封装意思就是读取一个媒体文件，并将其分离成一个个数据快（packets）

一个[packet](https://ffmpeg.org/doxygen/3.0/structAVPacket.html)包涵了一个或者多个编码帧，并且属于一个基础数据流。在lavf的API里，`avformat_open_input()`函数用于打开一个文件，`av_read_frame()`用于读取一个packet，最后要调用`avformat_close_input()`来做清理工作。

#### 2.1 打开媒体文件

打开一个文件至少需要的信息是它的URL，将它传递给`avformat_open_input()`，代码如下：

{% highlight c %}
const char    *url = "file:in.mp3";
AVFormatContext *s = NULL;
int ret = avformat_open_input(&s, url, NULL, NULL);
if (ret < 0)
    abort();
{% endhighlight %}

上述代码尝试分配一个`AVFormatContext`，打开指定文件（自动检测类型）然后读取文件头，然后导出相应的信息存储到`s`里。一些格式是没有文件头的，或者文件头里没有足够的信息，那样的话推荐你调用`avformat_find_stream_info()`函数，它会试图读取并解码一些帧，来寻找缺失的信息。

一些情况下你希望自己提前分配一个`AVFormatContext`。 调用`avformat_alloc_context()`然后在传入到`avformat_open_input()`之前做一些调整。 有一种情况就是你想使用自定义函数来读取输入数据，而不是用lavf内部的I/O层，要想实现这个，你需要用`avio_alloc_context()`创建你自己的AVIOContext，传递你的读取回调进去，然后将`AVFormatContext`的`pb`字段设置为你创建的`AVIOContext`。

由于打开文件的格式在调用`avformat_open_input()`之前是不知道的，因此对于自己预分配的`AVFormatContext`来说是无法设置解封装器的私有配置的。要设置这些options，需要在调用`avformat_open_input()`时封装进一个`AVDictonary`里传入：

{% highlight c %}
AVDictionary *options = NULL;
av_dict_set(&options, "video_size", "640x480", 0);
av_dict_set(&options, "pixel_format", "rgb24", 0);
if (avformat_open_input(&s, url, NULL, &options) < 0)
    abort();
av_dict_free(&options);
{% endhighlight %}

上述代码将`video_size`和`pixel_format`传入解封装器。对于raw video解封装器来说这是他们是必要参数，因为没有这些参数就无法知道怎么分析raw video数据。如果说格式其实是一些不同于raw video的东西，这些传入的参数就无法识别，因此也就没有用处。这些无法识别的参数会被返回来（识别的参数就被消费掉了，不会再返回），调用程序可以处理这些未识别的参数：

{% highlight c %}
AVDictionaryEntry *e;
if (e = av_dict_get(options, "", NULL, AV_DICT_IGNORE_SUFFIX)) {
    fprintf(stderr, "Option %s not recognized by the demuxer.\n", e->key);
    abort();
}
{% endhighlight %}

完成读取文件后，你必须调用`avformat_close_input()`来关闭文件，它将会释放所有与该文件相关的资源。

#### 2.2 读取文件

通过重复调用`av_read_frame()`可以读取一个已经打开的`AVFormatContext`，每一次调用，如果成功的话，会返回一个`AVPacket`，其中包含了从一个`AVStream`里取出的编码过的数据，这个流的索引为`AVPacket.stream_index`。如果用户想要解码数据，那么这个packet可以直接传递到编解码器`libavcodec`来解码，解码函数为`avcodec_decode_video2()`， `avcodec_decode_audio4()`， `avcodec_decode_subtitle2()`。分别可以解码视频，音频和字幕。

如果已知的话`AVPacket.pts`, `AVPacket.dts` 和 `AVPacket.duration`会自动设置好，如果无法获知的话他们也可能不会被设置（也就是说对于pts/dts来说是`AV_NOPTS_VALUE `,对于duration来说是0）。时间信息单位为`AVStream.time_base`，也就是如果要转换成秒的话，需要把时间信息乘以`AVStream.time_base`。

如果返回的packet中`AVPacket.buf`已经设好，那么代表这个packet是动态分配的，并且用户可以一直持有它。相反如果`AVPacket.buf`为`NULL`，那么代表这个packet的数据是由一个解封装器内部的静态存储持有的，并且这个packet在下次调用`av_read_frame()`或者关闭文件后就会失效了。 如果调用方想要一个更长生命周期，那么调用`av_dup_packet()`可以分配内存`av_malloc()`并复制一份。两种情况下在不需要这个packet的时候都要调用`av_packet_unref()`来释放。

### 3. 封装

封装器（Muxer）将编码后的数据以`AVPacket`的形式写入到文件或者其它输出指定容器格式的字节流中。

主要的封装器的API函数有：`avformat_write_header()`用于写入文件头，`av_write_frame()` / `av_interleaved_write_frame()`用于写数据块packet，以及`av_write_trailer()`用于结束写文件。

在封装过程开始时，调用者首先必须调用`avformat_alloc_context()`来创建一个封装context，接下来填入一些字段来配置封装器：

 - `oformat` 必须设置, 用于指定使用的封装器
 - 除非设置的格式是`AVFMT_NOFILE`类型,否则`pb`字段必须设置为一个已打开的`IO context`, 可以通过`avio_open2()`返回活着自定义一个
 - 除非设置的格式是`AVFMT_NOSTREAMS`类型,否则必须至少通过`avformat_new_stream()`创建一个流, 调用者需要填充这个流的`stream codec context`信息, 比如编解码类型,id以及其它的一些参数(例如:视频宽高,像素或者采样格式等). 流的timebase要设置成调用者期望的.(注意: 封装器使用的timebase可能有所不同).
 - 建议只手动初始化`AVCodecContext`中的有关的字段, 而不是在重封装过程中使用`avcodc_copy_context()`:无法保证解码器context适用于输入和输出的format context.
 - 调用者可能需要设置一些额外信息,比如一些全局信息,或者针对于每个流的元信息, 章节, 脚本之类的,正如在 [AVFormatContext](https://ffmpeg.org/doxygen/3.0/structAVFormatContext.html)文档里描述的.这些信息是否最终会存储在输出中取决于容器格式和封装器的支持.

当封装器设置完成后, 调用者必须调用`avformat_write_header()`来初始化封装器内部结构,并写入文件头. 这个阶段是否往IO context里写入东西取决于封装器, 但是这个函数必须要调用. 任何封装器的私有选项也要在这个函数调用时传入.

然后就可以重复调用`av_write_frame()`或者`av_interleaved_write_header()`来发送数据到封装器(参考这些函数的文档来分辨他们的区别. 一个封装context只允许使用其中一种,不能混着用). 注意发送到封装器的packet的时间信息必须对应`AVStream`的时间基数,这个时间基数会在`avformat_write_header()`步骤写入,有可能与调用者设置的不同.

当所有数据都写入后, 调用者必须调用`av_write_trailer()`进行文件缓冲数据的关闭和文件的关闭. 然后关闭IO context(如果有的话), 最后调用`avformat_free_context()`释放封装context.

### 4. I/O 读写(貌似只有读)

#### 4.1 目录列表

目录列表API可以用来列出远程服务器上的文件列表.

使用场景:

 - 一个打开文件的对话框用来选择远程地址上的文件
 - 一个可以播放指定目录下所有文件的播放器

##### 打开目录

首先, 打开目录需要调用`avio_open_dir()`,并提供URL和一些协议相关参数(通过`AVDictionary`传入). 如果调用成功,这个函数返回0或者正数,并且分配一个`AVIODirContext`.

{% highlight c %}
AVIODirContext *ctx = NULL;
if (avio_open_dir(&ctx, "smb://example.com/some_dir", NULL) < 0) {
    fprintf(stderr, "Cannot open directory.\n");
    abort();
}
{% endhighlight %}

上述代码尝试打开一个`smb`协议的目录, 没有传递额外参数.

##### 读取条目

每个目录的条目(例如:文件,目录,其它比如`AVIODirEntryType`的)由一个`AVIODirEntry`表示, 读取连续的条目可以通过`avio_read_dir()`来完成.每一次调用如果成功返回0或者正数. 如果读取到了`NULL`条目,就可以停止了, 表示已经没有可以读取的条目了. 下面的代码从一个context关联的目录中读取所有条目,并且把文件名打印到标准输出流里.

{% highlight c %}
AVIODirEntry *entry = NULL;
for (;;) {
    if (avio_read_dir(ctx, &entry) < 0) {
        fprintf(stderr, "Cannot list directory.\n");
        abort();
    }
    if (!entry)
        break;
    printf("%s\n", entry->name);
    avio_free_directory_entry(&entry);
}
{% endhighlight %}

### 5. 公共元数据API

元数据API可以让`libavformat`在解封装的时候导出元数据标签.

相反它也可以让客户端在封装的时候设置元数据标签.

元数据由键值对表示,使用`AVDictionary`封装好后,读取或设置到`AVFormatContext`, `AVStream`,`AVChapter`和`AVProgram`的`metadata`字段里. 和FFmpeg的其它所有字符串一样, 元数据默认都认为是UTF-8编码的Unicode字符, 注意大部分情况下解封装器导出的元数据不会做字符编码检查.

重要的一些概念:

 - 键是唯一的,不能存在两个相同的键. 在语义上也要求是唯一的,比如解封装器不知道如何处理多个不同的键但实际上语义是一样的,比如键名`Author5`,键名`Author6`. 这里必须把所有tag放到同一个字段里.
 - 元数据是平铺的,没有层次结构的. 没有子tag.如果你要保存类似于制作人Alice和演员Bob的孩子的电子邮件,你应该这样存:键名为`alice_and_bobs_childs_email_address`.
 - 多个modifier可以应用到键名上,modifier的意思就是说这个键在不同的情况下有不同的值. 可以通过短横线'-'加到键名的末尾,这些modifier列表如下,顺序必须和下面的一致.比如可以是 `foo-eng-sort`但不能是`foo-sort-eng`
    - 语言 例如:`Author-ger=Michael` ,`Author-eng=Mike`, 默认语言是没有指定的"Author标签"
    - 排序 可以调整一个tag的顺序 入`artist="The Beatles"`,`artiest-sort="Beatles The"`
 - 一些协议和解封装器支持元数据更新.在成功调用`av_read_packet()`后,`AVFormatContext.event_flags`或者`AVStream.event_flags`将会更新来表示元数据变化了.
 - 解封装器尝试导出元数据为一个通用格式. 没有通用等价tag会保留在容器里,下面是通用tag的名字:

| album        | -- | name of the set this work belongs to |
| album_artist | -- | main creator of the set/album, if different from artist. e.g. "Various Artists" for compilation albums. |
| artist       |--  | main creator of the work |
| comment      | -- | any additional description of the file. |
| composer     | -- | who composed the work, if different from artist.|
| copyright    | -- | name of copyright holder.|
| creation_time| -- | date when the file was created, preferably in ISO 8601.|
| date         | -- | date when the work was created, preferably in ISO 8601.|
| disc         | -- | number of a subset, e.g. disc in a multi-disc collection.|
| encoder      | -- | name/settings of the software/hardware that produced the file.|
| encoded_by   | -- | person/group who created the file.|
| filename     | -- | original name of the file.|
| genre        | -- | <self-evident>.|
| language     | -- | main language in which the work is performed, preferably in ISO 639-2 format. Multiple languages can be specified by separating them with commas. |
| performer   | -- |artist who performed the work, if different from artist. E.g for "Also sprach Zarathustra", artist would be "Richard Strauss" and performer "London Philharmonic Orchestra".|
| publisher |    -- | name of the label/publisher.|
| service_name |     -- | name of the service in broadcasting (channel name).|
| service_provider | -- | name of the service provider in broadcasting.|
| title |        -- | name of the work.|
| track |        -- | number of this work in the set, can be in form current/total.|
| variant_bitrate | -- | the total bitrate of the bitrate variant that the current stream is part of|

### 6. 核心函数

参考:[https://ffmpeg.org/doxygen/3.0/group__lavf__core.html](https://ffmpeg.org/doxygen/3.0/group__lavf__core.html)

### 7. 工具函数

参考:[https://ffmpeg.org/doxygen/3.0/group__lavf__misc.html](https://ffmpeg.org/doxygen/3.0/group__lavf__misc.html)

### 参考资料
- [https://ffmpeg.org/doxygen/3.0/group__libavf.html#details](https://ffmpeg.org/doxygen/3.0/group__libavf.html)
- [https://ffmpeg.org/doxygen/3.0/group__lavf__decoding.html](https://ffmpeg.org/doxygen/3.0/group__lavf__decoding.html)
- [https://ffmpeg.org/doxygen/3.0/group__lavf__encoding.html](https://ffmpeg.org/doxygen/3.0/group__lavf__encoding.html)

