 #####  和你一起终身学习，这里是程序员Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、加强APP 安全沟通
>1. 建议显示使用应用选择器
>2. 应用共享数据时，建议使用 签名权限
>3. 禁止其他应用访问自己的ContentProvider
>4. 获取敏感信息，需要提前询问凭据
>5. 提高应用网络安全措施
>6. 创建自己的信任管理器
>7. 谨慎使用 WebView 对象
>8. 使用HTML消息通道

>二、提供正确的权限permission
>1. 使用Intent 进行权限申请
>2. 跨应用 安全访问数据

>三、安全的进行数据存储
>1. 将私有数据存储在内部存储中
>2. 谨慎使用外部存储
>3. 检查数据的有效性
>4. 仅将非敏感数据存储在缓存文件中
>5. 请在私有模式SharedPreferences

>四、使用安全的三方服务
>1. 使用 Google Play服务
>2. 更新所有应用依赖项

通过提高应用程序的安全性，您可以帮助保持用户信任和设备完整性。

此文主要介绍了几种对您的应用安全性产生重大积极影响的最佳做法。


#一、加强APP 安全沟通

当您保护在应用程序与其他应用程序之间或应用程序与网站之间交换的数据时，可以提高应用程序的稳定性并保护您发送和接收的数据。


 
###1. 建议显示使用应用选择器

如果隐式意图可以在用户的​​设备上启动至少两个可能的应用程序，请明确显示应用程序选择器。此交互策略允许用户将敏感信息传输到他们信任的应用程序。
举例如下：
```
Intent intent = new Intent(Intent.ACTION_SEND);
List<ResolveInfo> possibleActivitiesList =
        queryIntentActivities(intent, PackageManager.MATCH_ALL);

// Verify that an activity in at least two apps on the user's device
// can handle the intent. Otherwise, start the intent only if an app
// on the user's device can handle the intent.
if (possibleActivitiesList.size() > 1) {

    // Create intent to show chooser.
    // Title is something similar to "Share this photo with".

    String title = getResources().getString(R.string.chooser_title);
    Intent chooser = Intent.createChooser(intent, title);
    startActivity(chooser);
} else if (intent.resolveActivity(getPackageManager()) != null) {
    startActivity(intent);
}
```

###2. 应用共享数据时，建议使用 签名权限 

在您控制或拥有的两个应用程序之间共享数据时，请使用 基于签名的权限。这些权限不需要用户确认，而是检查访问数据的应用程序是否使用相同的签名密钥进行签名。因此，这些权限提供了更简化，安全的用户体验。
举例如下：
```
<manifest xmlns：android = “http://schemas.android.com/apk/res/android” 
    package = “com.example.myapp” >
    <permission android：name = “my_custom_permission_name” android：protectionLevel = “signature” /> 
                
```

###3. 禁止其他应用访问自己的ContentProvider

除非您打算将数据从应用程序发送到您不拥有的其他应用程序，否则您应明确禁止其他开发人员的应用程序访问`ContentProvider`您的应用程序包含的对象。如果您的应用可以安装在运行`Android 4.1.1（API级别16）`或更低级别的设备上，则此设置尤其重要，因为 `默认情况下`，这些`Android`版本`android:exported `的`<provider> `元素属性` true`。

举例如下：
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <application ... >
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.example.myapp.fileprovider"
            ...
            android:exported="false">
            <!-- Place child elements of <provider> here. -->
        </provider>
        ...
    </application>
</manifest>
```
###4.获取敏感信息，需要提前询问凭据

在向用户请求凭据以便他们可以访问应用中的敏感信息或高级内容时，请询问`PIN/Password/Pattern `或 生物识别凭证，例如指纹，人脸识别、虹膜等。

本节重点介绍推荐的生物识别登录方法。

所述`生物识别库 指纹等`显示系统提示，要求用户登录使用的生物统计凭证。对话框外观的最终一致性创建了更可靠的用户体验。图1中显示了一个示例对话框。

**注意：**生物识别库扩展了其功能 [`FingerprintManager`](https://developer.android.google.cn/reference/android/hardware/fingerprint/FingerprintManager)。

![](https://upload-images.jianshu.io/upload_images/5851256-4ebd0b216ba516c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**要为您的应用提供生物识别身份验证，请完成以下步骤：**
- 1. 在`app/build.gradle` 中添加以下代码
```
dependencies {
    implementation 'androidx.biometric:biometric:1.0.0-alpha04'
}
```
- 2. 在创建承载生物识别登录对话框的`activity`或`fragment`时，使用以下代码段中显示的逻辑显示`dialog`： 

```
private Handler handler = new Handler();

private Executor executor = new Executor() {
    @Override
    public void execute(Runnable command) {
        handler.post(command);
    }
};

@Override
protected void onCreate(Bundle savedInstanceState) {
    // ...
    // Prompt appears when user clicks "Log in"
    Button biometricLoginButton = findViewById(R.id.biometric_login);
    biometricLoginButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            showBiometricPrompt();
        }
    });
}

private void showBiometricPrompt() {
    BiometricPrompt.PromptInfo promptInfo =
            new BiometricPrompt.PromptInfo.Builder()
            .setTitle("Biometric login for my app")
            .setSubtitle("Log in using your biometric credential")
            .setNegativeButtonText("Cancel")
            .build();

    BiometricPrompt biometricPrompt = new BiometricPrompt(MainActivity.this,
            executor, new BiometricPrompt.AuthenticationCallback() {
        @Override
        public void onAuthenticationError(int errorCode,
                @NonNull CharSequence errString) {
            super.onAuthenticationError(errorCode, errString);
            Toast.makeText(getApplicationContext(),
                "Authentication error: " + errString, Toast.LENGTH_SHORT)
                .show();
        }

        @Override
        public void onAuthenticationSucceeded(
                @NonNull BiometricPrompt.AuthenticationResult result) {
            super.onAuthenticationSucceeded(result);
            BiometricPrompt.CryptoObject authenticatedCryptoObject =
                    result.getCryptoObject();
            // User has verified the signature, cipher, or message
            // authentication code (MAC) associated with the crypto object, so
            // you can use it in your app's crypto-driven workflows.
        }

        @Override
        public void onAuthenticationFailed() {
            super.onAuthenticationFailed();
            Toast.makeText(getApplicationContext(), "Authentication failed",
                Toast.LENGTH_SHORT)
                .show();
        }
    });

    // Displays the "log in" prompt.
    biometricPrompt.authenticate(promptInfo);
}
```
###5. 提高应用网络安全措施

以下部分介绍了如何改善应用程序的网络安全性。

- 1. 使用 SSL traffic

如果您的应用与具有由知名可信`CA`颁发的证书的`Web`服务器通信，则`HTTPS`请求非常简单：

```
URL url = new URL("https://www.google.com");
HttpsURLConnection urlConnection = (HttpsURLConnection) url.openConnection();
urlConnection.connect();
InputStream in = urlConnection.getInputStream();
```

- 2. 添加网络安全配置

如果您的应用使用新的或自定义`CA`，则可以在配置文件中声明网络的安全设置。此过程允许您在不修改任何应用程序代码的情况下创建配置。

要将网络安全配置文件添加到您的应用，请按照下列步骤操作：


- 在AndroidMainfest.xml中声明
```
<manifest ... >
    <application
        android:networkSecurityConfig="@xml/network_security_config"
        ... >
        <!-- Place child elements of <application> element here. -->
    </application>
</manifest>
```

- 添加声明的 安全配置 xml（`res/xml/network_security_config.xml`） 文件

通过禁用明文指定特定域的所有流量都应使用HTTPS：
```
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">secure.example.com</domain>
        ...
    </domain-config>
</network-security-config>
```
在开发过程中，您可以使用该 `<debug-overrides>`元素以`expliticly`方式允许用户安装的证书。在调试和测试期间，此元素会覆盖应用程序的安全关键选项，而不会影响应用程序的发布配置。以下代码段显示了如何在应用程序的网络安全配置`XML`文件中定义此元素：
```
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

###6. 创建自己的信任管理器

您的`SSL`检查程序不应接受每个证书。您可能需要设置信任管理器并处理在以下条件之一适用于您的用例时发生的所有`SSL`警告：

*   您正在与具有由新`CA`或自定义`CA`签名的证书的`Web`服务器进行通信。
*   您正在使用的设备不信任该`CA`.
*   您无法使用[网络安全配置](https://developer.android.google.cn/topic/security/best-practices#network-security-config)。

###7.谨慎使用 WebView 对象

只要有可能，只加载`WebView`对象中的白名单内容 。换句话说，`WebView`应用中的 对象不应允许用户导航到您无法控制的网站。

此外，除非您完全控制并信任应用程序对象中的内容，否则 不应启用` JavaScript`界面​​支持`WebView`。

###8. 使用HTML消息通道

如果您的应用必须在运行`Android 6.0（API级别23）`及更高版本的设备上使用`JavaScript`界面​​支持，请使用`HTML`消息渠道而不是 [evaluateJavascript()](https://developer.android.google.cn/reference/android/webkit/WebView.html#evaluateJavascript(java.lang.String,%20android.webkit.ValueCallback%3Cjava.lang.String%3E))在网站和应用之间进行通信，如以下代码段所示：
```
WebView myWebView = (WebView) findViewById(R.id.webview);

// messagePorts[0] and messagePorts[1] represent the two ports.
// They are already tangled to each other and have been started.
WebMessagePort[] channel = myWebView.createWebMessageChannel();

// Create handler for channel[0] to receive messages.
channel[0].setWebMessageCallback(new WebMessagePort.WebMessageCallback() {
    @Override
    public void onMessage(WebMessagePort port, WebMessage message) {
         Log.d(TAG, "On port " + port + ", received this message: " + message);
    }
});

// Send a message from channel[1] to channel[0].
channel[1].postMessage(new WebMessage("My secure message"));
```

#二、提供正确的权限permission

您的应用只应请求正常运行所需的最少权限。如果可能，您的应用应在不再需要时放弃其中一些权限
### 1. 使用Intent 进行权限申请

只要有可能，请不要向您的应用添加权限，以完成可在其他应用中完成的操作。相反，使用意图将请求推迟到已具有必要权限的其他应用程序。

以下示例显示如何使用`intent`将用户定向到联系人应用程序，而不是请求 `READ_CONTACTS`和` WRITE_CONTACTS`权限：
```
// Delegates the responsibility of creating the contact to a contacts app,
// which has already been granted the appropriate WRITE_CONTACTS permission.
Intent insertContactIntent = new Intent(Intent.ACTION_INSERT);
insertContactIntent.setType(ContactsContract.Contacts.CONTENT_TYPE);

// Make sure that the user has a contacts app installed on their device.
if (insertContactIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(insertContactIntent);
}
```
此外，如果您的应用需要执行基于文件的`I / O`（例如访问存储或选择文件），则不需要特殊权限，因为系统可以代表您的应用完成操作。更好的是，在用户选择特定URI的内容后，调用应用程序将被授予对所选资源的权限。

### 2.跨应用 安全访问数据

请遵循以下最佳做法，以便以更安全的方式与其他应用分享您的应用内容：
*   根据需要强制执行只读或只写权限。
*   通过使用[FLAG_GRANT_READ_URI_PERMISSION](https://developer.android.google.cn/reference/android/content/Intent.html#FLAG_GRANT_READ_URI_PERMISSION)和 [FLAG_GRANT_WRITE_URI_PERMISSION](https://developer.android.google.cn/reference/android/content/Intent.html#FLAG_GRANT_WRITE_URI_PERMISSION)标志为客户端提供一次性数据访问 。
*   共享数据时，请使用`“content：//”URI`，而不是`“file：//”URI`。[FileProvider](https://developer.android.google.cn/reference/androidx/core/content/FileProvider.html) 为您执行此操作的实例 。

以下代码段显示了如何使用`URI`权限授予标志和内容提供程序权限在单独的`PDF Viewer`应用程序中显示应用程序的`PDF`文件：
```
// Create an Intent to launch a PDF viewer for a file owned by this app.
Intent viewPdfIntent = new Intent(Intent.ACTION_VIEW);
viewPdfIntent.setData(Uri.parse("content://com.example/personal-info.pdf"));

// This flag gives the started app read access to the file.
viewPdfIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);

// Make sure that the user has a PDF viewer app installed on their device.
if (viewPdfIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(viewPdfIntent);
}
```


#三、安全的进行数据存储

虽然您的应用可能需要访问敏感用户信息，但只有当用户信任您可以正确保护其数据时，您的用户才会授予您应用访问其数据的权限。

###1.将私有数据存储在内部存储中
将所有私有用户数据存储在设备的内部存储中，该存储是按应用程序沙箱化的。您的应用无需请求查看这些文件的权限，其他应用无法访问这些文件。作为一项额外的安全措施，当用户卸载应用程序时，设备会删除应用程序在内部存储中保存的所有文件。

##### 以下代码段演示了一种将数据写入内部存储的方法：

```
// Creates a file with this name, or replaces an existing file
// that has the same name. Note that the file name cannot contain
// path separators.
final String FILE_NAME = "sensitive_info.txt";
String fileContents = "This is some top-secret information!";

FileOutputStream fos = openFileOutput(FILE_NAME, Context.MODE_PRIVATE);
fos.write(fileContents.getBytes());
fos.close();
```
##### 以下代码段显示了反向操作，从内部存储中读取数据：
```
// The file name cannot contain path separators.
final String FILE_NAME = "sensitive_info.txt";
FileInputStream fis = openFileInput(FILE_NAME);

// available() determines the approximate number of bytes that can be
// read without blocking.
int bytesAvailable = fis.available();
StringBuilder topSecretFileContents = new StringBuilder(bytesAvailable);

// Make sure that read() returns a number of bytes that is equal to the
// file's size.
byte[] fileBuffer = new byte[bytesAvailable];
while (fis.read(fileBuffer) != -1) {
    topSecretFileContents.append(fileBuffer);
}
```
###2.谨慎使用外部存储
默认情况下，Android系统不会对驻留在外部存储中的数据实施安全限制，并且不保证存储介质本身保持与设备的连接。因此，您应该应用以下安全措施来提供对外部存储中信息的安全访问.

##### 使用范围目录访问 
如果您的应用只需要访问设备外部存储中的特定目录，则可以使用 范围目录访问来限制应用对相应设备外部存储的访问。为方便用户，您的应用应保存目录访问URI，以便用户每次尝试访问目录时都不需要批准对目录的访问权限。

**注意：**如果对外部存储中的特定目录使用作用域目录访问，请知道用户可能会在应用程序运行时弹出包含此存储的媒体。您应该包含逻辑以正常处理[Environment.getExternalStorageState()](https://developer.android.google.cn/reference/android/os/Environment.html#getExternalStorageState()), 此用户行为导致的返回值的更改 。

以下代码段使用作用域目录访问与设备主要共享存储中的`pictures`目录：
```
private static final int PICTURES_DIR_ACCESS_REQUEST_CODE = 42;

private void accessExternalPicturesDirectory() {
  StorageManager sm =
          (StorageManager) getSystemService(Context.STORAGE_SERVICE);
  StorageVolume volume = sm.getPrimaryStorageVolume();
  Intent intent =
          volume.createAccessIntent(Environment.DIRECTORY_PICTURES);
  startActivityForResult(intent, PICTURES_DIR_ACCESS_REQUEST_CODE);
}

...

@Override
public void onActivityResult(int requestCode, int resultCode,
        Intent resultData) {
    if (requestCode == PICTURES_DIR_ACCESS_REQUEST_CODE &&
            resultCode == Activity.RESULT_OK) {

        // User approved access to scoped directory.
        if (resultData != null) {
            Uri picturesDirUri = resultData.getData();

            // Save user's approval for accessing this directory
            // in your app.
            ContentResolver myContentResolver = getContentResolver();
            myContentResolver.takePersistableUriPermission(picturesDirUri,
                    Intent.FLAG_GRANT_READ_URI_PERMISSION);
        }
    }
}
```
**警告：**请勿`null`进行 [createAccessIntent()](https://developer.android.google.cn/reference/android/os/storage/StorageVolume.html#createAccessIntent(java.lang.String))不必要的操作，因为这会授予您对应用[StorageManager](https://developer.android.google.cn/reference/android/os/storage/StorageManager.html) 找到的整个卷的应用访问权限 。

### 3.检查数据的有效性

如果您的应用使用外部存储中的数据，请确保数据内容未被破坏或修改。您的应用还应包含处理不再采用稳定格式的文件的逻辑。

以下示例显示了检查文件有效性的权限和逻辑：
```
<manifest ... >
    <!-- Apps on devices running Android 4.4 (API level 19) or higher cannot
         access external storage outside their own "sandboxed" directory, so
         the READ_EXTERNAL_STORAGE (and WRITE_EXTERNAL_STORAGE) permissions
         aren't necessary. -->
    <uses-permission
          android:name="android.permission.READ_EXTERNAL_STORAGE"
          android:maxSdkVersion="18" />
    ...
</manifest>

```
**MyFileValidityChecker**
```
File ringtone = new File(getExternalFilesDir(DIRECTORY_RINGTONES,
        "my_awesome_new_ringtone.m4a"));
if (isExternalStorageEmulated(ringtone)) {
    Logger.e(TAG, "External storage is not present");
} else if (getExternalStorageState(ringtone) == MEDIA_REMOVED
        | MEDIA_UNMOUNTED | MEDIA_BAD_REMOVAL | MEDIA_UNMOUNTABLE) {
    Logger.e(TAG, "External storage is not available");
} else {
    FileInputStream fis = new FileInputStream(ringtone);

    // available() determines the approximate number of bytes that
    // can be read without blocking.
    int bytesAvailable = fis.available();
    StringBuilder fileContents = new StringBuilder(bytesAvailable);
    byte[] fileBuffer = new byte[bytesAvailable];
    while (fis.read(fileBuffer) != -1) {
        fileContents.append(fileBuffer);
    }

    // Implement appropriate logic for checking a file's validity.
    checkFileValidity(fileContents);
}
```

### 4. 仅将非敏感数据存储在缓存文件中

要更快地访问非敏感应用程序数据，请将其存储在设备的缓存中。对于大小超过`1 MB`的缓存，请使用 `getExternalCacheDir()`; 否则，使用`getCacheDir()`。每种方法都为您提供`File`包含应用程序缓存数据的对象。

以下代码段显示了如何缓存应用最近下载的文件：
```
File cacheDir = getCacheDir();
File fileToCache = new File(myDownloadedFileUri);
String fileToCacheName = fileToCache.getName();
File cacheFile = new File(cacheDir.getPath(), fileToCacheName);
```
**注意：**如果您使用  [getExternalCacheDir()](https://developer.android.google.cn/reference/android/content/Context.html#getExternalCacheDir()) 将应用程序的缓存放在共享存储中，则用户可能会在应用程序运行时弹出包含此存储的媒体。您应该包含逻辑以正常处理此用户行为导致的缓存未命中。
**警告：**没有对这些文件强制执行安全性。因此，任何具有该[WRITE_EXTERNAL_STORAGE](https://developer.android.google.cn/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) 权限的应用都 可以访问此缓存的内容。

### 5. 请在私有模式SharedPreferences  

使用 `getSharedPreferences()`创建或访问应用程序`SharedPreferences`对象时，请使用`MODE_PRIVATE`。这样，只有您的应用可以访问共享首选项文件中的信息。

如果要`跨应用程序共享数据`，请不要使用 `SharedPreferences`对象。相反，您应该按照必要的步骤在应用程序之间安全地共享数据。


#四、使用三方服务并保持SDK依赖库为最新状态


大多数应用程序使用外部库和设备系统信息来完成专门的任务。通过使应用程序的依赖关系保持最新，您可以使这些通信点更加安全。

### 1. 使用 Google Play服务

如果您的应用使用`Google Play`服务，请确保在安装了应用的设备上对其进行了更新。该检查应该在`UI`线程之外异步完成。如果设备不是最新的，则您的应用应触发授权错误。

要确定安装了应用的设备上的`Google Play`服务是否是最新的，请按照[更新安全提供程序以防止SSL攻击](https://developer.android.google.cn/training/articles/security-gms-provider.html)的指南中的步骤进行操作 。

### 2. 更新所有应用依赖项
在部署应用程序之前，请确保所有库，`SDK`和其他依赖项都是最新的：

对于第一方依赖项`（例如Android SDK）`，请使用`Android Studio`中的更新工具，例如 `SDK Manager`。
对于第三方依赖项，请检查应用程序使用的库的网站，并安装任何可用的更新和安全修补程序。

  

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
