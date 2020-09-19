 

##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、ShutdownThread 概述
>二、shutdown 关机方法
>三、shutdownInner 方法实现
>四、CloseDialogReceiver 注册关机对话框广播
>五、new sConfirmDialog 显示关机对话框
>六、beginShutdownSequence开始关机流程
>七、showShutdownDialog 显示关机进度对话框
>八、判断是否显示SystemUIReboot
>九、启动 关机线程 ，执行 Run 方法
>十、shutdownRadios 关闭Radio 实现
>十一、rebootOrShutdown 初始化并决定关机还是重启
>十二、关机log 分析




##一、ShutdownThread 概述
 
##1.关机线程实现类
`Android `关机线程实现类`frameworks/base/services/core/java/com/android/server/power/ShutdownThread.java` 

##2.ShutdownThread 大致流程

![关机线程大致流程](https://upload-images.jianshu.io/upload_images/5851256-cd10d5630862588c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



##3.ShutdownThread   部分静态变量如下：

```
public final class ShutdownThread extends Thread {
    // constants
    private static final String TAG = "ShutdownThread";
    private static final int ACTION_DONE_POLL_WAIT_MS = 500;
    private static final int RADIOS_STATE_POLL_SLEEP_MS = 100;
    // maximum time we wait for the shutdown broadcast before going on.
    private static final int MAX_BROADCAST_TIME = 10*1000;
    private static final int MAX_SHUTDOWN_WAIT_TIME = 20*1000;
    private static final int MAX_RADIO_WAIT_TIME = 12*1000;
    private static final int MAX_UNCRYPT_WAIT_TIME = 15*60*1000;
    // constants for progress bar. the values are roughly estimated based on timeout.
    private static final int BROADCAST_STOP_PERCENT = 2;
    private static final int ACTIVITY_MANAGER_STOP_PERCENT = 4;
    private static final int PACKAGE_MANAGER_STOP_PERCENT = 6;
    private static final int RADIO_STOP_PERCENT = 18;
    private static final int MOUNT_SERVICE_STOP_PERCENT = 20;
    // Time we should wait for vendor subsystem shutdown
    // Sleep times(ms) between checks of the vendor subsystem state
    private static final int VENDOR_SUBSYS_STATE_CHECK_INTERVAL_MS = 100;
    // Max time we wait for vendor subsystems to shut down before resuming
    // with full system shutdown
    private static final int VENDOR_SUBSYS_MAX_WAIT_MS = 10000;

    // length of vibration before shutting down
    private static final int SHUTDOWN_VIBRATE_MS = 500;

    ... ...
}
```

##4.ShutdownThread 无参构造方法
```
  private ShutdownThread() {
    }
```
#二、 shutdown 关机实现
长按 ·Power· 键调用 `shutdown `方法，通过调用内部`shutdownInner`方法，实现 弹出 关机 、重启对话框，供用户选择关机 或者重启。

```
 /**
     * Request a clean shutdown, waiting for subsystems to clean up their
     * state etc.  Must be called from a Looper thread in which its UI
     * is shown.
     *
     * @param context Context used to display the shutdown progress dialog. This must be a context
     *                suitable for displaying UI (aka Themable).
     * @param reason code to pass to android_reboot() (e.g. "userrequested"), or null.
     * @param confirm true if user confirmation is needed before shutting down.
     */
    public static void shutdown(final Context context, String reason, boolean confirm) {
        mReboot = false;
        mRebootSafeMode = false;
        mReason = reason;
        shutdownInner(context, confirm); // 详见分析3
    }
```

#三、shutdownInner 方法实现

## 1.`shutdownInner ` 主要作用是

1.获取用户关机 `Behavior`。
2.注册关机广播，详见4.
3.创建关机`Dialog`。
4.执行一系列关机流程。
```
   private static void shutdownInner(final Context context, boolean confirm) {
        // ShutdownThread is called from many places, so best to verify here that the context passed
        // in is themed.
        context.assertRuntimeOverlayThemable();

        // ensure that only one thread is trying to power down.
        // any additional calls are just returned
        synchronized (sIsStartedGuard) {
            if (sIsStarted) {
                Log.d(TAG, "Request to shutdown already running, returning.");
                return;
            }
        }
        // 获取用户长按Power键的处理行为
        final int longPressBehavior = context.getResources().getInteger(
                        com.android.internal.R.integer.config_longPressOnPowerBehavior);
        final int resourceId = mRebootSafeMode
                ? com.android.internal.R.string.reboot_safemode_confirm
                : (longPressBehavior == 2
                        ? com.android.internal.R.string.shutdown_confirm_question
                        : com.android.internal.R.string.shutdown_confirm);

        Log.d(TAG, "Notifying thread to start shutdown longPressBehavior=" + longPressBehavior);

        if (confirm) {
           //注册关机对话框广播  详细实现见 4
            final CloseDialogReceiver closer = new CloseDialogReceiver(context);
            if (sConfirmDialog != null) {
                sConfirmDialog.dismiss();
            }
            // 创建关机Dialg 详见 实现5
            sConfirmDialog = new AlertDialog.Builder(context)
                    .setTitle(mRebootSafeMode
                            ? com.android.internal.R.string.reboot_safemode_title
                            : com.android.internal.R.string.power_off)
                    .setMessage(resourceId)
                    .setPositiveButton(com.android.internal.R.string.yes, new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int which) {
                             // 开始一系列的关机流程，详见 6
                            beginShutdownSequence(context);
                        }
                    })
                    .setNegativeButton(com.android.internal.R.string.no, null)
                    .create();
            closer.dialog = sConfirmDialog;
            sConfirmDialog.setOnDismissListener(closer);
            sConfirmDialog.getWindow().setType(WindowManager.LayoutParams.TYPE_KEYGUARD_DIALOG);
            sConfirmDialog.show();
        } else {
            beginShutdownSequence(context);
        }
    }
```

其中 `config_longPressOnPowerBehavior` 用来控制用户长按`Power`键触发的行为。
主要行为如下：
```
 <!-- Control the behavior when the user long presses the power button.
            0 - Nothing
            1 - Global actions menu
            2 - Power off (with confirmation)
            3 - Power off (without confirmation)
            4 - Go to voice assist
    -->
    <integer name="config_longPressOnPowerBehavior">1</integer>
```
此源码 可以在 `config.xml`中查看，代码目录如下：`frameworks\base\core\res\res\values\config.xml`
#四、CloseDialogReceiver 注册关机对话框广播
注册`action` 为`ACTION_CLOSE_SYSTEM_DIALOGS`广播接收器。
```
    private static class CloseDialogReceiver extends BroadcastReceiver
            implements DialogInterface.OnDismissListener {
        private Context mContext;
        public Dialog dialog;

        CloseDialogReceiver(Context context) {
            mContext = context;
            IntentFilter filter = new IntentFilter(Intent.ACTION_CLOSE_SYSTEM_DIALOGS);
            context.registerReceiver(this, filter);
        }

        @Override
        public void onReceive(Context context, Intent intent) {
            dialog.cancel();
        }

        public void onDismiss(DialogInterface unused) {
            mContext.unregisterReceiver(this);
        }
    }
```
#五、new   sConfirmDialog  显示关机对话框
`Android `原生关机对话框如下：
![关机对话框创建](https://upload-images.jianshu.io/upload_images/5851256-64649ac2742c4e51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
            // 创建关机Dialg 详见 实现5
            sConfirmDialog = new AlertDialog.Builder(context)
                    .setTitle(mRebootSafeMode
                            ? com.android.internal.R.string.reboot_safemode_title
                            : com.android.internal.R.string.power_off)
                    .setMessage(resourceId)
                    .setPositiveButton(com.android.internal.R.string.yes, new DialogInterface.OnClickListener() {
                        public void onClick(DialogInterface dialog, int which) {
                             // 开始一系列的关机流程，详见 6
                            beginShutdownSequence(context);
                        }
                    })
                    .setNegativeButton(com.android.internal.R.string.no, null)
                    .create();
            closer.dialog = sConfirmDialog;
            sConfirmDialog.setOnDismissListener(closer);
            sConfirmDialog.getWindow().setType(WindowManager.LayoutParams.TYPE_KEYGUARD_DIALOG);
            sConfirmDialog.show();
```
#六、beginShutdownSequence开始关机流程

`beginShutdownSequence `开始一系列关机流程，主要功能如下：
1.显示关机进度条对话框。
2.启动`ShutdownThead ` 线程。

```

    private static void beginShutdownSequence(Context context) {
        synchronized (sIsStartedGuard) {
            if (sIsStarted) {
                Log.d(TAG, "Shutdown sequence already running, returning.");
                return;
            }
            sIsStarted = true;
        }
        // 显示关机进度对话框 详见 7 
        sInstance.mProgressDialog = showShutdownDialog(context);
        sInstance.mContext = context;
        sInstance.mPowerManager = (PowerManager)context.getSystemService(Context.POWER_SERVICE);

        // make sure we never fall asleep again
        sInstance.mCpuWakeLock = null;
        try {
            sInstance.mCpuWakeLock = sInstance.mPowerManager.newWakeLock(
                    PowerManager.PARTIAL_WAKE_LOCK, TAG + "-cpu");
            sInstance.mCpuWakeLock.setReferenceCounted(false);
            sInstance.mCpuWakeLock.acquire();
        } catch (SecurityException e) {
            Log.w(TAG, "No permission to acquire wake lock", e);
            sInstance.mCpuWakeLock = null;
        }

        // also make sure the screen stays on for better user experience
        sInstance.mScreenWakeLock = null;
        if (sInstance.mPowerManager.isScreenOn()) {
            try {
                sInstance.mScreenWakeLock = sInstance.mPowerManager.newWakeLock(
                        PowerManager.FULL_WAKE_LOCK, TAG + "-screen");
                sInstance.mScreenWakeLock.setReferenceCounted(false);
                sInstance.mScreenWakeLock.acquire();
            } catch (SecurityException e) {
                Log.w(TAG, "No permission to acquire wake lock", e);
                sInstance.mScreenWakeLock = null;
            }
        }

        if (SecurityLog.isLoggingEnabled()) {
            SecurityLog.writeEvent(SecurityLog.TAG_OS_SHUTDOWN);
        }

        // 启动 关机线程 ，执行 Run 方法，详见 9
        sInstance.mHandler = new Handler() {
        };
        sInstance.start();
    }

```
#七、showShutdownDialog 显示关机进度对话框

`showShutdownDialog `主要用来显示关机进度对话框
```

    private static ProgressDialog showShutdownDialog(Context context) {
        // Throw up a system dialog to indicate the device is rebooting / shutting down.
        ProgressDialog pd = new ProgressDialog(context);

        // Path 1: Reboot to recovery for update
        //   Condition: mReason startswith REBOOT_RECOVERY_UPDATE
        //
        //  Path 1a: uncrypt needed
        //   Condition: if /cache/recovery/uncrypt_file exists but
        //              /cache/recovery/block.map doesn't.
        //   UI: determinate progress bar (mRebootHasProgressBar == True)
        //
        // * Path 1a is expected to be removed once the GmsCore shipped on
        //   device always calls uncrypt prior to reboot.
        //
        //  Path 1b: uncrypt already done
        //   UI: spinning circle only (no progress bar)
        //
        // Path 2: Reboot to recovery for factory reset
        //   Condition: mReason == REBOOT_RECOVERY
        //   UI: spinning circle only (no progress bar)
        //
        // Path 3: Regular reboot / shutdown
        //   Condition: Otherwise
        //   UI: spinning circle only (no progress bar)

        // mReason could be "recovery-update" or "recovery-update,quiescent".
        if (mReason != null && mReason.startsWith(PowerManager.REBOOT_RECOVERY_UPDATE)) {
            // We need the progress bar if uncrypt will be invoked during the
            // reboot, which might be time-consuming.
            mRebootHasProgressBar = RecoverySystem.UNCRYPT_PACKAGE_FILE.exists()
                    && !(RecoverySystem.BLOCK_MAP_FILE.exists());
            pd.setTitle(context.getText(com.android.internal.R.string.reboot_to_update_title));
            if (mRebootHasProgressBar) {
                pd.setMax(100);
                pd.setProgress(0);
                pd.setIndeterminate(false);
                pd.setProgressNumberFormat(null);
                pd.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
                pd.setMessage(context.getText(
                            com.android.internal.R.string.reboot_to_update_prepare));
            } else {
                if (showSysuiReboot()) {
                    return null;
                }
                pd.setIndeterminate(true);
                pd.setMessage(context.getText(
                            com.android.internal.R.string.reboot_to_update_reboot));
            }
        } else if (mReason != null && mReason.equals(PowerManager.REBOOT_RECOVERY)) {
            if (RescueParty.isAttemptingFactoryReset()) {
                // We're not actually doing a factory reset yet; we're rebooting
                // to ask the user if they'd like to reset, so give them a less
                // scary dialog message.
                pd.setTitle(context.getText(com.android.internal.R.string.power_off));
                pd.setMessage(context.getText(com.android.internal.R.string.shutdown_progress));
                pd.setIndeterminate(true);
            } else {
                // Factory reset path. Set the dialog message accordingly.
                pd.setTitle(context.getText(com.android.internal.R.string.reboot_to_reset_title));
                pd.setMessage(context.getText(
                            com.android.internal.R.string.reboot_to_reset_message));
                pd.setIndeterminate(true);
            }
        } else {
           // 判断是否显示SystemUIReboot 详见 8
            if (showSysuiReboot()) {
                return null;
            }
            // 显示关机 进度对话框 详见 8
            pd.setTitle(context.getText(com.android.internal.R.string.power_off));
            pd.setMessage(context.getText(com.android.internal.R.string.shutdown_progress));
            pd.setIndeterminate(true);
        }
        pd.setCancelable(false);
        pd.getWindow().setType(WindowManager.LayoutParams.TYPE_KEYGUARD_DIALOG);

        pd.show();
        return pd;
    }

```
##1.关机 进度对话框实现
![关机 进度对话框实现](https://upload-images.jianshu.io/upload_images/5851256-3dc6b742669e4fed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#八、判断是否显示SystemUIReboot

`showSysuiReboot` 主要用来显示是否显示`SystemUIReboot`.
```

    private static boolean showSysuiReboot() {
        Log.d(TAG, "Attempting to use SysUI shutdown UI");
        try {
            StatusBarManagerInternal service = LocalServices.getService(
                    StatusBarManagerInternal.class);
            if (service.showShutdownUi(mReboot, mReason)) {
                // Sysui will handle shutdown UI.
                Log.d(TAG, "SysUI handling shutdown UI");
                return true;
            }
        } catch (Exception e) {
            // If anything went wrong, ignore it and use fallback ui
        }
        Log.d(TAG, "SysUI is unavailable");
        return false;
    }

```

#九、启动 关机线程 ，执行 Run 方法

在`beginShutdownSequence`方法中启动 关机线程。
```
    private static void beginShutdownSequence(Context context) {
      ... ...
    // static instance of this thread
    private static final ShutdownThread sInstance = new ShutdownThread();
        // start the thread that initiates shutdown
        sInstance.mHandler = new Handler() {
        };
        sInstance.start();
        ... ...
}
```

##1.ShutdownThread Run 实现

`ShutdownThread ` 主要作用：
1.发送有序的关机广播。
2.测量关闭某些项目的时间。
3.关闭` ActivityManager`。
4.关闭`PackageManager`。
5.关闭 `Radios`。
6.完成`System Server` 的关闭。
7.初始化并判定执行关机还是重启
```

    /**
     * Makes sure we handle the shutdown gracefully.
     * Shuts off power regardless of radio state if the allotted time has passed.
     */
    public void run() {
        TimingsTraceLog shutdownTimingLog = newTimingsLog();
        shutdownTimingLog.traceBegin("SystemServerShutdown");
        metricShutdownStart();
        metricStarted(METRIC_SYSTEM_SERVER);

        BroadcastReceiver br = new BroadcastReceiver() {
            @Override public void onReceive(Context context, Intent intent) {
                // We don't allow apps to cancel this, so ignore the result.
                actionDone();
            }
        };

        /*
         * Write a system property in case the system_server reboots before we
         * get to the actual hardware restart. If that happens, we'll retry at
         * the beginning of the SystemServer startup.
         */
        {
            String reason = (mReboot ? "1" : "0") + (mReason != null ? mReason : "");
            SystemProperties.set(SHUTDOWN_ACTION_PROPERTY, reason);
        }

        /*
         * If we are rebooting into safe mode, write a system property
         * indicating so.
         */
        if (mRebootSafeMode) {
            SystemProperties.set(REBOOT_SAFEMODE_PROPERTY, "1");
        }

        metricStarted(METRIC_SEND_BROADCAST);
        shutdownTimingLog.traceBegin("SendShutdownBroadcast");
	
        Log.i(TAG, "Sending shutdown broadcast...");

        // First send the high-level shut down broadcast.
        mActionDone = false;
        Intent intent = new Intent(Intent.ACTION_SHUTDOWN);
        intent.addFlags(Intent.FLAG_RECEIVER_FOREGROUND | Intent.FLAG_RECEIVER_REGISTERED_ONLY);
		// 发送 有序的 关机广播
        mContext.sendOrderedBroadcastAsUser(intent,
                UserHandle.ALL, null, br, mHandler, 0, null, null);

        final long endTime = SystemClock.elapsedRealtime() + MAX_BROADCAST_TIME;
        synchronized (mActionDoneSync) {
            while (!mActionDone) {
                long delay = endTime - SystemClock.elapsedRealtime();
                if (delay <= 0) {
                    Log.w(TAG, "Shutdown broadcast timed out");
                    break;
                } else if (mRebootHasProgressBar) {
                    int status = (int)((MAX_BROADCAST_TIME - delay) * 1.0 *
                            BROADCAST_STOP_PERCENT / MAX_BROADCAST_TIME);
                    sInstance.setRebootProgress(status, null);
                }
                try {
                    mActionDoneSync.wait(Math.min(delay, ACTION_DONE_POLL_WAIT_MS));
                } catch (InterruptedException e) {
                }
            }
        }
        if (mRebootHasProgressBar) {
            sInstance.setRebootProgress(BROADCAST_STOP_PERCENT, null);
        }
		//  测量 发送有序 关机广播所用的时间
        shutdownTimingLog.traceEnd(); // SendShutdownBroadcast
        metricEnded(METRIC_SEND_BROADCAST);

        Log.i(TAG, "Shutting down activity manager...");
        shutdownTimingLog.traceBegin("ShutdownActivityManager");
        metricStarted(METRIC_AM);
        // 关闭 Activity Manager
        final IActivityManager am =
                IActivityManager.Stub.asInterface(ServiceManager.checkService("activity"));
        if (am != null) {
            try {
                am.shutdown(MAX_BROADCAST_TIME);
            } catch (RemoteException e) {
            }
        }
        if (mRebootHasProgressBar) {
            sInstance.setRebootProgress(ACTIVITY_MANAGER_STOP_PERCENT, null);
        }
        shutdownTimingLog.traceEnd();// ShutdownActivityManager
        metricEnded(METRIC_AM);

        Log.i(TAG, "Shutting down package manager...");
        shutdownTimingLog.traceBegin("ShutdownPackageManager");
        metricStarted(METRIC_PM);
        // 关闭 Package Manager
        final PackageManagerService pm = (PackageManagerService)
            ServiceManager.getService("package");
        if (pm != null) {
            pm.shutdown();
        }
        if (mRebootHasProgressBar) {
            sInstance.setRebootProgress(PACKAGE_MANAGER_STOP_PERCENT, null);
        }
        shutdownTimingLog.traceEnd(); // ShutdownPackageManager
        metricEnded(METRIC_PM);

        // Shutdown radios.
        shutdownTimingLog.traceBegin("ShutdownRadios");
        metricStarted(METRIC_RADIOS);
		//关闭  Radios  shutdownRadios 详细 请看 10
        shutdownRadios(MAX_RADIO_WAIT_TIME);
        if (mRebootHasProgressBar) {
            sInstance.setRebootProgress(RADIO_STOP_PERCENT, null);
        }
        shutdownTimingLog.traceEnd(); // ShutdownRadios
        metricEnded(METRIC_RADIOS);

        //  关闭 System Server 
        if (mRebootHasProgressBar) {
            sInstance.setRebootProgress(MOUNT_SERVICE_STOP_PERCENT, null);

            // If it's to reboot to install an update and uncrypt hasn't been
            // done yet, trigger it now.
            uncrypt();
        }

        shutdownTimingLog.traceEnd(); // SystemServerShutdown
        metricEnded(METRIC_SYSTEM_SERVER);
        saveMetrics(mReboot, mReason);
        //初始化 并判断是关机还是重启 详见 11
        rebootOrShutdown(mContext, mReboot, mReason);
    }

```

#十、shutdownRadios 关闭Radio 实现
`shutdownRadios` 用来关闭`phone Radio`相关内容 ，主要实现如下：

```

    private void shutdownRadios(final int timeout) {
        // If a radio is wedged, disabling it may hang so we do this work in another thread,
        // just in case.
        final long endTime = SystemClock.elapsedRealtime() + timeout;
        final boolean[] done = new boolean[1];
		//另起线程 异步关闭 Radios
        Thread t = new Thread() {
            public void run() {
                TimingsTraceLog shutdownTimingsTraceLog = newTimingsLog();
                boolean radioOff;

                final ITelephony phone =
                        ITelephony.Stub.asInterface(ServiceManager.checkService("phone"));

                try {
                    radioOff = phone == null || !phone.needMobileRadioShutdown();
                    if (!radioOff) {
                        Log.w(TAG, "Turning off cellular radios...");
                        metricStarted(METRIC_RADIO);
                        phone.shutdownMobileRadios();
                    }
                } catch (RemoteException ex) {
                    Log.e(TAG, "RemoteException during radio shutdown", ex);
                    radioOff = true;
                }
                
                Log.i(TAG, "Waiting for Radio...");

                long delay = endTime - SystemClock.elapsedRealtime();
                while (delay > 0) {
                    if (mRebootHasProgressBar) {
                        int status = (int)((timeout - delay) * 1.0 *
                                (RADIO_STOP_PERCENT - PACKAGE_MANAGER_STOP_PERCENT) / timeout);
                        status += PACKAGE_MANAGER_STOP_PERCENT;
                        sInstance.setRebootProgress(status, null);
                    }

                    if (!radioOff) {
                        try {
                            radioOff = !phone.needMobileRadioShutdown();
                        } catch (RemoteException ex) {
                            Log.e(TAG, "RemoteException during radio shutdown", ex);
                            radioOff = true;
                        }
                        if (radioOff) {
                            Log.i(TAG, "Radio turned off.");
                            metricEnded(METRIC_RADIO);
                            shutdownTimingsTraceLog
                                    .logDuration("ShutdownRadio", TRON_METRICS.get(METRIC_RADIO));
                        }
                    }

                    if (radioOff) {
                        Log.i(TAG, "Radio shutdown complete.");
                        done[0] = true;
                        break;
                    }
                    SystemClock.sleep(RADIOS_STATE_POLL_SLEEP_MS);
                    delay = endTime - SystemClock.elapsedRealtime();
                }
            }
        };
        //另起线程 异步关闭 Radios
        t.start();
        try {
            t.join(timeout);
        } catch (InterruptedException ex) {
        }
        if (!done[0]) {
            Log.w(TAG, "Timed out waiting for Radio shutdown.");
        }
    }

```
#十一、rebootOrShutdown  初始化并决定关机还是重启 

`rebootOrShutdown`主要用来 初始化并决定关机还是重启。 
```


    /**
     * Do not call this directly. Use {@link #reboot(Context, String, boolean)}
     * or {@link #shutdown(Context, String, boolean)} instead.
     *
     * @param context Context used to vibrate or null without vibration
     * @param reboot true to reboot or false to shutdown
     * @param reason reason for reboot/shutdown
     */
    public static void rebootOrShutdown(final Context context, boolean reboot, String reason) {
        String subsysProp;
        subsysProp = SystemProperties.get("vendor.peripheral.shutdown_critical_list",
                        "ERROR");
        //If we don't have the shutdown critical subsystem list we can't
        //really do anything. Proceed with full system shutdown.
        if (!subsysProp.equals("ERROR")) {
                Log.i(TAG, "Shutdown critical subsyslist is :"+subsysProp+": ");
                Log.i(TAG, "Waiting for a maximum of " +
                           (VENDOR_SUBSYS_MAX_WAIT_MS) + "ms");
                String[] subsysList = subsysProp.split(" ");
                int wait_count = 0;
                boolean okToShutdown = true;
                String subsysState;
                int subsysListLength = subsysList.length;
                do {
                        okToShutdown = true;
                        for (int i = 0; i < subsysListLength; i++) {
                                subsysState =
                                        SystemProperties.get(
                                                        "vendor.peripheral." +
                                                        subsysList[i] +
                                                        ".state",
                                                        "ERROR");
                                if(subsysState.equals("ONLINE"))  {
                                        //We only want to delay shutdown while
                                        //one of the shutdown critical
                                        //subsystems still shows as 'ONLINE'.
                                        okToShutdown = false;
                                }
                        }
                        if (okToShutdown == false) {
                                SystemClock.sleep(VENDOR_SUBSYS_STATE_CHECK_INTERVAL_MS);
                                wait_count++;
                        }
                } while (okToShutdown != true &&
                                wait_count < (VENDOR_SUBSYS_MAX_WAIT_MS/VENDOR_SUBSYS_STATE_CHECK_INTERVAL_MS));
                if (okToShutdown != true) {
                        for (int i = 0; i < subsysList.length; i++) {
                                subsysState =
                                        SystemProperties.get(
                                                        "vendor.peripheral." +
                                                        subsysList[i] +
                                                        ".state",
                                                        "ERROR");
                                if(subsysState.equals("ONLINE"))  {
                                        Log.w(TAG, "Subsystem " + subsysList[i]+
                                                   "did not shut down within timeout");
                                }
                        }
                } else {
                        Log.i(TAG, "Vendor subsystem(s) shutdown successful");
                }
        }
        if (reboot) {
            Log.i(TAG, "Rebooting, reason: " + reason);
            PowerManagerService.lowLevelReboot(reason);
            Log.e(TAG, "Reboot failed, will attempt shutdown instead");
            reason = null;
        } else if (SHUTDOWN_VIBRATE_MS > 0 && context != null) {
            // vibrate before shutting down
            Vibrator vibrator = new SystemVibrator(context);
            try {
                vibrator.vibrate(SHUTDOWN_VIBRATE_MS, VIBRATION_ATTRIBUTES);
            } catch (Exception e) {
                // Failure to vibrate shouldn't interrupt shutdown.  Just log it.
                Log.w(TAG, "Failed to vibrate during shutdown.", e);
            }

            // vibrator is asynchronous so we need to wait to avoid shutting down too soon.
            try {
                Thread.sleep(SHUTDOWN_VIBRATE_MS);
            } catch (InterruptedException unused) {
            }
        }
        // Shutdown power
        Log.i(TAG, "Performing low-level shutdown...");
        PowerManagerService.lowLevelShutdown(reason);
    }

```
#十二、关机log 分析

通过 链接`adb`,我们可以抓取关机的`Log`， 部分`Log`信息如下:
```
Line 96: 07-26 01:02:26.205  2510  3766 I AudioController: internalShutdown
	Line 540: 07-26 01:02:30.968   790   790 D ShutdownThread: Notifying thread to start shutdown longPressBehavior=1
	Line 540: 07-26 01:02:30.968   790   790 D ShutdownThread: Notifying thread to start shutdown longPressBehavior=1
	Line 541: 07-26 01:02:30.970   790   790 D ShutdownThread: Attempting to use SysUI shutdown UI
	Line 541: 07-26 01:02:30.970   790   790 D ShutdownThread: Attempting to use SysUI shutdown UI
	Line 542: 07-26 01:02:30.971   790   790 D ShutdownThread: SysUI handling shutdown UI
	Line 542: 07-26 01:02:30.971   790   790 D ShutdownThread: SysUI handling shutdown UI
	Line 555: 07-26 01:02:30.997   790  3902 I ShutdownThread: Sending shutdown broadcast...
	Line 555: 07-26 01:02:30.997   790  3902 I ShutdownThread: Sending shutdown broadcast...
	Line 561: 07-26 01:02:31.014   790   790 W SyncManager: Writing sync state before shutdown...
	Line 666: 07-26 01:02:31.436  2510  3766 I AudioController: internalShutdown
	Line 669: 07-26 01:02:31.443   790   790 I StatsCompanionService: StatsCompanionService noticed a shutdown.
	Line 708: 07-26 01:02:31.671   790  3902 D ShutdownTiming: SendShutdownBroadcast took to complete: 675ms
	Line 708: 07-26 01:02:31.671   790  3902 D ShutdownTiming: SendShutdownBroadcast took to complete: 675ms
	Line 711: 07-26 01:02:31.671   790  3902 I ShutdownThread: Shutting down activity manager...
	Line 740: 07-26 01:02:31.954   790  3902 W AppOps  : Writing app ops before shutdown...
	Line 771: 07-26 01:02:32.152   790  3902 W BatteryStats: Writing battery stats before shutdown...
	Line 806: 07-26 01:02:32.386   790  1015 W system_server: Long monitor contention with owner Thread-29 (3902) at void com.android.server.am.BatteryStatsService.shutdown()(BatteryStatsService.java:249) waiters=0 in void com.android.server.am.BatteryStatsService.noteProcessStart(java.lang.String, int) for 132ms
	Line 840: 07-26 01:02:32.507   790  3902 W ProcessStatsService: Writing process stats before shutdown...
	Line 850: 07-26 01:02:32.563   790  3902 D ShutdownTiming: ShutdownActivityManager took to complete: 891ms
	Line 850: 07-26 01:02:32.563   790  3902 D ShutdownTiming: ShutdownActivityManager took to complete: 891ms
	Line 853: 07-26 01:02:32.563   790  3902 I ShutdownThread: Shutting down package manager...
	Line 872: 07-26 01:02:32.656   790  3902 D ShutdownTiming: ShutdownPackageManager took to complete: 93ms
	Line 872: 07-26 01:02:32.656   790  3902 D ShutdownTiming: ShutdownPackageManager took to complete: 93ms
	Line 879: 07-26 01:02:32.677  1804  1976 V PhoneInterfaceManager: [PhoneIntfMgr] 1 Phones are shutdown.
	Line 882: 07-26 01:02:32.678   790  3951 I ShutdownThread: Waiting for Radio...
	Line 883: 07-26 01:02:32.679   790  3951 I ShutdownThread: Radio shutdown complete.
	Line 883: 07-26 01:02:32.679   790  3951 I ShutdownThread: Radio shutdown complete.
	Line 884: 07-26 01:02:32.681   790  3902 D ShutdownTiming: ShutdownRadios took to complete: 25ms
	Line 884: 07-26 01:02:32.681   790  3902 D ShutdownTiming: ShutdownRadios took to complete: 25ms
	Line 885: 07-26 01:02:32.681   790  3902 D ShutdownTiming: SystemServerShutdown took to complete: 1705ms
	Line 885: 07-26 01:02:32.681   790  3902 D ShutdownTiming: SystemServerShutdown took to complete: 1705ms
	Line 886: 07-26 01:02:32.684   790  3902 I ShutdownThread: Shutdown critical subsyslist is :modem : 
	Line 886: 07-26 01:02:32.684   790  3902 I ShutdownThread: Shutdown critical subsyslist is :modem : 
	Line 887: 07-26 01:02:32.684   790  3902 I ShutdownThread: Waiting for a maximum of 10000ms
	Line 888: 07-26 01:02:32.684   790  3902 I ShutdownThread: Vendor subsystem(s) shutdown successful
	Line 888: 07-26 01:02:32.684   790  3902 I ShutdownThread: Vendor subsystem(s) shutdown successful
	Line 931: 07-26 01:02:33.192   790  3902 I ShutdownThread: Performing low-level shutdown...
	Line 931: 07-26 01:02:33.192   790  3902 I ShutdownThread: Performing low-level shutdown...
	Line 1003: 07-26 01:02:33.543   565   585 E Dpps    : ProcessAlMsg():471 Shutdown event recieved quit -108
	Line 1005: 07-26 01:02:33.543   565   585 I Dpps    : ProcessNextEvent():142 Shutdown notification quit processing
	Line 1009: 07-26 01:02:33.562   565   583 E Dpps    : ProcessAbaMsg():579 Shutdown event recieved quit -108
```

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

