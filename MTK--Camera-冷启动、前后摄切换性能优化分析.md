#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 >一、背景
>二、问题分解
>三、工具分析
>四、 traceView教程
>五、surface create优化
 >六、systrace的教程
>七、优化方案
>八、前后切换速度优化
>九、优化方案
>十、热启动优化
>十一、解决方案：
>十二、驱动优化
### 一、背景

id5a、id6平台我们的相机，对比相同平台红米6a和6的相机，冷热启动，前后摄切换性能要差，对比竞品，我们期望优化的比红米6a和6的相机快10%.

### 二、问题分解

性能优化的核心就是找到影响性能的hotspot,查找hotspot也是有套路的，首先要把相机启动流程分解出来，一个阶段一个阶段分析。

相机的冷启动，可以分解如下几步：

1.AMS启动相机的Activity -> 相机的Activity收到onCreated消息 (可体现系统的性能)。
2.onCreate begin -> onCreate end(主要是ui的加载)
3.openCamera -> CameraOpened
4.TextureView add -> SurfaceTexture created (Surface的创建时间)
5.startPreview -> PreviewStarted
6.setWindow -> PreviewCallback

Log如下：

```
01-01 12:01:57.099   794  3012 I ActivityManager: START u0 {act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10200000 cmp=com.mediatek.camera/.CameraLauncher bnds=[150,976][290,1202] (has extras)} from uid 10064
01-01 12:01:57.169  8261  8261 D CamAp_QuickActivity: [  0.000ms][BEGIN] onCreate
01-01 12:01:57.196  8261  8295 I CamAp_API1-Handler-0: [openCamera]+
01-01 12:01:57.217   532  2586 I mtkcam-dev1: [createSpecificCameraDevice1] dlopen libmtkcam_device1.so +
01-01 12:01:57.224   532  2586 I mtkcam-dev1: [createSpecificCameraDevice1] dlopen libmtkcam_device1.so -
01-01 12:01:57.259   532  2586 I mtkcam-dev1: [createSpecificCameraDevice1] - 0xe6a0ec00
01-01 12:01:57.260   532  2586 I mtkcam-dev1: 0[CameraDevice1Base::open] +
01-01 12:01:57.419   532  2586 I mtkcam-dev1: 0[CameraDevice1Base::open] Add new cameraId 0 - 0xe6a0ec00
01-01 12:01:57.419   532  2586 I mtkcam-dev1: 0[CameraDevice1Base::open] -

01-01 12:01:57.424  8261  8295 I CamAp_API1-Handler-0: [openCamera]-, executing time = 228ms.
01-01 12:01:57.537  8261  8261 D CamAp_QuickActivity: [367.294ms]  [END] onCreate
01-01 12:01:57.542  8261  8261 D CamAp_QuickActivity: [  0.000ms][BEGIN] onStart
01-01 12:01:57.546  8261  8261 D CamAp_QuickActivity: [  3.762ms]  [END] onStart
01-01 12:01:57.549  8261  8261 D CamAp_QuickActivity: [  0.000ms][BEGIN] onResume
01-01 12:01:57.552  8261  8261 D CamAp_QuickActivity: isKeyguardLocked = false
01-01 12:01:57.552  8261  8261 D CamAp_QuickActivity: onResume --> onPermissionResumeTasks()
01-01 12:01:57.600  8261  8261 D CamAp_QuickActivity: [ 50.774ms]  [END] onResume

01-01 12:01:57.685  8261  8294 I CamAp_API1-Handler-0: [setPreviewDisplay]+, pending time = 0ms.
01-01 12:01:57.686  8261  8294 I CamAp_API1-Handler-0: [setPreviewDisplay]-, executing time = 1ms.

01-01 12:01:57.700  8261  8294 I CamAp_API1-Handler-0: [startPreview]+, pending time = 0ms.
01-01 12:01:57.701   532  2586 W mtkcam-dev1: 0[CameraDevice1Base::initDisplayClient] NULL window is passed into...
01-01 12:01:57.701   532  2586 I mtkcam-dev1: 0[CameraDevice1Base::startPreview] +

01-01 12:01:57.720  8261  8261 I CamAp_TextureViewController: updatePreviewSize: new size (960 , 720 ) current size (0 , 0 ),mIsSurfaceCreated = false listener = com.mediatek.camera.common.mode.photo.PhotoMode$SurfaceChangeListener@a79ec89

01-01 12:01:57.735   532  2586 I mtkcam-dev1: 0[CameraDevice1Base::startPreview] - status(0)

01-01 12:01:57.736  8261  8294 I CamAp_API1-Handler-0: [startPreview]-, executing time = 36ms.
01-01 12:01:57.948   532  8363 I MtkCam/DCNode: [0::waitPreviewReady]wait for start preview done +
01-01 12:01:58.087   532  8363 I MtkCam/DCNode: [0::waitPreviewReady]wait for start preview done - , use 130 ms

01-01 12:01:58.053  8261  8261 D CamAp_TextureViewController: onSurfaceTextureAvailable surface  = android.graphics.SurfaceTexture@2d48d1c width 720 height 960

01-01 12:01:58.056  8261  8294 I CamAp_API1-Handler-0: [setPreviewTexture]+, pending time = 0ms.
01-01 12:01:58.087   532  8363 I MtkCam/CamAdapter: (8363)[transitState] StateIdle --> StatePreview
01-01 12:01:58.089   532  2586 I MtkCam/DisplayClient: [setWindow] + window(0xe6a0ecf0), WxH=960x720, count(6), iNativeWindowConsumerUsage(256)

01-01 12:01:58.093  8261  8294 I CamAp_API1-Handler-0: [setPreviewTexture]-, executing time = 37ms.

01-01 12:01:58.279  8261  8295 D CamAp_PhotoDeviceController: [onPreviewFrame] mModeDeviceCallback = com.mediatek.camera.common.mode.photo.PhotoMode@feb2273

```

其中OnCreate begin -> onCreate end:花了367ms

```
01-01 12:01:57.169  8261  8261 D CamAp_QuickActivity: [  0.000ms][BEGIN] onCreate
01-01 12:01:57.537  8261  8261 D CamAp_QuickActivity: [367.294ms]  [END] onCreate

```

其中openCamera: 花了238ms

```

01-01 12:01:57.196  8261  8295 I CamAp_API1-Handler-0: [openCamera]+
01-01 12:01:57.424  8261  8295 I CamAp_API1-Handler-0: [openCamera]-, executing time = 228ms.

```

其中Surface Create: 花了333ms

```
01-01 12:01:57.720  8261  8261 I CamAp_TextureViewController: updatePreviewSize: new size (960 , 720 ) current size (0 , 0 ),mIsSurfaceCreated = false listener = com.mediatek.camera.common.mode.photo.PhotoMode$SurfaceChangeListener@a79ec89
01-01 12:01:58.053  8261  8261 D CamAp_TextureViewController: onSurfaceTextureAvailable surface  = android.graphics.SurfaceTexture@2d48d1c width 720 height 960

```

其中startPreview:花386ms

```
01-01 12:01:57.701   532  2586 I mtkcam-dev1: 0[CameraDevice1Base::startPreview] +
01-01 12:01:58.087   532  8363 I MtkCam/CamAdapter: (8363)[transitState] StateIdle --> StatePreview

```

其中startWindow -> PreveiwCallback:花87ms

```
01-01 12:01:58.093  8261  8294 I CamAp_API1-Handler-0: [setPreviewTexture]-, executing time = 37ms.

01-01 12:01:58.279  8261  8295 D CamAp_PhotoDeviceController: [onPreviewFrame] mModeDeviceCallback = com.mediatek.camera.common.mode.photo.PhotoMode@feb2273

```

其中花时间的步骤，是2、3、4、5,主要ui加载，openCamera，surfaceCreated、startPreview。openCamera和startPreview花时间需要驱动优化。ui加载和surfaceCreate需要上层看。

### 三、工具分析

traceView分析ui加载，发现没有可优化的，现在加载的ui都是需要一进去就要看见的，不需要进去看见的view，都使用viewStub延时加载处理了，并且关于ui也优化了好几波。优化空间比较小。不过针对冷启动，可以使用内存常驻的方案，这样实际上冷启动就是热启动。主要原因是相机在手机启动时，收到启动广播会启动一个服务，用于检测是否温度过高，服务拉起了相机应用。内存常驻后，此应用就不被杀掉了。这样启动速度优化很大。
目前的方案：2G以上项目默认使用内存常驻方案。

###四、 traceView教程

[https://www.jianshu.com/p/f81b20b9a7e2](https://www.jianshu.com/p/f81b20b9a7e2)
[https://developer.android.com/studio/profile/traceview](https://developer.android.com/studio/profile/traceview)

### 五、surface create优化

现在主要的大头就是Surface create优化了。systrace分析创建surface过程，发现ArcFilter HandleThread一直在wait。

#### 六、systrace的教程

[https://developer.android.google.cn/studio/command-line/systrace](https://developer.android.google.cn/studio/command-line/systrace)

查看代码，是滤镜等待arc算法引擎初始化和GL引擎初始化。这里的设计是有争议的，其实相机启动的时候并没有开启滤镜，但是要初始化滤镜的引擎。做耗时benchmark:

```
01-02 01:22:07.991 23458-23554/? I/CamAp_TextureEnvExt: handleGlEnvInit(): Enter
01-02 01:22:07.991 23458-23458/? V/CamAp_TextureEnvExt: getCamSurfaceTexture:: in  -1
01-02 01:22:08.090 23458-23554/? I/CamAp_TextureEnvExt: handleGlEnvInit(): Exit with bRet = EGL_SUCCESS
01-02 01:22:08.091 23458-23554/? I/CamAp_TextureEnvExt: handleArcFilterEngine(): init begin 
01-02 01:22:08.095 23458-23458/? V/CamAp_TextureEnvExt: getCamSurfaceTexture:: ID=1
01-02 01:22:08.211 23458-23554/? I/CamAp_TextureEnvExt: handleArcFilterEngine(): init end engine = 0,prepareEngine=0

```

可以看出：初始化GLEnv需要99ms,初始引擎需要120ms.

### 七、优化方案

这有两种优化方案：

1.  启动时，不加载滤镜引擎，切换到滤镜时加载。后果是切换滤镜时会卡一下，需要修改整个流程，代码修改量大。
2.  分步初始化GLEnv和滤镜引擎，理论上可以优化120ms+,创建Surface和startPreview并行执行。代码改动小，并且能达到优化效果。

目前使用第二种方案，代码提交记录：

[http://192.168.10.10/#/c/213235/](http://192.168.10.10/#/c/213235/)

### 八、前后切换速度优化

前后切换速度对上层来说，主要有两个方面影响，一是翻转动画，一是高斯模糊。翻转动画使用高速相机测量，只有9帧，平台性能限制，有一种带不动的感觉，对标机也没有，和产品沟通去掉。 能优化的就是高斯模糊了。

高斯模糊的耗时benchmark:

```
01-02 02:00:21.636 8089-8089/? I/CamAp_GaussianBlur: GaussianBlur,create render:42
01-02 02:00:21.636 8089-8089/? D/CamAp_GaussianBlur: preview width 750 height 1500
01-02 02:00:21.731 8089-8089/? I/CamAp_GaussianBlur: getPreview bitmap spent time: 125
01-02 02:00:21.821 8089-8089/? I/CamAp_GaussianBlur: blurBitmap,render bitmap:112

```

整个高斯模糊流程大概需要花279ms，主要耗时是getPreview和blurBitmap，还有init每次都要做。

### 九、优化方案

init需要花40多少ms，只做一次。getPreview，对于高斯模糊，我们不需要一张清晰的图片，反正后面也要做模糊。getPreview时，我们设置一个采样率，采样质量降低16倍，做高斯模糊时，我们设置一个较小模糊半径因子，目前为5，这样做模糊的速度快，模糊效果好。

优化后的耗时benchmark:

```
01-02 02:14:15.628 20947-20947/? D/CamAp_GaussianBlur: preview width 750 height 1500
01-02 02:14:15.632 20947-20947/? I/CamAp_GaussianBlur: getPreview bitmap spent time: 4
01-02 02:14:15.636 20947-20947/? I/CamAp_GaussianBlur: blurBitmap,render bitmap:4

```

代码提交记录：

[http://192.168.10.10/#/c/213235/](http://192.168.10.10/#/c/213235/)

### 十、热启动优化

热启动优化，和对比机不同的是，在预览没有出来之前，我们做了一个高斯模糊的盖板。和产品沟通需要去掉，去掉盖板比较容易。引发两个问题需要解决。

1.  TextureView没有被销毁，会保留上一帧。主要表现热启动会闪上一帧。
2.  framework会有一个screenshot界面，同样会闪上帧。

### 十一、解决方案：

1.  在pause时，把TextureView的alpha设置为0,第一帧上来，再把alpha设置1，让其显示。
2.  应用层关掉screenshot.

代码提交记录：

[http://192.168.10.10/#/c/215036/](http://192.168.10.10/#/c/215036/)
[http://192.168.10.10/#/c/214142/](http://192.168.10.10/#/c/214142/)

### 十二、驱动优化

关于openCamera和startPreview优化，底层优化了3a算法、加入了fastae和使用speed mode模式。详情可以咨询彭叶斌同学。

 
原文链接：https://www.jianshu.com/p/77d035588bd4
 


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
