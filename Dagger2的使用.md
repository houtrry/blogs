##Dagger2的使用
###几个常用的注解 
####@Module, @Component, @Inject, @Provides
使用方法1><br>
① 生成一个普通的Bean
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
②构建Module<br>
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
③书写Component<br>
```
@Component(modules=MainModule.class)
public interface MainComponent {
    void inject(MainActivity mainActivity);
}
```
当Component在所拥有的Module类中找不到依赖需求方需要类型的提供方法时,Dagger2就会检查该需要类型的有没有用@Inject声明的构造方法,有则用该构造方法创建一个.