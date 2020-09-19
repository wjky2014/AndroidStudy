#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
  
>一、Android O上的Treble机制
>二、Camera HAL3的框架更新
>三、核心概念：Request
 
 

# 一、Android O上的Treble机制 

　　在 Android O 中，系统启动时，会启动一个 CameraProvider 服务，它是从 cameraserver 进程中分离出来，作为一个独立进程 `android.hardware.camera.provider@2.4-service ` 用来控制 camera HAL，cameraserver通过 HIDL 机制于camera provider进行通信。HIDL源自于 Android O 版本加入的 Treble 机制，它的主要功能是将 service 与 HAL 隔离，以方便 HAL 部分进行独立升级，类似于 APP 与 Framework 之间的 Binder 通信机制，通过引入一个进程间通信机制而针对不同层级进行解耦（从 Local call 变成了 Remote call）。如下图：

　　　　　　![ ](https://upload-images.jianshu.io/upload_images/2118860-b1e66e21117e2a9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 cameraserver 与 provider 这两个进程启动、初始化的调用逻辑，如下图：

　　　　　　![ ](https://upload-images.jianshu.io/upload_images/2118860-ff40d068dfed1b83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 二、Camera HAL3的框架更新 

-  **Application framework：**
用于给APP提供访问hardware的Camera API2，通过binder来访问camera service。

-  **AIDL:** 
基于Binder实现的一个用于让App fw代码访问natice fw代码的接口。其实现存在于下述路径：`frameworks/av/camera/aidl/android/hardware`。其中：

    (1) ICameraService 
是相机服务的接口。用于请求连接、添加监听等。
    (2) ICameraDeviceUser 
是已打开的特定相机设备的接口。应用框架可通过它访问具体设备。
    (3) ICameraServiceListener 和 ICameraDeviceCallbacks 
分别是从 CameraService 和 CameraDevice 到应用框架的回调。

-   **Natice framework**：
`frameworks/av/`。提供了ICameraService、ICameraDeviceUser、ICameraDeviceCallbacks、ICameraServiceListener等aidl接口的实现。以及camera server的main函数。
*   **Binder IPC interface**：
提供进程间通信的接口，APP和CameraService的通信、CameraService和HAL的通信。其中，AIDL、HIDL都是基于Binder实现的。
*   **Camera Service**：
`frameworks/av/services/camera/`。同APP、HAL交互的服务，起到了承上启下的作用。
*   **HAL：**
Google的HAL定义了可以让Camera Service访问的标准接口。对于供应商而言，必须要实现这些接口。

##2.1 Camera HAL3 构建连路的过程

如下图（红色虚线是上行路线，黑色虚线则是下行路线）：

**![ ](https://upload-images.jianshu.io/upload_images/2118860-575a295906b5839a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)** 

## 2.2 从 App 到 CameraService的调用流程 

　　从 Application 连接到 CameraService，这涉及到 Android 架构中的三个层次：App 层，Framework 层，Runtime 层。其中，App 层直接调用 Framework 层所封装的方法，而 Framework 层需要通过 Binder 远程调用 Runtime 中 CameraService 的函数。
　　这一部分主要的函数调用逻辑如下图所示：
　　![ ](https://upload-images.jianshu.io/upload_images/2118860-58f30690243b2215.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 　　在 App 中，需要调用打开相机的API如下:

*   CameraCharacteristics：
描述摄像头的各种特性，我们可以通过CameraManager的getCameraCharacteristics(@NonNull String cameraId)方法来获取。
*   CameraDevice：
描述系统摄像头，类似于早期的Camera。
*   CameraCaptureSession：
Session类，当需要拍照、预览等功能时，需要先创建该类的实例，然后通过该实例里的方法进行控制（例如：拍照 capture()）。
*   CaptureRequest：
描述了一次操作请求，拍照、预览等操作都需要先传入CaptureRequest参数，具体的参数控制也是通过CameraRequest的成员变量来设置。
*   CaptureResult：
描述拍照完成后的结果。

　　例如打开camera的java代码：

```　
mCameraManager.openCamera(currentCameraId, stateCallback, backgroundHandler);
```

　　Camera2拍照流程如下所示：

　　![ ](https://upload-images.jianshu.io/upload_images/2118860-a824a2ec1224c232.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 ### 2.2.1 Framework CameraManager  
`/frameworks/base/core/java/android/hardware/camera2/CameraManager.java` 

　　最初的入口就是 CameraManager 的 `openCamera` 方法，但通过代码可以看到，它仅仅是调用了 `openCameraForUid` 方法。

 

```
@RequiresPermission(android.Manifest.permission.CAMERA) public void openCamera(@NonNull String cameraId,
        @NonNull final CameraDevice.StateCallback callback, @Nullable Handler handler) throws CameraAccessException {

    openCameraForUid(cameraId, callback, handler, USE_CALLING_UID);
}
```

 

　　下面的代码**忽略掉了一些参数检查相关操作**，最终主要调用了 `openCameraDeviceUserAsync` 方法。

 
```
public void openCameraForUid(@NonNull String cameraId,
        @NonNull final CameraDevice.StateCallback callback, @Nullable Handler handler, int clientUid) throws CameraAccessException { /* Do something in*/ ...... /* Do something out*/ openCameraDeviceUserAsync(cameraId, callback, handler, clientUid);
}
```
 

　　参考如下注释分析：
```
private CameraDevice openCameraDeviceUserAsync(String cameraId,
        CameraDevice.StateCallback callback, Handler handler, final int uid) throws CameraAccessException {
    CameraCharacteristics characteristics = getCameraCharacteristics(cameraId);
    CameraDevice device = null; synchronized (mLock) {

        ICameraDeviceUser cameraUser = null;

        android.hardware.camera2.impl.CameraDeviceImpl deviceImpl =   //实例化一个 CameraDeviceImpl。构造时传入了 CameraDevice.StateCallback 以及 Handler。
                new android.hardware.camera2.impl.CameraDeviceImpl(  
                    cameraId,
                    callback,
                    handler,
                    characteristics,
                    mContext.getApplicationInfo().targetSdkVersion);

        ICameraDeviceCallbacks callbacks = deviceImpl.getCallbacks(); //获取 CameraDeviceCallback 实例，这是提供给远端连接到 CameraDeviceImpl 的接口。

       try { if (supportsCamera2ApiLocked(cameraId)) {  //HAL3 中走的是这一部分逻辑，主要是从 CameraManagerGlobal 中获取 CameraService 的本地接口，通过它远端调用（采用 Binder 机制） connectDevice 方法连接到相机设备。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　//注意返回的 cameraUser 实际上指向的是远端 CameraDeviceClient 的本地接口。 // Use cameraservice's cameradeviceclient implementation for HAL3.2+ devices
                ICameraService cameraService = CameraManagerGlobal.get().getCameraService(); if (cameraService == null) { throw new ServiceSpecificException(
                        ICameraService.ERROR_DISCONNECTED, "Camera service is currently unavailable");
                }
                cameraUser = cameraService.connectDevice(callbacks, cameraId,
                        mContext.getOpPackageName(), uid);
            } else { // Use legacy camera implementation for HAL1 devices
                int id; try {
                    id = Integer.parseInt(cameraId);
                } catch (NumberFormatException e) { throw new IllegalArgumentException("Expected cameraId to be numeric, but it was: "
                            + cameraId);
                }

                Log.i(TAG, "Using legacy camera HAL.");
                cameraUser = CameraDeviceUserShim.connectBinderShim(callbacks, id);
            }
        } catch (ServiceSpecificException e) { /* Do something in */ ...... /* Do something out */ } // TODO: factor out callback to be non-nested, then move setter to constructor // For now, calling setRemoteDevice will fire initial // onOpened/onUnconfigured callbacks. // This function call may post onDisconnected and throw CAMERA_DISCONNECTED if // cameraUser dies during setup.
        deviceImpl.setRemoteDevice(cameraUser); //将 CameraDeviceClient 设置到 CameraDeviceImpl 中进行管理。
        device = deviceImpl;
    } return device;
}
```

 ### 2.2.2 CameraDeviceImpl  `/frameworks/base/core/java/android/hardware/camera2/Impl/CameraDeviceImpl.java` 
　　在继续向下分析打开相机流程之前，先简单看看调用到的 CameraDeviceImpl 中的`setRemoteDevice` 方法，主要是将获取到的远端设备保存起来：

```
/** * Set remote device, which triggers initial onOpened/onUnconfigured callbacks
 *
 * <p>This function may post onDisconnected and throw CAMERA_DISCONNECTED if remoteDevice dies
 * during setup.</p>
 * */
public void setRemoteDevice(ICameraDeviceUser remoteDevice) throws CameraAccessException { synchronized(mInterfaceLock) { // TODO: Move from decorator to direct binder-mediated exceptions // If setRemoteFailure already called, do nothing
        if (mInError) return;

        mRemoteDevice = new ICameraDeviceUserWrapper(remoteDevice); //通过 ICameraDeviceUserWrapper 给远端设备实例加上一层封装。
 IBinder remoteDeviceBinder = remoteDevice.asBinder(); //使用 Binder 机制的一些基本设置。 // For legacy camera device, remoteDevice is in the same process, and // asBinder returns NULL.
        if (remoteDeviceBinder != null) { try {
                remoteDeviceBinder.linkToDeath(this, /*flag*/ 0); //如果这个binder消失，为标志信息注册一个接收器。
            } catch (RemoteException e) {
                CameraDeviceImpl.this.mDeviceHandler.post(mCallOnDisconnected); throw new CameraAccessException(CameraAccessException.CAMERA_DISCONNECTED, "The camera device has encountered a serious error");
            }
        }

        mDeviceHandler.post(mCallOnOpened); //需此处触发 onOpened 与 onUnconfigured 这两个回调，每个回调都是通过 mDeviceHandler 启用一个新线程来调用的。
 mDeviceHandler.post(mCallOnUnconfigured);
    }
}
```

### 2.2.3 Runtime

通过 Binder 机制，我们远端调用了 `connectDevice` 方法（在 C++ 中称为函数，但说成方法可能更顺口一些），这个方法实现在 CameraService 类中。 

### 2.2.4 CameraService 

`/frameworks/av/services/camera/libcameraservice/CameraService.cpp` 

```
Status CameraService::connectDevice( const sp<hardware::camera2::ICameraDeviceCallbacks>& cameraCb, const String16& cameraId, const String16& clientPackageName, int clientUid, /*out*/ sp<hardware::camera2::ICameraDeviceUser>* device) {

    ATRACE_CALL();
    Status ret = Status::ok();
    String8 id = String8(cameraId);
    sp<CameraDeviceClient> client = nullptr; //此处调用的 connectHelper 方法才真正实现了连接逻辑（HAL1 时最终也调用到这个方法）。需要注意的是，设定的模板类型是 ICameraDeviceCallbacks 以及 CameraDeviceClient。
    ret = connectHelper<hardware::camera2::ICameraDeviceCallbacks,CameraDeviceClient>(cameraCb, id,
            CAMERA_HAL_API_VERSION_UNSPECIFIED, clientPackageName,
            clientUid, USE_CALLING_PID, API_2, /*legacyMode*/ false, /*shimUpdateOnly*/ false, /*out*/client); if(!ret.isOk()) {
        logRejected(id, getCallingPid(), String8(clientPackageName),
                ret.toString8()); return ret;
    } *device = client; //client 指向的类型是 CameraDeviceClient，其实例则是最终的返回结果。
    return ret;
}
```

　　`connectHelper` 内容较多，忽略掉我们还无需关注的地方分析：

```
template<class CALLBACK, class CLIENT> Status CameraService::connectHelper(const sp<CALLBACK>& cameraCb, const String8& cameraId, int halVersion, const String16& clientPackageName, int clientUid, int clientPid,
        apiLevel effectiveApiLevel, bool legacyMode, bool shimUpdateOnly, /*out*/sp<CLIENT>& device) {
    binder::Status ret = binder::Status::ok();

    String8 clientName8(clientPackageName); /* Do something in */ ...... /* Do something out */ sp<BasicClient> tmp = nullptr; //调用 makeClient 生成 CameraDeviceClient 实例。
        if(!(ret = makeClient(this, cameraCb, clientPackageName, cameraId, facing, clientPid,
                clientUid, getpid(), legacyMode, halVersion, deviceVersion, effectiveApiLevel, /*out*/&tmp)).isOk()) { return ret;
        } //初始化 CLIENT 实例。注意此处的模板类型 CLIENT 即是 CameraDeviceClient，传入的参数 mCameraProviderManager 则是与 HAL service 有关。 
        client = static_cast<CLIENT*>(tmp.get());

        LOG_ALWAYS_FATAL_IF(client.get() == nullptr, "%s: CameraService in invalid state",
                __FUNCTION__);

        err = client->initialize(mCameraProviderManager); /* Do something in */ ...... /* Do something out */

    // Important: release the mutex here so the client can call back into the service from its // destructor (can be at the end of the call)
    device = client; return ret;
} 
```

　　makeClient 主要是根据 API 版本以及 HAL 版本来选择生成具体的 Client 实例，Client 就沿着前面分析下来的路径返回到 CameraDeviceImpl 实例中，被保存到 mRemoteDevice。

```
Status CameraService::makeClient(const sp<CameraService>& cameraService, const sp<IInterface>& cameraCb, const String16& packageName, const String8& cameraId, int facing, int clientPid, uid_t clientUid, int servicePid, bool legacyMode, int halVersion, int deviceVersion, apiLevel effectiveApiLevel, /*out*/sp<BasicClient>* client) { if (halVersion < 0 || halVersion == deviceVersion) { // Default path: HAL version is unspecified by caller, create CameraClient // based on device version reported by the HAL.
        switch(deviceVersion) { case CAMERA_DEVICE_API_VERSION_1_0: /* Do something in */ ...... /* Do something out */
          case CAMERA_DEVICE_API_VERSION_3_0: case CAMERA_DEVICE_API_VERSION_3_1: case CAMERA_DEVICE_API_VERSION_3_2: case CAMERA_DEVICE_API_VERSION_3_3: case CAMERA_DEVICE_API_VERSION_3_4: if (effectiveApiLevel == API_1) { // Camera1 API route
                sp<ICameraClient> tmp = static_cast<ICameraClient*>(cameraCb.get()); *client = new Camera2Client(cameraService, tmp, packageName, cameraIdToInt(cameraId),
                        facing, clientPid, clientUid, servicePid, legacyMode);
            } else { // Camera2 API route : 实例化了 CameraDeviceClient 类作为 Client（注意此处构造传入了 ICameraDeviceCallbacks，这是连接到 CameraDeviceImpl 的远端回调）
                sp<hardware::camera2::ICameraDeviceCallbacks> tmp = static_cast<hardware::camera2::ICameraDeviceCallbacks*>(cameraCb.get()); *client = new CameraDeviceClient(cameraService, tmp, packageName, cameraId,
                        facing, clientPid, clientUid, servicePid);
            } break; default: // Should not be reachable
            ALOGE("Unknown camera device HAL version: %d", deviceVersion); return STATUS_ERROR_FMT(ERROR_INVALID_OPERATION, "Camera device \"%s\" has unknown HAL version %d",
                    cameraId.string(), deviceVersion);
        }
    } else { /* Do something in */ ...... /* Do something out */ } return Status::ok();
}
```

　　至此，打开相机流程中，从 App 到 CameraService 的调用逻辑基本上就算走完了。

　　简图总结:

　　![image](https://upload-images.jianshu.io/upload_images/2118860-63410bab66567e39.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　Ps：
-  CameraManagerGlobal 是真正的实现层，它与 JAVA 层的 CameraService 创建连接，从而创建相机的连路。
- CameraDeviceImpl 相当于运行上下文，它取代了 Android N 之前的 JNI 层。

## 2.3 从 CameraService 到 HAL Service

由于 Android O 中加入了 Treble 机制，CameraServer 一端主体为 CameraService，它将会寻找现存的 Provider service，将其加入到内部的 CameraProviderManager 中进行管理，相关操作都是通过远端调用进行的。
而 Provider service 一端的主体为 CameraProvider，它在初始化时就已经连接到 libhardware 的 Camera HAL 实现层，并以 CameraModule 来进行管理。
进程的启动后，连路的 “载体” 就搭建完成了（需要注意，此时 QCamera3HWI 还未创建），可用下图简单表示：
　　![ ](https://upload-images.jianshu.io/upload_images/2118860-08d08933011873c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　　而在打开相机时，该层的完整连路会被创建出来，主要调用逻辑如下图：

　　![ ](https://upload-images.jianshu.io/upload_images/2118860-20989470bcc075f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

　　　　上回讲到，在 CameraService::makeClient 中，实例化了一个 CameraDeviceClient。现在我们就从它的构造函数开始，继续探索打开相机的流程。
这一部分主要活动在 Runtime 层，这里分成 CameraService 与 HAL Service 两侧来分析。

### 2.3.1 CameraDeviceClient  
 
`frameworks\av\services\camera\libcameraservice\api2\CameraDeviceClient.cpp`

```
CameraDeviceClient::CameraDeviceClient(const sp<CameraService>& cameraService, const sp<hardware::camera2::ICameraDeviceCallbacks>& remoteCallback, const String16& clientPackageName, const String8& cameraId, int cameraFacing, int clientPid,
        uid_t clientUid, int servicePid) :
    Camera2ClientBase(cameraService, remoteCallback, clientPackageName,
                cameraId, cameraFacing, clientPid, clientUid, servicePid),  //继承它的父类 Camera2ClientBase 
    mInputStream(),
    mStreamingRequestId(REQUEST_ID_NONE),
    mRequestIdCounter(0),
    mPrivilegedClient(false) { char value[PROPERTY_VALUE_MAX];
    property_get("persist.camera.privapp.list", value, "");
    String16 packagelist(value); if (packagelist.contains(clientPackageName.string())) {
        mPrivilegedClient = true;
    }

    ATRACE_CALL();
    ALOGI("CameraDeviceClient %s: Opened", cameraId.string());
}
```
　　CameraService 在创建 CameraDeviceClient 之后，会调用它的初始化函数：

```
//对外提供调用的初始化函数接口 initialize。
status_t CameraDeviceClient::initialize(sp<CameraProviderManager> manager) { return initializeImpl(manager);
} //初始化的具体实现函数，模板 TProviderPtr 在此处即是 CameraProviderManager 类。
template<typename TProviderPtr>
//首先将父类初始化，注意此处传入了 CameraProviderManager。
status_t CameraDeviceClient::initializeImpl(TProviderPtr providerPtr) {
    ATRACE_CALL();
    status_t res;

    res = Camera2ClientBase::initialize(providerPtr); if (res != OK) { return res;
    } //这里是关于 FrameProcessor 的创建与初始化配置等等
 String8 threadName;
    mFrameProcessor = new FrameProcessorBase(mDevice);
    threadName = String8::format("CDU-%s-FrameProc", mCameraIdStr.string());
    mFrameProcessor->run(threadName.string());

    mFrameProcessor->registerListener(FRAME_PROCESSOR_LISTENER_MIN_ID,
                                      FRAME_PROCESSOR_LISTENER_MAX_ID, /*listener*/this, /*sendPartials*/true); return OK;
}
```

### 2.3.2 Camera2ClientBase

`frameworks\av\services\camera\libcameraservice\common\Camera2ClientBase.cpp`**

```
template <typename TClientBase>　//模板 TClientBase，在 CameraDeviceClient 继承 Camera2ClientBase 时被指定为 CameraDeviceClientBase。
Camera2ClientBase<TClientBase>::Camera2ClientBase( //构造的相关参数，以及初始化列表，这里面需要注意 TCamCallbacks 在 CameraDeviceClientBase 中被指定为了 ICameraDeviceCallbacks。 
        const sp<CameraService>& cameraService, const sp<TCamCallbacks>& remoteCallback, const String16& clientPackageName, const String8& cameraId, int cameraFacing, int clientPid,
        uid_t clientUid, int servicePid):
        TClientBase(cameraService, remoteCallback, clientPackageName,
                cameraId, cameraFacing, clientPid, clientUid, servicePid),
        mSharedCameraCallbacks(remoteCallback),
        mDeviceVersion(cameraService->getDeviceVersion(TClientBase::mCameraIdStr)),
        mDeviceActive(false)
{
    ALOGI("Camera %s: Opened. Client: %s (PID %d, UID %d)", cameraId.string(),
            String8(clientPackageName).string(), clientPid, clientUid);

    mInitialClientPid = clientPid;
    mDevice = new Camera3Device(cameraId); //创建了一个 Camera3Device。
    LOG_ALWAYS_FATAL_IF(mDevice == 0, "Device should never be NULL here.");
}
```

　　回去再看看初始化函数：
```
template <typename TClientBase> //初始化函数接口，真正的实现部分在 initializeImpl 中。
status_t Camera2ClientBase<TClientBase>::initialize(sp<CameraProviderManager> manager) { return initializeImpl(manager);
} //TClientBase 对应 CameraDeviceClientBase，而 TProviderPtr 对应的是 CameraProviderManager。
template <typename TClientBase> template <typename TProviderPtr> status_t Camera2ClientBase<TClientBase>::initializeImpl(TProviderPtr providerPtr) {
    ATRACE_CALL();
    ALOGV("%s: Initializing client for camera %s", __FUNCTION__,
          TClientBase::mCameraIdStr.string());
    status_t res; // Verify ops permissions
    res = TClientBase::startCameraOps(); //调用 CameraDeviceClientBase 的 startCameraOps 方法，检查 ops 的权限。
    if (res != OK) { return res;
    } if (mDevice == NULL) {
        ALOGE("%s: Camera %s: No device connected",
                __FUNCTION__, TClientBase::mCameraIdStr.string()); return NO_INIT;
    }

    res = mDevice->initialize(providerPtr); //初始化 Camera3Device 的实例，注意此处传入了 CameraProviderManager。
    if (res != OK) {
        ALOGE("%s: Camera %s: unable to initialize device: %s (%d)",
                __FUNCTION__, TClientBase::mCameraIdStr.string(), strerror(-res), res); return res;
    } //在 Camera3Device 实例中设置 Notify 回调。
    wp<CameraDeviceBase::NotificationListener> weakThis(this);
    res = mDevice->setNotifyCallback(weakThis); return OK;
}
```


### 2.3.3 Camera3Device

`frameworks\av\services\camera\libcameraservice\device3\Camera3Device.cpp`**

```
Camera3Device::Camera3Device(const String8 &id):
        mId(id),
        mOperatingMode(NO_MODE),
        mIsConstrainedHighSpeedConfiguration(false),
        mStatus(STATUS_UNINITIALIZED),
        mStatusWaiters(0),
        mUsePartialResult(false),
        mNumPartialResults(1),
        mTimestampOffset(0),
        mNextResultFrameNumber(0),
        mNextReprocessResultFrameNumber(0),
        mNextShutterFrameNumber(0),
        mNextReprocessShutterFrameNumber(0),
        mListener(NULL),
        mVendorTagId(CAMERA_METADATA_INVALID_VENDOR_ID)
{
    ATRACE_CALL();
　　//在这个观察构造函数中设定了两个回调接口：
    camera3_callback_ops::notify = &sNotify;
    camera3_callback_ops::process_capture_result = &sProcessCaptureResult;
    ALOGV("%s: Created device for camera %s", __FUNCTION__, mId.string());
}
``` 

　　其初始化函数篇幅较长，这里省略掉了关于 RequestMetadataQueue 的相关操作。

```
status_t Camera3Device::initialize(sp<CameraProviderManager> manager) {
    ATRACE_CALL();
    Mutex::Autolock il(mInterfaceLock);
    Mutex::Autolock l(mLock);

    ALOGV("%s: Initializing HIDL device for camera %s", __FUNCTION__, mId.string()); if (mStatus != STATUS_UNINITIALIZED) {
        CLOGE("Already initialized!"); return INVALID_OPERATION;
    } if (manager == nullptr) return INVALID_OPERATION;

    sp<ICameraDeviceSession> session;
    ATRACE_BEGIN("CameraHal::openSession");
    status_t res = manager->openSession(mId.string(), this, //调用CameraProviderManager的openSession方法，开启了远端的**Session**
            /*out*/ &session);
    ATRACE_END(); if (res != OK) {
        SET_ERR_L("Could not open camera session: %s (%d)", strerror(-res), res); return res;
    } /* Do something in */ ...... /* Do something out */

    return initializeCommonLocked();
}
```

### 2.3.4 CameraProviderManager

`frameworks\av\services\camera\libcameraservice\common\CameraProviderManager.cpp` 

```
status_t CameraProviderManager::openSession(const std::string &id, const sp<hardware::camera::device::V3_2::ICameraDeviceCallback>& callback, /*out*/ sp<hardware::camera::device::V3_2::ICameraDeviceSession> *session) {

    std::lock_guard<std::mutex> lock(mInterfaceMutex);

    auto deviceInfo = findDeviceInfoLocked(id, //首先调用 findDeviceInfoLocked，获取 HAL3 相关的 DeviceInfo3
            /*minVersion*/ {3,0}, /*maxVersion*/ {4,0}); if (deviceInfo == nullptr) return NAME_NOT_FOUND;

    auto *deviceInfo3 = static_cast<ProviderInfo::DeviceInfo3*>(deviceInfo);

    Status status;
    hardware::Return<void> ret; //通过远端调用 CameraDevice 的 open 方法，创建 CameraDeviceSession 实例并将其本地调用接口通过入参 session 返回。
    ret = deviceInfo3->mInterface->open(callback, [&status, &session]
            (Status s, const sp<device::V3_2::ICameraDeviceSession>& cameraSession) {
                status = s; if (status == Status::OK) { *session = cameraSession;
                }
            }); if (!ret.isOk()) {
        ALOGE("%s: Transaction error opening a session for camera device %s: %s",
                __FUNCTION__, id.c_str(), ret.description().c_str()); return DEAD_OBJECT;
    } return mapToStatusT(status);
}
```

 ### 2.3.5 CameraDevice

`hardware\interfaces\camera\device\3.2\default\CameraDevice.cpp` 

　　CameraDevice 的实例实际上在初始化 HAL Service 之后就存在了。 前面说到，通过 CameraProviderManager 中的 `deviceInfo` 接口，调用**远端 CameraDevice 实例**的 `open` 方法，下面就来看看它的代码实现：

```
Return<void> CameraDevice::open(const sp<ICameraDeviceCallback>& callback, open_cb _hidl_cb)  {
    Status status = initStatus();
    sp<CameraDeviceSession> session = nullptr; if (callback == nullptr) {
        ALOGE("%s: cannot open camera %s. callback is null!",
                __FUNCTION__, mCameraId.c_str());
        _hidl_cb(Status::ILLEGAL_ARGUMENT, nullptr); return Void();
    } if (status != Status::OK) { /* Do something in */ ...... /* Do something out */ } else {
        mLock.lock(); /* Do something in */ ...... /* Do something out */

        /** Open HAL device */ status_t res;
        camera3_device_t *device;

        ATRACE_BEGIN("camera3->open");
        res = mModule->open(mCameraId.c_str(), //注意 mModule 是在 HAL Service 初始化时就已经配置好的，它对从libhardware库中加载的 Camera HAL 接口进行了一层封装，从这里往下就会一路走到 QCamera3HWI 的构造流程去。
                reinterpret_cast<hw_device_t**>(&device));
        ATRACE_END(); /* Do something in */ ...... /* Do something out */

　　　　 //创建 session 并让内部成员 mSession 持有，具体实现的函数为 creatSession。
        session = createSession(
                device, info.static_camera_characteristics, callback); /* Do something in */ ...... /* Do something out */ mSession = session;

        IF_ALOGV() {
            session->getInterface()->interfaceChain([](
                ::android::hardware::hidl_vec<::android::hardware::hidl_string> interfaceChain) {
                    ALOGV("Session interface chain:"); for (auto iface : interfaceChain) {
                        ALOGV("  %s", iface.c_str());
                    }
                });
        }
        mLock.unlock();
    }
    _hidl_cb(status, session->getInterface()); return Void();
}
```

　　而 createSession 中直接创建了一个 CameraDeviceSession。当然在其构造函数中会调用内部的初始化函数，然后会进入 HAL 接口层 QCamera3HWI 的初始化流程，至此，从 CameraService 到 HAL Service 这一部分的打开相机流程就基本走通了。
　　简图总结:

　　![ ](https://upload-images.jianshu.io/upload_images/2118860-84a480f85ec914dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.4 从  HAL Service 到 Camera  HAL 

　在 HAL3 中，Camera HAL 的接口转化层（以及流解析层）由 QCamera3HardwareInterface 担当，而接口层与实现层与 HAL1 中基本没什么差别，都是在 mm_camera_interface.c 与 mm_camera.c 中。
　　那么接口转化层的实例是何时创建的，又是怎么初始化的，创建它的时候，与接口层、实现层又有什么交互？通过下图展示的主要调用流程:

　　![image](https://upload-images.jianshu.io/upload_images/2118860-c00b88928cd68302.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.4.1 CameraModule(HAL Service) 

 `hardware\interfaces\camera\common\1.0\default\CameraModule.cpp` 

　　　　上回说到，CameraDevice::open 的实现中，调用了 mModule->open，即 CameraModule::open，通过代码来看，它做的事并不多，主要是调用 mModule->common.methods->open，来进入下一层级的流程。
　　　　而这里则需要注意了，open 是一个函数指针，它指向的是 QCamera2Factory 的 camera_device_open 方法，至于为什么和 QCamera2Factory 有关，这就要回头看 HAL Service 的启动初始化流程了。

```
int CameraModule::open(const char* id, struct hw_device_t** device) { int res;
    ATRACE_BEGIN("camera_module->open");
    res = filterOpenErrorCode(mModule->common.methods->open(&mModule->common, id, device));
    ATRACE_END(); return res;
}
```

### 2.4.2 QCamera2Factory（Camera HAL）

```
/*===========================================================================
 * FUNCTION   : camera_device_open
 *
 * DESCRIPTION: static function to open a camera device by its ID
 *
 * PARAMETERS :
 *   @camera_id : camera ID
 *   @hw_device : ptr to struct storing camera hardware device info
 *
 * RETURN     : int32_t type of status
 *              NO_ERROR  -- success
 *              none-zero failure code
 *==========================================================================*/
int QCamera2Factory::camera_device_open( const struct hw_module_t *module, const char *id,
    struct hw_device_t **hw_device)
{ /* Do something in */ ...... /* Do something out */ #ifdef QCAMERA_HAL1_SUPPORT //注意到这里通过宏定义添加了对 HAL1 的兼容操作。实际上是要调用 cameraDeviceOpen 来进行下一步操作。
    if(gQCameraMuxer)
        rc =  gQCameraMuxer->camera_device_open(module, id, hw_device); else #endif
        rc = gQCamera2Factory->cameraDeviceOpen(atoi(id), hw_device); return rc;
}

struct hw_module_methods_t QCamera2Factory::mModuleMethods = {
    .open = QCamera2Factory::camera_device_open, //这里就将前面所说的 open 函数指针指定为了 camera_device_open 这个方法。
};
```

　　`cameraDeviceOpen` 的工作：
```
/*===========================================================================
 * FUNCTION   : cameraDeviceOpen
 *
 * DESCRIPTION: open a camera device with its ID
 *
 * PARAMETERS :
 *   @camera_id : camera ID
 *   @hw_device : ptr to struct storing camera hardware device info
 *
 * RETURN     : int32_t type of status
 *              NO_ERROR  -- success
 *              none-zero failure code
 *==========================================================================*/
int QCamera2Factory::cameraDeviceOpen(int camera_id,
                    struct hw_device_t **hw_device)
{ /* Do something in */ ...... /* Do something out */

    if ( mHalDescriptors[camera_id].device_version == CAMERA_DEVICE_API_VERSION_3_0 ) {
        QCamera3HardwareInterface *hw = new QCamera3HardwareInterface(mHalDescriptors[camera_id].cameraId, //首先创建了 QCamera3HardwareInterface 的实例。
 mCallbacks); if (!hw) {
            LOGE("Allocation of hardware interface failed"); return NO_MEMORY;
        }
        rc = hw->openCamera(hw_device); //调用实例的 openCamera 方法。
        if (rc != 0) {
            delete hw;
        }
    } /* Do something in */ ...... /* Do something out */

    return rc;
}
```

### 2.4.3 QCamera3HardwareInterface 
 `hardware\qcom\camera\qcamera2\hal3\QCamera3HWI.cpp` 

　　首先需要注意的是内部成员 `mCameraOps` 的定义。 在构造实例时，有 `mCameraDevice.ops = &mCameraOps;`(关键点)

```
camera3_device_ops_t QCamera3HardwareInterface::mCameraOps = {
    .initialize = QCamera3HardwareInterface::initialize,
    .configure_streams = QCamera3HardwareInterface::configure_streams,
    .register_stream_buffers = NULL,
    .construct_default_request_settings = QCamera3HardwareInterface::construct_default_request_settings,
    .process_capture_request = QCamera3HardwareInterface::process_capture_request,
    .get_metadata_vendor_tag_ops = NULL,
    .dump = QCamera3HardwareInterface::dump,
    .flush = QCamera3HardwareInterface::flush,
    .reserved = {0},
};
```

　　再来继续看看 `openCamera` 实现：

```
int QCamera3HardwareInterface::openCamera(struct hw_device_t **hw_device)
{ /* Do something in */ ...... /* Do something out */ rc = openCamera(); //调用另一个 openCamera 方法，这是具体实现的部分。
    if (rc == 0) { *hw_device = &mCameraDevice.common; //打开相机成功后，将设备结构中的 common 部分通过双重指针 hw_device 返回。
    } else
        *hw_device = NULL; /* Do something in */ ...... /* Do something out */
    return rc;
} int QCamera3HardwareInterface::openCamera()
{ /* Do something in */ ...... /* Do something out */ rc = camera_open((uint8_t)mCameraId, &mCameraHandle);  //这里就开始进入接口层了，调用的是接口层中的 camera_open 接口。注意此处获取到了 mCameraHandle.

    /* Do something in */ ...... /* Do something out */ rc = mCameraHandle->ops->register_event_notify(mCameraHandle->camera_handle, //注意这里传入了一个 camEvtHandle
            camEvtHandle, (void *)this); /* Do something in */ ...... /* Do something out */ rc = mCameraHandle->ops->get_session_id(mCameraHandle->camera_handle, 
        &sessionId[mCameraId]); /* Do something in */ ...... /* Do something out */

    return NO_ERROR;
}
```

　　上面是接口转化层中，关于 `openCamera` 的部分，下面继续看看它的初始化函数。 在前面已经分析过，创建 CameraDeviceSession 实例时，会调用它内部的初始化方法，而这其中包含了调用 QCamera3HWI 的初始化方法 **`initialize`**
```
int QCamera3HardwareInterface::initialize(const struct camera3_device *device, const camera3_callback_ops_t *callback_ops)
{
    LOGD("E");
    QCamera3HardwareInterface *hw = reinterpret_cast<QCamera3HardwareInterface *>(device->priv); if (!hw) {
        LOGE("NULL camera device"); return -ENODEV;
    } int rc = hw->initialize(callback_ops); //调用了真正实现的初始化逻辑的函数
    LOGD("X"); return rc;
} int QCamera3HardwareInterface::initialize( const struct camera3_callback_ops *callback_ops)
{
    ATRACE_CALL(); int rc;

    LOGI("E :mCameraId = %d mState = %d", mCameraId, mState);
    pthread_mutex_lock(&mMutex); // Validate current state
    switch (mState) { case OPENED: /* valid state */
            break; default:
            LOGE("Invalid state %d", mState);
            rc = -ENODEV; goto err1;
    }

    rc = initParameters(); //参数（mParameters）初始化，注意这里的参数和 CameraParameter 是不同的，它是 metadata_buffer 相关参数的结构。
    if (rc < 0) {
        LOGE("initParamters failed %d", rc); goto err1;
    }
    mCallbackOps = callback_ops; //这里将 camera3_call_back_ops 与 mCallbackOps 关联了起来。
 mChannelHandle = mCameraHandle->ops->add_channel( //获取 mChannelHandle 这一句柄，调用的方法实际是 mm_camera_interface.c 中的 mm_camera_intf_add_channel。
            mCameraHandle->camera_handle, NULL, NULL, this); if (mChannelHandle == 0) {
        LOGE("add_channel failed");
        rc = -ENOMEM;
        pthread_mutex_unlock(&mMutex); return rc;
    }

    pthread_mutex_unlock(&mMutex);
    mCameraInitialized = true;
    mState = INITIALIZED;
    LOGI("X"); return 0;

err1:
    pthread_mutex_unlock(&mMutex); return rc;
}
```
### 2.4.4 mm_camera_interface.c(接口层) 

`hardware\qcom\camera\qcamera2\stack\mm-camera-interface\src\mm_camera_interface.c` 

　　　　`camera_open` 中干的事也不多，**省略掉了关于为 `cam_obj` 分配内存以及初始化的部分**。实际上是调用实现层中的 `mm_camera_open`来真正实现打开相机设备的操作，设备的各种信息都填充到 `cam_obj` 结构中。

```
int32_t camera_open(uint8_t camera_idx, mm_camera_vtbl_t **camera_vtbl)
{
    int32_t rc = 0;
    mm_camera_obj_t *cam_obj = NULL; /* Do something in */ ...... /* Do something out */ 
    rc = mm_camera_open(cam_obj); /* Do something in */ ...... /* Do something out */ 
}
```


而关于初始化时调用的 `mm_camera_intf_add_channel` 代码如下:

```
static uint32_t mm_camera_intf_add_channel(uint32_t camera_handle,
                                           mm_camera_channel_attr_t *attr,
                                           mm_camera_buf_notify_t channel_cb, void *userdata)
{
    uint32_t ch_id = 0;
    mm_camera_obj_t * my_obj = NULL;

    LOGD("E camera_handler = %d", camera_handle);
    pthread_mutex_lock(&g_intf_lock);
    my_obj = mm_camera_util_get_camera_by_handler(camera_handle); if(my_obj) {
        pthread_mutex_lock(&my_obj->cam_lock);
        pthread_mutex_unlock(&g_intf_lock);
        ch_id = mm_camera_add_channel(my_obj, attr, channel_cb, userdata); //通过调用实现层的 mm_camera_add_channel 来获取一个 channel id，也就是其句柄。
    } else {
        pthread_mutex_unlock(&g_intf_lock);
    }
    LOGD("X ch_id = %d", ch_id); return ch_id;
}
```
### 2.4.5 mm_camera.c(实现层)

`hardware\qcom\camera\qcamera2\stack\mm-camera-interface\src\mm_camera.c` 

　终于来到最底层的实现了，`mm_camera_open` 主要工作是填充 `my_obj`，并且启动、初始化一些线程相关的东西，关于线程的部分我这里就省略掉了。

```
int32_t mm_camera_open(mm_camera_obj_t *my_obj)
{ char dev_name[MM_CAMERA_DEV_NAME_LEN];
    int32_t rc = 0;
    int8_t n_try=MM_CAMERA_DEV_OPEN_TRIES;
    uint8_t sleep_msec=MM_CAMERA_DEV_OPEN_RETRY_SLEEP; int cam_idx = 0; const char *dev_name_value = NULL; int l_errno = 0;
    pthread_condattr_t cond_attr;

    LOGD("begin\n"); if (NULL == my_obj) { goto on_error;
    }
    dev_name_value = mm_camera_util_get_dev_name(my_obj->my_hdl); //此处调用的函数是为了获取 my_obj 的句柄，这里不深入分析。
    if (NULL == dev_name_value) { goto on_error;
    }
    snprintf(dev_name, sizeof(dev_name), "/dev/%s",
             dev_name_value);
    sscanf(dev_name, "/dev/video%d", &cam_idx);
    LOGD("dev name = %s, cam_idx = %d", dev_name, cam_idx); do{
        n_try--;
        errno = 0;
        my_obj->ctrl_fd = open(dev_name, O_RDWR | O_NONBLOCK); //读取设备文件的文件描述符，存到 my_obj->ctrl_fd 中。
        l_errno = errno;
        LOGD("ctrl_fd = %d, errno == %d", my_obj->ctrl_fd, l_errno); if((my_obj->ctrl_fd >= 0) || (errno != EIO && errno != ETIMEDOUT) || (n_try <= 0 )) { break;
        }
        LOGE("Failed with %s error, retrying after %d milli-seconds",
              strerror(errno), sleep_msec);
        usleep(sleep_msec * 1000U);
    }while (n_try > 0); if (my_obj->ctrl_fd < 0) {
        LOGE("cannot open control fd of '%s' (%s)\n",
                  dev_name, strerror(l_errno)); if (l_errno == EBUSY)
            rc = -EUSERS; else rc = -1; goto on_error;
    } else {
        mm_camera_get_session_id(my_obj, &my_obj->sessionid); //成功获取到文件描述符后，就要获取 session 的 id 了。
        LOGH("Camera Opened id = %d sessionid = %d", cam_idx, my_obj->sessionid);
    } /* Do something in */ ...... /* Do something out */

    /* unlock cam_lock, we need release global intf_lock in camera_open(),
     * in order not block operation of other Camera in dual camera use case.*/ pthread_mutex_unlock(&my_obj->cam_lock); return rc;
}
```

　　初始化的相关部分，`mm_camera_add_channel` 代码如下：

```
uint32_t mm_camera_add_channel(mm_camera_obj_t *my_obj,
                               mm_camera_channel_attr_t *attr,
                               mm_camera_buf_notify_t channel_cb, void *userdata)
{
    mm_channel_t *ch_obj = NULL;
    uint8_t ch_idx = 0;
    uint32_t ch_hdl = 0; //从现有的 Channel 中找到第一个状态为 NOTUSED 的，获取到 ch_obj 中
    for(ch_idx = 0; ch_idx < MM_CAMERA_CHANNEL_MAX; ch_idx++) { if (MM_CHANNEL_STATE_NOTUSED == my_obj->ch[ch_idx].state) {
            ch_obj = &my_obj->ch[ch_idx]; break;
        }
    }

　　/*初始化 ch_obj 结构。首先调用 mm_camera_util_generate_handler 为其生成一个句柄（也是该函数的返回值），
　　 *然后将状态设置为 STOPPED，注意这里还保存了 my_obj 的指针及其 session id，最后调用 mm_channel_init 完成了 Channel 的初始化。*/
    if (NULL != ch_obj) { /* initialize channel obj */ memset(ch_obj, 0, sizeof(mm_channel_t));
        ch_hdl = mm_camera_util_generate_handler(ch_idx);
        ch_obj->my_hdl = ch_hdl;
        ch_obj->state = MM_CHANNEL_STATE_STOPPED;
        ch_obj->cam_obj = my_obj;
        pthread_mutex_init(&ch_obj->ch_lock, NULL);
        ch_obj->sessionid = my_obj->sessionid;
        mm_channel_init(ch_obj, attr, channel_cb, userdata);
    }

    pthread_mutex_unlock(&my_obj->cam_lock); return ch_hdl;
}
```

　　简图总结:

![image](https://upload-images.jianshu.io/upload_images/2118860-32e6d2e82d067665.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  　　总而言之，上面这一顿操作下来后，相机从上到下的整个连路就已经打通，接下来应该只要 APP 按照流程下发 Preview 的 Request 就可以开始获取预览数据了。

# 三、核心概念：Request 

　　request是贯穿camera2数据处理流程最为重要的概念，应用框架是通过向camera子系统发送request来获取其想要的result。
　
　request有下述几个重要特征：
- 一个request可以对应一系列的result
- request应当包含所有必要的配置信息，存放于metadata中。如：分辨率和像素格式；sensor、镜头、闪光等的控制信息；3A 操作模式；RAW 到 YUV 处理控件；以及统计信息的生成等。 
- request需要携带对应的surface（也就是框架里面的stream），用于接收返回的图像。 
- 多个request可以同时处于in-flight状态，并且submit request是non-blocking方式的。也就是说，上一个request没有处理完，也可以submit新的request。 
- 队列中request的处理总是按照FIFO的形式。 
- snapshot的request比preview的request拥有更高的优先级。 

## 3.1request的整体处理流程图 

　　![ ](https://upload-images.jianshu.io/upload_images/2118860-589de1e0186da62b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

-  **open 流程（黑色箭头线条）**

    CameraManager注册AvailabilityCallback回调，用于接收相机设备的可用性状态变更的通知。
    CameraManager通过调用getCameraIdList()获取到当前可用的camera id，通过getCameraCharacteristcs()函数获取到指定相机设备的特性。
    CameraManager调用openCamera()打开指定相机设备，并返回一个CameraDevice对象，后续通过该CameraDevice对象操控具体的相机设备。
    使用CameraDevice对象的createCaptureSession()创建一个session，数据请求（预览、拍照等）都是通过session进行。在创建session时，需要提供Surface作为参数，用于接收返回的图像。

- **configure stream流程（蓝色箭头线条）**

    申请Surface，如上图的OUTPUT STREAMS DESTINATIONS框，用于在创建session时作为参数，接收session返回的图像。
    创建session后，surface会被配置成框架的stream。在框架中，stream定义了图像的size及format。
    每个request都需要携带target surface用于指定返回的图像是归属到哪个被configure的stream的。

 - **request处理流程（橙色箭头线条）**

    CameraDevice对象通过createCaptureRequest()来创建request，每个reqeust都需要有surface和settings（settings就是metadata，request包含的所有配置信息都是放在metadata中的）。
    使用session的capture()、captureBurst()、setStreamingRequest()、setStreamingBurst()等api可以将request发送到框架。
    预览的request，通过setStreamingRequest()、setStreamingBurst()发送，仅调用一次。将request set到repeating request list里面。只要pending request queue里面没有request，就将repeating list里面的request copy到pending queue里面。
    拍照的request，通过capture()、captureBurst()发送，每次需要拍照都会调用。每次触发，都会直接将request入到pending request queue里面，所以拍照的request比预览的request的优先级更高。
    in-progress queue代表当前正在处理的request的queue，每处理完一个，都会从pending queue里面拿出来一个新的request放到这里。

- **数据返回流程（紫色箭头线条）**

    硬件层面返回的数据会放到result里面返回，会通过session的capture callback回调响应。 

##3.2 request在HAL的处理方式 

- 1.framework发送异步的request到hal。
- 2.hal必须顺序处理request，对于每一个request都要返回timestamp（shutter，也就是帧的生成时间）、metadata、image buffers。
- 3.对于request引用的每一类steam，必须按FIFO的方式返回result。比如：对于预览的stream，result id 9必须要先于result id 10返回。但是拍照的stream，当前可以只返回到result id 7，因为拍照和预览用的stream不一样。
- 4.hal需要的信息都通过request携带的metadata接收，hal需要返回的信息都通过result携带的metadata返回。

　　HAL处理request的整体流程如下图。
　　![ ](https://upload-images.jianshu.io/upload_images/2118860-71bb6055730b103b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 - **request处理流程（黑色箭头线条）**

    framework异步地submit request到hal，hal依次处理，并返回result。
    每个被submit到hal的request都必须携带stream。stream分为input stream和output stream：input stream对应的buffer是已有图像数据的buffer，hal对这些buffer进行reprocess；output stream对应的buffer是empty buffer，hal将生成的图像数据填充的这些buffer里面。

- **input stream处理流程（图像的INPUT STREAM** **1****）**

    request携带input stream及input buffer到hal。
    hal进行reprocess，然后新的图像数据重新填充到buffer里面，返回到framework。

- **output stream处理流程（图像的OUTPUT STREAM** **1****…N）** 

    request携带output stream及output buffer到hal。
    hal经过一系列模块的的处理，将图像数据写到buffer中，返回到frameowork。 

>原文链接：https://www.cnblogs.com/blogs-of-lxl/p/10651611.html

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

 至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 
