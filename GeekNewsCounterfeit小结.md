###写在前面  
GeekNewsCounterfeit项目是模仿的[GeekNews项目](https://github.com/codeestX/GeekNews), 本篇文章是学习过程中的小结.  

----------------
###.gradle文件的配置.  
1.新建一个config.gradle文件, 将配置信息全部放到config.gradle中, 在project的gradle文件中添加
```
apply from: "config.gradle"
```
来依赖config.gradle文件  
2.config.gradle文件如下
```
ext {
    android = [
            compileSdkVersion: 25,
            buildToolsVersion: "26.0.0",
            applicationId    : "com.houtrry.geeknewscounterfeit",
            minSdkVersion    : 15,
            targetSdkVersion : 25,
            versionCode      : 1,
            versionName      : "1.0",
    ]

    def depenVersion = [
            support    : "25.3.1",
            constraint : "1.0.2",

            //rxjava相关
            rxjava     : "2.1.3",
            rxandroid  : "2.0.1",

            //dagger
            dagger     : "2.11",
            //butterknife
            butterknife: "8.8.1",

            //glide
            glide      : "4.0.0",

            //网络请求相关
            gson       : "2.8.0",
            retrofit   : "2.3.0"
    ]

    dependencies = [
            "appcompat-v7"        : "com.android.support:appcompat-v7:${depenVersion.support}",
            "constraint-layout"   : "com.android.support.constraint:constraint-layout:${depenVersion.constraint}",
            "rxjava"              : "io.reactivex.rxjava2:rxjava:${depenVersion.rxjava}",
            'rxandroid'           : "io.reactivex.rxjava2:rxandroid:${depenVersion.rxandroid}",

            "dagger"              : "com.google.dagger:dagger:${depenVersion.dagger}",
            "dagger-compiler"     : "com.google.dagger:dagger:${depenVersion.dagger}",

            "butterknife"         : "com.jakewharton:butterknife:${depenVersion.butterknife}",
            "butterknife-compiler": "com.jakewharton:butterknife-compiler:${depenVersion.butterknife}",

            "glide"               : "com.github.bumptech.glide:glide:${depenVersion.glide}",
            "support-v4"          : "com.android.support:support-v4:${depenVersion.support}",
            "glide:compiler"      : "com.github.bumptech.glide:compiler:${depenVersion.glide}",

            "gson"                : "com.google.code.gson:gson:${depenVersion.gson}",
            "retrofit"            : "com.squareup.retrofit2:retrofit:${depenVersion.retrofit}",
    ]
}
```
最外层是ext, 在app的.gradle中, 使用的方式是
```
    compileSdkVersion rootProject.ext.android.compileSdkVersion
    buildToolsVersion rootProject.ext.android.buildToolsVersion
    ...
    compile rootProject.ext.dependencies["rxjava"]
    compile rootProject.ext.dependencies["rxandroid"]
    ...
```
###App启动速度优化
1.Application的优化.   
尽可能地将第三方库的初始化放到子线程中, 而不是直接放到Application.onCreate方法中.  
例如, 使用IntentService来初始化.  
```
public class InitializeService extends IntentService {

    private static final String ACTION_INIT = "initApplication";

    public InitializeService() {
        super("InitializeService");
    }

    public static void start(Context context) {
        Intent intent = new Intent(context, InitializeService.class);
        intent.setAction(ACTION_INIT);
        context.startService(intent);
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        if (intent != null) {
            final String action = intent.getAction();
            if (ACTION_INIT.equals(action)) {
                initApplication();
            }
        }
    }

    private void initApplication() {
        //初始化日志
        Logger.init(getPackageName()).hideThreadInfo();

        //初始化错误收集
//        CrashHandler.init(new CrashHandler(getApplicationContext()));
        initBugly();

        //初始化内存泄漏检测
        LeakCanary.install(App.getInstance());

        //初始化过度绘制检测
        BlockCanary.install(getApplicationContext(), new AppBlockCanaryContext()).start();

        //初始化tbs x5 webview
        QbSdk.allowThirdPartyAppDownload(true);
        QbSdk.initX5Environment(getApplicationContext(), QbSdk.WebviewInitType.FIRSTUSE_AND_PRELOAD, new QbSdk.PreInitCallback() {
            @Override
            public void onCoreInitFinished() {
            }

            @Override
            public void onViewInitFinished(boolean b) {
            }
        });
    }

    private void initBugly() {
        Context context = getApplicationContext();
        String packageName = context.getPackageName();
        String processName = SystemUtil.getProcessName(android.os.Process.myPid());
        CrashReport.UserStrategy strategy = new CrashReport.UserStrategy(context);
        strategy.setUploadProcess(processName == null || processName.equals(packageName));
        CrashReport.initCrashReport(context, Constants.BUGLY_ID, isDebug, strategy);
    }
}
```
```
    @Override
    public void onCreate() {
        super.onCreate();
        //初始化数据库
        Realm.init(getApplicationContext());
        //在子线程中完成其他初始化
        InitializeService.start(this);
    }
```
<font color=red>当然, 实际使用过程中要考虑有些库在子线程中初始化会出问题.</font>  

2.