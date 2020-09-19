 ##### 和你一起终身学习，这里是程序员Android

Android O 推出出了Background Execution Limits，减少后台应用内存使用及耗电，一个很明显的应用就是不准后台应用通过startService启动服务，这里有两个问题需要弄清楚，第一：什么状态下startService的属于后台启动service；第二：如果想要在后台startService，如何兼容，因此分如下几个问题分析下

*   后台startService的场景
*   后台startService的Crash原理分析
*   如何修改达到兼容

对于普通APP而言，我们不考虑系统的各种白名单，一般后台startService服务分下面两种：

*   通过其他应用startService
*   通过自己应用startService

而每种又可以分不同的小场景，通过其他应用startService已经不被推荐，所以先看看自己应用startService。

> **本文基于Android P源码**

# 通过自己应用在后台startService限制

可以通过一个简单的实验观察什么情况属于后台startService，**注意：如果是自己APP启动Service，那么自身应用必定已经起来了**。通过延迟执行就复现该场景。比如：通过click事件，延迟执行一个startService操作，延迟时间是65s（**要超过一分钟，后面会看到这是个阈值**），然后点击Home键，回到桌面，之后等待一分钟就可复现Crash：

```
@OnClick(R.id.first)
void first() {
    new Handler().postDelayed(new Runnable() {
        @Override
        public void run() {
            Intent intent = new Intent(LabApplication.getContext(), BackGroundService.class);
            startService(intent);
            LogUtils.v("延迟执行");
        }
    },1000*65);

}

```

大概一分多钟之后，延迟消息被执行，然后会有如下Crash日志被打印：

```
        --------- beginning of crash
2019-06-17 19:47:43.148 25916-25916/com.snail.labaffinity E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.snail.labaffinity, PID: 25916
    java.lang.IllegalStateException: Not allowed to start service Intent { cmp=com.snail.labaffinity/.service.BackGroundService }: app is in background uid UidRecord{9048c2c u0a73 LAST bg:+1m4s376ms idle change:idle procs:1 seq(0,0,0)}
        at android.app.ContextImpl.startServiceCommon(ContextImpl.java:1577)
        at android.app.ContextImpl.startService(ContextImpl.java:1532)
        at android.content.ContextWrapper.startService(ContextWrapper.java:664)
        at com.snail.labaffinity.activity.MainActivity$2.run(MainActivity.java:41)
        at android.os.Handler.handleCallback(Handler.java:873)
        at android.os.Handler.dispatchMessage(Handler.java:99)
        at android.os.Looper.loop(Looper.java:193)
        at android.app.ActivityThread.main(ActivityThread.java:6669)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:858)

```

就会看到很经典的startService限制信息：

```
 Not allowed to start service Intent XXX : app is in background uid UidRecord  

```

也就是说，当前退到后台的APP已经属于后台应用，不能通过startService启动服务。Why？跟踪源码看下 startService会调用ContextImpl 的startServiceCommon，进而通过Binder调用AMS启动Service，根据返回值选择性抛出IllegalStateException异常：

> ContextImpl.java

```
private ComponentName startServiceCommon(Intent service, boolean requireForeground,
        UserHandle user) {
    try {
        validateServiceIntent(service);
        service.prepareToLeaveProcess(this);
        ComponentName cn = ActivityManager.getService().startService(
            mMainThread.getApplicationThread(), service, service.resolveTypeIfNeeded(
                        getContentResolver()), requireForeground,
                        getOpPackageName(), user.getIdentifier());
        if (cn != null) {
        <!--返回值是？的情况下就是后台启动service的异常-->
             if (cn.getPackageName().equals("?")) {
                throw new IllegalStateException(
                        "Not allowed to start service " + service + ": " + cn.getClassName());
            }
}

```

什么时候ActivityManager.getService().startService的返回值包名 **？**，核心代码在AMS端，AMS进一步调用ActiveServices.java的startServiceLocked：

> ActiveServices.java

```
ComponentName startServiceLocked(IApplicationThread caller, Intent service, String resolvedType,
        int callingPid, int callingUid, boolean fgRequired, String callingPackage, final int userId)
        throws TransactionTooLargeException {
         final boolean callerFg;

    if (caller != null) {
        final ProcessRecord callerApp = mAm.getRecordForAppLocked(caller);
          ...
        callerFg = callerApp.setSchedGroup != ProcessList.SCHED_GROUP_BACKGROUND;
    } else {
        callerFg = true;
    }

    ServiceLookupResult res =
        retrieveServiceLocked(service, resolvedType, callingPackage,
                callingPid, callingUid, userId, true, callerFg, false, false);
    ...
    ServiceRecord r = res.record;

    // If we're starting indirectly (e.g. from PendingIntent), figure out whether
    // we're launching into an app in a background state.  This keys off of the same
    // idleness state tracking as e.g. O+ background service start policy.

    <!--通过PendingIntent启动的也要检查-->
    // 是否当前Uid Active 不过不是activity就是后台启动
    final boolean bgLaunch = !mAm.isUidActiveLocked(r.appInfo.uid);
   // If the app has strict background restrictions, we treat any bg service
    // start analogously to the legacy-app forced-restrictions case, regardless
    // of its target SDK version.

    boolean forcedStandby = false;
    <!--appRestrictedAnyInBackground 一般人不会主动设置，所以这个经常是返回false-->
    if (bgLaunch && appRestrictedAnyInBackground(r.appInfo.uid, r.packageName)) {
        ...
       forcedStandby = true;
    }
    <!--forcedStandby可以先无视 这里注意两点，第一点 ：r.startRequested标志是通过startService调用启动过，第一次进来的时候是false，第二：对于普通是starServicefgRequired是false-->  

    if (forcedStandby || (!r.startRequested && !fgRequired)) {

        <!--检测当前app是否允许后台启动-->
        final int allowed = mAm.getAppStartModeLocked(r.appInfo.uid, r.packageName,
                r.appInfo.targetSdkVersion, callingPid, false, false, forcedStandby);
                <!--如果不允许  Background start not allowed-->
        if (allowed != ActivityManager.APP_START_MODE_NORMAL) {
            ...
            <!--返回 ? 告诉客户端现在处于后台启动状态，禁止你-->
            return new ComponentName("?", "app is in background uid " + uidRec);
        }
    }

```

假设我们是第一次startService，那么(!r.startRequested && !fgRequired)就等于true，进而走进mAm.getAppStartModeLocked，看看当前进程是否处于后台非激活状态，如果是的话 ，就不会允许startService：

> ActivityManagerService.java

```
  int getAppStartModeLocked(int uid, String packageName, int packageTargetSdk,
            int callingPid, boolean alwaysRestrict, boolean disabledOnly, boolean forcedStandby) {
        UidRecord uidRec = mActiveUids.get(uid);

         <!--UidRecord是关键  alwaysRestrict || forcedStandby 传入的都是false，忽略  -->

        if (uidRec == null || alwaysRestrict || forcedStandby || uidRec.idle) {
            boolean ephemeral;
            ...

                final int startMode = (alwaysRestrict)
                        ? appRestrictedInBackgroundLocked(uid, packageName, packageTargetSdk)
                        : appServicesRestrictedInBackgroundLocked(uid, packageName,
                                packageTargetSdk);
               ...
                return startMode;

        }
        return ActivityManager.APP_START_MODE_NORMAL;
    }

```

这里UidRecord是关键，UidRecord为null，则说明整个APP没有被启动，那么就一定属于后台启动Service，如果UidRecord非null，则要判断应用是否属于后台应用，而这个关键就是uidRec.idle，如果idle是true，就说明应用处于后台状态，继续调用 appServicesRestrictedInBackgroundLocked看看是否是O以后的，走Crash逻辑：

```
    int appServicesRestrictedInBackgroundLocked(int uid, String packageName, int packageTargetSdk) {
    <!--永久进程 -->
    // Persistent app?
    if (mPackageManagerInt.isPackagePersistent(packageName)) {
        return ActivityManager.APP_START_MODE_NORMAL;
    }

    <!--白名单-->
    // Non-persistent but background whitelisted?
    if (uidOnBackgroundWhitelist(uid)) {
        return ActivityManager.APP_START_MODE_NORMAL;
    }
    <!--白名单-->
    // Is this app on the battery whitelist?
    if (isOnDeviceIdleWhitelistLocked(uid, /*allowExceptIdleToo=*/ false)) {
        return ActivityManager.APP_START_MODE_NORMAL;
    }

    // 普通进程
    return appRestrictedInBackgroundLocked(uid, packageName, packageTargetSdk);
}

```

对于普通进程看看O限制

```
    int appRestrictedInBackgroundLocked(int uid, String packageName, int packageTargetSdk) {
        <!--对于targetSDKVersion>O 的直接 返回ActivityManager.APP_START_MODE_DELAYED_RIGID-->
        if (packageTargetSdk >= Build.VERSION_CODES.O) {
            return ActivityManager.APP_START_MODE_DELAYED_RIGID;
        }
        // 否则仅仅对老版本做兼容性限制
        int appop = mAppOpsService.noteOperation(AppOpsManager.OP_RUN_IN_BACKGROUND,
                uid, packageName);
        if (DEBUG_BACKGROUND_CHECK) {
            Slog.i(TAG, "Legacy app " + uid + "/" + packageName + " bg appop " + appop);
        }
        switch (appop) {
            case AppOpsManager.MODE_ALLOWED:
                // If force-background-check is enabled, restrict all apps that aren't whitelisted.
                if (mForceBackgroundCheck &&
                        !UserHandle.isCore(uid) &&
                        !isOnDeviceIdleWhitelistLocked(uid, /*allowExceptIdleToo=*/ true)) {
                    return ActivityManager.APP_START_MODE_DELAYED;
                }
           ...
    }

```

appServicesRestrictedInBackgroundLocked仅仅是根据是否是O以后，返回ActivityManager.APP_START_MODE_DELAYED_RIGID，只是兼容，核心还是UidRecord的idle，下面就重点看看UidRecord跟其idle的值，这个值是应用是否位于后台的核心指标，应用未启动的不考虑，未启动肯定也属于”后台“的一种极端。

不是特别老的Android版本都不允许没有LAUNCHER Activity的应用，不然压根没法编译运行，也就说普通场景通过桌面启动应用的时候，都是通过startActivity直接启动APP的，在启动App的时候，UidRecord会被新建（AMS端），UidRecord构造函数中默认 idle = true。

```
public UidRecord(int _uid) {
    uid = _uid;
    idle = true;
    reset();
}

```

其启动流程调用堆栈如下：

![image](//upload-images.jianshu.io/upload_images/1460468-aac4771c031e350f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

也就是启动APP时候，刚好一开UidRecord中idle的值是true，被看做后台应用，那么一定有某个地方设置为false，设置为前台应用。

## 前后台应用切换时机与原理

一个应用可以有一个或者多个进程，当任何一个进程变为被转换成前台可见进程的时候，APP都会被认作前台应用（对于startService应用而言），resumetopActivity是一个非常明确的切换时机，

会调用

```
final void scheduleIdleLocked() {
    mHandler.sendEmptyMessage(IDLE_NOW_MSG);
}

```

会通过updateOomAdjLocked修改当前即将可见Activity应用的idle状态，updateOomAdjLocked在期间可能会被调用多次，

```
 @GuardedBy("this")
 final void updateOomAdjLocked() {
      ...   
     for (int i=mActiveUids.size()-1; i>=0; i--) {
                final UidRecord uidRec = mActiveUids.valueAt(i);
                int uidChange = UidRecord.CHANGE_PROCSTATE;
                if (uidRec.curProcState != ActivityManager.PROCESS_STATE_NONEXISTENT
                        && (uidRec.setProcState != uidRec.curProcState
                               || uidRec.setWhitelist != uidRec.curWhitelist)) {
                    ...
                    if (ActivityManager.isProcStateBackground(uidRec.curProcState)
                            && !uidRec.curWhitelist) {
                        ...
                    } else {
                    <!--设置为false 标记为前台进程-->
                        if (uidRec.idle) {
                            uidChange = UidRecord.CHANGE_ACTIVE;
                            EventLogTags.writeAmUidActive(uidRec.uid);
                            uidRec.idle = false;
                        }
                        <!--清零后台进程锚点时间-->
                        uidRec.lastBackgroundTime = 0;
                    }

```

对于即将可见的APP而言 ActivityManager.isProcStateBackground为false，所以走else逻辑设置uidRec.idle = false,uidChange = UidRecord.CHANGE_ACTIVE，之后通过enqueueUidChangeLocked最后设置相对应的idle = false。相对应，上一个被切换走的应用可能会触发设置idle = true的操作，不过设置为true的操作不是即可执行的，而是延迟执行的，延迟时间60s：

```
 final void updateOomAdjLocked() {
  ...
         for (int i=mActiveUids.size()-1; i>=0; i--) {
            final UidRecord uidRec = mActiveUids.valueAt(i);
            int uidChange = UidRecord.CHANGE_PROCSTATE;
            if (uidRec.curProcState != ActivityManager.PROCESS_STATE_NONEXISTENT
                    && (uidRec.setProcState != uidRec.curProcState
                           || uidRec.setWhitelist != uidRec.curWhitelist)) {

                if (ActivityManager.isProcStateBackground(uidRec.curProcState)
                        && !uidRec.curWhitelist) {
                    // UID is now in the background (and not on the temp whitelist).  Was it
                    // previously in the foreground (or on the temp whitelist)?
                    if (!ActivityManager.isProcStateBackground(uidRec.setProcState)
                            || uidRec.setWhitelist) {
                            <!--切换后台时候更新lastBackgroundTime-->
                        uidRec.lastBackgroundTime = nowElapsed;
                        if (!mHandler.hasMessages(IDLE_UIDS_MSG)) {
                            <!--60s后更新-->
                            mHandler.sendEmptyMessageDelayed(IDLE_UIDS_MSG,
                                    mConstants.BACKGROUND_SETTLE_TIME);
                        }
                    }

```

延迟60s是为了防止60s之内多次切换APP导致的重复更新，系统只要保证60s内有一次就可以了。

```
 private static final long DEFAULT_BACKGROUND_SETTLE_TIME = 60*1000;

```

60s之后调用idleUids更新idle字段

```
 final void idleUids() {
    synchronized (this) {
        final int N = mActiveUids.size();
        if (N <= 0) {
            return;
        }
        final long nowElapsed = SystemClock.elapsedRealtime();
        final long maxBgTime = nowElapsed - mConstants.BACKGROUND_SETTLE_TIME;
        long nextTime = 0;

        for (int i=N-1; i>=0; i--) {
            final UidRecord uidRec = mActiveUids.valueAt(i);
            <!--刚才切后台的时候已经更新过uidRec.lastBackgroundTime-->
            final long bgTime = uidRec.lastBackgroundTime;
            if (bgTime > 0 && !uidRec.idle) {
            <!--标准：后台存在时间超过mConstants.BACKGROUND_SETTLE_TIME-->
                if (bgTime <= maxBgTime) {
                    uidRec.idle = true;
                    uidRec.setIdle = true;
                    doStopUidLocked(uidRec.uid, uidRec);
                } else {
                如果被提前执行了，则在下一个60s到达的时候执行
                    if (nextTime == 0 || nextTime > bgTime) {
                        nextTime = bgTime;
                    }
                }
            }
        }

        if (nextTime > 0) {
            mHandler.removeMessages(IDLE_UIDS_MSG);
            mHandler.sendEmptyMessageDelayed(IDLE_UIDS_MSG,
                    nextTime + mConstants.BACKGROUND_SETTLE_TIME - nowElapsed);
        }
    }
}   

```

之前是前台，现在变后台，那么uidRec.lastBackgroundTime = nowElapsed赋值，再次切前台，uidRec.lastBackgroundTime清零，简而言之， 应用变为前台，UID状态马上变更为active状态，应用变为后台，即procState大于等于PROCESS_STATE_TRANSIENT_BACKGROUND时，如果持续在后台60s后，UID状态会变更为idle=true状态,不能startService；

# 通过其他应用startService的情况

跨应用startService已经不被推荐了，不过也容易模拟，在A应用中通过setAction+setPackage就可以startService：

```
        var intent = Intent();
        intent.setAction("com.snail.BackGroundService");
        intent.setPackage("com.snail.labaffinity");
        startService(intent)

```

当然在B应用中AndroidManifest要暴露出来：

```
    <service
        android:name=".service.BackGroundService"
        <!--是否独立进程，无关紧要-->
        android:process=":service"
        android:exported="true">
        <intent-filter>
            <action android:name="com.snail.BackGroundService" />
        </intent-filter>
    </service>

```

这样A中startService同样要遵守不准后台启动的条件。比如如果B没启动过，直接在A中startService，则会Crash，如果B启动了，还没变成后台应用（退到后台没超过60S），则不会Crash。个人觉得通过adb命令startService也属于这种范畴，通过如下命令可以达到相同的效果。

```
am startservice -n com.snail.labaffinity/com.snail.labaffinity.service.BackGroundService 

```

如果APP没有启动就会看到如下日志：

```
app is in background uid null

```

如果启动了，但是属于后台应用，就会看到如下日志，跟自己APP后台启动Service类似：

```
  Not allowed to start service Intent { cmp=com.snail.labaffinity/.service.BackGroundService }: app is in background uid UidRecord{72bb30d u0a238 SVC  idle change:idle|uncached procs:1 seq(0,0,0)}

```

其实，**startService不是看调用的APP处于何种状态，而是看Servic所在APP处于何种状态，因为看的是Servic所处的UidRecord的状态，UidRecord仅仅跟APP安装有关系，跟进程pid没关系。**

# 特殊场景：进程通过Service恢复的场景

先看下如下代码，APP在启动的时候，在Application的onCreate中通过startService启动了一个服务，并且没有stop，这种场景下第一次通过Launcher冷启动没问题，如果我们在后台杀死APP，由于存在一个未stop的服务，系统会重新拉起该服务，也就是会重启一个进程，然后启动服务。

```
public class LabApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
          Intent intent = new Intent( this, BackGroundService.class);
        startService(intent);
    }
 }

  public class BackGroundService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        LogUtils.v("onStartCommand");
    }
}

```

在这个过程中，应用重启会复现如下Crash（禁止后台启动Service的Crash Log）：

```
java.lang.RuntimeException: Unable to create application com.snail.labaffinity.app.LabApplication: java.lang.IllegalStateException: Not allowed to start service Intent { cmp=com.snail.labaffinity/.service.BackGroundService }: app is in background uid UidRecord{72bb30d u0a238 SVC  idle change:idle|uncached procs:1 seq(0,0,0)}
    at android.app.ActivityThread.handleBindApplication(ActivityThread.java:5925)
    at android.app.ActivityThread.access$1100(ActivityThread.java:200)
    at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1656)
    at android.os.Handler.dispatchMessage(Handler.java:106)
    at android.os.Looper.loop(Looper.java:193)
    at android.app.ActivityThread.main(ActivityThread.java:6718)
    at java.lang.reflect.Method.invoke(Native Method)
    at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
    at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:858)

```

Why？为什么冷启动没问题，后台杀死自启动恢复就有问题，看日志是因为当app is in background，Not allowed to start service，也就是后台进程不能通过startService启动服务，在LabApplication的onCreate中我们确实主动startService(intent)，这个就是crash的原因，那为什么第一次没问题？在前文我们知道，通过Laucher启动应用是通过startActivity启动的，也就是存在一个resumeTopActivity的时机，在这个时机，APP的idle会被设置为false，也就是非后台应用，但是对于后台杀死又恢复的场景，他不是通过startActivity启动的，所以APP就算重启了，APP的idle还是true，是非激活的状态，也就是属于后台应用，不准通过startService启动服务（假设单进程）。

因为第一次冷启动时候，走正常启动Activity流程，新建进程，然后去AMS attachApplication，

```
@GuardedBy("this")
private final boolean attachApplicationLocked(IApplicationThread thread,
        int pid, int callingUid, long startSeq) {

      ...
      <!--   通知APP端创建Application-->
            thread.bindApplication(processName, appInfo, providers,
                    app.instr.mClass,
                    profilerInfo, app.instr.mArguments,...);
        ...
    boolean badApp = false;
    boolean didSomething = false;

    // See if the top visible activity is waiting to run in this process...
    if (normalMode) {
        try {
         <!-- 需要启动的Activity 关键点 -->
            if (mStackSupervisor.attachApplicationLocked(app)) {
                didSomething = true;
            }

    if (!badApp) {
        try {
    <!--  需要恢复的Service-->
            didSomething |= mServices.attachApplicationLocked(app, processName);
            checkTime(startTime, "attachApplicationLocked: after mServices.attachApplicationLocked");
        }  

```

第一次启动APP的时候，thread.bindApplication首先通知APP端启动Application，并执行onCreate，不过onCreate中的startService要等待AMS端上一个消息执行完毕（Handler保证），这个过程中mStackSupervisor.attachApplicationLocked(app)中会调用realStartActivityLocked启动Activity，先将UidRecord的idle给更新为false，attachApplicationLocked执行之后，才有可能轮到下一个消息startService执行，这个时候APP已经不是后台应用了，所以不会Crash。

```
 boolean attachApplicationLocked(ProcessRecord app) throws RemoteException {
        final String processName = app.processName;
        boolean didSomething = false;
        ...
                final int size = mTmpActivityList.size();
                <!--存在要启动的Activity-->
                for (int i = 0; i < size; i++) {
                    final ActivityRecord activity = mTmpActivityList.get(i);
                    if (activity.app == null && app.uid == activity.info.applicationInfo.uid
                            && processName.equals(activity.processName)) {
                        try {
                        <!--走realStartActivityLocked-->
                            if (realStartActivityLocked(activity, app,
                                    top == activity /* andResume */, true /* checkConfig */))  

```

realStartActivityLocked会更新oom，并设置idle为false，因为有Activity要启动，就不在是后台进程，调用流程如下：

![image](//upload-images.jianshu.io/upload_images/1460468-88d3dbe1a8fd4c94.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

但是对于而对于杀死并通过Service恢复的进程，没有明确的startActivity，所以size = mTmpActivityList.size()这里size是0，不会走realStartActivityLocked，也就在进程恢复阶段，不会将APP归为前台应用，这个时候再AMS执行下一个消息启动Service的时候，就会告诉APP端，不能在后台启动应用。

# 如何解决这个问题

既然不能再后台偷偷启动，那只能显示启动，Google提供的方案是：startForegroundService()。并且在系统创建Service后，需要在一定时间内调用startForeground()让Service为用户可见通知，否则则系统将停止此Service，抛出ANR，如果不像让用户可见可以参考JobScheduler。不过本篇只看startForegroundService：

```
@Override
public ComponentName startService(Intent service) {
    warnIfCallingFromSystemProcess();
    return startServiceCommon(service, false, mUser);
}

@Override
public ComponentName startForegroundService(Intent service) {
    warnIfCallingFromSystemProcess();
    return startServiceCommon(service, true, mUser);
}

```

同普通startService的区别那就是startServiceCommon的第二参数boolean requireForeground 是true：

```
ComponentName startServiceLocked(IApplicationThread caller, Intent service ...}

   <!--fgRequired为true，不会检测启动后台限制-->
    if (forcedStandby || (!r.startRequested && !fgRequired)) {

        final int allowed = mAm.getAppStartModeLocked(r.appInfo.uid, r.packageName,
                r.appInfo.targetSdkVersion, callingPid, false, false, forcedStandby);
        if (allowed != ActivityManager.APP_START_MODE_NORMAL) {

            return new ComponentName("?", "app is in background uid " + uidRec);
        }
    }
    ...   
    <!--ServiceRecord赋值r.fgRequired 后面会用到-->
    r.fgRequired = fgRequired;
    <!--添加后面回调StartItem-->
    r.pendingStarts.add(new ServiceRecord.StartItem(r, false, r.makeNextStartId(),
            service, neededGrants, callingUid));

```

在AMS端startForegroundService跟普通startService区别， ServiceRecord的fgRequired被设置为true，然后走后续流程bringUpServiceLocked->realStartServiceLocked-> sendServiceArgsLocked，在sendServiceArgsLocked的时候，Service其实已经创建并启动（可以看Service启动流程），

```
private final void sendServiceArgsLocked(ServiceRecord r, boolean execInFg,
        boolean oomAdjusted) throws TransactionTooLargeException {
    ...
    ArrayList<ServiceStartArgs> args = new ArrayList<>();
    while (r.pendingStarts.size() > 0) {
        ServiceRecord.StartItem si = r.pendingStarts.remove(0);
        ...
        if (r.fgRequired && !r.fgWaiting) {
            if (!r.isForeground) {
            <!--监听是否5S内startForeground-->
                scheduleServiceForegroundTransitionTimeoutLocked(r);
            } ...
       try {
        r.app.thread.scheduleServiceArgs(r, slice);
    }

```

可以看到对于要求前台启动的Service fgRequired = true，并且第一次r.fgWaiting=false，所以会走scheduleServiceForegroundTransitionTimeoutLocked，

```
void scheduleServiceForegroundTransitionTimeoutLocked(ServiceRecord r) {
    if (r.app.executingServices.size() == 0 || r.app.thread == null) {
        return;
    }
    Message msg = mAm.mHandler.obtainMessage(
            ActivityManagerService.SERVICE_FOREGROUND_TIMEOUT_MSG);
    msg.obj = r;
    r.fgWaiting = true;
    mAm.mHandler.sendMessageDelayed(msg, SERVICE_START_FOREGROUND_TIMEOUT);
}

```

r.fgWaiting会被设置为true，scheduleServiceForegroundTransitionTimeoutLocked过一次后，就不会再次走。

```
static final int SERVICE_START_FOREGROUND_TIMEOUT = 10*1000;

```

看9.0代码，是10s完成调用startForeground，否则在10s后Handler处理这一消息的时候，会停止该服务，并抛出Service的ANR异常。

```
  void serviceForegroundTimeout(ServiceRecord r) {
        ProcessRecord app;
        synchronized (mAm) {
            if (!r.fgRequired || r.destroying) {
                return;
            }
            app = r.app;
            r.fgWaiting = false;
            stopServiceLocked(r);
        }

        if (app != null) {
            mAm.mAppErrors.appNotResponding(app, null, null, false,
                    "Context.startForegroundService() did not then call Service.startForeground(): "
                        + r);
        }
    }

```

抛出异常栈如下

```
--------- beginning of crash
E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.snail.labaffinity, PID: 21513
    android.app.RemoteServiceException: Context.startForegroundService() did not then call Service.startForeground()
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1768)
        at android.os.Handler.dispatchMessage(Handler.java:106)
        at android.os.Looper.loop(Looper.java:164)
        at android.app.ActivityThread.main(ActivityThread.java:6494)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:438)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:807)

```

解决方案就是及时调用startForeground，对于O以后的还要注意Notification需要一个ChannelID

```
 public class BackGroundService extends Service {

    @Override
    public void onCreate() {
        super.onCreate();
        startForeground();
    }

    private void startForeground() {
        String CHANNEL_ONE_ID = "com.snail.labaffinity";
        String CHANNEL_ONE_NAME = "Channel One";
        NotificationChannel notificationChannel = null;
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
            notificationChannel = new NotificationChannel(CHANNEL_ONE_ID,
                    CHANNEL_ONE_NAME, NotificationManager.IMPORTANCE_DEFAULT);
            NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
            assert manager != null;
            manager.createNotificationChannel(notificationChannel);
            startForeground(1, new NotificationCompat.Builder(this, CHANNEL_ONE_ID).build());
        }
    }

}

```

startForeground主要就是讲Service至于前台可见，同时取消掉刚才的那个延时Message，这样就不会检测并抛出异常了。

```
 private void setServiceForegroundInnerLocked(final ServiceRecord r, int id,
            Notification notification, int flags) {

            <!--id不能为0-->
        if (id != 0) {
           ...
            if (r.fgRequired) {
            <!--设置fgRequired = false-->
                r.fgRequired = false;
                <!--设置 fgWaiting = false-->
                r.fgWaiting = false;
                alreadyStartedOp = true;
                <!--移除ActivityManagerService.SERVICE_FOREGROUND_TIMEOUT_MSG消息-->
                mAm.mHandler.removeMessages(
                        ActivityManagerService.SERVICE_FOREGROUND_TIMEOUT_MSG, r);
            }

```

不过不过这样的话，状态栏会有一个xxx正在运行的通知，体验不太好，如果是要完成某项任务完成后，最好主动stop掉。还有一个要注意的问题：在调用startForGround前不准调stop，否则也会抛出异常：

```
private final void bringDownServiceLocked(ServiceRecord r) {
        ...
        if (r.fgRequired) {
        r.fgRequired = false;
        r.fgWaiting = false;
        mAm.mAppOpsService.finishOperation(AppOpsManager.getToken(mAm.mAppOpsService),
                AppOpsManager.OP_START_FOREGROUND, r.appInfo.uid, r.packageName);
        mAm.mHandler.removeMessages(
                ActivityManagerService.SERVICE_FOREGROUND_TIMEOUT_MSG, r);
        if (r.app != null) {
            Message msg = mAm.mHandler.obtainMessage(
                    ActivityManagerService.SERVICE_FOREGROUND_CRASH_MSG);
            msg.obj = r.app;
            msg.getData().putCharSequence(
                ActivityManagerService.SERVICE_RECORD_KEY, r.toString());
            mAm.mHandler.sendMessage(msg);
        }
    }

```

如果调用了startForegroundService，但是没有调用startForGround，此时调用stopService时，r.fgRequired = true，那么bringDownServiceLocked就会直接移除ActivityManagerService.SERVICE_FOREGROUND_TIMEOUT_MSG消息，并抛出ActivityManagerService.SERVICE_FOREGROUND_CRASH_MSG异常，其实只要在onCreate中startForeground就行了。

# 总结

*   **startService抛异常不是看调用的APP处于何种状态，而是看Servic所在APP处于何种状态，因为看的是UID的状态，所以这里重要的是APP而不仅仅是进程状态**
*   不要通过Handler延迟太久再startService，否则可能会有问题
*   应用进入后台，60s之后就会变成idle状态，无法start其中的Service，但是可以通过startForegroundService来启动
*   Application里面不要startService，否则恢复的时候可能有问题
*   startForGround 要及时配合startForegroundService，否则会有各种异常
 
>原文链接：https://www.jianshu.com/p/f2db0f58d47f
 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
