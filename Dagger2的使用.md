##Dagger2的使用  
###引入dagger2  
参考[dagger2的GitHub](https://github.com/google/dagger)说明.  
```
dependencies {
  compile 'com.google.dagger:dagger:2.x'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.x'
}
```
---------------------
###几个常用的注解    
####@Module, @Component, @Inject, @Provides   
一. 生成对象的方式  
生成对象有两个方式:  
①.@Inject 注解提供方式：在所需类的构造函数中直接加上 @Inject 注解  
②.@Module 注解提供方式：通过新建一个专门的提供这类来提供类实例，然后在类名上面添加 @Module 注解，在类中自己定义方法，手动通过 new 来进行创建，这种主要是针对第三方库中，我们无法直接在构造函数中添加 @Inject 注解的情况。  
1.方式①  
1> 用@Inject注解Bean类的构造方法
```
public class Apple {

    @Inject
    public Apple() {

    }

    public String getDescription() {
        return "this is a red apple";
    }
}
```
2>使用@Component注解自己的Component类.比如下面的AppleComponent, 该类用@Component注解, 类中提供inject方法(方法名可以随便写, 一般用inject), 方法的参数Main9Activity是需要使用注解生成Apple的类.  
```
@Component
public interface AppleComponent {
    void inject(Main9Activity main9Activity);
}
```
3>使用@Inject注解需要生成Apple的变量  
```

public class Main9Activity extends AppCompatActivity {

    @BindView(R.id.textView9)
    TextView mTextView9;

    @Inject
    Apple mApple;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main9);
        ButterKnife.bind(this);
    }
}

```
4>编译项目, Android Studio  
 Build-->Make Project  
或者直接点击  
![Aaron Swartz](https://raw.githubusercontent.com/houtrry/blogs/master/image/001.png)  

5>注入Main9Activity  
```
public class Main9Activity extends AppCompatActivity {

    @BindView(R.id.textView9)
    TextView mTextView9;

    @Inject
    Apple mApple;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main9);
        ButterKnife.bind(this);

        DaggerAppleComponent.builder().build().inject(this);

        mTextView9.setText(mApple.getDescription());
    }
}
```
```
DaggerAppleComponent.builder().build().inject(this);
```
这里, 上面第4步生成了DaggerAppleComponent, 实现了接口AppleComponent的inject方法, 此时, 注入Main9Activity, 将Main9Activity和Apple关联起来.  

2.方式②  
1> 生成一个普通的Bean
```
public class Cloth {
    private String color;

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    @Override
    public String toString() {
        return color + "布料";
    }
}
```
2>构建Module  
注解@Module表明这是一个Module类, 方法上的@Provides表明该方法生成一个返回值为Cloth类型的依赖对象
```
@Module
public class MainModule {

    @Provides
    public Cloth getCloth() {
        Cloth cloth = new Cloth();
        cloth.setColor("红色");
        return cloth;
    }
}
```
3>书写Component  
```
@Component(modules=MainModule.class)
public interface MainComponent {
    void inject(MainActivity mainActivity);
}
```
MainComponent是一个接口, 接口上用@Component注解, 表明这是一个Component, @Component的参数model=MainModule.class, 将MainMoudle和MainComponent关联起来, MainComponent中有一个inject的方法(方法名随意, 方法参数是需要使用到注解生成对象的类, 这里是需要在MainActivity中使用注解生成Cloth)  
4>AS: Build-->make Project  
5>在MainActivity中使用注解.  
```
public class Main3Activity extends AppCompatActivity {

    private TextView mTextView;

    @Inject
    Cloth mCloth;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main3);

        mTextView = (TextView) findViewById(R.id.textView2);

        DaggerMainComponent.builder().mainModule(new MainModule()).build().inject(this);
    }
}
```
二.Dagger2原理分析
1.就使用上面方式一的例子吧.注入的目的是在Main9Activity中生成Apple的实例.  
我们从
```
DaggerAppleComponent.builder().build().inject(this);
```
开始分析吧.  
首先, 看下DaggerAppleComponent的源码  
```
public final class DaggerAppleComponent implements AppleComponent {
  private MembersInjector<Main9Activity> main9ActivityMembersInjector;

  private DaggerAppleComponent(Builder builder) {
    assert builder != null;
    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  public static AppleComponent create() {
    return new Builder().build();
  }

  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {

    this.main9ActivityMembersInjector =
        Main9Activity_MembersInjector.create(Apple_Factory.create());
  }

  @Override
  public void inject(Main9Activity main9Activity) {
    main9ActivityMembersInjector.injectMembers(main9Activity);
  }

  public static final class Builder {
    private Builder() {}

    public AppleComponent build() {
      return new DaggerAppleComponent(this);
    }
  }
}
```
2.DaggerAppleComponent.builder(): builder()方法里new 了一个DaggerAppleComponent.Builder, 并返回这个新建的Builder.  
3.DaggerAppleComponent.builder().build(): build()方法里new了DaggerAppleComponent的实例, 并将上个方法创建的Builder传给了DaggerAppleComponent的构造方法.  
在DaggerAppleComponent的构造方法里, 调用了DaggerAppleComponent.initialize(builder).   
在initialize里. 调用了Main9Activity_MembersInjector.create(Apple_Factory.create())方法.  
4.现在我们来看一下这个Main9Activity_MembersInjector.create(Apple_Factory.create())方法都做了什么事情.  
首先, 看下Apple_Factory的源码 
```
public final class Apple_Factory implements Factory<Apple> {
  private static final Apple_Factory INSTANCE = new Apple_Factory();

  @Override
  public Apple get() {
    return new Apple();
  }

  public static Factory<Apple> create() {
    return INSTANCE;
  }
}
```
从源码可知, Apple_Factory是个单例, create方法返回了该单例. get方法返回了一个Apple实例.  
这个时候就要注意了, 我们注解的目的是什么? 就是生成一个Apple实例啊, 那么, 现在我们知道, Apple_Factory的get方法创建了一个Apple实例. 记住这一点. 我们继续向下看Main9Activity_MembersInjector.  
```
public final class Main9Activity_MembersInjector implements MembersInjector<Main9Activity> {
  private final Provider<Apple> mAppleProvider;

  public Main9Activity_MembersInjector(Provider<Apple> mAppleProvider) {
    assert mAppleProvider != null;
    this.mAppleProvider = mAppleProvider;
  }

  public static MembersInjector<Main9Activity> create(Provider<Apple> mAppleProvider) {
    return new Main9Activity_MembersInjector(mAppleProvider);
  }

  @Override
  public void injectMembers(Main9Activity instance) {
    if (instance == null) {
      throw new NullPointerException("Cannot inject members into a null reference");
    }
    instance.mApple = mAppleProvider.get();
  }

  public static void injectMApple(Main9Activity instance, Provider<Apple> mAppleProvider) {
    instance.mApple = mAppleProvider.get();
  }
}
```
Main9Activity_MembersInjector.create(Apple_Factory.create())方法到底做了什么呢?  
我们已知Apple_Factory.create()返回了单例Apple_Factory并且, Apple_Factory.get方法返回了Apple的实例.  
现在来看Main9Activity_MembersInjector.create, 在该方法中, new了一个Main9Activity_MembersInjector, 并将Apple_Factory的单例传给这个构造方法.  
在Main9Activity_MembersInjector的构造方法中, 将Apple_Factory的实例赋值给了Main9Activity_MembersInjector的成员变量mAppleProvider, 此时, 通过成员变量mAppleProvider的get方法就可以得到一个Apple的实例了.  
5.DaggerMainComponent.builder().mainModule(new MainModule()).build().inject(this): 最后这个inject方法做了什么呢?  
```
  @Override
  public void inject(Main9Activity main9Activity) {
    main9ActivityMembersInjector.injectMembers(main9Activity);
  }
```
```
  @Override
  public void injectMembers(Main9Activity instance) {
    if (instance == null) {
      throw new NullPointerException("Cannot inject members into a null reference");
    }
    instance.mApple = mAppleProvider.get();
  }
```
此时, 我们终于找到了这个给Main9Activity的成员变量mApple赋值的地方了, 通过inject调用Main9Activity_MembersInjector.injectMembers方法, 在injectMembers方法中, 通过单例Apple_Factory.get获取Apple的实例, 并将该实例直接赋值飞Main9Activity.mApple, 到此, 给mApple生成实例的过程就结束了.  当然, 这里分析的只是最简单的使用模式, 随着Dagger2的使用, 代码会越来越复杂.
在这里, 我们也可以明白, 为什么成员变量mApple不能是private的, 因为给mApple赋值的方式使用过instance.mApple来实现的, 如果mApple是private, 这里将会报错.