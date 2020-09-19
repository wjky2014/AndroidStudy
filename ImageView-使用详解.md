
##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、ImageView 的继承关系
>二、ImageView 常用方法
>三、ImageView 背景 间距属性设置
>四、使用Bitmap 类型动态设置ImageView 资源
>五、ImageView 图片倒影实现
>六、ImageView 图片缩放实现
>七、ImageView 圆角图片实现
>八、Bitmap 与Drawable 转换工具类

# 一、ImageView 的继承关系

`ImageView `的继承关系 如下：
```
java.lang.Object   
       ↳ android.view.View      
                 ↳ android.widget.ImageView

```
# 二、ImageView 常用方法

`ImageView `主要用于显示图像资源，`Bitmap `或`Drawable`资源，同时也常用于图片渲染调色，图片缩放剪裁等。

以下`XML`代码段是使用`ImageView`显示图像资源的常见示例：

## 1. 在`xml `使用`ImageView` 控件

```
 <LinearLayout
     xmlns:android="http://schemas.android.com/apk/res/android"
     android:layout_width="match_parent"
     android:layout_height="match_parent">
     <ImageView
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:src="@mipmap/ic_launcher"
         />
 </LinearLayout>
 
```

# 三、 ImageView 背景 间距属性设置

##1. 在`xml `使用`ImageView `控件
```
    <ImageView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/grey"
        android:padding="5dp" 
        android:src="@drawable/ic_launcher" />
```
## 2. 实现效果如下：

![padding background 属性设置 ](https://upload-images.jianshu.io/upload_images/5851256-cf09520a299b7b43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、 使用Bitmap 类型动态设置ImageView 资源

## 1. 在`xml` 使用`ImageView `控件

```
<ImageView
        android:id="@+id/img_1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="5dp" />
```
## 2.`java` 类实现

```
		// 1.从资源中获取Bitmap
		ImageView mImageView1 = (ImageView) findViewById(R.id.img_1);
		DrawableUtils.UseBitmap(this, mImageView1, R.drawable.gril);
```
##  3.`DrawableUtils `类方法实现
```

	// 1.从资源中获取Bitmap
	public static void UseBitmap(Context context, ImageView imageView, int drawableId) {

		Bitmap bitmap = BitmapFactory.decodeResource(context.getResources(),
				drawableId);
		imageView.setImageBitmap(bitmap);

	}
```
## 4. 实现效果如下：

![bitmap 类型的图片](https://upload-images.jianshu.io/upload_images/5851256-bec3c34291e12e42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 五、ImageView 图片倒影实现

## 1. 在`xml `使用`ImageView` 控件

```
    <ImageView
        android:id="@+id/img_4"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:background="@color/grey"
        android:padding="5dp" />
```

## 2. `java `代码 实现效果

```
		// 4.倒影图片
		ImageView mImageView4 = (ImageView) findViewById(R.id.img_4);
		mImageView4.setImageBitmap(DrawableUtils.CreateReflectionImageWithOrigin(
				DrawableUtils.DrawableToBitmap(getResources().getDrawable(
						R.drawable.img1))));
```
## 3. `DrawableUtils` 工具类的方法实现

```
	// 5. Drawable----> Bitmap
	public static Bitmap DrawableToBitmap(Drawable drawable) {

		// 获取 drawable 长宽
		int width = drawable.getIntrinsicWidth();
		int heigh = drawable.getIntrinsicHeight();

		drawable.setBounds(0, 0, width, heigh);

		// 获取drawable的颜色格式
		Bitmap.Config config = drawable.getOpacity() != PixelFormat.OPAQUE ? Bitmap.Config.ARGB_8888
				: Bitmap.Config.RGB_565;
		// 创建bitmap
		Bitmap bitmap = Bitmap.createBitmap(width, heigh, config);
		// 创建bitmap画布
		Canvas canvas = new Canvas(bitmap);
		// 将drawable 内容画到画布中
		drawable.draw(canvas);
		return bitmap;
	}
```
##4. 实现效果如下：

![ImageView 倒影功能实现](https://upload-images.jianshu.io/upload_images/5851256-68d1302110676685.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 六、ImageView 图片缩放实现

##1. 在`xml `使用`ImageView` 控件

```
    <ImageView
        android:id="@+id/img_2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="5dp" />
```

##2. `java `代码 实现效果
```
		// 2. 图片缩放
		ImageView mImageView2 = (ImageView) findViewById(R.id.img_2);
		mImageView2.setImageDrawable(DrawableUtils.ZoomDrawable(getResources().getDrawable(R.drawable.img1),
				240, 200));
```
## 3. `DrawableUtils` 工具类方法实现

```
	// 9. drawable进行缩放 ---> bitmap 然后比对bitmap进行缩放
	public static Drawable ZoomDrawable(Drawable drawable, int w, int h) {
		int width = drawable.getIntrinsicWidth();
		int height = drawable.getIntrinsicHeight();
		// 调用5 中 drawable转换成bitmap
		Bitmap oldbmp = DrawableToBitmap(drawable);

		// 创建操作图片用的Matrix对象
		Matrix matrix = new Matrix();
		// 计算缩放比例
		float sx = ((float) w / width);
		float sy = ((float) h / height);
		// 设置缩放比例
		matrix.postScale(sx, sy);
		// 建立新的bitmap，其内容是对原bitmap的缩放后的图
		Bitmap newbmp = Bitmap.createBitmap(oldbmp, 0, 0, width, height,
				matrix, true);
		return new BitmapDrawable(newbmp);
	}

```
## 4. 实现效果如下：

![Imageview 缩放](https://upload-images.jianshu.io/upload_images/5851256-50831f413163001f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 七、ImageView 圆角图片 实现


## 1. 在`xml `使用`ImageView` 控件
```
    <ImageView
        android:id="@+id/img_3"
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:background="@color/grey"
        android:padding="5dp" />
```
## 2.` java `代码 实现效果

```
		// 3. 圆角图片
		ImageView mImageView3 = (ImageView) findViewById(R.id.img_3);
		mImageView3.setImageBitmap(DrawableUtils.SetRoundCornerBitmap(
				DrawableUtils.DrawableToBitmap(getResources().getDrawable(
						R.drawable.img1)), 60));
```
## 3. `DrawableUtils `工具类方法实现

```
	// 6.圆角图片
	public static Bitmap SetRoundCornerBitmap(Bitmap bitmap, float roundPx) {
		int width = bitmap.getWidth();
		int heigh = bitmap.getHeight();
		// 创建输出bitmap对象
		Bitmap outmap = Bitmap.createBitmap(width, heigh,
				Bitmap.Config.ARGB_8888);
		Canvas canvas = new Canvas(outmap);
		final int color = 0xff424242;
		final Paint paint = new Paint();
		final Rect rect = new Rect(0, 0, width, heigh);
		final RectF rectf = new RectF(rect);
		paint.setAntiAlias(true);
		canvas.drawARGB(0, 0, 0, 0);
		paint.setColor(color);
		canvas.drawRoundRect(rectf, roundPx, roundPx, paint);
		paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
		canvas.drawBitmap(bitmap, rect, rect, paint);

		return outmap;
	}
```
## 4. 实现效果如下：

![圆角图片实现](https://upload-images.jianshu.io/upload_images/5851256-2573845d4365932e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 八、`Bitmap `与`Drawable` 转换工具类

`Bitmap `与`Drawable` 转换常用工具类源代码如下：

```

public class DrawableUtils {

	// 1.从资源中获取Bitmap
	public static void UseBitmap(Context context, ImageView imageView, int drawableId) {

		Bitmap bitmap = BitmapFactory.decodeResource(context.getResources(),
				drawableId);
		imageView.setImageBitmap(bitmap);

	}

	// 2.Bitmap ---> byte[]
	public byte[] BitmapToBytes(Bitmap bitmap) {
		ByteArrayOutputStream baos = new ByteArrayOutputStream();
		bitmap.compress(Bitmap.CompressFormat.PNG, 100, baos);
		return baos.toByteArray();
	}

	// 3.byte[] ---->bitmap
	public Bitmap BytesToBitmap(byte[] b) {
		if (b.length != 0) {
			return BitmapFactory.decodeByteArray(b, 0, b.length);
		} else {
			return null;
		}
	}

	// 4.Bitmap 缩放方法
	public static Bitmap ZoomBitmap(Bitmap bitmap, int width, int heigh) {
		int w = bitmap.getWidth();
		int h = bitmap.getHeight();
		Matrix matrix = new Matrix();
		float scalewidth = (float) width / w;
		float scaleheigh = (float) heigh / h;
		matrix.postScale(scalewidth, scaleheigh);
		Bitmap newBmp = Bitmap.createBitmap(bitmap, 0, 0, w, h, matrix, true);
		return newBmp;

	}

	// 5. Drawable----> Bitmap
	public static Bitmap DrawableToBitmap(Drawable drawable) {

		// 获取 drawable 长宽
		int width = drawable.getIntrinsicWidth();
		int heigh = drawable.getIntrinsicHeight();

		drawable.setBounds(0, 0, width, heigh);

		// 获取drawable的颜色格式
		Bitmap.Config config = drawable.getOpacity() != PixelFormat.OPAQUE ? Bitmap.Config.ARGB_8888
				: Bitmap.Config.RGB_565;
		// 创建bitmap
		Bitmap bitmap = Bitmap.createBitmap(width, heigh, config);
		// 创建bitmap画布
		Canvas canvas = new Canvas(bitmap);
		// 将drawable 内容画到画布中
		drawable.draw(canvas);
		return bitmap;
	}

	// 6.圆角图片
	public static Bitmap SetRoundCornerBitmap(Bitmap bitmap, float roundPx) {
		int width = bitmap.getWidth();
		int heigh = bitmap.getHeight();
		// 创建输出bitmap对象
		Bitmap outmap = Bitmap.createBitmap(width, heigh,
				Bitmap.Config.ARGB_8888);
		Canvas canvas = new Canvas(outmap);
		final int color = 0xff424242;
		final Paint paint = new Paint();
		final Rect rect = new Rect(0, 0, width, heigh);
		final RectF rectf = new RectF(rect);
		paint.setAntiAlias(true);
		canvas.drawARGB(0, 0, 0, 0);
		paint.setColor(color);
		canvas.drawRoundRect(rectf, roundPx, roundPx, paint);
		paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
		canvas.drawBitmap(bitmap, rect, rect, paint);

		return outmap;
	}

	// 7.获取带倒影的图片
	public static Bitmap CreateReflectionImageWithOrigin(Bitmap bitmap) {

		final int reflectionGapLine = 4;
		int w = bitmap.getWidth();
		int h = bitmap.getHeight();
		Matrix matrix = new Matrix();
		matrix.preScale(1, -1);

		Bitmap reflectionImage = Bitmap.createBitmap(bitmap, 0, h / 2, w,
				h / 2, matrix, false);

		Bitmap bitmapWithReflection = Bitmap.createBitmap(w, (h + h / 2),
				Bitmap.Config.ARGB_8888);
		Canvas canvas = new Canvas(bitmapWithReflection);
		canvas.drawBitmap(bitmap, 0, 0, null);
		Paint deafalutPaint = new Paint();
		canvas.drawRect(0, h, w, h + reflectionGapLine, deafalutPaint);

		canvas.drawBitmap(reflectionImage, 0, h + reflectionGapLine, null);

		Paint paint = new Paint();
		LinearGradient shader = new LinearGradient(0, bitmap.getHeight(), 0,
				bitmapWithReflection.getHeight() + reflectionGapLine, 0x70ffffff,
				0x00ffffff, Shader.TileMode.CLAMP);
		paint.setShader(shader);
		// Set the Transfer mode to be porter duff and destination in
		paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
		// Draw a rectangle using the paint with our linear gradient
		canvas.drawRect(0, h, w, bitmapWithReflection.getHeight()
				+ reflectionGapLine, paint);
		return bitmapWithReflection;
	}

	// 8. bitmap ---Drawable
	public static Drawable BitmapToDrawable(Bitmap bitmap, Context context) {
		BitmapDrawable drawbale = new BitmapDrawable(context.getResources(),
				bitmap);
		return drawbale;
	}

	// 9. drawable进行缩放 ---> bitmap 然后比对bitmap进行缩放
	public static Drawable ZoomDrawable(Drawable drawable, int w, int h) {
		int width = drawable.getIntrinsicWidth();
		int height = drawable.getIntrinsicHeight();
		// 调用5 中 drawable转换成bitmap
		Bitmap oldbmp = DrawableToBitmap(drawable);

		// 创建操作图片用的Matrix对象
		Matrix matrix = new Matrix();
		// 计算缩放比例
		float sx = ((float) w / width);
		float sy = ((float) h / height);
		// 设置缩放比例
		matrix.postScale(sx, sy);
		// 建立新的bitmap，其内容是对原bitmap的缩放后的图
		Bitmap newbmp = Bitmap.createBitmap(oldbmp, 0, 0, width, height,
				matrix, true);
		return new BitmapDrawable(newbmp);
	}

}

```




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
