 

##### 和您一起终身学习，这里是程序员Android

本文主要是总结Intent 常用的方法，并封装成Utils类中
主要涉及以下内容
> 一、通过组件名启动 
> 二、通过包名、类名启动 
> 三、通过类启动
> 四、打电话 
> 五、发短信 
> 六、打开网页
> 七、播放音乐
> 八、打开图片
> 九、创建闹钟
>十、创建定时器
>十一、添加日历事件
> 十二、拍照
>十三、打开Camera
> 十四、打开视频录像
>十五、选择联系人
>十六、查看联系人
>十七、编辑联系人
>十八、插入联系人
>十九、写邮件
>二十、打开地图指定点
>二十一、检索特定类型图片

Intent 简介请看上篇文章
[Intent 使用方法详解](
http://www.jianshu.com/p/81e4951d8054)


# 一、通过组件名启动 Activity

## 1. 使用方法

```
	/**
	 * 通过组件名启动Activity
	 * **/
	public static void StartIntentFromComponent(Context context,
			Class intentClass) {
		Intent intent = new Intent();
		// 1.使用ComponentName 启动Activity
		ComponentName componentname = new ComponentName(context, intentClass);
		intent.setComponent(componentname);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}

```

# 二、通过包名、类名启动 Activity

## 1.使用方法

```
	/**
	 * 通过包名类名启动Activity
	 * **/
	public static void StartIntentFromPackage(Context context,
			String packageName, String className) {
		Intent intent = new Intent();
		// 1.使用ComponentName 启动Activity
		ComponentName componentname = new ComponentName(packageName, className);
		intent.setComponent(componentname);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```
# 三、 通过类启动 Activity

## 1. 使用方法

```
	/**
	 * 通过Class启动Activity
	 * **/
	public static void StartIntentFromClass(Context context, Class<?> classOpen) {
		Intent intent = new Intent();
		// 2.使用Setclass方法，类方法间接使用ComponentName
		intent.setClass(context, classOpen);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```
# 四、 打电话 

## 1. 使用Intent 打电话 方法如下

```
	/**
	 * 打电话
	 * **/
	public static void MakeCall(Context context, int number) {

		// 需要打电话权限
		// <uses-permission android:name="android.permission.CALL_PHONE"/>

		Intent intent = new Intent(Intent.ACTION_CALL, Uri.parse("tel:"
				+ number));
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}

	}

```
## 2.注意：打电话需要申请权限

```
<uses-permission android:name="android.permission.CALL_PHONE"/>
```
#五、 发短信 


## 1.基础发送短信
```
	/**
	 * 1.基础发送短信
	 * **/
	public static void SendMms(Context context, String mmsString) {

		Intent sendIntent = new Intent();
		sendIntent.setAction(Intent.ACTION_SEND);
		sendIntent.putExtra(Intent.EXTRA_TEXT, mmsString);
		sendIntent.setType("text/plain");
		// sendIntent.setData(Uri.parse("smsto:"));
		// This ensures only SMS apps respond
		// 修改 Intnent 选择器Tittle
		String title = context.getResources().getString(R.string.hello_world);
		Intent chooser = Intent.createChooser(sendIntent, title);

		// 验证是否有Activity 接收
		if (sendIntent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(chooser);
		}
	}
```
## 2.自定义 发送短信
```
	/**
	 * 2.自定义 发送短信
	 * **/
	public static void SendMmsCustom(Context context, String mmsString) {

		Intent sendIntent = new Intent();
		sendIntent.setAction(Intent.ACTION_SEND);
		sendIntent.putExtra(Intent.EXTRA_TEXT, mmsString);
		sendIntent.setType("text/plain");
		// sendIntent.setData(Uri.parse("smsto:"));
		// This ensures only SMS apps respond
		// 修改 Intnent 选择器Tittle

		String title = context.getResources().getString(R.string.hello_world);

		Intent chooser = Intent.createChooser(sendIntent, title);

		// 验证是否有Activity 接收
		if (sendIntent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(chooser);
		}
	}
```
# 六、打开网页

## 1. 使用方法

```
	/**
	 * 打开网页
	 * **/
	public static void OpenInternetUri(Context context, String uri) {

		Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(uri));
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}

	}

```
# 七、播放音乐

## 1. 使用方法

```
	/**
	 * 播放音乐
	 * **/
	public static void PlayMusic(Context context, String path) {

		// String
		// path=Environment.getExternalStorageDirectory().getAbsolutePath()+"test.mp3";
		Intent intent = new Intent(Intent.ACTION_VIEW);
		intent.setDataAndType(Uri.parse("file:///" + path), "audio/*");
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}

	}
```
## 2.播放特定艺术家专辑

```
	/**
	 * 搜索特定艺术家专辑
	 * **/
	public static void playSearchArtist(Context context, String artist) {

		Intent intent = new Intent(
				MediaStore.INTENT_ACTION_MEDIA_PLAY_FROM_SEARCH);
		intent.putExtra(MediaStore.EXTRA_MEDIA_FOCUS,
				MediaStore.Audio.Artists.ENTRY_CONTENT_TYPE);
		intent.putExtra(MediaStore.EXTRA_MEDIA_ARTIST, artist);
		intent.putExtra(SearchManager.QUERY, artist);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}

	}
```

# 八、 打开图片

## 1. 使用方法
```
	/**
	 * 打开图片
	 * **/
	public static void OpenImage(Context context, File file) {
		// File file =new File("/mnt/sdcard/1.png");
		Intent intent = new Intent(Intent.ACTION_VIEW);
		intent.setDataAndType(Uri.fromFile(file), "image/*");

		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}

	}
```

#九、 创建闹钟

## 1. 使用方法

```
	/**
	 * 创建闹钟
	 * **/

	public static void SetAlarmIntent(Context context, String message,
			int hour, int minutes) {
		Intent intent = new Intent(AlarmClock.ACTION_SET_ALARM)
				.putExtra(AlarmClock.EXTRA_MESSAGE, message)
				.putExtra(AlarmClock.EXTRA_HOUR, hour)
				.putExtra(AlarmClock.EXTRA_MINUTES, minutes);

		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		
```

## 2.设置闹钟action 机权限

```
  <!-- 设置闹钟的权限 -->
    <uses-permission android:name="com.android.alarm.permission.SET_ALARM" />
        <activity android:name=".Intent.IntentMethod" >
            <intent-filter>
                <action android:name="android.intent.action.SET_ALARM" />

                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
```

## 3.显示所有闹钟

![显示所有闹钟](http://upload-images.jianshu.io/upload_images/5851256-7c01f6e55f0f90e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#十、创建定时器

## 1. 使用方法

```
	/**
	 * 创建定时器
	 * **/
	public static void StartTimer(Context context, String message, int seconds) {
		Intent intent = new Intent(AlarmClock.ACTION_SET_TIMER)
				.putExtra(AlarmClock.EXTRA_MESSAGE, message)
				.putExtra(AlarmClock.EXTRA_LENGTH, seconds)
				.putExtra(AlarmClock.EXTRA_SKIP_UI, true);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```
添加设置SET_TIMER的Action
```
        <activity android:name=".Intent.IntentMethod" >
            <intent-filter>
                <action android:name="android.intent.action.SET_ALARM" />
                <action android:name="android.intent.action.SET_TIMER" />

                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
```
#十一、 添加日历事件

## 1. 使用方法

```
	/**
	 * 添加日历事件
	 * **/

	public static void AddCalendarEvent(Context context, String title,
			String location, Calendar begin, Calendar end) {
		Intent intent = new Intent(Intent.ACTION_INSERT)
				.setData(Events.CONTENT_URI).putExtra(Events.TITLE, title)
				.putExtra(Events.EVENT_LOCATION, location)
				.putExtra(CalendarContract.EXTRA_EVENT_BEGIN_TIME, begin)
				.putExtra(CalendarContract.EXTRA_EVENT_END_TIME, end);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}

```

## 2. 日历事件过滤

![过滤日历action](http://upload-images.jianshu.io/upload_images/5851256-cde0b000232636c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 十二、 拍照

## 1.使用方法

```
	/**
	 * 拍照
	 * **/

	public static void CapturePhoto(Context context, String targetFilename,
			Uri mLocationForPhotos) {
		Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
		intent.putExtra(MediaStore.EXTRA_OUTPUT,
				Uri.withAppendedPath(mLocationForPhotos, targetFilename));
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}

	}
```

## 2. 拍照过滤

![拍照过滤Action ](http://upload-images.jianshu.io/upload_images/5851256-59719f1425dad645.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#十三、打开Camera

## 1. 使用方法

```

	/**
	 * 打开Camera
	 * **/

	public static void OpenCamera(Context context) {
		Intent intent = new Intent(MediaStore.INTENT_ACTION_STILL_IMAGE_CAMERA);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}

```

## 2.打开Camera 过滤

![Camera 过滤 Action方法](http://upload-images.jianshu.io/upload_images/5851256-87e37588dc1ebcbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 十四、打开视频录像

## 1.使用方法

```

	/**
	 * 打开录像视频
	 * **/

	public static void OpenCameraVideo(Context context) {
		Intent intent = new Intent(MediaStore.INTENT_ACTION_VIDEO_CAMERA);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```

## 2. 打开录像功能过滤

![过滤录像功能方法](http://upload-images.jianshu.io/upload_images/5851256-f99c367c24835bd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#十五、 选择联系人

## 1. 使用方法

```
	/***
	 * 选择联系人
	 * **/
	public static void SelectContact(Context context) {
		Intent intent = new Intent(Intent.ACTION_PICK);
		intent.setType(ContactsContract.Contacts.CONTENT_TYPE);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```

#十六、 查看联系人

## 1.使用方法

```
	/***
	 * 查看联系人
	 * **/
	public static void ViewContact(Context context, Uri contactUri) {
		Intent intent = new Intent(Intent.ACTION_VIEW, contactUri);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```

#十七、 编辑联系人

##1.  使用方法

```
	/***
	 * 编辑联系人
	 * **/
	public static void EditContact(Context context, Uri contactUri, String email) {
		Intent intent = new Intent(Intent.ACTION_EDIT);
		intent.setData(contactUri);
		intent.putExtra(Intents.Insert.EMAIL, email);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```

#十八、插入联系人

## 1.使用方法

```
	/***
	 * 插入联系人
	 * **/
	public static void InsertContact(Context context, String name, String email) {
		Intent intent = new Intent(Intent.ACTION_INSERT);
		intent.setType(Contacts.CONTENT_TYPE);
		intent.putExtra(Intents.Insert.NAME, name);
		intent.putExtra(Intents.Insert.EMAIL, email);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```

#十九、写邮件

## 1. 使用方法

```

	/***
	 * 写邮件
	 * **/
	public static void composeEmail(Context context, String[] addresses,
			String subject, Uri attachment) {
		Intent intent = new Intent(Intent.ACTION_SEND);
		intent.setType("*/*");
		// intent.setData(Uri.parse("mailto:"));
		// only email apps should handle this
		intent.putExtra(Intent.EXTRA_EMAIL, addresses);
		intent.putExtra(Intent.EXTRA_SUBJECT, subject);
		intent.putExtra(Intent.EXTRA_STREAM, attachment);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}

```

## 2. 邮件过滤

![邮件过滤 方法](http://upload-images.jianshu.io/upload_images/5851256-3e499ba161f8a160.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#二十、 打开地图指定点

## 1. 使用方法

```
	/***
	 * 打开地图指定点
	 * **/
	public static void callCar(Context context, Uri geoLocation) {
		Intent intent = new Intent(Intent.ACTION_VIEW);
		intent.setData(geoLocation);
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}
```
#二十一、检索特定类型图片

## 1.使用方法
```

	/***
	 * 检索特定类型图片 获取照片
	 * **/
	public static void selectImage(Context context) {
		Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
		intent.setType("image/*");
		if (intent.resolveActivity(context.getPackageManager()) != null) {
			context.startActivity(intent);
		}
	}

```




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
