---
title: 使用 AudioTrack 播放 PCM 音频
date: 2019-07-08 15:31:02
tags:
- Android音视频
---

Android 中播放声音可以用 MediaPlayer 和 AudioTrack，两者都提供了 Java API 供应用开发者使用。

虽然都可以播放声音，但两者还是有很大的区别的。

其中最大的区别是 MediaPlayer 可以播放多种格式的声音文件，例如 MP3，AAC，WAV，OGG，MIDI 等。MediaPlayer会在 framework 层创建对应的音频解码器，而 AudioTrack 只能播放已经解码的 PCM 流，如果对比支持的文件格式的话则是 AudioTrack 只支持 wav 格式的音频文件，因为 wav 格式的音频文件大部分都是 PCM流。AudioTrack 不创建解码器，所以只能播放不需要解码的wav文件。

MediaPlayer在 framework 层还是会创建 AudioTrack，把解码后的 PCM 数流传递给 AudioTrack，AudioTrack再传递给 AudioFlinger 进行混音，然后才传递给硬件播放,所以是 MediaPlayer 包含了 AudioTrack。

AudioTrack 有两种数据加载模式（MODE_STREAM 和 MODE_STATIC）， 对应着两种完全不同的使用场景。

**MODE_STREAM**：在这种模式下，通过 write 一次次把音频数据写到 AudioTrack 中。这和平时通过 write 调用往文件中写数据类似，但这种方式每次都需要把数据从用户提供的 Buffer 中拷贝到 AudioTrack 内部的 Buffer 中，在一定程度上会引起延时。为解决这一问题，AudioTrack 就引入了第二种模式。

**MODE_STATIC**：在这种模式下，只需要在 play 之前通过一次 write 调用，把所有数据传递到 AudioTrack 中的内部缓冲区，后续就不必再传递数据了。这种模式适用于像铃声这种内存占用较小、延时要求较高的文件。但它也有一个缺点，就是一次 write 的数据不能太多，否则系统无法分配足够的内存来存储全部数据。

在 AudioTrack 构造函数中，会接触到 AudioManager.STREAM_MUSIC 这个参数。它的含义与 Android 系统对音频流的管理和分类有关。Android 将系统的声音分为好几种流类型，下面是几个常见的：

- STREAM_ALARM：警告声
- STREAM_MUSIC：音乐声，例如 music 等
- STREAM_RING：铃声
- STREAM_SYSTEM：系统声音，例如低电提示音，锁屏音等
- STREAM_VOICE_CALL：通话声

**播放流程：**

1\. 创建 AudioTrack 播放对象，参数与创建 AudioRecord 有相似之处。

```java
int channelConfig = AudioFormat.CHANNEL_OUT_MONO;
final int minBufferSize = AudioTrack.getMinBufferSize(AUDIO_SAMPLE_RATE_INHZ, channelConfig, AUDIO_ENCODING);
mAudioTrack = new AudioTrack(
    new AudioAttributes.Builder()
    .setUsage(AudioAttributes.USAGE_MEDIA)
    .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
    .build(),
    new AudioFormat.Builder()
    .setSampleRate(AUDIO_SAMPLE_RATE_INHZ)
    .setEncoding(AUDIO_ENCODING)
    .setChannelMask(channelConfig)
    .build(),
    minBufferSize,
    AudioTrack.MODE_STREAM,
    AudioManager.AUDIO_SESSION_ID_GENERATE);
```

2\. 开始播放

- MODE_STREAM 模式

```java
mAudioTrack.play();

File file = new File(getExternalFilesDir(Environment.DIRECTORY_MUSIC), "test.pcm");
try {
    mFileInputStream = new FileInputStream(file);
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            byte[] tempBuffer = new byte[minBufferSize];
            while (mFileInputStream.available() > 0) {
                int readCount = mFileInputStream.read(tempBuffer);
                if (readCount == AudioTrack.ERROR_INVALID_OPERATION ||
                    readCount == AudioTrack.ERROR_BAD_VALUE) {
                    continue;
                }
                if (readCount != 0 && readCount != -1) {
                    mAudioTrack.write(tempBuffer, 0, readCount);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}).start();
```

- MODE_STATIC 模式

```java
 try {
     InputStream in = getResources().openRawResource(R.raw.ding);
     try {
         ByteArrayOutputStream out = new ByteArrayOutputStream();
         for (int b; (b = in.read()) != -1; ) {
             out.write(b);
         }
         mAudioData = out.toByteArray();
     } finally {
         in.close();
     }
 } catch (IOException e) {
     Log.wtf(TAG, "Failed to read", e);
 }

mAudioTrack.write(mAudioData, 0, mAudioData.length);
mAudioTrack.play();
```

3\. 停止播放

```java
if (mAudioTrack != null) {
    mAudioTrack.stop();
    mAudioTrack.release();
    mAudioTrack = null;
}
```

## 源码

[https://github.com/zywudev/AndroidMultiMediaLearning](https://github.com/zywudev/AndroidMultiMediaLearning)

