##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:

> 一、Intent 简介
> 二、Intent 主要用途
> 三、Intent 分类
> 四、隐式Intent 接收过滤标签
> 五、PendingIntent 介绍
> 六、Intent的七大属性
> 七、使用ADB调试 Intent



# 一、Intent 简介

`Intent` 是一个消息传递对象，主要用于组件之间的通讯，例如：启动`Activity`、启动`Service`、传递`Broadcast`等。

Intent 主要功能流程图如下:

![Intent 主要功能流程图](http://upload-images.jianshu.io/upload_images/5851256-a06bf1d5a20f5405.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 二、 Intent 主要用途

## 1. 启动Activity

-  `startActivity()`
无返回值，直接启动`Activity`

-  `startActivityForResult() `
   有返回值，返回值在onActivityResult() 回调

## 2. 启动Service

- `startService()`
一次性操作

- `bindService()`
绑定组件，随组件生命周期结束而结束

## 3. 发送Broadcast 

- `sendBroadcast()`
普通无序广播

- `sendOrderedBroadcast() `
有序广播

- `sendStickyBroadcast()`
持续黏性广播


# 三、 Intent 分类

##  1.显示 Intent

按名称（完全限定类名）指定要启动的组件。
例如:

```
					Intent intentActivity = new Intent(MainActivity.this,
							ActivityMethods.class);
					startActivity(intentActivity);
```

##  2.隐式 Intent

不会指定特定的组件，而是声明要执行的常规操作，从而允许其他应用中的组件来处理它
例如：
```
	/**
	 * 发送短信
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


# 四、 隐式Intent 接收过滤标签

应用可以接收哪些隐式` Intent`，请在清单文件中使用 `<intent-filter> `元素为每个应用组件声明一个或多个 `Intent `过滤器。每个` Intent` 过滤器均根据 `Intent `的操作、数据和类别指定自身接受的` Intent `类型。 仅当隐式` Intent` 可以通过` Intent`过滤器之一传递时，系统才会将该 `Intent `传递给应用组件。


##1. <action>

在 `name `属性中，声明接受的 `Intent `操作。该值必须是操作的文本字符串值，而不是类常量。

例如：
`java` 代码中启动的`Intent`的`Action `

```
Intent sendIntent = new Intent("String_action");
```

Androidmanfest.xml 中过滤标签如下：

![Androidmanfest 标签声明 ](http://upload-images.jianshu.io/upload_images/5851256-dbdc127fe8ab9a81.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##  2.<data>

使用一个或多个指定数据 `URI` 各个方面（`scheme、host、port、path `等）和 `MIME` 类型的属性，声明接受的数据类型。

## 3.<category>

在 `name` 属性中，声明接受的` Intent` 类别。该值必须是操作的文本字符串值，而不是类常量。

例如：

![category 属性使用](http://upload-images.jianshu.io/upload_images/5851256-b3062c782c838430.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 4. 禁止其他应用通过Intent 掉起自己组件 

`android:exported="false"`

## 5. 应用主要入口点Action
` <action android:name="android.intent.action.MAIN" />`

## 6.  Launcher 图标入口Action
以下两个元素必须配对使用，`Activity `才会显示在应用启动器中。

![Launcher  标签入口 ](http://upload-images.jianshu.io/upload_images/5851256-b2d26eb1c1e3a1e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 7.注意：
` CATEGORY_LAUNCHER` 类别指示此 `Activity `的图标应放入系统的应用启动器。 如果` <activity> `元素未使用 `icon `指定图标，则系统将使用` <application> `元素中的图标

# 五、PendingIntent 介绍

`PendingIntent `对象是` Intent `对象的包装器。`PendingIntent` 的主要目的是授权外部应用使用包含的 `Intent`，就像是它从您应用本身的进程中执行的一样。

主要应用于以下场景
- 1.通知 `  NotificationManager`
- 2.应用小部件 `AppWidget`
- 3.定时任务 ` AlarmManager`

##1. PendingIntent 使用注意事项：

- 1.PendingIntent.getActivity()
适用于启动 `Activity `的 `Intent`。
- 2.PendingIntent.getService()
 适用于启动` Service `的 `Intent`。
- 3.PendingIntent.getBroadcast()
适用于启动 `BroadcastReceiver` 的` Intent`。

# 六、Intent的七大属性

##  1 . Component Name(目标组件的全类、组件名称)   

`setComponent(),   `
`getComponent(),  `
`setClass() , `
`setClassName()`

## 2 . Action   (intent 将执行的动作) 

`setAction() `
`getAction()
`
## 3 . Data (用于向Action 属性提供操作数据)  

`URI`对象`scheme://host:port/path ` (协议头，主机，端口，路径)

## 4 . Type 分类

指定`Data`所指定的`Uri`对应的`MIME`类型，不指定会根据数据自动推导

##  5 . Category  类别

为`Action` 提供额外的附件类别信息，可以有多个`Category`，但必须有一个`default`。
```
   <!-- 默认分类必须加上，否则会报错 -->
<category android:name="android.intent.category.DEFAULT"/>
```

## 6 .  Extra 数据载体

通过键值对进行数据存储，用于多个`Action`之间提供数据交换.
 
##  7 . Flags 标记

标记组件如何启动，以及启动后如何对待` FALG_ACTIVITY_SINGLE_TOP` 
`FALG_ACTIVITY_CLEAR_TOP`等等)

# 七、  使用ADB调试 Intent

## 1. 语法

```
adb shell am start -a <ACTION> -t <MIME_TYPE> -d <DATA> \
  -e <EXTRA_NAME> <EXTRA_VALUE> -n <ACTIVITY>
```

## 2.举例
```
adb shell am start -a android.intent.action.DIAL \
  -d tel:555-5555 -n org.example.MyApp/.MyActivity
```

 
 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
