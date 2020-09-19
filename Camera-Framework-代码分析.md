#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、 Camera Framework 列文件目录
>二、 JNI 相关
>三、 AIDL 相关
>四、 IInterface 类型文件
>五、Parcelable 类型文件
>六、ICameraService 相关
>七、 ICameraServiceProxy.aidl  文件
>八、 ICamera 相关
>九、 ICameraDevice 相关
>十、 Services 目录下的文件介绍
>十一、API1/API2
>十二、QTICamera2Client
>十三、Device1/Device3



## 一、 Camera Framework 列文件目录

 
1.API1:`(frameworks/base/core/java/android/hardware/Camera.java)`
2.API2:`(frameworks/base/core/java/android/hardware/camera2/)`
3.JNI: `(frameworks/base/core/jni/)`
4.AIDL:`( frameworks/av/camera/aidl/)`
5.Native: `(frameworks/av/camera/) `
6.Service: `(frameworks/av/services/camera/libcameraservice/)`
7.Qcom Hal: `(vendor/qcom/propietary/camx/)`
8.Qcom Kernel：`(kernel/msm-4.19/techpack/camera/)`

## 二、 JNI  相关

 主目录为 `frameworks/base/core/jni`
```
// frameworks/base/core/jni
./android_hardware_camera2_legacy_LegacyCameraDevice.cpp
./android_hardware_Camera.cpp
./android/graphics/Camera.cpp
./include/android_runtime/android_hardware_camera2_CameraMetadata.h
./android_hardware_camera2_DngCreator.cpp
./android_hardware_camera2_CameraMetadata.cpp
./android_hardware_camera2_legacy_PerfMeasurement.cpp
```
API 1 中，使用 jni 通过 Binder 机制和 CameraService 通信。
API 2 中，直接在 CameraManager.java 中通过 Binder 机制和 CameraService 通信。

## 三、 AIDL 相关
Framework Camera AIDL 是 Camera 中客户端和服务端跨进程通信时使用的 AIDL 文件，代码都在` frameworks/av/camera/ `目录下，其中 aidl 文件一共有 16 个：
```
xmt@server005:~/frameworks/av/camera/aidl/android/hardware$ tree
.
├—— camera2
│   ├—— CaptureRequest.aidl
│   ├—— ICameraDeviceCallbacks.aidl
│   ├—— ICameraDeviceUser.aidl
│   ├—— impl
│   │   ├—— CameraMetadataNative.aidl
│   │   └—— CaptureResultExtras.aidl
│   ├—— params
│   │   ├—— OutputConfiguration.aidl
│   │   ├—— VendorTagDescriptor.aidl
│   │   └—— VendorTagDescriptorCache.aidl
│   └—— utils
│       └—— SubmitInfo.aidl
├—— CameraInfo.aidl
├—— CameraStatus.aidl
├—— ICamera.aidl
├—— ICameraClient.aidl
├—— ICameraService.aidl
├—— ICameraServiceListener.aidl
└—— ICameraServiceProxy.aidl

4 directories, 16 files
```
`frameworks/av/camera/aidl/ `目录下的 aidl 文件有两种类型：

- 作为 Binder 中的 IInterface 跨进程通信中能提供的方法
- 作为 Binder 中的 parcelable 跨进程通信数据传输的数据结构

很容易从名字上区分这两种类型的文件，IInterface 类型的文件都是以 I 开头的，比如：`ICameraService.aidl, ICameraDeviceUser.aidl `等。不管是哪种类型的 aidl 文件，它们都会生成对应的 .java, .h, .cpp 文件，分别供 Java 层和 CPP 层调用。

## 四、 IInterface 类型文件
IInterface 类型文件一共有 7 个，它们的 .java, .h, .cpp 文件，绝大部分都是自动生成的。

Java 文件是在 `frameworks/base/Android.mk` 中定义规则，在编译时自动生成：
```
// frameworks/base/Android.mk
LOCAL_SRC_FILES += 
    ...
    ../av/camera/aidl/android/hardware/ICameraService.aidl 
    ../av/camera/aidl/android/hardware/ICameraServiceListener.aidl 
    ../av/camera/aidl/android/hardware/ICameraServiceProxy.aidl 
    ../av/camera/aidl/android/hardware/ICamera.aidl 
    ../av/camera/aidl/android/hardware/ICameraClient.aidl 
    ../av/camera/aidl/android/hardware/camera2/ICameraDeviceUser.aidl 
    ../av/camera/aidl/android/hardware/camera2/ICameraDeviceCallbacks.aidl 
    ...
 ...
```
 

在 `out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/dotdot/ ` 目录下生成对应的 `Java` 文件：

 

```
// out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/dotdot/
av/camera/aidl/android/hardware/ICameraService.java
av/camera/aidl/android/hardware/ICameraServiceListener.java
av/camera/aidl/android/hardware/ICameraServiceProxy.java
av/camera/aidl/android/hardware/ICamera.java
av/camera/aidl/android/hardware/ICameraClient.java
av/camera/aidl/android/hardware/camera2/ICameraDeviceUser.java
av/camera/aidl/android/hardware/camera2/ICameraDeviceCallbacks.java
```
`.h, .cpp` 文件中，`ICamera.aidl, ICameraClient.aidl` 两个文件是直接以代码形式手动实现的：

```
// 1. ICameraClient.aidl
frameworks/av/camera/aidl/android/hardware/ICameraClient.aidl
frameworks/av/camera/include/camera/android/hardware/ICameraClient.h
frameworks/av/camera/ICameraClient.cpp

// 2. ICamera.aidl
frameworks/av/camera/aidl/android/hardware/ICamera.aidl
frameworks/av/camera/include/camera/android/hardware/ICamera.h
frameworks/av/camera/ICamera.cpp
```

其他 5 个 `aidl` 文件是在 ` frameworks/av/camera/Android.bp` 中定义规则，编译时自动生成对应的 `.h, .cpp` 文件：

```
// frameworks/av/camera/Android.bp
cc_library_shared {
    name: "libcamera_client",

    aidl: {
        export_aidl_headers: true,
        local_include_dirs: ["aidl"],
        include_dirs: [
            "frameworks/native/aidl/gui",
        ],
    },

    srcs: [
        // AIDL files for camera interfaces
        // The headers for these interfaces will be 
        // available to any modules that
        // include libcamera_client, at the path "aidl/package/path/BnFoo.h"
        "aidl/android/hardware/ICameraService.aidl",
        "aidl/android/hardware/ICameraServiceListener.aidl",
        "aidl/android/hardware/ICameraServiceProxy.aidl",
        "aidl/android/hardware/camera2/ICameraDeviceCallbacks.aidl",
        "aidl/android/hardware/camera2/ICameraDeviceUser.aidl",


        // Source for camera interface parcelables, 
        // and manually-written interfaces
        "Camera.cpp",
        "CameraMetadata.cpp",
        "CameraParameters.cpp",
        ...
}
```
在 `out/soong/.intermediates/frameworks/av/camera/libcamera_client/ ` 目录下生成对应的 `.h, .cpp` 文件，通常在该目录下会同时生成 32 和 64 位两套代码，但实际两份代码是一样的，这里选取 64 位的：

*   64 位：`android_arm64_armv8-a_shared_core`
*   32 位：`android_arm_armv7-a-neon_cortex-a53_shared_core`


```
// 目录 out/soong/.intermediates/frameworks/av/camera/libcamera_client
// 64 位 android_arm64_armv8-a_shared_core/gen/aidl/
android/hardware/ICameraService.h
android/hardware/BnCameraService.h
frameworks/av/camera/aidl/android/hardware/ICameraService.cpp

android/hardware/ICameraServiceListener.h
android/hardware/BnCameraServiceListener.h
frameworks/av/camera/aidl/android/hardware/ICameraServiceListener.cpp

android/hardware/ICameraServiceProxy.h
android/hardware/BnCameraServiceProxy.h
frameworks/av/camera/aidl/android/hardware/ICameraServiceProxy.cpp

android/hardware/camera2/ICameraDeviceUser.h
android/hardware/camera2/BnCameraDeviceUser.h
frameworks/av/camera/aidl/android/hardware/camera2/ICameraDeviceUser.cpp

android/hardware/camera2/ICameraDeviceCallbacks.h
android/hardware/camera2/BnCameraDeviceCallbacks.h
frameworks/av/camera/aidl/android/hardware/camera2/ICameraDeviceCallbacks.cpp

```
##五、 parcelable  类型文件

`parcelable` 类型文件一共有 9 个，它们都是手动编写的代码。

`Java` 文件目录为 `frameworks/base/core/java/android/hardware/` ：

```
// frameworks/base/core/java/android/hardware/
camera2/CaptureRequest.java
camera2/impl/CameraMetadataNative.java
camera2/impl/CaptureResultExtras.java
camera2/params/OutputConfiguration.java
camera2/params/VendorTagDescriptor.java
camera2/params/VendorTagDescriptorCache.java
camera2/utils/SubmitInfo.java
CameraInfo.java
CameraStatus.java
```

`.h, .cpp` 文件并不一定是和 `aidl` 文件名称一一对应的，而是在 `aidl` 文件中定义的，比如 `CameraStatus.aidl` 定义如下：

```
package android.hardware;

/** @hide */
parcelable CameraStatus cpp_header "camera/CameraBase.h";

```

`parcelable` 类型的 `aidl` 文件对应的 `.h, .cpp` 文件目录为 `frameworks/av/camera` ，对应关系整理如下：
```
// .h, .cpp 文件目录 frameworks/av/camera
// CaptureRequest.aidl
include/camera/camera2/CaptureRequest.h
camera2/CaptureRequest.cpp

// CameraMetadataNative.aidl
include/camera/CameraMetadata.h
CameraMetadata.cpp

// CaptureResultExtras.aidl
include/camera/CaptureResult.h
CaptureResult.cpp

// OutputConfiguration.aidl
include/camera/camera2/OutputConfiguration.h
camera2/OutputConfiguration.cpp

// VendorTagDescriptor.aidl 和 VendorTagDescriptorCache.aidl
include/camera/VendorTagDescriptor.h
VendorTagDescriptor.cpp

// SubmitInfo.aidl
include/camera/camera2/SubmitInfo.h
camera2/SubmitInfo.cpp

// CameraInfo.aidl 和 CameraStatus.aidl
include/camera/CameraBase.h
CameraBase.cpp
```
##  六、ICameraService  相关

分为客户端向服务端的请求 `ICameraService.aidl` 和客户端监听服务端的变化 `ICameraServiceListener.aidl` 。这两个 `AIDL` 是在 `CameraService.cpp` 中实现对应功能的。
```
interface 
{
    ...
    const int CAMERA_TYPE_BACKWARD_COMPATIBLE = 0;
    const int CAMERA_TYPE_ALL = 1;

    // 返回指定类型的相机设备数量
    int getNumberOfCameras(int type);
    // 根据 id 返回当前相机设备信息
    CameraInfo getCameraInfo(int cameraId);

    ...
    const int CAMERA_HAL_API_VERSION_UNSPECIFIED = -1;
    // api1 + hal1
    ICamera connect(ICameraClient client,
            int cameraId,
            String opPackageName,
            int clientUid, int clientPid);

    // api2 + hal3
    ICameraDeviceUser connectDevice(ICameraDeviceCallbacks callbacks,
            String cameraId,
            String opPackageName,
            int clientUid);

    // api1 + 指定 hal 版本（通常为 hal1）
    ICamera connectLegacy(ICameraClient client,
            int cameraId,
            int halVersion,
            String opPackageName,
            int clientUid);

    // 添加和移除 ICameraServiceListener 监听
    CameraStatus[] addListener(ICameraServiceListener listener);
    void removeListener(ICameraServiceListener listener);

    // 根据 id 返回相机支持的属性
    CameraMetadataNative getCameraCharacteristics(String cameraId);

    // 获取 vendor tag 
    VendorTagDescriptor getCameraVendorTagDescriptor();
    VendorTagDescriptorCache getCameraVendorTagCache();

    // camera api 1 获取参数信息
    String getLegacyParameters(int cameraId);

    const int API_VERSION_1 = 1;
    const int API_VERSION_2 = 2;
    // 指定 id 支持的 API 版本
    boolean supportsCameraApi(String cameraId, int apiVersion);
    // 指定 id 设置手电筒模式
    void setTorchMode(String cameraId, boolean enabled, 
        IBinder clientBinder);

    // 服务端向系统打印系统消息
    const int EVENT_NONE = 0;
    const int EVENT_USER_SWITCHED = 1;
    oneway void notifySystemEvent(int eventId, in int[] args);
}

// 2. ICameraServiceListener.aidl
interface ICameraServiceListener
{
    const int STATUS_NOT_PRESENT      = 0;
    const int STATUS_PRESENT          = 1;
    const int STATUS_ENUMERATING      = 2;
    const int STATUS_NOT_AVAILABLE    = -2;
    const int STATUS_UNKNOWN          = -1;
    // 相机设备状态变化事件
    oneway void onStatusChanged(int status, String cameraId);

    const int TORCH_STATUS_NOT_AVAILABLE = 0;
    const int TORCH_STATUS_AVAILABLE_OFF = 1;
    const int TORCH_STATUS_AVAILABLE_ON  = 2;
    const int TORCH_STATUS_UNKNOWN = -1;
    // 手电筒状态变化事件
    oneway void onTorchStatusChanged(int status, String cameraId);
}
```
##七、 ICameraServiceProxy.aidl  文件

`CameraServiceProxy` 服务是在 `Java` 层注册的：

```
interface ICameraServiceProxy
{
    // CameraService 向代理服务发送消息，通知用户更新
    oneway void pingForUserUpdate();

    const int CAMERA_STATE_OPEN = 0;
    const int CAMERA_STATE_ACTIVE = 1;
    const int CAMERA_STATE_IDLE = 2;
    const int CAMERA_STATE_CLOSED = 3;
    const int CAMERA_FACING_BACK = 0;
    const int CAMERA_FACING_FRONT = 1;
    const int CAMERA_FACING_EXTERNAL = 2;

    // CameraService 向代理服务发送消息，通知相机设备状态更新
    oneway void notifyCameraState(String cameraId, int facing, 
            int newCameraState, String clientName);
}
```

##八、 ICamera  相关

Camera API1 才会使用到，分为 ICamera.aidl, ICameraClient.aidl
它们的代码是手动实现的，参考：CameraClient.h/cpp, Camera.h/cpp
 
## 九、ICameraDevice 相关

Camera API2 才会使用到，分为客户端向服务端的请求 ICameraDeviceUser.aidl 和服务端发给客户端的回调 ICameraDeviceCallbacks.aidl 。
表示相机设备具备的能力，能够提供的函数；这两个 AIDL 是在 CameraDeviceClient 中实现对应功能的。

```
// 1. ICameraDeviceUser.aidl 
interface ICameraDeviceUser
{
    void disconnect();

    const int NO_IN_FLIGHT_REPEATING_FRAMES = -1;
    // 向设备提交捕获请求
    SubmitInfo submitRequest(in CaptureRequest request, boolean streaming);
    SubmitInfo submitRequestList(in CaptureRequest[] requestList, 
        boolean streaming);
    // 取消置顶 id 的重复请求，并返回上次请求的帧 id
    long cancelRequest(int requestId);

    const int NORMAL_MODE = 0;
    const int CONSTRAINED_HIGH_SPEED_MODE = 1;
    const int VENDOR_MODE_START = 0x8000;

    // 在流处理前执行配置请求
    void beginConfigure();
    // 根据指定输出配置，创建流
    int createStream(in OutputConfiguration outputConfiguration);
    void endConfigure(int operatingMode);
    void deleteStream(int streamId);

    // 创建输入流，返回流 id
    int createInputStream(int width, int height, int format);
    // 返回输入流的 Surface
    Surface getInputSurface();

    // Keep in sync with public API in
    // frameworks/base/core/java/android/hardware/camera2/CameraDevice.java
    const int TEMPLATE_PREVIEW = 1;
    const int TEMPLATE_STILL_CAPTURE = 2;
    const int TEMPLATE_RECORD = 3;
    const int TEMPLATE_VIDEO_SNAPSHOT = 4;
    const int TEMPLATE_ZERO_SHUTTER_LAG = 5;
    const int TEMPLATE_MANUAL = 6;
    // 根据模板创建默认请求，返回相机参数信息
    CameraMetadataNative createDefaultRequest(int templateId);
    // 获取相机参数信息
    CameraMetadataNative getCameraInfo();
    void waitUntilIdle();
    long flush();
    void prepare(int streamId);
    void tearDown(int streamId);
    void prepare2(int maxCount, int streamId);
    void finalizeOutputConfigurations(int streamId, 
        in OutputConfiguration outputConfiguration);
}

// 2. ICameraDeviceCallbacks.aidl
interface ICameraDeviceCallbacks
{
    ...
    oneway void onDeviceError(int errorCode, 
        in CaptureResultExtras resultExtras);
    oneway void onDeviceIdle();
    oneway void onCaptureStarted(in CaptureResultExtras resultExtras, 
        long timestamp);
    oneway void onResultReceived(in CameraMetadataNative result,
                                 in CaptureResultExtras resultExtras);
    oneway void onPrepared(int streamId);
    // 重复请求引起的错误回调
    oneway void onRepeatingRequestError(in long lastFrameNumber,
                                        in int repeatingRequestId);
    oneway void onRequestQueueEmpty();
}
```

##  十、Services  目录下的文件介绍

`frameworks/av/services/camera/libcameraservice` AOSP 中这个目录下是 87 个文件，而 Qcom 的基线中增加了 27 个文件，分别为 api1/qticlient2 目录下的 25 个文件，以及 QTICamera2Client.cpp, QTICamera2Client.h 两个文件。

```
.
├—— Android.mk
├—— api1
│   ├—— client2
│   └—— qticlient2
├—— api2
├—— CameraFlashlight.cpp
├—— CameraFlashlight.h
├—— CameraService.cpp
├—— CameraService.h
├—— common
├—— device1
├—— device3
├—— gui
├—— MODULE_LICENSE_APACHE2
├—— NOTICE
├—— tests
└—— utils
```

从目录结构上可以看出，`API1/2` 和 `HAL1/3` 就是在这一层体现的。

## 十一、 API1/API2 

APP Java 客户端调用服务端方法时，Camera API1/2 接口对应功能都是在 CameraService 中实现的，而这里的 API1/2 目录对应的就是对上层不同版本接口的处理。

 

```
api1
├—— Camera2Client.cpp
├—— Camera2Client.h
├—— CameraClient.cpp
├—— CameraClient.h
├—— client2
│   ├—— CallbackProcessor.cpp
│   ├—— CallbackProcessor.h
│   ├—— Camera2Heap.h
│   ├—— CaptureSequencer.cpp
│   ├—— CaptureSequencer.h
│   ├—— FrameProcessor.cpp
│   ├—— FrameProcessor.h
│   ├—— JpegCompressor.cpp
│   ├—— JpegCompressor.h
│   ├—— JpegProcessor.cpp
│   ├—— JpegProcessor.h
│   ├—— Parameters.cpp
│   ├—— Parameters.h
│   ├—— StreamingProcessor.cpp
│   ├—— StreamingProcessor.h
│   ├—— ZslProcessor.cpp
│   └—— ZslProcessor.h
├—— QTICamera2Client.cpp
├—— QTICamera2Client.h
└—— qticlient2
    ├—— CallbackProcessor.cpp
    ├—— CallbackProcessor.h
    ├—— Camera2Heap.h
    ├—— CaptureSequencer.cpp
    ├—— CaptureSequencer.h
    ├—— FrameProcessor.cpp
    ├—— FrameProcessor.h
    ├—— JpegCompressor.cpp
    ├—— JpegCompressor.h
    ├—— JpegProcessor.cpp
    ├—— JpegProcessor.h
    ├—— Parameters.cpp
    ├—— Parameters.h
    ├—— QTICaptureSequencer.cpp
    ├—— QTICaptureSequencer.h
    ├—— QTIFrameProcessor.cpp
    ├—— QTIFrameProcessor.h
    ├—— QTIParameters.cpp
    ├—— QTIParameters.h
    ├—— RawProcessor.cpp
    ├—— RawProcessor.h
    ├—— StreamingProcessor.cpp
    ├—— StreamingProcessor.h
    ├—— ZslProcessor.cpp
    └—— ZslProcessor.h
api2
├—— CameraDeviceClient.cpp
└—— CameraDeviceClient.h
```
  

**BasicClient 有三个重要的子类：**

- CameraClient
如果平台仅支持 HAL 1，即 CAMERA_DEVICE_API_VERSION_1_0 ；使用 API 1/2 + HAL 1 都会对应该客户端。

- Camera2Client
如果平台支持 HAL 3 ，即 CAMERA_DEVICE_API_VERSION_3_0 及以上版本；使用 API 1 + HAL 3 对应的客户端。Camera2Client 会将 API1 中的接口转换为 API2 中对应的功能。

- CameraDeviceClient
如果平台支持 HAL 3 ，使用 API 2 + HAL 3 对应的客户端。
平台仅支持 HAL 1 时，API 2 在 openCamera 时，通过 CameraDeviceUserShim 将 API 2 转换为 API 1 ，即 HAL 1 + API 1 向下发起请求。
LegacyCameraDevice 会将 CAMERA API2 转换为 CAMERA API1 ，而 CameraDeviceUserShim 封装了 LegacyCameraDevice 。

## 十二、 QTICamera2Client 

Qcom 的基线中增加了 27 个文件，分别为 api1/qticlient2 目录下的 25 个文件，以及 QTICamera2Client.cpp, QTICamera2Client.h 两个文件。
而 QTICamera2Client 是高通针对 API1 做的优化？在什么情况下会转换为 QTICamera2Client 呢？看如下源码：
```// 1. Camera2Client.h
class Camera2Client :
        public Camera2ClientBase<CameraService::Client>
{

friend class QTICamera2Client;
#endif
...

    sp<camera2::RawProcessor> mRawProcessor;
#endif
...

    sp<QTICamera2Client> mQTICamera2Client;
#endif
...
}

// 2. Camera2Client.cpp
template<typename TProviderPtr>
status_t Camera2Client::initializeImpl(TProviderPtr providerPtr)
{
...

    mQTICamera2Client = new QTICamera2Client(this);
#endif
...

    mRawProcessor = new RawProcessor(this, mCaptureSequencer);
    threadName = String8::format("C2-%d-RawProc", mCameraId);
    mRawProcessor->run(threadName.string());
#endif
...
}
```
QTICamera2Client 是高通对 API 1 中 Camera2Client 做的一层封装，添加了部分功能，主要是向上提供 raw 数据。
```
// 1. QTICamera2Client.h
class QTICamera2Client: public virtual RefBase{
private:
    wp<Camera2Client> mParentClient;
    status_t stopPreviewExtn();

public:
    QTICamera2Client(sp<Camera2Client> client);
    ~QTICamera2Client();
    ...
}

// 2. QTICamera2Client.cpp
QTICamera2Client::QTICamera2Client(sp<Camera2Client> client):
        mParentClient(client) {
}
```
##十三、 device1/device3 

device1/device3 可以理解为 Framework 层对应 HAL 层的 HAL 1/3 。
```
device1
├—— CameraHardwareInterface.cpp
└—— CameraHardwareInterface.h
device3
├—— Camera3BufferManager.cpp
├—— Camera3BufferManager.h
├—— Camera3Device.cpp
├—— Camera3Device.h
├—— Camera3DummyStream.cpp
├—— Camera3DummyStream.h
├—— Camera3InputStream.cpp
├—— Camera3InputStream.h
├—— Camera3IOStreamBase.cpp
├—— Camera3IOStreamBase.h
├—— Camera3OutputStream.cpp
├—— Camera3OutputStream.h
├—— Camera3OutputStreamInterface.h
├—— Camera3SharedOutputStream.cpp
├—— Camera3SharedOutputStream.h
├—— Camera3StreamBufferFreedListener.h
├—— Camera3StreamBufferListener.h
├—— Camera3Stream.cpp
├—— Camera3Stream.h
├—— Camera3StreamInterface.h
├—— Camera3StreamSplitter.cpp
├—— Camera3StreamSplitter.h
├—— StatusTracker.cpp
└—— StatusTracker.h
```
*   `API1/device1/HAL1` 的连接过程
```
// API1: CameraClient.h
sp<CameraHardwareInterface>     mHardware;
// device1: CameraHardwareInterface.h
sp<hardware::camera::device::V1_0::ICameraDevice> mHidlDevice;
// 这里的 ICameraDevice 即为 HAL1
```
API1 的客户端 CameraClient 对应的 device1: CameraHardwareInterface，而它直接包含了 HAL1 中 ICameraDevice 

*   `API1/3/device3/HAL3` 的连接过程
```
// API1: Camera2Client.h
class Camera2Client :
    public Camera2ClientBase<CameraService::Client>{...}

// API2: CameraDeviceClient.h
class CameraDeviceClient :
    public Camera2ClientBase<CameraDeviceClientBase>,
    public camera2::FrameProcessorBase::FilteredListener{...}

// Camera2ClientBase.h
sp<CameraDeviceBase>  mDevice;

// Camera2ClientBase.cpp
template <typename TClientBase>
Camera2ClientBase<TClientBase>::Camera2ClientBase(
    const sp<CameraService>& cameraService,
    const sp<TCamCallbacks>& remoteCallback,
    const String16& clientPackageName,
    const String8& cameraId,
    int cameraFacing,
    int clientPid,
    uid_t clientUid,
    int servicePid):
    TClientBase(cameraService, remoteCallback, clientPackageName,
            cameraId, cameraFacing, clientPid, clientUid, servicePid),
    mSharedCameraCallbacks(remoteCallback),
    mDeviceVersion(cameraService->getDeviceVersion(
            TClientBase::mCameraIdStr)),
    mDeviceActive(false)
{
    ...
    mInitialClientPid = clientPid;
    // 只要是 HAL3 ，则 device 都是对应的 Camera3Device
    mDevice = new Camera3Device(cameraId);
    ...
}
 ```
从源码可以看出，不管是 API1/2 ，只要是 HAL 3 ，Camera2Client, CameraDeviceClient 两个客户端对应的都是 device3: Camera3Device 。

Camera3Device::HalInterface 内部类，用于和 HAL 层通信，实现了 HAL 层 ICameraDeviceSession.hal 部分代码：
```
// Camera3Device.h
class Camera3Device :
    ...
    class HalInterface : public camera3::Camera3StreamBufferFreedListener {
      public:
        ...
        // Calls into the HAL interface

        // Caller takes ownership of requestTemplate
        status_t constructDefaultRequestSettings(
                camera3_request_template_t templateId,
                /*out*/ camera_metadata_t **requestTemplate);
        status_t configureStreams(
            /*inout*/ camera3_stream_configuration *config);
        status_t processCaptureRequest(camera3_capture_request_t *request);
        status_t flush();
        status_t close();
        ...
    }
    ...
}
```
原文链接：https://www.dazhuanlan.com/2019/10/16/5da68019bdbe3/


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
