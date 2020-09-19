 #### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
 >一、ANR概览
>二、ANR类型
>三、ANR分析步骤
>四、其他分析思路
>五、死锁举例
>六、Binder Timeout举例
 






# 一、ANR概览
ANR的全称是Application No Responding，即应用程序无响应，具体是一些特定的Message(Key Dispatch、Broadcast、Service)在应用的UI线程（主线程）没有在规定的时间内处理完，进而触发ANR异常。

ANR由消息处理机制保证，Android在系统层实现了一套精密的机制来发现ANR，核心原理是消息调度和超时处理。ANR机制主体实现在系统层，所有与ANR相关的消息，都会经过系统进程system_server调度，具体是ActivityManagerService服务，然后派发到应用进程完成对消息的实际处理，同时，系统进程设计了不同的超时限制来跟踪消息的处理。 一旦应用程序处理消息不当，超时限制就起作用了，它收集一些系统状态，譬如CPU/IO使用情况、进程函数调用栈CallStack，（有些平台比如MTK，还会打印相应的Message出来供调试分析），最后报告用户有进程无响应了(ANR对话框)。

# 二、ANR类型
ANR一般有三种类型：


## 2.1 KeyDispatchTimeout
这个主要是按键或触摸事件在特定时间内无响应，一般Android平台默认超时时间是5s会报anr，不过有些平台会修改这个时间，比如MTK有些平台就是8s的超时时间。

这个超时时间可以在ActivityManagerService.java查看：
```
// M: ANR debug mechanism (Ori: 5*1000)
static final int KEY_DISPATCHING_TIMEOUT = 8*1000;
这类超时会有ANR提示框弹出，用户可以选择forceStop或者继续等待。
```
## 2.2 BroadcastTimeout
这个主要是BroadcastRecevier在规定时间无法处理完成。前台广播超时时间是10s,后台广播超时是60s,这类超时没有提示框弹出。

==> AMS.java
```
// How long we allow a receiver to run before giving up on it.
static final int BROADCAST_FG_TIMEOUT = 10*1000;
static final int BROADCAST_BG_TIMEOUT = 60*1000;
```

## 2.3 ServiceTimeout
Service在规定时间内无法处理完成操作，即会报出服务超时，这类ANR同样没有提示框出现。超时时间，前台Service是20s，后台Service是200s。

==>ActivityServices.java
```
// How long we wait for a service to finish executing.
static final int SERVICE_TIMEOUT = 20*1000;
// How long we wait for a service to finish executing.
static final int SERVICE_BACKGROUND_TIMEOUT = SERVICE_TIMEOUT * 10;
```
# 三、ANR分析步骤
ANR本质上是一个Performance问题，发生ANR的时候，如果问题可能出在APK自己身上，那么主要排查的方向是apk本身，分析看看是不是在UI线程做了耗时的操作？

个人认为，有一些ANR问题是很难分析的，比如app运行的时候由于当前系统底层的一些影响，导致当前message处理失败。这类问题往往没有规律，又难以重现。对于这类ANR问题，一般需要花费大量时间去了解系统的一些其他行为，而超出了ANR本身的范畴，这类问题就相当于一个警示信号，告诉你系统哪个地方出问题了。

## 3.1 ANR时间点
找到anr时间点，方便确认问题发生的时机：

event log : 搜 am_anr关键字，比如：

```
02-13 14:24:26.381   725   748 I am_anr  : [0,1542,com.ijidou.fm,13155909,Input dispatching timed out 
(Waiting to send key event because the focused window has not finished processing all of the input events that were previously delivered to it.  
Outbound queue length: 0.  Wait queue length: 1.)
```
main log : 直接搜anr或者anr in，比如：

```
02-17 16:13:33.117 E/ActivityManager(  187): ANR in com.dla.android (com.dla.android/com.amco.clarovideo.app.ui.activities.ContentActivity)
3.2 CPU Usage
```
在main log里面搜寻：CPU usage:

```
02-13 14:25:25.493   725   748 E ANRManager: ANR in com.ijidou.fm (com.ijidou.fm/.MainActivity), time=684319
02-13 14:25:25.493   725   748 E ANRManager: Reason: Input dispatching timed out (Waiting because no window has focus but there is a focused application that may eventually add a window when it finishes starting up.)
02-13 14:25:25.493   725   748 E ANRManager: Load: 10.9 / 9.65 / 5.75
02-13 14:25:25.493   725   748 E ANRManager: Android time :[2017-02-13 14:25:25.47] [694.891]
02-13 14:25:25.493   725   748 E ANRManager: CPU usage from 8237ms to 2157ms ago:
02-13 14:25:25.493   725   748 E ANRManager:   37% 252/mediaserver: 28% user + 9% kernel / faults: 186 minor
02-13 14:25:25.493   725   748 E ANRManager:   8.8% 257/mobile_log_d: 4.9% user + 3.9% kernel
02-13 14:25:25.493   725   748 E ANRManager:   5.4% 1406/com.txznet.txz: 4.1% user + 1.3% kernel / faults: 79 minor
02-13 14:25:25.493   725   748 E ANRManager:   5% 2263/com.txznet.txz:svr1: 4.2% user + 0.8% kernel / faults: 195 minor
02-13 14:25:25.493   725   748 E ANRManager:   4.1% 725/system_server: 3.4% user + 0.6% kernel / faults: 73 minor
02-13 14:25:25.493   725   748 E ANRManager:   4.1% 1496/com.ijidou.duc: 2.6% user + 1.4% kernel / faults: 181 minor
02-13 14:25:25.493   725   748 E ANRManager:   2.1% 3155/com.ijidou.carcorder: 0.6% user + 1.4% kernel
02-13 14:25:25.493   725   748 E ANRManager:   1.1% 1477/com.android.phone: 0.8% user + 0.3% kernel / faults: 185 minor
02-13 14:25:25.493   725   748 E ANRManager:   1.1% 5317/com.ijidou.fm: 0% user + 1.1% kernel
02-13 14:25:25.493   725   748 E ANRManager:   0.9% 2448/com.autonavi.amapautolite:locationservice: 0.3% user + 0.6% kernel / faults: 54 minor
02-13 14:25:25.493   725   748 E ANRManager:   0.8% 1513/com.ijidou.launcher: 0.4% user + 0.3% kernel / faults: 30 minor
02-13 14:25:25.493   725   748 E ANRManager:   0.6% 1387/com.ijidou.mirrornavi: 0.3% user + 0.3% kernel / faults: 1266 minor
02-13 14:25:25.493   725   748 E ANRManager:   0.6% 3001/com.ijidou.duc:remote: 0.6% user + 0% kernel / faults: 1 minor
...
02-13 14:25:25.493   725   748 E ANRManager: 27% TOTAL: 18% user + 8% kernel + 0.7% iowait + 0.1% softirq
```
ANRManager会打印出anr前后的cpu使用情况，这个可以反映出当时系统的Performance状态：

如果CPU使用量接近100%，说明当前设备很忙，有可能是CPU饥饿导致了ANR。
如果CPU使用量很少，说明主线程被BLOCK了
如果IOwait很高，说明ANR有可能是主线程在进行I/O操作造成的
那么这个时候，我们就要看看anr发生的时候，主线程在做什么了。

## 3.3 data/anr/traces.txt
App的进程发生ANR时，系统让活跃的Top进程都进行了一下dump，进程中的各种Thread就都dump到这个trace文件里了，所以trace文件中包含了每一条线程的运行时状态。不过有些平台也会自己专门收集这些信息，打包traces信息到DB文件中利用专门的工具去分析。这里不便多说。

分析trace信息需要了解几点：

traces中都是按照进程去进行的，会有pid提示。
当一个线程占有一个锁的时候，会打印-locked<0xxxxxxx>
当该线程正在等待别的线程释放该锁，会打印waiting to lock <0xxxxxx>
如果代码中有wait()调用的话，首先是locked，然后会打印waiting on <0xxxxxx>
# 四、其他分析思路
有时候我们log也分析了，traces也分析了，还是很难分析出ANR的原因，那么我们可能就需要尝试从其他方面分析了：

从main log里面找anr发生时间前8s，看看app的main thread在做什么操作？是否有异常的地方，一般process的pid会等于主线程id。
大范围搜寻log，看是否anr发生之前，哪里有发生crash？是否有可能引起anr。
看情况抓取平台的一些辅助信息，包括但不局限于：
```
memory info
process status
cpu info
Binder info
ftrace
```
# 五、死锁举例
我们来看一个死锁的例子：

```
 
"PowerManagerService" prio=5 tid=24 MONITOR
  | group="main" sCount=1 dsCount=0 obj=0x41dd0eb0 self=0x5241b218
  | sysTid=567 nice=0 sched=0/0 cgrp=apps handle=1380038664
  | state=S schedstat=( 6682116007 11324451214 33313 ) utm=450 stm=219 core=1
  at com.android.server.am.ActivityManagerService.broadcastIntent(ActivityManagerService.java:~13045)
  - waiting to lock <0x41a874a0> (a com.android.server.am.ActivityManagerService) held by tid=12 (android.server.ServerThread)
  at android.app.ContextImpl.sendBroadcast(ContextImpl.java:1144)
  at com.android.server.power.PowerManagerService$DisplayBlankerImpl.unblankAllDisplays(PowerManagerService.java:3442)
  at com.android.server.power.DisplayPowerState$PhotonicModulator$1.run(DisplayPowerState.java:456)
  at android.os.Handler.handleCallback(Handler.java:800)
  at android.os.Handler.dispatchMessage(Handler.java:100)
  at android.os.Looper.loop(Looper.java:194)
  at android.os.HandlerThread.run(HandlerThread.java:60)
```
tid=24线程在等待一个<0x41a874a0>的锁，而这个锁被tid=12的线程占用了，我们来看看tid=12：
 ```
"android.server.ServerThread" prio=5 tid=12 MONITOR
  | group="main" sCount=1 dsCount=0 obj=0x41a76178 self=0x507837a8
  | sysTid=545 nice=-2 sched=0/0 cgrp=apps handle=1349936616
  | state=S schedstat=( 15368096286 21707846934 69485 ) utm=1226 stm=310 core=0
  at com.android.server.power.PowerManagerService.isScreenOnInternal(PowerManagerService.java:~2529)
  - waiting to lock <0x41a7e2e8> (a java.lang.Object) held by tid=85 (Binder_B)
  at com.android.server.power.PowerManagerService.isScreenOn(PowerManagerService.java:2522)
  at com.android.server.wm.WindowManagerService.sendScreenStatusToClientsLocked(WindowManagerService.java:7749)
  at com.android.server.wm.WindowManagerService.setEventDispatching(WindowManagerService.java:7628)
  at com.android.server.am.ActivityManagerService.updateEventDispatchingLocked(ActivityManagerService.java:8083)
  at com.android.server.am.ActivityManagerService.wakingUp(ActivityManagerService.java:8077)
```
tid=12线程在等待<0x41a7e2e8>的锁，而这个锁被tid=85的线程占用了，我们来看看tid=85：
```
"Binder_B" prio=5 tid=85 MONITOR
 | group="main" sCount=1 dsCount=0 obj=0x42744770 self=0x58329e88
 | sysTid=3700 nice=-20 sched=0/0 cgrp=apps handle=1471424616
 | state=S schedstat=( 1663727513 2044643318 6806 ) utm=132 stm=34 core=1
 at com.android.server.power.PowerManagerService$DisplayBlankerImpl.toString(PowerManagerService.java:~3449)
 - waiting to lock <0x41a7e420> (a com.android.server.power.PowerManagerService$DisplayBlankerImpl) held by tid=24 (PowerManagerService)
```
tid=85线程在等待<0x41a7e420>的锁释放，而这个锁被tid=24占用了，所以就发送了死锁。那么这种情况下我们就需要找到发生死锁的source code，进行分析并修改。
值得注意的是，trace里面一般会包含时间，尽量分析跟anr时间相近的trace，避免受到其他干扰。

# 六、Binder Timeout举例
最后举一个Binder Timeout的例子：

## 6.1 分析SYS_BINDER_INFO
从log看到，PID 1650的main thread 呼叫PID 141的TID 2031 发生了Timeout。
[1650:1650]、[141:2031]的格式是[pid:tid]

```
------ BINDER TIMEOUT LOG ------
2:exec 1650:1650 to 141:2031 spends 4000 ms () dex_code 22 start_at 
606.464 android 2013-01-18 11:36:16.891
```
## 6.2 分析kernel log
PID 1650的main thread 呼叫PID 141的TID 2031执行超过4s。

```
[  527.923216] (1)[50:binder_watchdog]binder: 205873 exec 
1650:1650 to 141:2031 over 4.006 sec () dex_code 22 start_at 606.464 android 2013-01-18 11:36:16.891
```
## 6.3 分析anr的traces
1.看看PID 1650的main thread 呼叫PID 141的TID 2031 在做什么

![image.png](https://upload-images.jianshu.io/upload_images/5851256-8de3fe476ddce0e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.check binder call transition

![](https://upload-images.jianshu.io/upload_images/5851256-915be7a8b0846d49.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结论：发现最终停留在pid=141 tid=2031的 stopOmxComponent_l 不动，那么接下来的步骤就是找到对应的source code进行分析原因。

# 七、总结
ANR的问题分析，说简单有时候也挺简单，说复杂有时候确实能让人分析的抓狂，有时候ANR问题只是平台表现出来的一种现象，可能是某种预警信号，深入分析就要工程师看对Android系统的了解程度了。
博主曾经处理过一个不规律的anr的问题，产品在烧机一段时间后，会概率性触发ANR问题，Check了很多次log发现IO比较高，但是log基本没有什么其他异常的地方，最后经过多次烧机尝试和分析，发现原因是某些平台的EMMC因为寿命的原因发生老化导致读写速率变低，从而导致IO变高，更换EMMC后问题解决。

原文作者： 风中老狼的博客 
原文链接：https://maoao530.github.io/2017/02/21/anr-analyse/



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
