
 

##### 和您一起终身学习，这里是程序员Android

ANR`(Application Not Responding ) `应用无响应的简称，是为了在 `APP`卡死时，用户 可以强制退出`APP`的选择，从而避免卡机无响应问题，这是`Android `系统的一种自我保护机制。

通过本篇阅读，您将学习到以下内容

>一、ANR 概述
> 二、ANR的类型
> 三、ANR 产生的原因
> 四、如何分析解决 ANR问题
> 五、ANR 问题分析解决建议
> 六、MTK 平台 ANR问题分析



# 一、 ANR 概述


在`Android `中，应用程序响应由`Activity Manager`和`Window Manager `系统服务进行监视。`ANR(Application Not Responding )`，则是`Android`的一种自我保护措施，当主线程出现卡顿时候，`Android` 系统会给用户一个弹出提示，让用户手动选择继续等待还是强制关闭此`APP`。

 当Android检测到以下情况之一时，`Android`将显示特定应用程序的`ANR`对话框，比如以下三种情况下`ANR `将经常发生：
- 1.` UI Thread  `超过 `5 s `没有响应
- 2.`Broadcast `广播超过`10 s`没响应
- 3.`Service`  服务超过 `20s` 没响应 

因此，为避免`ANR `发生，请不要在主线程中进行耗时操作，耗时操作请尽量在子线程中运行。
- 4.发生`ANR `截图 如下：


![ANR Dialog 举例 ](http://upload-images.jianshu.io/upload_images/5851256-42d61aaad1b15303.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 
# 二、 ANR的类型

`ANR`在` Android` 手机中很常见，按其相应类型可以分为以下 常见 三种类型。

 ANR类型如下：
 1. 按键响应分发超时(Key Dispatch Timeout)
默认 `5 s`，超过则会出现ANR。
 2. 广播超时(Broadcast Timeout)
默认 `10 s`，超过则会出现ANR。
 3. 服务超时(Service Timeout)
默认 `20 s`，超过则会出现ANR。


   
# 三、ANR 产生的原因

在`Android`系统中，`APP` 通常运行在一个`UI Thread `或者叫`MainThread`里。并且`Android`中只有一个`MainThread` 和`Main Message Queue`。`MainThread`主要用于`UI`的绘制、事件响应，监听与接收事件处理等功能。`Main Message Queue` 主要存放用户要处理消息的队列，主线程`MainThread`从消息队列`Main Message Queue`中取消息`Message`后，尽快分发下去，一旦某条消息分发超时，则`ANR`可能发生。

因此，当`ANR` 发生时，我们要分析`ANR `产生的原因，也就是查找消息处理不及时的原因。例如可以从以下几个疑问点进行分析：
- 1.为什么 `APP `不能获取`CPU `时间片？
- 2.`APP` 是否是等待一些没能及时处理的事件完成？
- 3.消息处理流程是不是太复杂？
 
#四、如何分析解决 ANR问题  

在分析`ANR`时有一些常见的模式可供选择：
1. `APP`正在主线程上进行缓慢的`I/O`操作。
2. `APP`正在主线程中进行很复杂的计算操作
3. 主线程正在对另一个进程执行同步`Binder`程序调用，但另一个进程需要很长时间才能返回结果。
4. 主线程在等待另一个正在长时间执行块操作的子线程时被阻塞。
5. 主线程因为另一个线程死锁，无论是`Bind`调用还是主线程调用，都不能让主线程等待很久，更不能在主线程中进行复杂的计算。

 知道产生ANR的原因，那么如何避免ANR 问题呢？

## 1.Strict mode

使用`StrictMode`可以帮助您在开发应用程序时在主线程上发现意外的`I / O`操作。 您可以在`application`或`activity`使用`StrictMode`。

## 2.关闭 ANR Dialog 提示

查看方法ANR控制的方法：
设置---- 开发者选项---`显示所有ANR `

 注意 ： 
如没有开发者选项，请进入设置---关于手机--- 多次连击 版本号 即可打开隐藏的开发者选项的item

![后台 app ANR 开关](http://upload-images.jianshu.io/upload_images/5851256-f336299d4f8e86f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3.Traceview 

Traceview获取正在运行的应用程序的跟踪信息，分析此`traces.txt`文件 可以推测出主线程在忙于某些事情。

`traces `文件通常保存在`/data/anr/traces.txt`下，你可以直接用`adb cat ` 查看，或者 `adb pull `出来都可以。
 
 
建议使用此方法
```

 adb root 
 adb remount 
 adb pull /data/anr/traces.txt  .

```
![pull traces 文件到桌面 ](http://upload-images.jianshu.io/upload_images/5851256-207d12fe65783366.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



# 五、ANR 问题分析解决建议

分析查看`ANR`原因，接着解决`ANR`问题。
## 1. 耗时操作
请放在工作现场中进行，可以使用`Handler、AsyncTask `等。

## 2. IO 操作
(比如：网络操作、存储操作等)也是引起ANR的常见因素。强烈建议在工作线程中进行。

## 3. 程序锁竞争

某些情况，`ANR`产生的原因不是直接因为在主线程中产生的。 比如： 工作线程对某个`资源`等上锁，恰好此时，主线程需要此`资源`，如等待超时，则此时ANR可能发生。

## 4. 死锁

当主线程因为请求一个其他线程正在持有的资源而进入等待状态时，`ANR`可能会发生。

## 5. 广播接收慢

应用程序可以通过广播接收器响应广播消息，例如启用或禁用飞行模式或更改连接状态。 当应用程序花费太长时间来处理广播消息时，理论上超过10s 未处理完成，`ANR`可能会发生。

## 6.广播 ANR发生在下列情况下：
1. `onReceive()` 方法长时间未执行完毕。

尽量避免在`onReceive()` 中进行耗时操作。

2. 广播接收者调用`goAsync()`方法并且未能在`PendingResult`对象上调用`finish()`。

如要处理的广播内容较多，请使用`IntentService` 进行处理。

比如下面例子：

3.不建议在onReceive 方法中进行耗时操作，超过10s 未处理，会引起ANR

![不建议在onReceive 方法中进行耗时操作，超过10s 未处理，会引起ANR ](http://upload-images.jianshu.io/upload_images/5851256-54507f33b1ce6c8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 建议使用`IntentService` ，避免ANR发生

![IntentService 避免处理广播消息过多引起ANR](http://upload-images.jianshu.io/upload_images/5851256-12df0e2754224247.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


您的广播接收机可以使用`goAsync()`来通知系统需要更多的时间来处理消息。 但是，您应该在`PendingResult`对象上调用`finish()`。 以下示例显示如何调用finish（）以让系统回收广播接收器并避免ANR：

![goAsync()---finish 获取更多广播响应时间](http://upload-images.jianshu.io/upload_images/5851256-b378bd3ccf228d60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 六、MTK 平台 ANR问题分析

前提，抓取一份`ANR`的` MTK log`。

## 1.`event_log`   

搜索关键字 `am_anr`或者`anr`,分析并查看`ANR`原因


![event_log 分析 ANR原因  ](http://upload-images.jianshu.io/upload_images/5851256-723e0593f5aba751.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2. `main_log`   

搜索关键字`Application Not Responding`或者`anr` ,分析并查看`ANR`原因。

![main_log 中分析ANR 原因 ](http://upload-images.jianshu.io/upload_images/5851256-0e9479c887439b3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3.  MTK ANR 策略建议

![MTK 官方总结图 ](http://upload-images.jianshu.io/upload_images/5851256-1ce6dc06e4a319a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![MTK ANR 分析步骤](http://upload-images.jianshu.io/upload_images/5851256-ab69ca09cf1e1538.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![MTK ANR  Debug SOP](http://upload-images.jianshu.io/upload_images/5851256-824c0ef08b7cc126.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![MTK ANR  Debug SOP](http://upload-images.jianshu.io/upload_images/5851256-36b4ddb3c0d3344b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 4.常见ANR 举例分析如下：


![Main Thread is idle ](http://upload-images.jianshu.io/upload_images/5851256-0fb16d549f762be8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Stuck in IO ](http://upload-images.jianshu.io/upload_images/5851256-bf0ccd20dca518bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Main Thread Waiting a lock ](http://upload-images.jianshu.io/upload_images/5851256-55366ece2abab72c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Wait Binder Transaction](http://upload-images.jianshu.io/upload_images/5851256-888d20309608a916.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Main Thread Query DB](http://upload-images.jianshu.io/upload_images/5851256-d3c00c4d4d08d4c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
