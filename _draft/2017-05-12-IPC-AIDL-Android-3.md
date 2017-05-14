#IPC之AIDL(3)系统为我们做了什么
内容大纲：
<pre>
1.了解基本的aidl原理（不涉及底层）
</pre>

前面几篇我们介绍了如何使用AIDL实现IPC 那么你会有疑问了 我们用的asInterface是什么， Stub又是什么，现在让我们一点一点来看。在我们编译的时候，系统会吧aidl生成对应的java类（这就是为什么支持aidl这种文件格式了），我们先来看一下系统生成的类(本文涉及到的系统源码都是基于android-23)：  
<code>
<pre>
package com.wlh.animation.ipctest;
// Declare any non-default types here with import statements

public interface IBookManager extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.wlh.animation.ipctest.IBookManager {
        private static final java.lang.String DESCRIPTOR = "com.wlh.animation.ipctest.IBookManager";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.wlh.animation.ipctest.IBookManager interface,
         * generating a proxy if needed.
         */
        public static com.wlh.animation.ipctest.IBookManager asInterface(android.os.IBinder obj) {
    		...
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags)
                throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                   ...
                }
                case TRANSACTION_getBookList: {
                   ...
                }
                case TRANSACTION_addBook: {
                  ...
                }
                case TRANSACTION_registerListener: {
                   ...
                }
                case TRANSACTION_unRegisterListener: {
                   ...
                }
            }
            return super.onTransact(code, data, reply, flags);
        }

        private static class Proxy implements com.wlh.animation.ipctest.IBookManager {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public java.util.List<com.wlh.animation.ipctest.Book> getBookList() throws android.os.RemoteException {
       			...
            }

            @Override
            public void addBook(com.wlh.animation.ipctest.Book book) throws android.os.RemoteException {
     			...
            }

            @Override
            public void registerListener(com.wlh.animation.ipctest.IBookListener listener)
                    throws android.os.RemoteException {
               		...
            }

            @Override
            public void unRegisterListener(com.wlh.animation.ipctest.IBookListener listener)
                    throws android.os.RemoteException {
          		...
            }
        }

        static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
        static final int TRANSACTION_registerListener = (android.os.IBinder.FIRST_CALL_TRANSACTION + 2);
        static final int TRANSACTION_unRegisterListener = (android.os.IBinder.FIRST_CALL_TRANSACTION + 3);
    }

    public java.util.List<com.wlh.animation.ipctest.Book> getBookList() throws android.os.RemoteException;

    public void addBook(com.wlh.animation.ipctest.Book book) throws android.os.RemoteException;

    public void registerListener(com.wlh.animation.ipctest.IBookListener listener) throws android.os.RemoteException;

    public void unRegisterListener(com.wlh.animation.ipctest.IBookListener listener) throws android.os.RemoteException;
}
</pre>
</code>

我们先忽略方法的具体实现，来看一下这个系统为我们生成的类中都有哪些东西：  
1.整个文件是一个IBookManager(你aidl文件名字)的接口 继承IInterface  
2.一个抽象类Stub继承Binder（前面我们说过  Binder有跨进程的能力）实现了这个IBookManager接口  
3.一个代理类实现了IBookManager(注意没有继承Binder)  
然后我们依次来解析一下这写类：  
##IBookManager IInterface 
首先是IBookManager,它是一个继承自IInterface的接口，我们先来看下IInterface接口中有哪些东西：  
<pre>
<code>
public interface IInterface
{
    public IBinder asBinder();
}
</code>
</pre>
我们可以看到IInterface中只有一个方法 就是asBinder 返回一个IBinder,IBinder也是一个接口。

##Stub Binder
Stub是一个抽象类继承了Binder 实现了IBookManager接口

###Binder
Binder实现了IBinder，我们来看下IBinder中的内容.我们这里只介绍一些重要的变量和方法，其它的说明读者可自行查阅源码中的注释。  
###getInterfaceDescriptor
源码定义如下：  
<pre>
<code>
  /**
     * Get the canonical name of the interface supported by this binder.
     */
    public String getInterfaceDescriptor() throws RemoteException;
</code>
</pre>
这个返回接口的名称，我们在Binder中可以看到它的具体实现：  
<pre>
<code>
   public void attachInterface(IInterface owner, String descriptor) {
        mOwner = owner;
        mDescriptor = descriptor;
    }

      public String getInterfaceDescriptor() {
        return mDescriptor;
    }
</code>
</pre>
我们可以看到系统通过attachInterface来给接口名称赋值，主要是为了作为跨进程通信时候接口的标识，我们可以在Binder的子类Stub也就是系统为我们生成的类中看到具体的接口名称：  
<pre>
<code>
        private static final java.lang.String DESCRIPTOR = "com.wlh.animation.ipctest.IBookManager";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }
</code>
</pre>
我们可以看到其实就是接口的全路径。  

###pingBinder, isBinderAlive, 
<pre>
<code>
  /**
     * Check to see if the object still exists.
     * 
     * @return Returns false if the
     * hosting process is gone, otherwise the result (always by default
     * true) returned by the pingBinder() implementation on the other
     * side.
     */
    public boolean pingBinder();
</code>
</pre>
就是字面意思ping一下 来确认Binder是否可以链接到Binder，只有当宿主进程不存在的时候才返回false。  

<pre>
<code>
   /**
     * Check to see if the process that the binder is in is still alive.
     *
     * @return false if the process is not alive.  Note that if it returns
     * true, the process may have died while the call is returning.
     */
    public boolean isBinderAlive();
</code>
</pre>
返回Binder是否存活。如果进程不是存活状态，那么返回false。  


###queryLocalInterface

<pre>
<code>
 /**
     * Attempt to retrieve a local implementation of an interface
     * for this Binder object.  If null is returned, you will need
     * to instantiate a proxy class to marshall calls through
     * the transact() method.
     */
    public IInterface queryLocalInterface(String descriptor);
</code>
</pre>
根据接口的描述返回一个本地接口，如果返回的是null的话（就是跨进程），需要你去实现代理（在使用aidl的时候系统为我们已经实现好了）  


###transact
<pre>
<code>
/**
     * Perform a generic operation with the object.
     * 
     * @param code The action to perform.  This should
     * be a number between {@link #FIRST_CALL_TRANSACTION} and
     * {@link #LAST_CALL_TRANSACTION}.
     * @param data Marshalled data to send to the target.  Must not be null.
     * If you are not sending any data, you must create an empty Parcel
     * that is given here.
     * @param reply Marshalled data to be received from the target.  May be
     * null if you are not interested in the return value.
     * @param flags Additional operation flags.  Either 0 for a normal
     * RPC, or {@link #FLAG_ONEWAY} for a one-way RPC.
     */
    public boolean transact(int code, Parcel data, Parcel reply, int flags)
        throws RemoteException;
</code>
</pre>
这个基本上是IBinder中最重要的一个函数了，它用来相应对象的操作，例如我们本例中的一个addBook操作，它的参数主要有如下几个：  
1.code 一次操作的唯一标识，要介于常量FIRST_CALL_TRANSACTION（ 0x00000001）和 LAST_CALL_TRANSACTION（0x00ffffff）之间  

2.data 需要传递的数据不能为空   

3.reply 返回的数据  

4.flats 附加的操作标识，通常返回0. 如果设置成FLAG_ONEWAY表示呼叫方不会等待被呼叫方放回结果（只在跨进程的时候生效）。  


我们再来看下Stub中的具体实现：  

###asInterface
<pre>
<code>
     public static com.wlh.animation.ipctest.IBookManager asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.wlh.animation.ipctest.IBookManager))) {
                return ((com.wlh.animation.ipctest.IBookManager) iin);
            }
            return new com.wlh.animation.ipctest.IBookManager.Stub.Proxy(obj);
        }
</code>
</pre>
将IBinder转换成我们需要的接口，我们可以看到基本的逻辑是：如果从本地找到了接口就返回本地接口（没有跨进程），否则返回Stub的代理类，关于代理类我们稍后再说，这个逻辑规则基本贯穿了Stub中的所有方法：即先找本地，如果没有（跨进程）则返回代理。  

###onTransact
<pre>
<code>
 @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags)
                throws android.os.RemoteException {


   switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_addBook: {
                   。。。
                }
}
}
</code>
</pre>
首先在Stub类中 系统为我们的每一个方法值都赋予了一个id
<pre>
    static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
        static final int TRANSACTION_registerListener = (android.os.IBinder.FIRST_CALL_TRANSACTION + 2);
        static final int TRANSACTION_unRegisterListener = (android.os.IBinder.FIRST_CALL_TRANSACTION + 3);
</pre>
可以看到是基于TRANSACTIOn_{$method name}来命名的参数是基于FIRST_CALL_TRANSACTION递增的。然后再onTransaction方法中系统会switch code判断当前调用的是哪个方法然后做相应的操作 我们来看下addBook的具体实现：  
<pre>
<code>
  case TRANSACTION_addBook: {
    data.enforceInterface(DESCRIPTOR);
                    com.wlh.animation.ipctest.Book _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = com.wlh.animation.ipctest.Book.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.addBook(_arg0);
                    reply.writeNoException();
                    if ((_arg0 != null)) {
                        reply.writeInt(1);
                        _arg0.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    return true;
	}
</code>
</pre>  
我们不难看出，基本思路就是将对象和Parceable之间转换处理。  

我们再来看下代理类Proxy:  
Proxy是实现了接口IBookManager的代理类，其构造方法如下：  
<pre>
<code>
  Proxy(android.os.IBinder remote) {
                mRemote = remote;
   }
</code>
</pre>

构造函数是一个IBinder,之后代理中的相关调用都会转到这个Binder中去处理，我们还是来看一下addBookManager接口：  
<pre>
<code>
     @Override
            public void addBook(com.wlh.animation.ipctest.Book book) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((book != null)) {
                        _data.writeInt(1);
                        book.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_addBook, _data, _reply, 0);
                    _reply.readException();
                    if ((0 != _reply.readInt())) {
                        book.readFromParcel(_reply);
                    }
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
</code>
</pre>

可以看到和onTransact中最大的区别在于 它将具体的处理时间交给了跨进程的Binder处理。  

以上就是在我们编写AIDL文件后 系统为我们生成文件的解析。




