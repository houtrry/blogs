###首先, 贴上IntentService的源码
```
public abstract class IntentService extends Service {
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;
    private String mName;
    private boolean mRedelivery;

    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }

    public IntentService(String name) {
        super();
        mName = name;
    }

    public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }

    @Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        mServiceLooper.quit();
    }

    @Override
    @Nullable
    public IBinder onBind(Intent intent) {
        return null;
    }

    @WorkerThread
    protected abstract void onHandleIntent(@Nullable Intent intent);
}
```
其中, 有一个关键类HandlerThread
```
public class HandlerThread extends Thread {
    int mPriority;
    int mTid = -1;
    Looper mLooper;

    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }
    
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }
    
    protected void onLooperPrepared() {
    }

    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }

    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        
        // If the thread has been started, wait until the looper has been created.
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }

    public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

    public int getThreadId() {
        return mTid;
    }
}
```
###分析
1.IntentService.onCreate  
```
HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
thread.start();
```
方法中创建了HandlerThread, 而HandlerThread继承自Thread, 即IntentService.onCreate方法中创建了一个子线程, 并开启了子线程.
```
mServiceLooper = thread.getLooper();
mServiceHandler = new ServiceHandler(mServiceLooper);
```
将子线程的Looper交给内部类ServiceHandler, 而ServiceHandler继承自Handler.  
于是, 先来查看下HandlerThread到底做了什么.
2.HandlerThread继承自Thread, 那先看下run方法
```
    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
```
在run方法中, 我们看到了Looper.prepare()方法, 了解Handler的都知道, 在子线程中创建Handler, 需要先调用Looper.prepare()方法在子线程中创建looper.那这里run方法就清晰了, run方法创建了looper并且调用了Looper.loop()方法轮询Looper中的消息队列.  
3.知道了ServiceHandler的looper来自于HandlerThread中创建的looper后, 我们可以知道, ServiceHandler.handleMessage将会执行在线程ServiceHandler中.  
```
        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
```
handleMessage中有onHandleIntent((Intent)msg.obj);和stopSelf(msg.arg1);  
onHandleIntent((Intent)msg.obj)方法就是我们要具体实现的方法, 我们实现这个方法, 将需要在子线程中完成的事情放到这里.  
stopSelf(msg.arg1)方法就是停止service.  
4.那什么时候执行handleMessage方法中的内容呢?  
```
    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }
```
  在IntentService#onStart方法中, mServiceHandler发出消息, 开始在子线程HandlerThread中执行onHandleIntent((Intent)msg.obj)和stopSelf(msg.arg1)  
###总结
1.在IntentService#onCreate方法中  
   ①创建子线程, 并在子线程中创建looper, 调用Looper.loop()方法轮询消息.   
   ②用子线程中创建的Looper创建Handler.  
2.在IntentService#onStart方法中, 通过mServiceHandler发送Message, 触发在子线程(HandlerThread所在线程)中执行IntentService#onHandleIntent方法和IntentService#stopSelf来执行任务, 并在任务结束的时候停止Service.  
3.IntentService#onDestroy方法中, 执行mServiceLooper.quit();即, 停止loop的轮询.轮询停止后, 子线程HandlerThread也就结束了.

