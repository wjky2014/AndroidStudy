
 
##### 和您一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、Handler 简介
>二、Handler 消息处理机制原理
>三、Handler 机制处理的4个关键对象
>四、 Handler常用方法
> 五、子线程更新UI 异常处理
>六、主线程给子线程发送消息的方法
>七、主线程发送消息给子线程的例子
>八、子线程给主线程发送消息的方法
>九、 主、子 线程 互发消息方法
>十、子线程方法中调用主线程更新UI的方法
>十一、移除Handler 发送的消息方法





# 一、Handler 简介

在了解Handler 之前，我们需要先了解Handler的继承关系 继承关系如下：

```
java.lang.Object
   ↳	android.os.Handler
```


`Handler`是 `Android`中用来更新UI 的一套消息处理机制。`Handler`  允许线程间发送`Message`或`Runnable`对象进行通信。在`Android`中UI修改只能通过`UI Thread`，子线程不能更新`UI`。如果子线程想更新`UI`，需要通过 `Handler `发送消息给主线程，进而达到更新`UI`的目的。

# 二、Handler 消息处理机制原理

当`Android` 应用程序创建的时候，系统会给每一个进程提供一个`Looper `，`Looper` 是一个死循环，它内部维护一个消息队列，`Looper `不停的从消息队列中取`Message`，取到的消息就发送给`handler`，最后`Handler` 根据接收的消息去修改`UI`等。

# 三、Handler 机制处理的4个关键对象

##  1.Message

线程之间传递的消息，可以携带一些简单的数据供子线程与主线程进行交换数据。

## 2.Message Queue

存放通过`Handler` 发送的` Message` 的消息队列，每一个线程只有一个消息队列。

##  3.Handler

消息处理者，主要用于发送跟处理消息。

主要功能： 
发送消息`SendMessage()`
处理消息 `HandleMessage()`

##  4.Looper 

内部包含一个死循环的`MessageQueue`，用于存储`handler `发送的`Message`，`Looper`则是不断的从消息队列中取消，如果有消息就取出发送给`Handler` 处理，没有则阻塞。

##  5.总结：

`Handler `负责发送`Message`到`Message Queue`，`Looper`负责从`Message Queue` 遍历`Message` ,然后直接把遍历的消息回传给`Handler `自己，通过`Handler `自身的`handleMessage`处理更新`UI`等操作。

![主线程、子线程间通信简单流程](http://upload-images.jianshu.io/upload_images/5851256-dfc1b5ecf2d4b4aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、 Handler常用方法

##  1.Runnable对象

- post(Runnable)

使用方法举例：
```
	public void BtnRunnableMethod(View view) {
		// 1.Runnable 对象
		RunnableHandlderMethod();
	}

	/**
	 * Runnable 对象更新 UI 
	 * **/
	private Handler mRunnableHandler = new Handler();
	public void RunnableHandlderMethod() {
		new Thread() {
			@Override
			public void run() {
				try {
					Thread.sleep(1000);

					mRunnableHandler.post(new Runnable() {
						@Override
						public void run() {
							((Button) findViewById(R.id.btn_runnable))
									.setText("Runnable");
						}
					});

				} catch (InterruptedException e) {
					e.printStackTrace();
				}

			}
		}.start();
	}

```

- postAtTime(Runnable, long) 
- postDelayed(Runnable, long) 

## 2. Message 对象

- sendEmptyMessage(int)

使用方法举例：

```
	public void BtnMessageThreadMethod(View view) {
		// 2.Message 对象
		new MessageHandlerThreadMethod("子线程不能更新UI").start();
	}
	/**
	 * Message 对象举例
	 * ***/
	private int mCount = 0;
	private Handler mMessageHandler = new Handler() {
		@Override
		public void handleMessage(Message msg) {
			super.handleMessage(msg);

			((Button) findViewById(R.id.btn_thread)).setText("" + mCount);
		}
	};

	class MessageHandlerThreadMethod extends Thread {

		String mString;

		public MessageHandlerThreadMethod(String str) {
			mString = str;
		}

		@Override
		public void run() {
			for (int i = 0; i < 10; i++) {
				try {
					Thread.sleep(1000);
				} catch (Exception e) {

				}

				mCount++;
				mMessageHandler.sendEmptyMessage(0);
			}

		}
	}
```
- sendMessage(Message)

使用方法举例：

```
public void BtnMessageObjMethod(View view) {
		HandlerMessageObjMethods();
	}

	/***
	 * handler sendmessage 处理方法
	 * **/
	private Handler mHandlerMessageObj = new Handler() {

		@Override
		public void handleMessage(Message msg) {

			((Button) findViewById(R.id.btn_message)).setText("arg1:"
					+ msg.arg1 + "\n" + msg.obj);
		}
	};

	private void HandlerMessageObjMethods() {
		new Thread() {
			@Override
			public void run() {

				try {
					Thread.sleep(1000);
					// Message message = new Message();
					Message message = mHandlerMessageObj.obtainMessage();

					message.arg1 = 100;

					Person person = new Person();
					person.name = "Lucy";
					person.age = 12;

					message.obj = person;
					mHandlerMessageObj.sendMessage(message);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}.start();
	}

	class Person {

		public int age;
		public String name;

		public String toString() {
			return "Name=" + name + "\n Age=" + age;
		}

	}
```

- sendMessageAtTime(Message, long),
- sendMessageDelayed(Message, long) 

##  3.接收、处理Message 

- handleMessage(Message) 

使用方法举例：
```
	private Handler mMessageHandler = new Handler() {
		@Override
		public void handleMessage(Message msg) {
			super.handleMessage(msg);

			((Button) findViewById(R.id.btn_thread)).setText("" + mCount);
		}
	};
```
# 五、子线程更新UI 异常处理

子线程不能更新`UI`，如果在子线程中更新`UI`，会出现`CalledFromWrongThreadException` 异常。

- CalledFromWrongThreadException

![CalledFromWrongThreadException 子线程不能更新UI ](http://upload-images.jianshu.io/upload_images/5851256-07c6bfaacf607a17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###  解决方法：
子线程通过`Handler `发送消息给主线程，让主线程处理消息，进而更新`UI`。

# 六、主线程给子线程发送消息的方法

此例子中子线程通过`Looper`不断遍历主线程发送的消息，`Looper `使用方法如下：

## 1. 准备Looper 轮询器

` Looper.prepare(); `

##  2. Handler 处理遍历消息

`Handler mHandler = new Handler()`

##  3. 遍历消息队列

`   Looper.loop();`

##  4.`Looper` 使用方法如下：
```
	// 自定义 Loop 线程 ---> 不停的处理主线程发的消息
	class ChildLooperThread extends Thread {
		@Override
		public void run() {
			// 1.准备成为loop线程
			Looper.prepare();
			// 2.处理消息
			mMainHandler = new Handler() {
				// 处理消息
				public void handleMessage(Message msg) {
					super.handleMessage(msg);
					        ... ...
						}
					});
				}
			};
			// 3.Loop循环方法
			Looper.loop();
		}

	}
```

# 七、主线程发送消息给子线程的例子

## 1. 启动 子线程，并再启动后发送消息
```

	public void BtnMainMessageMethod(View view) {
		// 点击主线程 按钮，启动子线程，并在子线程启动后发送消息
		Message msg = new Message();
		msg.obj = "主线程：这是我携带的信息";
		if (mMainHandler != null) {
			// 2.主线程发送消息
			mMainHandler.sendMessage(msg);
		} else {
			Toast.makeText(getApplicationContext(), "开启子线程轮询消息，请再次点击发送消息",
					Toast.LENGTH_SHORT).show();
			// 1.开启轮询线程，不断等待接收主线成消息
			new ChildLooperThread().start();
		}
	}
```
##2. 子线程启动，不停的变量主线程发送的消息

```
	private Handler mMainHandler;
	String mMainMessage;

	// 自定义 Loop 线程 ---> 不停的处理主线程发的消息
	class ChildLooperThread extends Thread {
		@Override
		public void run() {
			// 1.准备成为loop线程
			Looper.prepare();
			// 2.处理消息
			mMainHandler = new Handler() {
				// 处理消息
				public void handleMessage(Message msg) {
					super.handleMessage(msg);
					mMainMessage = (String) msg.obj;
					Log.i("TAG", "子线程：从主线程中接受的消息为：\n" + mMainMessage);
					// 使用 runOnUiThread 在主线程中更新UI
					runOnUiThread(new Runnable() {
						@Override
						public void run() {
							((Button) findViewById(R.id.btn_main_message))
									.setText(mMainMessage);
						}
					});
				}
			};
			// 3.Loop循环方法
			Looper.loop();
		}

	}
```

# 八、子线程给主线程发送消息的方法

##1.子线程发送消息给主线程方法
```

	public void BtnChildMessageMethod(View view) {

		new Thread() {
			public void run() {
				while (mCount < 100) {
					mCount++;
					try {
						Thread.sleep(100);
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
					/**
					 * 利用handler 对象发送消息 Message msg=Message.obtain(); Message
					 * msg=new Message(); 获取一个消息对象message
					 * */
					Message msg = Message.obtain();
					// 消息标记
					msg.what = 1;
					// 传递整型值msg.obj="传递object数据"
					msg.arg1 = mCount;
					Log.i("TAG", "count 值=" + mCount);
					if (mhandler != null) {
						mhandler.sendMessage(msg);
					}
				}
			}
		}.start();
	}
```

##2.主线程接收并处理消息的方法
```
// 定义一个handler 主线程 接收子线程发来的信息
	private Handler mhandler = new Handler() {
		// 處理消息的方法
		public void handleMessage(android.os.Message msg) {

			switch (msg.what) {
			case 1:
				int value = msg.arg1;
				Log.i("TAG", "value值=" + value);
				((Button) findViewById(R.id.btn_child_message)).setText("当前值="
						+ value);
				break;

			default:
				break;
			}
		}

	};
```

#九、 主、子 线程 互发消息方法
主要实现主、子线程每隔`1s`中通信一次
 - 实现打印`Log`如下：

![主、子线程通信log信息](http://upload-images.jianshu.io/upload_images/5851256-8a2b63b8486ddd1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 实现方法如下：

##1. 启动子线程并发送给主线程消息

```
	public void BtnMainChildMessageMethod(View view) {

		// 创建 名称为currentThread 子线程
		HandlerThread mChildThread = new HandlerThread("ChildThread");
		mChildThread.start();
		mChildHandler = new Handler(mChildThread.getLooper()) {
			@Override
			public void handleMessage(Message msg) {

				Log.i("TAG", "主线程对我说：" + msg.obj);

				// 子线程携带的消息
				Message message = new Message();
				message.obj = Thread.currentThread() + "我是子线程，小样，让我听你的没门";
				// 向主线程发送消息
				mainhandler.sendMessageDelayed(message, 1000);
			}
		};
		// 主线成发送空消息，开启通信
		mainhandler.sendEmptyMessage(1);
	}
```

##2.主线程接收并处理子线程发送的消息
```
	// 创建主线程
	private Handler mainhandler = new Handler() {
		@Override
		public void handleMessage(Message msg) {
			Log.i("TAG", "子线程对我说：" + msg.obj);

			// 主线成携带的消息内容
			Message message = new Message();
			message.obj = Thread.currentThread() + "我是主线程：小子你得听我的。";

			// 向子线程发送消息
			mChildHandler.sendMessageDelayed(message, 1000);
		}
	};

```

# 十、子线程方法中调用主线程更新UI的方法

##   1.Activity 中 可以使用 runOnUiThread(Runnable)

```
					// 使用 runOnUiThread 在主线程中更新UI
					runOnUiThread(new Runnable() {
						@Override
						public void run() {
							((Button) findViewById(R.id.btn_main_message))
									.setText(mMainMessage);
						}
					});
```

## 2.子线程使用 Handler.post(Runnable)

```

					mRunnableHandler.post(new Runnable() {
						@Override
						public void run() {
							((Button) findViewById(R.id.btn_runnable))
									.setText("Runnable");
						}
					});
```

##  3.View.post()

```
							((Button) findViewById(R.id.btn_runnable)).post(new Runnable() {
								
								@Override
								public void run() {
									// TODO Auto-generated method stub
									((Button) findViewById(R.id.btn_runnable)).setText("View.post()方法使用");
								}
							});
```

## 4.Handler.sendMessage(Message)

```
	public void BtnMainMessageMethod(View view) {
		// 点击主线程 按钮，启动子线程，并在子线程启动后发送消息
		Message msg = new Message();
		msg.obj = "主线程：这是我携带的信息";
		if (mMainHandler != null) {
			// 2.主线程发送消息
			mMainHandler.sendMessage(msg);
			}
}
```
# 十一、移除Handler 发送的消息方法
## 1.移除 handler 发送的所有消息
```
private Handler mChildHandler;
mChildHandler.removeCallbacksAndMessages(null);
```
## 2.移除 指定消息
```
private Handler mainhandler;
mainhandler.removeMessages(what);
```




**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
