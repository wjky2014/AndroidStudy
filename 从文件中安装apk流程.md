#### 和你一起终身学习，这里是程序员 Android

本篇文章主要介绍 `Android` 开发中的部分知识点，通过阅读本篇文章，您将收获以下内容:
>一、前言
>二、PackageInstaller介绍
>三、App安装过程中涉及类
>四、App安装流程分析
>五、总结


# 一、前言
首先本文不是做PackageManagerService学习总结，PackageManagerService这货有1万2千多行代码，学习起来颇费劲，并且这货功能强大，本文只会总结其中一个小小的功能
为何要做这个总结呢？说来话长，鄙人菜鸟一枚，接到一个安装应用过程中重启的问题，原因找到，但不知如何解决，无奈，只有硬着头皮学习了下这部分内容
OK，废话不多说，接下来直接上干货，如果文中有问题或有质疑的地方可以直接修改，不胜感激。

# 二、PackageInstaller介绍
PackageInstaller是个神马东西呢？
我们知道安装app有很多中方式，诸如adb install,应用助手（豌豆荚）,开机安装（开机启动时），下载到手机存储后点击安装。PackageInstaller这哥就是给手动安装app提供一个界面的apk。
当我们点安装应用时会启动这个应用来显示安装过程，安装的事情并不是他在做，正在安装是由PackageManagerService来完成，当然幕后英雄确是Installer。
代码位置
packages/apps/PackageInstaller
疑问【以下有几个问题，如果亲都知道的话，那么可以不用再看本文啦】
如何使用PackageInstaller来安装应用？
应用首选安装位置是在何时确定的？是在设置里设置的，还是在app中定义的，还是PackageManagerService这货说了算？
安装应用过程中，哪些服务和类会插手这件事？
安装过程中首先会生成一个.tmp临时文件，这个文件在何时被rename为apk的？
应用都有uid，这个uid是在什么时候被赋值的？
packages.xml and packages.list有什么用？
安装app主要做了哪些事？

# 三、App安装过程中涉及类
先来盘类图看看，如对PackageManagerService不熟悉的话，先看后面的流程，看完再来看这个类图

安装过程时会插手的主要类如下
```
packages/apps/PackageInstaller/src/com/android/packageinstaller/PackageInstallerActivity.java
packages/apps/PackageInstaller/src/com/android/packageinstaller/InstallAppProgress.java
frameworks/base /core/java/android/app/ApplicationPackageManager.java
frameworks/base /core/java/android/content/pm/PackageParser.java
frameworks/base /core/java/android/content/res/AssetManager.java
frameworks/base /packages/DefaultContainerService/src/com/android/defcontainer/DefaultContainerService.java
frameworks/base /services/java/com/android/server/pm/Installer.java
frameworks/base /services/java/com/android/server/pm/PackageManagerService.java
frameworks/base /services/java/com/android/server/pm/Settings.java
```
简单说明这些类作用

- PackageInstallerActivity.java：
在文件管理器里点击apk后就会调用该类，主要用于显示要安装的apk的一些权限信息
- InstallAppProgress.java：
当看完所有权限后，点安装后就会调用该类，用于显示安装进度，这时候- PackageManagerService 就在默默的安装应用
- ApplicationPackageManager.java：
这是类是PackageManager的儿子，我们使用mContext.getPackageManager得到的其实就是ApplicationPackageManager的对象，它爹PackageManager是个抽象类，对外的方法都定义在里面
- PackageParser.java：
解析app，主要解析apk中的AndroidManifest.xml，解析里面的四大组件以及权限信息放入内存里，最后写到packages.xml和package.list（/data/system下）中
- AssetManager.java：
把AndroidManifest.xml从app中拿出来给PackageParser.java去解析
- DefaultContainerService.java：
这个服务用于检查存储状态，得到合适的安装位置
- Installer.java：
PackageManagerService调用它去执行安装，他会把PackageManagerService传过来的数据封装成命令，然后让底层的Installer去执行
- PackageManagerService.java：
管理app的大神，安装、移动、卸载、查询等都由他管

# 四、App安装流程分析
先来个时序图--安装成功的时序图。点击两次可看大图

![安装流程](https://upload-images.jianshu.io/upload_images/5851256-734d9945e332b247.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



流程分析，当然对着代码看上面的时序图也很明了
1.当点击文件管理器中的apk时，会调用FolderFragment的openFile方法，该方法里会将应用信息传给PackageInstallerActivity，并启动PackageInstaller
```
代码位置：vendor/qcom/proprietary/qrdplus/FileExplorer/src/com/android/qrdfileexplorer/FolderFragment.java

private void openFile(File f) {
        final Uri fileUri = Uri.fromFile(f);
        final Intent intent = new Intent();
        intent.setAction(android.content.Intent.ACTION_VIEW);
        intent.putExtra(Intent.EXTRA_TITLE, f.getName());
        intent.putExtra(EXTRA_ALL_VIDEO_FOLDER, true);
        Uri contentUri = null;
        String type = getMIMEType(f);
        ......
            if (contentUri != null) {
                intent.setDataAndType(contentUri, type);
            } else {
                intent.setDataAndType(fileUri, type);
            }
            try {
                startActivitySafely(intent);
            } 
        ......
    }
```
2.PackageInstaller启动过后会检查是否开启未知来源，未开启就需要先进入设置设置后，方可继续安装，之后会依次调用initiateInstall()->startInstallConfirm();
在initiateInstall中会检查是否已经安装过，是否是系统应用等，调用startInstallConfirm去初始化界面，显示权限信息，当点击安装按钮时，启动安装，切换界面到InstallAppProgress
```
代码位置：packages/apps/PackageInstaller/src/com/android/packageinstaller/PackageInstallerActivity.java
 @Override
    protected void onCreate(Bundle icicle) {
        ......
        mPm = getPackageManager();
        boolean requestFromUnknownSource = isInstallRequestFromUnknownSource(intent);
        ......
        initiateInstall();
    }
    private void initiateInstall() {
        String pkgName = mPkgInfo.packageName;
        String[] oldName = mPm.canonicalToCurrentPackageNames(new String[] { pkgName });
        if (oldName != null && oldName.length > 0 && oldName[0] != null) {
            pkgName = oldName[0];
            mPkgInfo.packageName = pkgName;
            mPkgInfo.applicationInfo.packageName = pkgName;
        }
        // Check if package is already installed. display confirmation dialog if replacing pkg
        try {
            // This is a little convoluted because we want to get all uninstalled
            // apps, but this may include apps with just data, and if it is just
            // data we still want to count it as "installed".
            mAppInfo = mPm.getApplicationInfo(pkgName,
                    PackageManager.GET_UNINSTALLED_PACKAGES);
            if ((mAppInfo.flags&ApplicationInfo.FLAG_INSTALLED) == 0) {
                mAppInfo = null;
            }
        } catch (NameNotFoundException e) {
            mAppInfo = null;
        }
        mInstallFlowAnalytics.setReplace(mAppInfo != null);
        mInstallFlowAnalytics.setSystemApp(
                (mAppInfo != null) && ((mAppInfo.flags & ApplicationInfo.FLAG_SYSTEM) != 0));
        startInstallConfirm();
    }
```
3.在InstallAppProgress中会调用initView去初始化界面并调用ApplicationPackageManager的installPackageWithVerificationAndEncryption方法来安装
```
代码位置：packages/apps/PackageInstaller/src/com/android/packageinstaller/InstallAppProgress.java
  @Override
    public void onCreate(Bundle icicle) {
        ......
        initView();
    }
    public void initView() {
        ......
        if ("package".equals(mPackageURI.getScheme())) {
            try {
                pm.installExistingPackage(mAppInfo.packageName);
                observer.packageInstalled(mAppInfo.packageName,
                        PackageManager.INSTALL_SUCCEEDED);
            } catch (PackageManager.NameNotFoundException e) {
                observer.packageInstalled(mAppInfo.packageName,
                        PackageManager.INSTALL_FAILED_INVALID_APK);
            }
        } else {
            pm.installPackageWithVerificationAndEncryption(mPackageURI, observer, installFlags,
                    installerPackageName, verificationParams, null);
        }
    }
```
4.ApplicationPackageManager的installPackageWithVerificationAndEncryption里也是调用PMS的installPackageWithVerificationAndEncryption方法
```
代码位置：frameworks/base/core/java/android/app/ApplicationPackageManager.java
    @Override
    public void installPackageWithVerificationAndEncryption(Uri packageURI,
            IPackageInstallObserver observer, int flags, String installerPackageName,
            VerificationParams verificationParams, ContainerEncryptionParams encryptionParams) {
        try {
            mPM.installPackageWithVerificationAndEncryption(packageURI, observer, flags,
                    installerPackageName, verificationParams, encryptionParams);
        } catch (RemoteException e) {
            // Should never happen!
        }
    }
```
5.installPackageWithVerificationAndEncryption方法里，首先会获取设置中的用户安装位置，并且会把InstallParams对象和安装位置flag封装到Message里，然后发出一个消息后就撒手不管了。
```
代码位置：frameworks/base/services/java/com/android/server/pm/PackageManagerService.java
public void installPackageWithVerificationAndEncryption(Uri packageURI,
            IPackageInstallObserver observer, int flags, String installerPackageName,
            VerificationParams verificationParams, ContainerEncryptionParams encryptionParams) {
        mContext.enforceCallingOrSelfPermission(android.Manifest.permission.INSTALL_PACKAGES,
                null);
        final int uid = Binder.getCallingUid();
        if(getInstallLocation() == PackageHelper.APP_INSTALL_INTERNAL){
            userFilteredFlags = flags |PackageManager.INSTALL_INTERNAL;
        } else if(getInstallLocation() == PackageHelper.APP_INSTALL_EXTERNAL){
            userFilteredFlags = flags |PackageManager.INSTALL_EXTERNAL;
        } else{
            userFilteredFlags = filteredFlags;
        }
        final Message msg = mHandler.obtainMessage(INIT_COPY);
        msg.obj = new InstallParams(packageURI, observer, userFilteredFlags, installerPackageName,
                verificationParams, encryptionParams, user);
        mHandler.sendMessage(msg);
    }
```
6.接下来就该PackageHandler上场了，会依次处理INIT_COPY、MCS_BOUN消息，这里面会去连接DefaultContainerService服务，接着会InstallParams的startCopy方法
```
代码位置：frameworks/base/services/java/com/android/server/pm/PackageManagerService.java ->内部类：PackageHandler
 public void handleMessage(Message msg) {
        try {
            doHandleMessage(msg);
        } finally {
            Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        }
    }
    void doHandleMessage(Message msg) {
        switch (msg.what) {
            case INIT_COPY: {
                HandlerParams params = (HandlerParams) msg.obj;
                if (!mBound) {
                    if (!connectToService()) {
                        params.serviceError();
                        return;
                    } else {
                        mPendingInstalls.add(idx, params);
                    }
                } else {
                    mPendingInstalls.add(idx, params);
                    if (idx == 0) {
                        mHandler.sendEmptyMessage(MCS_BOUND);
                    }
                }
                break;
            }
            case MCS_BOUN: {
                if (msg.obj != null) {
                    mContainerService = (IMediaContainerService) msg.obj;
                }
                if (mContainerService == null) {
                    for (HandlerParams params : mPendingInstalls) {
                        params.serviceError();
                    }
                    mPendingInstalls.clear();
                } else if (mPendingInstalls.size() > 0) {
                    HandlerParams params = mPendingInstalls.get(0);
                    if (params != null) {
                        if (params.startCopy()) {
                            ......
                        }
                    }
                } else {
                    Slog.w(TAG, "Empty queue");
                }
                break;
            }
            ......
            case POST_INSTALL: {
                ...
                if (data != null) {
                    InstallArgs args = data.args;
                    PackageInstalledInfo res = data.res;
                    if (res.returnCode == PackageManager.INSTALL_SUCCEEDED) {
                        ......
                        sendPackageBroadcast(Intent.ACTION_PACKAGE_ADDED,
                                res.pkg.applicationInfo.packageName, extras, null, null, firstUsers);
                        final boolean update = res.removedInfo.removedPackage != null;
                        if (update) {
                            extras.putBoolean(Intent.EXTRA_REPLACING, true);
                        }
                        sendPackageBroadcast(Intent.ACTION_PACKAGE_ADDED,
                                res.pkg.applicationInfo.packageName, extras, null, null, updateUsers);
                        if (update) {
                            sendPackageBroadcast(Intent.ACTION_PACKAGE_REPLACED,
                                    res.pkg.applicationInfo.packageName, extras, null, null, updateUsers);
                            sendPackageBroadcast(Intent.ACTION_MY_PACKAGE_REPLACED,
                                    null, null, res.pkg.applicationInfo.packageName, null, updateUsers);
                            if (isForwardLocked(res.pkg) || isExternal(res.pkg)) {
                                ......
                                sendResourcesChangedBroadcast(true, true, pkgList,uidArray, null);
                            }
                        }
                        if (res.removedInfo.args != null) {
                            deleteOld = true;
                        }
                    }
                    ......
                    if (args.observer != null) {
                        try {
                            args.observer.packageInstalled(res.name, res.returnCode);
                        } catch (RemoteException e) {
                            Slog.i(TAG, "Observer no longer exists.");
                        }
                    }
                } else {
                    Slog.e(TAG, "Bogus post-install token " + msg.arg1);
                }
                break;
            }
        }
    }
```
7.InstallParams的startCopy方法里，会调用handleStartCopy方法
```
代码位置：frameworks/base/services/java/com/android/server/pm/PackageManagerService.java ->内部类：InstallParams 继承于HandlerParams
    final boolean startCopy() {
        boolean res;
        try {
            if (++mRetries > MAX_RETRIES) {
                Slog.w(TAG, "Failed to invoke remote methods on default container service. Giving up");
                mHandler.sendEmptyMessage(MCS_GIVE_UP);
                handleServiceError();
                return false;
            } else {
                handleStartCopy();
                res = true;
            }
        } catch (RemoteException e) {
            mHandler.sendEmptyMessage(MCS_RECONNECT);
            res = false;
        }
        handleReturnCode();
        return res;
    }
````
8.handleStartCopy方法中会检查应用是否能安装，如不合法则返回FAILED的CODE，接着会调用DefaultContainerService的getMinimalPackageInfo方法，该方法用于获取存储状态，返回合适的安装位置
如果返回码是INSTALL_SUCCEEDED，那接下来就会调用InstallParams的copyApk，如果安装到内置，调用的就是FileInstallArgs的copyApk方法，如安装到外置就调用AsecInstallArgs的copyApk方法
AsecInstallArgs和FileInstallArgs都是InstallParams的子类
```
代码位置：frameworks/base/services/java/com/android/server/pm/PackageManagerService.java ->内部类：FileInstallArgs 继承于InstallParams
public void handleStartCopy() throws RemoteException {
        ......
        if (onInt && onSd) {
            ret = PackageManager.INSTALL_FAILED_INVALID_INSTALL_LOCATION;
        } else {
            ......
            try {
                mContext.grantUriPermission(DEFAULT_CONTAINER_PACKAGE, mPackageURI,
                        Intent.FLAG_GRANT_READ_URI_PERMISSION);
                ........
                if (packageFile != null) {
                    ......
                    pkgLite = mContainerService.getMinimalPackageInfo(packageFilePath, flags, lowThreshold);
                }
            }
        }
        final InstallArgs args = createInstallArgs(this);
        mArgs = args;
        ......
        if (ret == PackageManager.INSTALL_SUCCEEDED) {
            ......
                ret = args.copyApk(mContainerService, true);
            ......
        }
        mRet = ret;
    }
```
9.copyApk方法中会依次调用FileInstallArgs 的createCopyFile->PackageManagerService的createTempPackageFile方法去创建临时文件。
```
代码位置：frameworks/base/services/java/com/android/server/pm/PackageManagerService.java ->内部类：FileInstallArgs 继承于InstallParams
          frameworks/base/services/java/com/android/server/pm/PackageManagerService.java
vmdl*.tmp就是copy成的临时文件
    void createCopyFile() {
        installDir = isFwdLocked() ? mDrmAppPrivateInstallDir : mAppInstallDir;
        codeFileName = createTempPackageFile(installDir).getPath();
        resourceFileName = getResourcePathFromCodePath();
        libraryPath = getLibraryPathFromCodePath();
        created = true;
    }
    private File createTempPackageFile(File installDir) {
        File tmpPackageFile;
        try {
            tmpPackageFile = File.createTempFile("vmdl", ".tmp", installDir);
        } catch (IOException e) {
            Slog.e(TAG, "Couldn't create temp file for downloaded package file.");
            return null;
        }
        try {
            FileUtils.setPermissions(
                    tmpPackageFile.getCanonicalPath(), FileUtils.S_IRUSR|FileUtils.S_IWUSR,
                    -1, -1);
            if (!SELinux.restorecon(tmpPackageFile)) {
                return null;
            }
        } catch (IOException e) {
            Slog.e(TAG, "Trouble getting the canoncical path for a temp file.");
            return null;
        }
        return tmpPackageFile;
    }
```
10.临时文件已经有了，handleStartCopy方法走完，接着回到步骤7，调用InstallParams的handleReturnCode方法，handleReturnCode中会执行processPendingInstall，在该方法中做了大量工作
```
    @Override
    void handleReturnCode() {
        if (mArgs != null) {
            processPendingInstall(mArgs, mRet);
            if (mTempPackage != null) {
                if (!mTempPackage.delete()) {
                    Slog.w(TAG, "Couldn't delete temporary file: " +
                            mTempPackage.getAbsolutePath());
                }
            }
        }
    }
```
11.来看看processPendingInstall到底做了什么？processPendingInstall中最关键方法--installPackageLI，主要的操作（验证签名，创建/data/data，分配UID,dexopt）都在这个方法中完成。
```
    private void processPendingInstall(final InstallArgs args, final int currentStatus) {
        mHandler.post(new Runnable() {
            public void run() {
                mHandler.removeCallbacks(this);
                PackageInstalledInfo res = new PackageInstalledInfo();
                res.returnCode = currentStatus;
                res.uid = -1;
                res.pkg = null;
                res.removedInfo = new PackageRemovedInfo();
                if (res.returnCode == PackageManager.INSTALL_SUCCEEDED) {
                    args.doPreInstall(res.returnCode);
                    synchronized (mInstallLock) {
                        installPackageLI(args, true, res);
                    }
                    args.doPostInstall(res.returnCode, res.uid);
                }
                ......
                if (!doRestore) {
                    Message msg = mHandler.obtainMessage(POST_INSTALL, token, 0);
                    mHandler.sendMessage(msg);
                }
            }
        });
    }
```
12.我们来看一下installPackageLI方法，首选会让parsePackage去解析apk里的AndroidManifest.xml,使用的是parsePackage方法，把解析出来的内容放到Package对象中
接着调用doRename去将之前的tmp文件重命名为apk。apk已经在/data/app下了，apk的属性也被解析出来放在内存（Package对象）中了
那么现在还需要做什么呢？apk有了，数据目录(/data/data）还没有，所以后面会进行uid赋值，验证签名，创建相应的/data/data目录，dexopt操作，这些工作是由installNewPackageLI来完成
```
    private void installPackageLI(InstallArgs args,
            boolean newInstall, PackageInstalledInfo res) {
        ......
        int parseFlags = mDefParseFlags | PackageParser.PARSE_CHATTY
                | (forwardLocked ? PackageParser.PARSE_FORWARD_LOCK : 0)
                | (onSd ? PackageParser.PARSE_ON_SDCARD : 0);
        PackageParser pp = new PackageParser(tmpPackageFile.getPath());
        pp.setSeparateProcesses(mSeparateProcesses);
        final PackageParser.Package pkg = pp.parsePackage(tmpPackageFile,
                null, mMetrics, parseFlags);
        ......
        if (!args.doRename(res.returnCode, pkgName, oldCodePath)) {
            res.returnCode = PackageManager.INSTALL_FAILED_INSUFFICIENT_STORAGE;
            return;
        }
        ......
        if (replace) {
            replacePackageLI(pkg, parseFlags, scanMode, args.user,
                    installerPackageName, res);
        } else {
            installNewPackageLI(pkg, parseFlags, scanMode | SCAN_DELETE_DATA_ON_FAILURES, args.user,
                    installerPackageName, res);
        }
        synchronized (mPackages) {
            final PackageSetting ps = mSettings.mPackages.get(pkgName);
            if (ps != null) {
                res.newUsers = ps.queryInstalledUsers(sUserManager.getUserIds(), true);
            }
        }
    }
```
13.让我们来看看installNewPackageLI具体怎么完成这些工作的吧。
```
private void installNewPackageLI(PackageParser.Package pkg,
            int parseFlags, int scanMode, UserHandle user,
            String installerPackageName, PackageInstalledInfo res) {
        ......
        PackageParser.Package newPackage = scanPackageLI(pkg, parseFlags, scanMode, System.currentTimeMillis(), user);
        if (newPackage == null) {
            ......
        } else {
            updateSettingsLI(newPackage, installerPackageName, null, null, res);
            ......
            if (res.returnCode != PackageManager.INSTALL_SUCCEEDED) {
                deletePackageLI(pkgName, UserHandle.ALL, false, null, null, dataDirExists ? PackageManager.DELETE_KEEP_DATA : 0,
                        res.removedInfo, true);
            }
        }
    }
```
14.scanPackageLI方法中，先调用getPackageLPw->newUserIdLPw（Settings类方法)去设置uid，在调用verifySignaturesLP验证签名
然后调用createDataDirsLI创建/data/data数据目录，最后调用performDexOptLI进行dexopt操作
createDataDirsLI是靠调用mInstaller.install方法来完成目录创建，framework中的Installer会和底层幕后Installer勾兑，完成目录创建工作
performDexOptLI操作最后也是通过mInstaller.dexopt来完成的
```
    private PackageParser.Package scanPackageLI(PackageParser.Package pkg,
            int parseFlags, int scanMode, long currentTime, UserHandle user) {
        ......
        synchronized (mPackages) {
            ......
            pkgSetting = mSettings.getPackageLPw(pkg, origPackage, realName, suid, destCodeFile,
                    destResourceFile, pkg.applicationInfo.nativeLibraryDir,
                    pkg.applicationInfo.flags, user, false);
            if (pkgSetting == null) {
                mLastScanError = PackageManager.INSTALL_FAILED_INSUFFICIENT_STORAGE;
                return null;
            }
            ......
            if (!verifySignaturesLP(pkgSetting, pkg)) {
                if ((parseFlags&PackageParser.PARSE_IS_SYSTEM_DIR) == 0) {
                    return null;
                }
                // The signature has changed, but this package is in the system
                // image...  let's recover!
                pkgSetting.signatures.mSignatures = pkg.mSignatures;
                // However...  if this package is part of a shared user, but it
                // doesn't match the signature of the shared user, let's fail.
                // What this means is that you can't change the signatures
                // associated with an overall shared user, which doesn't seem all
                // that unreasonable.
                if (pkgSetting.sharedUser != null) {
                    if (compareSignatures(pkgSetting.sharedUser.signatures.mSignatures,
                            pkg.mSignatures) != PackageManager.SIGNATURE_MATCH) {
                        Log.w(TAG, "Signature mismatch for shared user : " + pkgSetting.sharedUser);
                        mLastScanError = PackageManager.INSTALL_PARSE_FAILED_INCONSISTENT_CERTIFICATES;
                        return null;
                    }
                }
                // File a report about this.
                String msg = "System package " + pkg.packageName
                        + " signature changed; retaining data.";
                reportSettingsProblem(Log.WARN, msg);
            }
            ......
        }
        ......
        if (mPlatformPackage == pkg) {
            ......
        } else {
            ......
            if (dataPath.exists()) {
               ......
            } else {
                ......
                int ret = createDataDirsLI(pkgName, pkg.applicationInfo.uid,
                                           pkg.applicationInfo.seinfo);
                if (ret < 0) {
                    // Error from installer
                    mLastScanError = PackageManager.INSTALL_FAILED_INSUFFICIENT_STORAGE;
                    return null;
                }
                if (dataPath.exists()) {
                    pkg.applicationInfo.dataDir = dataPath.getPath();
                } else {
                    Slog.w(TAG, "Unable to create data directory: " + dataPath);
                    pkg.applicationInfo.dataDir = null;
                }
            }
            /*
             * Set the data dir to the default "/data/data/<package name>/lib"
             * if we got here without anyone telling us different (e.g., apps
             * stored on SD card have their native libraries stored in the ASEC
             * container with the APK).
             *
             * This happens during an upgrade from a package settings file that
             * doesn't have a native library path attribute at all.
             */
            if (pkg.applicationInfo.nativeLibraryDir == null && pkg.applicationInfo.dataDir != null) {
                if (pkgSetting.nativeLibraryPathString == null) {
                    setInternalAppNativeLibraryPath(pkg, pkgSetting);
                } else {
                    pkg.applicationInfo.nativeLibraryDir = pkgSetting.nativeLibraryPathString;
                }
            }
            pkgSetting.uidError = uidError;
        }
        ......
        // We also need to dexopt any apps that are dependent on this library.  Note that
        // if these fail, we should abort the install since installing the library will
        // result in some apps being broken.
        if (clientLibPkgs != null) {
            if ((scanMode&SCAN_NO_DEX) == 0) {
                for (int i=0; i<clientLibPkgs.size(); i++) {
                    PackageParser.Package clientPkg = clientLibPkgs.get(i);
                    if (performDexOptLI(clientPkg, forceDex, (scanMode&SCAN_DEFER_DEX) != 0, false)
                            == DEX_OPT_FAILED) {
                        if ((scanMode & SCAN_DELETE_DATA_ON_FAILURES) != 0) {
                            removeDataDirsLI(pkg.packageName);
                        }
 
                        mLastScanError = PackageManager.INSTALL_FAILED_DEXOPT;
                        return null;
                    }
                }
            }
        }
        // Request the ActivityManager to kill the process(only for existing packages)
        // so that we do not end up in a confused state while the user is still using the older
        // version of the application while the new one gets installed.
        ......
        // Also need to kill any apps that are dependent on the library.
        ......
        return pkg;
    }
    private int createDataDirsLI(String packageName, int uid, String seinfo) {
        int[] users = sUserManager.getUserIds();
        int res = mInstaller.install(packageName, uid, uid, seinfo);
        if (res < 0) {
            return res;
        }
        for (int user : users) {
            if (user != 0) {
                res = mInstaller.createUserData(packageName,
                        UserHandle.getUid(user, uid), user);
                if (res < 0) {
                    return res;
                }
            }
        }
        return res;
    }
    private int performDexOptLI(PackageParser.Package pkg, boolean forceDex, boolean defer,
            boolean inclDependencies) {
        ......
        return performDexOptLI(pkg, forceDex, defer, done);
    }
    private int performDexOptLI(PackageParser.Package pkg, boolean forceDex, boolean defer,
            HashSet<String> done) {
        ......
        if ((pkg.applicationInfo.flags&ApplicationInfo.FLAG_HAS_CODE) != 0) {
            ......
            try {
                if (forceDex || dalvik.system.DexFile.isDexOptNeeded(path)) {
                    if (!forceDex && defer) {
                        if (mDeferredDexOpt == null) {
                            mDeferredDexOpt = new HashSet<PackageParser.Package>();
                        }
                        mDeferredDexOpt.add(pkg);
                        return DEX_OPT_DEFERRED;
                    } else {
                        final int sharedGid = UserHandle.getSharedAppGid(pkg.applicationInfo.uid);
                        ret = mInstaller.dexopt(path, sharedGid, !isForwardLocked(pkg));
                        pkg.mDidDexOpt = true;
                        performed = true;
                    }
                }
            } catch (FileNotFoundException e) {
                ......
            } catch (IOException e) {
                ......
            } catch (dalvik.system.StaleDexCacheError e) {
                ......
            } catch (Exception e) {
                ......
            }
            if (ret < 0) {
                //error from installer
                return DEX_OPT_FAILED;
            }
        }
        return performed ? DEX_OPT_PERFORMED : DEX_OPT_SKIPPED;
    }
performDexOptLI后生成的文件
```
15.到目前为止，scanPackageLI已经走完了，接下来就该更新packages.list,packages.xml了
系统中所有app的信息都保存在这两个文件中，当有app安装、卸载、更新时都会更新这两个文件
回到步骤13，当installNewPackageLI中的scanPackageLI走完后，后面会调用updateSettingsLI去更新文件
mSettings.writeLPr()来完成往packages.list,packages.xml中更新数据
```
    private void updateSettingsLI(PackageParser.Package newPackage, String installerPackageName,
            int[] allUsers, boolean[] perUserInstalled,
            PackageInstalledInfo res) {
        String pkgName = newPackage.packageName;
        synchronized (mPackages) {
            ......
            mSettings.setInstallStatus(pkgName, PackageSettingBase.PKG_INSTALL_INCOMPLETE);
            mSettings.writeLPr();
        }
        ......
        synchronized (mPackages) {
            updatePermissionsLPw(newPackage.packageName, newPackage,
                    UPDATE_PERMISSIONS_REPLACE_PKG | (newPackage.permissions.size() > 0
                            ? UPDATE_PERMISSIONS_ALL : 0));
            ......
            mSettings.writeLPr();
        }
    }
```
packages.list文件部分截取

packages.xml部分截取，包含包名，安装时间，签名，权限，文件安装路径等信息

16.installPackageLI到这里已经执行完了，现在回到步骤11，后续会执行到Message msg = mHandler.obtainMessage(POST_INSTALL, token, 0); mHandler.sendMessage(msg);发送消息
消息发出后，回到步骤POST_INSTALL，这里面主要做了两件事，发生一条ACTION_PACKAGE_ADDED广播告诉大家，又有新包了，这是launcher什么的赶紧把图标加上
然后回调args.observer.packageInstalled(res.name, res.returnCode);告诉PackageInstaller安装结果
然后就显示安装完成界面。欧拉，应用安装结束。


# 五、总结
小结一下安装app到底主要做了哪些事情？

1.验证是否允许安装未知来源应用
2.得到用户设置的首选安装位置
3.检验app有效性，检查存储状态，得到最佳安装位置
4.拷贝app到安装位置，此时为.tmp临时文件
5.解析AndroidManifest.xml
6.重命名tmp为apk
7.赋值UID，验证权限
8.创建/data/data/下数据目录
9.执行dexopt操作
10.更新packages.xml,packages.list
11.发送广播，回调安装状态

原文作者：默默9518 
原文链接：https://blog.csdn.net/sgzy001/article/details/44857057



**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
