---
layout: post
title: IPC之Messenger
description: IPC之Messenger
categories: android
keywords: android, IPC, Messenger
---

内容大纲：
<pre>
1.学会使用Messenger进行进程间通信
</pre>  


在进行具体的探讨之前，我们先明确2个基本概念客户端和服务端。在本例中客户端是指发起处理请求的进程（app的主进程）， 服务端（service端）指一个另外一个提供服务的进程（主要是指service）。

Messenger可以理解为信使，它可以实现消息的发送，底层使用Binder实现的（所以可以跨进程）。我们来看Messenger的构造函数：  
<pre>
<code>
   public Messenger(IBinder target) {
        mTarget = IMessenger.Stub.asInterface(target);
    }

   public Messenger(Handler target) {
        mTarget = target.getIMessenger();
    }
</code>
</pre>
Messenger的构造函数主要有2个 一个 接受一个IBinder参数 一个接受一个Handler参数，我们可以看到它们参数的名称都是叫target，从字面上我们也不难理解，这个参数的作用就是定义消息发送的目标。我们看下Handler中MessengerImpl的具体实现：  
<pre>
<code>
    private final class MessengerImpl extends IMessenger.Stub {
        public void send(Message msg) {
            msg.sendingUid = Binder.getCallingUid();
            Handler.this.sendMessage(msg);
        }
    }
</code>
</pre>
我们可以看到在发送消息的时候，给msg添加了一个属性，CallingUid,这个CallingUid又是什么呢，我们来看一下Binder中的注释：  
<pre>
<code>
  /**
     * Return the Linux uid assigned to the process that sent you the
     * current transaction that is being processed.  This uid can be used with
     * higher-level system services to determine its identity and check
     * permissions.  If the current thread is not currently executing an
     * incoming transaction, then its own uid is returned.
     */
    public static final native int getCallingUid();
</code>
</pre>
不难发现这个UId其实就是调用方进程的UID(用户身份的标识符)。
那么这个IMessenger.Stub是什么呢？这里我们不加讨论，在AIDL的实现中我们会重点讨论这个。  
我们接着再来看下如何用Messenger实现IPC，我们先看下客户端的实现:    
<pre>
<code>
     mServiceConnection = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                Messenger messenger = new Messenger(service);
                Message message = new Message();
                Bundle data = new Bundle();
                data.putString("info", "this is from client");
                message.setData(data);
                try {
                    messenger.send(message);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
                unbindService(mServiceConnection);
            }

            @Override
            public void onServiceDisconnected(ComponentName name) {

            }
        };
        bindService(new Intent(this, MessengerService.class), mServiceConnection, Context.BIND_AUTO_CREATE);
</code>
</pre>
我们在onServiceConnected中用返回的IBinder new一个Messenger（消息的发送者），然后创建Message直接send即可，那么服务端又是如何接收处理这个消息的呢：  
<pre>
<code>
public class MessengerService extends Service{
    Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            Log.i("wlh", "service : " + msg.getData().getString("info"));
        }
    };
    Messenger messenger = new Messenger(handler);
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return messenger.getBinder();
    }
}
</code>
</pre>
服务端的实现也很简单 定义一个Handler 和 Messenger（消息的接收者，交给Handler处理） 然后返回messenger的getBinder即可。这样我们就实现了进程的单向通信，那么如何实现双向通信呢，其实也很简单我们只要给message添加一个replyTo的对象（也是一个Messenger）然后使用这个replyTo就可以进行消息的应答。其实我们可以思考一下如果我们需要实现双向沟通是不是需要一个发送者来发送消息， 一个接收者来接受消息。我们来一下基于上面单向通信的改进，首先来看客户端的实现：  
<pre>
<code>
  	Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            Log.i("wlh", "reply info : " + msg.getData().getString("info"));
        }
    };

    Messenger receiveMessenger = new Messenger(handler);

     mServiceConnection = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName name, IBinder service) {
                Messenger messenger = new Messenger(service);
                Message message = new Message();
                Bundle data = new Bundle();
                data.putString("info", "this is from client");
                message.setData(data);
				message.replyTo = receiveMessenger;
                try {
                    messenger.send(message);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
                unbindService(mServiceConnection);
            }

            @Override
            public void onServiceDisconnected(ComponentName name) {

            }
        };
        bindService(new Intent(this, MessengerService.class), mServiceConnection, Context.BIND_AUTO_CREATE);
</code>
</pre>
我们可以发现在单向通信的基础上我们添加了一个接收者 receiverMessenter, 然后将其赋值给msg的replyTo， 服务端就可以取到这个Messenger来进行消息回复。  
我们再来看下服务端的改进：
<pre>
<code>
    Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            Log.i("wlh", "service : " + msg.getData().getString("info"));
            Message replyMsg = new Message();
            Bundle data = new Bundle();
            data.putString("info", " this is from server");
            replyMsg.setData(data);
            try {
                msg.replyTo.send(replyMsg);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }
    };
</code>
</pre>
我们只需要通过msg.replyTo来发送消息回复就可以了，是不是很简单。
