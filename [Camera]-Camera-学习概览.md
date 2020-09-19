#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>1.Camera 概述
>2.Camera 架构
>3.HAL 层实现
>4.旧版HAL 组件架构及实现

#1.Camera 概述

Android 的相机硬件抽象层 (HAL) 可将 [Camera 2](http://developer.android.google.cn/reference/android/hardware/package-summary.html) 中较高级别的相机框架 API 连接到底层的相机驱动程序和硬件。相机子系统包括相机管道组件的实现，而相机 HAL 则可提供用于实现您的这些组件版本的接口

如果您要在搭载 Android 8.0 或更高版本的设备上实现相机 HAL，则必须使用 HIDL 接口。要了解旧版组件，请参阅[旧版 HAL 组件](https://source.android.google.cn/devices/camera#architecture-legacy)。

# 2.Camera 架构

下列图表和列表说明了 HAL 组件：
![图 1. 相机架构](https://upload-images.jianshu.io/upload_images/5851256-fa35d59082d426c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
##  app 框架
应用代码位于应用框架级别，它使用 [Camera 2](https://developer.android.google.cn/reference/android/hardware/camera2/package-summary) API 与相机硬件进行交互。在内部，这些代码会调用相应的 [Binder](https://developer.android.google.cn/reference/android/os/Binder.html) 接口，以访问与相机互动的原生代码

## AIDL
与 CameraService 关联的 Binder 接口可在 [frameworks/av/camera/aidl/android/hardware](https://android.googlesource.com/platform/frameworks/av/+/master/camera/aidl/android/hardware/ICameraService.aidl) 中找到。生成的代码会调用较低级别的原生代码以获取对实体相机的访问权限，并返回用于在框架级别创建 [CameraDevice](https://developer.android.google.cn/reference/android/hardware/camera2/CameraDevice) 并最终创建 [CameraCaptureSession](https://developer.android.google.cn/reference/android/hardware/camera2/CameraCaptureSession.html) 对象的数据。

## Native 框架
此框架位于 `frameworks/av/` 中，并提供相当于 [CameraDevice](https://developer.android.google.cn/reference/android/hardware/camera2/CameraDevice) 和 [CameraCaptureSession](https://developer.android.google.cn/reference/android/hardware/camera2/CameraCaptureSession) 类的原生类。另请参阅 [NDK camera2 参考](https://developer.android.google.cn/ndk/reference/group/camera)。

## Binder IPC 通讯接口
IPC binder 接口用于实现跨越进程边界的通信。调用相机服务的若干个相机 Binder 类位于 `frameworks/av/camera/camera/aidl/android/hardware` 目录中。 [ICameraService](https://android.googlesource.com/platform/frameworks/av/+/master/camera/aidl/android/hardware/ICameraService.aidl) 是相机服务的接口；[ICameraDeviceUser](https://android.googlesource.com/platform/frameworks/av/+/master/camera/aidl/android/hardware/camera2/ICameraDeviceUser.aidl) 是已打开的特定相机设备的接口；[ICameraServiceListener](https://android.googlesource.com/platform/frameworks/av/+/master/camera/aidl/android/hardware/ICameraServiceListener.aidl) 和 [ICameraDeviceCallbacks](https://android.googlesource.com/platform/frameworks/av/+/master/camera/aidl/android/hardware/camera2/ICameraDeviceCallbacks.aidl) 分别是对应用框架的 CameraService 和 CameraDevice 回调。

## CameraService 
位于` frameworks/av/services/camera/libcameraservice/CameraService.cpp`下的相机服务是与 HAL 进行互动的实际代码。

## HAL 
硬件抽象层定义了由相机服务调用、且您必须实现以确保相机硬件正常运行的标准接口。

# 3.HAL 层实现
HAL 位于相机驱动程序和更高级别的 Android 框架之间，它定义您必须实现的接口，以便应用可以正确地操作相机硬件。从 Android 8.0 开始，相机 HAL 接口是 Project [Treble](https://source.android.google.cn/devices/architecture/treble) 的一部分，相应的 [HIDL](https://source.android.google.cn/devices/architecture/hidl/) 接口在 [hardware/interfaces/camera](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/) 中定义。

典型的绑定式 HAL 必须实现以下 HIDL 接口：

*   [ICameraProvider](https://source.android.google.cn/reference/hidl/android/hardware/camera/provider/2.4/ICameraProvider)：用于枚举单个设备并管理其状态。
*   [ICameraDevice](https://source.android.google.cn/reference/hidl/android/hardware/camera/device/3.2/ICameraDevice)：相机设备接口。
*   [ICameraDeviceSession](https://source.android.google.cn/reference/hidl/android/hardware/camera/device/3.2/ICameraDeviceSession)：活跃的相机设备会话接口。

参考 HIDL 实现适用于 [CameraProvider.cpp](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/provider/2.4/default/CameraProvider.cpp)、[CameraDevice.cpp](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/device/3.2/default/CameraDevice.cpp) 和 [CameraDeviceSession.cpp](https://android.googlesource.com/platform/hardware/interfaces/+/master/camera/device/3.2/default/CameraDeviceSession.cpp)。该实现封装了仍在使用[旧版 API](https://android.googlesource.com/platform/hardware/libhardware/+/master/include/hardware/camera3.h) 的旧 HAL。从 Android 8.0 开始，相机 HAL 实现必须使用 HIDL API；不支持使用旧版接口。

要详细了解 Treble 和 HAL 开发，请参阅 [Treble 资源](https://source.android.google.cn/devices/architecture/treble#treble-resources)。

# 4.旧版HAL 组件架构及实现

此部分介绍了旧版 HAL 组件的架构以及如何实现 HAL。搭载 Android 8.0 或更高版本的设备上的相机 HAL 实现必须改用 HIDL API（如上所述）。

## 旧版HAL 框架
下列图表和列表说明了旧版相机 HAL 组件：

![图 1. 相机架构](https://upload-images.jianshu.io/upload_images/5851256-e02f20f1d883f029.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## app 框架
应用代码位于应用框架级别，它利用 [android.hardware.Camera](http://developer.android.google.cn/reference/android/hardware/Camera.html) API 与相机硬件进行交互。在内部，此代码会调用相应的 JNI 粘合类，以访问与相机互动的原生代码。

## JNI

与 [android.hardware.Camera](http://developer.android.google.cn/reference/android/hardware/Camera.html) 关联的 JNI 代码位于 `frameworks/base/core/jni/android_hardware_Camera.cpp` 中。此代码会调用较低级别的原生代码以获取对实体相机的访问权限，并返回用于在框架级别创建 [android.hardware.Camera](http://developer.android.google.cn/reference/android/hardware/Camera.html) 对象的数据。

## Native 框架
在 `frameworks/av/camera/Camera.cpp` 中定义的原生框架可提供相当于 [android.hardware.Camera](http://developer.android.google.cn/reference/android/hardware/Camera.html) 类的原生类。此类会调用 IPC binder 代理，以获取对相机服务的访问权限。

## Binder IPC 代理
PC binder 代理用于促进跨越进程边界的通信。调用相机服务的 3 个相机 binder 类位于 frameworks/av/camera 目录中。ICameraService 是相机服务的接口，ICamera 是已打开的特定相机设备的接口，ICameraClient 是指回应用框架的设备接口。
## CameraService

位于 `frameworks/av/services/camera/libcameraservice/CameraService.cpp `下的相机服务是与 HAL 进行互动的实际代码。
## HAL
硬件抽象层定义了由相机服务调用、且您必须实现以确保相机硬件正常运行的标准接口。

## kernel 驱动
相机的驱动程序可与实际相机硬件以及您的 HAL 实现进行互动。相机和驱动程序必须支持 YV12 和 NV21 图像格式，以便在显示和视频录制时支持预览相机图像。
## 旧版HAL 实现
HAL 位于相机驱动程序和更高级别的 Android 框架之间，它定义您必须实现的接口，以便应用可以正确地操作相机硬件。HAL 接口在 `hardware/libhardware/include/hardware/camera.h` 和 `hardware/libhardware/include/hardware/camera_common.h` 标头文件中定义。

`camera_common.h` 定义 `camera_module`；这是一个标准结构，可用于获取有关相机的一般信息，例如相机 ID 和所有相机通用的属性（例如，摄像头是前置还是后置）。

`camera.h` 包含与 [android.hardware.Camera](http://developer.android.google.cn/reference/android/hardware/Camera.html) 对应的代码。此标头文件会声明一个 `camera_device` 结构，该结构又反过来包含一个带函数指针（可实现 HAL 接口）的 `camera_device_ops` 结构。有关开发者可以设置的相机参数的文档，请参阅 `frameworks/av/include/camera/CameraParameters.h`。通过 HAL 中的 `int (*set_parameters)(struct camera_device *, const char *parms)` 来设置这些参数以及指向的函数。

有关 HAL 实现的示例，请参阅 `hardware/ti/omap4xxx/camera` 中的 Galaxy Nexus HAL 实现。

## 配置 Camera 共享库

设置 Android 编译系统，以将 HAL 实现正确打包到共享库中，并通过创建 Android.mk 文件将其复制到相应位置：

1.创建一个 `device/<company_name>/<device_name>/camera` 目录以包含您库的源文件。
2.创建一个 Android.mk 文件来编译共享库。确保 Makefile 包含以下行：
```
LOCAL_MODULE := camera.<device_name>
LOCAL_MODULE_RELATIVE_PATH := hw
```
您的库必须命名为` camera.<device_name>（自动附加 .so）`，以便 Android 可以正确加载库。例如，请参阅 `hardware/ti/omap4xxx/Android.mk` 中的 Galaxy Nexus 相机的 Makefile。

3.使用您设备的 Makefile 复制` frameworks/native/data/etc` 目录中的必要功能 XML 文件，以指定您的设备具有相机功能。例如，要指定您的设备具有相机闪光灯并可自动对焦，请在您设备的 `<device>/<company_name>/<device_name>/device.mk Makefile `中添加以下行：
```
PRODUCT_COPY_FILES := \ ...

PRODUCT_COPY_FILES += \
frameworks/native/data/etc/android.hardware.camera.flash-autofocus.xml:system/etc/permissions/android.hardware.camera.flash-autofocus.xml \
```
4.在 `device/<company_name>/<device_name>/media_profiles.xml` 和 `device/<company_name>/<device_name>/media_codecs.xml` XML 文件中声明您相机的媒体编解码器、格式和分辨率功能。如需了解详情，请参阅[将编解码器展示给框架](https://source.android.google.cn/devices/media/index.html#expose)。
5.在您设备的 `device/<company_name>/<device_name>/device.mk` Makefile 中添加以下行，以将 `media_profiles.xml` 和 `media_codecs.xml` 文件复制到相应位置：
```
# media config xml file
PRODUCT_COPY_FILES += \
    <device>/<company>/<device>/media_profiles.xml:system/etc/media_profiles.xml

# media codec config xml file
PRODUCT_COPY_FILES += \
    <device>/<company>/<device>/media_codecs.xml:system/etc/media_codecs.xml
```
要将相机应用包含在设备的系统映像中，请在设备的 device/<company>/<device>/device.mk Makefile 中的 PRODUCT_PACKAGES 变量中指定该应用：
```
PRODUCT_PACKAGES := \
Gallery2 \
...
```

 **友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

 至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
