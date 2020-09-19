
##### 和你一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、EditText 继承关系
>二、EditText 常用举例
>三、EditText 自定义背景框
>四、EditText自动检测输入内容
>五、Edittext 密文显示
>六、EditText 限制只能输入特定字符
>七、EditText 输入保存的字符串不能为空


#一、EditText 继承关系

`EditText `继承关系 如下：

```
java.lang.Object    
       ↳ android.view.View     
              ↳ android.widget.TextView      
                      ↳ android.widget.EditText
```
# 二、EditText 常用举例

`EditText  `主要用于输入和修改文本内容。
限制只能输入纯文本内容举例如下：
```
<EditText
      android:id="@+id/plain_text_input"
      android:layout_height="wrap_content"
      android:layout_width="match_parent"
      android:inputType="text"/>

```
# 三、EditText 自定义背景框

- 1. `xml` 中使用`EditText` 控件
```
    <!-- 自定义EditText 背景 -->

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="10dp"
        android:background="@drawable/custom_edittext_background"
        android:gravity="center"
        android:hint="一、自定义EditText背景框"
        android:padding="8dp"
        android:textSize="16sp" />

```
- 2. 自定义 `EditText` 背景框
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">

    <!-- 圆角-->
    <corners android:radius="5dp" />
    <!--描边-->
    <stroke
        android:width="1dp"
        android:color="@android:color/holo_blue_light" />

</shape>
```
- 3. 实现效果

![自定义背景框实现](https://upload-images.jianshu.io/upload_images/5851256-c7e2b3ca1eac606d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、EditText自动检测输入内容

- 1. `xml` 中使用`EditText` 控件
```
    <!-- 自动检测输入更正 -->

    <EditText
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:hint="二、自动检测输入更正属性 autoText"
        android:textColor="#FF6100" />
```

- 2. 实现效果

![自动检测输入正确性
](https://upload-images.jianshu.io/upload_images/5851256-654b52236c9ffa4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 五、Edittext 密文显示
- 1. `xml` 中使用`EditText` 控件

```
    <!-- 以密文的形式显示 -->

    <EditText
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="三、以密文的形式显示密码"
        android:password="true" />

```
- 2. 实现效果

![密文显示属性](https://upload-images.jianshu.io/upload_images/5851256-0016baa565a6bb32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 六、EditText 限制只能输入特定字符

限定只能输入阿拉伯数字实现如下：
- 1. `xml` 中使用`EditText` 控件
```
   <!-- 设置允许输入的字符 -->

    <EditText
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:digits="123456789.+-*/\n()"
        android:hint="四、设置限制允许输入阿拉伯数字" />

```
- 2. 实现效果

![限定只能输入阿拉伯数字](https://upload-images.jianshu.io/upload_images/5851256-8bf6de679f461a7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#七、EditText  输入保存的字符串不能为空

`EditText  `常用来获取用户输入内容，因为我们要规避用户输入的内容为空的情况。
- 1. 实现效果如下：
![EditText  输入保存的字符串不能为空](https://upload-images.jianshu.io/upload_images/5851256-85d21e1670a07785.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 2.实现代码如下：

```
public class EditTextMethod extends Activity {
	EditText mEditText;
	Button mBtn;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_edittext);
		mEditText = (EditText) findViewById(R.id.test_et);
		mBtn = (Button) findViewById(R.id.btn_commit);

		mBtn.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				if (!TextUtils.isEmpty(mEditText.getText().toString().trim())) {

					Toast.makeText(
							EditTextMethod.this,
							"你输入的字符串是：" + mEditText.getText().toString().trim(),
							1).show();
				} else {
					Toast.makeText(EditTextMethod.this, "输入的字符串不能为空", 1).show();
				}
			}
		});

	}

}
```




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
