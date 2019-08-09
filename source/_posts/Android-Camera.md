---
title: 使用 Camera 和 Camera2 采集视频数据
date: 2019-07-15 16:06:01
tags: Android音视频
---

Android 中预览相机画面主要用 SurfaceView 和 TextureView。

SurfaceView：SurfaceView 是一个有自己 Surface 的 View。界面渲染可以放在单独线程而不是主线程中。它更像是一个 Window，自身不能做变形和动画。

TextureView：TextureView 同样也有自己的 Surface。但是它只能在拥有硬件加速层的 Window 中绘制，它更像是一个普通 View，可以做变形和动画。

更多关于 SurfaceView 和 TextureView 的知识可以看这篇文章 [Android 5.0(Lollipop)中的SurfaceTexture，TextureView, SurfaceView和GLSurfaceView](https://blog.csdn.net/jinzhuojun/article/details/44062175)。

Android 5.0 之前系统提供了 Camera API ，5.0 之后提供了 Camera2 API。接下来，我们使用 SurfaceView 和 TextureView 实现相机预览的功能。

## Camera

### 使用 SurfaceView

SurfaceView 用于展示相机画面，SurfaceView 持有 SurfaceHolder，我们通过 SurfaceHolder 中的回调可以知道 Surface 的状态（创建、变化、销毁）。

继承 SurfaceView，实现 SurfaceHolder.CallBack 接口。在 `surfaceCreated` 方法中打开相机预览，在 `surfaceDestroyed` 方法中关闭相机预览就可以了。Camera 的 `open` 方法有些耗时，为了避免阻塞 UI 线程，可以创建子线程打开相机。

```java
public class CameraSurfaceView extends SurfaceView implements SurfaceHolder.Callback {

    private Camera mCamera;

    public CameraSurfaceView(Context context) {
        this(context, null);
    }

    public CameraSurfaceView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CameraSurfaceView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        getHolder().addCallback(this);
    }

    @Override
    public void surfaceCreated(final SurfaceHolder holder) {

        ThreadHelper.getInstance().runOnHandlerThread(new Runnable() {
            @Override
            public void run() {
                openCamera();
                startPreview(holder);
            }
        });
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        ThreadHelper.getInstance().runOnHandlerThread(new Runnable() {
            @Override
            public void run() {
                releaseCamera();
            }
        });
    }

    /**
     * 打开相机
     */
    private void openCamera() {
        int number = Camera.getNumberOfCameras();
        Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
        for (int i = 0; i < number; ++i) {
            Camera.getCameraInfo(i, cameraInfo);
            if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {
                // 打开后置摄像头
                mCamera = Camera.open(i);
                mCamera.setDisplayOrientation(90);
            }
        }
    }

    /**
     * 开始预览
     * @param holder
     */
    private void startPreview(SurfaceHolder holder) {
        if (mCamera != null) {
            mCamera.setPreviewCallback(new Camera.PreviewCallback() {
                @Override
                public void onPreviewFrame(byte[] data, Camera camera) {
                    // 取得 NV21 数据，进一步处理
                }
            });
            try {
                mCamera.setPreviewDisplay(holder);
                mCamera.startPreview();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 关闭相机
     */
    private void releaseCamera() {
        if (mCamera != null) {
            try {
                mCamera.stopPreview();
                mCamera.setPreviewDisplay(null);
                mCamera.release();
            } catch (IOException e) {
                e.printStackTrace();
            }
            mCamera = null;
        }
    }
```

### 使用 TextureView

继承 TextureView，实现`TextureView.SurfaceTextureListener `。在 `onSurfaceTextureAvailable` 方法中打开相机预览，在`onSurfaceTextureDestroyed` 中关闭预览。

```java
public class CameraTextureView extends TextureView implements TextureView.SurfaceTextureListener {

    private Camera mCamera;

    public CameraTextureView(Context context) {
        this(context, null);
    }

    public CameraTextureView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CameraTextureView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        setSurfaceTextureListener(this);
    }

    @Override
    public void onSurfaceTextureAvailable(final SurfaceTexture surface, int width, int height) {
        ThreadHelper.getInstance().runOnHandlerThread(new Runnable() {
            @Override
            public void run() {
                openCamera();
                startPreview(surface);
            }
        });
    }

    @Override
    public void onSurfaceTextureSizeChanged(SurfaceTexture surface, int width, int height) {

    }

    @Override
    public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {
        ThreadHelper.getInstance().runOnHandlerThread(new Runnable() {
            @Override
            public void run() {
                releaseCamera();
            }
        });
        return true;
    }

    @Override
    public void onSurfaceTextureUpdated(SurfaceTexture surface) {

    }

    /**
     * 打开相机
     */
    private void openCamera() {
        int number = Camera.getNumberOfCameras();
        Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
        for (int i = 0; i < number; ++i) {
            Camera.getCameraInfo(i, cameraInfo);
            if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {
                // 打开后置摄像头
                mCamera = Camera.open(i);
                mCamera.setDisplayOrientation(90);
            }
        }
    }

    /**
     * 开始预览
     * @param texture
     */
    private void startPreview(SurfaceTexture texture) {
        if (mCamera != null) {
            mCamera.setPreviewCallback(new Camera.PreviewCallback() {
                @Override
                public void onPreviewFrame(byte[] data, Camera camera) {
                    // 取得 NV21 数据，进一步处理
                }
            });
            try {
                mCamera.setPreviewTexture(texture);
                mCamera.startPreview();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 关闭相机
     */
    private void releaseCamera() {
        if (mCamera != null) {
            try {
                mCamera.stopPreview();
                mCamera.setPreviewDisplay(null);
                mCamera.release();
            } catch (IOException e) {
                e.printStackTrace();
            }
            mCamera = null;
        }
    }
```

## Camera2

### 使用 SurfaceView

```java
public class Camera2SurfaceView extends SurfaceView implements SurfaceHolder.Callback {

    private Context mContext;
    private SurfaceHolder mSurfaceHolder;
    private Handler mWorkHandler;
    private String mCameraId;
    private CameraDevice mCameraDevice;
    private ImageReader mImageReader;
    private CameraCaptureSession mCameraCaptureSession;

    public Camera2SurfaceView(Context context) {
        this(context, null);
    }

    public Camera2SurfaceView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public Camera2SurfaceView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        mContext = getContext();
        mSurfaceHolder = getHolder();
        mSurfaceHolder.addCallback(this);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        HandlerThread handlerThread = new HandlerThread("camera2");
        handlerThread.start();
        mWorkHandler = new Handler(handlerThread.getLooper());
        checkCamera();
        openCamera();
    }

    /**
     * 检测相机
     */
    private void checkCamera() {
        CameraManager cameraManager = (CameraManager) mContext.getSystemService(Context.CAMERA_SERVICE);
        try {
            String[] cameraIdList = cameraManager.getCameraIdList();
            for (String s : cameraIdList) {
                CameraCharacteristics characteristics = cameraManager.getCameraCharacteristics(s);
                Integer lensFacing = characteristics.get(CameraCharacteristics.LENS_FACING);
                Integer sensorOrientation = characteristics.get(CameraCharacteristics.SENSOR_ORIENTATION);
                Integer supportedHardwareLevel = characteristics.get(CameraCharacteristics.INFO_SUPPORTED_HARDWARE_LEVEL);
                if (lensFacing != null && lensFacing == CameraCharacteristics.LENS_FACING_BACK) {
                    mCameraId = s;
                    break;
                }
            }
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }

    }

    /**
     * 打开相机
     */
    private void openCamera() {
        if (ActivityCompat.checkSelfPermission(mContext, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
            return;
        }

        if (mCameraId == null) {
            return;
        }

        CameraManager cameraManager = (CameraManager) mContext.getSystemService(Context.CAMERA_SERVICE);
        try {
            cameraManager.openCamera(mCameraId, new CameraDevice.StateCallback() {
                @Override
                public void onOpened(CameraDevice camera) {
                    mCameraDevice = camera;
                    mImageReader = ImageReader.newInstance(getWidth(), getHeight(), ImageFormat.YUV_420_888, 8);
                    mImageReader.setOnImageAvailableListener(new ImageReader.OnImageAvailableListener() {
                        @Override
                        public void onImageAvailable(ImageReader reader) {
                            Image image = reader.acquireLatestImage();
                            //我们可以将这帧数据转成字节数组，类似于Camera1的PreviewCallback回调的预览帧数据
                            //ByteBuffer buffer = image.getPlanes()[0].getBuffer();
                            //byte[] data = new byte[buffer.remaining()];
                            //buffer.get(data);
                            image.close();
                        }
                    }, mWorkHandler);

                    createCameraPreview();
                }

                @Override
                public void onDisconnected(CameraDevice camera) {
                    camera.close();
                    mCameraDevice = null;
                }

                @Override
                public void onClosed(CameraDevice camera) {
                    super.onClosed(camera);
                }

                @Override
                public void onError(CameraDevice camera, int error) {
                    camera.close();
                    mCameraDevice = null;
                }
            }, mWorkHandler);
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }
    }

    /**
     * 相机预览
     */
    private void createCameraPreview() {
        try {
            final CaptureRequest.Builder captureRequestBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
            Surface surface = mSurfaceHolder.getSurface();
            captureRequestBuilder.addTarget(surface);
            Surface imageReaderSurface = mImageReader.getSurface();
            captureRequestBuilder.addTarget(imageReaderSurface);
            captureRequestBuilder.set(CaptureRequest.CONTROL_MODE, CaptureRequest.CONTROL_MODE_AUTO);
            mCameraDevice.createCaptureSession(Arrays.asList(surface, imageReaderSurface), new CameraCaptureSession.StateCallback() {
                @Override
                public void onConfigured(@NonNull CameraCaptureSession session) {
                    mCameraCaptureSession = session;
                    CaptureRequest captureRequest = captureRequestBuilder.build();
                    try {
                        session.setRepeatingRequest(captureRequest, null, null);
                    } catch (CameraAccessException e) {
                    }
                }

                @Override
                public void onConfigureFailed(@NonNull CameraCaptureSession session) {
                }
            }, mWorkHandler);
        } catch (Exception e) {
        }
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        closeCameraPreview();
        if (mCameraDevice != null) {
            mCameraDevice.close();
        }
        if (mImageReader != null) {
            mImageReader.close();
        }
        mWorkHandler.getLooper().quitSafely();
    }

    private void closeCameraPreview() {
        if (mCameraCaptureSession != null) {
            try {
                mCameraCaptureSession.stopRepeating();
                mCameraCaptureSession.abortCaptures();
                mCameraCaptureSession.close();
            } catch (Exception e) {
            }
            mCameraCaptureSession = null;
        }
    }
```

### 使用 TextureView

TextureView 与 SurfaceView 类似，这里就不贴代码了，具体可以看源码。

## 源码

[https://github.com/zywudev/AndroidMultiMediaLearning](https://github.com/zywudev/AndroidMultiMediaLearning)

以上只是 Camera 和 Camera2 的简单使用，更多细节可以查看官方 API。此外，关于 Android 相机的方向和尺寸适配、Camera 和 Camera2 的兼容选择等问题，后续会单独写文章描述。

## 参考

[https://juejin.im/post/5a33a5106fb9a04525782db5](https://juejin.im/post/5a33a5106fb9a04525782db5)

[https://www.cnblogs.com/renhui/p/7472778.html](https://www.cnblogs.com/renhui/p/7472778.html)