 
##### 和您一起终身学习，这里是程序员Android 

**BatteryService**是电池管理的重要服务，该服务继承`SystemService`，主要用于管理 电池的充电状态，充电百分比等。

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>1. BatteryService 简介
>2. BatteryService 继承关系
>3. BatteryService 构造函数
>4. BatteryService 开始引导阶段
>5. processValuesLocked 处理电池属性变化事件
>6. 低电量关机，高温、NTC 关机实现
>7. NTC 电池高低温警示及关机实现
>8. 充电提示音实现





#1. BatteryService 简介
 
**BatteryService**是电池管理的重要服务，该服务继承`SystemService`，主要用于管理 电池的充电状态，充电百分比等。

另外，`power manager` 会吊起`BatteryService`，并获取使用锁，因此在`Activity Manager `中通过Handler post 消息时候，请谨慎使用！
 
#2. BatteryService 继承关系

BatteryService 继承 SystemService，是电池管理中一个重要的服务之一。

![BatteryService 继承 SystemService](https://upload-images.jianshu.io/upload_images/5851256-02483746333697f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#3. BatteryService 构造函数

BatteryService  主要控制手机充电呼吸灯，低电量提示，高温关机等。

 ![BatteryService 构造函数](https://upload-images.jianshu.io/upload_images/5851256-59d2179d1d5fe62a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

电量相关常用配置修改路径如下：
`frameworks\base\core\res\res\values\config.xml`

电量相关常用配置信息如下：

![BatteryService  常用配置信息 ](https://upload-images.jianshu.io/upload_images/5851256-cd93071409d33773.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 4.  BatteryService 开始引导阶段

BatteryService 开始引导阶段加载电池属性，注册监听，监视事件。

![BatteryService 开始引导阶段加载电池属性，注册监听，监视事件](https://upload-images.jianshu.io/upload_images/5851256-71c8ffc4dad7b111.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 5.processValuesLocked 处理电池属性变化事件

updateBatteryWarningLevelLocked() 方法中会调用 processValuesLocked() 处理监听电池信息改变事件。

![processValuesLocked 处理电池属性变化事件
](https://upload-images.jianshu.io/upload_images/5851256-6af3aebc37ae644d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![processValuesLocked 对外广播电池信息变更
](https://upload-images.jianshu.io/upload_images/5851256-15abebb81da78ec0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 6.低电量关机，高温、NTC 关机实现，

![高温关机实现代码](https://upload-images.jianshu.io/upload_images/5851256-0ceb4a5bc4b72d86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![低电量关机](https://upload-images.jianshu.io/upload_images/5851256-288e0957839e6cc1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![NTC 关机模式状态](https://upload-images.jianshu.io/upload_images/5851256-906305565e5d4399.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![NTC 常见配置值 ](https://upload-images.jianshu.io/upload_images/5851256-5f281fc0471d8a8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 7. NTC 电池高低温警示及关机实现

大概实现就是电池在高温或者低温时候给客户一个提示，在低于某个温度后关机。

```
   AlertDialog mNtcWarnDialog = null;
   AlertDialog mNtcShutdownDialog = null;
   private static final int NTC_LEVEL_NORMAL = 0;
    private static final int NTC_LEVEL_HIGH_WARN = 1;
    private static final int NTC_LEVEL_CRITICAL_HIGH_SHUTDOWN = 2;
    private static final int NTC_LEVEL_LOW_WARN = 3;
    private static final int NTC_LEVEL_CRITICAL_LOW_SHUTDOWN = 4;
    private int getNtcLevel(int temperature) {
        boolean DEBUG_FAKE_TEMP = false;
        if(DEBUG_FAKE_TEMP) {
            int t = android.os.SystemProperties.getInt("persist.debug.battery.high", 200);
            Slog.w(TAG, "DEBUG: ntc change temperature = " + temperature + " to " + t);
            temperature = t;
        }
        if(temperature >= Features.JAVA_VALUE_NTC_HIGH_SHUTDOWN) {
            return NTC_LEVEL_CRITICAL_HIGH_SHUTDOWN;
        }else if(temperature >= Features.JAVA_VALUE_NTC_HIGH_WARN){
            return NTC_LEVEL_HIGH_WARN;
        }else if(temperature <= Features.JAVA_VALUE_NTC_LOW_SHUTDOWN) {
            return NTC_LEVEL_CRITICAL_LOW_SHUTDOWN;
        }else if(temperature <= Features.JAVA_VALUE_NTC_LOW_WARN) {
            return NTC_LEVEL_LOW_WARN;
        }else {
            return NTC_LEVEL_NORMAL;
        }
    }

    private void doShutdown() {
        mHandler.post(new Runnable() {
            @Override
            public void run() {
                if (mActivityManagerInternal.isSystemReady()) {
                    Slog.w(TAG, "shutdownIfOverTempLockedNtc, batteryTemperature : "
                        + mBatteryProps.batteryTemperature);
                    Intent intent = new Intent(Intent.ACTION_REQUEST_SHUTDOWN);
                    intent.putExtra(Intent.EXTRA_KEY_CONFIRM, false);
                    intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                    mContext.startActivityAsUser(intent, UserHandle.CURRENT);
                }
                mNtcShutdownDialog = null;
                mShutDownTimer = null;
            }
        });
    }

    private AlertDialog getAlertDialog(String msg, boolean cancel, Runnable r) {
        AlertDialog tempDialog = null;
        AlertDialog.Builder b = new AlertDialog.Builder(mContext);
        b.setCancelable(cancel);
        b.setIconAttribute(android.R.attr.alertDialogIcon);
        b.setPositiveButton(android.R.string.ok, null);
        b.setMessage(msg);
        tempDialog = b.create();
        tempDialog.setOnDismissListener(new DialogInterface.OnDismissListener() {
            public void onDismiss(DialogInterface dialog) {
                if(r != null)
                    r.run();
            }
        });
        tempDialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);
        return tempDialog;
    }
    private CountDownTimer mShutDownTimer;
    private void shutdownIfOverTempLockedNtc() {
        // shut down gracefully if temperature is too high (> 68.0C by default)
        // wait until the system has booted before attempting to display the
        // shutdown dialog.
        final int ntcLevel = getNtcLevel(mBatteryProps.batteryTemperature);
        if ((ntcLevel == NTC_LEVEL_CRITICAL_HIGH_SHUTDOWN || ntcLevel == NTC_LEVEL_CRITICAL_LOW_SHUTDOWN)
            && mActivityManagerInternal.isSystemReady()) {
            if(ntcLevel == NTC_LEVEL_CRITICAL_HIGH_SHUTDOWN && Features.JAVA_VALUE_NTC_HIGH_SHUTDOWN_INTERVAL > 0) {
                android.util.Log.d(TAG, "mNtcShutdownDialog=" + mNtcShutdownDialog);
                if(mNtcShutdownDialog == null && mShutDownTimer == null) {
                    mHandler.post(new Runnable(){
                        @Override
                        public void run() {
                            mNtcShutdownDialog = getAlertDialog(mContext.getString(com.android.internal.R.string.battery_temperature_shutdown, 
                                Features.JAVA_VALUE_NTC_HIGH_SHUTDOWN_INTERVAL), false, 
                                new Runnable() {
                                    @Override
                                    public void run() {
                                        mNtcShutdownDialog = null;
                                    }
                                });
                            mShutDownTimer = new CountDownTimer(Features.JAVA_VALUE_NTC_HIGH_SHUTDOWN_INTERVAL * 1000, 1000) {
                                @Override  
                                public void onTick(long millisUntilFinished) {  
                                    if(mNtcShutdownDialog != null) 
                                        mNtcShutdownDialog.setMessage(mContext.getString(com.android.internal.R.string.battery_temperature_shutdown, 
                                            millisUntilFinished/1000));
                                }
                                @Override  
                                public void onFinish() {
                                    doShutdown();
                                }
                            };
                            android.util.Log.d(TAG, "show=" + mNtcShutdownDialog);
                            mNtcShutdownDialog.show();
                            mShutDownTimer.start();
                        }
                    });
                }
            }else{
                doShutdown();
            }
        }

        if((ntcLevel == NTC_LEVEL_HIGH_WARN || ntcLevel == NTC_LEVEL_LOW_WARN)
            && isPoweredLocked(BatteryManager.BATTERY_PLUGGED_ANY)
            && mNtcWarnDialog == null
            && mActivityManagerInternal.isSystemReady()){
            mHandler.post(new Runnable(){
                @Override
                public void run() {
                    if(mNtcWarnDialog == null){
                        if(ntcLevel == NTC_LEVEL_HIGH_WARN) {
                            mNtcWarnDialog = getAlertDialog(mContext.getString(com.android.internal.R.string.battery_temperature_warning_charger), false, 
                                new Runnable() {
                                    @Override
                                    public void run() {
                                        mNtcWarnDialog = null;
                                    }
                                });
                            mNtcWarnDialog.show();
                        }else {
                            mNtcWarnDialog = getAlertDialog(mContext.getString(com.android.internal.R.string.battery_temperature_low_warning_charger), false,
                                new Runnable() {
                                    @Override
                                    public void run() {
                                        mNtcWarnDialog = null;
                                    }
                                });
                            mNtcWarnDialog.show();
                        }
                    }
                }
            });
         }

        if(ntcLevel == NTC_LEVEL_NORMAL){
            if(mNtcWarnDialog != null) {
                mNtcWarnDialog.dismiss();
                mNtcWarnDialog = null;
            }
        }
    }

```

```
<string name="battery_temperature_low_warning_charger">Battery Under Temperature, Remove the charger!</string>
<string name="battery_temperature_warning_charger">Battery Over Temperature, Remove the charger!</string>
<string name="battery_temperature_shutdown">Battery Over Temperature, Phone will shutdown in <xliff:g id="secs">%d</xliff:g>s!</string>
<java-symbol type="string" name="battery_temperature_warning_charger" />
<java-symbol type="string" name="battery_temperature_low_warning_charger" />
<java-symbol type="string" name="battery_temperature_shutdown" />
```
# 8. 充电提示音实现
无论插入座充，还是 USB充电，当连接充电器的时候，发个提示音给客户，表示正在充电，也是一个满友好的设计，具体实现代码如下:
```
private MediaPlayer mMediaPlayer;
	private void StartUSBAlarm() {
		try {
			if (mMediaPlayer == null) {
				mMediaPlayer = new MediaPlayer();
			}
			if (mMediaPlayer.isPlaying()) {
				mMediaPlayer.stop();
			}
			mMediaPlayer.reset();
			mMediaPlayer.setDataSource(this, RingtoneManager
					.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION));
			mMediaPlayer.prepare();
			mMediaPlayer.start();
			mMediaPlayer.setOnCompletionListener(new OnCompletionListener() {

				@Override
				public void onCompletion(MediaPlayer mp) {
					// TODO Auto-generated method stub
					StopUSBAlarm();
				}
			});
			mMediaPlayer.setOnErrorListener(new OnErrorListener() {

				@Override
				public boolean onError(MediaPlayer mp, int what, int extra) {
					StopUSBAlarm();
					return false;
				}
			});
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	private void StopUSBAlarm() {
		if (mMediaPlayer != null) {
			mMediaPlayer.stop();
			mMediaPlayer.reset();
			mMediaPlayer.release();
			mMediaPlayer = null;
		}
	}
```

这里要注意 MediaPlayer 资源使用释放不当，会导致播放声音无效，使用前请注意使用**mMediaPlayer.reset();**。

在 Power Connect 以及disconnect 时候添加声音的播放与停止，实现代码如下：

![在 Power Connect 以及disconnect 时候添加声音的播放与停止](https://upload-images.jianshu.io/upload_images/5851256-8afde6e0418ec5b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
