---
layout: post
title: IPC之AIDL(1)实现AIDL
description: AIDL实现 
categories: Android
keywords: IPC, Android, AIDL 
---

内容大纲
<pre>
1.使用AIDL实现IPC
</pre>
 
本文是基于Android studio来说明的。我们先明确2个基本概念客户端和服务端。在本例中客户端是指发起处理请求的进程（app的主进程）， 服务端（service端）指一个另外一个提供服务的进程（主要是指service）。  

##AIDL文件
AIDL 全程 Android Interface Definition Language，即接口定义语言。Android通过aidl文件将服务的接口公开，来实现跨进程的调用。   
在Android studio中创建aidl文件，你只需要右键->新建 ->aidl即可，其目录结构基本如下：  
![](/images/ipc-aidle1-1.png)  
 
我们来明确一些AIDL的规则：  
1.aidl文件中的参数只能从以下类型选择： 基本数据类型， parcelable的类， Aidl接口  
2.即使在同一个包下，aidl文件也必须指明import  
3.parcelable的类作为方法参数时要指明in out inout  
4.接口中使用到的Parcelable类 必须建立同名的aidl文件，包名必须与实际的java类文件相同  

我们来看一个具体的例子：  
IBookManager.aidl：  
<pre>
<code>
package com.wlh.animation.ipctest;
import com.wlh.animation.ipctest.Book;
import com.wlh.animation.ipctest.IBookListener;

interface IBookManager {

    List<Book> getBookList();
    void addBook(in Book book);

    void registerListener(IBookListener listener);

    void unRegisterListener(IBookListener listener);
}
</code>
</pre>

Book.aidl:
<pre>
<code>
package com.wlh.animation.ipctest;
parcelable Book;
</code>
</pre>
在例子中我们定义了一个IBookManager的接口来实现进程通信，它又4个方法 getBookList()(不带参数)， add Book(in Book book)带Parcelable参数必须指明in out inout中的一个，registerListener 和 unRegisterListener，也是Aidl的接口。
写完aidl文件后我们 make一下可以看到系统为我们生成的类：  
![](/images/ipc-aidle1-2.png)  
关于系统为我们做了什么 我们在下一篇中再进行详细的介绍，现在我们来看一下服务端的代码实现：  
<pre>
<code>
public class AIDLService extends Service{
    RemoteCallbackList<IBookListener> mCallbacks = new RemoteCallbackList<>();
    private IBinder mBinder = new IBookManager.Stub() {
        @Override
        public List<Book> getBookList() throws RemoteException {
            Log.i("wlh", "getBookList");
            return null;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            Log.i("wlh", "addBook : " + book.name);
            int N = mCallbacks.beginBroadcast();
            book.name += " : server";
            for (int i = 0; i < N; i++ ) {
                mCallbacks.getBroadcastItem(i).onBookAdd(book);
            }
            mCallbacks.finishBroadcast();

        }

        @Override
        public void registerListener(IBookListener listener) throws RemoteException {
            mCallbacks.register(listener);
        }

        @Override
        public void unRegisterListener(IBookListener listener) throws RemoteException {
            mCallbacks.unregister(listener);
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
}
</code>
</pre>
我们只要实现IBookManager.Stub 然后再onBind方法中返回即可，我们再来看下客户端是如何实现的：  
<pre>
<code>
        mServiceConnection = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                mBookManager = IBookManager.Stub.asInterface(service);
                try {
                    mBookManager.registerListener(new IBookListener.Stub() {
                        @Override
                        public void onBookAdd(Book book) throws RemoteException {
                            Log.i("wlh " , "addBook callback : " + book.name);
                        }
                    });
                    Book book = new Book();
                    book.name = "TestBook";
                    mBookManager.addBook(book);
                    Log.i("wlh",  book.name);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
                unbindService(mServiceConnection);
            }

            @Override
            public void onServiceDisconnected(ComponentName name) {

            }
        };
      
</code>
</pre>
我们以Bind的方式启动Service然后再onServiceConnected中的使用IBookManager.Stub.asInterface来转换成我们定义的接口即可实现和服务端的交互。 
