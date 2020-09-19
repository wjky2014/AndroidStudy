#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一. MtkCam3的代码结构及学习资料
>二. MtkCam3设计架构概览
>三. MtkCam3代码跟读
>四. MtkCam3 Debug


## 一.MtkCam3的代码结构及学习资料

mtk online里搜Camera 可以搜到很全面的Mtk Hal3的学习文档,Mtk整理的文档很棒,简单到位!
https://online.mediatek.com/QuickStart/2a17666a-9d46-4686-9222-610ec0f087cc

下述的代码结构只是列出了mtk平台的camera路径，Android Camera相关路径并未记录

**APP**
MTK Camera
```
vendor/mediatek/proprietary/packages/apps/
```

**HAL**
MTK Camera Hal,目前最新Android Camera Api2下用的都是HAL3的内存,HAL3主要代码在mtkcam3中,有些工具类复用了mtkcam中
```
vendor/mediatek/proprietary/hardware/mtkcam/
vendor/mediatek/proprietary/hardware/mtkcam3/
```
**以下是和camera强相关**
```
vendor/mediatek/proprietary/hardware/jpeg/
vendor/mediatek/proprietary/hardware/bwc/
vendor/mediatek/proprietary/hardware/m4u/
```
**Kernel**
```
kernel-x.xx/drivers/misc/mediatek/imgsensor/
```
## 二.MtkCam3设计架构概览

![ ](https://upload-images.jianshu.io/upload_images/2118860-c4d9bd06711a6092?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.  MtkCam3实现了Android定义的几个HAL3的接口:ICameraProvider, ICameraDevice, ICameraDeviceSession, ICameraDeviceCallback;ICameraProvider 的实现类CameraProviderImpl包在 camera device manager 外围，只是一个 adapter, 适配不同版本的 camera device interface。 Camera Service(指的是camera android层的进程: cameraserver ) 可以通过 ICameraProvider 去拿到 ICameraDevice 。ICameraDevice 和 ICameraDeviceSession 的实现类 CameraDevice3Impl, CameraDevice3SessionImpl 。用于Camera Service 去操作每一个 camera。 比如: open, close, configureStreams, processCaptureRequest 。

2.  AppStreamManager位于framework与pipeline之间，主要职责有如下三条：
-  Callback result to Android framework according to the returning rules which are defined in camera3.h
- Update vendor defined gralloc usage
- Android/ MTK streamInfo conversion

3.  IPipelineModel的角色
    在open/close stage，Power on/off sensor；在config stage，根据APP的createCaptureSession里面带下来的surface list，推测Output以及按照Topological推测Pipeline各个Node是否需要创建以及各个Node的I/O buffer，建立整条PipelineModel；在Request Stage，接到上层queue下来的request，转化为Pipleline统一的IPipelineFrame，决定这个request的I/O buffer、Topological、sub frame、dummy frame、feature set等信息；

4.  HWNode是大Node,三方算法的挂载在这些node里面,作为小node.
    P1Node负责输出raw图,P2CaptureNode主要负责拍照的frame的处理，P2StreamingNode主要负责录像预览的数据处理,JpegNode的输入时main YUV、Thumbnail YUV及metadata，输出是Jpeg及App metadata。

## 三. MtkCam3代码跟读

### 3.1 Camera HAL3 init

![ ](https://upload-images.jianshu.io/upload_images/2118860-45655d5b51651616?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.2 OpenCamera

![ ](https://upload-images.jianshu.io/upload_images/2118860-18921a9a509cefc9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.3 ConfigureStream

![ ](https://upload-images.jianshu.io/upload_images/2118860-94b1c1f787352b10?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.4 Request

![ ](https://upload-images.jianshu.io/upload_images/2118860-c6dbd3fbe6e75f6f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 四. MtkCam3 Debug

#####1.Mtk日志开关
**`设置log level,cameraHalserver重启生效`**

persist.vendor.mtk.camera.log_level 控制代码如下:

```
#define CAM_ULOGMD(fmt, arg...)        ALOGD(fmt, ##arg)

mtkcam/include/mtkcam/utils/std/Log.h
#define CAM_LOGD(fmt, arg...)   do{ if(0!=mtkcam_testLog(LOG_TAG, 'D')) ALOGD(fmt, ##arg); } while(0)

mtkcam/utils/std/Misc.cpp
static int32_t determinePersistLogLevel()
{
    int32_t level = ::property_get_int32("persist.vendor.mtk.camera.log_level", -1);
    CAM_ULOGMD("###### get camera log property =%d", level);
    if  (-1 == level) {
        level = MTKCAM_LOG_LEVEL_DEFAULT;
    }
    return level;
}

__BEGIN_DECLS
static int32_t gLogLevel = determinePersistLogLevel();
int mtkcam_testLog(char const* /*tag*/, int prio)
{
    switch (prio)
    {
        case 'V':       return (gLogLevel>=4);
        case 'D':       return (gLogLevel>=3);
        case 'I':       return (gLogLevel>=2);
        case 'W':       return (gLogLevel>=1);
        case 'E':       return (1);
        default:        break;
    }
    return 0;
}
```
原文链接：https://blog.csdn.net/TaylorPotter/article/details/105707181
 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
