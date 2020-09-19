 
##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
> 一、Fragment 简介
> 二、Fragment的设计原理
> 三、Fragment 生命周期
> 四、Fragment 在Activity中的使用方法
> 五、动态添加Fragment到Activity的方法
> 六、Activity 中获取Fragment
>七、Fragment 获取宿主Activity的方法
>八、两个Fragment的通讯的方法
>九、Fragment 与 Activity通讯方法



#一、 Fragment 简介
在学Fragment之前我们需要了解一下Fragment的继承关系，Fragment 继承关系如下：
```
java.lang.Object
   ↳ android.app.Fragment
```

`Fragment` 片段，在`Activity` 中常用于负责用户界面部分，可以将多个`Fragment `组合在一个`Activity`中来创建多窗口`UI`，或者在`Activity`中重复使用某个`Fragment`。您可以将`Fragment` 视为`Activity`的模块化组成部分，`Fragment `具有自己的生命周期，能接收自己的输入事件，并且可以在`Activity `运行时候添加或者移除`Fragment`。

`Fragment `必须嵌套在`Activity`中，其生命周期受`Activity`生命周期的影响。


# 二、Fragment的设计原理

`Fragment `主要是为了给大屏幕（平板等）上更加动态和灵活的`UI`设计提供支持。由于平板电脑的屏幕比手机屏幕大得多，因此可用于组合和交换`UI` 组件的空间更大。利用`Fragment `实现此类设计时，您无需管理对视图层次结构的复杂更改。 通过将 `Activity `布局分成`Fragment`，您可以在运行时修改` Activity` 的外观，并在由 `Activity` 管理的返回栈中保留这些更改。

您应该将每个`Fragment`都设计为可重复使用的模块化` Activity` 组件。也就是说，由于每个`Fragment` 都会通过各自的生命周期回调来定义其自己的布局和行为，您可以将一`Fragment`加入多个` Activity`，因此，您应该采用可复用式设计，避免直接从某个`Fragment`直接操纵另一个`Fragment`。 这特别重要，因为模块化片段让您可以通过更改`Fragment`的组合方式来适应不同的屏幕尺寸。 在设计可同时支持平板电脑和手机的应用时，您可以在不同的布局配置中重复使用您的`Fragment`，以根据可用的屏幕空间优化用户体验。 例如，在手机上，如果不能在同一 `Activity `内储存多个片段，可能必须利用单独片段来实现单窗格` UI`。

![通过两个Fragment组合成一个Activity适应平板，通过一个Fragment来适应手机](http://upload-images.jianshu.io/upload_images/5851256-6145721f6361000c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、Fragment 生命周期

## 1.   Fragment 生命周期图 

![Fragment 生命周期图](http://upload-images.jianshu.io/upload_images/5851256-ca04c0449c0eaca8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2. `Fragment `生命周期回调方法
```
	public void onAttach(Context context) {
		// TODO Auto-generated method stub
		Log.i(TAG, "----onAttach----");
		super.onAttach(context);
	}

	@Override
	public void onCreate(Bundle savedInstanceState) {
		Log.i(TAG, "----onCreate----");
		// 在Fragment 中调用Activity 中的方法
		((FragmentAutoCreate) getActivity()).test();

		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
	}

	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		Log.i(TAG, "----onCreateView----");
		// 将layout布局转换成View
		View view = inflater.inflate(R.layout.fragment_left_layout, null);
		Toast.makeText(getActivity(),
				"通过对外提供方法setMsg(String msg)，供Activity 调用", 0).show();
		return view;

	}

	@Override
	public void onActivityCreated(Bundle savedInstanceState) {
		Log.i(TAG, "----onActivityCreated----");
		// TODO Auto-generated method stub
		super.onActivityCreated(savedInstanceState);
	}

	@Override
	public void onStart() {
		Log.i(TAG, "----onStart----");
		// TODO Auto-generated method stub
		super.onStart();
	}

	@Override
	public void onResume() {
		Log.i(TAG, "----onResume----");
		// TODO Auto-generated method stub
		super.onResume();
	}

	@Override
	public void onPause() {
		Log.i(TAG, "----onPause----");
		// TODO Auto-generated method stub
		super.onPause();
	}

	@Override
	public void onStop() {
		Log.i(TAG, "----onStop----");
		// TODO Auto-generated method stub
		super.onStop();
	}

	@Override
	public void onDestroyView() {
		Log.i(TAG, "----onDestroyView----");
		// TODO Auto-generated method stub
		super.onDestroyView();
	}

	@Override
	public void onDestroy() {
		Log.i(TAG, "----onDestroy----");
		// TODO Auto-generated method stub
		super.onDestroy();
	}

	@Override
	public void onDetach() {
		Log.i(TAG, "----onDetach----");
		// TODO Auto-generated method stub
		super.onDetach();
	}

```


## 3.Fragment 在宿主Activity 的生命周期

`Fragment `不能脱离`Activity`而存在，其生命周期受`Activity` 生命周期影响
![Activity于Fragment生命周期交互图](http://upload-images.jianshu.io/upload_images/5851256-f0d3c2386679c145.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Log 打印信息如上](http://upload-images.jianshu.io/upload_images/5851256-b914d7686b3d9c60.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、Fragment 在Activity中的使用方法

## 1.静态添加Fragment到Activity

```
  <fragment
        android:id="@+id/left_fragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        class="com.programandroid.Fragment.FragmentLeft" />
```
## 2.创建自定义Fragment类

实现方法如下
```
public class FragmentRight extends Fragment {
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		// 将layout布局转换成View
		View view = inflater.inflate(R.layout.fragment_right_layout, null);

		// 取出值
		Bundle values = getArguments();
		if (values != null) {
			String str = values.getString("key", "");
			Toast.makeText(getActivity(), "接收Activity传递的至为：" + str, 0).show();

		} else {
			Toast.makeText(getActivity(), "接收Activity传递的至为空", 0).show();

		}

		return view;

	}
}
```




#五、动态添加Fragment到Activity中

## 1.创建自定义Fragment类  

```
public class FragmentRight extends Fragment {
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		// 将layout布局转换成View
		View view = inflater.inflate(R.layout.fragment_right_layout, null);

		// 取出值
		Bundle values = getArguments();
		if (values != null) {
			String str = values.getString("key", "");
			Toast.makeText(getActivity(), "接收Activity传递的至为：" + str, 0).show();

		} else {
			Toast.makeText(getActivity(), "接收Activity传递的至为空", 0).show();

		}

		return view;

	}
}
```

##  2.创建Fragment填充布局载体

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >

    <FrameLayout
        android:id="@+id/fl_left"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:background="#f00" />

    <FrameLayout
        android:id="@+id/fl_right"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="2"
        android:background="#0f0" />

</LinearLayout>
```

##   3.使用FragmentManager动态填充  

1.创建容器接收`Fragment` 的` Activity`容器。 
2.使用`FragmentManager`与`Fragment `进行交互
3.开启事务将`Fragment` 填充到`Activity` 创建的容器中
4.提交事务。

```
public class FragmentAutoCreate extends Activity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_fragment_autocreate);

		// 获取fragmentManager --->开启事务--->容器中填充内容--->提交事务
		// FragmentManager : Activity 内部用来与Fragment进行交互的接口
		FragmentManager fragmentManager = getFragmentManager();
		// 开启一个事务
		FragmentTransaction transaction = fragmentManager.beginTransaction();
		// 将左右两侧的Fragment 添加的R.id.fl_left 所代表的容器视图中
		FragmentLeft leftFragment = new FragmentLeft();
		transaction.add(R.id.fl_left, leftFragment, "left_tag");
		// 1. 调用 Fragment 对外提供的方法
		leftFragment.setMsg("tttt");

		FragmentRight rightFragment = new FragmentRight();
		// 2. Activity --setArguments-> 创值给Fragment
		Bundle args = new Bundle();
		args.putString("key", "Activity --setArguments-> 创值给Fragment ");
		// 传递数据
		rightFragment.setArguments(args);
		transaction.add(R.id.fl_right, rightFragment);
		// transaction.replace(R.id.fl_right, rightFragment);
		// transaction.hide(rightFragment);
		transaction.show(rightFragment);

		// 提交事务
		transaction.commit();

	}

	public void test() {

	}
}
```

# 六、Activity 中获取Fragment 
```
		Fragment idFragment=getFragmentManager().findFragmentById(R.id.fl_left);
		FragmentRight tagFragment = (FragmentRight) getFragmentManager().findFragmentByTag("left_tag");		
```

# 七、Fragment 获取宿主Activity的方法

## 1. `getActivity() `方法获取`宿主Activity`

```
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		Log.i(TAG, "----onCreateView----");
		// 将layout布局转换成View
		View view = inflater.inflate(R.layout.fragment_left_layout, null);
		Toast.makeText(getActivity(),
				"通过对外提供方法setMsg(String msg)，供Activity 调用", 0).show();
		return view;

	}
```

# 八、两个Fragment的通讯的方法

## 1. 通过宿主`Activity` 到`FragmentManger` 方法获取不同的`Fragment`

```
		Fragment idFragment=getFragmentManager().findFragmentById(R.id.fl_left);
		FragmentRight tagFragment = (FragmentRight) getFragmentManager().findFragmentByTag("left_tag");		
```
# 九、Fragment 与 Activity通讯方法
## 1.Activity 调用 setArguments  方法

```
public class FragmentAutoCreate extends Activity {
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_fragment_autocreate);

		// 获取fragmentManager --->开启事务--->容器中填充内容--->提交事务
		// FragmentManager : Activity 内部用来与Fragment进行交互的接口
		FragmentManager fragmentManager = getFragmentManager();
		// 开启一个事务
		FragmentTransaction transaction = fragmentManager.beginTransaction();
		// 将左右两侧的Fragment 添加的R.id.fl_left 所代表的容器视图中
		FragmentLeft leftFragment = new FragmentLeft();
		transaction.add(R.id.fl_left, leftFragment, "left_tag");
		// 1. 调用 Fragment 对外提供的方法
		leftFragment.setMsg("tttt");

		FragmentRight rightFragment = new FragmentRight();
		// 2. Activity --setArguments-> 创值给Fragment
		Bundle args = new Bundle();
		args.putString("key", "Activity --setArguments-> 创值给Fragment ");
		// 传递数据
		rightFragment.setArguments(args);
		transaction.add(R.id.fl_right, rightFragment);
		// transaction.replace(R.id.fl_right, rightFragment);
		// transaction.hide(rightFragment);
		transaction.show(rightFragment);
		// 提交事务
		transaction.commit();
	}

	public void test() {
	}
}

```
## 2.通过Fragment 对外提供接口方法

通过`Fragment `对外提供接口方法,供`Activity`调用

```
public class FragmentLeft extends Fragment {

	private static final String TAG = "   F   wj";
	private String mMessage;

	public void setMsg(String msg) {
		this.mMessage = msg;
	}


	@Override
	public void onCreate(Bundle savedInstanceState) {
		Log.i(TAG, "----onCreate----");
		// 在Fragment 中调用Activity 中的方法
		((FragmentAutoCreate) getActivity()).test();

		// TODO Auto-generated method stub
		super.onCreate(savedInstanceState);
	}

	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container,
			Bundle savedInstanceState) {
		Log.i(TAG, "----onCreateView----");
		// 将layout布局转换成View
		View view = inflater.inflate(R.layout.fragment_left_layout, null);
		Toast.makeText(getActivity(),
				"通过对外提供方法setMsg(String msg)，供Activity 调用", 0).show();
		return view;

	}

```



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
