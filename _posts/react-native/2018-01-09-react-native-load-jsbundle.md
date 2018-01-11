---
layout: post
title: ReactNative源码分析-加载JSBundle
description: ReactNative加载JSBundle过程
categories: react-native, android
keywords: react-native, android, ReactNative, JsBundle

---
之前我们分析了加载ReactNativeActivitiy的启动，在启动过程中会调用到CatalysInstanceImpl的runJSBundle，此篇我们来看react-native是如何加载JSBundle，我们就从CastalysInstanceImpl中的runJSBundle开始看起。  

```
  @Override
  public void runJSBundle() {
   ... 
   mJSBundleLoader.loadScript(CatalystInstanceImpl.this);
   ...
  }
```

runJSBundle中直接调用mJSBundleLoader的loadScript方法， mJSBundleLoader变量是构造CatalysInstanceImpl时传入的，上篇我们分析过，mJSBundleLoader又是构建ReactInstanceManager创建的，我们直接来看创建的代码  
ReactInstanceManagerBuilder.java  

```java  
new ReactInstanceManager(
        mApplication,
        mCurrentActivity,
        mDefaultHardwareBackBtnHandler,
        mJavaScriptExecutorFactory == null
            ? new JSCJavaScriptExecutorFactory(appName, deviceName)
            : mJavaScriptExecutorFactory,
        (mJSBundleLoader == null && mJSBundleAssetUrl != null)
            ? JSBundleLoader.createAssetLoader(
                mApplication, mJSBundleAssetUrl, false /*Asynchronous*/)
            : mJSBundleLoader,
        mJSMainModulePath,
        mPackages,
        mUseDeveloperSupport,
        mBridgeIdleDebugListener,
        Assertions.assertNotNull(mInitialLifecycleState, "Initial lifecycle state was not set"),
        mUIImplementationProvider,
        mNativeModuleCallExceptionHandler,
        mRedBoxHandler,
        mLazyNativeModulesEnabled,
        mLazyViewManagersEnabled,
        mDelayViewManagerClassLoadsEnabled,
        mDevBundleDownloadListener,
        mMinNumShakes,
        mMinTimeLeftInFrameForNonBatchedOperationMs);
```

可以看到如果用户没有注入一个JSBundleLoader的话， 默认的Loader是AssetLoader, 而在AssertLoader中则又唤起了CatalysInstanceIml的loadScriptFromAssets方法（同级别的方法还有loadScriptFromFile） 

```java 
  public static JSBundleLoader createAssetLoader(
      final Context context,
      final String assetUrl,
      final boolean loadSynchronously) {
    return new JSBundleLoader() {
      @Override
      public String loadScript(CatalystInstanceImpl instance) {
        instance.loadScriptFromAssets(context.getAssets(), assetUrl, loadSynchronously);
        return assetUrl;
      }
    };
  }
```  

CatalysIntanceImpl.java

```java
/* package */ void loadScriptFromAssets(AssetManager assetManager, String assetURL, boolean loadSynchronously) {
	mSourceURL = assetURL;
	jniLoadScriptFromAssets(assetManager, assetURL, loadSynchronously);
}

  private native void jniLoadScriptFromAssets(AssetManager assetManager, String assetURL, boolean loadSynchronously);
```

下面直调用了jni的loadScriptFromAssets方法，我们来看c++部分的实现：  

react/jni/CatalystInstanceImpl.cpp
```c++
void CatalystInstanceImpl::jniLoadScriptFromAssets(
    jni::alias_ref<JAssetManager::javaobject> assetManager,
    const std::string& assetURL,
    bool loadSynchronously) {
  const int kAssetsLength = 9;  // strlen("assets://");
  auto sourceURL = assetURL.substr(kAssetsLength);

  auto manager = extractAssetManager(assetManager);
  //从asset中读取字符串到内存
  auto script = loadScriptFromAssets(manager, sourceURL);
  if (JniJSModulesUnbundle::isUnbundle(manager, sourceURL)) {
    auto bundle = JniJSModulesUnbundle::fromEntryFile(manager, sourceURL);
    auto registry = RAMBundleRegistry::singleBundleRegistry(std::move(bundle));
    instance_->loadRAMBundle(
      std::move(registry),
      std::move(script),
      sourceURL,
      loadSynchronously);
    return;
  } else {
    instance_->loadScriptFromString(std::move(script), sourceURL, loadSynchronously);
  }
}i
```  
其中loadSynchronously为false， 可以回溯到ReactInstanceManagerBuilder重穿件JSBundleLoader的时候。 
从assets中加载完JSBundle后调用jniJSModulesUnBundle的isUnbundle方法判断是否是JSBundle资源，具体判断方法见：  
react/jni/JniJSModulesUnbundle.cpp  

```c++
bool JniJSModulesUnbundle::isUnbundle(
    AAssetManager *assetManager,
    const std::string& assetName) {
  if (!assetManager) {
    return false;
  }

  auto magicFileName = jsModulesDir(assetName) + MAGIC_FILE_NAME;
  auto asset = openAsset(assetManager, magicFileName.c_str());
  if (asset == nullptr) {
    return false;
  }

  magic_number_t fileHeader = 0;
  AAsset_read(asset.get(), &fileHeader, sizeof(fileHeader));
  return fileHeader == htole32(MAGIC_FILE_HEADER);
} 
```

可以看到系统会找到文件，然后读取文件头， 判断文件头是否是MAGEC_FILE_HEADER,来判断是否是JSBundle文件，回到loadScriptFromAssets方法中， 如果此时isUnBundle方法返回true， 则调用instance_的loadRAMBundle方法， 否则调用instance的loadScriptFromString方法。  
instance_ 是Instance对象的实例，可以在CatalystInstanceImpl.h中查找到声明：  

```c++
  std::shared_ptr<Instance> instance_; 
```

可以查看Instance中的具体方法， Instance.cpp文件存在于 ReactCommon/cxxreact目录下：  

```c++
void Instance::loadRAMBundle(std::unique_ptr<RAMBundleRegistry> bundleRegistry,
                             std::unique_ptr<const JSBigString> startupScript,
                             std::string startupScriptSourceURL,
                             bool loadSynchronously) {
  if (loadSynchronously) {
    loadApplicationSync(std::move(bundleRegistry), std::move(startupScript),
                        std::move(startupScriptSourceURL));
  } else {
    loadApplication(std::move(bundleRegistry), std::move(startupScript),
                    std::move(startupScriptSourceURL));
  }
}
```

最终走到上述方法， 因为loadSynchronously是false，则走入到loadAplication中（异步加载）  

```c++
void Instance::loadApplication(std::unique_ptr<RAMBundleRegistry> bundleRegistry,
                               std::unique_ptr<const JSBigString> string,
                               std::string sourceURL) {
  callback_->incrementPendingJSCalls();
  SystraceSection s("Instance::loadApplication", "sourceURL",
                    sourceURL);
  nativeToJsBridge_->loadApplication(std::move(bundleRegistry), std::move(string),
                                     std::move(sourceURL));
}
```  

我们来看加载Module的过程，是通过fromEntryFile加载的是一个JniJsModulesUnbunle, fromEntryFile方法中只是获取了文件路径，并设置了assetManager.然后在CatalysInstanceImpl.cpp中又通过singleBundleRegistry获取了一个RAMBundleRegistry对象，我们来看一下它的具体结构  

```c++
std::unique_ptr<RAMBundleRegistry> RAMBundleRegistry::singleBundleRegistry(std::unique_ptr<JSModulesUnbundle> mainBundle) {
  RAMBundleRegistry *registry = new RAMBundleRegistry(std::move(mainBundle));
  return std::unique_ptr<RAMBundleRegistry>(registry);
}
```
这里直接传入了之前获得的JNIJSModulesUnbundle到RAmBundleRegistry对象中，记录在unique_ram_bundle字段。 

之后最终调用了NativeToJsBridge的loadApplication方法，和CatalystInstnaceImpl.cpp loadScruptFromString的调用最终路径一致，区别在于RanBundleRegistry的参数后者为空. 上一篇我们已经分析了Native和JS之间的通信机制，可以看到这里也是通过NativeToJsBridge来执行js文件的，我们来看loadApplication方法:   

```c++
  runOnExecutorQueue(
      [bundleRegistryWrap=folly::makeMoveWrapper(std::move(bundleRegistry)),
       startupScript=folly::makeMoveWrapper(std::move(startupScript)),
       startupScriptSourceURL=std::move(startupScriptSourceURL)]
        (JSExecutor* executor) mutable {
    auto bundleRegistry = bundleRegistryWrap.move();
    if (bundleRegistry) {
      executor->setBundleRegistry(std::move(bundleRegistry));
    }
    executor->loadApplicationScript(std::move(*startupScript),
                                    std::move(startupScriptSourceURL));
  });
```  
和js通信机制一样在js的线程中调用了JSCExecutor的loadApplicationScript,区别在于含有RamBundleRegistry的额外调用了JSCExecutor的setBundleRegistry方法，我们先看setBundleRegistry方法:  

```c++
 void JSCExecutor::setBundleRegistry(std::unique_ptr<RAMBundleRegistry> bundleRegistry) {
      if (!m_bundleRegistry) {
        installNativeHook<&JSCExecutor::nativeRequire>("nativeRequire");
      }
      m_bundleRegistry = std::move(bundleRegistry);
    }
```  
第一次本地m_bundleRegistry不存在的时候，添加了Hook，至于具体内容不是我们关心的不做套路，然后记录下了RAMBundleRegistry.之后如果出发到registerBundle的时候都回记录在这个BundleRegistry中。  

我们接着看loadloadApplicationScript,此方法中最终掉到了ReactCommon/jschelpers/JSCHelpers.cpp 中的evaluateScript方法:   

```c++
JSValueRef evaluateScript(JSContextRef context, JSStringRef script, JSStringRef sourceURL) {
  JSValueRef exn, result;
  result = JSC_JSEvaluateScript(context, script, NULL, sourceURL, 0, &exn);
  if (result == nullptr) {
    throw JSException(context, exn, sourceURL);
  }
  return result;
}
```

JSC_JSEJSC_JSEvaluateScript 定义在JavaScriptCore中。。。实际上是调用的webkitcore来加载js的。。。可以在ReactAndroid下的libs下的BUCK中找到远程的文件

<pre>
remote_file(
    name = "android-jsc-aar",
    sha1 = "880cedd93f43e0fc841f01f2fa185a63d9230f85",
    url = "mvn:org.webkit:android-jsc:aar:r174650",
</pre>

在maven上可以找到此包，组组有3.9m大，，不难看出为什么RN比较大了。   


