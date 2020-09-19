 

##### 和您一起终身学习，这里是程序员Android 


本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>1.DisplayManagerService的启动
>2.DisplayManagerService 作用
>3.DisplayManagerService 继承关系
>4.DisplayManagerService 的构造方法
>5.DisplayManagerService 的onStart 方法
>6.DisplayManagerService 的onBootPhase(int phase) 方法

**前言**
本文涉及代码类路径如下，后续涉及代码内容，请参考以下目录。

```

frameworks\base\services\java\com\android\server\SystemServer.java

frameworks\base\services\core\java\com\android\server\display\DisplayManagerService.java

```

#1.DisplayManagerService的启动

**DisplayManagerService** 是有 **SystemServer** 
在`startBootstrapServices` 引导阶段中通过`startService`启动，代码如下：

```
public final class SystemServer {
    ...

    private void startBootstrapServices() {
        ...
        // Display manager is needed to provide display metrics before package manager
        // starts up.
        traceBeginAndSlog("StartDisplayManager");
        //1.启动 DisplayManagerService
        mDisplayManagerService = mSystemServiceManager.startService(DisplayManagerService.class);
        traceEnd();
        ...
     }
    ...

}
```
#2.DisplayManagerService 作用

**DisplayManagerService** 用来管理显示的生命周期，它决定如何根据当前连接的物理显示设备控制其逻辑显示，并且在状态更改时，向系统和应用程序发送通知，等等。

**DisplayAdapter** 是 **DisplayManagerService**  所依赖的集合组件，其为系统显示，收集并发现物理显示设备提供了适配器的作用。
目前有以下两种方式的适配器供使用
一、为本地显示设备提供适配器。
二、为开发者提供的模拟显示适配器。

**DisplayAdapter** 与 **DisplayManagerService** 是弱耦合关系。`DisplayAdapter`通过注册在 `DisplayManagerService`类中的 **DisplayAdapter.Listener** 实现异步通信。

这样做有两个原因 
一、巧妙地封装了这两个类的职责，
**DisplayAdapter** ：处理各个显示设备
**DisplayManagerService**：处理全局显示状态。 
二、消除异步显示设备发现导致死锁的可能性

**Synchronization（同步锁）**

因为显示管理器可能被多个线程访问，所以同步锁就会变得有点复杂。 特别是当窗口管理器（`window manager`）在保持绘制事务的同时调用显示管理器（`display manager`），窗口管理器期望它可以立即应用并更改。 但不幸的是，显示管理器（`display manager`）不能异步地做所有事情。
为了解决这个问题，显示管理器的所有对象必须持有相同的锁。 我们将此锁称为同步锁，它具有唯一性。

# 3.DisplayManagerService 继承关系

`DisplayManagerService`继承 `SystemService`, 由 `SystemServer` 启动。 
```
public final class DisplayManagerService extends SystemService {
   ... 
}
```
**SystemService** 是系统Service的基类，相关类使用要重写它的以下生命周期方法（构造方法、onStart() 、onBootPhase(int)），并且所有生命周期内地方法都可以被 `system server`主线程循环调用。
**构造方法**
在系统 初始化`SystemService`的时候被调用。
**onStart()方法**
`Services` 运行时候被调用，并且此时需要对外公布`Binder`接口
**publishBinderService(String, IBinder)方法**
有时候会同时对外公布本地服务`publishLocalService`共系统进程调用。
**onBootPhase(int)方法**
在启动阶段会被调用多次，一直到`PHASE_BOOT_COMPLETED`
下面`DisplayManagerService`的使用方法也是按照这个流程来的。

#4.DisplayManagerService 的构造方法

`DisplayManagerService ` 构造方法代码如下：

```
    public DisplayManagerService(Context context) {
        this(context, new Injector());
    }

    @VisibleForTesting
    DisplayManagerService(Context context, Injector injector) {
        super(context);
        mInjector = injector;
        mContext = context;
        // mHandler 用来发送 display 消息
        mHandler = new DisplayManagerHandler(DisplayThread.get().getLooper());
        mUiHandler = UiThread.getHandler();
        mDisplayAdapterListener = new DisplayAdapterListener();
        mSingleDisplayDemoMode = SystemProperties.getBoolean("persist.demo.singledisplay", false);
        mDefaultDisplayDefaultColorMode = mContext.getResources().getInteger(
            com.android.internal.R.integer.config_defaultDisplayDefaultColorMode);

        PowerManager pm = (PowerManager) mContext.getSystemService(Context.POWER_SERVICE);
        mGlobalDisplayBrightness = pm.getDefaultScreenBrightnessSetting();
    }
```
#5.DisplayManagerService 的onStart 方法

onStart 主要加载持久化数据（主要是显示设备的宽高等），发送 `MSG_REGISTER_DEFAULT_DISPLAY_ADAPTERS`消息，对外公布`Binder、Local Service`等。`onStart()` 方法如下：

```
   @Override
    public void onStart() {
        // We need to pre-load the persistent data store so it's ready before the default display
        // adapter is up so that we have it's configuration. We could load it lazily, but since
        // we're going to have to read it in eventually we may as well do it here rather than after
        // we've waited for the display to register itself with us.
		synchronized(mSyncRoot) {
                        //1. 加载本地持久化数据
			mPersistentDataStore.loadIfNeeded();
			loadStableDisplayValuesLocked();
        }
 // 2. 发送MSG_REGISTER_DEFAULT_DISPLAY_ADAPTERS 消息       mHandler.sendEmptyMessage(MSG_REGISTER_DEFAULT_DISPLAY_ADAPTERS);
        //3.对外公布Binder、Local 服务
        publishBinderService(Context.DISPLAY_SERVICE, new BinderService(),
                true /*allowIsolated*/);
        publishLocalService(DisplayManagerInternal.class, new LocalService());
        publishLocalService(DisplayTransformManager.class, new DisplayTransformManager());
    }
```

1. 加载本地持久化数据
```
 private void loadIfNeeded() {
        if (!mLoaded) {
            load();
            mLoaded = true;
        }
    }
    private void load() {
        clearState();

        final InputStream is;
        try {
            is = mAtomicFile.openRead();
        } catch (FileNotFoundException ex) {
            return;
        }

        XmlPullParser parser;
        try {
            parser = Xml.newPullParser();
            parser.setInput(new BufferedInputStream(is), StandardCharsets.UTF_8.name());
            loadFromXml(parser);
        } catch (IOException | XmlPullParserException ex) {
            Slog.w(TAG, "Failed to load tv input manager persistent store data.", ex);
            clearState();
        } finally {
            IoUtils.closeQuietly(is);
        }
    }
```

2.`MSG_REGISTER_DEFAULT_DISPLAY_ADAPTERS`消息 处理方法如下：
```
  private final class DisplayManagerHandler extends Handler {
        public DisplayManagerHandler(Looper looper) {
            super(looper, null, true /*async*/);
        }

        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_REGISTER_DEFAULT_DISPLAY_ADAPTERS:
                    // a.注册默认的显示适配器
                    registerDefaultDisplayAdapters();
                    break;

                case MSG_REGISTER_ADDITIONAL_DISPLAY_ADAPTERS:
                    registerAdditionalDisplayAdapters();
                    break;

         ...
    }
```
a.`registerDefaultDisplayAdapters` 实现方法如下：
`registerDefaultDisplayAdapters` 最主要功能就是将显示设备添加注册到`mDisplayAdapters`适配器中。
```
 private void registerDefaultDisplayAdapters() {
        // Register default display adapters.
        synchronized (mSyncRoot) {
            // b. 主要的显示适配器，注册本地适配器lock
            registerDisplayAdapterLocked(new LocalDisplayAdapter(
                    mSyncRoot, mContext, mHandler, mDisplayAdapterListener));

            ...
        }
```

b. 主要的显示适配器，注册本地适配器lock `registerDisplayAdapterLocked`

```
    private void registerDisplayAdapterLocked(DisplayAdapter adapter) {
        mDisplayAdapters.add(adapter);
        adapter.registerLocked();
    }

```

**LocalDisplayAdapter** 继承 **DisplayAdapter** ，主要为本地显示设备提供的适配器。
```
final class LocalDisplayAdapter extends DisplayAdapter {

   ...
    // Called with SyncRoot lock held.
    public LocalDisplayAdapter(DisplayManagerService.SyncRoot syncRoot,
            Context context, Handler handler, Listener listener) {
        super(syncRoot, context, handler, listener, TAG);
    }

    // registerDisplayAdapterLocked 中 调用 adapter.registerLocked();
   @Override
    public void registerLocked() {
        super.registerLocked();
       // 1.创建显示设备热插拔时间的监听器
        mHotplugReceiver = new HotplugDisplayEventReceiver(getHandler().getLooper());

        for (int builtInDisplayId : BUILT_IN_DISPLAY_IDS_TO_SCAN) {
            //2.连接显示设备
            tryConnectDisplayLocked(builtInDisplayId);
        }
    }
   
    ...
}
```
1.创建显示设备热插拔时间的监听器，部分代码如下：
```
    private final class HotplugDisplayEventReceiver extends DisplayEventReceiver {
        public HotplugDisplayEventReceiver(Looper looper) {
            super(looper, VSYNC_SOURCE_APP);
        }

        @Override
        public void onHotplug(long timestampNanos, int builtInDisplayId, boolean connected) {
            synchronized (getSyncRoot()) {
                if (connected) {
                   //连接显示设备
                    tryConnectDisplayLocked(builtInDisplayId);
                } else {
                    tryDisconnectDisplayLocked(builtInDisplayId);
                }
            }
        }
    }
```
2.连接显示设备，部分代码如下：
```
    private void tryConnectDisplayLocked(int builtInDisplayId) {
        IBinder displayToken = SurfaceControl.getBuiltInDisplay(builtInDisplayId);
        if (displayToken != null) {
            SurfaceControl.PhysicalDisplayInfo[] configs =
                    SurfaceControl.getDisplayConfigs(displayToken);
           
            int activeConfig = SurfaceControl.getActiveConfig(displayToken);
            
            int activeColorMode = SurfaceControl.getActiveColorMode(displayToken);

            int[] colorModes = SurfaceControl.getDisplayColorModes(displayToken);
            LocalDisplayDevice device = mDevices.get(builtInDisplayId);
            if (device == null) {
                // Display was added.
                device = new LocalDisplayDevice(displayToken, builtInDisplayId,
                        configs, activeConfig, colorModes, activeColorMode);
                mDevices.put(builtInDisplayId, device);
                sendDisplayDeviceEventLocked(device, DISPLAY_DEVICE_EVENT_ADDED);
            } else if (device.updatePhysicalDisplayInfoLocked(configs, activeConfig,
                        colorModes, activeColorMode)) {
                // Display properties changed.
                sendDisplayDeviceEventLocked(device, DISPLAY_DEVICE_EVENT_CHANGED);
            }
        } else {
            // The display is no longer available. Ignore the attempt to add it.
            // If it was connected but has already been disconnected, we'll get a
            // disconnect event that will remove it from mDevices.
        }
    }
```
 然后对 其他`services`以及app 公开`publishBinderService`接口
```
    /**
     * Publish the service so it is accessible to other services and apps.
     */
    protected final void publishBinderService(String name, IBinder service,
            boolean allowIsolated) {
        ServiceManager.addService(name, service, allowIsolated);
    }
```
 然后对 系统进程 公开`publishLocalService`接口

```
 /**
     * Publish the service so it is only accessible to the system process.
     */
    protected final <T> void publishLocalService(Class<T> type, T service) {
        LocalServices.addService(type, service);
    }
```

#6.DisplayManagerService 的onBootPhase(int phase) 方法
```
    @Override
    public void onBootPhase(int phase) {
        if (phase == PHASE_WAIT_FOR_DEFAULT_DISPLAY) {
            synchronized (mSyncRoot) {
                long timeout = SystemClock.uptimeMillis()
                        + mInjector.



				();
                while (mLogicalDisplays.get(Display.DEFAULT_DISPLAY) == null ||
                        mVirtualDisplayAdapter == null) {
                    long delay = timeout - SystemClock.uptimeMillis();
                    if (delay <= 0) {
                        throw new RuntimeException("Timeout waiting for default display "
                                + "to be initialized. DefaultDisplay="
                                + mLogicalDisplays.get(Display.DEFAULT_DISPLAY)
                                + ", mVirtualDisplayAdapter=" + mVirtualDisplayAdapter);
                    }
                    if (DEBUG) {
                        Slog.d(TAG, "waitForDefaultDisplay: waiting, timeout=" + delay);
                    }
                    try {
                        mSyncRoot.wait(delay);
                    } catch (InterruptedException ex) {
                    }
                }
            }
        }
    }
```

![](https://upload-images.jianshu.io/upload_images/5851256-600dca83084c7424.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
