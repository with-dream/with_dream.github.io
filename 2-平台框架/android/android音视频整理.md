[TOC]

# android 多媒体

### 一、音频
#### 1、参数
#### 2、AudioRecord

https://www.cnblogs.com/renhui/p/7457321.html

#### 3、AudioTrack
只支持pcm和wav
MODE_STREAM/MODE_STATIC 模式

https://www.cnblogs.com/renhui/p/7463287.html

### 二、视频
#### 1、MediaRecorder
录制音视频
https://www.jianshu.com/p/61c42fa3a055
https://www.jianshu.com/p/9f70197f79a8
https://blog.csdn.net/zeqiao/article/details/84303114


### 三、编解码
#### 1、MediaExtractor
把音频和视频的数据进行分离
advance()：读取下一帧数据
https://www.cnblogs.com/renhui/p/7474096.html

#### 2、MediaMuxer
生成音视频
```java
int audioTrackIndex = muxer.addTrack(audioFormat);
int videoTrackIndex = muxer.addTrack(videoFormat);
//currentTrackIndex为音视频的trackIndex
muxer.writeSampleData(currentTrackIndex, inputBuffer, bufferInfo);
```
https://www.cnblogs.com/renhui/p/7474096.html
#### 1、MediaExtractor

### 四、其他
#### 1、同步
MediaSync

https://www.cnblogs.com/renhui/p/10751755.html
