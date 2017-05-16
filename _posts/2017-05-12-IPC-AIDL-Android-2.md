---
layout: post
title: IPC之AIDL(2)in out inout
description: IPC之AIDL(2)in out inout
categories: android
keywords: android, IPC, AIDL, in, out, inout
---
内容大纲：  
<pre>
1.在AIDL的时候正确使用in out inout
</pre>

上一篇我们用AIDL简单实现了一个IPC，其中我们谈到在定义aidl接口中的除基本类型和AIDL接口外的参数要调价修饰符in out 或 inout中的一种，本文将帮助大家理解in out inout，并让读者可以正确的使用in out inout。  
在介绍in out inout的区别之前我们先明确两个基本概念：起点 和 终点，起点指调用方，终点指响应方，比如我在客户端调用aidl接口那么客户端就是起点 服务端就是重点，在一次调用中服务端如果要调用一个aidl接口回调给客户端，那么服务端就是起点，客户端就是重点。  
然后我们来定义in out inout:  
in ： 将对象从起点传递给终点，在终点部分中对对象的修改不会反映到起点，即只输入  
out : 对象中的值不会传递给终点，但是在终点部分对对象的修改会反映到起点，即只输出  
intout : 将对象从起点传递给终点，在终点部分的修改会反应到起点，即输入输出都有影响  

我们再用一个具体的例子来看一下这个区分：
我们来看下客户端的代码：  
<code>
<pre>
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
</pre>
</code>

我们输出了callback 调用addBook 和调用后book对象的名称。我们再来看下服务端的代码：  
<code>
<pre>
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
</pre>
</code>

观察addBook方法 我们在客户端传过来的book中修改了name字段 追加了一个：server，然后我们输出了客户端传递过来的book名称。现在我们来看在book的修饰符分别为in out 和inout时候的系统输出。（我们假设BookListener中的参数都是in,其实BookListener中的in out inout修饰就是起点 和 终点的转换，这个时候起点是调用方服务端 终点是响应方客户端，具体的读者可自行分析）  

in:   
 我们可以先猜测一下输出，in表示数据会传递到终点，那么服务端会输出TestBook，然后由于对对象的修改不会反映到起点，所以对象不会反映到客户端，客户端依然输出的是TestBook，而callback中的回调是TestBook : server,我们来看下实际效果：  

![](/images/aidl2-1.png)  

out:  
out不会将数据传递到终点在这里也就是服务端，那么服务端会输出null， 然后由于对象修改会反应到起点也就是客户端，所以客户端会输出 null : server,我们来看下实际效果：  
![](/images/aidl2-2.png)  

inout:
inout即会把数据传递到终点，起点也会响应终点的变化，那么服务端会输出 TestBook
然后客户端输出 TestBook : server, 我们来看下实际效果：  
![](/images/aidl2-3.png)  

