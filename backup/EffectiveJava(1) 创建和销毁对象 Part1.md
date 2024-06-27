## 1 考虑使用静态工厂方法替代构造方法

静态工厂方法，指的是类中的静态方法，用于创建类的工厂方法；

和设计模式中的工厂方法主要区别在于

- 静态工厂方法创建的是同一个实例，比如Boolean.of()，返回的是TRUE和FALSE两个实例中的一个，没有更多的
- 工厂方法，永远是new一个

先看下Boolean的静态工厂方法实例吧

```java
public final class Boolean implements java.io.Serializable,
                                      Comparable<Boolean>
{
    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code true}.
     */
    public static final Boolean TRUE = new Boolean(true);

    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code false}.
     */
    public static final Boolean FALSE = new Boolean(false);
                                   
    public static Boolean valueOf(String s) {
        return parseBoolean(s) ? TRUE : FALSE;
    }                                      
  }
```

工厂方法的简单工厂，抽象工厂就不写了；但是静态工厂方法和静态单例模式还是挺像的，比如

```java
public class Singleton{
    //lazy
    private static Singleton instance = null;
    
    //private防止直接构造
    private Singleton(){}
    
    public static Singleton getInstance(){
        if(instance == null){
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

静态工厂方法的优点

- 可读性更好：有名字，可以直接表示含义，比如Collections.emptyList()
- 与构造方法不同，它们不需要每次调用时都创建一个新对象
- 可以返回该类的任意子类：比如Collections.unmodifiableList();
- 静态工厂的第四个优点是返回对象的类可以根据输入参数的不同而不同
- 静态工厂的第五个优点是，在编写包含该方法的类时，返回的对象的类不需要存在

缺点

- 名字用于表示含义了，没有标识是实例化的方法：所以用getInstance，newInstance什么的命名，保留至少一个
- 如果类不含public或protect的构造方法，将不能被继承：这也是单例模式的问题，分需要使用的情况来决定吧

综上，给出两种较好的实现：一种是单例模式，线程安全的静态方法；一种如下

```java
public class RandomIntGenerator {
    /**
     * 最小值
     */
    private int min = Integer.MIN_VALUE;
    /**
     * 最大值
     */
    private int max = Integer.MAX_VALUE;

    /**
     * 大于min 小于max
     * @param min
     * @param max
     */
    public RandomIntGenerator(int min, int max)
    {
        this.min = min;
        this.max = max;
    }
    
    /**
     * 大于min 小于max
     * @param min
     * @param max
     */
    public static RandomIntGenerator between(int min, int max)
    {
        return new RandomIntGenerator(min, max);
    }
    /**
     * 大于min 小于Integer.MAX_VALUE
     */
    public static RandomIntGenerator biggerThan(int min)
    {
        return new RandomIntGenerator(min, Integer.MAX_VALUE);
    }

    /**
     * 大于Integer.MIN_VALUE 小于max
     */
    public static RandomIntGenerator smallerThan(int max)
    {
        return new RandomIntGenerator(Integer.MIN_VALUE, max);
    }

    public static RandomIntGenerator getInstance(){
        return new RandomIntGenerator(Integer.MIN_VALUE, Integer.MAX_VALUE);
    }
}
```

### 使用体验

- 感觉更多的是工具类可以这么去定义，毕竟都是单例
- spring本身都是单例了，其实也不需要单独这个了

## 2 当构造方法参数过多时使用builder模式

构造方法比较多的时候，比较几种构造设计

- 伸缩构造方法：就是构造方法中强行塞参数
- JavaBeans：就是属性都是private，通过setter来构造
- Builder模式

使用Builder模式的优势

- 可读性好
- 更安全（相比于JavaBeans，因为JavaBeans的构造方法被割裂为了多次调用，可能导致实例不一致）
- 可以有多个可变参数，其实就是递归使用builder的某个方法，微小优势，不常用

缺点：

- 代码冗长，因为要单独写builder

综合考虑

- 参数大于等于4个的时候，最好使用builder模式
- 注意：需要估计类的参数数量，在一开始的时候，而不是从一种构造方法切换到builder模式，代价较大

最佳实践1

- 使用Lombok的@Builder

  ```java
  import lombok.Builder;
  
  @Builder
  public class SingletonBuilder {
      private String param1;
      private String param2;
      private String param3;
      private String param4;
  }
  ```

- 手写Builder

  ```java
  // Builder Pattern
  public class NutritionFacts {
      private final int servingSize;
      private final int servings;
      private final int calories;
      private final int fat;
      private final int sodium;
      private final int carbohydrate;
      public static class Builder {
          // Required parameters
          private final int servingSize;
          private final int servings;
          // Optional parameters - initialized to default values
          private int calories      = 0;
          private int fat           = 0;
          private int sodium        = 0;
          private int carbohydrate  = 0;
          public Builder(int servingSize, int servings) {
              this.servingSize = servingSize;
              this.servings    = servings;
          }
          public Builder calories(int val) { 
              calories = val;      
          }
          public Builder fat(int val) { 
             fat = val;           
             return this;
          }
          public Builder sodium(int val) { 
             sodium = val;        
             return this; 
          }
          public Builder carbohydrate(int val) { 
             carbohydrate = val;  
             return this; 
          }
          public NutritionFacts build() {
              return new NutritionFacts(this);
          }
      }
      private NutritionFacts(Builder builder) {
          servingSize  = builder.servingSize;
          servings     = builder.servings;
          calories     = builder.calories;
          fat          = builder.fat;
          sodium       = builder.sodium;
          carbohydrate = builder.carbohydrate;
      }
  }
  ```

- 抽象基类使用Builder实现可变参数构造

  ```java
  public abstract class Pizza {
      public enum Topping {HAM,MUSHROOM,ONION,PEPPER,SAUSAGE}
      final Set<Topping> toppings;
  
      abstract static class Builder<T extends Builder<T>> {
          EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
          public T addTopping(Topping topping){
              toppings.add(Objects.requireNonNull(topping));
              return self();//模拟的self类型
          }
          abstract Pizza build();
  
          // Subclasses must override this method to return "this"
          protected abstract T self();
      }
  
      Pizza(Builder<?> builder) {
          toppings = builder.toppings.clone();
      }
  }
  ```

### 使用体验

## 3 使用私有构造方法或枚举实现单例模式

讨论了3种实现单例模式的优劣，分别是

- 私有构造方法+公开的实例；

  ```java
  public class Singleton{
      private Singleton(){}
      public static final Singleton INSTANCE = new Singleton();
  }
  ```

- 私有构造方法+私有的实例+公开的静态方法获取实例

  ```java
  public class Singleton{
      private Singleton(){}
      private static final Singleton INSTANCE = new Singleton();
      public static Singleton getInstance(){
          return INSTANCE;
      }
  }
  ```

- 枚举

  ```java
  public enum Singleton {
      INSTANCE;
  }
  ```

书中最倾向使用枚举来实现单例，原因如下：

- 最简洁

- 无偿地提供了序列化机制（还不怎么理解..https://www.jianshu.com/p/d3d797c3cd45）

  > 绝对防止多次实例化，即使是在面对复杂的序列化或者反射攻击的时候

其次是静态工厂的模式，优点在于

- 通过API表示这是单例模式
- 灵活
  - 需要改变为每个调用该方法的线程返回一个唯一的实例，直接在静态工厂方法中 new instance()，但此时需要去掉单例对象中final修饰的关键字
  - 如果应用程序需要他，我们可以将它改为一个泛型单例工厂（条目30）

缺点在于，小心序列化问题（第11章有讨论）

> 为了防止单例类变成可序列化的，仅仅将添加 *implements Serializable* 到声明中是不够的。我们必须声明所有的实例字段为 *transient*，并提供一个 *readResolve* 方法。否则每当序列化的实例被反序列化时，都会创建一个新的实例，代码实现如下：
>
> ```java
> public class Singleton implements Serializable {
>  //私有无参构造函数
>  private Singleton() {}
> 
>  //单例对象
>  private static final Singleton instance = new Singleton();
> 
>  //静态工厂方法
>  public static Singleton getInstance() {
>      return instance;
>  }
> 
>  //提供该方法，以便重新指定反序列化得到的对象.
>  public Object readResolve() {
>      return instance;
>  }
> }
> ```

### 使用体验

1. 确实有见过比较大的枚举，里面记录了各种单例服务实例的情况，spring之前的情况；

2. 至于spring的单例模式，需要探究下

   [[spring怎么实现单例模式？](https://www.cnblogs.com/nickup/p/9800120.html)](https://www.cnblogs.com/nickup/p/9800120.html)

   可以看到，spring其实使用的是私有属性+公开的方法实现的，但是对单例做了单独的管理的；

   线程安全直接针对的这个单例的map去做的

   ```java
   public abstract class AbstractBeanFactory implements ConfigurableBeanFactory{  
          /** 
           * 充当了Bean实例的缓存，实现方式和单例注册表相同 
           */  
          private final Map singletonCache=new HashMap();  
          public Object getBean(String name)throws BeansException{  
              return getBean(name,null,null);  
          }  
       ...  
          public Object getBean(String name,Class requiredType,Object[] args)throws BeansException{  
             //对传入的Bean name稍做处理，防止传入的Bean name名有非法字符(或则做转码)  
             String beanName=transformedBeanName(name);  
             Object bean=null;  
             //手工检测单例注册表  
             Object sharedInstance=null;  
             //使用了代码锁定同步块，原理和同步方法相似，但是这种写法效率更高  
             synchronized(this.singletonCache){  
                sharedInstance=this.singletonCache.get(beanName);  
              }  
             if(sharedInstance!=null){  
                ...  
                //返回合适的缓存Bean实例  
                bean=getObjectForSharedInstance(name,sharedInstance);  
             }else{  
               ...  
               //取得Bean的定义  
               RootBeanDefinition mergedBeanDefinition=getMergedBeanDefinition(beanName,false);  
                ...  
               //根据Bean定义判断，此判断依据通常来自于组件配置文件的单例属性开关  
               //<bean id="date" class="java.util.Date" scope="singleton"/>  
               //如果是单例，做如下处理  
               if(mergedBeanDefinition.isSingleton()){  
                  synchronized(this.singletonCache){  
                   //再次检测单例注册表  
                    sharedInstance=this.singletonCache.get(beanName);  
                    if(sharedInstance==null){  
                       ...  
                      try {  
                         //真正创建Bean实例  
                         sharedInstance=createBean(beanName,mergedBeanDefinition,args);  
                         //向单例注册表注册Bean实例  
                          addSingleton(beanName,sharedInstance);  
                      }catch (Exception ex) {  
                         ...  
                      }finally{  
                         ...  
                     }  
                    }  
                  }  
                 bean=getObjectForSharedInstance(name,sharedInstance);  
               }  
              //如果是非单例，即prototpye，每次都要新创建一个Bean实例  
              //<bean id="date" class="java.util.Date" scope="prototype"/>  
              else{  
                 bean=createBean(beanName,mergedBeanDefinition,args);  
              }  
       }  
       ...  
          return bean;  
       }  
   }
   ```


##