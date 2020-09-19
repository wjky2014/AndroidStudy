 
##### 和您一起终身学习，这里是程序员Android

`Android`是一种基于`Linux`的自由及开放源代码的操作系统，主要使用于移动设备，如智能手机和平板电脑，由`Google`公司和开放手机联盟领导及开发。这里会不断收集和更新`Android`基础相关的面试题，目前已收集`100`题。

## 1.Android系统的架构

##### 应用程序

`Android`会同一系列核心应用程序包一起发布，该应用程序包包括`Email`客户端，`SMS`短消息程序，日历，地图，浏览器，联系人管理程序等。所有的应用程序都是使用`JAVA`语言编写的。
##### 应用程序框架
 开发人员可以完全访问核心应用程序所使用的`API`框架`（android.jar）`。该应用程序的架构设计简化了组件的重用;任何一个应用程序都可以发布它的功能块并且任何其它的应用程序都可以使用其所发布的功能块。
#####系统运行库
`Android `包含一些`C/C++`库，这些库能被`Android`系统中不同的组件使用。它们通过` Android` 应用程序框架为开发者提供服务。
#####Linux 内核
`Android `的核心系统服务依赖于 `Linux ` 内核，如安全性，内存管理，进程管理， 网络协议栈和驱动模型。 `Linux` 内核也同时作为硬件和软件栈之间的抽象层。

## 2.Activity的生命周期

![Activity的生命周期
](https://upload-images.jianshu.io/upload_images/5851256-fda7c45b040a636b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/513/format/webp)

#####Activity生命周期方法

主要有`onCreate()、onStart()、onResume()、onPause()、onStop()、onDestroy()和onRestart()`等7个方法。

*   启动一个`A Activity`，
分别执行`onCreate()、onStart()、onResume()`方法。
*   从`A Activity`打开`B Activity`
分别执行`A onPause()、B onCreate()、B onStart()、B onResume()、A onStop()`方法。
*   关闭`B Activity`
分别执行`B onPause()、A onRestart()、A onStart()、A onResume()、B onStop()、B onDestroy()`方法。
*   横竖屏切换`A Activity`
清单文件中不设置`android:configChanges`属性时，先销毁`onPause()、onStop()、onDestroy()`再重新创建`onCreate()、onStart()、onResume()`方法，
设置`orientation|screenSize（一定要同时出现）`属性值时，不走生命周期方法，只会执行`onConfigurationChanged()`方法。
*  ` Activity`之间的切换
可以看出`onPause()、onStop()`这两个方法比较特殊，切换的时候`onPause()`方法不要加入太多耗时操作否则会影响体验。

## 3.Fragment的生命周期

#####Fragment的生命周期

![Fragment的生命周期](https://upload-images.jianshu.io/upload_images/5851256-ca04c0449c0eaca8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/327/format/webp)



#####Fragment与Activity生命周期对比

![](https://upload-images.jianshu.io/upload_images/5851256-410b6eaf4ba41b88.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####Fragment的生命周期方法
主要有`onAttach()、onCreate()、onCreateView()、onActivityCreated()、onstart()、onResume()、onPause()、onStop()、onDestroyView()、onDestroy()、onDetach()`等11个方法。

*   切换到该Fragment
分别执行`onAttach()、onCreate()、onCreateView()、onActivityCreated()、onstart()、onResume()`方法。
*   锁屏
分别执行`onPause()、onStop()`方法。
*   亮屏
分别执行`onstart()、onResume()`方法。
*   覆盖切换到其他Fragment
分别执行`onPause()、onStop()、onDestroyView()`方法。
*   从其他Fragment回到之前Fragment
分别执行`onCreateView()、onActivityCreated()、onstart()、onResume()`方法。

## 4.Service生命周期

在`Service`的生命周期里，常用的有：

#####4个手动调用的方法

```
startService()    启动服务
stopService()    关闭服务
bindService()    绑定服务
unbindService()    解绑服务
```

##### 5个内部自动调用的方法

```
onCreat()            创建服务
onStartCommand()    开始服务
onDestroy()            销毁服务
onBind()            绑定服务
onUnbind()            解绑服务
```

1.  手动调用`startService()`启动服务，自动调用内部方法：`onCreate()、onStartCommand()`，如果一个`Service`被`startService()`多次启动，那么`onCreate()`也只会调用一次。
2.  手动调用`stopService()`关闭服务，自动调用内部方法：`onDestory()`，如果一个`Service`被启动且被绑定，如果在没有解绑的前提下使用`stopService()`关闭服务是无法停止服务的。
3.  手动调用`bindService()`后，自动调用内部方法：`onCreate()、onBind()`。
4.  手动调用`unbindService()`后，自动调用内部方法：`onUnbind()、onDestory()`。
5. ` startService()`和`stopService()`只能开启和关闭`Service`，无法操作`Service`，调用者退出后`Service`仍然存在；`bindService()`和`unbindService()`可以操作`Service`，调用者退出后，`Service`随着调用者销毁。

## 5.Android中动画

`Android`中动画分别帧动画、补间动画和属性动画`（Android 3.0以后的）`

##### 帧动画

帧动画是最容易实现的一种动画，这种动画更多的依赖于完善的`UI`资源，他的原理就是将一张张单独的图片连贯的进行播放，从而在视觉上产生一种动画的效果；有点类似于某些软件制作`gif`动画的方式。在有些代码中，我们还会看到`android：oneshot="false" `，这个`oneshot `的含义就是动画执行一次`（true）`还是循环执行多次。

```
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:drawable="@drawable/a_0"
        android:duration="100" />
    <item
        android:drawable="@drawable/a_1"
        android:duration="100" />
    <item
        android:drawable="@drawable/a_2"
        android:duration="100" />
</animation-list>
```

##### 补间动画

补间动画又可以分为四种形式，分别是` alpha（淡入淡出）`，`translate（位移）`，`scale（缩放大小）`，`rotate（旋转）`。
补间动画的实现，一般会采用`xml `文件的形式；代码会更容易书写和阅读，同时也更容易复用。`Interpolator `主要作用是可以控制动画的变化速率 ，就是动画进行的快慢节奏。`pivot `决定了当前动画执行的参考位置

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@[package:]anim/interpolator_resource"
    android:shareInterpolator=["true" | "false"] >
    <alpha
        android:fromAlpha="float"
        android:toAlpha="float" />
    <scale
        android:fromXScale="float"
        android:toXScale="float"
        android:fromYScale="float"
        android:toYScale="float"
        android:pivotX="float"
        android:pivotY="float" />
    <translate
        android:fromXDelta="float"
        android:toXDelta="float"
        android:fromYDelta="float"
        android:toYDelta="float" />
    <rotate
        android:fromDegrees="float"
        android:toDegrees="float"
        android:pivotX="float"
        android:pivotY="float" />
    <set>
        ...
    </set>
</set>
```

##### 属性动画

属性动画，顾名思义它是对于对象属性的动画。因此，所有补间动画的内容，都可以通过属性动画实现。属性动画的运行机制是通过不断地对值进行操作来实现的，而初始值和结束值之间的动画过渡就是由`ValueAnimator`这个类来负责计算的。它的内部使用一种时间循环的机制来计算值与值之间的动画过渡，我们只需要将初始值和结束值提供给`ValueAnimator`，并且告诉它动画所需运行的时长，那么`ValueAnimator`就会自动帮我们完成从初始值平滑地过渡到结束值这样的效果。除此之外，`ValueAnimator`还负责管理动画的播放次数、播放模式、以及对动画设置监听器等。

## 6.Android中4大组件

#####1.  Activity：
`Activity`是`Android`程序与用户交互的窗口，是`Android`构造块中最基本的一种，它需要为保持各界面的状态，做很多持久化的事情，妥善管理生命周期以及一些跳转逻辑。
#####2.  BroadCast Receiver：
接受一种或者多种`Intent`作触发事件，接受相关消息，做一些简单处理，转换成一条`Notification`，统一了`Android`的事件广播模型。
#####3.  Content Provider：
是`Android`提供的第三方应用数据的访问方案，可以派生`Content Provider`类，对外提供数据，可以像数据库一样进行选择排序，屏蔽内部数据的存储细节，向外提供统一的接口模型，大大简化上层应用，对数据的整合提 供了更方便的途径。
#####4.  service：
后台服务于`Activity`，封装有一个完整的功能逻辑实现，接受上层指令，完成相关的事务，定义好需要接受的`Intent`提供同步和异步的接口。

## 7.Android中常用布局

#####常用的布局：


`FrameLayout(帧布局):`
所有东西依次都放在左上角，会重叠
`LinearLayout(线性布局):`
按照水平和垂直进行数据展示
`RelativeLayout(相对布局):`
以某一个元素为参照物，来定位的布局方式


#####不常用的布局：


`TableLayout(表格布局): `
每一个`TableLayout`里面有表格行`TableRow`，`TableRow`里面可以具体定义每一个元素`（Android TV上使用）`
`AbsoluteLayout(绝对布局):`
用`X,Y`坐标来指定元素的位置，元素多就不适用。（机顶盒上使用）
`

#####新增布局：


`  PercentRelativeLayout（百分比相对布局）  `
可以通过百分比控制控件的大小。
`  PercentFrameLayout（百分比帧布局）  `
可以通过百分比控制控件的大小。


## 8.消息推送的方式

*   方案1、使用极光和友盟推送。
*   方案2、使用`XMPP`协议`（Openfire + Spark + Smack）`

    *   简介：基于`XML`协议的通讯协议，前身是`Jabber`，目前已由`IETF`国际标准化组织完成了标准化工作。
    *   优点：协议成熟、强大、可扩展性强、目前主要应用于许多聊天系统中，且已有开源的`Java`版的开发实例`androidpn`。
        缺点：协议较复杂、冗余`（基于XML）`、费流量、费电，部署硬件成本高。
*   方案3、使用`MQTT`协议

    *   简介：轻量级的、基于代理的“发布/订阅”模式的消息传输协议。
    *   优点：协议简洁、小巧、可扩展性强、省流量、省电，目前已经应用到企业领域。
    *   缺点：不够成熟、实现较复杂、服务端组件rsmb不开源，部署硬件成本较高。
*   方案4、使用`HTTP`轮循方式
    *   简介：定时向`HTTP`服务端接口`（Web Service API）`获取最新消息。
    *   优点：实现简单、可控性强，部署硬件成本低。
    *   缺点：实时性差。

## 9.Android的数据存储

#####1.  使用`SharedPreferences`存储数据
它是`Android`提供的用来存储一些简单配置信息的一种机制，采用了`XML`格式将数据存储到设备中。只能在同一个包内使用，不能在不同的包之间使用。
#####2.  文件存储数据
文件存储方式是一种较常用的方法，在`Android`中`读取/写入文件`的方法，与`Java`中实现`I/O`的程序是完全一样的，提供了`openFileInput()`和`openFileOutput()`方法来读取设备上的文件。
#####3.  SQLite数据库存储数据
`SQLite`是`Android`所带的一个标准的数据库，它支持`SQL`语句，它是一个轻量级的嵌入式数据库。
#####4.  使用ContentProvider存储数据
主要用于应用程序之间进行数据交换，从而能够让其他的应用保存或读取此`Content Provider`的各种数据类型。
#####5.  网络存储数据
通过网络上提供给我们的存储空间来上传(存储)和下载(获取)我们存储在网络空间中的数据信息。

## 10.Activity启动模式

介绍 `Android `启动模式之前，先介绍两个概念`task`和`taskAffinity`

##### task
翻译过来就是“任务”，是一组相互有关联的` activity `集合，可以理解为` Activity `是在 `task` 里面活动的。` task` 存在于一个称为` back stack` 的数据结构中，也就是说，` task `是以栈的形式去管理 `activity` 的，所以也叫可以称为`任务栈`。
##### taskAffinity：
官方文档解释是：`The task that the activity has an affinity for.`，可以翻译为 `activity `相关或者亲和的任务，这个参数标识了一个` Activity`所需要的任务栈的名字。默认情况下，所有`Activity`所需的任务栈的名字为应用的包名。 `taskAffinity `属性主要和` singleTask `启动模式或者 `allowTaskReparenting` 属性配对使用。

#####4种启动模式

#####1.  standard标准模式
也是系统默认的启动模式。假如` activity A `启动了` activity B` ，` activity B `则会运行在 `activity A` 所在的任务栈中。而且每次启动一个 `Activity `，都会重新创建新的实例，不管这个实例在任务中是否已经存在。非` Activity `类型的` context （如 ApplicationContext ）`启动` standard `模式的` Activity `时会报错。非 `Activity `类型的 `context `并没有所谓的任务栈，由于上面第 1 点的原因所以系统会报错。此解决办法就是为待启动` Activity `指定 `FLAG_ACTIVITY_NEW_TASK `标记位，这样启动的时候系统就会为它创建一个新的任务栈。这个时候待启动 `Activity` 其实是以 `singleTask `模式启动的。
#####2.  singleTop 栈顶复用模式
假如` activity A `启动了 `activity B `，就会判断 `A` 所在的任务栈栈顶是否是 `B` 的实例。如果是，则不创建新的 `activity B` 实例而是直接引用这个栈顶实例，同时 `onNewIntent` 方法会被回调，通过该方法的参数可以取得当前请求的信息；如果不是，则创建新的 `activity B `实例。
#####3.  singleTask 栈内复用模式
在第一次启动这个 `Activity `时，系统便会创建一个新的任务，并且初始化` Activity `的实例，放在新任务的底部。不过需要满足一定条件的。那就是需要设置` taskAffinity `属性。前面也说过了，` taskAffinity` 属性是和` singleTask `模式搭配使用的。

![](https://upload-images.jianshu.io/upload_images/5851256-9981ae2d97d0709d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#####4.  singleInstance 单实例模式
这个是` singleTask` 模式的加强版，它除了具有` singleTask `模式的所有特性外，它还有一点独特的特性，那就是此模式的` Activity` 只能单独地位于一个任务栈，不与其他 `Activity `共存于同一个任务栈。

## 11.广播注册

首先写一个类要继承`BroadCastReceiver`

#####第一种：在清单文件中声明，添加

```
<receive android:name=".BroadCastReceiverDemo">
      <intent-filter>
          <action android:name="android.provider.Telephony.SMS_RECEIVED">
      </intent-filter>
</receiver>
```

#####第二种：使用代码进行注册如：

```
    IntentFilter filter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");
    BroadCastReceiverDemo receiver = new BroadCastReceiver();
    registerReceiver(receiver, filter);
```

#####两种注册类型的区别是：
a.第一种是常驻型广播，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行。
b.第二种不是常驻广播，也就是说广播跟随程序的生命周期。

## 12.Android中的ANR

`ANR`的全称`application not responding `应用程序未响应。

ANR 类型|最长ANR时间
----|----
事件分发(点击输入等)：|5s
BroadcastReceiver|10s
Service|20s`


超出执行时间就会产生`ANR`。注意：`ANR`是系统抛出的异常，程序是捕捉不了这个异常的。

#####解决方法:

1.  运行在主线程里的任何方法都尽可能少做事情。特别是，`Activity`应该在它的关键生命周期方法 `（如onCreate()和onResume()）`里尽可能少的去做创建操作。可以采用重新开启子线程的方式，然后使用`Handler+Message `的方式做一些操作，比如更新主线程中的`ui`等。
2.  应用程序应该避免在·BroadcastReceiver·里做耗时的操作或计算。但不再是在子线程里做这些任务`（因为 BroadcastReceiver的生命周期短）`，替代的是，如果响应`Intent`广播需要执行一个耗时的动作的话，应用程序应该启动一个 `Service`。

## 13.ListView优化

#####1.  convertView重用
利用好` convertView`来重用` View`，切忌每次 `getView()` 都新建。`ListView `的核心原理就是重用` View`，如果重用` view` 不改变宽高，重用`View`可以减少重新分配缓存造成的内存频繁分配/回收;
#####2.  ViewHolder优化
使用`ViewHolder`的原因是`findViewById`方法耗时较大，如果控件个数过多，会严重影响性能，而使用`ViewHolder`主要是为了可以省去这个时间。通过`setTag，getTag`直接获取`View`。
#####3.  减少Item View的布局层级
这是所有`Layout`都必须遵循的，布局层级过深会直接导致`View`的测量与绘制浪费大量的时间。
#####4.  adapter中的getView方法尽量少使用逻辑
#####5.  图片加载采用三级缓存，避免每次都要重新加载。
#####6.  尝试开启硬件加速来使ListView的滑动更加流畅。
#####7.  使用 `RecycleView` 代替。

## 14.Android数字签名

1.  所有的应用程序都必须有数字证书，`Android`系统不会安装一个没有数字证书的应用程序
2.  `Android`程序包使用的数字证书可以是自签名的，不需要一个权威的数字证书机构签名认证
3.  如果要正式发布一个`Android `，必须使用一个合适的私钥生成的数字证书来给程序签名。
4.  数字证书都是有有效期的，`Android`只是在应用程序安装的时候才会检查证书的有效期。如果程序已经安装在系统中，即使证书过期也不会影响程序的正常功能。

## 15.Android root机制

`root`指的是你有权限可以再系统上对所有档案有 "读" "写" "执行"的权力。`root`机器不是真正能让你的应用程序具有`root`权限。它原理就跟`linux`下的像`sudo`这样的命令。在系统的`bin`目录下放个`su`程序并属主是`root`并有`suid`权限。则通过`su`执行的命令都具有`Android root`权限。当然使用临时用户权限想把`su`拷贝的`/system/bin`目录并改属性并不是一件容易的事情。这里用到`2`个工具跟`2`个命令。把`busybox`拷贝到你有权限访问的目录然后给他赋予`4755`权限，你就可以用它做很多事了。

## 16.View、surfaceView、GLSurfaceView

##### View

显示视图，内置画布，提供图形绘制函数、触屏事件、按键事件函数等，必须在UI主线程内更新画面，速度较慢

##### SurfaceView

基于`view`视图进行拓展的视图类，更适合`2D`游戏的开发，是`view`的子类，类似使用双缓机制，在新的线程中更新画面所以刷新界面速度比`view`快。

#####GLSurfaceView

基于`SurfaceView`视图再次进行拓展的视图类，专用于`3D`游戏开发的视图，是`surfaceView`的子类，`openGL`专用

### AsyncTask

#####AsyncTask的三个泛型参数说明
1.  第一个参数：传入`doInBackground()`方法的参数类型
2.  第二个参数：传入`onProgressUpdate()`方法的参数类型
3.  第三个参数：传入`onPostExecute()`方法的参数类型，也是`doInBackground()`方法返回的类型

#####运行在主线程的方法:

```
onPostExecute（）
onPreExecute()
onProgressUpdate(Progress...)
```

#####运行在子线程的方法：

```
doInBackground（）
```

#####控制AsyncTask停止的方法：

```
cancel(boolean mayInterruptIfRunning)
```

#####AsyncTask的执行分为四个步骤

1.  继承`AsyncTask。`
2.  实现`AsyncTask`中定义的下面一个或几个方法`onPreExecute()、doInBackground(Params...)、onProgressUpdate(Progress...)、onPostExecute(Result)`。
3.  调用`execute`方法必须在`UI thread`中调用。
4.  该`task`只能被执行一次，否则多次调用时将会出现异常，取消任务可调用`cancel`。

## 17.Android i18n

`I18n `叫做国际化。`Android `对`i18n`和`L10n`提供了非常好的支持。软件在`res/vales` 以及 其他带有语言修饰符的文件夹。如： `values-zh` 这些文件夹中 提供语言，样式，尺寸` xml` 资源。

## 18.NDK

1.  `NDK`是一系列工具集合，`NDK`提供了一系列的工具，帮助开发者迅速的开发`C/C++`的动态库，并能自动将`so`和`Java`应用打成`apk`包。
2.  `NDK`集成了交叉编译器，并提供了相应的`mk`文件和隔离`cpu`、平台等的差异，开发人员只需要简单的修改`mk`文件就可以创建出`so`文件。

## 19.启动一个程序，可以主界面点击图标进入，也可以从一个程序中跳转过去，二者有什么区别？

通过主界面进入，就是设置默认启动的`activity`。在`manifest.xml`文件的`activity`标签中，写以下代码

```
<intent- filter>
    <intent android:name=“android.intent.action.MAIN”>
    <intent android:name=”android:intent.category.LAUNCHER”>
</intent-filter>
```

从另一个组件跳转到目标  activity  ，需要通过  intent  进行跳转。具体

```
Intent intent=new Intent(this,activity.class),startActivity(intent)
```

## 20.内存溢出和内存泄漏有什么区别？何时会产生内存泄漏？

#####内存溢出：
当程序运行时所需的内存大于程序允许的最高内存，这时会出现内存溢出；

#####内存泄漏：
在一些比较消耗资源的操作中，如果操作中内存一直未被释放，就会出现内存泄漏。比如未关闭`io,cursor`。

## 21.sim卡的EF 文件有何作用

`sim`卡就是电话卡，`sim`卡内有自己的操作系统，用来与手机通讯的。`Ef`文件用来存储数据的。

## 22.Activity的状态有几种？

#####主要有以下三种状态：
1.运行
2.暂停
3.停止


## 23.让Activity变成一个窗口

设置`activity的style`属性=`”@android:style/Theme.Dialog”`
```
        <activity
            android:name=".CondorMainActivity"
            android:label="@string/app_name"
            android:theme="@android:style/Theme.Dialog" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```
## 24.android:gravity与android:layout_gravity的区别

`gravity：`
表示组件内元素的对齐方式
`layout_gravity：`
相对于父类容器，该视图组件的对齐方式

## 25.如何退出Activity

#####结束当前`activity`

```
Finish()
killProgress()
System.exit(0)
```

关闭应用程序时，结束所有的`activity`
可以创建一个`List`集合，每新创建一个`activity`，将该`activity`的实例放进`list`中，程序结束时，从集合中取出循环取出`activity`实例，调用`finish()`方法结束

## 26.如果后台的Activity由于某原因被系统回收了，如何在被系统回收之前保存当前状态？

在`onPuase`方法中调用`onSavedInstanceState()`

## 27.Android中的长度单位详解
`Px：`
像素
`Sp与dp`
是长度单位，但是与屏幕的单位密度无关.

## 28.activity，service，intent之间的关系

这三个都是`Android`应用频率非常的组件。`Activity`与`service`是四大核心组件。`Activity`用来加载布局，显示窗口界面，`service`运行后台，没有界面显示，`intent`是`activity`与`service`的通信使者。

## 29.activity之间传递参数，除了intent，广播接收器，contentProvider之外，还有那些方法？


`File：`
文件存储，推荐使用`sharedPreferecnces`
`静态变量`


## 30.Adapter是什么？你所接触过的adapter有那些？

是适配器，用来为列表提供数据适配的。经常使用的`adapter`有`baseadapter`，`arrayAdapter，SimpleAdapter,cursorAdapter,SpinnerAdapter`等

## 31.Fragment与activity如何传值和交互？


`Fragment`对象有一个`getActivity()`的方法，通过该方法与`activity`交互
使用`framentmentManager.findFragmentByXX`可以获取`fragment`对象，在`activity`中直接操作`fragment`对象。


## 32.如果Listview中的数据源发生改变，如何更新listview中的数据

使用`adapter`的`notifyDataSetChanged`方法

## 33.广播接受者的生命周期？


广播接收者的生命周期非常短。当执行`onRecieve`方法之后，广播就会销毁
在广播接受者不能进行耗时较长的操作
在广播接收者不要创建子线程。广播接收者完成操作后，所在进程会变成空进程，很容易被系统回收

## 34.ContentProvider与sqlite有什么不一样的？


`ContentProvider
会对外隐藏内部实现，只需要关注访问`contentProvider`的`uri`即可，`contentProvider`应用在app间共享。
`Sqlite`操作本应用程序的数据库。
`ContentProiver`可以对本地文件进行增删改查操作


## 35.如何保存activity的状态？

默认情况下`activity`的状态系统会自动保存，有些时候需要我们手动调用保存。

当`activity`处于`onPause，onStop`之后，`activity`处于未活动状态，但是`activity`对象却仍然存在。当内存不足，`onPause，onStop`之后的`activity`可能会被系统摧毁。

当通过返回退出`activity`时，`activity`状态并不会保存。

保存`activity`状态需要重写`onSavedInstanceState()`方法，在执行`onPause,onStop`之前调用`onSavedInstanceState`方法，`onSavedInstanceState`需要一个`Bundle`类型的参数，我们可以将数据保存到`bundle`中，通过实参传递给`onSavedInstanceState`方法。

`Activity`被销毁后，重新启动时，在`onCreate`方法中，接受保存的`bundle`参数，并将之前的数据取出。

## 36.Android中activity，context，application有什么不同。

`Content`与`application`都继承与`contextWrapper`，`contextWrapper`继承于`Context`类。

`Context：`
表示当前上下文对象，保存的是上下文中的参数和变量，它可以让更加方便访问到一些资源。
`Context`通常与`activity`的生命周期是一样的，`application`表示整个应用程序的对象。

对于一些生命周期较长的，不要使用`context`，可以使用`application`。

在`activity`中，尽量使用静态内部类，不要使用内部类。内部里作为外部类的成员存在，不是独立于`activity`，如果内存中还有内存继续引用到`context`，`activity`如果被销毁，`context`还不会结束。

## 37.Service 是否在 main thread 中执行, service 里面是否能执行耗时的操作?

默认情况`service`在`main thread`中执行，当`service`在主线程中运行，那在`service`中不要进行一些比较耗时的操作，比如说网络连接，文件拷贝等。

## 38.Service 和 Activity 在同一个线程吗

默认情况下`service`与`activity`在同一个线程，都在`main Thread`，或者`ui`线程中。

如果在清单文件中指定`service`的`process`属性，那么`service`就在另一个进程中运行。

## 39.Service 里面可以弹Toast么

可以。

## 40.在 service 的生命周期方法 onstartConmand()可不可以执行网络操作？如何在 service 中执行网络操作？

可以的，就在`onstartConmand`方法内执行。

## 41.说说 ContentProvider、ContentResolver、ContentObserver 之间的关系


`ContentProvider：`
内容提供者，对外提供数据的操作，`contentProvider.notifyChanged(uir)`：可以更新数据
`contentResolver：`
内容解析者，解析`ContentProvider`返回的数据
`ContentObServer:`
内容监听者，监听数据的改变，`contentResolver.registerContentObServer()`

## 42.请介绍下 ContentProvider 是如何实现数据共享的

`ContentProvider`是一个对外提供数据的接口，首先需要实现`ContentProvider`这个接口，然后重写`query，insert，getType，delete，update`方法，最后在清单文件定义`contentProvider`的访问`uri`。

## 43.Intent 传递数据时，可以传递哪些类型数据？


#####1.基本数据类型以及对应的数组类型
#####2.可以传递`bundle`类型，但是`bundle`类型的数据需要实现`Serializable`或者`parcelable`接口

## 44.Serializable 和 Parcelable 的区别？

如果存储在内存中，推荐使用`parcelable`，使用`serialiable`在序列化的时候会产生大量的临时变量，会引起频繁的`GC`

如果存储在硬盘上，推荐使用`Serializable`，虽然`serializable`效率较低

`Serializable的实现：`
只需要实现`Serializable`接口，就会自动生成一个序列化`id`

`Parcelable的实现：`
需要实现`Parcelable`接口，还需要`Parcelable.CREATER`变量

## 45.请描述一下` Intent` 和 `IntentFilter`


`Intent`是组件的通讯使者，可以在组件间传递消息和数据。
`IntentFilter`是`intent`的筛选器，可以对`intent`的`action，data，catgory，uri`这些属性进行筛选，确定符合的目标组件。


## 46.什么是IntentService？有何优点？

`IntentService `是` Service `的子类，比普通的 `Service `增加了额外的功能。先看` Service `本身存在两个问题：

1.`Service` 不会专门启动一条单独的进程，`Service `与它所在应用位于同一个进程中；
2.`Service` 也不是专门一条新线程，因此不应该在 `Service `中直接处理耗时的任务；

#####特征


会创建独立的 `worker `线程来处理所有的` Intent `请求；
会创建独立的` worker `线程来处理 `onHandleIntent()`方法实现的代码，无需处理多线程问题；
所有请求处理完成后，`IntentService `会自动停止，无需调用 `stopSelf()`方法停止 `Service`；
为 `Service `的 `onBind()`提供默认实现，返回 `null`；
为 `Service `的 `onStartCommand` 提供默认实现，将请求 `Intent` 添加到队列中


#####使用


让  `service`类继承`IntentService`，重写`onStartCommand`和`onHandleIntent`实现

## 47.Android 引入广播机制的用意

从 `MVC` 的角度考虑(应用程序内) 其实回答这个问题的时候还可以这样问，`android `为什么要有那 `4` 大组件，现在的移动开发模型基本上也是照搬的 `web `那一套 `MVC `架构，只不过稍微做了修改。`android `的四大组件本质上就是为了实现移动或者说嵌入式设备上的 `MVC `架构，它们之间有时候是一种相互依存的关系，有时候又是一种补充关系，引入广播机制可以方便几大组件的信息和数据交互。

程序间互通消息(例如在自己的应用程序内监听系统来电)

效率上(参考` UDP `的广播协议在局域网的方便性)

设计模式上(反转控制的一种应用，类似监听者模式)

## 48.ListView 如何提高其效率？

当 `convertView `为空时，用` setTag()`方法为每个 `View` 绑定一个存放控件的 `ViewHolder `对象。当`convertView `不为空， 重复利用已经创建的` view` 的时候， 使用 `getTag()`方法获取绑定的 `ViewHolder`对象，这样就避免了` findViewById `对控件的层层查询，而是快速定位到控件。 复用 `ConvertView`，使用历史的` view`，提升效率 200%

自定义静态类 `ViewHolder`，减少 `findViewById` 的次数。提升效率 50%

异步加载数据，分页加载数据。

使用` WeakRefrence` 引用` ImageView` 对象

## 49.ListView 如何实现分页加载

设置 `ListView `的滚动监听器：`setOnScrollListener(new OnScrollListener{….})`在监听器中有两个方法： 滚动状态发生变化的方法`(onScrollStateChanged)`和` listView `被滚动时调用的方法`(onScroll)`

在滚动状态发生改变的方法中，有三种状态：
手指按下移动的状态：
 `SCROLL_STATE_TOUCH_SCROLL`
惯性滚动（滑翔（flgin）状态）：
` SCROLL_STATE_FLING`:
静止状态：
 `SCROLL_STATE_IDLE:` 

分批加载数据，只关心静止状态：关心最后一个可见的条目，如果最后一个可见条目就是数据适配器（集合）里的最后一个，此时可加载更多的数据。在每次加载的时候，计算出滚动的数量，当滚动的数量大于等于总数量的时候，可以提示用户无更多数据了。

## 50.ListView 可以显示多种类型的条目吗

这个当然可以的，`ListView` 显示的每个条目都是通过 `baseAdapter` 的 `getView(int position,View convertView, ViewGroup parent)`来展示的，理论上我们完全可以让每个条目都是不同类型的view。

比如：从服务器拿回一个标识为` id=1`,那么当` id=1 `的时候，我们就加载类型一的条目，当 `id=2`的时候，加载类型二的条目。常见布局在资讯类客户端中可以经常看到。

除此之外` adapter` 还提供了 `getViewTypeCount（）`和 `getItemViewType(int position)`两个方法。在 `getView `方法中我们可以根据不同的 `viewtype `加载不同的布局文件。

## 51.ListView 如何定位到指定位置

可以通过 `ListView `提供的 `lv.setSelection(listView.getPosition())`方法。

## 52.如何在 ScrollView 中如何嵌入 ListView

通常情况下我们不会在 `ScrollView `中嵌套 `ListView`。

在 `ScrollView` 添加一个 `ListView `会导致` listview` 控件显示不全，通常只会显示一条，这是因为两个控件的滚动事件冲突导致。所以需要通过 `listview` 中的` item` 数量去计算` listview `的显示高度，从而使其完整展示。

现阶段最好的处理的方式是： 自定义 `ListView`，重载 `onMeasure()`方法，设置全部显示。

## 53.Manifest.xml文件中主要包括哪些信息？


`manifest：`
根节点，描述了`package`中所有的内容。
`uses-permission：`
请求你的`package`正常运作所需赋予的安全许可。
`permission： `
声明了安全许可来限制哪些程序能你`package`中的组件和功能。
`instrumentation：`
声明了用来测试此`package`或其他`package`指令组件的代码。
`application：`
包含`package`中`application`级别组件声明的根节点。
`activity：`
Activity是用来与用户交互的主要工具。
`receiver：`
`IntentReceiver`能使的`application`获得数据的改变或者发生的操作，即使它当前不在运行。
`service：`
`Service`是能在后台运行任意时间的组件。
`provider：`
`ContentProvider`是用来管理持久化数据并发布给其他应用程序使用的组件。

## 54.ListView 中图片错位的问题是如何产生的

图片错位问题的本质源于我们的 `listview `使用了缓存` convertView`， 假设一种场景， 一个 `listview`一屏显示九个 `item`，那么在拉出第十个` item` 的时候，事实上该` item `是重复使用了第一个 `item`，也就是说在第一个` item` 从网络中下载图片并最终要显示的时候，其实该 `item `已经不在当前显示区域内了，此时显示的后果将可能在第十个` item `上输出图像，这就导致了图片错位的问题。所以解决办法就是`可见则显示，不可见则不显示`。

## 55.Fragment 的 replace 和 add 方法的区别

`Fragment `本身并没有 `replace` 和 `add `方法，`FragmentManager`才有`replace`和`add`方法。我们经常使用的一个架构就是通过`RadioGroup`切换`Fragment`，每个` Fragment` 就是一个功能模块。

`Fragment `的容器一个` FrameLayout`，`add `的时候是把所有的 `Fragment `一层一层的叠加到了。`FrameLayout `上了，而 `replace `的话首先将该容器中的其他` Fragment `去除掉然后将当前`Fragment`添加到容器中。

一个` Fragment` 容器中只能添加一个` Fragment` 种类，如果多次添加则会报异常，导致程序终止，而` replace` 则无所谓，随便切换。因为通过 `add `的方法添加的 `Fragment`，每个 `Fragment `只能添加一次，因此如果要想达到切换效果需要通过` Fragment` 的的` hide` 和 `show `方法结合者使用。将要显示的` show` 出来，将其他` hide`起来。这个过程 `Fragment `的生命周期没有变化。

通过 `replace` 切换` Fragment`，每次都会执行上一个` Fragment `的 `onDestroyView`，新 `Fragment`的 `onCreateView、onStart、onResume `方法。基于以上不同的特点我们在使用的使用一定要结合着生命周期操作我们的视图和数据。

## 56.Fragment 如何实现类似 Activity 栈的压栈和出栈效果的？

`Fragment `的事物管理器内部维持了一个双向链表结构，该结构可以记录我们每次 `add` 的`Fragment` 和 `replace `的` Fragment`，然后当我们点击 `back` 按钮的时候会自动帮我们实现退栈操作。

## 57.Fragment 在你们项目中的使用

`Fragment `是` android3.0 `以后引入的的概念，做局部内容更新更方便，原来为了到达这一点要把多个布局放到一个 `activity `里面，现在可以用多 `Fragment` 来代替，只有在需要的时候才加载`Fragment`，提高性能。

#####Fragment 的好处：


`Fragment `可以使你能够将 `activity `分离成多个可重用的组件，每个都有它自己的生命周期和`UI`。
`Fragment `可以轻松得创建动态灵活的` UI` 设计，可以适应于不同的屏幕尺寸。从手机到平板电脑。
`Fragment `是一个独立的模块,紧紧地与 `activity` 绑定在一起。可以运行中动态地移除、加入、交换等。
`Fragment `提供一个新的方式让你在不同的安卓设备上统一你的 UI。
`Fragment `解决 `Activity `间的切换不流畅，轻量切换。
`Fragment` 替代` TabActivity `做导航，性能更好。
`Fragment` 在 `4.2.`版本中新增嵌套 `fragment `使用方法，能够生成更好的界面效果。

## 58.如何切换 fragement,不重新实例化

翻看了` Android `官方` Doc`，和一些组件的源代码，发现 `replace()`这个方法只是在上一个 `Fragment`不再需要时采用的简便方法.

正确的切换方式是 `add()`，切换时` hide()`，`add()`另一个 Fragment；再次切换时，只需 `hide()`当前，`show()`另一个。

这样就能做到多个 `Fragment` 切换不重新实例化：

## 59.如何对 Android 应用进行性能分析

如果不考虑使用其他第三方性能分析工具的话，我们可以直接使用` ddms` 中的工具，其实 `ddms` 工具已经非常的强大了。`ddms `中有 `traceview、heap、allocation tracker `等工具都可以帮助我们分析应用的方法执行时间效率和内存使用情况。

`Traceview `是 `Android `平台特有的数据采集和分析工具，它主要用于分析 `Android `中应用程序的 `hotspot（瓶颈）`。`Traceview `本身只是一个数据分析工具，而数据的采集则需要使用 `AndroidSDK` 中的` Debug `类或者利用 `DDMS` 工具。

`heap `工具可以帮助我们检查代码中是否存在会造成内存泄漏的地方。

`allocation tracker `是内存分配跟踪工具

## 60.Android 中如何捕获未捕获的异常

#####UncaughtExceptionHandler


自 定 义 一 个 `Application `， 比 如 叫` MyApplication` 继 承 `Application `实 现`UncaughtExceptionHandler`。
覆写 `UncaughtExceptionHandler` 的` onCreate `和 `uncaughtException `方法。
 注意：上面的代码只是简单的将异常打印出来。在` onCreate` 方法中我们给` Thread `类设置默认异常处理 `handler`，如果这句代码不执行则一切都是白搭。在` uncaughtException `方法中我们必须新开辟个线程进行我们异常的收集工作，然后将系统给杀死。
在 `AndroidManifest `中配置该 `Application：<application android:name="com.example.uncatchexception.MyApplication"`


#####Bug 收集工具 Crashlytics

`Crashlytics   `是专门为移动应用开发者提供的保存和分析应用崩溃的工具。国内主要使用的是友盟做数据统计。
**Crashlytics 的好处：**
    1.`Crashlytics `不会漏掉任何应用崩溃信息。
    2.`Crashlytics` 可以像` Bug `管理工具那样，管理这些崩溃日志。
    3.`Crashlytics` 可以每天和每周将崩溃信息汇总发到你的邮箱，所有信息一目了然。


## 61.如何将SQLite数据库(dictionary.db文件)与apk文件一起发布

把这个文件放在`/res/raw`目录下即可。`res\raw`目录中的文件不会被压缩，这样可以直接提取该目录中的文件，会生成资源`id`。

## 62.什么是 IntentService？有何优点？

`IntentService `是 `Service` 的子类，比普通的 `Service `增加了额外的功能。先看 `Service` 本身存在两个问题：

 `Service` 不会专门启动一条单独的进程，`Service` 与它所在应用位于同一个进程中；
`Service` 也不是专门一条新线程，因此不应该在` Service` 中直接处理耗时的任务；


#####IntentService 特征


会创建独立的 `worker `线程来处理所有的` Intent `请求；
会创建独立的 `worker` 线程来处理` onHandleIntent()`方法实现的代码，无需处理多线程问题；
所有请求处理完成后，`IntentService `会自动停止，无需调用 `stopSelf()`方法停止 `Service`；
为` Service` 的 `onBind()`提供默认实现，返回 `null`；
为 `Service `的 `onStartCommand` 提供默认实现，将请求` Intent `添加到队列中；


## 63.谈谈对Android NDK的理解

`NDK`是一系列工具的集合.`NDK`提供了一系列的工具,帮助开发者快速开发`C或C++`的动态库,并能自动将`so`和`java`应用一起打包成`apk.`这些工具对开发者的帮助是巨大的.`NDK`集成了交叉编译器,并提供了相应的`mk`文件隔离`CPU,平台,ABI`等差异,开发人员只需要简单修改 `mk`文件(指出"哪些文件需要编译","编译特性要求"等),就可以创建出`so`.

`NDK`可以自动地将`so`和`Java`应用一起打包,极大地减轻了开发人员的打包工作.`NDK`提供了一份稳定,功能有限的`API`头文件声明.

`Google`明确声明该`API`是稳定的,在后续所有版本中都稳定支持当前发布的`API`.从该版本的`NDK`中看出,这些 `API`支持的功能非常有限,包含有:`C标准库(libc),标准数学库(libm ),压缩库(libz),Log库(liblog)`.

## 64.AsyncTask使用在哪些场景？它的缺陷是什么？如何解决？

`AsyncTask` 运用的场景就是我们需要进行一些耗时的操作，耗时操作完成后更新主线程，或者在操作过程中对主线程的`UI`进行更新。

#####缺陷：
`AsyncTask`中维护着一个长度为`128`的线程池，同时可以执行`5`个工作线程，还有一个缓冲队列，当线程池中已有`128`个线程，缓冲队列已满时，如果 此时向线程提交任务，将会抛出`RejectedExecutionException。`

#####解决：
由一个控制线程来处理`AsyncTask`的调用判断线程池是否满了，如果满了则线程睡眠否则请求`AsyncTask`继续处理。

## 65.Android 线程间通信有哪几种方式（重要）


1.共享内存（变量）
2.文件，数据库
3.`Handler`
4.`Java` 里的 `wait()，notify()，notifyAll()`


## 66.请解释下 Android 程序运行时权限与文件系统权限的区别？

`apk` 程序是运行在虚拟机上的,对应的是` Android` 独特的权限机制，只有体现到文件系统上时才

#####使用 `linux` 的权限设置。

`linux `文件系统上的权限
  `  -rwxr-x--x system system 4156 2010-04-30 16:13 test.apk`
代表的是相应的用户/用户组及其他人对此文件的访问权限，与此文件运行起来具有的权限完全不相关。比如上面的例子只能说明 `system` 用户拥有对此文件的读写执行权限；`system` 组的用户对此文件拥有读、执行权限；其他人对此文件只具有执行权限。而 `test.apk `运行起来后可以干哪些事情，跟这个就不相关了。千万不要看` apk` 文件系统上属于` system/system` 用户及用户组，或者`root/root` 用户及用户组，就认为` apk` 具有` system` 或 `root `权限


#####Android 的权限规则

`Android `中的`apk `必须签名
基于 `UserID` 的进程级别的安全机制
默认 `apk `生成的数据对外是不可见的
`AndroidManifest.xml` 中的显式权限声明


## 67.Framework 工作方式及原理，Activity 是如何生成一个 view 的，机制是什么？

所有的框架都是基于反射 和 配置文件`（manifest）`的。

#####普通的情况:


`Activity `创建一个 `view` 是通过 `ondraw `画出来的, 画这个` view `之前呢,还会调用 `onmeasure`方法来计算显示的大小.


#####特殊情况：


`Surfaceview `是直接操作硬件的，因为 或者视频播放对帧数有要求，`onDraw` 效率太低，不够使，`Surfaceview` 直接把数据写到显存。


## 68.什么是 AIDL？如何使用？

`aidl `是 `Android interface definition Language` 的英文缩写，意思 `Android` 接口定义语言。

使用` aidl `可以帮助我们发布以及调用远程服务，实现跨进程通信。


将服务的 `aidl` 放到对应的 `src `目录，工程的 `gen `目录会生成相应的接口类
我们通过 `bindService（Intent，ServiceConnect，int）`方法绑定远程服务，在 `bindService`中 有 一 个 `ServiceConnect` 接 口 ， 我 们 需 要 覆 写 该 类 的`onServiceConnected(ComponentName,IBinder)`方法，这个方法的第二个参数` IBinder `对象其实就是已经在 `aidl `中定义的接口，因此我们可以将` IBinder` 对象强制转换为` aidl `中的接口类。我们通过` IBinder` 获取到的对象（也就是 `aidl `文件生成的接口）其实是系统产生的代理对象，该代理对象既可以跟我们的进程通信， 又可以跟远程进程通信， 作为一个中间的角色实现了进程间通信。


## 69.AIDL 的全称是什么?如何工作?能处理哪些类型的数据？

`AIDL `全称 `Android Interface Definition Language`（AndRoid 接口描述语言） 是一种接口描述语言; 编译器可以通过 `aidl `文件生成一段代码，通过预先定义的接口达到两个进程内部通信进程跨界对象访问的目的。需要完成两件事情：

1.引入` AIDL` 的相关类.; 
2.调用` aidl `产生的 `class`


理论上, 参数可以传递基本数据类型和 `String`, 还有就是 `Bundle `的派生类, 不过在` Eclipse `中,目前的 `ADT` 不支持` Bundle` 做为参数。

## 70.Android 判断SD卡是否存在

```	
/**
	 * 判断SD是否挂载
	 */
	public static boolean isSDCardMount() {
		return Environment.getExternalStorageState().equals(
				Environment.MEDIA_MOUNTED);
	}
```
## 71.Android中任务栈的分配

`Task`实际上是一个`Activity`栈，通常用户感受的一个`Application`就是一个`Task`。从这个定义来看，`Task`跟`Service`或者其他`Components`是没有任何联系的，它只是针对`Activity`而言的。

`Activity`有不同的启动模式, 可以影响到`task`的分配

## 72.SQLite支持事务吗? 添加删除如何提高性能?

在`sqlite`插入数据的时候默认一条语句就是一个事务，有多少条数据就有多少次磁盘操作 比如`5000`条记录也就是要`5000`次读写磁盘操作。

添加事务处理，把多条记录的插入或者删除作为一个事务

## 73.Android中touch事件的传递机制是怎样的?


1.`Touch`事件传递的相关`API`有`dispatchTouchEvent、onTouchEvent、onInterceptTouchEvent `
2.`Touch`事件相关的类有`View、ViewGroup、Activity `
3.`Touch`事件会被封装成`MotionEvent`对象，该对象封装了手势按下、移动、松开等动作 
4.`Touch`事件通常从`Activity#dispatchTouchEvent`发出，只要没有被消费，会一直往下传递，到最底层的`View`。 
5.如果`Touch`事件传递到的每个`View`都不消费事件，那么`Touch`事件会反向向上传递,最终交由`Activity#onTouchEvent`处理. 
6.`onInterceptTouchEvent`为`ViewGroup`特有，可以拦截事件. 
7.`Down`事件到来时，如果一个`View`没有消费该事件，那么后续的`MOVE/UP`事件都不会再给它

## 74.描述下Handler 机制


#####1)Looper: 
一个线程可以产生一个`Looper`对象，由它来管理此线程里的`MessageQueue(消息队列)`。 
#####2)Handler: 
你可以构造`Handler`对象来与`Looper`沟通，以便`push`新消息到`MessageQueue`里;或者接收`Looper`从`Message Queue`取出所送来的消息。
3) Message Queue(消息队列):
#####用来存放线程放入的消息。 
#####4)线程：
`UIthread `通常就是`main thread`，而`Android`启动程序时会替它建立一个`MessageQueue`。


`Hander`持有对`UI`主线程消息队列`MessageQueue`和消息循环`Looper`的引用，子线程可以通过`Handler`将消息发送到`UI`线程的消息队列`MessageQueue`中。

## 75.自定义view的基本流程


1.自定义`View`的属性 编写`attr.xml`文件 
2.在`layout`布局文件中引用，同时引用命名空间 
3.在`View`的构造方法中获得我们自定义的属性 ，在自定义控件中进行读取（构造方法拿到`attr.xml`文件值） 
4.重写`onMesure `
5.重写`onDraw`

## 76.子线程发消息到主线程进行更新 UI，除了 handler 和 AsyncTask，还有什么？

#####用 Activity 对象的 runOnUiThread 方法更新

在子线程中通过 `runOnUiThread()`方法更新` UI`：
如果在非上下文类中`（Activity）`，可以通过传递上下文实现调用；


#####用 View.post(Runnable r)方法更新 UI

## 77.子线程中能不能 new handler？为什么？

不能,如果在子线程中直接 `new Handler()`会抛出异常 `java.lang.RuntimeException: Can'tcreate handler inside thread that has not called`



## 78.Android 中的动画有哪几类，它们的特点和区别是什么

#####Frame Animation(帧动画)
主要用于播放一帧帧准备好的图片，类似`GIF`图片，优点是使用简单方便、缺点是需要事先准备好每一帧图片；

#####Tween Animation(补间动画)
仅需定义开始与结束的关键帧，而变化的中间帧由系统补上，优点是不用准备每一帧，缺点是只改变了对象绘制，而没有改变`View`本身属性。因此如果改变了按钮的位置，还是需要点击原来按钮所在位置才有效。

#####Property Animation(属性动画)
是`3.0`后推出的动画，优点是使用简单、降低实现的复杂度、直接更改对象的属性、几乎可适用于任何对象而仅非`View`类，主要包括`ValueAnimator`和`ObjectAnimator`

## 79.如何修改 Activity 进入和退出动画

#####可以通过两种方式  
一 是通过定义 `Activity `的主题 
通过设置主题样式在` styles.xml `中编辑如下代码：
```
添加 themes.xml 文件： 
在 AndroidManifest.xml 中给指定的 Activity 指定 theme。 
```
二 是通过覆写 `Activity` 的`overridePendingTransition` 方法。

覆写 `overridePendingTransition `方法
```
overridePendingTransition(R.anim.fade, R.anim.hold);
```

## 80.Android与服务器交互的方式中的对称加密和非对称加密是什么?

对称加密，就是加密和解密数据都是使用同一个`key`，这方面的算法有`DES`。
非对称加密，加密和解密是使用不同的`key`。发送数据之前要先和服务端约定生成公钥和私钥，使用公钥加密的数据可以用私钥解密，反之。这方面的算法有`RSA`。`ssh` 和` ssl`都是典型的非对称加密。

## 82.事件分发中的 onTouch 和 onTouchEvent 有什么区别，又该如何使用？

这两个方法都是在` View` 的 `dispatchTouchEvent `中调用的，`onTouch `优先于 `onTouchEvent`执行。如果在` onTouch` 方法中通过返回` true `将事件消费掉，`onTouchEvent `将不会再执行。

另外需要注意的是，`onTouch `能够得到执行需要两个前提条件
第一 `mOnTouchListener` 的值不能为空，
第二当前点击的控件必须是 `enable `的。
因此如果你有一个控件是非 `enable `的，那么给它注册` onTouch` 事件将永远得不到执行。对于这一类控件，如果我们想要监听它的 `touch `事件，就必须通过在该控件中重写 `onTouchEvent `方法来实现。

## 83.属性动画，例如一个 button 从 A 移动到 B 点，B 点还是可以响应点击事件，这个原理是什么？

补间动画只是显示的位置变动，View 的实际位置未改变，表现为 View 移动到其他地方，点击事件仍在原处才能响应。而属性动画控件移动后事件相应就在控件移动后本身进行处理

## 84.谈谈你在工作中是怎样解决一个 bug

异常附近多打印 `log` 信息；
分析` log `日志，实在不行的话进行断点调试；
调试不出结果，上 `Stack Overflow `贴上异常信息，请教大牛
再多看看代码，或者从源代码中查找相关信息
实在不行就 `GG `了，找师傅来解决！


## 85.嵌入式操作系统内存管理有哪几种， 各有何特性

页式，段式，段页，用到了`MMU`,虚拟空间等技术

## 86.开发中都使用过哪些框架、平台


- EventBus（事件处理）    
- xUtils（网络、图片、ORM）
- JPush（推送平台）
- 友盟（统计平台）
- 有米（优米）（广告平台）
- 百度地图
- bmob（服务器平台、短信验证、邮箱验证、第三方支付）
- 阿里云 OSS（云存储）
- ShareSDK（分享平台、第三方登录）
- Gson（解析 json 数据框架）
- imageLoader （图片处理框架）
- zxing （二维码扫描）
- anroid-asyn-http（网络通讯）
- DiskLruCache(硬盘缓存框架)
- Viatimo（多媒体播放框架）
- universal-image-loader(图片缓存框架)
- 讯飞语音（语音识别）


## 87.谈谈你对 Bitmap 的理解, 什么时候应该手动调用 bitmap.recycle()

`Bitmap` 是 `android` 中经常使用的一个类，它代表了一个图片资源。 `Bitmap` 消耗内存很严重，如果不注意优化代码，经常会出现 `OOM `问题，优化方式通常有这么几种：
1.使用缓存；
2.压缩图片；
3.及时回收；

至于什么时候需要手动调用 `recycle`，这就看具体场景了，原则是当我们不再使用 `Bitmap` 时，需要回收之。另外，我们需要注意，`2.3` 之前 `Bitmap` 对象与像素数据是分开存放的，`Bitmap` 对象存在`java Heap `中而像素数据存放在 `Native Memory `中， 这时很有必要调用` recycle` 回收内存。 但是 `2.3`之后，`Bitmap` 对象和像素数据都是存在` Heap` 中，`GC` 可以回收其内存。

## 88.请介绍下 AsyncTask 的内部实现和适用的场景

`AsyncTask `内部也是 `Handler` 机制来完成的，只不过 `Android `提供了执行框架来提供线程池来执行相应地任务，因为线程池的大小问题，所以 `AsyncTask` 只应该用来执行耗时时间较短的任务，比如` HTTP` 请求，大规模的下载和数据库的更改不适用于 `AsyncTask`，因为会导致线程池堵塞，没有线程来执行其他的任务，导致的情形是会发生` AsyncTask` 根本执行不了的问题

## 89.Activity间通过Intent传递数据大小有没有限制？

`Intent`在传递数据时是有大小限制的，这里官方并未详细说明，不过通过实验的方法可以测出数据应该被限制在`1MB`之内`（1024KB）`，笔者采用的是传递`Bitmap`的方法，发现当图片大小超过`1024（准确地说是1020左右）`的时候，程序就会出现闪退、停止运行等异常(不同的手机反应不同)，因此可以判断Intent的传输容量在`1MB`之内。

## 90.你一般在开发项目中都使用什么设计模式？如何来重构，优化你的代码？

较为常用的就是单例设计模式，工厂设计模式以及观察者设计模式,

一般需要保证对象在内存中的唯一性时就是用单例模式,例如对数据库操作的 `SqliteOpenHelper `的对象。

工厂模式主要是为创建对象提供过渡接口，以便将创建对象的具体过程屏蔽隔离起来，达到提高灵活性的目的。

观察者模式定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新

## 91.Android 应用中验证码登陆都有哪些实现方案


从服务器端获取图片
通过短信服务，将验证码发送给客户端


## 92.定位项目中，如何选取定位方案，如何平衡耗电与实时位置的精度？

开始定位，`Application `持有一个全局的公共位置对象，然后隔一定时间自动刷新位置，每次刷新成功都把新的位置信息赋值到全局的位置对象， 然后每个需要使用位置请求的地方都使用全局的位置信息进行请求。


#####该方案好处：
请求的时候无需再反复定位，每次请求都使用全局的位置对象，节省时间。
#####该方案弊端：
耗电，每隔一定时间自动刷新位置，对电量的消耗比较大。


按需定位，每次请求前都进行定位。这样做的好处是比较省电，而且节省资源，但是请求时间会变得相对较长。

## 93.andorid 应用第二次登录实现自动登录

前置条件是所有用户相关接口都走` https`，非用户相关列表类数据走` http`。

#####步骤

第一次登陆 `getUserInfo `里带有一个长效` token`，该长效 `token `用来判断用户是否登陆和换取短 `token` 
把长效 `token `保存到 `SharedPreferences `
接口请求用长效 `token` 换取短`token`，短 `token` 服务端可以根据你的接口最后一次请求作为标示，超时时间为一天。
所有接口都用短效` token` 
如果返回短效 `token `失效，执行第`3`步，再直接当前接口 
如果长效 `token `失效（用户换设备或超过一月），提示用户登录。


## 94.说说 LruCache 底层原理

`LruCache` 使用一个` LinkedHashMap `简单的实现内存的缓存，没有软引用，都是强引用。

如果添加的数据大于设置的最大值，就删除最先缓存的数据来调整内存。`maxSize `是通过构造方法初始化的值，他表示这个缓存能缓存的最大值是多少。

`size` 在添加和移除缓存都被更新值， 他通过 `safeSizeOf` 这个方法更新值。` safeSizeOf` 默认返回 `1`，但一般我们会根据` maxSize `重写这个方法，比如认为` maxSize `代表是` KB` 的话，那么就以` KB` 为单位返回该项所占的内存大小。

除异常外，首先会判断 `size `是否超过` maxSize`，如果超过了就取出最先插入的缓存，如果不为空就删掉，并把 `size` 减去该项所占的大小。这个操作将一直循环下去，直到 `size` 比 `maxSize` 小或者缓存为空。

## 95.jni 的调用过程?


安装和下载 `Cygwin`，下载` Android NDK`。
`ndk` 项目中 `JNI `接口的设计。
使用 `C/C++`实现本地方法。
`JNI` 生成动态链接库`.so `文件。
将动态链接库复制到 `java` 工程，在` java` 工程中调用，运行` java` 工程即可。


## 96.一条最长的短信息约占多少byte?

中文`70(`包括标点)，英文`160`，`160`个字节。

## 98.即时通讯是是怎么做的?

使用`asmark `开源框架实现的即时通讯功能.该框架基于开源的` XMPP `即时通信协议，采用 `C／S `体系结构，通过` GPRS `无线网络用` TCP` 协议连接到服务器，以架设开源的`Openfn'e `服务器作为即时通讯平台。

客户端基于 `Android` 平台进行开发。负责初始化通信过程，进行即时通信时，由客户端负责向服务器发起创建连接请求。系统通过` GPRS `无线网络与 Internet 网络建立连接，通过服务器实现与`Android `客户端的即时通信脚。

服务器端则采用 `Openfire` 作为服务器。 允许多个客户端同时登录并且并发的连接到一个服务器上。服务器对每个客户端的连接进行认证，对认证通过的客户端创建会话，客户端与服务器端之间的通信就在该会话的上下文中进行。

## 99.怎样对 android 进行优化？


- 对` listview `的优化。
- 对图片的优化。
- 对内存的优化。
- 具体一些措施
- 尽量不要使用过多的静态类` static`
- 数据库使用完成后要记得关闭 `cursor`
- 广播使用完之后要注销

## 100.如果有个100M大的文件，需要上传至服务器中，而服务器form表单最大只能上传2M，可以用什么方法。

首先来说使用`http`协议上传数据，特别在`android`下，跟`form`没什么关系。

传统的在`web`中，在`form`中写文件上传，其实浏览器所做的就是将我们的数据进行解析组拼成字符串，以流的方式发送到服务器，且上传文件用的都是`POST`方式，`POST`方式对大小没什么限制。

回到题目，可以说假设每次真的只能上传`2M`，那么可能我们只能把文件截断，然后分别上传了，断点上传。


**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
