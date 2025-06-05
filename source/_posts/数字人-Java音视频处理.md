---
title: 数字人-Java音视频处理-推流和拉流
date: 2025-05-21 15:17:08
categories: [Digital Human]
---

# Java处理音视频流-推流和拉流
Java本身不支持音视频处理，但可以通过第三方库来实现。

一般音视频流处理流程如下：

- 读取流数据：通过网络（如 RTMP、HTTP）或本地文件读取音视频流。
- 解码：将压缩的音视频数据解码为原始帧（如 PCM、YUV）。
- 处理/分析：对帧数据进行处理，如转码、剪辑、特效、识别等。
- 编码：将处理后的帧重新编码为目标格式。
- 输出/推流：保存到文件或通过网络推送到服务器

## 什么是RTMP协议？
RTMP（Real-Time Messaging Protocol）是一种用于音视频流传输的协议，最初由Adobe开发。它主要用于Flash Player与服务器之间的实时音视频传输。

RTMP协议支持低延迟的音视频流传输，适用于直播、视频会议等场景。

RTMP协议的特点包括：
- 低延迟：适合直播等实时性要求高的场景。
- 支持音视频同步传输：可以同时传输音频、视频和数据。
- 广泛应用：被许多流媒体服务器（如 Nginx-RTMP、Wowza、Red5）和平台（如斗鱼、B站、抖音等）支持。

典型推流地址格式：
```bash
rtmp://服务器地址/应用名/流名
```

Java使用示例：

```java
import org.bytedeco.javacv.FFmpegFrameGrabber;
import org.bytedeco.javacv.FFmpegFrameRecorder;
import org.bytedeco.javacv.Frame;

public class RtmpPushExample {
    public static void main(String[] args) throws Exception {
        // RTMP 拉流地址
        String rtmpUrl = "rtmp://服务器地址/应用名/流名";
        FFmpegFrameGrabber grabber = new FFmpegFrameGrabber(rtmpUrl);
        grabber.setOption("rtmp_buffer", "1000"); // 设置缓冲区大小
        grabber.setFormat("flv"); // 设置输入格式
        grabber.start(); // 开始抓取：初始化解码器并打开输入流（比如 RTMP 网络流、本地文件等），为后续逐帧读取音视频数据做准备。

        // 目标采样率
        int targetSampleRate = 44100;

        // 推流到RTMP服务器
        FFmpegFrameRecorder recorder = new FFmpegFrameRecorder(
            "rtmp://服务器地址/应用名/流名",
            grabber.getImageWidth(),
            grabber.getImageHeight(),
            grabber.getAudioChannels()
        );
        recorder.setFormat("flv");
        recorder.setSampleRate(targetSampleRate); // 设置目标采样率
        recorder.start(); 

        // 音频重采样器
        AudioResampler resampler = null;
        if (grabber.getSampleRate() != targetSampleRate) {
            resampler = AudioResampler.create(
                targetSampleRate, grabber.getSampleRate(),
                grabber.getAudioChannels(), grabber.getAudioChannels(),
                avutil.AV_SAMPLE_FMT_S16, grabber.getSampleFormat()
            );
        }

        // grabber.grab()方法用于逐帧读取音视频数据，返回一个Frame对象。Frame对象包含了音视频的原始数据和相关信息。
        Frame frame;
        while ((frame = grabber.grab()) != null) {
            if (frame.samples != null && resampler != null) {
                // 对音频帧进行采样率转换
                Frame resampledFrame = new Frame();
                resampledFrame.sampleRate = targetSampleRate;
                resampledFrame.audioChannels = frame.audioChannels;
                resampledFrame.samples = resampler.resample(frame.samples);
                recorder.record(resampledFrame);
            } else {
                recorder.record(frame);
            }
        }

        recorder.stop();
        grabber.stop();
    }
}
```

`FFmpegFrameGrabber`是 JavaCV（基于 FFmpeg 的 Java 封装库）中的一个类，用于抓取（读取）音视频流的数据帧。它可以从本地文件、网络流（如 RTMP、HTTP、RTSP）等多种来源读取音视频内容，并将其解码为可处理的帧（Frame）对象。

`FFmpegFrameRecorder`是 JavaCV 中用于录制（写入）音视频流的类。它可以将处理后的帧数据编码为指定格式，并输出到文件或网络流中。

`Frame`是 JavaCV 中的一个类，表示音视频的单帧数据。它包含了音频样本、视频图像等信息。

**FFmpegFrameGrabber 和 FFmpegFrameRecorder 的典型用法就是一帧帧地处理。**

具体流程如下：

- FFmpegFrameGrabber 负责从流或文件中逐帧读取音视频数据（每次调用 grab() 返回一帧 Frame）。
- 对每一帧进行处理（如重采样、转码、加特效等）。
- 处理完一帧后，立即用 FFmpegFrameRecorder 推流或写出（每次调用 record(frame) - 送一帧到目标服务器或文件）。

## 从Frame中读取原始数据

`Frame`类中包含了音频和视频的原始数据，具体字段如下：
- `samples`：音频样本数据，通常是一个二维数组，表示音频的 PCM 数据。
- `image`：视频图像数据，通常是一个三维数组，表示视频图像的像素数据。
- `timestamp`：时间戳，表示该帧的时间位置。
- `imageWidth`：视频图像的宽度。
- `imageHeight`：视频图像的高度。
- `audioChannels`：音频通道数。
- `sampleRate`：音频采样率。
- `sampleFormat`：音频样本格式。
- `audioData`：音频数据，通常为一个一维数组，表示音频的采样数据。
- `audioTimestamp`：音频时间戳，表示该帧的时间位置。
- `videoData`：视频数据，通常为一个一维数组，表示视频的像素数据。
- `videoTimestamp`：视频时间戳，表示该帧的时间位置。

读取音频数据并处理：
```java
if (frame.samples != null) {
    Buffer[] samples = frame.samples;

    // 例如：假设是 ShortBuffer
    ShortBuffer audioBuffer = (ShortBuffer) samples[0];
    short[] audioData = new short[audioBuffer.remaining()];
    audioBuffer.get(audioData); // 复制数据到数组
    myFrame.setAudioData(audioData); // 填充到自定义的结构
}
```

读取视频数据并处理：
```java
if (frame.image != null) {
    Buffer[] images = frame.image;
    // 例如：假设是 ByteBuffer
    ByteBuffer imageBuffer = (ByteBuffer) images[0];
    byte[] imageData = new byte[imageBuffer.remaining()];
    imageBuffer.get(imageData); // 复制数据到数组
    myFrame.setImageData(imageData); // 填充到自定义的结构
}
```

Buffer 是桥梁，用于在 JavaCV 的 Frame 和你的自定义帧结构之间传递原始数据。

如何判断音频和视频数据的类型？
- `frame.samples`：如果不为 null，则表示音频数据存在。
- `frame.image`：如果不为 null，则表示视频数据存在。
- `frame.imageType`：可以通过该字段判断视频数据的类型，如 RGB、YUV 等。
- `frame.sampleFormat`：可以通过该字段判断音频数据的格式，如 PCM、AAC 等。

```java
int format = frame.sampleFormat;
if (format == avutil.AV_SAMPLE_FMT_S16) {
    // 16位整型，samples[0] 是 ShortBuffer
} else if (format == avutil.AV_SAMPLE_FMT_FLT) {
    // 32位浮点型，samples[0] 是 FloatBuffer
} else if (format == avutil.AV_SAMPLE_FMT_S32) {
    // 32位整型，samples[0] 是 IntBuffer
}
// 还可以根据 avutil 的其他常量判断更多格式
```

## 如何理解Java中的Buffer
Buffer 是 Java 中用于处理原始数据的类，通常用于音频、视频等多媒体数据的存储和传输。

Buffer 提供了一种高效的方式来处理原始数据，避免了频繁的内存分配和复制。

Buffer 是一个抽象类，具体实现有 ByteBuffer、ShortBuffer、IntBuffer、
FloatBuffer 等。每种 Buffer 都有自己的数据类型和操作方法。

Buffer 的基本操作包括：
- `put()`：将数据写入 Buffer。
- `get()`：从 Buffer 中读取数据。
- `flip()`：将 Buffer 的读写模式切换。
- `clear()`：清空 Buffer。
- `rewind()`：重置 Buffer 的读写位置。
- `remaining()`：获取 Buffer 中剩余可读数据的大小。
- `position()`：获取当前读写位置。
- `limit()`：获取 Buffer 的限制位置。
- `capacity()`：获取 Buffer 的总容量。
- `hasRemaining()`：判断 Buffer 中是否还有可读数据。

`flip()` 方法是 Java NIO Buffer 中非常重要的一个操作，用于切换 Buffer 的读写模式。

当向 Buffer 写入数据时（比如 put 操作），Buffer 处于“写模式”，position 指向下一个要写入的位置。

写完数据后，如果想从 Buffer 读取刚才写入的数据，需要先调用 flip()。

`flip()` 会把 position 设为 0，把 limit 设为原来的 position，这样就可以从头开始读取刚才写入的数据。

```java
ByteBuffer buffer = ByteBuffer.allocate(10);
buffer.put((byte)1);
buffer.put((byte)2);
// 此时 position=2, limit=10

buffer.flip(); // 切换到读模式
// 现在 position=0, limit=2

byte a = buffer.get(); // 读取第一个字节
byte b = buffer.get(); // 读取第二个字节
```