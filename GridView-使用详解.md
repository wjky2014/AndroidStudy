
 

##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
> 一、GridView 简介
>二、 GridView 主要使用方法
>三、 GridView 使用Demo








# 一、GridView 简介

在学习GridView 之前，我们需要先了解GridView的继承关系，
GridView的继承关系如下：
```
java.lang.Object
   ↳	android.view.View
 	   ↳	android.view.ViewGroup
 	 	   ↳	android.widget.AdapterView<android.widget.ListAdapter>
 	 	 	   ↳	android.widget.AbsListView
 	 	 	 	   ↳	android.widget.GridView
```
`GridView` 跟`ListView` 很类似，`Listview` 主要以列表形式显示数据，`GridView` 则是以网格形式显示数据，掌握`ListView` 使用方法后，会很轻松的掌握`GridView`的使用方法。

# 二、 GridView 主要使用方法
`GridView `主要通过使用自定义`BaseAdapter` 来适配数据，进而显示到`GridView`中。主要使用方法如下：

## 1. 准备数据源
```
        list = new ArrayList<Map<String, Object>>();
```
##  2. 为数据源设置适配器
    
```
        MyAdapter adapter = new MyAdapter();
```
##  3. 将适配过后点数据显示在GridView 上
```
        gridView.setAdapter(adapter);
```
# 三、 GridView 使用Demo

##1.实现效果如下

![GridView](http://upload-images.jianshu.io/upload_images/5851256-be1b84ecb1871f4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 实现代码如下
```
	private GridView gridView;
	private List<Map<String, Object>> list;
	private int images[] = { R.drawable.gril, R.drawable.ic_launcher,
			R.drawable.gril, R.drawable.ic_launcher, R.drawable.gril,
			R.drawable.ic_launcher, R.drawable.gril, R.drawable.ic_launcher,
			R.drawable.gril };

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_grid_view_method);
		gridView = (GridView) findViewById(R.id.gv);

		// 1. 准备数据源
		list = new ArrayList<Map<String, Object>>();
		for (int i = 0; i < images.length; i++) {
			Map<String, Object> map = new HashMap<String, Object>();
			map.put("image", images[i]);
			map.put("text", "图片" + i);
			list.add(map);

		}

		// 2.为数据源设置适配器
		MyAdapter adapter = new MyAdapter();
		// 3.将适配过后点数据显示在GridView 上
		gridView.setAdapter(adapter);
		// item点击事件处理
		gridView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
			@Override
			public void onItemClick(AdapterView<?> parent, View view,
					int position, long id) {
				//

				Toast.makeText(GridViewMethod.this,
						list.get(position).get("text").toString(),
						Toast.LENGTH_SHORT).show();
			}
		});
	}

	class MyAdapter extends BaseAdapter {

		@Override
		public int getCount() {
			return list.size();
		}

		@Override
		public Object getItem(int position) {
			return list.get(position);
		}

		@Override
		public long getItemId(int position) {
			return position;
		}

		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			ViewHolder holder = null;
			if (convertView == null) {
				// 第一次加载创建View，其余复用 View
				convertView = LayoutInflater.from(GridViewMethod.this).inflate(
						R.layout.gridview_item_img_tv, null);
				holder = new ViewHolder();
				holder.imageView = (ImageView) convertView
						.findViewById(R.id.grid_img);
				holder.textView = (TextView) convertView
						.findViewById(R.id.grid_tv);
				// 打标签
				convertView.setTag(holder);

			} else {
				// 从标签中获取数据
				holder = (ViewHolder) convertView.getTag();
			}

			// 根据key值设置不同数据内容
			holder.imageView.setImageResource((Integer) list.get(position).get(
					"image"));
			holder.textView.setText((String) list.get(position).get("text"));

			return convertView;
		}
	}

	class ViewHolder {
		ImageView imageView;
		TextView textView;

	}


```

## 2.GridView 布局如下
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.android.program.programandroid.ListView.GridViewMethod">


    <GridView
        android:id="@+id/gv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="center"
        android:gravity="center"
        android:horizontalSpacing="10dp"
        android:numColumns="3"
        android:verticalSpacing="10dp" />
</LinearLayout>

```

## 3.item  布局 如下
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/grid_img"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="5dp"
        android:gravity="center_horizontal"
        android:src="@drawable/ic_launcher" />

    <TextView
        android:id="@+id/grid_tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="5dp"
        android:textColor="@android:color/darker_gray"
        android:text="test"
        android:gravity="center_horizontal"
        android:textSize="25sp" />
</LinearLayout>
```

至此`GridView` 的基本使用方法结束。 
如果不是太明白，可以查看上篇文章
[ListView  使用详解](https://www.jianshu.com/p/0eef3be8cd8b)




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

