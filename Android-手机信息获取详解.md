##### 和你一起终身学习，这里是程序员android

`Android `手机是我们常用的工具之一，买手机之前，手机厂商会提供一些手机参数给我们，那么问题来了，我们该如何获取手机上的参数信息呢?
 
本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:

> 一、 获取手机基本信息(厂商、型号等参数)
> 二、设备信息获取实现图
> 三、 获取手机设备 宽、高、IMEI 信息
> 四、 获取手机厂商名、产品名、手机品牌、手机型号、主板名、设备名
> 五、获取手机硬件名、SDK版本、android版本 、语言支持、默认语言
> 六、 获取 SD 卡存储信息
> 七、 获取手机 RAM、ROM存储信息 
> 八、DeviceInfoUtils 封装类
> 九、SDCardUtils 封装类

下面将讲解以上信息的获取方法。




# 一、 获取手机基本信息(厂商、型号等参数)

以小米手机为例，手机常用的基本信息可以在`Settings `--> `About Phone `中看到，
例如下图：


![小米手机设备信息图](http://upload-images.jianshu.io/upload_images/5851256-9003f5c1c38241f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那么如何获取这些设备信息呢？ `Android`中 通常通过 `android.os.Build`类方法可以获取更多手机设备信息。

# 二、 设备信息获取实现图

![获取手机IMEI、宽、高、是否有SD卡，RAM、ROM、SD卡、是否联网、网络类型](http://upload-images.jianshu.io/upload_images/5851256-0d28bd7f1d81b060.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![默认语言，设备名，型号、厂商、Fingerprint、Android 版本、SDK版本、Google 安全patch、发布时间、版本类型、用户名 ](http://upload-images.jianshu.io/upload_images/5851256-ff421894ff874c8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![产品名、ID、产品名、主板名](http://upload-images.jianshu.io/upload_images/5851256-27b110154dcacc0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、 获取手机设备 宽、高、IMEI 信息方法

获取手机宽、高、`IMEI `信息方法如下：
```
	/**
	 * 获取设备宽度（px）
	 * 
	 */
	public static int getDeviceWidth(Context context) {
		return context.getResources().getDisplayMetrics().widthPixels;
	}

	/**
	 * 获取设备高度（px）
	 */
	public static int getDeviceHeight(Context context) {
		return context.getResources().getDisplayMetrics().heightPixels;
	}

	/**
	 * 获取设备的唯一标识， 需要 “android.permission.READ_Phone_STATE”权限
	 */
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

注意： 获取`IMEI` 需要获取手机状态权限

```
 <!-- 读取手机IMEI的设备权限 -->
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
```
如果是` Android 6.0` 之后的代码请使用动态申请权限的方法申请权限，否认会报安全异常的错误`SecurityException`，进而导致运行报错。

如需了解更多 系统安全权限的内容，请看 之前写的文章 [Android 系统权限使用详解](http://www.jianshu.com/p/6b8fc1fb13ef)

# 四、 获取手机厂商名、产品名、手机品牌、手机型号、主板名、设备名的方法

获取手机厂商名、产品名、手机品牌、手机型号、主板名、设备名的方法如下:
```
	/**
	 * 获取厂商名
	 * **/
	public static String getDeviceManufacturer() {
		return android.os.Build.MANUFACTURER;
	}

	/**
	 * 获取产品名
	 * **/
	public static String getDeviceProduct() {
		return android.os.Build.PRODUCT;
	}

	/**
	 * 获取手机品牌
	 */
	public static String getDeviceBrand() {
		return android.os.Build.BRAND;
	}

	/**
	 * 获取手机型号
	 */
	public static String getDeviceModel() {
		return android.os.Build.MODEL;
	}

	/**
	 * 获取手机主板名
	 */
	public static String getDeviceBoard() {
		return android.os.Build.BOARD;
	}

	/**
	 * 设备名
	 * **/
	public static String getDeviceDevice() {
		return android.os.Build.DEVICE;
	}

	/**
	 * 
	 * 
	 * fingerprit 信息
	 * **/
	public static String getDeviceFubgerprint() {
		return android.os.Build.FINGERPRINT;
	}
```
#五、  获取手机硬件名、SDK版本、android版本 、语言支持、默认语言等方法

获取手机硬件名、`SDK版本`、`android版本` 、语言支持、默认语言等方法如下:
```
	/**
	 * 硬件名
	 * 
	 * **/
	public static String getDeviceHardware() {
		return android.os.Build.HARDWARE;
	}

	/**
	 * 主机
	 * 
	 * **/
	public static String getDeviceHost() {
		return android.os.Build.HOST;
	}

	/**
	 * 
	 * 显示ID
	 * **/
	public static String getDeviceDisplay() {
		return android.os.Build.DISPLAY;
	}

	/**
	 * ID
	 * 
	 * **/
	public static String getDeviceId() {
		return android.os.Build.ID;
	}

	/**
	 * 获取手机用户名
	 * 
	 * **/
	public static String getDeviceUser() {
		return android.os.Build.USER;
	}

	/**
	 * 获取手机 硬件序列号
	 * **/
	public static String getDeviceSerial() {
		return android.os.Build.SERIAL;
	}

	/**
	 * 获取手机Android 系统SDK
	 * 
	 * @return
	 */
	public static int getDeviceSDK() {
		return android.os.Build.VERSION.SDK_INT;
	}

	/**
	 * 获取手机Android 版本
	 * 
	 * @return
	 */
	public static String getDeviceAndroidVersion() {
		return android.os.Build.VERSION.RELEASE;
	}

	/**
	 * 获取当前手机系统语言。
	 */
	public static String getDeviceDefaultLanguage() {
		return Locale.getDefault().getLanguage();
	}

	/**
	 * 获取当前系统上的语言列表(Locale列表)
	 */
	public static String getDeviceSupportLanguage() {
		Log.e("wangjie", "Local:" + Locale.GERMAN);
		Log.e("wangjie", "Local:" + Locale.ENGLISH);
		Log.e("wangjie", "Local:" + Locale.US);
		Log.e("wangjie", "Local:" + Locale.CHINESE);
		Log.e("wangjie", "Local:" + Locale.TAIWAN);
		Log.e("wangjie", "Local:" + Locale.FRANCE);
		Log.e("wangjie", "Local:" + Locale.FRENCH);
		Log.e("wangjie", "Local:" + Locale.GERMANY);
		Log.e("wangjie", "Local:" + Locale.ITALIAN);
		Log.e("wangjie", "Local:" + Locale.JAPAN);
		Log.e("wangjie", "Local:" + Locale.JAPANESE);
		return Locale.getAvailableLocales().toString();
	}

```

# 六、 获取 SD 卡存储信息

![SD卡信息](http://upload-images.jianshu.io/upload_images/5851256-ab608a9601cf8082.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.判断SD是否挂载方法

判断`SD`是否挂载方法如下:
```
	/**
	 * 判断SD是否挂载
	 */
	public static boolean isSDCardMount() {
		return Environment.getExternalStorageState().equals(
				Environment.MEDIA_MOUNTED);
	}
```
###2. 获取SD 存储信息的方法

获取`SD` 存储信息的方法如下:
```
/**
	 * 获取手机存储 ROM 信息
	 * 
	 * type： 用于区分内置存储于外置存储的方法
	 * 
	 * 内置SD卡 ：INTERNAL_STORAGE = 0;
	 * 
	 * 外置SD卡： EXTERNAL_STORAGE = 1;
	 * **/
	public static String getStorageInfo(Context context, int type) {

		String path = getStoragePath(context, type);
		/**
		 * 无外置SD 卡判断
		 * **/
		if (isSDCardMount() == false || TextUtils.isEmpty(path) || path == null) {
			return "无外置SD卡";
		}

		File file = new File(path);
		StatFs statFs = new StatFs(file.getPath());
		String stotageInfo;

		long blockCount = statFs.getBlockCountLong();
		long bloackSize = statFs.getBlockSizeLong();
		long totalSpace = bloackSize * blockCount;

		long availableBlocks = statFs.getAvailableBlocksLong();
		long availableSpace = availableBlocks * bloackSize;

		stotageInfo = "可用/总共："
				+ Formatter.formatFileSize(context, availableSpace) + "/"
				+ Formatter.formatFileSize(context, totalSpace);

		return stotageInfo;

	}
```

###3. 获取手机ROM （内置存储，外置存储）存储路径的方法

获取手机`ROM` 存储信息的方法如下：
```
/**
	 * 使用反射方法 获取手机存储路径
	 * 
	 * **/
	public static String getStoragePath(Context context, int type) {

		StorageManager sm = (StorageManager) context
				.getSystemService(Context.STORAGE_SERVICE);
		try {
			Method getPathsMethod = sm.getClass().getMethod("getVolumePaths",
					null);
			String[] path = (String[]) getPathsMethod.invoke(sm, null);

			switch (type) {
			case INTERNAL_STORAGE:
				return path[type];
			case EXTERNAL_STORAGE:
				if (path.length > 1) {
					return path[type];
				} else {
					return null;
				}

			default:
				break;
			}

		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}

	/**
	 * 获取 手机 RAM 信息 方法 一
	 * */
	public static String getTotalRAM(Context context) {
		long size = 0;
		ActivityManager activityManager = (ActivityManager) context
				.getSystemService(context.ACTIVITY_SERVICE);
		MemoryInfo outInfo = new MemoryInfo();
		activityManager.getMemoryInfo(outInfo);
		size = outInfo.totalMem;

		return Formatter.formatFileSize(context, size);
	}

	/**
	 * 手机 RAM 信息 方法 二
	 * */
	public static String getTotalRAMOther(Context context) {
		String path = "/proc/meminfo";
		String firstLine = null;
		int totalRam = 0;
		try {
			FileReader fileReader = new FileReader(path);
			BufferedReader br = new BufferedReader(fileReader, 8192);
			firstLine = br.readLine().split("\\s+")[1];
			br.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
		if (firstLine != null) {

			totalRam = (int) Math.ceil((new Float(Float.valueOf(firstLine)
					/ (1024 * 1024)).doubleValue()));

			long totalBytes = 0;

		}

		return Formatter.formatFileSize(context, totalRam);
	}

	/**
	 * 获取 手机 可用 RAM
	 * */
	public static String getAvailableRAM(Context context) {
		long size = 0;
		ActivityManager activityManager = (ActivityManager) context
				.getSystemService(context.ACTIVITY_SERVICE);
		MemoryInfo outInfo = new MemoryInfo();
		activityManager.getMemoryInfo(outInfo);
		size = outInfo.availMem;

		return Formatter.formatFileSize(context, size);
	}
```


# 七、获取手机 RAM、ROM存储信息 

### 1.RAM:
运行时内存，此大小直接决定手机运行的流畅度，相当于电脑内存。

###2.ROM :
手机存储（分内置`SD卡`，外置`SD卡`），此大小直接决定着手机可以存储资源的大小，相当于电脑硬盘。

以红米手机为例：
`RAM= 1904716KB= 1.82G`

![红米4 手机 RAM、ROM存储信息](http://upload-images.jianshu.io/upload_images/5851256-991d5182467c288f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![红米4 memory 信息 meminfo](http://upload-images.jianshu.io/upload_images/5851256-363c1371e566454e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###3.获取 `RAM `存储信息的方法如下：

```
	/**
	 * 获取 手机 RAM 信息
	 * */
	public static String getRAMInfo(Context context) {
		long totalSize = 0;
		long availableSize = 0;

		ActivityManager activityManager = (ActivityManager) context
				.getSystemService(context.ACTIVITY_SERVICE);

		MemoryInfo memoryInfo = new MemoryInfo();
		activityManager.getMemoryInfo(memoryInfo);
		totalSize = memoryInfo.totalMem;
		availableSize = memoryInfo.availMem;

		return "可用/总共：" + Formatter.formatFileSize(context, availableSize)
				+ "/" + Formatter.formatFileSize(context, totalSize);
	}
```

###4. 获取手机` ROM `存储信息的方法如下：

```
/**
	 * 获取手机存储 ROM 信息
	 * 
	 * type： 用于区分内置存储于外置存储的方法
	 * 
	 * 内置SD卡 ：INTERNAL_STORAGE = 0;
	 * 
	 * 外置SD卡： EXTERNAL_STORAGE = 1;
	 * **/
	public static String getStorageInfo(Context context, int type) {

		String path = getStoragePath(context, type);
		/**
		 * 无外置SD 卡判断
		 * **/
		if (isSDCardMount() == false || TextUtils.isEmpty(path) || path == null) {
			return "无外置SD卡";
		}

		File file = new File(path);
		StatFs statFs = new StatFs(file.getPath());
		String stotageInfo;

		long blockCount = statFs.getBlockCountLong();
		long bloackSize = statFs.getBlockSizeLong();
		long totalSpace = bloackSize * blockCount;

		long availableBlocks = statFs.getAvailableBlocksLong();
		long availableSpace = availableBlocks * bloackSize;

		stotageInfo = "可用/总共："
				+ Formatter.formatFileSize(context, availableSpace) + "/"
				+ Formatter.formatFileSize(context, totalSpace);

		return stotageInfo;

	}
```
# 八、DeviceInfoUtils 封装类
为了方便查询使用设备信息，小编已经封装成一个`Utils`类。代码如下：
```
package com.programandroid.Utils;

import java.util.Locale;

import android.R.string;
import android.content.Context;
import android.telephony.TelephonyManager;
import android.util.Log;

/*
 * DeviceInfoUtils.java
 *
 *  Created on: 2017-11-16
 *      Author: wangjie
 * 
 *  Welcome attention to weixin public number get more info
 *
 *  WeiXin Public Number : ProgramAndroid
 *  微信公众号 ：程序员Android
 *
 */
public class DeviceInfoUtils {

	/**
	 * 获取设备宽度（px）
	 * 
	 */
	public static int getDeviceWidth(Context context) {
		return context.getResources().getDisplayMetrics().widthPixels;
	}

	/**
	 * 获取设备高度（px）
	 */
	public static int getDeviceHeight(Context context) {
		return context.getResources().getDisplayMetrics().heightPixels;
	}

	/**
	 * 获取设备的唯一标识， 需要 “android.permission.READ_Phone_STATE”权限
	 */
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

	/**
	 * 获取厂商名
	 * **/
	public static String getDeviceManufacturer() {
		return android.os.Build.MANUFACTURER;
	}

	/**
	 * 获取产品名
	 * **/
	public static String getDeviceProduct() {
		return android.os.Build.PRODUCT;
	}

	/**
	 * 获取手机品牌
	 */
	public static String getDeviceBrand() {
		return android.os.Build.BRAND;
	}

	/**
	 * 获取手机型号
	 */
	public static String getDeviceModel() {
		return android.os.Build.MODEL;
	}

	/**
	 * 获取手机主板名
	 */
	public static String getDeviceBoard() {
		return android.os.Build.BOARD;
	}

	/**
	 * 设备名
	 * **/
	public static String getDeviceDevice() {
		return android.os.Build.DEVICE;
	}

	/**
	 * 
	 * 
	 * fingerprit 信息
	 * **/
	public static String getDeviceFubgerprint() {
		return android.os.Build.FINGERPRINT;
	}

	/**
	 * 硬件名
	 * 
	 * **/
	public static String getDeviceHardware() {
		return android.os.Build.HARDWARE;
	}

	/**
	 * 主机
	 * 
	 * **/
	public static String getDeviceHost() {
		return android.os.Build.HOST;
	}

	/**
	 * 
	 * 显示ID
	 * **/
	public static String getDeviceDisplay() {
		return android.os.Build.DISPLAY;
	}

	/**
	 * ID
	 * 
	 * **/
	public static String getDeviceId() {
		return android.os.Build.ID;
	}

	/**
	 * 获取手机用户名
	 * 
	 * **/
	public static String getDeviceUser() {
		return android.os.Build.USER;
	}

	/**
	 * 获取手机 硬件序列号
	 * **/
	public static String getDeviceSerial() {
		return android.os.Build.SERIAL;
	}

	/**
	 * 获取手机Android 系统SDK
	 * 
	 * @return
	 */
	public static int getDeviceSDK() {
		return android.os.Build.VERSION.SDK_INT;
	}

	/**
	 * 获取手机Android 版本
	 * 
	 * @return
	 */
	public static String getDeviceAndroidVersion() {
		return android.os.Build.VERSION.RELEASE;
	}

	/**
	 * 获取当前手机系统语言。
	 */
	public static String getDeviceDefaultLanguage() {
		return Locale.getDefault().getLanguage();
	}

	/**
	 * 获取当前系统上的语言列表(Locale列表)
	 */
	public static String getDeviceSupportLanguage() {
		Log.e("wangjie", "Local:" + Locale.GERMAN);
		Log.e("wangjie", "Local:" + Locale.ENGLISH);
		Log.e("wangjie", "Local:" + Locale.US);
		Log.e("wangjie", "Local:" + Locale.CHINESE);
		Log.e("wangjie", "Local:" + Locale.TAIWAN);
		Log.e("wangjie", "Local:" + Locale.FRANCE);
		Log.e("wangjie", "Local:" + Locale.FRENCH);
		Log.e("wangjie", "Local:" + Locale.GERMANY);
		Log.e("wangjie", "Local:" + Locale.ITALIAN);
		Log.e("wangjie", "Local:" + Locale.JAPAN);
		Log.e("wangjie", "Local:" + Locale.JAPANESE);
		return Locale.getAvailableLocales().toString();
	}

	public static String getDeviceAllInfo(Context context) {

		return "\n\n1. IMEI:\n\t\t" + getIMEI(context)

		+ "\n\n2. 设备宽度:\n\t\t" + getDeviceWidth(context)

		+ "\n\n3. 设备高度:\n\t\t" + getDeviceHeight(context)

		+ "\n\n4. 是否有内置SD卡:\n\t\t" + SDCardUtils.isSDCardMount()

		+ "\n\n5. RAM 信息:\n\t\t" + SDCardUtils.getRAMInfo(context)

		+ "\n\n6. 内部存储信息\n\t\t" + SDCardUtils.getStorageInfo(context, 0)

		+ "\n\n7. SD卡 信息:\n\t\t" + SDCardUtils.getStorageInfo(context, 1)

		+ "\n\n8. 是否联网:\n\t\t" + Utils.isNetworkConnected(context)

		+ "\n\n9. 网络类型:\n\t\t" + Utils.GetNetworkType(context)

		+ "\n\n10. 系统默认语言:\n\t\t" + getDeviceDefaultLanguage()

		+ "\n\n11. 硬件序列号(设备名):\n\t\t" + android.os.Build.SERIAL

		+ "\n\n12. 手机型号:\n\t\t" + android.os.Build.MODEL

		+ "\n\n13. 生产厂商:\n\t\t" + android.os.Build.MANUFACTURER

		+ "\n\n14. 手机Fingerprint标识:\n\t\t" + android.os.Build.FINGERPRINT

		+ "\n\n15. Android 版本:\n\t\t" + android.os.Build.VERSION.RELEASE

		+ "\n\n16. Android SDK版本:\n\t\t" + android.os.Build.VERSION.SDK_INT

		+ "\n\n17. 安全patch 时间:\n\t\t" + android.os.Build.VERSION.SECURITY_PATCH

		+ "\n\n18. 发布时间:\n\t\t" + Utils.Utc2Local(android.os.Build.TIME)

		+ "\n\n19. 版本类型:\n\t\t" + android.os.Build.TYPE

		+ "\n\n20. 用户名:\n\t\t" + android.os.Build.USER

		+ "\n\n21. 产品名:\n\t\t" + android.os.Build.PRODUCT

		+ "\n\n22. ID:\n\t\t" + android.os.Build.ID

		+ "\n\n23. 显示ID:\n\t\t" + android.os.Build.DISPLAY

		+ "\n\n24. 硬件名:\n\t\t" + android.os.Build.HARDWARE

		+ "\n\n25. 产品名:\n\t\t" + android.os.Build.DEVICE

		+ "\n\n26. Bootloader:\n\t\t" + android.os.Build.BOOTLOADER

		+ "\n\n27. 主板名:\n\t\t" + android.os.Build.BOARD

		+ "\n\n28. CodeName:\n\t\t" + android.os.Build.VERSION.CODENAME
				+ "\n\n29. 语言支持:\n\t\t" + getDeviceSupportLanguage();

	}
}

```
# 九、SDCardUtils 封装类
为了方便查询使用设备信息，小编已经封装成一个`Utils`类。代码如下：
```
package com.programandroid.Utils;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.lang.reflect.Method;

import android.app.ActivityManager;
import android.app.ActivityManager.MemoryInfo;
import android.content.Context;
import android.os.Build;
import android.os.Environment;
import android.os.StatFs;
import android.os.storage.StorageManager;
import android.text.TextUtils;
import android.text.format.Formatter;

/*
 * SDCardUtils.java
 *
 *  Created on: 2017-11-22
 *      Author: wangjie
 * 
 *  Welcome attention to weixin public number get more info
 *
 *  WeiXin Public Number : ProgramAndroid
 *  微信公众号 ：程序员Android
 *
 */
public class SDCardUtils {

	private static final int INTERNAL_STORAGE = 0;
	private static final int EXTERNAL_STORAGE = 1;

	/**
	 * 获取 手机 RAM 信息
	 * */
	public static String getRAMInfo(Context context) {
		long totalSize = 0;
		long availableSize = 0;

		ActivityManager activityManager = (ActivityManager) context
				.getSystemService(context.ACTIVITY_SERVICE);

		MemoryInfo memoryInfo = new MemoryInfo();
		activityManager.getMemoryInfo(memoryInfo);
		totalSize = memoryInfo.totalMem;
		availableSize = memoryInfo.availMem;

		return "可用/总共：" + Formatter.formatFileSize(context, availableSize)
				+ "/" + Formatter.formatFileSize(context, totalSize);
	}

	/**
	 * 判断SD是否挂载
	 */
	public static boolean isSDCardMount() {
		return Environment.getExternalStorageState().equals(
				Environment.MEDIA_MOUNTED);
	}

	/**
	 * 获取手机存储 ROM 信息
	 * 
	 * type： 用于区分内置存储于外置存储的方法
	 * 
	 * 内置SD卡 ：INTERNAL_STORAGE = 0;
	 * 
	 * 外置SD卡： EXTERNAL_STORAGE = 1;
	 * **/
	public static String getStorageInfo(Context context, int type) {

		String path = getStoragePath(context, type);
		/**
		 * 无外置SD 卡判断
		 * **/
		if (isSDCardMount() == false || TextUtils.isEmpty(path) || path == null) {
			return "无外置SD卡";
		}

		File file = new File(path);
		StatFs statFs = new StatFs(file.getPath());
		String stotageInfo;

		long blockCount = statFs.getBlockCountLong();
		long bloackSize = statFs.getBlockSizeLong();
		long totalSpace = bloackSize * blockCount;

		long availableBlocks = statFs.getAvailableBlocksLong();
		long availableSpace = availableBlocks * bloackSize;

		stotageInfo = "可用/总共："
				+ Formatter.formatFileSize(context, availableSpace) + "/"
				+ Formatter.formatFileSize(context, totalSpace);

		return stotageInfo;

	}

	/**
	 * 使用反射方法 获取手机存储路径
	 * 
	 * **/
	public static String getStoragePath(Context context, int type) {

		StorageManager sm = (StorageManager) context
				.getSystemService(Context.STORAGE_SERVICE);
		try {
			Method getPathsMethod = sm.getClass().getMethod("getVolumePaths",
					null);
			String[] path = (String[]) getPathsMethod.invoke(sm, null);

			switch (type) {
			case INTERNAL_STORAGE:
				return path[type];
			case EXTERNAL_STORAGE:
				if (path.length > 1) {
					return path[type];
				} else {
					return null;
				}

			default:
				break;
			}

		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}

	/**
	 * 获取 手机 RAM 信息 方法 一
	 * */
	public static String getTotalRAM(Context context) {
		long size = 0;
		ActivityManager activityManager = (ActivityManager) context
				.getSystemService(context.ACTIVITY_SERVICE);
		MemoryInfo outInfo = new MemoryInfo();
		activityManager.getMemoryInfo(outInfo);
		size = outInfo.totalMem;

		return Formatter.formatFileSize(context, size);
	}

	/**
	 * 手机 RAM 信息 方法 二
	 * */
	public static String getTotalRAMOther(Context context) {
		String path = "/proc/meminfo";
		String firstLine = null;
		int totalRam = 0;
		try {
			FileReader fileReader = new FileReader(path);
			BufferedReader br = new BufferedReader(fileReader, 8192);
			firstLine = br.readLine().split("\\s+")[1];
			br.close();
		} catch (Exception e) {
			e.printStackTrace();
		}
		if (firstLine != null) {

			totalRam = (int) Math.ceil((new Float(Float.valueOf(firstLine)
					/ (1024 * 1024)).doubleValue()));

			long totalBytes = 0;

		}

		return Formatter.formatFileSize(context, totalRam);
	}

	/**
	 * 获取 手机 可用 RAM
	 * */
	public static String getAvailableRAM(Context context) {
		long size = 0;
		ActivityManager activityManager = (ActivityManager) context
				.getSystemService(context.ACTIVITY_SERVICE);
		MemoryInfo outInfo = new MemoryInfo();
		activityManager.getMemoryInfo(outInfo);
		size = outInfo.availMem;

		return Formatter.formatFileSize(context, size);
	}

	/**
	 * 获取手机内部存储空间
	 * 
	 * @param context
	 * @return 以M,G为单位的容量
	 */
	public static String getTotalInternalMemorySize(Context context) {
		File file = Environment.getDataDirectory();
		StatFs statFs = new StatFs(file.getPath());
		long blockSizeLong = statFs.getBlockSizeLong();
		long blockCountLong = statFs.getBlockCountLong();
		long size = blockCountLong * blockSizeLong;
		return Formatter.formatFileSize(context, size);
	}

	/**
	 * 获取手机内部可用存储空间
	 * 
	 * @param context
	 * @return 以M,G为单位的容量
	 */
	public static String getAvailableInternalMemorySize(Context context) {
		File file = Environment.getDataDirectory();
		StatFs statFs = new StatFs(file.getPath());
		long availableBlocksLong = statFs.getAvailableBlocksLong();
		long blockSizeLong = statFs.getBlockSizeLong();
		return Formatter.formatFileSize(context, availableBlocksLong
				* blockSizeLong);
	}

	/**
	 * 获取手机外部存储空间
	 * 
	 * @param context
	 * @return 以M,G为单位的容量
	 */
	public static String getTotalExternalMemorySize(Context context) {
		File file = Environment.getExternalStorageDirectory();
		StatFs statFs = new StatFs(file.getPath());
		long blockSizeLong = statFs.getBlockSizeLong();
		long blockCountLong = statFs.getBlockCountLong();
		return Formatter
				.formatFileSize(context, blockCountLong * blockSizeLong);
	}

	/**
	 * 获取手机外部可用存储空间
	 * 
	 * @param context
	 * @return 以M,G为单位的容量
	 */
	public static String getAvailableExternalMemorySize(Context context) {
		File file = Environment.getExternalStorageDirectory();
		StatFs statFs = new StatFs(file.getPath());
		long availableBlocksLong = statFs.getAvailableBlocksLong();
		long blockSizeLong = statFs.getBlockSizeLong();
		return Formatter.formatFileSize(context, availableBlocksLong
				* blockSizeLong);
	}

	/**
	 * 
	 * SD 卡信息
	 * */

	public static String getSDCardInfo() {

		SDCardInfo sd = new SDCardInfo();
		if (!isSDCardMount())
			return "SD card 未挂载!";

		sd.isExist = true;
		StatFs sf = new StatFs(Environment.getExternalStorageDirectory()
				.getPath());

		sd.totalBlocks = sf.getBlockCountLong();
		sd.blockByteSize = sf.getBlockSizeLong();
		sd.availableBlocks = sf.getAvailableBlocksLong();
		sd.availableBytes = sf.getAvailableBytes();
		sd.freeBlocks = sf.getFreeBlocksLong();
		sd.freeBytes = sf.getFreeBytes();
		sd.totalBytes = sf.getTotalBytes();
		return sd.toString();
	}

	public static class SDCardInfo {
		boolean isExist;
		long totalBlocks;
		long freeBlocks;
		long availableBlocks;
		long blockByteSize;
		long totalBytes;
		long freeBytes;
		long availableBytes;

		@Override
		public String toString() {
			return "isExist=" + isExist + "\ntotalBlocks=" + totalBlocks
					+ "\nfreeBlocks=" + freeBlocks + "\navailableBlocks="
					+ availableBlocks + "\nblockByteSize=" + blockByteSize
					+ "\ntotalBytes=" + totalBytes + "\nfreeBytes=" + freeBytes
					+ "\navailableBytes=" + availableBytes;
		}
	}

	// add start by wangjie for SDCard TotalStorage
	public static String getSDCardTotalStorage(long totalByte) {

		double byte2GB = totalByte / 1024.00 / 1024.00 / 1024.00;
		double totalStorage;
		if (byte2GB > 1) {
			totalStorage = Math.ceil(byte2GB);
			if (totalStorage > 1 && totalStorage < 3) {
				return 2.0 + "GB";
			} else if (totalStorage > 2 && totalStorage < 5) {
				return 4.0 + "GB";
			} else if (totalStorage >= 5 && totalStorage < 10) {
				return 8.0 + "GB";
			} else if (totalStorage >= 10 && totalStorage < 18) {
				return 16.0 + "GB";
			} else if (totalStorage >= 18 && totalStorage < 34) {
				return 32.0 + "GB";
			} else if (totalStorage >= 34 && totalStorage < 50) {
				return 48.0 + "GB";
			} else if (totalStorage >= 50 && totalStorage < 66) {
				return 64.0 + "GB";
			} else if (totalStorage >= 66 && totalStorage < 130) {
				return 128.0 + "GB";
			}
		} else {
			// below 1G return get values
			totalStorage = totalByte / 1024.00 / 1024.00;

			if (totalStorage >= 515 && totalStorage < 1024) {
				return 1 + "GB";
			} else if (totalStorage >= 260 && totalStorage < 515) {
				return 512 + "MB";
			} else if (totalStorage >= 130 && totalStorage < 260) {
				return 256 + "MB";
			} else if (totalStorage > 70 && totalStorage < 130) {
				return 128 + "MB";
			} else if (totalStorage > 50 && totalStorage < 70) {
				return 64 + "MB";
			}
		}

		return totalStorage + "GB";
	}
	// add end by wangjie for SDCard TotalStorage

}

```

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
