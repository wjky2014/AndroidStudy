
#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Broadcast概述
>二、Broadcast的注册
>三、Broadcast的注册类型
>四、静态注册开机广播的实现
>五、动态监听亮灭屏幕广播实现
>六、广播的发送方法






# 一、Broadcast概述

在了解广播之前，我们先了解`Broadcast`继承关系 ，
`Broadcast`的继承关系如下：
```
java.lang.Object
   ↳    android.content.BroadcastReceiver
```

`Broadcast `是 `Android` 四大组件之一，是一种广泛运用在应用程序之间异步传输信息的机制。`Broadcast` 本质上是一个`Intent` 对象，差别在于` Broadcast `可以被多个 `BroadcastReceiver `处理。`BroadcastReceiver` 是一个全局监听器，通过它的 `onReceive()` 可以过滤用户想要的广播，进而进行其它操作。

`BroadcastReceiver ` 默认是在主线程中执行，如果`onReceiver()  `方法处理事件超过`10s`，则应用将会发生`ANR（Application Not Responding）`，此时，如果建立工作线程并不能解决此问题，因此建议：如处理耗时操作，请用 `Service `代替。

` BroadcastReceiver `的主要声明周期方法`onReceiver()`，此方法执行完之后，`BroadcastReceiver `实例将被销毁。












# 二、Broadcast的注册
 
`Broadcast` 属于`Android`四大组件之一，必须在`Androidmainfest.xml `中注册.

`Broadcast`  注册方法如下： 
```
        <receiver
            android:name="ReceiverMethod"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="String....." />
            </intent-filter>
        </receiver>
```
**注意：**
 如不注册，将导致无法接收处理广播消息

# 三、Broadcast的注册类型

广播的注册分两种(静态注册、动态注册)，一种在`Androidmainfest.xml`中静态注册，另一种是在`Java`代码中动态注册。

## 1.静态注册

一些系统发送的广播需要在`Androidmainfest.xml `中静态注册，例如 开机广播，apk状态改变广播，电量状态改变广播等。这些静态注册的广播，通常在`Androidmainfest.xml `中拦截特定的字符串。

静态注册广播的方法如下：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.programandroid"
    android:versionCode="1"
    android:versionName="1.0" >
         ... ...
        <receiver
            android:name="com.programandroid.BroadcastReceiver.NotificationReceived"
            android:enabled="true"
            android:exported="true" >
            <intent-filter>
                <action android:name="Notification_cancel" />
                <action android:name="Notification_music_pre" />
                <action android:name="Notification_music_play" />
                <action android:name="Notification_music_next" />
            </intent-filter>
        </receiver>
        ... ...
```
 


##  2.动态注册广播

在Java中动态注册广播，通常格式如下：

```
  //动态注册广播
  registerReceiver(BroadcastReceiver, IntentFilter);
```

# 四、静态注册开机广播的实现

## 1. 静态开机广播实现

```
p ublic class BootReceiverMethod extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        //   接收开机广播处理事情，比如启动服务
        Intent mStartIntent = new Intent(context, StartServiceMethods.class);
        context.startService(mStartIntent);
    }
}
```

##  2.静态注册开机广播

```
        <!-- 必须声明接收开机广播完成的权限-->
        <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

        <receiver
            android:name=".component.BroadcastReceiver.BootReceiverMethod"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>
```

# 五、动态监听亮灭屏幕广播实现

## 1.动态注册亮灭屏实现

动态注册广播方法是`registerReceiver()`。
**注意：**
在广播中动态注册广播请注意一定要使用`context.getApplicationContext()`，防止`context` 为空 ，引起空指针异常。

动态注册亮灭屏实现实现如下：
```
public class ScreenOnOffReceiver {

    public static void ReceiverScreenOnOff(Context context) {
        IntentFilter screenOffFilter = new IntentFilter();
        screenOffFilter.addAction(Intent.ACTION_SCREEN_OFF);
        screenOffFilter.addAction(Intent.ACTION_SCREEN_ON);
        BroadcastReceiver mScreenOnOffReceiver = new BroadcastReceiver() {

            @Override
            public void onReceive(Context context, Intent intent) {
                // TODO Auto-generated method stub
                String action = intent.getAction();
                if (action.equals(Intent.ACTION_SCREEN_OFF)) {

                    Toast.makeText(context, "接收屏幕熄灭广播", Toast.LENGTH_SHORT).show();

                }
                if (action.equals(Intent.ACTION_SCREEN_ON)) {

                    Toast.makeText(context, "接收屏幕点亮广播", Toast.LENGTH_SHORT).show();
                }
            }
        };
        /**
         * context.getApplicationContext()
         * 在广播中注册广播时候需要注意，防止context 为空 ，引起空指针异常
         * **/
// 2.动态注册广播的方法
        context.registerReceiver(mScreenOnOffReceiver, screenOffFilter);
    }
}
```

# 六、广播的发送方法

广播的方法有以下三种：
1.无序sendBroadcast(intent)
2.有序sendOrderedBroadcast()
3.持续sendStickyBroadcast())

## 1.发送无序广播的方法

发送无序广播在`Android` 中很常见，是一种一对多的关系，主要通过 `sendBroadcast(intent);`发送广播。

- 1.发送自定义广播案例
广播说白了就是一个带`Action`等字符串标记的`Intent`。发送自定义广播举例如下：
```
        Intent customIntent=new Intent();
        customIntent.setAction("SendCustomBroadcast");
        sendBroadcast(customIntent);
```
- 2.接收自定义广播的方法

当用户对某些广播感兴趣的话，此时可以获取此广播，然后在`onReceive`方法中处理接收广播的一下操作。

```

public class CustomBroadcast extends BroadcastReceiver {
    public CustomBroadcast() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {

        if (intent.getAction().equals("SendCustomBroadcast")){
            Toast.makeText(context,"自定义广播接收成功：Action:SendCustomBroadcast",Toast.LENGTH_SHORT).show();
        }
    }
}
```

**注意**
自定义广播是在`Androidmanfest.xml `中静态注册的。

##  2.发送有序广播
广播在`Android`是有优先级的，优先级高的广播可以终止或修改广播内容。发送有序广播的方法如下`sendOrderedBroadcast(intent,"str_receiver_permission");`

- 1.发送自定义有序广播
```
        Intent customOrderIntent=new Intent();
        customOrderIntent.setAction("SendCustomOrderBroadcast");
        customOrderIntent.putExtra("str_order_broadcast","老板说：公司每人发 10 个 月饼");
        sendOrderedBroadcast(customOrderIntent,"android.permission.ORDERBROADCAST");
```
广播属于四大组件，一定要在`AndroidMainfest.xml `中注册。

- 2. 有序广播静态注册
接收有序广播的静态注册方法如下：

```
       <receiver
            android:name=".component.BroadcastReceiver.CustomHightBrodcast"
            android:enabled="true"
            android:exported="true"
           >
            <intent-filter android:priority="1000">
                <action android:name="SendCustomOrderBroadcast" />
            </intent-filter>
        </receiver>

        <receiver
            android:name=".component.BroadcastReceiver.CustomMiddleBroadcast"
            android:enabled="true"
            android:exported="true"
          >
            <intent-filter android:priority="500">
                <action android:name="SendCustomOrderBroadcast" />
            </intent-filter>
        </receiver>
        <receiver
            android:name=".component.BroadcastReceiver.CustomLowerBroadcast"
            android:enabled="true"
            android:exported="true"
           >
            <intent-filter android:priority="100">
                <action android:name="SendCustomOrderBroadcast" />
            </intent-filter>
        </receiver>
```

- 3. 有序广播处理，高优先级广播可以优先处理 

```
public class CustomHightBrodcast extends BroadcastReceiver {
    public CustomHightBrodcast() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {

        if (intent.getAction().equals("SendCustomOrderBroadcast")) {
            Toast.makeText(context, intent.getStringExtra("str_order_broadcast"), Toast.LENGTH_SHORT).show();
            Bundle bundle=new Bundle();
            bundle.putString("str_order_broadcast","经理说：公司每人发 5 个 月饼");
//            修改广播传输数据
            setResultExtras(bundle);
        }
    }
}
```
- 4. 中优先级的广播后序处理
```
public class CustomMiddleBroadcast extends BroadcastReceiver {
    public CustomMiddleBroadcast() {
    }
    @Override
    public void onReceive(Context context, Intent intent) {

        if (intent.getAction().equals("SendCustomOrderBroadcast")) {
            Toast.makeText(context, getResultExtras(true).getString("str_order_broadcast"), Toast.LENGTH_SHORT).show();
            Bundle bundle=new Bundle();
            bundle.putString("str_order_broadcast","主管说：公司每人发 2 个 月饼");
            setResultExtras(bundle);
        }
    }
}
```
- 5. 低优先级广播最后处理
```
public class CustomLowerBroadcast extends BroadcastReceiver {
    public CustomLowerBroadcast() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals("SendCustomOrderBroadcast")) {
            String notice= getResultExtras(true).getString("str_order_broadcast");
            Toast.makeText(context,notice, Toast.LENGTH_SHORT).show();
//          终止广播继续传播下去
            abortBroadcast();
        }
    }
}
```

**注意 ：**
 有序广播需要声明并使用权限
- 1.声明使用权限
```
 <!-- 申请使用自定义 有序广播的权限 -->
 <uses-permission >   android:name="android.permission.ORDERBROADCAST" />
``` 
- 2.声明权限
```
 <!-- 自定义 有序广播的权限 -->
 <permission>
android:name="android.permission.ORDERBROADCAST"/>
```

在有序广播中高优先级的广播接收广播，可以修改数据，然后传给低优先级的广播。

##  3.发送持续广播(已经被弃用)  
粘性广播会在`Android `系统中一直存在，不过随着 `Android`系统的不断更新，此方法逐渐被抛弃，使用方法如下：`sendStickyBroadcast(intent);`


 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 
