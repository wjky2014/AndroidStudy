 

#####  和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、 Framework 层字符串添加
>二、Service 中实时监测 电池异常温度并弹窗提醒用户


# 检测电池温度，提示用户温度异常，请注意

`Android `电池信息状态主要是在`frameworks/base/services/core/java/com/android/server/BatteryService.java`，本文也是基于此`BatteryService`实时监测 电池温度，及时提醒用户，为了安全起见，在电池温度异常时候，请勿继续充电。

# 一、 Framework 层字符串添加

##### 1.添加弹窗字符串资源

`alps/frameworks/base/core/res/res/values/strings.xml`
```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
    ... ...
    <!-- add NTC Test Case-->
	<string name="warningLowBatteryTemperature">"Battery temperature is too low, in order to protect battery life, charging function will be temporarily stopped."</string>
	<string name="warningHightBatteryTemperature">"Battery temperature is too high, in order to protect battery life, charging function will be temporarily stopped."</string>
	<string name="shutdownBatteryTemperatureMsg">"Battery temperature is too high. For safety reasons, the phone will shut down automatically later."</string>
	<string name="batteryTemperatureWarning">Warning</string>
	<string name="batteryTemperatureOk">"OK"</string>
	<!-- add NTC Test Case-->	
    ... ...
</resources>
```
##### 2.添加自定义电池异常温度定义

自定义温度 主要修改`alps/frameworks/base/core/res/res/values/config.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
    ... ...
    <!-- Shutdown if the battery temperature exceeds (this value * 0.1) Celsius. -->
    <integer name="config_shutdownBatteryTemperature">680</integer>
    <!-- add NTC Test Case-->
	<integer name="config_warningHightBatteryTemperature">550</integer>
	<integer name="config_warningLowBatteryTemperature">0</integer>
	<!-- add NTC Test Case-->
    ... ...
</resources>
```
##### 3. 在symbols 文件中添加对应java-symbol方便Framework代码引用
在`symbols`文件中为所有`string`，`int`值注册，方便`Framework`层代码的引用。
`alps/frameworks/base/core/res/res/values/symbols.xml`

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
  <!-- Private symbols that we need to reference from framework code.  See
       frameworks/base/core/res/MakeJavaSymbols.sed for how to easily generate
       this.

       Can be referenced in java code as: com.android.internal.R.<type>.<name>
       and in layout xml as: "@*android:<type>/<name>"
  -->
... ... 
  <java-symbol type="integer" name="config_shutdownBatteryTemperature" />
  <!-- add NTC Test Case-->
  <java-symbol type="integer" name="config_warningHightBatteryTemperature" />
  <java-symbol type="integer" name="config_warningLowBatteryTemperature" />
  <java-symbol type="string" name="warningLowBatteryTemperature" />
  <java-symbol type="string" name="warningHightBatteryTemperature" />
  <java-symbol type="string" name="shutdownBatteryTemperatureMsg" />
  <java-symbol type="string" name="batteryTemperatureWarning" />
  <java-symbol type="string" name="batteryTemperatureOk" />
  <!-- add NTC Test Case--> 
```



# 二、Service 中实时监测 电池异常温度并弹窗提醒用户

`BatteryService`后台服务可以实时监测 电池问题，当电池温度异常时候，我们要及时提醒用户，如果此时用户正在充电，提示请勿充电等。

实现方法如下：
```
//add NTC Test Case
import android.app.AlertDialog;
import android.view.WindowManager;
import android.content.DialogInterface;
//add NTC Test Case

... ... 
/**
 * <p>BatteryService monitors the charging status, and charge level of the device
 * battery.  When these values change this service broadcasts the new values
 * to all {@link android.content.BroadcastReceiver IntentReceivers} that are
 * watching the {@link android.content.Intent#ACTION_BATTERY_CHANGED
 * BATTERY_CHANGED} action.</p>
 * <p>The new values are stored in the Intent data and can be retrieved by
 * calling {@link android.content.Intent#getExtra Intent.getExtra} with the
 * following keys:</p>
 * <p>&quot;scale&quot; - int, the maximum value for the charge level</p>
 * <p>&quot;level&quot; - int, charge level, from 0 through &quot;scale&quot; inclusive</p>
 * <p>&quot;status&quot; - String, the current charging status.<br />
 * <p>&quot;health&quot; - String, the current battery health.<br />
 * <p>&quot;present&quot; - boolean, true if the battery is present<br />
 * <p>&quot;icon-small&quot; - int, suggested small icon to use for this state</p>
 * <p>&quot;plugged&quot; - int, 0 if the device is not plugged in; 1 if plugged
 * into an AC power adapter; 2 if plugged in via USB.</p>
 * <p>&quot;voltage&quot; - int, current battery voltage in millivolts</p>
 * <p>&quot;temperature&quot; - int, current battery temperature in tenths of
 * a degree Centigrade</p>
 * <p>&quot;technology&quot; - String, the type of battery installed, e.g. "Li-ion"</p>
 *
 * <p>
 * The battery service may be called by the power manager while holding its locks so
 * we take care to post all outcalls into the activity manager to a handler.
 *
 * FIXME: Ideally the power manager would perform all of its calls into the battery
 * service asynchronously itself.
 * </p>
 */
public final class BatteryService extends SystemService {
	// add NTC Test Case 定义电池温度检测所需变量
	private int mConfigWarningLowBatteryTemperature;
	private int mConfigWarningHightBatteryTemperature;
	private String mWarningLowBatteryTemperature;
	private String mWarningHightBatteryTemperature;
	private String mShutdownBatteryTemperatureMsg;
	private	AlertDialog mbatteryTemperatureDialog = null;
	//add NTC Test Case
        ... ...
    public BatteryService(Context context) {
        super(context);
 
       ... ... 
        //add NTC Test Case BatteryService 构造获取电池检测所需资源
        mConfigWarningLowBatteryTemperature= mContext.getResources().getInteger(
                com.android.internal.R.integer.config_warningLowBatteryTemperature);
        mConfigWarningHightBatteryTemperature= mContext.getResources().getInteger(
                com.android.internal.R.integer.config_warningHightBatteryTemperature);
        mWarningHightBatteryTemperature = mContext.getResources().getString(
                com.android.internal.R.string.warningHightBatteryTemperature);
        mShutdownBatteryTemperatureMsg = mContext.getResources().getString(
                com.android.internal.R.string.shutdownBatteryTemperatureMsg);
        mWarningLowBatteryTemperature = mContext.getResources().getString(
                com.android.internal.R.string.warningLowBatteryTemperature);
		//add NTC Test Case
    }

    private void processValuesLocked(boolean force) {
          ... ...
        shutdownIfNoPowerLocked();
        shutdownIfOverTempLocked();
        //add NTC Test Case 
       //调用自定义电池温度异常检测方法
        warningBatteryTemperature();
        //add NTC Test Case
         ... ...
    }
  
     ... ... 

    //add NTC Test Case 自定义判断电池温度是否异常
    private void warningBatteryTemperature(){
		if(mPlugType != BATTERY_PLUGGED_NONE && mHealthInfo!=null && mbatteryTemperatureDialog == null){
		   if (mHealthInfo.batteryTemperature >= mConfigWarningHightBatteryTemperature) {
				mHandler.post(new Runnable() {
					public void run() {
                        if(mHealthInfo.batteryTemperature >= (mShutdownBatteryTemperature-10)){
						    batteryTemperatureDialog(mShutdownBatteryTemperatureMsg);
						}else{
						    batteryTemperatureDialog(mWarningHightBatteryTemperature);
						}
					}
				});
		   }
		   if (mHealthInfo.batteryTemperature < mConfigWarningLowBatteryTemperature) {
				 mHandler.post(new Runnable() {
					public void run() {
						batteryTemperatureDialog(mWarningLowBatteryTemperature);
					}
				  });
		   } 
		}
	}
	// 自定义电池温度Dialog弹窗
	private void batteryTemperatureDialog(String batteryTemperature) {

		AlertDialog.Builder builder = new AlertDialog.Builder(mContext);
		builder.setTitle(mContext.getResources().getString(
				com.android.internal.R.string.batteryTemperatureWarning));
		builder.setCancelable(false);
		builder.setMessage(batteryTemperature);
		builder.setIconAttribute(android.R.attr.alertDialogIcon);
		builder.setPositiveButton(com.android.internal.R.string.batteryTemperatureOk,
				new DialogInterface.OnClickListener() {
					public void onClick(DialogInterface dialog, int id) {
						dialog.cancel();
						mbatteryTemperatureDialog = null;
					}
				});
		mbatteryTemperatureDialog = builder.create();
		mbatteryTemperatureDialog.getWindow().setType(
				WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY);
		if (mbatteryTemperatureDialog != null&& !mbatteryTemperatureDialog.isShowing()) {
		 	mbatteryTemperatureDialog.show();
		}

	}
	//add NTC Test Case
       ... ...
}
```
 


![](https://upload-images.jianshu.io/upload_images/5851256-9942702b8d1a73b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
