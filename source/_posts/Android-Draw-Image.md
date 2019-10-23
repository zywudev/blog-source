---
title: 使用三种不同的方式绘制图片
date: 2019-07-05 15:07:02
tags:
- Android音视频
---

在 Android 平台上绘制一张图片，使用三种不同的 API，ImageView、SurfaceView、自定义 View。

## ImageView 绘制图片

这种方式较为普遍简单。

```java
 ImageView imageView = findViewById(R.id.image_view);
        Bitmap bitmap = BitmapFactory.decodeFile(new File(FileUtil.getExternalAssetsDir(this), "jaqen.png").getPath());
        imageView.setImageBitmap(bitmap);
```

## SurfaceView 绘制图片

SurfaceView 是 View 的一个子类，特点在于其实现了双缓冲技术，适用于频繁刷新页面的场景。

```java
SurfaceView surfaceView = findViewById(R.id.surface_view);
        surfaceView.getHolder().addCallback(new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {
                if (holder == null) {
                    return;
                }

                Paint paint = new Paint();
                paint.setAntiAlias(true);
                paint.setStyle(Paint.Style.STROKE);
                Bitmap bitmap = BitmapFactory.decodeFile(new File(FileUtil.getExternalAssetsDir(SurfaceViewActivity.this), "jaqen.png").getPath());

                Canvas canvas = holder.lockCanvas();
                canvas.drawBitmap(bitmap, 0, 0, paint);
                holder.unlockCanvasAndPost(canvas);
            }

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {

            }
        });
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
        mBitmap = BitmapFactory.decodeFile(new File(FileUtil.getExternalAssetsDir(getContext()), "jaqen.png").getPath());

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

## 源码

[https://github.com/zywudev/AndroidMultiMediaLearning](https://github.com/zywudev/AndroidMultiMediaLearning)