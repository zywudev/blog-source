---
title: Android 音视频学习：使用三种不同的方式绘制图片
date: 2020-04-24 08:07:02
tags: Android 音视频
categories: Android 音视频
---

本系列是个人 Android 音视频学习总结，这是第一篇，主要学习内容是：

在 Android 平台上绘制一张图片，使用三种不同的 API，ImageView、SurfaceView、自定义 View。

## ImageView 绘制图片

这种方式较为普遍简单。

```java
public class ImageViewActivity extends BaseActivity {

    private ImageView mImageView;

    @Override
    protected View getContentView() {
        mImageView = new ImageView(this);
        return mImageView;
    }

    @Override
    protected int getTitleResId() {
        return R.string.image_view;
    }

    @Override
    protected void initView() {
        super.initView();
        // 绘制图片
        mImageView.setImageBitmap(FileUtil.getDrawImageBitmap(this));
    }
}
```

## SurfaceView 绘制图片

SurfaceView 是 View 的一个子类，特点在于其实现了双缓冲技术，适用于频繁刷新页面的场景。

```java
public class SurfaceViewActivity extends BaseActivity {

    private SurfaceView mSurfaceView;

    @Override
    protected int getTitleResId() {
        return R.string.surface_view;
    }

    @Override
    protected View getContentView() {
        mSurfaceView = new SurfaceView(this);
        return mSurfaceView;
    }

    @Override
    protected void initView() {
        super.initView();
        mSurfaceView.getHolder().addCallback(new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {
                if (holder == null) {
                    return;
                }

                Paint paint = new Paint();
                paint.setAntiAlias(true);
                paint.setStyle(Paint.Style.STROKE);

                Canvas canvas = holder.lockCanvas();
                // 绘制图片
                canvas.drawBitmap(FileUtil.getDrawImageBitmap(SurfaceViewActivity.this), 0, 0, paint);
                holder.unlockCanvasAndPost(canvas);
            }

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {

            }
        });
    }
}
```

## 自定义 View 绘制图片

还可以通过自定义 View 绘制图片。

```java
public class CustomView extends View {

    private Paint mPaint;
    private Bitmap mBitmap;

    public CustomView(Context context) {
        this(context, null);
    }

    public CustomView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CustomView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
        mPaint.setStyle(Paint.Style.STROKE);
        mBitmap = FileUtil.getDrawImageBitmap(getContext());

    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (mBitmap != null) {
            canvas.drawBitmap(mBitmap, 0, 0, mPaint);
        }
    }
}
```

具体源码看这里：[AndroidMultiMediaLearning](https://github.com/zywudev/AndroidMultiMediaLearning)，下一篇总结使用 AudioRecord 和 AudioTrack API 完成音频 PCM 数据的采集和播放，并实现读写音频 wav 文件。