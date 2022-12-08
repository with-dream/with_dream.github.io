# SDL总结

## 一、概述
官网:http://www.libsdl.org/

sdl有八个子系统:音频、CDROM、事件处理、文件I/O、游戏杆、线程、记时器和视频
```cpp
#define SDL_INIT_TIMER          0x00000001u  // 定时器
#define SDL_INIT_AUDIO          0x00000010u  // 音频
#define SDL_INIT_VIDEO          0x00000020u  // 视频
#define SDL_INIT_JOYSTICK       0x00000200u  // 摇杆
#define SDL_INIT_HAPTIC         0x00001000u  // 触摸屏
#define SDL_INIT_GAMECONTROLLER 0x00002000u  // 游戏控制器
#define SDL_INIT_EVENTS         0x00004000u  // 事件
```

#### 一、基础
##### 1、初始化
sdl使用前 必须调用SDL_Init或SDL_InitSubSystem初始化
默认会初始化视频和计时器

SDL_Quit()用于关闭所有子系统，在程序退出前调用

#### 二、视频
视频是SDL最完整的子系统，而且可以配合opengl使用

使用getpixel()/putpixel()可以直接对像素进行擦欧洲哦

常用的API

| api | 说明 |
|--------|--------|
|    SDL_CreateWindow    |    创建窗口    |
|    SDL_GetWindowSize    |    获取窗口尺寸    |
|    SDL_DestroyWindow    |    销毁窗口    |
|    SDL_CreateRenderer    |    创建一个渲染器 绘制上下文   |
|    SDL_DestroyRenderer    |    销毁渲染器   |
|    SDL_CreateTexture    |    创建一个纹理 相当于画布 需要图像格式    |
|    SDL_UpdateTexture    |    更新Texture中Rect中的数据    |
|    SDL_LockTexture/SDL_UnlockTexture    |    像素只读模式 可以修改像素值    |
|    SDL_RenderCopy    |    将纹理复制到渲染器 纹理要使用相应的Render创建   |
|    SDL_RenderPresent    |    将纹理渲染到window上    |
|    SDL_RenderFillRect    |     配合SDL_SetRenderDrawColor使用 填充矩形   |
|    SDL_SetTextureBlendMode    |    texture的混合模式 配合SDL_RenderCopy使用    |
|        |        |

#### 三、音频
| api | 说明 |
|--------|--------|
|    SDL_OpenAudio    |     打开一个音频设备 需要音频数据填充回调   |
|    SDL_PauseAudio    |    使能播放    |
|    SDL_MixAudio    |    混音 使用SDL_MixAudioFormat替代    |
|        |        |

#### 四、输入事件处理
##### 1、键盘事件
SDL_PollEvent()从消息队列里读取，并用switch-case检测SDL_KEYUP和SDL_KEYDOWN

| api | 说明 |
|--------|--------|
|    SDL_PushEvent    |    将一个事件放入队列    |
|    SDL_PumpEvents    |    收集输入事件，放入循环中    |
|    SDL_WaitEvent    |    等到一个可用的事件    |
|    SDL_PollEvent    |    轮询事件 非阻塞    |
|    SDL_PeepEvents    |    增删遍历事件    |
|        |        |




参考:
https://www.cnblogs.com/lidabo/p/3758465.html