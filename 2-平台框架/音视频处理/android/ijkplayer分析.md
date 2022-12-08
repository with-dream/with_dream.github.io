
[TOC]

## IJKPlayer源码分析

### 一、概述
ijkplayer主要是对ffmpeg的ffplayer.c进行改造 主要用于移动端
对编解码接口(IJKFF_Pipeline/IJKFF_Pipenode)和显示(SDL_Vout/SDL_Aout)层添加了一层抽象
音频一直使用软解 视频可以根据配置 使用硬解

对于android 主要在IMediaPlayer.createPlayer中封装了ExoMediaPlayer、MediaPlayer、IjkMediaPlayer(默认)， 接口偏向为原生MediaPlayer的接口

jni使用动态注册 封装在ijkplayer_jni.c中

### 二、主要类



### 三、主要api

##### 1、ijkmp_android_create
```cpp
//msg_loop 模仿Handler 用于处理异步消息
IjkMediaPlayer *ijkmp_android_create(int(*msg_loop)(void*)) {
	//初始化IjkMediaPlayer和FFPlayer
    IjkMediaPlayer *mp = ijkmp_create(msg_loop);
	//初始化SDL_Vout
    mp->ffplayer->vout = SDL_VoutAndroid_CreateForAndroidSurface();
	//创建IJKFF_Pipeline 会根据软硬解码 指向不同的函数
    mp->ffplayer->pipeline = ffpipeline_create_from_android(mp->ffplayer);
}
```

```cpp
IJKFF_Pipeline *ffpipeline_create_from_android(FFPlayer *ffp)
{
    IJKFF_Pipeline *pipeline = ffpipeline_alloc(&g_pipeline_class, sizeof(IJKFF_Pipeline_Opaque));

    pipeline->func_open_video_decoder   = func_open_video_decoder;
    pipeline->func_open_audio_output    = func_open_audio_output;
    pipeline->func_init_video_decoder   = func_init_video_decoder;
    pipeline->func_config_video_decoder = func_config_video_decoder;
}

//android和ios有不同的实现
IJKFF_Pipenode *func_open_video_decoder(IJKFF_Pipeline *pipeline, FFPlayer *ffp) {
    IJKFF_Pipeline_Opaque *opaque = pipeline->opaque;
    IJKFF_Pipenode        *node = NULL;
	//优先使用硬解 如果找不到 则使用软解
    if (ffp->mediacodec_all_videos || ffp->mediacodec_avc || ffp->mediacodec_hevc || ffp->mediacodec_mpeg2)
        node = ffpipenode_create_video_decoder_from_android_mediacodec(ffp, pipeline, opaque->weak_vout);
    if (!node) {
        node = ffpipenode_create_video_decoder_from_ffplay(ffp);
    }
}
```
ffpipenode_create_video_decoder_from_android_mediacodec最终调用reconfigure_codec_l 通过jni创建java层的Mediacodec
如果使用软解 则直接使用ffmpeg的

##### 2、video_refresh_thread
主要用于视频的显示刷新

```cpp
//初始化SDL_Vout 在初始化时 ijkmp_android_create调用
SDL_Vout *SDL_VoutAndroid_CreateForANativeWindow()
{
    SDL_Vout *vout = SDL_Vout_CreateInternal(SDL_Vout_Opaque);
    SDL_Vout_Opaque *opaque = vout->opaque;

    opaque->egl = IJK_EGL_create();
    if (!opaque->egl)
        goto fail;

    vout->create_overlay  = func_create_overlay;
    vout->free_l          = func_free_l;
    vout->display_overlay = func_display_overlay;
}
```

video_refresh_thread的刷新线程最终会调用`vout->display_overlay`
最终调用
```cpp
int func_display_overlay_l(SDL_Vout *vout, SDL_VoutOverlay *overlay)
{
    SDL_Vout_Opaque *opaque = vout->opaque;
    ANativeWindow *native_window = opaque->native_window;
    switch(overlay->format) {
   	//硬解的显示是releaseOutputBuffer jni对其进行了封装
    case SDL_FCC__AMC: {
        // only ANativeWindow support
        IJK_EGL_terminate(opaque->egl);
        return SDL_VoutOverlayAMediaCodec_releaseFrame_l(overlay, NULL, true);
    }
    case SDL_FCC_RV24:
    case SDL_FCC_I420:
    case SDL_FCC_I444P10LE: {
        // only GLES support
        if (opaque->egl)
            return IJK_EGL_display(opaque->egl, native_window, overlay);
        break;
    }
    case SDL_FCC_YV12:
    case SDL_FCC_RV16:
    case SDL_FCC_RV32: {
        // both GLES & ANativeWindow support
        if (vout->overlay_format == SDL_FCC__GLES2 && opaque->egl)
            return IJK_EGL_display(opaque->egl, native_window, overlay);
        break;
    }
    //默认使用ANativeWindow 比较高效的方式 Surface为ANativeWindow的具体实现 即子类
    //上层Surface转换为ANativeWindow在SDL_VoutAndroid_SetAndroidSurface
    // fallback to ANativeWindow
    IJK_EGL_terminate(opaque->egl);
    return SDL_Android_NativeWindow_display_l(native_window, overlay); 
}
```
