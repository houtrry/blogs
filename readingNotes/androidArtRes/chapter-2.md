# 进程间通信
##  进程间通信方式

### Bundle  
* 优点: 简单易用
* 缺点: 只能传输Bundle支持的数据类型(比如基本类型, 实现了Parcelable接口的对象, 实现了Serializable接口的对象, 以及一些Android支持的特殊对象)
* 适用场景: 四大组件之间的进程间通信

注意: Intent使用Bundle传递数据的数据大小限制问题([江湖问题研究-- intent传递有没有大小限制，是多少](http://blog.csdn.net/wingichoy/article/details/50679322))

### 文件共享
* 优点: 简单易用
* 缺点: 不适合高并发的场景, 并且无法做到进程间的及时通信
* 适用场景: 无并发访问情形, 交换简单的数据, 实时性不高的场景

### AIDL
* 优点: 功能强大, 支持一对多并发通信, 支持实时通信
* 缺点: 使用稍复杂, 需要处理好线程同步
* 适用场景: 一对多通信, 切有RPC需求

### Messenger
* 优点: 功能一般, 支持一对多并发通信, 支持实时通信
* 缺点: 不能很好处理高并发情形, 不支持RPC, 数据通过Message进行传输, 因此,只能传输Bundle支持的数据类型
* 适用场景: 低并发的一对多即时通信, 无RPC需求, 或者无需要返回结果的RPC需求

### ContentProvider
* 优点: 在数据源访问方面功能强大, 支持一对多并发数据共享, 可通过Call方法扩展其他操作.
* 缺点: 可以理解为受约束的AIDL, 主要提供数据源的CRUD操作
* 适用场景: 一对多的进程间的数据共享

### Socket
* 优点: 功能强大, 可以通过网络传输字节流, 支持一对多并发实时通信
* 缺点: 实现细节稍微有点繁琐, 不直接支持的RPC
* 适用场景: 网络数据的交换


## AIDL的使用

目标:  
① 在服务端维护一个图书列表, 在客户端, 可以远程获取这个服务端的列表, 并且可以向列表中添加新的图书.  
② 在客户端监听这个列表, 每当有新的图书添加进来时, 客户端收到这个新书的提醒
### 步骤
1. 服务端:  
	服务端要创建一个Service来监听客户端的连接请求, 然后创建一个AIDL文件, 将暴露给客户端的接口在这个AIDL文件中声明, 最后在Service中实现这个AIDL接口即可.
2. 客户端:  
	客户端首先要绑定服务端的Service, 绑定成功后, 将服务端返回的Binder对象转成AIDL接口所属的类型, 接着就可以调用AIDL中的方法了.
3. 
首先, 在app下的build.gradle文件中添加你的aidl文件路径  

```
		    sourceSets {
		        main {
		            aidl.srcDirs = ['src/main/java']
		        }
		    }
```

完整的如下:  
```

		apply plugin: 'com.android.application'
		
		android {
		    compileSdkVersion 26
		    buildToolsVersion '26.0.2'
		    defaultConfig {
		        applicationId "com.houtrry.aidlsamples"
		        minSdkVersion 15
		        targetSdkVersion 26
		        versionCode 1
		        versionName "1.0"
		        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
		    }
		    buildTypes {
		        release {
		            minifyEnabled false
		            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
		        }
		    }
		    /**
		     * !!!核能预警, 下面的这些不能少
		     */
		    sourceSets {
		        main {
		            aidl.srcDirs = ['src/main/java']
		        }
		    }
		
		}
		
		dependencies {
		    compile fileTree(include: ['*.jar'], dir: 'libs')
		    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
		        exclude group: 'com.android.support', module: 'support-annotations'
		    })
		    compile 'com.android.support:appcompat-v7:26.+'
		    compile 'com.android.support.constraint:constraint-layout:1.0.2'
		    testCompile 'junit:junit:4.12'
		}


```


①创建数据类Book.java并实现Parcelable接口    

```

		public class Book implements Parcelable {
		
		    private int bookId;
		    private String bookName;
		
		    public Book() {
		    }
		
		    public Book(int bookId, String bookName) {
		        this.bookId = bookId;
		        this.bookName = bookName;
		    }
		
		    public int getBookId() {
		        return bookId;
		    }
		
		    public void setBookId(int bookId) {
		        this.bookId = bookId;
		    }
		
		    public String getBookName() {
		        return bookName;
		    }
		
		    public void setBookName(String bookName) {
		        this.bookName = bookName;
		    }
		
		    @Override
		    public String toString() {
		        return "Book{" +
		                "bookId=" + bookId +
		                ", bookName='" + bookName + '\'' +
		                '}';
		    }
		
		    @Override
		    public int describeContents() {
		        return 0;
		    }
		
		    @Override
		    public void writeToParcel(Parcel dest, int flags) {
		        dest.writeInt(this.bookId);
		        dest.writeString(this.bookName);
		    }
		
		    protected Book(Parcel in) {
		        this.bookId = in.readInt();
		        this.bookName = in.readString();
		    }
		
		    public static final Parcelable.Creator<Book> CREATOR = new Parcelable.Creator<Book>() {
		        @Override
		        public Book createFromParcel(Parcel source) {
		            return new Book(source);
		        }
		
		        @Override
		        public Book[] newArray(int size) {
		            return new Book[size];
		        }
		    };
		}


```     

②创建Book.aidl文件, 用来声明Book.Java    
    

```

		// Book.aidl
		package com.houtrry.aidlsamples.aidl;
		
		// Declare any non-default types here with import statements
		
		parcelable Book;

```

注意:   
1>AIDL支持的数据类型  
* 基本数据类型(int, long, char, boolean, double等)
* String和CharSequence
* List: 只支持ArrayList, 里面的每个元素都能够被AIDL支持
* Map: 只支持HashMap, 里面的每个元素都能够被AIDL支持
* Parcelable: 所有实现了Parcelable接口的对象
* AIDL: 所有的AIDL接口本身也可以在AIDL文件中使用

2> 自定义的Parcelable对象和AIDL对象必须要显式import进来, 不管他们是否和当前的AIDL文件位于同一个包下.
3> 如果AIDL文件中用到了自定义的Parcelable对象, 那么必须新建一个和它同名的AIDL文件, 并在其中声明它为Parcelable类型, 比如上面的Book.aidl
4> AIDL中除了基本的数据类型外, 其他类型的参数必须标上方向: in/out/inout. 
	in/out/inout都是针对方法参数而言的. 
	in: 服务端将会接收到客户端传来完整的数据, 但是客户端的那个对象不会因为服务端对接收到的参数的修改而发生改变.  
	out: 服务端将会接收到客户端传来的数据为空, 尽管客户端实际传来的参数不是空, 但是服务端对接收到的参数的修改(比如说new一个新的)都会影响客户端传来的那个参数.  
	inout: 服务端将会接收到客户端传来完整的数据, 但是客户端会同步服务端对接收到的参数的修改.  
5> AIDL接口中只支持方法, 不支持声明静态常量
6> AIDL的包结构在服务端和客户端要保持一致, 否则会报错.  
③ 声明服务端提供给客户端的接口方法

```   

		// IBookManager.aidl
		package com.houtrry.aidlsamples.aidl;
		
		import com.houtrry.aidlsamples.aidl.Book;
		import com.houtrry.aidlsamples.aidl.INewBookArrivedListener;
		// Declare any non-default types here with import statements
		
		interface IBookManager {
		    List<Book> getBookList();
		    void addBook(in Book book);
		    boolean addNewBookArrivedListener(in INewBookArrivedListener newBookArrivedListener);
		    boolean removeNewBookArrivedListener(in INewBookArrivedListener newBookArrivedListener);
		}

```

④ 实现服务端Service
参见BookManagerRemoteService.java

```

		public class BookManagerRemoteService extends Service {
		
		    private static final String TAG = BookManagerRemoteService.class.getSimpleName();
		
		    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();
		    //    private CopyOnWriteArrayList<INewBookArrivedListener> mINewBookArrivedListeners = new CopyOnWriteArrayList<>();
		
		    /**
		     * !!!重要
		     * 这里为什么要使用RemoteCallbackList!!!
		     * 因为客户端调用服务端的方法的时候, 比如说removeNewBookArrivedListener(INewBookArrivedListener newBookArrivedListener), 传的参数newBookArrivedListener
		     * 与我们在服务端的removeNewBookArrivedListener方法里接收到的参数newBookArrivedListener并不是同一个
		     * 可以参考AIDLSamples\app\build\generated\source\aidl\debug\com\houtrry\aidlsamples\aidl\IBookManager.java
		     * 服务端接收到的是一个新的INewBookArrivedListener对象.
		     * 这就导致了一个问题, 如果用CopyOnWriteArrayList, mINewBookArrivedListeners.contains(newBookArrivedListener)会返回FALSE
		     */
		    private RemoteCallbackList<INewBookArrivedListener> mINewBookArrivedListeners = new RemoteCallbackList<>();
		
		    private AtomicBoolean isDestroy = new AtomicBoolean();
		
		    @Nullable
		    @Override
		    public IBinder onBind(Intent intent) {
		        return mIBookManager;
		    }
		
		    @Override
		    public void onCreate() {
		        super.onCreate();
		        isDestroy.set(false);
		        mBookList.add(new Book(0x0061, "唐吉坷德"));
		        mBookList.add(new Book(0x0062, "堕落"));
		        mBookList.add(new Book(0x0063, "尤利西斯"));
		
		        new Thread(new ServiceWorkRunner()).start();
		    }
		
		    @Override
		    public void onDestroy() {
		        isDestroy.set(true);
		        super.onDestroy();
		    }
		
		    private IBookManager.Stub mIBookManager = new IBookManager.Stub() {
		        @Override
		        public List<Book> getBookList() throws RemoteException {
		            Log.d(TAG, "getBookList: ThreadName: " + Thread.currentThread().getName());
		            return mBookList;
		        }
		
		        @Override
		        public void addBook(Book book) throws RemoteException {
		            Log.d(TAG, "addBook: ThreadName: " + Thread.currentThread().getName());
		            if (mBookList != null) {
		                mBookList.add(book);
		            }
		        }
		
		        @Override
		        public boolean addNewBookArrivedListener(INewBookArrivedListener newBookArrivedListener) throws RemoteException {
		            //            if (mINewBookArrivedListeners.contains(newBookArrivedListener)) {
		            //                Log.d(TAG, "addNewBookArrivedListener: has add this listener");
		            //                return false;
		            //            }
		            mINewBookArrivedListeners.register(newBookArrivedListener);
		            return true;
		        }
		
		        @Override
		        public boolean removeNewBookArrivedListener(INewBookArrivedListener newBookArrivedListener) throws RemoteException {
		            //            if (!mINewBookArrivedListeners.contains(newBookArrivedListener)) {
		            //                Log.e(TAG, "addNewBookArrivedListener: don't add this listener");
		            //                return false;
		            //            }
		
		            /**
		             * mINewBookArrivedListeners.beginBroadcast()和mINewBookArrivedListeners.finishBroadcast();必须一起使用
		             */
		
		            Log.d(TAG, "removeNewBookArrivedListener: " + mINewBookArrivedListeners.beginBroadcast());
		            mINewBookArrivedListeners.finishBroadcast();
		            mINewBookArrivedListeners.unregister(newBookArrivedListener);
		            Log.d(TAG, "removeNewBookArrivedListener: " + mINewBookArrivedListeners.beginBroadcast());
		            mINewBookArrivedListeners.finishBroadcast();
		            return true;
		        }
		    };
		
		    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
		        @Override
		        public void binderDied() {
		            if (mIBookManager != null) {
		                mIBookManager.asBinder().unlinkToDeath(mDeathRecipient, 0);
		                mIBookManager = null;
		
		                // TODO: 2017/10/29 在这里重新绑定
		
		
		            }
		        }
		    };
		
		
		    private void arrivedNewBook(Book book) throws RemoteException {
		        mBookList.add(book);
		        //        for (INewBookArrivedListener listener: mINewBookArrivedListeners) {
		        //            listener.onNewBookArrived(book);
		        //        }
		
		        final int length = mINewBookArrivedListeners.beginBroadcast();
		
		        for (int i = 0; i < length; i++) {
		            INewBookArrivedListener listener = mINewBookArrivedListeners.getBroadcastItem(i);
		            if (listener != null) {
		                //针对每一个try-catch的好处是, 一个失败了, 不会影响循环的继续执行
		                try {
		                    listener.onNewBookArrived(book);
		                } catch (RemoteException e) {
		                    e.printStackTrace();
		                }
		            }
		        }
		        mINewBookArrivedListeners.finishBroadcast();
		    }
		
		    private class ServiceWorkRunner implements Runnable {
		
		        @Override
		        public void run() {
		            while (!isDestroy.get()) {
		                try {
		                    Thread.sleep(5000);
		                } catch (InterruptedException e) {
		                    e.printStackTrace();
		                }
		                try {
		                    arrivedNewBook(new Book(new Random().nextInt(10000), "## book name " + System.currentTimeMillis()));
		                } catch (RemoteException e) {
		                    e.printStackTrace();
		                }
		            }
		        }
		    }
		}


```  

⑤ 在AndroidManifest.xml注册BookManagerRemoteService, 并添加进程信息

```

        <service
            android:name=".BookManagerRemoteService"
            android:process=":remote"/>

```

⑥ 客户端的实现
参见MainActivity.java

```

		public class MainActivity extends AppCompatActivity {
		
		    private static final String TAG = MainActivity.class.getSimpleName();
		    private IBookManager mBookManager;
		
		    @Override
		    protected void onCreate(Bundle savedInstanceState) {
		        super.onCreate(savedInstanceState);
		        setContentView(R.layout.activity_main);
				//绑定远程服务端
		        Intent intent = new Intent(this, BookManagerRemoteService.class);
		        bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
		    }
		
		    private ServiceConnection mServiceConnection = new ServiceConnection() {
		        @Override
		        public void onServiceConnected(ComponentName name, IBinder service) {
		            Log.d(TAG, "onServiceConnected: ThreadName: "+Thread.currentThread().getName());
					//将服务端返回的IBinder对象转为AIDL接口对象, 通过AIDL接口对象调用接口方法
		            IBookManager bookManager = IBookManager.Stub.asInterface(service);
		            mBookManager = bookManager;
		            try {
		                bookManager.addNewBookArrivedListener(mINewBookArrivedListener);
		            } catch (RemoteException e) {
		                e.printStackTrace();
		            }
		            List<Book> bookList;
		            try {
		                bookList = bookManager.getBookList();
		                Log.d(TAG, "onServiceConnected: bookList: "+bookList);
		            } catch (RemoteException e) {
		                e.printStackTrace();
		            }
		
		            try {
		                bookManager.addBook(new Book(0x0100, "唐璜"));
		            } catch (RemoteException e) {
		                e.printStackTrace();
		            }
		
		            try {
		                bookList = bookManager.getBookList();
		                Log.d(TAG, "onServiceConnected: bookList: "+bookList);
		            } catch (RemoteException e) {
		                e.printStackTrace();
		            }
		        }
		
		        @Override
		        public void onServiceDisconnected(ComponentName name) {
		            mBookManager = null;
		            Log.d(TAG, "onServiceDisconnected: ");
		        }
		    };
		
		    @Override
		    protected void onDestroy() {
		        removeNewBookArrivedListener();
		
		        unbindService(mServiceConnection);
		        super.onDestroy();
		    }
		
		    private void removeNewBookArrivedListener() {
		        if (mBookManager == null || !mBookManager.asBinder().isBinderAlive()) {
		            Log.e(TAG, "removeNewBookArrivedListener: mBookManager: "+mBookManager);
		            return;
		        }
		        try {
		            mBookManager.removeNewBookArrivedListener(mINewBookArrivedListener);
		        } catch (RemoteException e) {
		            e.printStackTrace();
		        }
		    }
		
		    private INewBookArrivedListener mINewBookArrivedListener = new INewBookArrivedListener.Stub(){
		
		        @Override
		        public void onNewBookArrived(Book book) throws RemoteException {
		            Log.d(TAG, "onNewBookArrived: ThreadName: "+Thread.currentThread().getName());
		            Log.d(TAG, "onNewBookArrived: book: "+book);
		        }
		    };
		
		    public void toService2(View view) {
		        Intent intent = new Intent(this, Main2Activity.class);
		        startActivity(intent);
		    }
		}

```
### 需要注意的问题

#### 线程

1. 由于客户端的onServiceConnected和onServiceDisconnected方法都运行在UI线程中, 所以不可以在他们里面直接调用服务端的耗时方法.
2. 服务端的方法本身就运行在服务端的Binder线程池中, 所以服务端方法本身就可以执行大量耗时操作, 这个时候切记不要再服务端方法中开线程去进行异步任务, 除非拟明确知道自己在干什么

#### RemoteCallbackList

#### Binder意外停止, 重新连接服务
方式有两种: 
1. 设置DeathRecipient监听, binderDied在服务端的Binder线程池中被调用
2. 在onServiceDisconnected中重连远程服务, onServiceDisconnected运行在客户端的UI线程中

#### 注意INewBookArrivedListener的实现方式


## 扩展

* [Binder学习指南](http://weishu.me/2016/01/12/binder-index-for-newer/)
* [Android Binder之应用层总结与分析](http://blog.csdn.net/qian520ao/article/details/78089877)
* [为什么 Android 要采用 Binder 作为 IPC 机制？](https://www.zhihu.com/question/39440766/answer/89210950)