 

##### 和您一起终身学习，这里是程序员Android 

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、实现效果
>二、主要实现思路
>三、主要实现代码
>四、在Framework 层添加资源的方法 

# 一、实现效果

仿华为`USB Mode`弹窗实现效果如下：
![底部USB mode 弹窗实现](https://upload-images.jianshu.io/upload_images/5851256-d8b1c3f5db6949ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 二、主要实现思路

主要实现思路如下：
在手机连接`USB`发送/取消通知的同时，控制弹窗`Dialog `的显示/消失。


#三、主要实现代码

连接`USB`发送/取消通知主要实现是在`UsbDeviceManager.java`类中,类路径如下：
`frameworks\base\services\usb\java\com\android\server\usb\UsbDeviceManager.java`，通过查看此类代码，实现想要的功能。
修改实现代码如下：
```
// add start by wj for usb mode
import android.app.AlertDialog;
import android.app.AlertDialog.Builder;
import android.content.DialogInterface;
import android.view.WindowManager;
import android.view.Gravity;
// add end by wj for usb mode

/**
 * UsbDeviceManager manages USB state in device mode.
 */
public class UsbDeviceManager implements ActivityManagerInternal.ScreenObserver {

    private static final String TAG = UsbDeviceManager.class.getSimpleName();
    ... ...
	// add start by wj for usb mode
    private static UsbHandler mHandler;
	private static AlertDialog mServiceDialog = null;
	// add end by wj for usb mode
    ... ...
      protected void updateUsbNotification(boolean force) {
            if (mNotificationManager == null || !mUseUsbNotification
                    || ("0".equals(getSystemProperty("persist.charging.notify", "")))) {
                return;
            }

            // Dont show the notification when connected to a USB peripheral
            // and the link does not support PR_SWAP and DR_SWAP
            if (mHideUsbNotification && !mSupportsAllCombinations) {
                if (mUsbNotificationId != 0) {
                    mNotificationManager.cancelAsUser(null, mUsbNotificationId,
                            UserHandle.ALL);
                    mUsbNotificationId = 0;
                    Slog.d(TAG, "Clear notification");
                }
                return;
            }

            int id = 0;
            int titleRes = 0;
            Resources r = mContext.getResources();
            CharSequence message = r.getText(
                    com.android.internal.R.string.usb_notification_message);
            if (mAudioAccessoryConnected && !mAudioAccessorySupported) {
                titleRes = com.android.internal.R.string.usb_unsupported_audio_accessory_title;
                id = SystemMessage.NOTE_USB_AUDIO_ACCESSORY_NOT_SUPPORTED;
            } else if (mConnected) {
                if (mCurrentFunctions == UsbManager.FUNCTION_MTP) {
                    titleRes = com.android.internal.R.string.usb_mtp_notification_title;
                    id = SystemMessage.NOTE_USB_MTP;
                } else if (mCurrentFunctions == UsbManager.FUNCTION_PTP) {
                    titleRes = com.android.internal.R.string.usb_ptp_notification_title;
                    id = SystemMessage.NOTE_USB_PTP;
                } else if (mCurrentFunctions == UsbManager.FUNCTION_MIDI) {
                    titleRes = com.android.internal.R.string.usb_midi_notification_title;
                    id = SystemMessage.NOTE_USB_MIDI;
                } else if (mCurrentFunctions == UsbManager.FUNCTION_RNDIS) {
                    titleRes = com.android.internal.R.string.usb_tether_notification_title;
                    id = SystemMessage.NOTE_USB_TETHER;
                } else if (mCurrentFunctions == UsbManager.FUNCTION_ACCESSORY) {
                    titleRes = com.android.internal.R.string.usb_accessory_notification_title;
                    id = SystemMessage.NOTE_USB_ACCESSORY;
                }
                if (mSourcePower) {
                    if (titleRes != 0) {
                        message = r.getText(
                                com.android.internal.R.string.usb_power_notification_message);
                    } else {
                        titleRes = com.android.internal.R.string.usb_supplying_notification_title;
                        id = SystemMessage.NOTE_USB_SUPPLYING;
                    }
                } else if (titleRes == 0) {
                    titleRes = com.android.internal.R.string.usb_charging_notification_title;
                    id = SystemMessage.NOTE_USB_CHARGING;
                }
            } else if (mSourcePower) {
                titleRes = com.android.internal.R.string.usb_supplying_notification_title;
                id = SystemMessage.NOTE_USB_SUPPLYING;
            } else if (mHostConnected && mSinkPower && mUsbCharging) {
                titleRes = com.android.internal.R.string.usb_charging_notification_title;
                id = SystemMessage.NOTE_USB_CHARGING;
            }
            if (id != mUsbNotificationId || force) {
                // clear notification if title needs changing
                if (mUsbNotificationId != 0) {
                    mNotificationManager.cancelAsUser(null, mUsbNotificationId,
                            UserHandle.ALL);
                    Slog.d(TAG, "Clear notification");
                    mUsbNotificationId = 0;
                    // add start by wj for usb mode
                    if (mServiceDialog != null) {
                        mServiceDialog.dismiss();
                        mServiceDialog=null;
                    }
                    // add start by wj for usb mode
               .... ... 
                }
}


        protected void updateAdbNotification(boolean force) {
            if (mNotificationManager == null) return;
            final int id = SystemMessage.NOTE_ADB_ACTIVE;
            final int titleRes = com.android.internal.R.string.adb_active_notification_title;

            if (mAdbEnabled && mConnected) {
                if ("0".equals(getSystemProperty("persist.adb.notify", ""))) return;

                if (force && mAdbNotificationShown) {
                    mAdbNotificationShown = false;
                    mNotificationManager.cancelAsUser(null, id, UserHandle.ALL);
                }

                if (!mAdbNotificationShown) {
                    Resources r = mContext.getResources();
                    CharSequence title = r.getText(titleRes);
                    CharSequence message = r.getText(
                            com.android.internal.R.string.adb_active_notification_message);

                    Intent intent = new Intent(Settings.ACTION_APPLICATION_DEVELOPMENT_SETTINGS);
                    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK
                            | Intent.FLAG_ACTIVITY_CLEAR_TASK);
                    PendingIntent pi = PendingIntent.getActivityAsUser(mContext, 0,
                            intent, 0, null, UserHandle.CURRENT);
                    Notification notification =
                            new Notification.Builder(mContext, SystemNotificationChannels.DEVELOPER)
                                    .setSmallIcon(com.android.internal.R.drawable.stat_sys_adb)
                                    .setWhen(0)
                                    .setOngoing(true)
                                    .setTicker(title)
                                    .setDefaults(0)  // please be quiet
                                    .setColor(mContext.getColor(
                                            com.android.internal.R.color
                                                    .system_notification_accent_color))
                                    .setContentTitle(title)
                                    .setContentText(message)
                                    .setContentIntent(pi)
                                    .setVisibility(Notification.VISIBILITY_PUBLIC)
                                    .extend(new Notification.TvExtender()
                                            .setChannelId(ADB_NOTIFICATION_CHANNEL_ID_TV))
                                    .build();
                    mAdbNotificationShown = true;
                    mNotificationManager.notifyAsUser(null, id, notification,
                            UserHandle.ALL);
					// add start by wj for usb mode
					final String USBModeStr[] = mContext.getResources().getStringArray(com.android.internal.R.array.spro_usb_mode);
					AlertDialog.Builder builder = new AlertDialog.Builder(mContext);
					builder.setTitle(mContext.getResources().getString(com.android.internal.R.string.usb_mode_tittle));
					builder.setSingleChoiceItems(USBModeStr, 0,
						new DialogInterface.OnClickListener() {
							@Override
							public void onClick(DialogInterface dialog,
									int which) {
								Slog.e(TAG, "------USBModeStr[which]--------"+USBModeStr[which]);
								dialog.dismiss();
						switch (which) {
					        case 1:
							       mHandler.sendMessage(MSG_SET_CURRENT_FUNCTIONS, UsbManager.FUNCTION_MTP);
								break;
							case 2:
								    mHandler.sendMessage(MSG_SET_CURRENT_FUNCTIONS, UsbManager.FUNCTION_PTP);
								break;

							default:
								    mHandler.sendMessage(MSG_SET_CURRENT_FUNCTIONS, UsbManager.FUNCTION_NONE);
								break;
						}
							}
						});
						builder.setPositiveButton(mContext.getResources().getString(com.android.internal.R.string.usb_mode_cancel),
												   new DialogInterface.OnClickListener() {
														public void onClick(DialogInterface dialog, int id) {
															dialog.cancel();

														}
														});
						mServiceDialog = builder.create();
						mServiceDialog.getWindow().setType(
								WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY);
						mServiceDialog.getWindow().setGravity(Gravity.BOTTOM);
						if (mServiceDialog != null && !mServiceDialog.isShowing()) {
							mServiceDialog.show();
						}
						// add start by wj for usb mode
                }
            } else if (mAdbNotificationShown) {
                mAdbNotificationShown = false;
                mNotificationManager.cancelAsUser(null, id, UserHandle.ALL);
            }
        }


}
```

# 四、在Framework 层添加资源的方法

三种`USB  Mode` 的资源需要在`Framework`资源文件中添加。

在`Framework`中添加资源的方案如下：

####1. 添加strings 资源

字符串需要多语言支持，一般建议至少支持英文 、中文。

#####添加英文资源

英文主要修改`values`文件下的`strings.xml`，
路径如下：`frameworks\base\core\res\res\values\strings.xml`

添加内容如下：
```
<resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
     ... ... 
	<!--start usb mode by wj-->
	<string name="usb_mode_tittle">Use USB for</string>
	<string name="usb_mode_charging_only">Charging only</string>
	<string name="usb_mode_mtp">File Transfer(MTP)</string>
	<string name="usb_mode_ptp">Transfer Photos(PTP)</string>
	<string name="usb_mode_cancel">Cancel</string>
	<!--end usb mode by wj-->
     ... ... 
</resources>
```
####添加中文资源

中文主要修改`values-zh-rCN`文件下的`strings.xml`，
修改路径如下：
`frameworks\base\core\res\res\values-zh-rCN\strings.xml`

添加内容如下：
```
<resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
     ... ... 
	<!--start usb mode by wj-->
	<string name="usb_mode_tittle">USB 用途</string>
	<string name="usb_mode_charging_only">仅限充电</string>
	<string name="usb_mode_mtp">传输文件(MTP)</string>
	<string name="usb_mode_ptp">传输照片(PTP)</string>
	<string name="usb_mode_cancel">取消</string>
	<!--end usb mode by wj-->
     ... ... 
</resources>
```


####2.添加引用的数组资源

因为3种 `USB mode`，引用了数组资源，所以需要在`Framework`层添加数组资源。

在`Framework`中添加数组主要修改`values`下`arrays.xml`文件`frameworks\base\core\res\res\values\arrays.xml`
添加内容如下：
```
<resources xmlns:xliff="urn:oasis:names:tc:xliff:document:1.2">
     ... ... 
	<!--start usb mode by wj-->
    <string-array name="spro_usb_mode">
          <item>@string/usb_mode_charging_only</item>
          <item>@string/usb_mode_mtp</item>
          <item>@string/usb_mode_ptp</item>
    </string-array>
    <!--end usb mode by wj-->
     ... ... 
</resources>
```
#### 3.修改Framwork 资源，需要添加symbol，否则无法引用
在`Framework`中添加资源后，由于无法像`Eclipse`或者`Androd Studio`那样自动生成`R.java`文件，需要在`symbols.xml`文件中手动添加自己的资源文件名，否则会导致无法根据`com.android.internal.R.**`引用所需的字符串资源。
`symbols.xml`主要在`valuse`文件夹下，详细路径为`frameworks\base\core\res\res\values\symbols.xml`

添加内容如下：
```
<resources>
     ... ... 
  <!--start usb mode by wj-->
  <java-symbol type="string" name="usb_mode_tittle" />
  <java-symbol type="string" name="usb_mode_charging_only" />
  <java-symbol type="string" name="usb_mode_mtp" />
  <java-symbol type="string" name="usb_mode_ptp" />
  <java-symbol type="string" name="usb_mode_cancel" />
  <java-symbol type="array" name="spro_usb_mode" />
  <!--end usb mode by wj-->
     ... ... 
</resources>
```



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
