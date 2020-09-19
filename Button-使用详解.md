

##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Button 的继承关系
>二、Button 简单使用举例
>三、自定义 Button 选择器
>四、Button 点击事件
>五、onClick属性 实现点击事件


# 一、Button 的继承关系

`Button `继承 `TextView`，具体关系如下： 
```
java.lang.Object
   ↳	android.view.View
 	   ↳	android.widget.TextView
 	 	   ↳	android.widget.Button
```

#二、Button  简单使用举例

使用 `xml `布局跟`java`代码动态设置`TextView`。

- 1.`xml `布局如下：
```
 <Button
     android:id="@+id/button_id"
     android:layout_height="wrap_content"
     android:layout_width="wrap_content"
     android:text="@string/self_destruct" />
```
- 2. `java`代码中使用方法如下：

`Button OnClickListener`方法实现如下：
```
 public class MyActivity extends Activity {
     protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);

         setContentView(R.layout.content_layout_id);

         final Button button = findViewById(R.id.button_id);
         button.setOnClickListener(new View.OnClickListener() {
             public void onClick(View v) {
                 // Code here executes on main thread after user presses button
             }
         });
     }
 }
```
# 三、 自定义 Button 选择器

自定义`Button `选择器，可以更加友好的跟用户进行交互。

- 1. `xml `布局使用

```
    <Button
        android:id="@+id/btn_selector"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/custom_btn_green_selector"
        android:text="一、自定义Button背景选择器 "
        android:textColor="@color/white" />
```

- 2.`Button `背景选择器实现

```
<?xml version="1.0" encoding="UTF-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- 按下去的背景颜色显示效果 -->
    <item android:drawable="@drawable/btn_pressed" android:state_pressed="true"/>
    <!-- 获取焦点时背景颜色显示效果 -->
    <item android:drawable="@drawable/btn_pressed" android:state_focused="true"/>
    <!-- 没有任何状态下的背景颜色 -->
    <item android:drawable="@drawable/btn_normal"/>

</selector>
```

- 3. `java `代码中点击实现 效果

```
public class ButtonMethod extends Activity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);

		setContentView(R.layout.activity_button);
		// 一、自定义Button背景选择器、匿名内部类实现点击事件
		Button mBtnSelector = (Button) findViewById(R.id.btn_selector);
		mBtnSelector.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				Toast.makeText(ButtonMethod.this, "你点击了按钮选择器", 1).show();
			}
		});
		// 一、自定义Button背景选择器、匿名内部类实现点击事件
	}
}
```
- 4.` Button` 正常以及获取焦点图片素材


![btn_pressed.9.png](https://upload-images.jianshu.io/upload_images/5851256-ef58c20081b43811.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![btn_normal.9.png](https://upload-images.jianshu.io/upload_images/5851256-46690f1f244ed145.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、Button 点击事件

- 1. `xml` 布局使用
```
    <Button
        android:id="@+id/btn_test"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:text="二、按钮点击事件 实现"
        android:textColor="@color/white" />

```
- 2. `java `代码中点击实现 效果

```
public class ButtonMethod extends Activity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);

		setContentView(R.layout.activity_button);

		// 二、按钮点击事件 实现
		Button mButton = (Button) findViewById(R.id.btn_test);
		BtnClick mBtnClick = new BtnClick();
		mButton.setOnClickListener(mBtnClick);
		// 二、按钮点击事件 实现

	}

	// 二、按钮点击事件 实现
	class BtnClick implements OnClickListener {

		@Override
		public void onClick(View v) {
			// TODO Auto-generated method stub
			Toast.makeText(ButtonMethod.this, "你点击了按钮点击事件 实现", 1).show();
		}

	}

	// 二、按钮点击事件 实现
}
```
# 五、onClick 属性  实现点击事件

- 1. `xml` 布局使用
```
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/custom_btn_white_selector"
        android:onClick="BtnTestonClick"
        android:text="三、使用 onClick 属性待替 Click 事件"
        android:textColor="@color/grey" />
```
- 2. `java` 代码中点击实现 效果
```
public class ButtonMethod extends Activity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_button);
	}

	
	// 三、使用 onClick 实现点击事件
	public void BtnTestonClick(View view) {

		Toast.makeText(this, "你点击了onClick属性按钮", 1).show();
	}
	// 三、使用 onClick 实现点击事件
}
```
- 3. 实现效果如下：

![ 点击实现效果](https://upload-images.jianshu.io/upload_images/5851256-697aea549ef18274.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
