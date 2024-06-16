### 1. 动态注册和静态注册
动态注册的优势是，相较于静态注册，动态注册执行速度更快，原因是，静态注册需要遍历查找，第一次执行的速度会稍慢（查找耗时，方法越多越明显）

### 2. 动态注册
#### 2.1. 获取签名信息
##### 方案一 使用javap命令
```
javap -s -p class文件路径
```
示例
```
D:\develop\tools\androidsdk\ndk\20.1.5948944\toolchains\aarch64-linux-android-4.9\prebuilt\windows-x86_64\bin> D:\develop\tools\as\jre\bin\javap.exe -s -p D:\develop\code\OpenglSample\lopengl\build\intermediates\runtime_library_classes_dir\debug\com\houtrry\lopengl\view\MapView.class
Compiled from "MapView.kt"
public final class com.houtrry.lopengl.view.MapView extends android.opengl.GLSurfaceView implements android.opengl.GLSurfaceView$Renderer {
  public static final com.houtrry.lopengl.view.MapView$Companion Companion;
    descriptor: Lcom/houtrry/lopengl/view/MapView$Companion;
  public com.houtrry.lopengl.view.MapView(android.content.Context, android.util.AttributeSet);
    descriptor: (Landroid/content/Context;Landroid/util/AttributeSet;)V

  public com.houtrry.lopengl.view.MapView(android.content.Context, android.util.AttributeSet, int, kotlin.jvm.internal.DefaultConstructorMarker);
    descriptor: (Landroid/content/Context;Landroid/util/AttributeSet;ILkotlin/jvm/internal/DefaultConstructorMarker;)V

  public void onSurfaceCreated(javax.microedition.khronos.opengles.GL10, javax.microedition.khronos.egl.EGLConfig);
    descriptor: (Ljavax/microedition/khronos/opengles/GL10;Ljavax/microedition/khronos/egl/EGLConfig;)V

  public void onSurfaceChanged(javax.microedition.khronos.opengles.GL10, int, int);
    descriptor: (Ljavax/microedition/khronos/opengles/GL10;II)V

  public void onDrawFrame(javax.microedition.khronos.opengles.GL10);
    descriptor: (Ljavax/microedition/khronos/opengles/GL10;)V

  private final native void ndkCreate();
    descriptor: ()V

  private final native void ndkResize(int, int);
    descriptor: (II)V

  private final native void ndkDraw();
    descriptor: ()V

  private final native int ndkReadAssertManager(android.content.res.AssetManager, java.lang.String);
    descriptor: (Landroid/content/res/AssetManager;Ljava/lang/String;)I

  private final native boolean ndkReadAssertManagers(android.content.res.AssetManager, java.lang.String[]);
    descriptor: (Landroid/content/res/AssetManager;[Ljava/lang/String;)Z

  public com.houtrry.lopengl.view.MapView(android.content.Context);
    descriptor: (Landroid/content/Context;)V

  static {};
    descriptor: ()V
}
```
descriptor就是方法签名
##### 方案二 使用kotlin bytecode插件
在目标kt类页面，Tools -> Kotlin -> show kotlin Bytecode
便会生成kotlin 字节码信息
```
// ================com/houtrry/lopengl/view/MapView.class =================
// class version 52.0 (52)
// access flags 0x31
public final class com/houtrry/lopengl/view/MapView extends android/opengl/GLSurfaceView implements android/opengl/GLSurfaceView$Renderer {
  //...省略...

  // access flags 0x112
  private final native ndkCreate()V

  // access flags 0x112
  private final native ndkResize(II)V

  // access flags 0x112
  private final native ndkDraw()V

  // access flags 0x112
  private final native ndkReadAssertManager(Landroid/content/res/AssetManager;Ljava/lang/String;)I

  // access flags 0x112
  private final native ndkReadAssertManagers(Landroid/content/res/AssetManager;[Ljava/lang/String;)Z

  // access flags 0x112
  private final native test(Ljava/lang/String;)Ljava/lang/String;

  //...省略...
}
```
以test方法为例， 方法名test， 签名信息(Ljava/lang/String;)Ljava/lang/String;

#### 注册方法信息
模板代码如下：
```
static const JNINativeMethod nativeMethod[] = {
        // Java中的函数名    // 函数签名信息    // native的函数指针
        {"ndkCreate", "()V", (void *) (ndkCreate)},
        {"ndkResize", "(II)V", (void *) (ndkResize)},
        {"ndkDraw", "()V", (void *) (ndkDraw)},
        {"ndkReadAssertManager", "(Landroid/content/res/AssetManager;Ljava/lang/String;)I", (void *) (ndkReadAssertManager)},
        {"ndkReadAssertManagers", "(Landroid/content/res/AssetManager;[Ljava/lang/String;)Z", (void *) (ndkReadAssertManagers)},
};
const char *target_class_name = "com/houtrry/lopengl/view/MapView";
// 类库加载时自动调用
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reversed)
{
    JNIEnv *env = NULL;
    // 初始化JNIEnv
    if(vm->GetEnv(reinterpret_cast<void **>(&env), JNI_VERSION_1_6) != JNI_OK){
        return JNI_FALSE;
    }
    // 找到需要动态动态注册的Jni类
    jclass jniClass = env->FindClass(target_class_name);
    if(nullptr == jniClass){
        return JNI_FALSE;
    }
    // 动态注册
    jint registerResult = env->RegisterNatives(jniClass, nativeMethod, sizeof(nativeMethod)/sizeof(JNINativeMethod));
    if (registerResult) {
        LOGD("register method successfully.");
    } else {
        LOGE("register method failure!!!");
    }
    // 返回JNI使用的版本
    return JNI_VERSION_1_6;
}
```
JNI_OnLoad方法写法固定。
只需要替换类名target_class_name、方法信息nativeMethod

target_class_name是native method所在的类

