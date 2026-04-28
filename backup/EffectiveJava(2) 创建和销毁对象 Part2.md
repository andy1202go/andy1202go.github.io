## 4 使用私有构造方法强化不可实例化

有些类，不可实例化：

- 只包含静态方法和静态字段
- 比如java.lang.Math java.util.Arrays
- java.util.Collections还把一些工厂方法放进去了，比如UnmodifiableCollection这种只读类型的类，可以通过Collections.unmodifiableCollection()方法获得实例

这种不可以实例的类，通常是工具类

> 这样的工具类（utility classes）不是设计用来被实例化的，因为实例化对它没有任何意义。然而，在没有显式构造器的情况下，编译器提供了一个公共的、无参的默认构造器。对于用户来说，该构造器与其他构造器没有什么区别。在已发布的 API 中经常看到无意识的被实例的类。

不应该试图去实例化这种工具类，所以在设计的时候就应该限制它不能被实例化，也不能被继承。

一步实现上述两个要求，就是显式的写一个私有的构造器：

```java
public class UtilityClass {
    private UtilityClass() {
    }
}
```

一个类如果不能被继承，要满足下面某一点

- final的类
- 构造器都是private的（所有的构造器都必须显式或隐式地调用父类构造器，而在这群情况下子类则没有可访问的父类构造器来调用。）

更好的做法是加上注释；更更好的做法是在私有构造器中加上抛异常的逻辑，以防止类的内部调用私有构造器使用

```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

## 5 依赖注入优于硬连接资源

许多类依赖于一个或多个底层资源，这里的底层资源，可以理解为其他类的实例；

这种类做单例的话是不合理的，最多做成静态工厂类可能。

> **静态工具类和单例类不适合需要引用底层资源的类。**
>
> 满足这一需求的简单模式是，**在创建新实例时将资源传递到构造器中。** 这是依赖项注入（dependency injection）的一种形式

就是把所需要的参数传入构造器，同样适用于

> 构造器、静态工厂（详见第 1 条）和 builder 模式（详见第 2 条）。

```java
// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

该模式的一个变体是把工厂传入构造器，比如

```java
@Data
public class Tile {
    private int x;
    private int y;
    private String color;
}

public class Mosaic {
    public Mosaic(Tile tile){}
    public Mosaic(){}
}

public class MosaicFactory {
    public Mosaic create(Supplier<? extends Tile> tileFactory) {
        return new Mosaic(tileFactory.get());
    }
}

        Mosaic mosaic = new MosaicFactory().create(()->new Tile());
```

> Java 8 中引入的 `Supplier<T>` 接口非常适合代表工厂。 在输入上采用 `Supplier<T>` 的方法通常应该使用有界的通配符类型（bounded wildcard type）（详见第 31 条）约束工厂的类型参数，以便客户端能够传入一个工厂，来创建指定类型的任意子类型

> 尽管依赖注入极大地提高了灵活性和可测试性，但它可能使大型项目变得混乱，这些项目通常包含数千个依赖项。使用依赖注入框架（如 Dagger [Dagger]、Guice [Guice] 或 Spring [Spring]）可以消除这些混乱。这些框架的使用超出了本书的范围，但是请注意，为手动依赖注入而设计的 API 非常适合这些框架的使用。

## 6 避免创建不必要的对象

就是说，创建对象怎么都是有开销的，所以尽量重用已有的对象。

比如说不可变类，String。所以不要用 new String("shit")这样子创建String实例。

所以避免创建的方法有以下几种：

- 使用不可变类
- 使用静态方法和构造器，减少创建对象数量，比如Boolean.valueOf()
- 缓存创建开销昂贵的对象

第三种有个典型case，比如正则匹配，如果使用String.matches的话，每次会创建一个Pattern实例，不如把Pattern静态final写好，每次使用就行了

```java
    public static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
                + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }

    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    public static boolean isRomanNumeralGood(String s) {
        return ROMAN.matcher(s).matches();
    }
```

实测差距可能在一倍。

> 当一个对象是不可变的时，很明显它可以被安全地重用，但是在其他情况下，它远没有那么明显，甚至是违反直觉的。考虑适配器（adapters）的情况[Gamma95]，也称为视图（views）。一个适配器是一个对象，它委托一个支持对象（backing object），提供一个可替代的接口。由于适配器没有超出其支持对象的状态，因此不需要为给定对象创建多个给定适配器的实例。

这里说的是，除了上述很明显的避免创建不必要对象的方法，有些已实现的也在践行这个理念，只是比较讨巧。核心是认准对象的使用场景，是否使用一个实例就够用了，虽然这个实例可以变，比如Map.keySet

> 例如，Map 接口的 `keySet` 方法返回 Map 对象的 Set 视图，包含 Map 中的所有 key。 天真地说，似乎每次调用 `keySet` 都必须创建一个新的 Set 实例，但是对给定 Map 对象的 `keySet` 的每次调用都返回相同的 Set 实例。 尽管返回的 Set 实例通常是可变的，但是所有返回的对象在功能上都是相同的：当其中一个返回的对象发生变化时，所有其他对象也都变化，因为它们全部由相同的 Map 实例支持。 虽然创建 `keySet` 视图对象的多个实例基本上是无害的，但这是没有必要的，也没有任何好处。

另一个需要注意会创建不必要对象的场景是自动装箱。本义是好的，但用不好，性能差距很大

```java
    public static long autoBoxingElasps() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }

    public static long autoBoxingElaspsGood() {
        long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }
```

性能差距在10倍左右。

> **优先使用基本类型而不是装箱的基本类型，也要注意无意识的自动装箱。**

最后，有两点重要提示：

1. 这个条目不应该被误解为暗示对象创建是昂贵的，应该避免创建对象。 相反，使用构造方法创建和回收小的对象是非常廉价，构造方法只会做很少的显示工作，尤其是在现代 JVM 实现上
2. 不要自己维护对象池，比如数据库的连接池，交给专业的去做就好了

总结编码中需要注意的是：

- 考虑对象的创建，能不能是单例的
- 考虑对象的构建，能不能是静态的
- 考虑对象的使用，能不能复用
- 基本类型多用，自动装箱注意循环创建
- 对象池什么的不要搞