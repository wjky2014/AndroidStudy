 
##### 和您一起终身学习，这里是程序员Android




通过本篇文章，您将收获以下内容
> 一、NullPointerException 空指针
> 二、ClassCastException 类型转换异常
> 三、IndexOutOfBoundsException 下标越界异常
> 四、ActivityNotFoundException  Activity未找到异常
> 五、IllegalStateException   非法状态异常
> 六、ArrayIndexOutOfBoundsException 数组越界异常
> 七、SecurityException 安全异常
> 八、llegalArgumentException: Service not registered 服务未注册异常
>九、BadTokenException
>十、DeadObjectException 

`Exception` 在`Android` 中经常会遇到，那么遇到异常我们该如何解决，本文将举例解决部分`Android`看法中遇到的异常。

#一、NullPointerException 空指针

`NullPointerException `在开发中经常会碰到，比如引用的对象为空，数组为空等等都会引起空指针异常，如不及时处理，就会导致 应用`Crash`。

## 1.  数组 NullPointerException 

不能向一个`null `数组元素赋值，获取长度，否则报
`NullPointerException: Attempt to write to null array`和
`NullPointerException Attempt to get length of null array`，以下代码会引起上面两种空指针异常。

##2. 数组NullPointerException 代码举例

```
	public static void ArrayNullPointer() {
		/**
		 * 数组空指针 NullPointerException
		 * 
		 * 1.获取null数组长度
		 * 2.为null 数组元素复制
		 * */
		int[] array = null;
		// 1. NullPointerException: Attempt to get length of null array
		int length = array.length;
		// 2. NullPointerException: Attempt to write to null array
		array[0] = 1;

	}
```
![NullPointerException 代码举例](http://upload-images.jianshu.io/upload_images/5851256-7e00c0b8e3c99029.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##3. 数组NullPointerException Log 举例

- Log 信息如下

获取 空数组长度导致的 `NullPointerException ` 如下：
```
12-27 17:17:44.627  8839  8839 E AndroidRuntime:  Caused by: java.lang.NullPointerException: 
                                                   Attempt to get length of null array
12-27 17:17:44.627  8839  8839 E AndroidRuntime: 	at com.programandroid.Exception.NullPointerException.ArrayNullPointer
                                                   //产生空指针代码行
                                                   (NullPointerException.java:32)
```
##4. Log 分析如下
![数组NullPointerException ](http://upload-images.jianshu.io/upload_images/5851256-d1c1afc11d816389.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


空数组无法获取下标内容，如果获取则会导致` NullPointerException ` 
```
12-27 17:23:24.168 11649 11649 E AndroidRuntime: Caused by: java.lang.NullPointerException: Attempt to write to null array
12-27 17:23:24.168 11649 11649 E AndroidRuntime: 	at com.programandroid.Exception.NullPointerException.ArrayNullPointer(NullPointerException.java:34)
12-27 17:23:24.168 11649 11649 E AndroidRuntime: 	at com.programandroid.Exception.ExceptionActivity.NullPointerException(ExceptionActivity.java:37)
```




## 5. `Object` 对象 `NullPointerException` 

对象空指针,这个是常见的空指针，主要是因为引用一个`null` 对象，进而导致空指针，常报以下错误
`Attempt to invoke a virtual method on a null object reference`，以下代码可能会引起空指针异常。

##6. object 对象 NullPointerException 代码举例

 简单代码举例如下：
```
	public static void ListNullPointer() {

		ArrayList<String> mArrayList = null;
			mArrayList.size();
	}
```
![Object  对象 NullPointerException  ](http://upload-images.jianshu.io/upload_images/5851256-a402aaea50c863de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##7. object 对象 NullPointerException log 举例
- Log 信息如下：

```
12-27 17:28:22.565 12725 12725 E AndroidRuntime: Caused by: java.lang.NullPointerException: Attempt to invoke a virtual method on a null object reference
12-27 17:28:22.565 12725 12725 E AndroidRuntime: 	at com.programandroid.Exception.NullPointerException.ListNullPointer(NullPointerException.java:45)
12-27 17:28:22.565 12725 12725 E AndroidRuntime: 	at com.programandroid.Exception.ExceptionActivity.NullPointerException(ExceptionActivity.java:37)
```
##8. object 对象 NullPointerException Log 分析如下：
![Object NullPointerException  ](http://upload-images.jianshu.io/upload_images/5851256-8bc9128957d36d1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##9. NullPointerException  解决方案

- 1. 使用时多注意判断对象是否为空
规避空指针举例如下：
```
	public static void ListNullPointer() {

		ArrayList<String> mArrayList = null;
		if (mArrayList != null) {
			mArrayList.size();
		}
	}
```
![使用对象是，最好判断对象是否为空](http://upload-images.jianshu.io/upload_images/5851256-b5b0e1694e1637de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 2. 使用`try-catch`将抛出的异常抓住

`try-catch` 可以抓住抛出的异常，使应用程序不崩溃，但是，这个不是从根本上解决问题，会引起一些莫名其妙的问题。
```
	public static void ListNullPointer() {
			try {
				ArrayList<String> mArrayList = null;
				mArrayList.size();
			} catch (Exception e) {
				// TODO: handle exception
			}
	}

```
![try-catch 代码异常，防止app crash](http://upload-images.jianshu.io/upload_images/5851256-088de958310edfc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 二、 ClassCastException 类型转换异常

`ClassCastException` 类型转换异常:
此异常发生在类型转换时，并且在编译期间，编译器不会提示报错，但是当运行时，如果存在此异常，可能会导致`app `崩溃 `crash`。
比如当`父类`强制转换为`子类`时，ClassCastException  就会发生

## 1. 以下代码 会引起 ClassCastException 

请勿 父类强制转换为子类，否则就会发生`ClassCastException`异常。
```
public void ClassCastExample() {
		Fruit banana = new Fruit();
		/**
		 * ClassCastException
		 * 
		 * 1. 此处强制转换，会导致 app 编译没问题，运行挂掉， Caused by:
		 * java.lang.ClassCastException:
		 * com.programandroid.Exception.ExceptionActivity$ Fruit cannot be cast
		 * to com.programandroid.Exception.ExceptionActivity$Apple
		 * 
		 ***/
		Apple apple = (Apple) banana;

	}

	/**
	 * ClassCastException
	 * 
	 * 2. 此处强转回导致app crash return (Apple) banana;
	 * */
	public Apple isRight() {
		Fruit banana = new Fruit();
		return (Apple) banana;
	}

	class Fruit {
		public Fruit() {
		}
	}

	class Apple extends Fruit {
		public Apple() {
		}
	}
```

![ClassCastException 类型转换异常举例](http://upload-images.jianshu.io/upload_images/5851256-7293b543f9acc2f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2. ClassCastException Log 举例

`ClassCastException `通常会打印以下类似信息
```
Caused by: java.lang.ClassCastException:
com.programandroid.Exception.ExceptionActivity$
Fruit cannot be cast to com.programandroid.Exception.ExceptionActivity$Apple
```
##3. ClassCastException Log 分析
![ClassCastException log 分析
](http://upload-images.jianshu.io/upload_images/5851256-ae8a1cb57edbafe9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4. ClassCastException  解决方案

使用`try-catch`抓住异常,或者从代码上解决根本问题。

![使用 try-catch抓住 ClassCastException异常  ](http://upload-images.jianshu.io/upload_images/5851256-777bd9f3a772d4c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 5. Android 手机 Settings ClassCastException 解决方案

举例是为了更好的解决开发中的异常。比如在开发中，使用 `monkey` 测试`Settings`模块时，报出的 `ClassCastException`，`Settings`代码比较多，一时也无法看完，此时，`try-catch` 也是一种不错的选择。
比如`monkey`测试某平台代码时，报出以下异常

- log 信息如下：

```
FATAL EXCEPTION: ApplicationsState.Loader
01-05 03:36:56.101  6304  6941 E AndroidRuntime: Process: com.android.settings, PID: 6304
01-05 03:36:56.101  6304  6941 E AndroidRuntime: java.lang.ClassCastException: 
                                                   com.android.settings.datausage.AppStateDataUsageBridge$DataUsageState 
												   cannot be cast to com.android.settings.notification.NotificationBackend$AppRow
												   
01-05 03:36:56.101  6304  6941 E AndroidRuntime: 	at com.android.settings.applications.AppStateNotificationBridge$3.filterApp(AppStateNotificationBridge.java:110)
```
##6. Settings ClassCastException  Log分析

![Settings ClassCastException Log1](http://upload-images.jianshu.io/upload_images/5851256-462eed04cd62621f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Settings ClassCastException Log2](http://upload-images.jianshu.io/upload_images/5851256-e8685d67ae4b3cce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##7. Setting crash ClassCastException  解决方案：

![try-catch 异常报错的地方 ](http://upload-images.jianshu.io/upload_images/5851256-99521adeb4d734e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![try-catch 异常报错的地方](http://upload-images.jianshu.io/upload_images/5851256-21f3b82cd1f93e68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![try-catch 异常报错的地方](http://upload-images.jianshu.io/upload_images/5851256-cf9b09353d15ef0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 # 三、IndexOutOfBoundsException 下标越界异常

List 在开发中经常会被用的，那么错误的使用下标，将会导致`IndexOutOfBoundsException` 越界异常。以下代码就会引起`IndexOutOfBoundsException `异常

##1. IndexOutOfBoundsException 代码举例

![IndexOutOfBoundsException  异常举例](http://upload-images.jianshu.io/upload_images/5851256-19ee5a2c825010b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2. IndexOutOfBoundsException Log举例

- Log 信息如下：
```
12-27 17:41:24.231 16891 16891 E AndroidRuntime: Caused by: java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
12-27 17:41:24.231 16891 16891 E AndroidRuntime: 	at java.util.ArrayList.get(ArrayList.java:411)
12-27 17:41:24.231 16891 16891 E AndroidRuntime: 	at com.programandroid.Exception.IndexOutOfBoundsException.isAppOnRecent(IndexOutOfBoundsException.java:40)
12-27 17:41:24.231 16891 16891 E AndroidRuntime: 	at com.programandroid.Exception.ExceptionActivity.IndexOutOfBoundsException(ExceptionActivity.java:80)
```
##3. Log 分析如下：
![IndexOutOfBoundsException Log分析](http://upload-images.jianshu.io/upload_images/5851256-a30514eb4d5f5be6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##4. IndexOutOfBoundsException 解决方案

在使用时判断对象内容是否为0.

![使用判断List 的size是否为0](http://upload-images.jianshu.io/upload_images/5851256-828d998c037b32ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 四、ActivityNotFoundException

`ActivityNotFoundException` 常见于`Eclipse` 开发`Android`中,Android studio 已经帮忙自动生成Activity，以及布局文件。
主要原因是未在`AndroidMainfest.xml `文件中注册，如未注册，会引起`app crash` ，`crash log`如下：
`ActivityNotFoundException: Unable to find explicit activity class`

##1. ActivityNotFoundException 代码举例
比如以下代码会引起此异常
![Activity未在Androidmainfest.xml 中注册会引起ActivityNotFoundException ](http://upload-images.jianshu.io/upload_images/5851256-00addaedc4848f2a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2. ActivityNotFoundException Log 举例

- Log信息如下：
```
12-27 17:46:05.994 17893 17893 E AndroidRuntime: Caused by: android.content.ActivityNotFoundException: Unable to find explicit activity class {com.programandroid/com.programandroid.Test.TestActivity}; have you declared this activity in your AndroidManifest.xml?
12-27 17:46:05.994 17893 17893 E AndroidRuntime: 	at android.app.Instrumentation.checkStartActivityResult(Instrumentation.java:1810)
```

##3. Log 分析如下：

![ActivityNotFoundException Log分析](http://upload-images.jianshu.io/upload_images/5851256-7b08f70d3e9ba250.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##4. ActivityNotFoundException 解决方案

在`AndroidMainfest.xml`中注册即可
![四大组件一定，一定要在AndroidMainfest.xml 中注册](http://upload-images.jianshu.io/upload_images/5851256-e616e9afad021709.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#  五、IllegalStateException

`IllegalStateException` 非法状态异常，是因为软件中代码状态非法导致的。
以下代码会引起`IllegalStateException` 。当`Button`控件声明`android:onClick="IllegalStateException"` 却未在`Java`代码中使用时，点击`Button`，就会出现此类异常。

##1. IllegalStateException 代码举例

![IllegalStateException 代码举例](http://upload-images.jianshu.io/upload_images/5851256-bf9d34cafcca7763.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2. IllegalStateException Log 举例

- log信息如下：
```
12-27 16:07:41.158  1715  1715 E AndroidRuntime: FATAL EXCEPTION: main
12-27 16:07:41.158  1715  1715 E AndroidRuntime: Process: com.programandroid, PID: 1715
12-27 16:07:41.158  1715  1715 E AndroidRuntime: java.lang.IllegalStateException: 
                                                Could not find method IllegalStateException(View) in a parent 
												or ancestor Context for android:onClick attribute defined on view class 
												android.widget.Button with id 'btn_on_click'
12-27 16:07:41.158  1715  1715 E AndroidRuntime: 	at android.view.View$DeclaredOnClickListener.resolveMethod(View.java:4781)
12-27 16:07:41.158  1715  1715 E AndroidRuntime: 	at android.view.View$DeclaredOnClickListener.onClick(View.java:4740)
```
##3. IllegalStateException   Log分析如下：

![IllegalStateException  Log截图](http://upload-images.jianshu.io/upload_images/5851256-608413e16f4adb82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



##4. IllegalStateException 解决方案

`IllegalStateException`  类异常很多，不同的代码会有不同的解决方案，上述举例解决方案如下
![IllegalStateException ](http://upload-images.jianshu.io/upload_images/5851256-af709765d6fed799.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 六、 ArrayIndexOutOfBoundsException 数组越界异常

数组在代码中经常被用到，当适用数组下标不当时，就会出现`ArrayIndexOutOfBoundsException`。比如数组长度为`4`，但你要引用下标为`5`的元素，这时候，就会异常`crash`。

##1. ArrayIndexOutOfBoundsException 代码举例：

```
	public static void ArrayIndexOutOfBounds() {

		String[] mStrings = { "a", "b", "c", "d" };
		String testsString = mStrings[5];
	}
```
![ArrayIndexOutOfBoundsException  代码举例](http://upload-images.jianshu.io/upload_images/5851256-e131b97d346fb7c0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2. ArrayIndexOutOfBoundsException Log举例：
- Log信息如下：
```
12-27 17:51:15.420 19185 19185 E AndroidRuntime: Caused by: java.lang.ArrayIndexOutOfBoundsException: length=4; index=5
12-27 17:51:15.420 19185 19185 E AndroidRuntime: 	at com.programandroid.Exception.ArrayIndexOutOfBoundsException.ArrayIndexOutOfBounds(ArrayIndexOutOfBoundsException.java:20)
12-27 17:51:15.420 19185 19185 E AndroidRuntime: 	at com.programandroid.Exception.ExceptionActivity.ArrayIndexOutOfBoundsException(ExceptionActivity.java:105)
12-27 17:51:15.420 19185 19185 E AndroidRuntime: 	... 11 more
```
##3. ArrayIndexOutOfBoundsException Log分析如下：

![ArrayIndexOutOfBoundsException Log分析](http://upload-images.jianshu.io/upload_images/5851256-a33bb88c964c6549.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4. ArrayIndexOutOfBoundsException解决方案

- 1.正确使用数组下标
- 2.如果不确定数组长度，请先获取长度，然后在判断下标是否大于等于数组长度。
- 3.try-catch 抓住异常，防止crash，但不能从根本上解决问题。
 
# 七、SecurityException 安全异常

`SecurityException` 安全异常在`Android` 中也会经常发生，主要是`Android` 的安全机制原因造成的，为了管理应用获取手机的一些敏感信息，`Android `安全机制规定，必须在`AndroidMainfest.xml` 文件中声明，并且，`Android 6.0 `之后，获取手机敏感信息时候，需要动态申请权限，只有用户授权后才可以获取手机敏感信息。

##1. SecurityException 代码举例

获取手机的IMEI 号属于手机的敏感信息
```
/**
	 * 
	 * <!-- 读取手机IMEI的设备权限 -->
	 * 
	 * <uses-permission android:name="android.permission.READ_PHONE_STATE" />
	 * */
	public static String getIMEI(Context context) {
		TelephonyManager tm = (TelephonyManager) context
				.getSystemService(Context.TELEPHONY_SERVICE);
		String deviceId = tm.getDeviceId();
		if (deviceId == null) {
			return "UnKnown";
		} else {
			return deviceId;
		}
	}
```
![获取手机IMEI号 ](http://upload-images.jianshu.io/upload_images/5851256-5db49bd190e9f4d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##2. SecurityException log举例
```
12-27 18:05:55.663 21467 21467 E AndroidRuntime: Caused by: java.lang.SecurityException: getDeviceId: Neither user 10117 nor current process has android.permission.READ_PHONE_STATE.
12-27 18:05:55.663 21467 21467 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1683)
12-27 18:05:55.663 21467 21467 E AndroidRuntime: 	at android.os.Parcel.readException(Parcel.java:1636)
12-27 18:05:55.663 21467 21467 E AndroidRuntime: 	at com.android.internal.telephony.ITelephony$Stub$Proxy.getDeviceId(ITelephony.java:4281)
```
##3. SecurityException log 分析
![SecurityException log 分析](http://upload-images.jianshu.io/upload_images/5851256-c5ef9c9cd59d760e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##4. SecurityException 解决方案

`Android 6.0 `之前，在`AndroidMainfest.xml `中申请权限即可，
`Android 6.0` 之后，请动态申请权限。

![AndroidMainfest.xml 中申请权限](http://upload-images.jianshu.io/upload_images/5851256-7979a2d837d2d710.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 八、IllegalArgumentException: Service not registered 服务未注册异常

## 1.报错信息如下：
```
01-30 09:10:26.257 23681 23681 W System.err: java.lang.IllegalArgumentException: Service not registered: com.programandroid.Exception.ExceptionActivity$1@5f3161e
01-30 09:10:26.257 23681 23681 W System.err: 	at android.app.LoadedApk.forgetServiceDispatcher(LoadedApk.java:1363)
01-30 09:10:26.257 23681 23681 W System.err: 	at android.app.ContextImpl.unbindService(ContextImpl.java:1499)
01-30 09:10:26.257 23681 23681 W System.err: 	at android.content.ContextWrapper.unbindService(ContextWrapper.java:648)
01-30 09:10:26.257 23681 23681 W System.err: 	at com.programandroid.Exception.ExceptionActivity.ServiceNotRegisteredCrash(ExceptionActivity.java:276)
01-30 09:10:26.257 23681 23681 W System.err: 	at java.lang.reflect.Method.invoke(Native Method)
01-30 09:10:26.258 23681 23681 W System.err: 	at android.view.View$DeclaredOnClickListener.onClick(View.java:4744)
01-30 09:10:26.258 23681 23681 W System.err: 	at android.view.View.performClick(View.java:5675)
```
##2.Log分析如下：

![Log 分析](http://upload-images.jianshu.io/upload_images/5851256-d2ffd488526452a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此异常经常发生在错误的解除绑定服务造成的，解决方法：
1.解除绑定服务之前，先判断是否绑定过，只有绑定过后才可以解绑
2.使用`try-catch` 抓取住异常
代码举例如下：

![Service not registered 异常举例 ](http://upload-images.jianshu.io/upload_images/5851256-a42907c26d48b3d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 九、BadTokenException 解决方案
##1. log 举例
```
03-12 14:55:13.734  5564  5564 E AndroidRuntime: FATAL EXCEPTION: main
03-12 14:55:13.734  5564  5564 E AndroidRuntime: Process: com.android.fmradio, PID: 5564
03-12 14:55:13.734  5564  5564 E AndroidRuntime: java.lang.RuntimeException: Error receiving broadcast Intent { act=android.intent.action.HEADSET_PLUG flg=0x40000010 (has extras) } in com.android.fmradio.FmService$FmServiceBroadcastReceiver@b3d2a03
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.app.LoadedApk$ReceiverDispatcher$Args.lambda$getRunnable$0(LoadedApk.java:1401)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.app.-$$Lambda$LoadedApk$ReceiverDispatcher$Args$_BumDX2UKsnxLVrE6UJsJZkotuA.run(Unknown Source:2)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.os.Handler.handleCallback(Handler.java:873)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.os.Handler.dispatchMessage(Handler.java:99)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.os.Looper.loop(Looper.java:193)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.app.ActivityThread.main(ActivityThread.java:6702)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at java.lang.reflect.Method.invoke(Native Method)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:493)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:911)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: Caused by: android.view.WindowManager$BadTokenException: Unable to add window android.view.ViewRootImpl$W@f652dba -- permission denied for window type 2003
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.view.ViewRootImpl.setView(ViewRootImpl.java:851)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.view.WindowManagerGlobal.addView(WindowManagerGlobal.java:356)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:93)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.app.Dialog.show(Dialog.java:329)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at com.android.fmradio.FmService$FmServiceBroadcastReceiver.onReceive(FmService.java:322)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	at android.app.LoadedApk$ReceiverDispatcher$Args.lambda$getRunnable$0(LoadedApk.java:1391)
03-12 14:55:13.734  5564  5564 E AndroidRuntime: 	... 8 more
```

##2.产生原因

Android 8.0 之后如果要弹出系统弹窗，需要使用 	`TYPE_APPLICATION_OVERLAY `以及
` <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />`来进行系统弹窗，否则会报以下异常` BadTokenException: Unable to add window android.view.ViewRootImpl$W@f652dba -- permission denied for window type 2003`

##3. 解决方案
 
系统弹窗，请用**TYPE_APPLICATION_OVERLAY** 替换之前的Windows Type。
```
Dialog mFMDialog = new AlertDialog.Builder(context)
                        .setTitle(R.string.airplane_title).setMessage(R.string.airplane_message)
                        .setPositiveButton(R.string.close_FM,
                            new DialogInterface.OnClickListener() {
                                @Override
                                public void onClick(DialogInterface dialog, int which) {
                        
                                }
                            }
                        ).setCancelable(false).create();
					// Android 8.0 之后弹出系统弹窗，需要使用 	TYPE_APPLICATION_OVERLAY 
					// <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
					// 一下两个 之前常用的系统的Dialog 会报 
					// BadTokenException: Unable to add window android.view.ViewRootImpl$W@f652dba -- permission denied for window type 2003
                    //mFMDialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_DIALOG);
					//mFMDialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);
                    mFMDialog.getWindow().setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY);
                    mFMDialog.show();
```

##4. 参考 Google Android GO 行为变更

Google 官方链接如下：
[Android 8.0 Alert 弹窗行为变更](https://developer.android.google.cn/about/versions/oreo/android-8.0-changes.html#cwt)
![Android 8.0 Alert 弹窗行为变更](https://upload-images.jianshu.io/upload_images/5851256-8e150e5395a22987.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 十、DeadObjectException 

## 1. 异常log举例

下面是 Documentsui 中出现DeadObjectException 问题的Log 举例
```
08-15 23:25:41.649 11079 11079 E AndroidRuntime: FATAL EXCEPTION: main
08-15 23:25:41.649 11079 11079 E AndroidRuntime: Process: com.android.documentsui, PID: 11079
08-15 23:25:41.649 11079 11079 E AndroidRuntime: java.lang.RuntimeException: android.os.DeadObjectException
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at android.database.BulkCursorToCursorAdaptor.getExtras(BulkCursorToCursorAdaptor.java:173)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at com.android.documentsui.RootCursorWrapper.getExtras(RootCursorWrapper.java:68)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at com.android.documentsui.SortingCursorWrapper.getExtras(SortingCursorWrapper.java:101)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at com.android.documentsui.DirectoryFragment$DocumentsAdapter.swapResult(DirectoryFragment.java:869)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at com.android.documentsui.DirectoryFragment$1.onLoadFinished(DirectoryFragment.java:321)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at com.android.documentsui.DirectoryFragment$1.onLoadFinished(DirectoryFragment.java:277)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at android.app.LoaderManagerImpl$LoaderInfo.callOnLoadFinished(LoaderManager.java:483)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at android.app.LoaderManagerImpl$LoaderInfo.onLoadComplete(LoaderManager.java:451)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at android.content.Loader.deliverResult(Loader.java:144)
08-15 23:25:41.649 11079 11079 E AndroidRuntime:     at com.android.documentsui.DirectoryLoader.deliverResult(DirectoryLoader.java:208)
```
## 2. DeadObjectException  Crash 原因
DeadObjectException 的问题的原因，通常是因为访问了被杀进程的资源。如果内存优化不能够解决或改善，可以考虑对异常进行捕获处理。但具体问题要具体分析，异常捕获后接下来如何进行后续流程是比较关键的。

## 3.DeadObjectException  解决方案
在DocumentsUi中发生的这个DeadObjectException，是一个子线程对directory进行load数据，子线程load完成将数据向主线程更新，因为主线程因为低内存被杀，所以导致发生exception。但是针对这个case，子线程的工作已经完成，而且主线程被杀也不再需要子线程再提供数据，所以直接将异常捕获（try--catch）然后退出，对于monkey不会产生额外影响。

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

