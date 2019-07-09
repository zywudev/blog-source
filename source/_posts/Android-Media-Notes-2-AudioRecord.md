---
title: 使用 AudioRecord 采集音频
date: 2019-07-08 14:02:55
tags:
- Android音视频
---

## AudioRecord 详解

首先看一下 Android 官方文档中对 [AudioRecord](https://developer.android.google.cn/reference/android/media/AudioRecord) 的概述：

> AndioRecord 类的主要功能是让各种 Java 应用能够管理音频资源，以便它们通过此类能够录制平台的声音输入硬件所收集的声音。此功能的实现就是通过 "pulling 同步"（reading读取）AudioRecord 对象的声音数据来完成的。在录音过程中，应用所需要做的就是通过后面三个类方法中的一个去及时地获取 AudioRecord 对象的录音数据。 AudioRecord 类提供的三个获取声音数据的方法分别是 read(byte[], int, int), read(short[], int, int), read(ByteBuffer, int)。无论选择使用那一个方法都必须事先设定方便用户的声音数据的存储格式。
> 

简单来说，AudioRecord 类是 Android 系统提供的用于实现录音的功能类。

使用 AudioRecord 采集音频的一般步骤：

- 初始化一个音频缓存大小，该缓存大于等于 AudioRecord 对象用于写声音数据的缓存大小，最小录音缓存可以通过`AudioRecord.getMinBufferSize` 方法得到。
- 构造一个 AudioRecord 对象，需要传入缓冲区大小，如果缓存容量过小，将导致对象构造的失败。
- 开始录音
- 创建一个数据流，一边从 AudioRecord 中读取声音数据到初始化的缓存，一边将缓存中数据导入数据流。
- 关闭数据流
- 停止录音

## 使用 AudioRecord 采集音频

在具体使用 AudioRecord 采集音频之前，简单了解一些音频理论知识。

AudioRecord 采集的音频文件是  `pcm` 格式。

pcm(Pulse Code Modulation) : 脉冲编码调制。模拟音频信号经模数转换（A/D变换）直接形成的二进制序列，pcm 文件没有附加的文件头和文件结束标志。

在模数转换过程中，使用三个参数来表示声音：采样率、声道数和采样位数。

- 采样率：采样频率，指每秒钟采集声音样本的次数。采样频率越高，声音的质量也就越好，声音还原度也就越真实。
- 声道数：即单声道和立体声，相比单声道，立体声更能感觉到空间效果。
- 采样位数：采样值。衡量声音波动变化的一个参数，也就是声卡的分辨率。数值越大，分辨率越高，所发出的声音能力越强。

更为详细的解释可以看这篇文章「[PCM文件格式简介](https://blog.csdn.net/ce123_zhouwei/article/details/9359389)」。接下来介绍如何使用 AudioRecord 采集音频。

1、初始化缓存大小

可以通过`AudioRecord.getMinBufferSize` 方法得到最小录音缓存大小，传入的参数依次是采样频率、声道数和采样位数。

```java
final int minBufferSize = AudioRecord.getMinBufferSize(AUDIO_SAMPLE_RATE_INHZ, AUDIO_CHANNEL, AUDIO_ENCODING);
```

2、构造 AudioRecord 对象

第一个参数表示音频来源。

```java
private AudioRecord mAudioRecord;
mAudioRecord = new AudioRecord(MediaRecorder.AudioSource.MIC, AUDIO_SAMPLE_RATE_INHZ, AUDIO_CHANNEL, AUDIO_ENCODING, minBufferSize);
```

3、初始化一个 buffer 数组

```java
final byte[] data = new byte[minBufferSize];
```

4、开始录音

```java
mAudioRecord.startRecording();
```

5、创建数据流，从 AudioRecord 中读取声音数据到初始化的 buffer 中，从 buffer 中将数据导入到数据流

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        FileOutputStream fos = null;
        try {
            fos = new FileOutputStream(file);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        if (fos != null) {
            while (mIsRecording) {
                int read = mAudioRecord.read(data, 0, minBufferSize);
                if (AudioRecord.ERROR_INVALID_OPERATION != read) {
                    try {
                        fos.write(data);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
            try {
                Log.e(TAG, "关闭");
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}).start();
```

6、停止录音

```java
if (null != mAudioRecord) {
    mAudioRecord.stop();
    mAudioRecord.release();
    mAudioRecord = null;
}
```

## PCM 转 WAV

上面说过，pcm 文件没有附加的文件头和文件结束标志。播放器是无法打开 pcm 文件的 ，所以必须将音频文件进行编码转换。

pcm 转 wav 的实现代码如下：

```java
public class PcmToWavUtil {
    /**
     * 缓存的音频大小
     */
    private int mBufferSize;
    /**
     * 采样率
     */
    private int mSampleRate;
    /**
     * 声道数
     */
    private int mChannel;

    /**
     * @param sampleRate sample rate、采样率
     * @param channel    channel、声道
     * @param encoding   Audio data format、音频格式
     */
    public PcmToWavUtil(int sampleRate, int channel, int encoding) {
        this.mSampleRate = sampleRate;
        this.mChannel = channel;
        this.mBufferSize = AudioRecord.getMinBufferSize(mSampleRate, mChannel, encoding);
    }


    /**
     * pcm文件转wav文件
     *
     * @param inFilename  源文件路径
     * @param outFilename 目标文件路径
     */
    public void pcmToWav(String inFilename, String outFilename) {
        FileInputStream in;
        FileOutputStream out;
        long totalAudioLen;
        long totalDataLen;
        long longSampleRate = mSampleRate;
        int channels = mChannel == AudioFormat.CHANNEL_IN_MONO ? 1 : 2;
        long byteRate = 16 * mSampleRate * channels / 8;
        byte[] data = new byte[mBufferSize];
        try {
            in = new FileInputStream(inFilename);
            out = new FileOutputStream(outFilename);
            totalAudioLen = in.getChannel().size();
            totalDataLen = totalAudioLen + 36;

            writeWaveFileHeader(out, totalAudioLen, totalDataLen,
                    longSampleRate, channels, byteRate);
            while (in.read(data) != -1) {
                out.write(data);
            }
            in.close();
            out.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    /**
     * 加入wav文件头
     */
    private void writeWaveFileHeader(FileOutputStream out, long totalAudioLen,
                                     long totalDataLen, long longSampleRate, int channels, long byteRate)
            throws IOException {
        byte[] header = new byte[44];
        // RIFF/WAVE header
        header[0] = 'R';
        header[1] = 'I';
        header[2] = 'F';
        header[3] = 'F';
        header[4] = (byte) (totalDataLen & 0xff);
        header[5] = (byte) ((totalDataLen >> 8) & 0xff);
        header[6] = (byte) ((totalDataLen >> 16) & 0xff);
        header[7] = (byte) ((totalDataLen >> 24) & 0xff);
        //WAVE
        header[8] = 'W';
        header[9] = 'A';
        header[10] = 'V';
        header[11] = 'E';
        // 'fmt ' chunk
        header[12] = 'f';
        header[13] = 'm';
        header[14] = 't';
        header[15] = ' ';
        // 4 bytes: size of 'fmt ' chunk
        header[16] = 16;
        header[17] = 0;
        header[18] = 0;
        header[19] = 0;
        // format = 1
        header[20] = 1;
        header[21] = 0;
        header[22] = (byte) channels;
        header[23] = 0;
        header[24] = (byte) (longSampleRate & 0xff);
        header[25] = (byte) ((longSampleRate >> 8) & 0xff);
        header[26] = (byte) ((longSampleRate >> 16) & 0xff);
        header[27] = (byte) ((longSampleRate >> 24) & 0xff);
        header[28] = (byte) (byteRate & 0xff);
        header[29] = (byte) ((byteRate >> 8) & 0xff);
        header[30] = (byte) ((byteRate >> 16) & 0xff);
        header[31] = (byte) ((byteRate >> 24) & 0xff);
        // block align
        header[32] = (byte) (2 * 16 / 8);
        header[33] = 0;
        // bits per sample
        header[34] = 16;
        header[35] = 0;
        //data
        header[36] = 'd';
        header[37] = 'a';
        header[38] = 't';
        header[39] = 'a';
        header[40] = (byte) (totalAudioLen & 0xff);
        header[41] = (byte) ((totalAudioLen >> 8) & 0xff);
        header[42] = (byte) ((totalAudioLen >> 16) & 0xff);
        header[43] = (byte) ((totalAudioLen >> 24) & 0xff);
        out.write(header, 0, 44);
    }
}
```

## 源码

[https://github.com/zywudev/AndroidMediaNotes/tree/master/AudioDemo](https://github.com/zywudev/AndroidMediaNotes/tree/master/AudioDemo)









