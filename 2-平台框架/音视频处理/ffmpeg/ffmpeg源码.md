### 一、目录结构
libavcodec 用于存放各个encode/decode模块，用于各种类型声音/图像编解码

libavdevice 对输出输入设备的支持；

Libavfilter 用于存放滤镜和滤镜相关的操作

libavformat 用于存放muxer/demuxer模块，对音频视频格式的解析;用于各种音视频封装格式的生成和解析，包括获取解码所需信息以生成解码上下文结构和读取音视频帧等功能；

libavresample 音频混音和重采样

libavutil 集项工具，包含一些公共的工具函数；用于存放内存操作等辅助性模块，是一个通用的小型函数库，该库中实现了CRC校验码的产生，128位整数数学，最大公约数，整数开方，整数取对数，内存分配，大端小端格式的转换等功能

libpostproc 用于后期效果处理；

libswscale 用于视频场景比例缩放、色彩映射转换；

比较重要的是libavcodec/libavformat

#### 二、重要的结构体


#### 三、重要的api
##### 3.1、AVFormatContext *avformat_alloc_context();
```cpp
AVFormatContext *avformat_alloc_context(void)
{
    AVFormatContext *ic;
    AVFormatInternal *internal;
    ic = av_malloc(sizeof(AVFormatContext));
    internal = av_mallocz(sizeof(*internal));
    avformat_get_context_defaults(ic);

    return ic;
}

void avformat_get_context_defaults(AVFormatContext *s)
{
    s->io_open  = io_open_default;
    s->io_close = io_close_default;
}
```
主要是初始化AVFormatContext和AVFormatInternal
AVFormatInternal是ffmpeg内部使用
为io_open函数赋值 ffio_open_whitelist即是avio_open的函数

```cpp
int io_open_default(AVFormatContext *s, AVIOContext **pb,
                           const char *url)
{
    return ffio_open_whitelist(pb, url, flags, &s->interrupt_callback, options, s->protocol_whitelist, s->protocol_blacklist);
}
```


##### 3.2、avformat_open_input
打开媒体文件 读取头信息
```cpp
int avformat_open_input(AVFormatContext **ps, const char *filename,
ff_const59 AVInputFormat *fmt, AVDictionary **options) {
	if ((ret = init_input(s, filename, &tmp)) < 0)
        goto fail;
    if (!(s->flags&AVFMT_FLAG_PRIV_OPT) && s->iformat->read_header)
        if ((ret = s->iformat->read_header(s)) < 0)
            goto fail;
}
```
主要是init_input 用于打开媒体数据 并根据匹配规则 找到最合适的协议类型(AVInputFormat)
AVInputFormat->read_header即根据对应的协议 读取媒体头信息并创建AVStream
AVInputFormat内部有函数指针 不同的文件协议 有不同的实现

** 3.2.2、init_input **

参考:
https://blog.csdn.net/u011913612/article/details/53482773

s->pb为AVFormatContext的AVIOContext

```cpp
int init_input(AVFormatContext *s, const char *filename,
                      AVDictionary **options)
{
    AVProbeData pd = { filename, NULL, 0 };
    int score = AVPROBE_SCORE_RETRY;
	//1、如果自定义了AVIOContext 则使用自定义的
    if (s->pb) {
        s->flags |= AVFMT_FLAG_CUSTOM_IO;
        //1.2 如果没有定义AVInputFormat 则由ffmpeg去推测
        if (!s->iformat)
            return av_probe_input_buffer2(s->pb, &s->iformat, filename,
                                         s, 0, s->format_probesize);
        else if (s->iformat->flags & AVFMT_NOFILE)
            av_log(s, AV_LOG_WARNING, "Custom AVIOContext makes no sense and "
                                      "will be ignored with AVFMT_NOFILE format.\n");
        return 0;
    }
	//2、如果指定了AVInputFormat 则直接返回
    //或者如果没有指定AVInputFormat 就av_probe_input_format2(扩展名/文件类型字符串)去推测
    if ((s->iformat && s->iformat->flags & AVFMT_NOFILE) ||
        (!s->iformat && (s->iformat = av_probe_input_format2(&pd, 0, &score))))
        return score;
	//3、如果通过文件名或类型没有匹配到 就打开文件 然后读取内容再推测
    if ((ret = s->io_open(s, &s->pb, filename, AVIO_FLAG_READ | s->avio_flags, options)) < 0)
        return ret;

    if (s->iformat)
        return 0;
    return av_probe_input_buffer2(s->pb, &s->iformat, filename,
                                 s, 0, s->format_probesize);
}
```

** 3.2.2.1、av_probe_input_format2 **
内部调用av_probe_input_format3
```cpp
AVInputFormat *av_probe_input_format3(ff_const59 AVProbeData *pd
, int is_opened, int *score_ret) {
	AVProbeData lpd = *pd;
    const AVInputFormat *fmt1 = NULL;
    ff_const59 AVInputFormat *fmt = NULL;
    int score, score_max = 0;

	//遍历所有的复用解复用器
    //demuxer_list位于demuxer_list.c
	while ((fmt1 = av_demuxer_iterate(&i))) {
        score = 0;
        //如果有探测函数 则调用
        if (fmt1->read_probe) {
            score = fmt1->read_probe(&lpd);
        //否则匹配扩展名(.mp4/.flv)
        } else if (fmt1->extensions) {
            if (av_match_ext(lpd.filename, fmt1->extensions))
                score = AVPROBE_SCORE_EXTENSION;//50
        }
        //匹配媒体类型(video/x-flv video/mp4)
        if (av_match_name(lpd.mime_type, fmt1->mime_type)) {
            if (AVPROBE_SCORE_MIME > score) {
                av_log(NULL, AV_LOG_DEBUG, "Probing %s score:%d increased to %d due to MIME type\n", fmt1->name, score, AVPROBE_SCORE_MIME);
                score = AVPROBE_SCORE_MIME;//75
            }
        }
        //记录得分最高的
        if (score > score_max) {
            score_max = score;
            fmt       = (AVInputFormat*)fmt1;
        } else if (score == score_max)
            fmt = NULL;
    }
    *score_ret = score_max;

    return fmt;
}

```
总结:遍历所有的解复用器 然后记录最匹配的解复用器
1、如果有探测函数 则调用 并记录分数 否则匹配扩展名(50分)
2、然后匹配媒体类型 如果匹配且比以前分高 就为75分

** 3.2.2.2、av_probe_input_buffer2 **

```cpp
int av_probe_input_buffer2(AVIOContext *pb, ff_const59 AVInputFormat **fmt,
                          const char *filename, void *logctx,
                          unsigned int offset, unsigned int max_probe_size) {
    //读取一定的数据 判断AVInputFormat 如果数据不够 再读取一些
	for (probe_size = PROBE_BUF_MIN; probe_size <= max_probe_size && !*fmt;
         probe_size = FFMIN(probe_size << 1,
                            FFMAX(max_probe_size, probe_size + 1))) {
        //读取数据
        if ((ret = avio_read(pb, buf + buf_offset,
                             probe_size - buf_offset)) < 0) {
		//匹配最合适的解复用器
        *fmt = av_probe_input_format2(&pd, 1, &score);
    }
}
```
##### 3.3、avformat_find_stream_info
获取流信息 并更新AVStream参数

** 第一个循环 主要初始化解析器 查找解码器 **
```cpp
int avformat_find_stream_info(AVFormatContext *ic, AVDictionary **options) {
    for (i = 0; i < ic->nb_streams; i++) {
    	//如果解析器为空 则创建
    	if (!st->parser && !(ic->flags & AVFMT_FLAG_NOPARSE) && st->request_probe <= 0) {
            st->parser = av_parser_init(st->codecpar->codec_id);
        }
		//将解析器的参数拷贝到解码上下文中
        ret = avcodec_parameters_to_context(avctx, st->codecpar);
        //查找解码器
        codec = find_probe_decoder(ic, st, st->codecpar->codec_id);
    }
```
主要方法
```cpp
AVCodecParserContext *av_parser_init(int codec_id) {
	//根据id查找解析器 av_parser_iterate为parser_list
    //定义在parser_list.c 不同的解码器有不同的实现
	 while ((parser = av_parser_iterate(&i))) {
            goto found;
    }

    ret = parser->parser_init(s);
}
//查找解码器
AVCodec *find_probe_decoder(AVFormatContext *s, const AVStream *st, enum AVCodecID codec_id)
{
    const AVCodec *codec;
#if CONFIG_H264_DECODER
	//如果是h264 直接查找
    if (codec_id == AV_CODEC_ID_H264)
        return avcodec_find_decoder_by_name("h264");
#endif
	//如果本身有解码器就用 否则通过id查找解码器
    codec = find_decoder(s, st, codec_id);

    return codec;
}

```

** 第二个循环 主要用于查找解码器 更新解码器信息 **
```cpp
for (;;) {
	for (i = 0; i < ic->nb_streams; i++) {
    	//检查编解码器里的参数是否完整
    }

    //如果编解码器有参数不完整 需要读取一部分数据(一些没有头的媒体文件)
    ret = read_frame_internal(ic, &pkt1);
	//如果信息还是不全 则需要打开解码器 解码一部分数据
    try_decode_frame(ic, st, pkt,);
}
```

```cpp
int read_frame_internal(AVFormatContext *s, AVPacket *pkt) {
	while (!got_packet && !s->internal->parse_queue) {
    	//读取一帧数据 最终调用AVInputFormat去读
		ret = ff_read_packet(s, pkt);
        //使用解复用器解数据
        parse_packet(s, pkt, st->index, 1);
	}
}
```

** 文件尾操作 **
如果到了文件尾 再次检查解码器参数 如果不完整 调用avcodec_open2更新
```cpp
if (eof_reached) {
        for (stream_index = 0; stream_index < ic->nb_streams; stream_index++) {
            st = ic->streams[stream_index];
            if (!has_codec_parameters(st, NULL)) {
                const AVCodec *codec = find_probe_decoder(ic, st, st->codecpar->codec_id);
                if (avcodec_open2(avctx, codec)；
            }

            // EOF already reached while reading the stream above.
            // So continue with reoordering DTS with whatever delay we have.
            if (ic->internal->packet_buffer && !has_decode_delay_been_guessed(st)) {
                update_dts_from_pts(ic, stream_index, ic->internal->packet_buffer);
            }
        }
    }
```

** 将帧数据刷出来 **
```cpp
if (flush_codecs) {
	err = try_decode_frame()
}
```

** 计算fps **

```cpp
ff_rfps_calculate(ic);
```

** 获取视频原始图像格式 计算音频disposition **
```cpp
for (i = 0; i < ic->nb_streams; i++) {
	if (avctx->codec_type == AVMEDIA_TYPE_VIDEO) {
    	uint32_t tag= avcodec_pix_fmt_to_codec_tag(avctx->pix_fmt);
        if (avpriv_find_pix_fmt(avpriv_get_raw_pix_fmt_tags(), tag) == avctx->pix_fmt)
    } else if (avctx->codec_type == AVMEDIA_TYPE_AUDIO) {
   }
}
```

** 更新流的时间 **

```cpp
if (probesize)
        estimate_timings(ic, old_offset);
```

** 将内部参数更新到AVFormatContext.AVStream中 **

```cpp
    for (i = 0; i < ic->nb_streams; i++) {
        st = ic->streams[i];
        if (st->internal->avctx_inited) {
            ret = avcodec_parameters_from_context(st->codecpar, st->internal->avctx);
            ret = add_coded_side_data(st, st->internal->avctx);
        }
```

参考:
https://www.cnblogs.com/jiaoxiangjie/p/9984052.html
https://my.oschina.net/u/2326611/blog/809586
https://blog.csdn.net/u011913612/article/details/53642355

##### 3.4、av_find_best_stream
查找流(音频流、视频流)对应的索引

```cpp
int av_find_best_stream(AVFormatContext *ic, enum AVMediaType type,
                        int wanted_stream_nb, int related_stream,
                        AVCodec **decoder_ret, int flags)
{
    for (i = 0; i < nb_streams; i++) {
    	if (par->codec_type != type)
            continue;

        if (decoder_ret)
            decoder = find_decoder(ic, st, par->codec_id);
    }
    if (decoder_ret)
        *decoder_ret = (AVCodec*)best_decoder;
    return ret;
}
```
参考:
https://blog.csdn.net/wxl1986622/article/details/52779515

##### 3.5、av_read_frame

读取包数据 视频是一帧数据 音频可能有多帧

```cpp
int av_read_frame(AVFormatContext *s, AVPacket *pkt) {
	//如果有缓存 则从缓存读取一包数据
    //否则 从文件读取
	ret = s->internal->packet_buffer
           ? avpriv_packet_list_get(&s->internal->packet_buffer,
                   &s->internal->packet_buffer_end, pkt)
              : read_frame_internal(s, pkt);
	//从文件读取数据
    ret = read_frame_internal(s, pkt);
    //将读取的数据加入缓存
    ret = avpriv_packet_list_put(&s->internal->packet_buffer,
                                 &s->internal->packet_buffer_end,
                                 pkt, NULL, 0);
}
```

```cpp
//pkt_buffer为packet链表 获取头部packet
int avpriv_packet_list_get(AVPacketList **pkt_buffer,
                           AVPacketList **pkt_buffer_end,
                           AVPacket      *pkt) {
}

//将packet放入链表尾部
int avpriv_packet_list_put();

//读取一包数据
int read_frame_internal(AVFormatContext *s, AVPacket *pkt) {
	while (!got_packet && !s->internal->parse_queue) {
    	//读取一包数据
    	ret = ff_read_packet(s, pkt);
		//解复用
        if (st->discard < AVDISCARD_ALL)
            if ((ret = parse_packet(s, pkt, pkt->stream_index, 0)) < 0)
                return ret;
		//从缓存获取
        if (!got_packet && s->internal->parse_queue)
        	ret = avpriv_packet_list_get(&s->internal->parse_queue, &s->internal->parse_queue_end, pkt);
		//更新流数据
		update_stream_avctx(s);
    }
}

int ff_read_packet(AVFormatContext *s, AVPacket *pkt) {
	for (;;) {
    	//真正的读取一包数据
    	ret = s->iformat->read_packet(s, pkt);
        //放入缓存
        err = avpriv_packet_list_put(&s->internal->raw_packet_buffer,
                                 &s->internal->raw_packet_buffer_end,
                                 pkt, NULL, 0);
        //读取数据 更新解码器
        if ((err = probe_codec(s, st, pkt1)) < 0)
            return err;
    }
}

int parse_packet(AVFormatContext *s, AVPacket *pkt,
                        int stream_index, int flush) {
	len = av_parser_parse2();
    ret = avpriv_packet_list_put(&s->internal->parse_queue,
                                 &s->internal->parse_queue_end,
                                 &out_pkt, NULL, 0);
}

int av_parser_parse2(AVCodecParserContext *s, AVCodecContext *avctx,) {
	//最终调用parser_list.c中的解复用器解析
	index = s->parser->parser_parse(s, avctx,);
}

```

##### 3.6、avcodec_find_decoder_by_name/avcodec_find_decoder

```cpp

//通过名字查找解码器 avcodec_find_decoder_by_name最终调用
AVCodec *find_codec_by_name(const char *name, int (*x)(const AVCodec *))
{
	//所有编解码器定义在codec_list.c
    while ((p = av_codec_iterate(&i))) {
        if (!x(p))
            continue;
        if (strcmp(name, p->name) == 0)
            return (AVCodec*)p;
    }
}
AVCodec *find_codec(enum AVCodecID id, int (*x)(const AVCodec *))
{
    const AVCodec *p, *experimental = NULL;
    id = remap_deprecated_codec_id(id);
    while ((p = av_codec_iterate(&i))) {
        if (!x(p))
            continue;
    }

    return (AVCodec*)experimental;
}
```

##### 3.7、avcodec_open2

如果是编码器 则检查各种参数
然后调用codec->init初始化

```cpp
int avcodec_open2(AVCodecContext *avctx, const AVCodec *codec,){
	if (avcodec_is_open(avctx))
        return 0;

    if (avctx->codec->init && (!(avctx->active_thread_type&FF_THREAD_FRAME)
        || avci->frame_thread_encoder)) {
        ret = avctx->codec->init(avctx);
    }
}
```

##### 3.8、avcodec_send_packet
将一包数据发送到解码器
```cpp
int attribute_align_arg avcodec_send_packet(AVCodecContext *avctx, const AVPacket *avpkt) {
	AVCodecInternal *avci = avctx->internal;
    //将AVPacket放入AVCodecInternal的buffer_pkt 用于解码
    ret = av_packet_ref(avci->buffer_pkt, avpkt);
	//最终将avpkt的指针指向AVBSFInternal->buffer_pkt
    ret = av_bsf_send_packet(avci->bsf, avci->buffer_pkt);
    //如果帧缓存为空 则从解码器读取一帧数据
    if (!avci->buffer_frame->buf[0]) {
        ret = decode_receive_frame_internal(avctx, avci->buffer_frame);
    }
}
```

```cpp
int decode_receive_frame_internal(AVCodecContext *avctx, AVFrame *frame) {
	AVCodecInternal *avci = avctx->internal;
    if (avctx->codec->receive_frame) {
    	//从解码器读取一帧解码后的数据
        ret = avctx->codec->receive_frame(avctx, frame);
        if (ret != AVERROR(EAGAIN))
            av_packet_unref(avci->last_pkt_props);
    } else
        ret = decode_simple_receive_frame(avctx, frame);
}
```

##### 3.9、avcodec_receive_frame

从解码器读取一帧解码后的数据

```cpp
int avcodec_receive_frame(AVCodecContext *avctx, AVFrame *frame) {
	AVCodecInternal *avci = avctx->internal;
    //如果还有数据 将缓存数据返回
    if (avci->buffer_frame->buf[0]) {
        av_frame_move_ref(frame, avci->buffer_frame);
    } else {
    	//否则从解码器读取一帧数据
        ret = decode_receive_frame_internal(avctx, frame);
        if (ret < 0)
            return ret;
    }
}
```

##### 3.10、avio_open2
用于根据路径查找文件协议 然后打开
再创建AVIOContext 然后将读写函数指向具体的文件协议


avio_open2会直接调用ffio_open_whitelist

```cpp
int ffio_open_whitelist(AVIOContext **s, const char *filename,) {
	//查找文件协议 并打开文件
	err = ffurl_open_whitelist(&h, filename, );
    //创建AVIOContext 并将具体协议的读写函数指针赋值
    err = ffio_fdopen(s, h);
}
```

```cpp
int ffurl_open_whitelist(URLContext **puc, const char *filename) {
	int ret = ffurl_alloc(puc, filename, flags, int_cb);
    ret = ffurl_connect(*puc, options);
}

int ffurl_alloc(URLContext **puc, const char *filename,) {
	p = url_find_protocol(filename);
    if (p)
    	//根据找到的URLProtocol填充URLContext
       return url_alloc_for_protocol(puc, p, filename, flags, int_cb);
}

//根据文件路径 查找对应的文件协议
struct URLProtocol *url_find_protocol(const char *filename) {
	//获取所有的文件协议 定义在protocol_list.c中
	protocols = ffurl_get_protocols(NULL, NULL);
}

int ffurl_connect(URLContext *uc, AVDictionary **options) {
	//根据具体的协议 调用对应的url_open方法
	err =
        uc->prot->url_open2 ? uc->prot->url_open2() :
        uc->prot->url_open(uc, uc->filename, uc->flags);
}
```

```cpp
int ffio_fdopen(AVIOContext **s, URLContext *h) {
	*s = avio_alloc_context(buffer, buffer_size, h->flags & AVIO_FLAG_WRITE, h,
                            (int (*)(void *, uint8_t *, int))  ffurl_read,
                            (int (*)(void *, uint8_t *, int))  ffurl_write,
                            (int64_t (*)(void *, int64_t, int))ffurl_seek);
}

//创建AVIOContext 并将读写函数指针添加到AVIOContext中
//ffurl_read/ffurl_write即是具体协议的函数指针
AVIOContext *avio_alloc_context(
                  unsigned char *buffer,)
{
    AVIOContext *s = av_malloc(sizeof(AVIOContext));
    ffio_init_context(s, buffer, buffer_size, write_flag, opaque,
                  read_packet, write_packet, seek);
}
```

