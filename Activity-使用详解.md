#### 和你一起终身学习，这里是 程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
> 一、Activity 概览
> 二、Activity 生命周期
> 三、Activity 的注册方法
> 四、App的MainActivity
> 五、Activity 的启动方法
> 六、Activity结束方法
>七、Activity状态保存，恢复的方法
> 八、面试中经常问到题型

# 一、Activity 概览

`Activity `是`Android `最基本的四大组件之一（`Activity` 活动，`Service` 服务，`ContentProvider` 内容提供者，`BroadcastReceiver` 广播），`Activity`主要负责与用户进行交互，是每位`Android` 开发必须掌握的知识点。四大组件必须在`AndroidMainfest.xml ` 文件中声明。

`Activity` 继承关系如下：

```
java.lang.Object
   ↳	android.content.Context
 	   ↳	android.content.ContextWrapper
 	 	   ↳	android.view.ContextThemeWrapper
 	 	 	   ↳	android.app.Activity
```

理解完`Activity`的继承关系后，我们开始了解`Activity`的声明周期，`Activity`的生命周期直接影响到与用户的交互，此生命周期很重要。

#二、Activity 生命周期

`Activity`  生命周期图如下：

![Activity 生命周期图](http://upload-images.jianshu.io/upload_images/5851256-fda7c45b040a636b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 在代码中` Activity`生命周期回调方法如下：

```
	// Activity 创建方法
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		Log.i(TAG, "----onCreate----");
		setContentView(R.layout.activity_methods);
	}

	// Activity 在最新任务列表中打开时候会走此方法
	@Override
	protected void onRestart() {
		super.onRestart();
		Log.i(TAG, "----onRestart----");
	}

	// Activity 在onCreate 或者 onRestart之后执行
	@Override
	protected void onStart() {
		super.onStart();
		Log.i(TAG, "----onStart----");
	}

	// 正在与用户交互的界面
	@Override
	protected void onResume() {
		super.onResume();
		Log.i(TAG, "----onResume----");
	}

	// 被其他与用户交互的Activity 部分覆盖
	@Override
	protected void onPause() {
		super.onPause();
		Log.i(TAG, "----onPause----");
	}

	// 被其它与用户交互的Activity 全部覆盖
	@Override
	protected void onStop() {
		super.onStop();
		Log.i(TAG, "----onStop----");
	}

	// Activity 销毁时候调用此方法
	@Override
	protected void onDestroy() {
		super.onDestroy();
		Log.i(TAG, "----onDestroy----");
	}

```

根据不同的生命周期状态，`Activity `可以分为以下四种生命周期状态

- 1.Active  运行状态
- 2.Pause   暂停状态
- 3.Stop    停止状态
- 4.Killed  消亡状态

#  三、 Activity的注册方法

`Activity `是四大组件之一，`Android`规定四大组件必须在`AndroidMainfest.xml` 中注册，`Activity `如果不注册，则会引起 `App Crash`。比如以下常见的`ActivityNotFoundException `。

例如以下`ActivityNotFoundException `报错信息：
```

//提示未在 AndroidMainfest.xml 中找到Activity类的声明
android.content.ActivityNotFoundException:
                           Unable to find explicit activity class 
                           //具体类名，包名如下：
                          {com.wj.utils/com.wj.utils.basewidget.BaseButtonMethods};
                          //需要在AndroidManifest 中声明
                          have you declared this activity in your AndroidManifest.xml?       
                                            
```
![ActivityNotFoundException 异常Log 分析 ](http://upload-images.jianshu.io/upload_images/5851256-534dfbb2cc791c98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因此，我们在使用Activity的时候必须在`AndroidMainfest.xml `中注册，注册方法如下：
```
<manifest ... >
  <application ... >
      <activity android:name=".BaseButtonMethods" />
      ...
  </application ... >
  ...
</manifest >
```
# 四、App的MainActivity

一个`App `会有多个`Activity`，那么我们如何知道哪个`Activity `是 `Main Activity`呢？
其实`Android ` 是通过`android.action.MAIN ` 标签来区别当前`App`的入口 `Main Activity `类。

另外 一半跟随着`android.action.MAIN `标签的地方，同时还有
`android.intent.category.LAUNCHER`这个标签，它表示 此`Action` 会被`Launcher `扫描到，并可以显示在`Launcher`的`app list`列表中。如果去掉此`Action`标签，则无法在`Launcher `中查看到此`app`的图标。

常见举例如下：

```
<activity android:name=".MainActivity">
       <intent-filter>
         <action android:name="android.intent.action.MAIN" />
         <category android:name="android.intent.category.LAUNCHER" />
       </intent-filter>
</activity>
```

# 五、Activity 的启动方法

Activity 的启动方法大致分显示启动、隐式启动、带返回参数启动三种。

## 1. 显示启动

显示启动常用于app 内部Activity 的启动，使用方法如下：

```  

        Intent intent = new Intent(ActivityMethods.this, OtherActivity.class)
        startActivity(intent);

```
## 2.  隐式启动 

隐式启动即可以调用App内部Activity ，也可以调用其他过滤到包含启动Action 的Activity。使用方法如下：

```

        Intent intent = new Intent("string_action");
        //或者分开设置Action也可以
        // intent.setAction("string_action");
        startActivity(intent);

```
## 3. 启动带返回值的Activity

开发过程中我们经常需要启动一个Activity ，然后返回一些数据给启动的Activity，这时候，使用以下启动带返回值的Activity 是最合适的方法。使用方法举例如下：

- ### 1.启动要返回数据的Activity 

```
    ... ...
   // 1.启动要返回数据的Activity 
    Intent intent = new Intent();
    intent.setClass(ActivityMethods.this, OtherActivity.class);
    startActivityForResult(intent, mRequestCode);
    ... ...

    //    2.获取并处理 启动返回Activity 返回结果携带的数据
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == mRequestCode && resultCode == mResultCode) {
            String result = data.getStringExtra("str_set_result");
            Toast.makeText(this, "result :" + result, Toast.LENGTH_SHORT).show();
        }

    }

```
- ### 2. 设置并返回 Bundle 数据类型的数据给启动的Activity

``` 

                int resultCode = 101;
                Intent intent = new Intent();
                intent.putExtra("str_set_result", "带返回结果的Activity");
                setResult(resultCode, intent);

```

#  六、Activity结束 方法

如果想结束掉当前`Activity` ，可以调用一下方法
```

        //1.直接调用finish方法 ，结束当前Activity
        finish();
        //2.根据请求码结束Activity
        finishActivity(int requestCode);

```
# 七、 Activity状态保存，恢复的方法

当`Activity `异常退出时候，`Activity `会自动保存一些数据。为了安全起见，如果是`App`重要的数据，还请在代码中手动保存`Bundle`类型的数据，防止`Activity`自动保存的数据没有保存完整。

Activity 状态保存与恢复的周期图如下：

![Activity 状态保存生命周期图](http://upload-images.jianshu.io/upload_images/5851256-f17d5f0c94c8eecc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


代码中 `Activity `状态保存与恢复的回调方法如下：

``` 

    // Activity 恢复数据的方法,经常在 oncreate 方法中恢复数据
    @Override
    protected void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
        Log.i(TAG, "----onRestoreInstanceState----");
    }

    // Activity 保存数据的方法，经常在 onPause 方法中保存数据
    @Override
    public void onSaveInstanceState(Bundle outState, PersistableBundle outPersistentState) {
        super.onSaveInstanceState(outState, outPersistentState);
        Log.i(TAG, "----onSaveInstanceState----");
    }

```
# 八、 面试中常问的题型

## 1. Activity A 启动 Activity B， 然后再返回A，简述一下 A与B生命周期的调用方法。

 - 1. 首先` A `启动的生命周期如下：

```
01-02 21:25:22.357 21225-21225/com.android.program.programandroid I/ActivityMethods wjwj:: ----onCreate----
01-02 21:25:22.396 21225-21225/com.android.program.programandroid I/ActivityMethods wjwj:: ----onStart----
01-02 21:25:22.402 21225-21225/com.android.program.programandroid I/ActivityMethods wjwj:: ----onResume----
``` 
- 2. 点击`A`中的`Button` ，跳转到`B`，此时声明周期关系如下：
 
```
01-02 21:28:30.617 23845-23845/com.android.program.programandroid I/ActivityMethods wjwj:: ----onPause----
01-02 21:28:30.723 23845-23845/com.android.program.programandroid I/OtherActivity wjwj:: ----onCreate----
01-02 21:28:30.729 23845-23845/com.android.program.programandroid I/OtherActivity wjwj:: ----onStart----
01-02 21:28:30.738 23845-23845/com.android.program.programandroid I/OtherActivity wjwj:: ----onResume----
01-02 21:28:31.320 23845-23845/com.android.program.programandroid I/ActivityMethods wjwj:: ----onStop----
```
- 3. 结束` B`，返回 `A `，生命周期如下：
```
01-02 21:29:38.646 23845-23845/com.android.program.programandroid I/OtherActivity wjwj:: ----onPause----
01-02 21:29:38.668 23845-23845/com.android.program.programandroid I/ActivityMethods wjwj:: ----onRestart----
01-02 21:29:38.672 23845-23845/com.android.program.programandroid I/ActivityMethods wjwj:: ----onStart----
01-02 21:29:38.674 23845-23845/com.android.program.programandroid I/ActivityMethods wjwj:: ----onResume----
01-02 21:29:39.058 23845-23845/com.android.program.programandroid I/OtherActivity wjwj:: ----onStop----
01-02 21:29:39.059 23845-23845/com.android.program.programandroid I/OtherActivity wjwj:: ----onDestroy----
```
**总结：**
`Activity A `启动 `Activity B`， 简述一下其生命周期？ 大致流程如下图：

![Activity A 启动 Activity B， 然后再返回A生命周期大致图](http://upload-images.jianshu.io/upload_images/5851256-ceab18b32fd1e068.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

