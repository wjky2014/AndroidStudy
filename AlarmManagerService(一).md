 

##### 和您一起终身学习，这里是程序员Android 

本篇文章主要介绍 `Android` 开发中的**AlarmManagerService**部分知识点，通过阅读本篇文章，您将收获以下内容:
>1.AlarmManager的使用
>2.AlarmManagerService初始化


```
本文转自网络地址如下：
http://www.robinheztto.com/2017/03/10/android-alarm-1/
```
Android系统通过AlarmManager向应用提供定时/闹钟服务，以使应用在其生命周期之外可执行基于特定时间的操作，本篇将具体分析AlarmManager的使用及AlarmManagerService服务的初始化。
  
相关源码位于以下文件中:
```
frameworks/base/core/java/android/app/AlarmManager.java
frameworks/base/services/core/java/com/android/server/AlarmManagerService.java
frameworks/base/services/core/jni/com_android_server_AlarmManagerService.cpp
```
### AlarmManager的使用 
1.获取AlarmManager:
Alarm相关的服务接口定义在AlarmManager中，与其他系统服务一样，通过Context获取AlarmManager。
```
Context.getSystemService(Context.ALARM_SERVICE);
```
2. Alarm的类型:
- AlarmManager.RTC_WAKEUP
使用系统绝对时间(当前系统时间，System.currentTimeMillis())，系统休眠状态也将唤醒系统。

- AlarmManager.RTC
使用系统绝对时间(当前系统时间，System.currentTimeMillis())，系统休眠状态下不可用。

- AlarmManager.ELAPSED_REALTIME_WAKEUP
使用系统相对时间(相对系统启动时间，SystemClock.elapsedRealtime())，系统休眠状态也将唤醒系统。

- AlarmManager.ELAPSED_REALTIME
使用系统相对时间(相对系统启动时间，SystemClock.elapsedRealtime())，系统休眠状态下不可用
RTC/RTC_WAKEUP和ELAPSED_REALTIME/ELAPSED_REALTIME_WAKEUP最大的差别就是RTC受time zone/locale的影响，可以通过修改手机时间触发闹钟事件，ELAPSED_REALTIME/ELAPSED_REALTIME_WAKEUP要通过真实时间的流逝，即使在休眠状态时间也会被计算。

**WAKEUP**类型的Alarm会唤醒系统，休眠状态下会增加系统的功耗，所以在使用中应尽量避免使用该种类型的Alarm。

3. Alarm的Flag:

- FLAG_STANDALONE
指定stand-alone精准alarm，该alarm不会被batch，设置WINDOW_EXACT的alarm会指定此flag。
- FLAG_WAKE_FROM_IDLE
指定alarm即使在idle模式也将唤醒系统，如alarm clock。
- FLAG_ALLOW_WHILE_IDLE
针对Doze模式，alarm即使在系统idle状态下也会执行，但是不会使系统退出idle mode，只有特殊alarm才需要标记该Flag。
- FLAG_ALLOW_WHILE_IDLE_UNRESTRICTED
针对Doze模式，alarm即使在系统idle状态下也会执行而且没有时间限制，但是不会使系统退出idle mode，只有特殊alarm才需要标记该Flag。
- FLAG_IDLE_UNTIL
只有调用AlarmManager.setIdleUntil()接口才可能设置该flag，用来使系统进入idle mode直到marker alarm被执行，执行marker alarm时系统会退出idle mode(设置后进入DozeIdle状态让Alarm系统挂起，直到这个Alarm到期)。

4.Alarm的set:

非精准Alarm，其window被指定为WINDOW_HEURISTIC：

```

public void set(int type, long triggerAtMillis, PendingIntent operation) {}

public void set(int type, long triggerAtMillis, String tag, OnAlarmListener listener,Handler targetHandler) {}

public void setRepeating(int type, long triggerAtMillis,long intervalMillis, PendingIntent operation) {}

public void setInexactRepeating(int type, long triggerAtMillis,long intervalMillis, PendingIntent operation) {}// Doze模式下

public void setAndAllowWhileIdle(int type, long triggerAtMillis, PendingIntent operation) {} 

```
精准Alarm，其window被标记为WINDOW_EXACT
```
public void setWindow(int type, long windowStartMillis, long windowLengthMillis,PendingIntent operation) {}

public void setWindow(int type, long windowStartMillis, long windowLengthMillis, String tag, OnAlarmListener listener, Handler targetHandler) {}

public void setExact(int type, long triggerAtMillis, PendingIntent operation) {}  
 
public void setExact(int type, long triggerAtMillis, String tag, OnAlarmListener listener, Handler targetHandler) {}  

public void setAlarmClock(AlarmClockInfo info, PendingIntent operation) {}// Doze模式下

public void setIdleUntil(int type, long triggerAtMillis, String tag, OnAlarmListener listener,Handler targetHandler) {}

public void setExactAndAllowWhileIdle(int type, long triggerAtMillis, PendingIntent operation) {}

```

AlarmManager中的set方法最终都是调用setImpl，下面是setImpl的具体实现。
`frameworks/base/core/java/android/app/AlarmManager.java`

```
 private void setImpl(@AlarmType int type, long triggerAtMillis, long windowMillis,
            long intervalMillis, int flags, PendingIntent operation, final OnAlarmListener listener,
            String listenerTag, Handler targetHandler, WorkSource workSource,
            AlarmClockInfo alarmClock) {
        if (triggerAtMillis < 0) {
            /* NOTYET
            if (mAlwaysExact) {
                // Fatal error for KLP+ apps to use negative trigger times
                throw new IllegalArgumentException("Invalid alarm trigger time "
                        + triggerAtMillis);
            }
            */
            triggerAtMillis = 0;
        }
// OnAlarmListener封装到ListenerWrapper，并添加到sWrappers管理
        ListenerWrapper recipientWrapper = null;
        if (listener != null) {
            synchronized (AlarmManager.class) {
                if (sWrappers == null) {
                    sWrappers = new ArrayMap<OnAlarmListener, ListenerWrapper>();
                }

                recipientWrapper = sWrappers.get(listener);
                // no existing wrapper => build a new one
                if (recipientWrapper == null) {
                    recipientWrapper = new ListenerWrapper(listener);
                    sWrappers.put(listener, recipientWrapper);
                }
            }

            final Handler handler = (targetHandler != null) ? targetHandler : mMainThreadHandler;
            recipientWrapper.setHandler(handler);
        }
 // 调用AlarmManagerService
        try {
            mService.set(mPackageName, type, triggerAtMillis, windowMillis, intervalMillis, flags,
                    operation, recipientWrapper, listenerTag, workSource, alarmClock);
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
    }
```
**AlarmManagerService初始化**

```
    public AlarmManagerService(Context context) {
        super(context);
        mConstants = new Constants(mHandler);
    }
```
下面先看Constants类的具体实现，主要负责Alarm相关的常量的读取及更新。


```
 private final class Constants extends ContentObserver {
        // Key names stored in the settings value.
        private static final String KEY_MIN_FUTURITY = "min_futurity";
        private static final String KEY_MIN_INTERVAL = "min_interval";
        private static final String KEY_ALLOW_WHILE_IDLE_SHORT_TIME = "allow_while_idle_short_time";
        private static final String KEY_ALLOW_WHILE_IDLE_LONG_TIME = "allow_while_idle_long_time";
        private static final String KEY_ALLOW_WHILE_IDLE_WHITELIST_DURATION
                = "allow_while_idle_whitelist_duration";
        private static final String KEY_LISTENER_TIMEOUT = "listener_timeout";

        private static final long DEFAULT_MIN_FUTURITY = 5 * 1000;
        private static final long DEFAULT_MIN_INTERVAL = 60 * 1000;
        private static final long DEFAULT_ALLOW_WHILE_IDLE_SHORT_TIME = DEFAULT_MIN_FUTURITY;
        private static final long DEFAULT_ALLOW_WHILE_IDLE_LONG_TIME = 9*60*1000;
        private static final long DEFAULT_ALLOW_WHILE_IDLE_WHITELIST_DURATION = 10*1000;

        private static final long DEFAULT_LISTENER_TIMEOUT = 5 * 1000;

        // Minimum futurity of a new alarm
        public long MIN_FUTURITY = DEFAULT_MIN_FUTURITY;

        // Minimum alarm recurrence interval
        public long MIN_INTERVAL = DEFAULT_MIN_INTERVAL;

        // Minimum time between ALLOW_WHILE_IDLE alarms when system is not idle.
        public long ALLOW_WHILE_IDLE_SHORT_TIME = DEFAULT_ALLOW_WHILE_IDLE_SHORT_TIME;

        // Minimum time between ALLOW_WHILE_IDLE alarms when system is idling.
        public long ALLOW_WHILE_IDLE_LONG_TIME = DEFAULT_ALLOW_WHILE_IDLE_LONG_TIME;

        // BroadcastOptions.setTemporaryAppWhitelistDuration() to use for FLAG_ALLOW_WHILE_IDLE.
        public long ALLOW_WHILE_IDLE_WHITELIST_DURATION
                = DEFAULT_ALLOW_WHILE_IDLE_WHITELIST_DURATION;

        // Direct alarm listener callback timeout
        public long LISTENER_TIMEOUT = DEFAULT_LISTENER_TIMEOUT;

        private ContentResolver mResolver;
        private final KeyValueListParser mParser = new KeyValueListParser(',');
        private long mLastAllowWhileIdleWhitelistDuration = -1;

        public Constants(Handler handler) {
            super(handler);
            updateAllowWhileIdleMinTimeLocked();
            updateAllowWhileIdleWhitelistDurationLocked();
        }

        public void start(ContentResolver resolver) {
            mResolver = resolver;
            mResolver.registerContentObserver(Settings.Global.getUriFor(
                    Settings.Global.ALARM_MANAGER_CONSTANTS), false, this);
            updateConstants();
        }

        public void updateAllowWhileIdleMinTimeLocked() {
            mAllowWhileIdleMinTime = mPendingIdleUntil != null
                    ? ALLOW_WHILE_IDLE_LONG_TIME : ALLOW_WHILE_IDLE_SHORT_TIME;
        }

        public void updateAllowWhileIdleWhitelistDurationLocked() {
            if (mLastAllowWhileIdleWhitelistDuration != ALLOW_WHILE_IDLE_WHITELIST_DURATION) {
                mLastAllowWhileIdleWhitelistDuration = ALLOW_WHILE_IDLE_WHITELIST_DURATION;
                BroadcastOptions opts = BroadcastOptions.makeBasic();
                opts.setTemporaryAppWhitelistDuration(ALLOW_WHILE_IDLE_WHITELIST_DURATION);
                mIdleOptions = opts.toBundle();
            }
        }

        @Override
        public void onChange(boolean selfChange, Uri uri) {
            updateConstants();
        }

        private void updateConstants() {
            synchronized (mLock) {
                try {
                    mParser.setString(Settings.Global.getString(mResolver,
                            Settings.Global.ALARM_MANAGER_CONSTANTS));
                } catch (IllegalArgumentException e) {
                    // Failed to parse the settings string, log this and move on
                    // with defaults.
                    Slog.e(TAG, "Bad alarm manager settings", e);
                }

                MIN_FUTURITY = mParser.getLong(KEY_MIN_FUTURITY, DEFAULT_MIN_FUTURITY);
                MIN_INTERVAL = mParser.getLong(KEY_MIN_INTERVAL, DEFAULT_MIN_INTERVAL);
                ALLOW_WHILE_IDLE_SHORT_TIME = mParser.getLong(KEY_ALLOW_WHILE_IDLE_SHORT_TIME,
                        DEFAULT_ALLOW_WHILE_IDLE_SHORT_TIME);
                ALLOW_WHILE_IDLE_LONG_TIME = mParser.getLong(KEY_ALLOW_WHILE_IDLE_LONG_TIME,
                        DEFAULT_ALLOW_WHILE_IDLE_LONG_TIME);
                ALLOW_WHILE_IDLE_WHITELIST_DURATION = mParser.getLong(
                        KEY_ALLOW_WHILE_IDLE_WHITELIST_DURATION,
                        DEFAULT_ALLOW_WHILE_IDLE_WHITELIST_DURATION);
                LISTENER_TIMEOUT = mParser.getLong(KEY_LISTENER_TIMEOUT,
                        DEFAULT_LISTENER_TIMEOUT);

                updateAllowWhileIdleMinTimeLocked();
                updateAllowWhileIdleWhitelistDurationLocked();
            }
        }

        void dump(PrintWriter pw) {
            pw.println("  Settings:");

            pw.print("    "); pw.print(KEY_MIN_FUTURITY); pw.print("=");
            TimeUtils.formatDuration(MIN_FUTURITY, pw);
            pw.println();

            pw.print("    "); pw.print(KEY_MIN_INTERVAL); pw.print("=");
            TimeUtils.formatDuration(MIN_INTERVAL, pw);
            pw.println();

            pw.print("    "); pw.print(KEY_LISTENER_TIMEOUT); pw.print("=");
            TimeUtils.formatDuration(LISTENER_TIMEOUT, pw);
            pw.println();

            pw.print("    "); pw.print(KEY_ALLOW_WHILE_IDLE_SHORT_TIME); pw.print("=");
            TimeUtils.formatDuration(ALLOW_WHILE_IDLE_SHORT_TIME, pw);
            pw.println();

            pw.print("    "); pw.print(KEY_ALLOW_WHILE_IDLE_LONG_TIME); pw.print("=");
            TimeUtils.formatDuration(ALLOW_WHILE_IDLE_LONG_TIME, pw);
            pw.println();

            pw.print("    "); pw.print(KEY_ALLOW_WHILE_IDLE_WHITELIST_DURATION); pw.print("=");
            TimeUtils.formatDuration(ALLOW_WHILE_IDLE_WHITELIST_DURATION, pw);
            pw.println();
        }
    }
```
AlarmManagerService实例化后即调用onStart()方法。
```
  @Override
    public void onStart() {
       // native层初始化
        mNativeData = init();
        mNextWakeup = mNextNonWakeup = 0;

        // We have to set current TimeZone info to kernel
        // because kernel doesn't keep this after reboot
        setTimeZoneImpl(SystemProperties.get(TIMEZONE_PROPERTY));
        /// M:add for PPL feature ,@{
        initPpl();
        ///@}
        /// M: For handling non-wakeup alarms while WFD is connected
        registerWFDStatusChangeReciever();
        ///@}
        /// M: added for BG powerSaving feature @{
        initAlarmGrouping();
        ///@}
        // Also sure that we're booting with a halfway sensible current time
        if (mNativeData != 0) {
            final long systemBuildTime = Environment.getRootDirectory().lastModified();
            if (System.currentTimeMillis() < systemBuildTime) {
                Slog.i(TAG, "Current time only " + System.currentTimeMillis()
                        + ", advancing to build time " + systemBuildTime);
                setKernelTime(mNativeData, systemBuildTime);
            }
        }

        // Determine SysUI's uid
        final PackageManager packMan = getContext().getPackageManager();
        try {
            PermissionInfo sysUiPerm = packMan.getPermissionInfo(SYSTEM_UI_SELF_PERMISSION, 0);
            ApplicationInfo sysUi = packMan.getApplicationInfo(sysUiPerm.packageName, 0);
            if ((sysUi.privateFlags&ApplicationInfo.PRIVATE_FLAG_PRIVILEGED) != 0) {
                mSystemUiUid = sysUi.uid;
            } else {
                Slog.e(TAG, "SysUI permission " + SYSTEM_UI_SELF_PERMISSION
                        + " defined by non-privileged app " + sysUi.packageName
                        + " - ignoring");
            }
        } catch (NameNotFoundException e) {
        }

        if (mSystemUiUid <= 0) {
            Slog.wtf(TAG, "SysUI package not found!");
        }

        PowerManager pm = (PowerManager) getContext().getSystemService(Context.POWER_SERVICE);
        mWakeLock = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK, "*alarm*");

        mTimeTickSender = PendingIntent.getBroadcastAsUser(getContext(), 0,
                new Intent(Intent.ACTION_TIME_TICK).addFlags(
                        Intent.FLAG_RECEIVER_REGISTERED_ONLY
                        | Intent.FLAG_RECEIVER_FOREGROUND
                        | Intent.FLAG_RECEIVER_VISIBLE_TO_INSTANT_APPS), 0,
                        UserHandle.ALL);
        Intent intent = new Intent(Intent.ACTION_DATE_CHANGED);
        intent.addFlags(Intent.FLAG_RECEIVER_REPLACE_PENDING
                | Intent.FLAG_RECEIVER_VISIBLE_TO_INSTANT_APPS);
        mDateChangeSender = PendingIntent.getBroadcastAsUser(getContext(), 0, intent,
                Intent.FLAG_RECEIVER_REGISTERED_ONLY_BEFORE_BOOT, UserHandle.ALL);
        
        // now that we have initied the driver schedule the alarm
        mClockReceiver = new ClockReceiver();
        mClockReceiver.scheduleTimeTickEvent();
        mClockReceiver.scheduleDateChangedEvent();
        mInteractiveStateReceiver = new InteractiveStateReceiver();
        mUninstallReceiver = new UninstallReceiver();
        
        if (mNativeData != 0) {
            AlarmThread waitThread = new AlarmThread();
            waitThread.start();
        } else {
            Slog.w(TAG, "Failed to open alarm driver. Falling back to a handler.");
        }

        try {
            ActivityManager.getService().registerUidObserver(new UidObserver(),
                    ActivityManager.UID_OBSERVER_IDLE, ActivityManager.PROCESS_STATE_UNKNOWN, null);
        } catch (RemoteException e) {
            // ignored; both services live in system_server
        }

        publishBinderService(Context.ALARM_SERVICE, mService);
        publishLocalService(LocalService.class, new LocalService());
    }
```
在onStart()后，SYSTEM_SERVICES_READY时onBootPhase()将被回调。
```
    @Override
    public void onBootPhase(int phase) {
        if (phase == PHASE_SYSTEM_SERVICES_READY) {
            mConstants.start(getContext().getContentResolver());
            mAppOps = (AppOpsManager) getContext().getSystemService(Context.APP_OPS_SERVICE);
            mLocalDeviceIdleController
                    = LocalServices.getService(DeviceIdleController.LocalService.class);
        }
    }

```
回到onStart()，分析native层init的调用，下面看native init()的实现。
```
frameworks/base/services/core/java/com/android/server/AlarmManagerService.java

private native long init();
```
`frameworks/base/services/core/jni/com_android_server_AlarmManagerService.cpp`

```
static const JNINativeMethod sMethods[] = {
     /* name, signature, funcPtr */
    {"init", "()J", (void*)android_server_AlarmManagerService_init},
    {"close", "(J)V", (void*)android_server_AlarmManagerService_close},
    {"set", "(JIJJ)V", (void*)android_server_AlarmManagerService_set},
    {"clear", "(JIJJ)V", (void*)android_server_AlarmManagerService_clear},
    {"waitForAlarm", "(J)I", (void*)android_server_AlarmManagerService_waitForAlarm},
    {"setKernelTime", "(JJ)I", (void*)android_server_AlarmManagerService_setKernelTime},
    {"setKernelTimezone", "(JI)I", (void*)android_server_AlarmManagerService_setKernelTimezone},
};

int register_android_server_AlarmManagerService(JNIEnv* env)
{
    return jniRegisterNativeMethods(env, "com/android/server/AlarmManagerService",
                                    sMethods, NELEM(sMethods));
}

```
register_android_server_AlarmManagerService中注册了native方法，init()即调用android_server_AlarmManagerService_init。

`frameworks/base/services/core/jni/com_android_server_AlarmManagerService.cpp`

```
static jlong android_server_AlarmManagerService_init(JNIEnv*, jobject)
{
    // 初始化/dev/alarm
    jlong ret = init_alarm_driver();
    if (ret) {
        return ret;
    }

    // 如果初始化/dev/alarm不成功，则进入timerfd初始化，现一般采用timerfd方式采用
    return init_timerfd();
}
```
Native Alarm初始化采用了二种方案，AlarmDriver与timerfd，当alarm_driver失败时则使用timerfd，现在基本使用的是timerfd。如下，AlarmImpl是Native层Alarm操作的统一接口，AlarmImplAlarmDriver与AlarmImplTimerFd是AlarmDriver与timerfd二种不同方式的具体实现。
```
class AlarmImpl
{
public:
    AlarmImpl(int *fds, size_t n_fds);
    virtual ~AlarmImpl();

    virtual int set(int type, struct timespec *ts) = 0;
    virtual int clear(int type, struct timespec *ts) = 0;
    virtual int setTime(struct timeval *tv) = 0;
    virtual int waitForAlarm() = 0;

protected:
    int *fds;
    size_t n_fds;
};

class AlarmImplAlarmDriver : public AlarmImpl
{
public:
    AlarmImplAlarmDriver(int fd) : AlarmImpl(&fd, 1) { }

    int set(int type, struct timespec *ts);
    int clear(int type, struct timespec *ts);
    int setTime(struct timeval *tv);
    int waitForAlarm();
};

class AlarmImplTimerFd : public AlarmImpl
{
public:
    AlarmImplTimerFd(int fds[N_ANDROID_TIMERFDS], int epollfd, int rtc_id) :
        AlarmImpl(fds, N_ANDROID_TIMERFDS), epollfd(epollfd), rtc_id(rtc_id) { }
    ~AlarmImplTimerFd();

    int set(int type, struct timespec *ts);
    int clear(int type, struct timespec *ts);
    int setTime(struct timeval *tv);
    int waitForAlarm();

private:
    int epollfd;
    int rtc_id;
};
```
下面先看AlarmDriver的方式。

`frameworks/base/services/core/jni/com_android_server_AlarmManagerService.cpp`
```
static jlong init_alarm_driver()
{
    // 打开/dev/alarm,失败则退出
    int fd = open("/dev/alarm", O_RDWR);
    if (fd < 0) {
        ALOGV("opening alarm driver failed: %s", strerror(errno));
        return 0;
    }

    // 根据fd创建AlarmImplAlarmDriver对象
    AlarmImpl *ret = new AlarmImplAlarmDriver(fd);
    return reinterpret_cast<jlong>(ret);
}

int AlarmImplAlarmDriver::set(int type, struct timespec *ts)
{
    return ioctl(fds[0], ANDROID_ALARM_SET(type), ts);
}

int AlarmImplAlarmDriver::clear(int type, struct timespec *ts)
{
    return ioctl(fds[0], ANDROID_ALARM_CLEAR(type), ts);
}

int AlarmImplAlarmDriver::setTime(struct timeval *tv)
{
    struct timespec ts;
    int res;

    ts.tv_sec = tv->tv_sec;
    ts.tv_nsec = tv->tv_usec * 1000;
    res = ioctl(fds[0], ANDROID_ALARM_SET_RTC, &ts);
    if (res < 0)
        ALOGV("ANDROID_ALARM_SET_RTC ioctl failed: %s\n", strerror(errno));
    return res;
}

int AlarmImplAlarmDriver::waitForAlarm()
{
    return ioctl(fds[0], ANDROID_ALARM_WAIT);
}
```
AlarmImplAlarmDriver中主要通过ioctl来实现Alarm的操作。当init_alarm_driver打开/dev/alarm失败时，选择timerfd实现Alarm的操作。

`frameworks/base/services/core/jni/com_android_server_AlarmManagerService.cpp
`
```
static const size_t N_ANDROID_TIMERFDS = ANDROID_ALARM_TYPE_COUNT + 1;
static const clockid_t android_alarm_to_clockid[N_ANDROID_TIMERFDS] = {
    CLOCK_REALTIME_ALARM,
    CLOCK_REALTIME,
    CLOCK_BOOTTIME_ALARM,
    CLOCK_BOOTTIME,
    CLOCK_MONOTONIC,
    CLOCK_POWEROFF_ALARM,
    CLOCK_REALTIME,
};

static jlong init_timerfd()
{
    int epollfd;
    int fds[N_ANDROID_TIMERFDS];

    // 创建epoll句柄，监听N_ANDROID_TIMERFDS个文件描述符
    epollfd = epoll_create(N_ANDROID_TIMERFDS);
    if (epollfd < 0) {
        ALOGV("epoll_create(%zu) failed: %s", N_ANDROID_TIMERFDS,
                strerror(errno));
        return 0;
    }

    for (size_t i = 0; i < N_ANDROID_TIMERFDS; i++) {
        // 创建定时器文件
        fds[i] = timerfd_create(android_alarm_to_clockid[i], 0);
        if (fds[i] < 0) {
            ALOGV("timerfd_create(%u) failed: %s",  android_alarm_to_clockid[i],
                    strerror(errno));
            close(epollfd);
            for (size_t j = 0; j < i; j++) {
                close(fds[j]);
            }
            return 0;
        }
    }

    // 根据fds创建AlarmImplTimerFd对象，AlarmImplTimerFd也继承于AlarmImpl
    AlarmImpl *ret = new AlarmImplTimerFd(fds, epollfd, wall_clock_rtc());

    for (size_t i = 0; i < N_ANDROID_TIMERFDS; i++) {
        epoll_event event;
        event.events = EPOLLIN | EPOLLWAKEUP;
        event.data.u32 = i;

        // 将创建的定时器文件列表加入到epoll监听中
        int err = epoll_ctl(epollfd, EPOLL_CTL_ADD, fds[i], &event);
        if (err < 0) {
            ALOGV("epoll_ctl(EPOLL_CTL_ADD) failed: %s", strerror(errno));
            delete ret;
            return 0;
        }
    }

    struct itimerspec spec;
    memset(&spec, 0, sizeof(spec));

    int err = timerfd_settime(fds[ANDROID_ALARM_TYPE_COUNT],
            TFD_TIMER_ABSTIME | TFD_TIMER_CANCEL_ON_SET, &spec, NULL);
    if (err < 0) {
        ALOGV("timerfd_settime() failed: %s", strerror(errno));
        delete ret;
        return 0;
    }

    return reinterpret_cast<jlong>(ret);
}
```
init_timerfd()中利用epoll+timerfd的方式，创建timerfd文件并加入到epoll监听中，创建的定时器文件中，我们主要使用的是CLOCK_REALTIME_ALARM，CLOCK_BOOTTIME_ALARM，CLOCK_POWEROFF_ALARM，分别对应RTC_WAKEUP(RTC)，ELAPSED_REALTIME_WAKEUP(ELAPSED_REALTIME)，RTC_POWEROFF_WAKEUP。下面具体看一下AlarmImplTimerFd::set设置定时器的实现。
`frameworks/base/services/core/jni/com_android_server_AlarmManagerService.cpp`
```
int AlarmImplTimerFd::set(int type, struct timespec *ts)
{
    if (type > ANDROID_ALARM_TYPE_COUNT) {
        errno = EINVAL;
        return -1;
    }

    if (!ts->tv_nsec && !ts->tv_sec) {
        ts->tv_nsec = 1;
    }
    /* timerfd interprets 0 = disarm, so replace with a practically
       equivalent deadline of 1 ns */

    struct itimerspec spec;
    memset(&spec, 0, sizeof(spec));
    memcpy(&spec.it_value, ts, sizeof(spec.it_value));

    // 直接调用timerfd_settime设置Alarm定时时间
    return timerfd_settime(fds[type], TFD_TIMER_ABSTIME, &spec, NULL);
}
```
在waitForAlarm中将等待Alarm的到来，下面看AlarmImplTimerFd::waitForAlarm的实现。

`frameworks/base/services/core/jni/com_android_server_AlarmManagerService.cpp`
```
int AlarmImplTimerFd::waitForAlarm()
{
    epoll_event events[N_ANDROID_TIMERFDS];

    // 利用epolle_wait监听定时器的事件
    int nevents = epoll_wait(epollfd, events, N_ANDROID_TIMERFDS, -1);
    if (nevents < 0) {
        return nevents;
    }

    int result = 0;
    // 事件到来，循环读取定时器文件
    for (int i = 0; i < nevents; i++) {
        uint32_t alarm_idx = events[i].data.u32;
        uint64_t unused;
        ssize_t err = read(fds[alarm_idx], &unused, sizeof(unused));
        if (err < 0) {
            if (alarm_idx == ANDROID_ALARM_TYPE_COUNT && errno == ECANCELED) {
                // 时间改变
                result |= ANDROID_ALARM_TIME_CHANGE_MASK;
            } else {
                return err;
            }
        } else {
            // 设置result为触发的alarm_idx
            result |= (1 << alarm_idx);
        }
    }
    // 返回结果给AlarmManagerService
    return result;
}

```
waitForAlarm中一直在epoll_wait监听等待Alarm fd事件，当事件到来，循环读取定时器文件并向上层返回触发的Alarm index或时间改变事件。




![](https://upload-images.jianshu.io/upload_images/5851256-e3290cc1f96f982f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/5851256-9df6c3500752ee49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
