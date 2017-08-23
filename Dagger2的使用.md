##Dagger2的使用
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