 
##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、获取系统Settings 中的亮度
>二、修改APP界面屏幕亮度，不会影响其他APP
>三、修改系统Settings 中屏幕亮度，影响所有APP
>四、完整代码实现

 
#一、获取系统Settings 中的亮度

系统屏幕亮度值在`（0~255）`之间，获取方法很简单，只需要调用以下方法即可。

```
/**
	 * 1.获取系统默认屏幕亮度值 屏幕亮度值范围（0-255）
	 * **/
	private int getScreenBrightness(Context context) {
		ContentResolver contentResolver = context.getContentResolver();
		int defVal = 125;
		return Settings.System.getInt(contentResolver,
				Settings.System.SCREEN_BRIGHTNESS, defVal);
	}
```

**修改屏幕亮度包含两种：**
1.修改APP界面屏幕亮度，不会影响其他`APP`。
2.修改系统 `Settings` 中屏幕亮度，影响所有`APP`.

#二，修改APP界面屏幕亮度，不会影响其他APP
修改自身 `APP` 亮度很简单，只需要在`Activity OnCreate `方法调用如下代码即可。
```
	/**
	 * 2.设置 APP界面屏幕亮度值方法
	 * **/
	private void setAppScreenBrightness(int birghtessValue) {
		Window window = getWindow();
		WindowManager.LayoutParams lp = window.getAttributes();
		lp.screenBrightness = birghtessValue / 255.0f;
		window.setAttributes(lp);
	}
```
#三、修改系统Settings 中屏幕亮度，影响所有APP

修改系统 `Settings` 中的屏幕亮度，由于会影响到所有`APP`，需要申请修改`Settings `的权限`<uses-permission
	 * android:name="android.permission.WRITE_SETTINGS"/>`，同时需要取消光感自动调节屏幕亮度的功能，设置为手动调节模式，否则光感传感器会随着光照强度的变化修改系统屏幕亮度，并且非系统签名的`APP`，需要用户手动授权后才可以修改背光亮度。

#####关闭光感，设置手动调节背光模式实现方法如下：

```
	/**
	 * 3.关闭光感，设置手动调节背光模式
	 * 
	 * SCREEN_BRIGHTNESS_MODE_AUTOMATIC 自动调节屏幕亮度模式值为1
	 * 
	 * SCREEN_BRIGHTNESS_MODE_MANUAL 手动调节屏幕亮度模式值为0
	 * **/
	public void setScreenManualMode(Context context) {
		ContentResolver contentResolver = context.getContentResolver();
		try {
			int mode = Settings.System.getInt(contentResolver,
					Settings.System.SCREEN_BRIGHTNESS_MODE);
			if (mode == Settings.System.SCREEN_BRIGHTNESS_MODE_AUTOMATIC) {
				Settings.System.putInt(contentResolver,
						Settings.System.SCREEN_BRIGHTNESS_MODE,
						Settings.System.SCREEN_BRIGHTNESS_MODE_MANUAL);
			}
		} catch (Settings.SettingNotFoundException e) {
			e.printStackTrace();
		}
	}
```
##### 非系统签名应用，引导用户手动授权修改Settings 权限

非系统签名应用，无法直接修改`Settings`,需要引导用户手动授权。
![引导用户手动授权](https://upload-images.jianshu.io/upload_images/5851256-6a5f49cc12e3b8f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```

	/**
	 * 4.非系统签名应用，引导用户手动授权修改Settings 权限
	 * **/
	private static final int REQUEST_CODE_WRITE_SETTINGS = 1000;

	private void allowModifySettings() {
		// Settings.System.canWrite(MainActivity.this)
		// 检测是否拥有写入系统 Settings 的权限
		if (!Settings.System.canWrite(MainActivity.this)) {
			AlertDialog.Builder builder = new AlertDialog.Builder(this,
					android.R.style.Theme_Material_Light_Dialog_Alert);
			builder.setTitle("请开启修改屏幕亮度权限");
			builder.setMessage("请点击允许开启");
			// 拒绝, 无法修改
			builder.setNegativeButton("拒绝",
					new DialogInterface.OnClickListener() {
						@Override
						public void onClick(DialogInterface dialog, int which) {
							Toast.makeText(MainActivity.this,
									"您已拒绝修系统Setting的屏幕亮度权限", Toast.LENGTH_SHORT)
									.show();
						}
					});
			builder.setPositiveButton("去开启",
					new DialogInterface.OnClickListener() {
						@Override
						public void onClick(DialogInterface dialog, int which) {
							// 打开允许修改Setting 权限的界面
							Intent intent = new Intent(
									Settings.ACTION_MANAGE_WRITE_SETTINGS, Uri
											.parse("package:"
													+ getPackageName()));
							startActivityForResult(intent,
									REQUEST_CODE_WRITE_SETTINGS);
						}
					});
			builder.setCancelable(false);
			builder.show();
		}
	}

	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		// TODO Auto-generated method stub
		super.onActivityResult(requestCode, resultCode, data);
		if (requestCode == REQUEST_CODE_WRITE_SETTINGS) {
			if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
				// Settings.System.canWrite方法检测授权结果
				if (Settings.System.canWrite(getApplicationContext())) {
					// 5.调用修改Settings屏幕亮度的方法 屏幕亮度值 200
					ModifySettingsScreenBrightness(MainActivity.this, 200);
					Toast.makeText(this, "系统屏幕亮度值" + getScreenBrightness(this),
							Toast.LENGTH_SHORT).show();
				} else {
					Toast.makeText(MainActivity.this, "您已拒绝修系统Setting的屏幕亮度权限",
							Toast.LENGTH_SHORT).show();
				}
			}
		}
	}

```
##### 修改Setting 中屏幕亮度值 实现
拥有系统签名的应用可以直接调用此方法修改系统屏幕亮度，非系统签名应用，只有用户授权后才可以修改。

```
/**
	 * 5.修改Setting 中屏幕亮度值
	 * 
	 * 修改Setting的值需要动态申请权限 <uses-permission
	 * android:name="android.permission.WRITE_SETTINGS"/>
	 * **/
	private void ModifySettingsScreenBrightness(Context context,
			int birghtessValue) {
		// 首先需要设置为手动调节屏幕亮度模式
		setScreenManualMode(context);

		ContentResolver contentResolver = context.getContentResolver();
		Settings.System.putInt(contentResolver,
				Settings.System.SCREEN_BRIGHTNESS, birghtessValue);
	}
```

# 四、完整代码实现

完整代码实现如下：
```
package com.example.test;

import android.app.Activity;
import android.app.AlertDialog;
import android.content.ContentResolver;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.provider.Settings;
import android.view.Window;
import android.view.WindowManager;
import android.widget.Toast;

public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		//获取屏幕亮度
		getScreenBrightness(this);
		Toast.makeText(this, "系统屏幕亮度值" + getScreenBrightness(this),
				Toast.LENGTH_SHORT).show();
		// 设置APP 屏幕亮度后，系统Setting亮度将对此app 不生效
		setAppScreenBrightness(100);
		allowModifySettings();
		setContentView(R.layout.activity_main);

	}

	/**
	 * 1.获取系统默认屏幕亮度值 屏幕亮度值范围（0-255）
	 * **/
	private int getScreenBrightness(Context context) {
		ContentResolver contentResolver = context.getContentResolver();
		int defVal = 125;
		return Settings.System.getInt(contentResolver,
				Settings.System.SCREEN_BRIGHTNESS, defVal);
	}

	/**
	 * 2.设置 APP界面屏幕亮度值方法
	 * **/
	private void setAppScreenBrightness(int birghtessValue) {
		Window window = getWindow();
		WindowManager.LayoutParams lp = window.getAttributes();
		lp.screenBrightness = birghtessValue / 255.0f;
		window.setAttributes(lp);
	}

	/**
	 * 3.关闭光感，设置手动调节背光模式
	 * 
	 * SCREEN_BRIGHTNESS_MODE_AUTOMATIC 自动调节屏幕亮度模式值为1
	 * 
	 * SCREEN_BRIGHTNESS_MODE_MANUAL 手动调节屏幕亮度模式值为0
	 * **/
	public void setScreenManualMode(Context context) {
		ContentResolver contentResolver = context.getContentResolver();
		try {
			int mode = Settings.System.getInt(contentResolver,
					Settings.System.SCREEN_BRIGHTNESS_MODE);
			if (mode == Settings.System.SCREEN_BRIGHTNESS_MODE_AUTOMATIC) {
				Settings.System.putInt(contentResolver,
						Settings.System.SCREEN_BRIGHTNESS_MODE,
						Settings.System.SCREEN_BRIGHTNESS_MODE_MANUAL);
			}
		} catch (Settings.SettingNotFoundException e) {
			e.printStackTrace();
		}
	}

	/**
	 * 4.非系统签名应用，引导用户手动授权修改Settings 权限
	 * **/
	private static final int REQUEST_CODE_WRITE_SETTINGS = 1000;

	private void allowModifySettings() {
		// Settings.System.canWrite(MainActivity.this)
		// 检测是否拥有写入系统 Settings 的权限
		if (!Settings.System.canWrite(MainActivity.this)) {
			AlertDialog.Builder builder = new AlertDialog.Builder(this,
					android.R.style.Theme_Material_Light_Dialog_Alert);
			builder.setTitle("请开启修改屏幕亮度权限");
			builder.setMessage("请点击允许开启");
			// 拒绝, 无法修改
			builder.setNegativeButton("拒绝",
					new DialogInterface.OnClickListener() {
						@Override
						public void onClick(DialogInterface dialog, int which) {
							Toast.makeText(MainActivity.this,
									"您已拒绝修系统Setting的屏幕亮度权限", Toast.LENGTH_SHORT)
									.show();
						}
					});
			builder.setPositiveButton("去开启",
					new DialogInterface.OnClickListener() {
						@Override
						public void onClick(DialogInterface dialog, int which) {
							// 打开允许修改Setting 权限的界面
							Intent intent = new Intent(
									Settings.ACTION_MANAGE_WRITE_SETTINGS, Uri
											.parse("package:"
													+ getPackageName()));
							startActivityForResult(intent,
									REQUEST_CODE_WRITE_SETTINGS);
						}
					});
			builder.setCancelable(false);
			builder.show();
		}
	}

	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		// TODO Auto-generated method stub
		super.onActivityResult(requestCode, resultCode, data);
		if (requestCode == REQUEST_CODE_WRITE_SETTINGS) {
			if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
				// Settings.System.canWrite方法检测授权结果
				if (Settings.System.canWrite(getApplicationContext())) {
					// 5.调用修改Settings屏幕亮度的方法 屏幕亮度值 200
					ModifySettingsScreenBrightness(MainActivity.this, 200);
					Toast.makeText(this, "系统屏幕亮度值" + getScreenBrightness(this),
							Toast.LENGTH_SHORT).show();
				} else {
					Toast.makeText(MainActivity.this, "您已拒绝修系统Setting的屏幕亮度权限",
							Toast.LENGTH_SHORT).show();
				}
			}
		}
	}

	/**
	 * 5.修改Setting 中屏幕亮度值
	 * 
	 * 修改Setting的值需要动态申请权限 <uses-permission
	 * android:name="android.permission.WRITE_SETTINGS"/>
	 * **/
	private void ModifySettingsScreenBrightness(Context context,
			int birghtessValue) {
		// 首先需要设置为手动调节屏幕亮度模式
		setScreenManualMode(context);

		ContentResolver contentResolver = context.getContentResolver();
		Settings.System.putInt(contentResolver,
				Settings.System.SCREEN_BRIGHTNESS, birghtessValue);
	}
}

```
 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
