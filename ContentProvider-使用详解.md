

#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
> 一、ContentProvider 概述
> 二、ContentProvider的注册
> 三、自定义ContentProvider 的实现
> 四、获取联系人ContactProvider实现的方法
> 五、获取短信内容的实现方法
>六、ContentResolver 内容解析者
>七、ContentObserver 内容观察者
>八、ContentProvider ContentResolver ContentObserver 三者关系


# 一、ContentProvider 概述

 在了解 `ContentProvider`  之前，我们首先了解一下`ContentProvider` 的继承关系。

`ContentProvider` 继承关系如下：

```
java.lang.Object
    ↳ android.content.ContentProvider
```
`ContentProvider `是`Android `四大组件之一，其本质上是一个标准化的数据管道，它屏蔽了底层的数据管理和服务等细节，以标准化的方式在`Android` 应用间共享数据。用户可以灵活实现`ContentProvider `所封装的数据存储以及增删改查等，所有的`ContentProvider` 必须实现一个对外统一的接口`（URI）`。



# 二、ContentProvider的注册
  `ContentProvider` 属于四大组件之一，必须在`Androidmainfest.xml` 中注册。

`ContentProvider`注册方法如下：
```
  <provider           
            android:name="com.programandroid.CustomContentProviderMethod"
            android:authorities="ProgramAndroid"
            android:exported="true" />
```

**注意 :**
URI 中的元素是`android:authorities="ProgramAndroid"`。

# 三、自定义ContentProvider 的实现

自定义`ContentProvider` 需要继承 `ContentProvider` ,并实现增删改查等方法。

##1.自定义ContentProvider 类
```
public class CustomContentProviderMethod extends ContentProvider {

	private SQLiteDatabase db;
	private static final String MAUTHORITIESNAME = "ProgramAndroid";
	private static UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
	private static final int PERSON = 1;
	private static final int PERSON_NUMBER = 2;
	private static final int PERSON_TEXT = 3;
	private static final String TABLE_NAME = "table_person";
	// 构建URI
	static {
		// content://programandroid/person
		matcher.addURI(MAUTHORITIESNAME, "person", PERSON);
		// # 代表任意数字content://programandroid/person/4
		matcher.addURI(MAUTHORITIESNAME, "person/#", PERSON_NUMBER);
		// * 代表任意文本 content://programandroid/person/filter/ssstring
		matcher.addURI(MAUTHORITIESNAME, "person/filter/*", PERSON_TEXT);
	}

	@Override
	public boolean onCreate() {
		DBHelper helper = new DBHelper(getContext());
		// 创建数据库
		db = helper.getWritableDatabase();
		return true;

	}

	@Nullable
	@Override
	public Cursor query(Uri uri, String[] projection, String selection,
			String[] selectionArgs, String sortOrder) {

		// 过滤URI
		int match = matcher.match(uri);
		switch (match) {
		case PERSON:
			// content://autoname/person

			return db.query(TABLE_NAME, projection, selection, selectionArgs,
					null, null, sortOrder);

		case PERSON_NUMBER:
			break;
		case PERSON_TEXT:
			break;
		default:
			break;
		}
		return null;
	}

	@Nullable
	@Override
	public Uri insert(Uri uri, ContentValues values) {
		// 过滤URI
		int match = matcher.match(uri);
		switch (match) {
		case PERSON:
			// content://autoname/person

			long id = db.insert(TABLE_NAME, null, values);

			// 将原有的uri跟id进行拼接从而获取新的uri
			return ContentUris.withAppendedId(uri, id);

		case PERSON_NUMBER:
			break;
		case PERSON_TEXT:
			break;
		default:
			break;
		}
		return null;
	}

	@Override
	public int delete(Uri uri, String selection, String[] selectionArgs) {
		return 0;
	}

	@Override
	public int update(Uri uri, ContentValues values, String selection,
			String[] selectionArgs) {
		return 0;
	}

	@Nullable
	@Override
	public String getType(Uri uri) {
		return null;
	}

}

```
##2. 提供对外提供操作的数据库方法

```
public class DBHelper extends SQLiteOpenHelper {
	private static final String DB_NAME = "persons.db";
	private static final int DB_VERSION = 1;
	private static final String TABLE_NAME = "table_person";
	private static final String ID = "_id";
	private static final String NAME = "name";

	public DBHelper(Context context) {
		super(context, DB_NAME, null, DB_VERSION);
	}

	@Override
	public void onCreate(SQLiteDatabase db) {

		String sql = "CREATE TABLE " + TABLE_NAME + "(" + ID
				+ " INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL" + "," + NAME
				+ " CHAR(10) )";

		db.execSQL(sql);
	}

	@Override
	public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

	}

}

```

## 3.其他APK 访问此ContentProvider 数据库的方法
```
public class MainActivity extends Activity {

	private String uri = "content://ProgramAndroid/person";
	private EditText mEditText;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		mEditText = (EditText) findViewById(R.id.ed_name);
	}

	public void QureyData(View view) {
		String name = null;
		Cursor cursor = getContentResolver().query(Uri.parse(uri), null, null, null, null);
		while (cursor.moveToNext()) {
			name = cursor.getString(cursor.getColumnIndex("name"));
		}
		mEditText.setText(name);
	}

	public void InsertData(View view) {
		String editName = mEditText.getText().toString();
		ContentValues values = new ContentValues();
		values.put("name, editName);

		Uri result = getContentResolver().insert(Uri.parse(uri), values);
//             注意 ： 此条添加上才ContentObserver可以监听数据库改变
		getContentResolver().notifyChange(Uri.parse(uri),null);
		long parseid = ContentUris.parseId(result);
		if (parseid > 0) {
			Toast.makeText(MainActivity.this, "保存成功", Toast.LENGTH_LONG).show();
			mEditText.setText("");
		}

	}

}
```

**注意 ：** 
```
   //  此条添加上才ContentObserver可以监听数据库改变
		getContentResolver().notifyChange(Uri.parse(uri),null);
```
至此，自定义`ContentProvider`的使用方法已经实现。

# 四、获取联系人ContactProvider实现的方法

` Android` 系统自带一些`ContentProvider` ,比如 联系人的`ContactProvider`
例如： 源码` packages\providers` 下的内容

![Android 系统Provider.png](http://upload-images.jianshu.io/upload_images/5851256-cc744daef02bc557.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##1. 获取联系人实现方法
```
public class ContactListActivity extends Activity {
	private static final String tag = "ContactListActivity";
	private ListView lv_contact_list;
	private List<HashMap<String, String>> mContactList = new ArrayList<HashMap<String, String>>();

	private Handler mHandler = new Handler() {
		public void handleMessage(android.os.Message msg) {
			// 给数据适配器设置数据
			MyAdapter myAdapter = new MyAdapter();

			TextView emptyView = new TextView(getApplicationContext());
			emptyView.setLayoutParams(new LayoutParams(
					LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
			emptyView.setText(getResources().getString(
					R.string.please_add_contanct));
			emptyView.setVisibility(View.GONE);
			emptyView.setTextColor(Color.BLACK);
			emptyView.setTextSize(20);
			emptyView.setGravity(Gravity.CENTER);
			((ViewGroup) lv_contact_list.getParent()).addView(emptyView);
			lv_contact_list.setEmptyView(emptyView);

			lv_contact_list.setAdapter(myAdapter);
		};
	};

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_contact_list);

		initUI();
		initData();
	}

	/**
	 * 从系统数据库中获取联系人数据,权限,读取联系人
	 */
	private void initData() {
		new Thread() {
			public void run() {
				// 1,获取内容解析器(访问地址(后门))
				ContentResolver contentResolver = getContentResolver();
				// 2,对数据库指定表进行查询操作
				Cursor cursor = contentResolver.query(Uri
						.parse("content://com.android.contacts/raw_contacts"),
						new String[] { "contact_id" }, null, null, null);
				// 3,判断游标中是否有数据,有数据一直度
				while (cursor.moveToNext()) {
					String id = cursor.getString(0);
					Log.i(tag, "id = " + id);// 1,2,3
					// 4,通过此id去关联data表和mimetype表生成视图,data1(数据),mimetype(数据类型)
					Cursor indexCursor = contentResolver.query(
							Uri.parse("content://com.android.contacts/data"),
							new String[] { "data1", "mimetype" },
							"raw_contact_id = ?", new String[] { id }, null);
					HashMap<String, String> hashMap = new HashMap<String, String>();
					// 5,游标向下移动获取数据过程
					while (indexCursor.moveToNext()) {
						String data = indexCursor.getString(0);
						String type = indexCursor.getString(1);

						// Log.i(tag, "data = "+data);
						// Log.i(tag, "type = "+type);

						if (type.equals("vnd.android.cursor.item/phone_v2")) {
							// data就为电话号码
							hashMap.put("phone", data);
						} else if (type.equals("vnd.android.cursor.item/name")) {
							// data 为联系人名字
							hashMap.put("name", data);
						}
					}
					indexCursor.close();
					mContactList.add(hashMap);
				}
				cursor.close();
				// 告知主线程集合中的数据以及准备完毕,可以让主线程去使用此集合,填充数据适配器
				mHandler.sendEmptyMessage(0);
			};
		}.start();
	}

	private void initUI() {
		lv_contact_list = (ListView) findViewById(R.id.lv_contact_list);
		lv_contact_list.setOnItemClickListener(new OnItemClickListener() {
			@Override
			public void onItemClick(AdapterView<?> parent, View view,
					int position, long id) {
				// 1,position点中条目的索引值,集合的索引值
				String phone = mContactList.get(position).get("phone");
				// 2,将此电话号码传递给前一个界面
				Intent intent = new Intent();
				intent.putExtra("phone", phone);
				setResult(0, intent);
				// 3,关闭此界面
				finish();
			}
		});
	}

	class MyAdapter extends BaseAdapter {
		@Override
		public int getCount() {
			return mContactList.size();
		}

		@Override
		public HashMap<String, String> getItem(int position) {
			return mContactList.get(position);
		}

		@Override
		public long getItemId(int position) {
			return position;
		}

		@Override
		public View getView(int position, View convertView, ViewGroup parent) {

			Holder holder;
			if (convertView == null) {
				holder = new Holder();
				// 1,生成当前listview一个条目相应的view对象
				convertView = View.inflate(getApplicationContext(),
						R.layout.list_item_contact, null);
				// 2,找到view中的控件
				holder.tv_name = (TextView) convertView
						.findViewById(R.id.tv_name);
				holder.tv_phone = (TextView) convertView
						.findViewById(R.id.tv_phone);
				convertView.setTag(holder);

			} else {
				holder = (Holder) convertView.getTag();
			}

			// 3,给控件赋值
			holder.tv_name.setText(getItem(position).get("name"));
			holder.tv_phone.setText(getItem(position).get("phone"));

			return convertView;
		}
	}

	class Holder {

		public TextView tv_name;
		public TextView tv_phone;
	}
}
```
##2. ListView 显示布局如下
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
   
    <ListView 
        android:id="@+id/lv_contact_list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
##3. item 布局如下：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="10dp"
    android:orientation="vertical" >
    <TextView 
        android:text="联系人名称"
        android:id="@+id/tv_name"
        android:textSize="18sp"
        android:textColor="@color/black"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <TextView 
        android:text="联系人电话号码"
        android:id="@+id/tv_phone"
        android:textSize="18sp"
        android:textColor="@color/grey"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</LinearLayout>
```

**注意：**
 获取联系人需要申请权限
```
 <!-- 读取联系人的权限 -->
    <uses-permission android:name="android.permission.READ_CONTACTS" />
```
至此，已经可以获取并显示联系人信息。

# 五、获取短信内容的实现方法

短信内容数据也是`Android `系统提供的，获取方法如下：

## 1.短信内容获取方法
```
public class MmsListActivity extends Activity {
	private ContentResolver resolver;
	private ListView listView;
	private static final String SMS_URI = "content://sms";
	private Cursor cursor;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_mms_list);
		listView = (ListView) findViewById(R.id.lv_mms);
		resolver = getContentResolver();

	}

	public void GetMMSBtn(View view) {

		// 插入数据
		ContentValues values = new ContentValues();
		values.put("address", "136259");
		values.put("body", "测试数据中。。。。。");
		resolver.insert(Uri.parse(SMS_URI), values);
		// 查询数据方法
		cursor = resolver.query(Uri.parse(SMS_URI), null, null, null, null);
		// 将数据显示到ListView中
		listView.setAdapter(new MyAdapter(MmsListActivity.this, cursor,
				CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER));

	}

	@Override
	protected void onDestroy() {
		super.onDestroy();
		if (cursor != null) {
			// 关闭cursor
			// cursor.close();
		}
	}

	class MyAdapter extends CursorAdapter {

		public MyAdapter(Context context, Cursor c, int flags) {
			super(context, c, flags);
		}

		// 创建一个视图，引入listview要展示的子视图
		@Override
		public View newView(Context context, Cursor cursor, ViewGroup parent) {
			return getLayoutInflater().inflate(R.layout.list_item_mms, null);
		}

		// 绑定数据的方法
		@Override
		public void bindView(View view, Context context, Cursor cursor) {

			TextView tvNumber = (TextView) view.findViewById(R.id.tv_number);
			TextView tvContent = (TextView) view.findViewById(R.id.tv_content);
			TextView tvState = (TextView) view.findViewById(R.id.tv_state);
			TextView tvDate = (TextView) view.findViewById(R.id.tv_date);
			TextView tvId = (TextView) view.findViewById(R.id.tv_id);
			TextView tvRead = (TextView) view.findViewById(R.id.tv_read);

			String number = cursor.getString(cursor.getColumnIndex("address"));
			String body = cursor.getString(cursor.getColumnIndex("body"));
			String date = cursor.getString(cursor.getColumnIndex("date"));
			int read = cursor.getInt(cursor.getColumnIndex("read"));
			int id = cursor.getInt(cursor.getColumnIndex("_id"));
			int type = cursor.getInt(cursor.getColumnIndex("type"));

			if (read == 0) {
				tvRead.setText("短信状态：未读");
			} else {
				tvRead.setText("短信状态：已读");
			}

			tvNumber.setText("手机号：" + number);
			tvContent.setText("短信内容：" + body);
			tvDate.setText("接收短信时间:" + date);
			tvId.setText("短信Id：" + id);

			if (type == 1) {
				tvState.setText("短信状态：已接收");

			} else {
				tvState.setText("短信状态：已发送");
			}
		}
	}

}

```

##2.ListView 布局如下
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >
    
  <ListView
        android:id="@+id/lv_mms"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
    
</LinearLayout>
```

##3.item 布局如下：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/tv_number"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ddd" />

    <TextView
        android:id="@+id/tv_content"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"

        android:text="ddd" />

    <TextView
        android:id="@+id/tv_state"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ddd" />
    <TextView
        android:id="@+id/tv_date"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ddd" />
    <TextView
        android:id="@+id/tv_id"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ddd" />
    <TextView
        android:id="@+id/tv_read"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="ddd" />
</LinearLayout>
```
 # 六、ContentResolver 内容解析者

`ContentResolver `主要是通过`URI`调用`getContentResolver()`获取`ContentProvider` 提供的数据接口，进而进行增删改查等操作。
`ContentResolver ` 使用举例如下：

```
// 查询
	Cursor cursor = getContentResolver().query(Uri.parse(uri), null, null, null, null);
// 插入数据到指定 URI 中
 getContentResolver().insert(Uri.parse(uri), ContentValues);
```

# 七、ContentObserver 内容观察者

`ContentObserver` 内容观察者通过指定`URI` 监听`ContentProvider`数据是否改变。

下面介绍自定义 `ContentObserver`内容观察者

## 1.注册ContentObserver 内容观察者   
    
```
	/**
	 * 监听ContentProvider数据库变化
	 */
	private void ContentObserverDatabase() {
		// [1]注册内容观察者
		Uri uri = Uri.parse("content://ProgramAndroid/person");
		// false 观察的uri 必须是一个确切的uri 如果是true
		getContentResolver().registerContentObserver(uri, true,
				new CustomContentObserver(new Handler()));
	}
```

## 2.继承 ContentObserver 实现 onChange方法

```
public class CustomContentObserver extends ContentObserver {

	/**
	 * @param handler
	 */
	public CustomContentObserver(Handler handler) {
		super(handler);
		// TODO Auto-generated constructor stub
	}

	// 当我们观察的uri发生改变的时候调用
	@Override
	public void onChange(boolean selfChange) {
		System.out.println(" 数据库被操作了 ");

		super.onChange(selfChange);
	}

}
```

至此自定义内容观察者已经实现完成

## 3.调用ContentObserver 监听短信数据改变

```
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //[1]注册一个内容观察者
        Uri uri = Uri.parse("content://sms/");
        getContentResolver().registerContentObserver(uri, true, new MyContentObserver(new Handler()));

    }

    private class MyContentObserver extends ContentObserver{

        public MyContentObserver(Handler handler) {
            super(handler);
        }
        //当观察的内容发生改变的时候调用
        @Override
        public void onChange(boolean selfChange) {
            System.out.println(" 短信的数据库发生了改变");
            super.onChange(selfChange);
        }

    }

```
# 八、ContentProvider ContentResolver ContentObserver 三者关系
- 三者关系图如下

![关系图.png](http://upload-images.jianshu.io/upload_images/5851256-c0fbde40f9ded955.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

