 
##### 和您一起终身学习，这里是程序员Android

# 1. 在AndroidMainfest.xml 中申请 WAKE_LOCK 唤醒锁权限

```
<?xml version="1.0" encoding="utf-8"?>

    ... ...

    <uses-permission android:name="android.permission.WAKE_LOCK" />

    ... ...

</manifest>
```
# 2. Activity OnCreate 方法中设置Flag

```
... ...

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        /**
         * 屏幕常亮需要 申请屏幕 WAKE_LOCK 唤醒锁 权限
         *  <uses-permission android:name="android.permission.WAKE_LOCK" />
         * **/
		getWindow().setFlags(android.view.WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON,
				android.view.WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
        setContentView(R.layout.activity_main);

    }

}
```

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
