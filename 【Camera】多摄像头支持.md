#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 >一、多Camera 概述
>二、多摄像头举例
>三、多摄像头支持列表
>四、Camera 流配置
>五、多摄像头客制化


# 一、多Camera 概述

Android 9通过一个新的逻辑相机设备引入了对多相机设备的API支持，该逻辑相机设备由指向同一方向的两个或多个物理相机设备组成。逻辑摄像机设备作为单个CameraDevice / CaptureSession公开给应用程序，允许与HAL集成的多摄像机功能进行交互。应用程序可以选择访问和控制基础物理相机流，元数据和控件。

![图1。多相机支持](https://upload-images.jianshu.io/upload_images/5851256-f381569f259200b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在此图中，不同的摄像机ID用颜色编码。该应用程序可以同时从每个物理相机流式传输原始缓冲区。也可以设置单独的控件并从不同的物理摄像机接收单独的元数据。

# 二、多摄像头举例
多摄像机设备必须通过 [逻辑多摄像机功能发布](https://developer.android.google.cn/reference/android/hardware/camera2/CameraMetadata#REQUEST_AVAILABLE_CAPABILITIES_LOGICAL_MULTI_CAMERA)。

摄像机客户端可以通过调用来查询组成特定逻辑摄像机的物理设备的摄像机ID[`getPhysicalCameraIds()`](https://developer.android.google.cn/reference/android/hardware/camera2/CameraCharacteristics.html#getPhysicalCameraIds())。然后，作为结果一部分返回的ID将通过分别用于控制物理设备[`setPhysicalCameraId()`](https://developer.android.google.cn/reference/android/hardware/camera2/params/OutputConfiguration.html#setPhysicalCameraId(java.lang.String))。可以通过调用从完整结果中查询此类单个请求的结果[`getPhysicalCameraResults()`](https://developer.android.google.cn/reference/android/hardware/camera2/TotalCaptureResult.html#getPhysicalCameraResults())。

各个物理摄像机请求可能仅支持有限的参数子集。要接收支持的参数列表，开发人员可以调用[`getAvailablePhysicalCameraRequestKeys()`](https://developer.android.google.cn/reference/android/hardware/camera2/CameraCharacteristics#getAvailablePhysicalCameraRequestKeys())。

物理摄像机流仅支持非处理请求，并且仅支持单色和拜耳传感器。


# 三、 多摄像头支持列表

要在HAL端添加逻辑多摄像机设备：

*   [`ANDROID_REQUEST_AVAILABLE_CAPABILITIES_LOGICAL_MULTI_CAMERA`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.3/types.hal#231) 为由两个或更多物理相机支持的逻辑相机设备添加 功能，这些相机也向应用程序公开。
*   [`ANDROID_LOGICAL_MULTI_CAMERA_PHYSICAL_IDS`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.3/types.hal#160) 用物理摄像机ID列表填充静态 元数据字段。
*   填充物理相机流的像素之间进行相关所需要的深度相关静态元数据：[`ANDROID_LENS_POSE_ROTATION`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.2/types.hal#747)， [`ANDROID_LENS_POSE_TRANSLATION`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.2/types.hal#753)，[`ANDROID_LENS_INTRINSIC_CALIBRATION`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.2/types.hal#773)， [`ANDROID_LENS_DISTORTION`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.3/types.hal#89)，[`ANDROID_LENS_POSE_REFERENCE`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.3/types.hal#78)。
*   将静态[`ANDROID_LOGICAL_MULTI_CAMERA_SENSOR_SYNC_TYPE`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.3/types.hal#166) 元数据字段设置 为：

    *   [`ANDROID_LOGICAL_MULTI_CAMERA_SENSOR_SYNC_TYPE_APPROXIMATE`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.3/types.hal#255)：对于处于主-主模式的传感器，没有硬件快门/曝光同步。
    *   [`ANDROID_LOGICAL_MULTI_CAMERA_SENSOR_SYNC_TYPE_CALIBRATED`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.3/types.hal#256)：对于处于主从模式的传感器，硬件快门/曝光同步。
*   [`ANDROID_REQUEST_AVAILABLE_PHYSICAL_CAMERA_REQUEST_KEYS`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/metadata/3.3/types.hal#105) 用单个物理摄像机支持的参数列表填充 。如果逻辑设备不支持单个请求，则该列表可以为空。

*   如果支持个人请求，则处理并应用[`physicalCameraSettings`](https://android.googlesource.com/platform/hardware/interfaces/+/b75aa350e71b0c7dc59c4d51420a37608577a650/camera/device/3.4/types.hal#197) 可以作为捕获请求的一部分到达的个人 ，并相应地附加该个人 [`physicalCameraMetadata`](https://android.googlesource.com/platform/hardware/interfaces/+/39cf8fd9fe587e39b44e1ed63171c6eb5049f2df/camera/device/3.4/types.hal#238) 。

*   对于[3.5](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/device/3.5/ICameraDevice.hal) 或更高版本的Camera HAL设备（在Android 10中引入），请[`ANDROID_LOGICAL_MULTI_CAMERA_ACTIVE_PHYSICAL_ID`](https://android.googlesource.com/platform/hardware/interfaces/+/cc266b16998cc01adea0a76bf58403deb8d843df/camera/metadata/3.4/types.hal#82) 使用支持逻辑相机的当前活动物理相机的ID 填充 结果键。

对于运行Android 9的设备，摄像头设备必须支持用两个物理摄像头的相同大小（不适用于RAW流）和相同格式的物理流替换一个逻辑YUV / RAW流。这不适用于运行Android 10的设备。

对于运行Android 10且相机HAL设备版本为[3.5](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/device/3.5/ICameraDevice.hal) 或更高版本 的设备，相机设备必须支持[`isStreamCombinationSupported`](https://android.googlesource.com/platform/hardware/interfaces/+/40a8c6ed51abdf0ebebd566879ef232573696ab0/camera/device/3.5/ICameraDevice.hal#114) 应用程序以查询是否支持包含物理流的特定流组合。

# 四、Camera 流配置
对于逻辑摄像机，特定硬件级别的摄像机设备的强制流组合与中的要求相同[`CameraDevice.createCaptureSession`](https://developer.android.google.cn/reference/android/hardware/camera2/CameraDevice.html#createCaptureSession(java.util.List%3Candroid.view.Surface%3E,%20android.hardware.camera2.CameraCaptureSession.StateCallback,%20android.os.Handler))。流配置图中的所有流必须是逻辑流。

对于支持具有不同大小的物理子摄像机的RAW功能的逻辑摄像机设备，如果应用程序配置逻辑RAW流，则该逻辑摄像机设备不得切换到具有不同传感器大小的物理子摄像机。这样可以确保现有的RAW捕获应用程序不会损坏。

为了通过在RAW捕获期间在物理子摄像机之间切换来利用HAL实现的光学变焦，应用程序必须配置物理子摄像机流而不是逻辑RAW流。
逻辑摄像机及其基础物理摄像机都必须保证 其设备级别所需的 [强制流组合](https://developer.android.google.cn/reference/android/hardware/camera2/CameraDevice#createCaptureSession(java.util.List%3Candroid.view.Surface%3E,%20android.hardware.camera2.CameraCaptureSession.StateCallback,%20android.os.Handler))。

逻辑相机设备应基于其硬件级别和功能以与物理相机设备相同的方式操作。建议其功能集是单个物理相机的功能集的超集。

在运行Android 9的设备上，对于每种保证的流组合，逻辑摄像机必须支持：

*   假定物理摄像机支持大小和格式，则用两个大小和格式相同的物理流替换一个逻辑YUV_420_888或原始流，每个物理流都来自单独的物理摄像机。

*   如果逻辑摄像机不宣告RAW功能，但基础物理摄像机则宣告两个原始流，则从每个物理摄像机添加一个原始流。当物理相机具有不同的传感器尺寸时，通常会发生这种情况。

*   使用物理流代替相同大小和格式的逻辑流。当物理流和逻辑流的最小帧持续时间相同时，这一定不能减慢捕获的帧速率。

###性能和功耗考虑
性能：

由于资源限制，配置和流式传输物理流可能会减慢逻辑摄像机的捕获速率。
如果将基础相机设置为不同的帧速率，则应用物理相机设置可能会降低捕获速率。
功率：

在默认情况下，HAL的功率优化将继续起作用。
配置或请求物理流可能会覆盖HAL的内部电源优化，并导致更多的电源使用。

# 五、多摄像头客制化
您可以通过以下方式自定义设备实现。

*   逻辑摄像机设备的融合输出完全取决于HAL实现。关于如何从物理摄像机中导出融合逻辑流的决定对于应用程序和Android摄像机框架是透明的。
*   可以单独支持各个物理请求和结果。此类请求中的可用参数集也完全取决于特定的HAL实现。
*   从Android 10开始，HAL可以选择不发布中的某些或全部PHYSICAL_ID，从而减少应用程序可以直接打开的摄像头数量 [`getCameraIdList`](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/provider/2.4/ICameraProvider.hal#121)。[`getPhysicalCameraCharacteristics`](https://android.googlesource.com/platform/hardware/interfaces/+/d3feb3d62c139f08879bbb7f7a0513d593dafcc0/camera/device/3.5/ICameraDevice.hal#59) 然后，调用必须返回物理摄像机的特征。
逻辑多摄像机设备必须像其他任何常规摄像机一样通过摄像机CTS。可在[`LogicalCameraDeviceTest`](https://android.googlesource.com/platform/cts/+/master/tests/camera/src/android/hardware/camera2/cts/LogicalCameraDeviceTest.java) 模块中找到针对此类设备的测试用例 。

这三个ITS测试针对多相机系统，以促进图像的正确融合：

*   [`scene1/test_multi_camera_match.py`](https://android.googlesource.com/platform/cts/+/master/apps/CameraITS/tests/scene1/test_multi_camera_match.py)
*   [`scene4/test_multi_camera_alignment.py`](https://android.googlesource.com/platform/cts/+/master/apps/CameraITS/tests/scene4/test_multi_camera_alignment.py)
*   [`sensor_fusion/test_multi_camera_frame_sync.py`](https://android.googlesource.com/platform/cts/+/master/apps/CameraITS/tests/sensor_fusion/test_multi_camera_frame_sync.py)

Scene1和Scene4测试是使用 [ITS-in-a-box](https://source.android.google.cn/compatibility/cts/camera-its-box)测试装置运行的。该`test_multi_camera_match`测试断言，两个摄像机同时启用时，图像中心的亮度匹配。该 `test_multi_camera_alignment`测试断言相机间距，方向和变形参数已正确加载。如果多摄像机系统包括Wide FoV摄像机（> 90o），则需要ITS盒的rev2版本。

`Sensor_fusion` 是第二个测试设备，可重复执行规定的电话运动，并声称陀螺仪和图像传感器的时间戳匹配，并且多相机帧是同步的。

所有包装盒均可通过AcuSpec，Inc.（[www.acuspecinc.com](http://www.acuspecinc.com/)，fred @ acuspecinc.com）和MYWAY Manufacturing（[www.myway.tw](http://www.myway.tw/)，[sales @ myway.tw](http://www.myway.tw/)）获得。此外，可以通过West-Mark（[www.west-mark.com](http://www.west-mark.com/)，dgoodman@west-mark.com）购买rev1 ITS包装盒。


 **友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

 至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
