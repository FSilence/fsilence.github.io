---
layout: post
title: Architecture Components概述
description: Android Architecture Components的简介
categories: android 
keywords: android, architecture, LiveData, Room, ViewModel, Handling lifecycles
---
Google 在2017的google大会上发布了Architecture Components一系列Libraries来帮助开发者构建健壮的，可测试的，易维护的apps。 它可以帮助你更好的管理UI的生命周期和数据持久化，避免在非UI可用状态的页面更新导致的问题，例如在Activity 或者Fragment销毁时，依然有UI操作的异常bug。  
相关包的引入如下:  

```gradle
// ViewModel and LiveData
implementation "android.arch.lifecycle:extensions:1.1.0"
annotationProcessor "android.arch.lifecycle:compiler:1.1.0"

// Room
implementation "android.arch.persistence.room:runtime:1.0.0"
annotationProcessor "android.arch.persistence.room:compiler:1.0.0"

// Paging
implementation "android.arch.paging:runtime:1.0.0-alpha4-1"

// Test helpers for LiveData
testImplementation "android.arch.core:core-testing:1.1.0"

// Test helpers for Room
testImplementation "android.arch.persistence.room:testing:1.0.0"
```

我们后续将逐步介绍以下内容:  
1.如何观察UI的生命周期  
2.使用LiveData 让UI更新只发生在UI安全的情况下  
3.使用ViewModel来管理数据  
4.使用Room 进行数据库持久化  

## Handling Lifecycle 
本节我们将涉及到3个主要的概念 Lifecycle LifecycleOwner LifecycleObserver,其中Lifecycle是核心类，可以通过Lifecycle的addObserver 和  removeObserver方法，来添加或移除生命周期的观察者（LifecycleObserver）, 而LifecycleOwner表示了Lifecyle的宿主，可以通过getLifecycle方法来获取一个Lifecyle对象的实例。  
  
### Lifecycle
可以通过Lifecycle类来管理activity 或 fragment的生命周期，其中2个枚举类 Event 和 Status，基于事件的状态流转如下图所示:  
![](/images/arch-lifecycle-states.png)

lifecyle的使用方式如下:  
```java
public class MyObserver implements LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```
可以通过实现LifeecyleObserver接口，然后通过注解OnLifeCycleEevent来申明各个生命周期唤醒的方法。  

在State类中，还有一个isAtLeast 的帮助方法，可以帮助我们在某个状态之后的状态，例如 isAtLeast(CREATED)， 在CREATED, STARTED, RESUMED的时候回返回true，其它为false。  

### LifecyleOwner
其中只有一个方法getLifecyle()用来获取Lifecyle实例，在Support 26.1.0上，Fragment和AppComponentActivity都默认实现了此接口，如果使用低于此版本的support包，可以使用文章开头的方式导入相关包，并使用如下方式定义通用Activity.  

```java

public class MyActivity extends Activity implements LifecycleOwner {
    private LifecycleRegistry mLifecycleRegistry;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mLifecycleRegistry = new LifecycleRegistry(this);
        mLifecycleRegistry.markState(Lifecycle.State.CREATED);
    }

    @Override
    public void onStart() {
        super.onStart();
        mLifecycleRegistry.markState(Lifecycle.State.STARTED);
    }

    @NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
}
```

### 注意
在AppCompatActivity 和 Fragment的onSavfeInstanceState触发时，就会触发ON_STOP事件，并将状态变化为CREATED. 因为系统配套的LiveData中会认为状态STARTED之后的为UI活跃的事件，在23以前，跳转半透明的Activity会导致onSaveInstanceState触发,但是onStop不一定触发，这样就导致了在onSaveInstanceState和onStop之间有很长时间误认为为active状态，为了避免这种情况在onSaveInstanceState中将状态修改为了CREATED。为了是流程简单统一，之后将ON_STOP事件也迁移到了在onSaveInstanceState中触发。  

## LiveData
LiveData只会在状态为STARTED RESUMED的时候推送最新的数据.LiveData有一些优势:  
1.及时同步数据到UI  
2.没有内存泄漏  
3.不用再关注UI的生命周期  
3.当在UI非活动状态更新的数据将保留到状态变为active时再推送  
4.可以使用单例让数据共享

LiveData通常和ViewModel配合使用，基本用法如下:  

```java
public class NameViewModel extends ViewModel {

// Create a LiveData with a String
private MutableLiveData<String> mCurrentName;

    public MutableLiveData<String> getCurrentName() {
        if (mCurrentName == null) {
            mCurrentName = new MutableLiveData<String>();
        }
        return mCurrentName;
    }

// Rest of the ViewModel...
}

public class NameActivity extends AppCompatActivity {

    private NameViewModel mModel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Other code to setup the activity...

        // Get the ViewModel.
        mModel = ViewModelProviders.of(this).get(NameViewModel.class);

        // Create the observer which updates the UI.
        final Observer<String> nameObserver = new Observer<String>() {
            @Override
            public void onChanged(@Nullable final String newName) {
                // Update the UI, in this case, a TextView.
                mNameTextView.setText(newName);
            }
        };

        // Observe the LiveData, passing in this activity as the LifecycleOwner and the observer.
        mModel.getCurrentName().observe(this, nameObserver);
    }
}
``` 

### 更新LiveData中的数据
 在UI线程可以通过setValue来更新数据，在非UI线程通过postValue来个鞥新。LiveData中的setValue 和 postValue方法都是protected的，可以使用子类MutableLiveData，MutableLiveData中将setValue 和 postValue方法发布为public的。  

```java
mModel.getCurrentName().setValue("张三");

``` 

### 扩展LiveData
你可以通过集成LiveData来实现自己的业务逻辑，此时你需要关心onActive(), onInActive() setValue(T)方法，其中onActive onInActive分别额在UI切换为active 和 inActive时触发
