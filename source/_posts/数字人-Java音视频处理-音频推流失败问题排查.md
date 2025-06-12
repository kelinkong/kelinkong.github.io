---
title: 数字人-Java音视频处理-音频推流失败问题排查
date: 2025-06-05 10:16:11
categories: [Digital Human]
---

## 需求
1. 从brtc房间拉流数字人的音视频
2. 将音视频推流到trtc房间
3. 从trtc房间获取用户音频数据
4. 将音频数据推流到brtc房间

## 问题
在将音频数据推流到brtc房间时，遇到了音频推流失败的问题。

## 排查步骤

1、音频格式：经过多次确认，音频格式均为16kHz，单声道，16bit。

2、用户登陆brtc房间参数和百度提供的demo保持一致

这里登陆brtc房间有个坑，卡了两天，最后发现是websocket的线程和底层jni异步调用登陆的线程冲突了，导致登陆失败。后启动新线程去执行登陆操作，问题解决。

3、音频数据

从trtc拉下来的音频流使用brtc的demo能推成功，音频格式没问题。

4、排查网络问题

把demo放到cicd的容器中运行，能推流成功。

5、排查代码问题

使用多种方式推流，均推不上去。

6、怀疑和trtc的sdk不兼容

把使用到trtc的sdk的代码注释掉，能推流成功。

## 原因
经过排查，问题的根本原因是 **`.so`（native shared library）加载的顺序冲突**。

这在 Android/Java 中是非常常见的问题，特别是当两个 SDK 都依赖于底层音视频处理库（如 FFmpeg、libyuv、libaudio、OpenSL ES 等）时：

## 来自大模型的解答

### 为什么加载顺序会影响音频推流？

多个 SDK 引入相同名字但版本不同的 `.so` 库（如 `libavcodec.so`、`libaudio.so` 等）时：

* JVM 在 `System.loadLibrary` 第一次加载某个 so 后，会缓存该 native 映射；
* 后续加载同名 so 的 SDK 无法覆盖已加载的符号表；
* 结果导致某些 native 方法绑定错误，或者 native 初始化逻辑出错；
* 表现为推流失败、音频失真、甚至崩溃。

是的，**`.so`库的加载顺序**确实会影响音频推流，尤其是你当前这种使用多个音视频 SDK（如腾讯 + 百度）的场景。这类问题本质是 **多个 SDK 引用了相同名字的 `.so` 或底层 C/C++ 库，并存在符号冲突或初始化覆盖**。

---

### ✅ **问题原因**

当你用 JNI 加载多个 SDK（例如腾讯 SDK 和百度 SDK）时：

* 两者可能都使用了 **相同的第三方库**（如 `libavcodec.so`、`libyuv.so`、`libaudio.so`）；
* JVM 加载 `.so` 是**按顺序且一次性的**，后加载的 `.so` 无法覆盖已加载的；
* 如果两个库中有相同的函数名、变量名或全局符号，**先加载的 `.so` 优先生效**，后加载的可能会失效或行为异常；
* 导致你使用百度 SDK 推流时，native 层没有绑定到它自己的实现，而是用了腾讯 SDK 的符号，结果推流失败或异常。

---

### ✅ **你的验证结果说明了什么？**

> ✅ 你说“改变顺序，**先加载百度的 so 库就可以解决推流的问题**”，这说明是**库冲突 + 加载顺序导致 native 方法绑定错误**。

---

### ✅ **解决方案**

#### 1. **强制控制加载顺序（推荐）**

在 Java 中，使用 `System.loadLibrary()` 明确指定加载顺序：

```java
static {
    // 先加载百度的 so，确保百度的 native 方法先绑定
    System.loadLibrary("baidu_audio_core");
    System.loadLibrary("baidu_audio_jni");

    // 再加载腾讯的 so
    System.loadLibrary("tx_audio_core");
    System.loadLibrary("tx_audio_jni");
}
```

> **放在同一个类的 static 块中或初始化时调用，确保顺序生效。**

---

#### 2. **避免加载重复库**

* 如果你自己也打了 JNI 的 `.so` 包，确保名称唯一，比如用前缀 `yourapp_***.so`；
* 可以用 `nm` 或 `readelf` 工具检查两个 SDK 是否有相同符号：

  ```bash
  nm -D libbaidu_audio.so | grep Init
  nm -D libtx_audio.so | grep Init
  ```

---

#### 3. **手动使用 `dlopen()`（高级方案）**

如果你在 JNI 层调用了百度和腾讯的 `.so`，可以通过 `dlopen(path, RTLD_NOW)` 在 native 层动态加载并隔离符号空间（部分平台支持）。

---

#### 4. **使用不同进程隔离（可选）**

如两个 SDK 冲突过于严重，甚至可以把两套逻辑放在两个 Android `Service` 的子进程中运行，通过 Binder 通信。这会带来一定复杂度，但是彻底隔离 native 的终极方案。

## 改变加载顺序后，对trtc有影响

继续排查，使用trtc的线程池去执行往brtc房间推流的操作，会导致推流失败。
这说明 trtc 的线程池和 brtc 的推流操作存在冲突。

解决方案：启动新线程去执行推流的操作。