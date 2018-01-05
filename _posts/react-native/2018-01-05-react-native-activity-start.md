---
layout: post
title: ReactNative源码分析-ReactActivity启动
description: ReactiActivity启动过程
categories: react-native, android
keywords: react-native, android, ReactNative, ReactActivity, ReactActivityDelegate

---

最近想要阅读下ReactNative的C++和Android部分的源码。我先罗列一些比较关心的问题：  
1.ReactNative的启动过程   
4.js和native之间是如何通信的 
3.jsBundle是如何被加载的 
4.ReactNative是如何渲染出UI界面的  
5.界面上的Touch事件是如何传递的  

之后的Blogs中我将从源码中一次来寻找上述问题的答案, 我们先来看ReactActivity的启动  

```java
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mDelegate.onCreate(savedInstanceState);
  }
```
mDelegate是ReactActivityDelgate类型  
我们可以看到ReactActivity内部没有做什么特殊处理， 都交给了ReactActivityDelegate去处理，我们来看ReactActivityDelegate中的相应处理  

```java
  protected void onCreate(Bundle savedInstanceState) {
    if (mMainComponentName != null) {
      loadApp(mMainComponentName);
    }
    mDoubleTapReloadRecognizer = new DoubleTapReloadRecognizer();
  }

  protected void loadApp(String appKey) {
    if (mReactRootView != null) {
      throw new IllegalStateException("Cannot loadApp while app is already running.");
    }
    mReactRootView = createRootView();
    mReactRootView.startReactApplication(
      getReactNativeHost().getReactInstanceManager(),
      appKey,
      getLaunchOptions());
    getPlainActivity().setContentView(mReactRootView);
  }

  protected ReactRootView createRootView() {
    return new ReactRootView(getContext());
  }
```

在onCreate中， 调用了loadApp方法，参数是MainCompoentName, 这个就是我们注册到JavaScrip中的mainComponent，在loadApp中delegate直接创建了一个ReactRootView, 然后调用了ReactRootView的startReactApplication方法， 并传入了一个ReactInstanceManager的实例， ReactInstanceManager是一个关键的参数，我们在之后会看到它的具体功能。  

那么创建出来的ReactRootView又是什么呢  

```java
public class ReactRootView extends SizeMonitoringFrameLayout
    implements RootView, MeasureSpecProvider {
}
```
ReactRootView是一个FrameLayout, 这就是原生界面和js的链接出了，阅读一个自定义View我们需要关注它的onMeasure onLayout 和 onDraw方法，我们先来看下onMeasure方法，  

```java 
  @Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    Systrace.beginSection(TRACE_TAG_REACT_JAVA_BRIDGE, "ReactRootView.onMeasure");
    try {
      mWidthMeasureSpec = widthMeasureSpec;
      mHeightMeasureSpec = heightMeasureSpec;

      int width = 0;
      int height = 0;
      int widthMode = MeasureSpec.getMode(widthMeasureSpec);
      if (widthMode == MeasureSpec.AT_MOST || widthMode == MeasureSpec.UNSPECIFIED) {
        for (int i = 0; i < getChildCount(); i++) {
          View child = getChildAt(i);
          int childSize =
              child.getLeft()
                  + child.getMeasuredWidth()
                  + child.getPaddingLeft()
                  + child.getPaddingRight();
          width = Math.max(width, childSize);
        }
      } else {
        width = MeasureSpec.getSize(widthMeasureSpec);
      }
      int heightMode = MeasureSpec.getMode(heightMeasureSpec);
      if (heightMode == MeasureSpec.AT_MOST || heightMode == MeasureSpec.UNSPECIFIED) {
        for (int i = 0; i < getChildCount(); i++) {
          View child = getChildAt(i);
          int childSize =
              child.getTop()
                  + child.getMeasuredHeight()
                  + child.getPaddingTop()
                  + child.getPaddingBottom();
          height = Math.max(height, childSize);
        }
      } else {
        height = MeasureSpec.getSize(heightMeasureSpec);
      }
      setMeasuredDimension(width, height);
      mWasMeasured = true;

      // Check if we were waiting for onMeasure to attach the root view.
      if (mReactInstanceManager != null && !mIsAttachedToInstance) {
        attachToReactInstanceManager();
      } else {
        updateRootLayoutSpecs(mWidthMeasureSpec, mHeightMeasureSpec);
      }

      enableLayoutCalculation();

    } finally {
      Systrace.endSection(TRACE_TAG_REACT_JAVA_BRIDGE);
    }
  }

```

我们总结下，就是如果是AT_MOST 或者 UNSPECTIFIED就遍历子View取出最大值最为当前View的，负责直接使用建议值。特殊的地方在于measure结束后，还触发了一个attachToReactInstanceManager(第一次) 或者 updateupdateRootLayoutSpecs,我们在之后会看到其中具体的行为。  
我们再来看onLayout方法  

```java  
  @Override
  protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    // No-op since UIManagerModule handles actually laying out children.
  }
```
ReactRootView中的onLayout是个空实现，通过注释可以了解到布局被UIManagerModule拦截并进行排布，还记得我们要研究UI的渲染吗，我们先mark下，之后研究UI渲染的时候我们可以从UIManagerModule入手。  
ReactRootView并没有重写onDraw方法。

翻过头来我们继续看ReactiActivityDelegate中的loadApp, 刚才我们说到创建了ReactRootView 并调用了ReactRootView的startReactApplication，我们先来看ReactInstanceManager的注释   

```java
/**
 * This class is managing instances of {@link CatalystInstance}. It exposes a way to configure
 * catalyst instance using {@link ReactPackage} and keeps track of the lifecycle of that
 * instance. It also sets up connection between the instance and developers support functionality
 * of the framework.
 */
``` 

可以理解ReactInstanceManager管理了CatalystInstance, 一起负责js和native之间的通信。想想我们开头提出的问题就有一个关心js和native的通信，可以看到关键的入口点被我们找到了。 我们先mark下之后我们再来深入研究这个问题。   
我们接着来看ReactRootView中的startReactApplication方法：  

```java
  public void startReactApplication(
      ReactInstanceManager reactInstanceManager,
      String moduleName,
      @Nullable Bundle initialProperties) {
  if (!mReactInstanceManager.hasStartedCreatingInitialContext()) {
        mReactInstanceManager.createReactContextInBackground();
      }

      attachToReactInstanceManager();
}
``` 
可以看到这里先是判断了是否初始化创建了ReactContext 然后调用了ReactInstanceManager的attachToReactInstanceManager, 在后面的分析中我们可以发现不管是在onMeasuer中 还是这里 还是在ReactContext创建成功后都会最终出发到attachRootViewToInstance方法，只不过attachRootViewToInstance要求必须已创建并初始化了ReactContext对象， 我们先不管ReactContext对象的创建过程，我们先来看下attachRootViewToInstance方法：  

```java
  private void attachRootViewToInstance(
      final ReactRootView rootView,
      CatalystInstance catalystInstance) {
    UIManagerModule uiManagerModule = catalystInstance.getNativeModule(UIManagerModule.class);
    final int rootTag = uiManagerModule.addRootView(rootView);
    rootView.setRootViewTag(rootTag);
    rootView.invokeJSEntryPoint();
    UiThreadUtil.runOnUiThread(new Runnable() {
      @Override
      public void run() {
        rootView.onAttachedToReactInstance();
      }
    });
  }
```
这里主要做了一下几件事：  
1. 获取UIManagerModule, 具体功能我们再探究UI渲染的时候再深入研究   
2. setRootViewTag  
3. inVokeJsEntryPoint  
4. 回调rootView的onAttachedToReactInstance （UI线程）  

```java
  public void setRootViewTag(int rootViewTag) {
    mRootViewTag = rootViewTag;
  }
```
setRootViewTag 只是记录了一个标志。  
再来看inVokeJsEntryPoint  

```java
 /**
   * Calls into JS to start the React application. Can be called multiple times with the
   * same rootTag, which will re-render the application from the root.
   */
  /*package */ void invokeJSEntryPoint() {
    if (mJSEntryPoint == null) {
      defaultJSEntryPoint();
    } else {
      mJSEntryPoint.run();
    }
  }
  
  private void defaultJSEntryPoint() {
        ReactContext reactContext = mReactInstanceManager.getCurrentReactContext();
        if (reactContext == null) {
          return;
        }

        CatalystInstance catalystInstance = reactContext.getCatalystInstance();

        WritableNativeMap appParams = new WritableNativeMap();
        appParams.putDouble("rootTag", getRootViewTag());
        @Nullable Bundle appProperties = getAppProperties();
        if (appProperties != null) {
          appParams.putMap("initialProps", Arguments.fromBundle(appProperties));
        }

        mShouldLogContentAppeared = true;

        String jsAppModuleName = getJSModuleName();
        catalystInstance.getJSModule(AppRegistry.class).runApplication(jsAppModuleName, appParams);
  
  }
  
```
可以看到如果用户没有set一个自定义过饿JSEntryPoint 则启动默认的，而默认的JSEntryPoint， 从catalystInstance中获取AppRegistry的module然后执行runApplication接口（和jS交互，之后的行为在JS中了）

我们最后回过头来看下ReactContext的创建过程，之前我们说到在ReactRootView的startReactApplication中会调用mReactIntanceManager的createReactContextInBackgroud方法， 我们来看下createReactContextInBackGround方法。  
经过一些方法，最终走到了recreateReactContextInBackground方法  

```java
  private void recreateReactContextInBackground(
    JavaScriptExecutorFactory jsExecutorFactory,
    JSBundleLoader jsBundleLoader) {
    Log.d(ReactConstants.TAG, "ReactInstanceManager.recreateReactContextInBackground()");
    UiThreadUtil.assertOnUiThread();

    final ReactContextInitParams initParams = new ReactContextInitParams(
      jsExecutorFactory,
      jsBundleLoader);
    if (mCreateReactContextThread == null) {
      runCreateReactContextOnNewThread(initParams);
    } else {
      mPendingReactContextInitParams = initParams;
    }
  }
```   
在这里有2个很重要的参数JavaScriptExecutor 和 JSBundleLoader 我们暂且记下，我们来看recreateReactContextInBackground方法， 判断mCurrentReactContextThread == null 的话就调用runCreateReactContextOnNewThread。很明显如果当前没有一个线程正在创建ReactContext的话就启动创建线程， 否则记录创建的参数。  
我们来看runCreateReactContextOnNewThread  

```java
  @ThreadConfined(UI)
  private void runCreateReactContextOnNewThread(final ReactContextInitParams initParams) {
    ...
    mCreateReactContextThread =
        new Thread(
            new Runnable() {
              @Override
              public void run() {
                try {
                  Process.setThreadPriority(Process.THREAD_PRIORITY_DISPLAY);
                  final ReactApplicationContext reactApplicationContext =
                      createReactContext(
                          initParams.getJsExecutorFactory().create(),
                          initParams.getJsBundleLoader());

                  mCreateReactContextThread = null;
                  ReactMarker.logMarker(PRE_SETUP_REACT_CONTEXT_START);
                  final Runnable maybeRecreateReactContextRunnable =
                      new Runnable() {
                        @Override
                        public void run() {
                          if (mPendingReactContextInitParams != null) {
                            runCreateReactContextOnNewThread(mPendingReactContextInitParams);
                            mPendingReactContextInitParams = null;
                          }
                        }
                      };
                  Runnable setupReactContextRunnable =
                      new Runnable() {
                        @Override
                        public void run() {
                          try {
                            setupReactContext(reactApplicationContext);
                          } catch (Exception e) {
                            mDevSupportManager.handleException(e);
                          }
                        }
                      };

                  reactApplicationContext.runOnNativeModulesQueueThread(setupReactContextRunnable);
                  UiThreadUtil.runOnUiThread(maybeRecreateReactContextRunnable);
                } catch (Exception e) {
                  mDevSupportManager.handleException(e);
                }
              }
            });
    mCreateReactContextThread.start();
  }
```

此函数做了一下事情：  
1. 创建ReactApplicationContext对象  
2. 创建 CatalystInstanceImpl用于和js通信  
3. 调用CatalystInstanceImpl的runJSBundle 启动js
4. reactContext.initializeWithInstance(catalystInstance)用CatalystInstanceImpl 初始化ReactContext
5. 判断是否有mPendingReactContextInitParams， 有的话重新启动创建过程  
6. 回到UI线程调用setupReactContext,触发attachRootViewToInstance方法  

到此为止我们基本了解了ReactActivity的启动过程，用时序图来表示的话，如下图所示（部分嵌套调用的方法省略）：  

![](/image/react-native/start-seque.jpg)


