如何使用Java提供的基本元素来合理设计类和接口。

## 15 使类和成员的可访问性最小化

遵循软件设计的基本原则，即封装。隐藏内部实现细节，仅通过API进行通信。

Java提供了许多机制来帮助信息隐藏。访问控制机制指定了类，接口和成员的可访问性。

### 让每个类或成员尽可能的不可访问

尽可能的缩小可被访问的范围！！从小到大的范围分别如下

- 类和接口：包级私有或者public
- 成员：private,包级私有，protected，public

当遇到需要调整类、接口、成员的访问控制级别的时候，都应该反思之前是不是设定有问题，要仔细考虑再调整（主要是级别升高的情况）

### 特殊情况

1. 继承

   子类复写父类的方法，要求子类中的访问级别不能低于父类中的访问级别

   > 这对于确保子类的实例在父类的实例可用的地方是可用的（Liskov 替换原则，见条目 15）是必要的。 如果违反此规则，编译器将在尝试编译子类时生成错误消息。 这个规则的一个特例是，如果一个类实现了一个接口，那么接口中的所有类方法都必须在该类中声明为 public。

   反例来想，比如子类是private，而父类是public，那父类在多态时候就无法调用子类的方法了...

   取舍问题，访问控制让步于多态。

2. 公有类的实例域绝不能是共有的，包含公有可变域的类通常并不是线程安全的

3. 用final符修饰的内容，如果知识可变对象的引用，则比较危险，比如数组、List等。可以暴露Immutable的List,或者使用方法，每次返回一个clone实例。

   ```java
   // Potential security hole!
   public static final Thing[] VALUES = { ... };
   
   //you should
   private static final Thing[] PRIVATE_VALUES = { ... };
   public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
   
   //or
   private static final Thing[] PRIVATE_VALUES = { ... };
   public static final Thing[] values() {
       return PRIVATE_VALUES.clone();
   }
   
   ```

### 关于内部类

对成员变量，通常是private修饰，然后提供public方法访问获取，封装的较好。但是对于类，往往一个public了事。

文中对于类，有一段说明是，如果类的范围明确，尽量缩小，比如在使用类中声明和使用即可。

> 如果一个类或接口只是在某一个类的内部被用到，就应该考虑使它成为唯一使用它的那个类的私有嵌套类。

内部类除了能够提供更好的封装以外，还具有一些特殊的性质，如内部类可以直接访问到外部类中的成员，包括私有的成员。

在 Java 的集合框架中大量地使用了内部类，比如集合类中迭代器的实现。这些迭代器实现了同样的接口（Iterator Interface），但是具体的实现又各不相同。外部类也不必关心它们的具体实现方法，只需要按照约定的接口进行访问即可。

```java
//ArrayList
    public Iterator<E> iterator() {
        return new Itr();
    }

    private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        Itr() {}

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        @Override
        @SuppressWarnings("unchecked")
        public void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }

        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }
```

内部类分为

- 成员内部类

  ```java
  public class Outer {
  
      private int i = 1;
  
      public class Inner {
          private int i = 2;
  
          public void print() {
              int i = 3;
              System.out.println(i);
              System.out.println(this.i);
              System.out.println(Outer.this.i);
          }
      }
  
      public static void main(String[] args) {
          Outer outer = new Outer();
          Inner inner = outer.new Inner();
          inner.print();
      }
  }
  //3 2 1
  ```

  内部类可以访问外部类的成员变量，反之不行。这种内部类只能先实例外部类，才能实例内部类

- 静态内部类

  static的内部类，只是这个类写在外部类中，不属于任何外部类的实例

  ```java
  public class InnerTest1 {
      private int age;
  
      public static class Inner{
          private int in;
          public void setIn(int in){
              this.in = in;
          }
          public int getIn() {
              return in;
          }    
      }
  
      // public Inner getInner(){
      //     return new Inner();
      // }
  
      public int getAge() {
          return age;
      }
  
      public void setAge(int age) {
          this.age = age;
      }
  
      public static void main(String[] args){
          Inner inner = new Inner();
          inner.setIn(1);
          System.out.println(inner.getIn());
      }
  }
  ```

  

- 局部内部类

- 匿名内部类

  > 匿名内部类 ：是内部类的简化写法。它的本质是一个带具体实现的父类或者父接口的 匿名的 子类对象。开发中，最常用到的内部类就是匿名内部类了。以接口举例，当你使用一个接口时，似乎得做如下几步操作，
  >
  > 
  >
  > 1. 定义子类
  > 2. 重写接口中的方法
  > 3. 创建子类对象
  > 4. 调用重写后的方法
  >
  > 
  >
  > 我们的目的，最终只是为了调用方法，那么能不能简化一下，把以上四步合成一步呢？匿名内部类就是做这样的快捷方式。

  ```java
  public abstract class FlyAble{
      public abstract void fly();
  }
  
  public class InnerDemo2 {
      public static void main(String[] args) {
          /**
          1.等号右边:是匿名内部类，定义并创建该接口的子类对象
          2.等号左边:是多态赋值,接口类型引用指向子类对象
          */
          FlyAble f = new FlyAble(){
              public void fly() {
                  System.out.println("芜湖，起飞！！✈️✈️✈️");
              }
          };
          // 将f传递给showFly方法中
          showFly(f);
      }
      public static void showFly(FlyAble f) {
          f.fly();
      }
  }
  ```

  Java8之后的lambda表达式就是这个东西的进一步简化。

  > 这里分享一个 『Java 核心技术』 中提到的使用匿名内部类的一个场景。在生成日志或调试消息是，通常希望包含当前类的类名，如：
  >
  > ```java
  > System.err.println("Something awful happened in :" + getClass());
  > ```
  >
  > 但是，这对于静态方法不奏效，因为 getClass() 不是一个静态方法。可以通过以下技巧在静态方法中获取到所在的类：
  >
  > ```java
  > new Object(){}.getClass().getEnclosingClass(); //get class of static method
  > ```
  >
  > 在这里，会建立一个 Object 的匿名子类的对象，进而通过 getEnclosingClass 得到其外围类，及包含该静态方法的类。
  >
  > Java 8 之后，很多使用匿名内部类的场景都可以使用 Lambda 表达式进行替换，使用起来也更为方便。

## 16 在公共类中使用访问方法而不是公共属性

解析下标题

- 公共类：public class
- 公共属性：public String value比如说
- 访问方法：getValue()

就比较常用的封装，属性不能被直接访问，而是通过方法。

> **如果一个类在其包之外是可访问的，则提供访问方法来保留更改类内部表示的灵活性。**如果一个公共类暴露其数据属性，那么以后更改其表示形式基本上没有可能，因为客户端代码可以散布在很多地方。

反例是java.awt包中的Point对象

```java
public class Point extends Point2D implements java.io.Serializable {
    /**
     * The X coordinate of this <code>Point</code>.
     * If no X coordinate is set it will default to 0.
     *
     * @serial
     * @see #getLocation()
     * @see #move(int, int)
     * @since 1.0
     */
    public int x;

    /**
     * The Y coordinate of this <code>Point</code>.
     * If no Y coordinate is set it will default to 0.
     *
     * @serial
     * @see #getLocation()
     * @see #move(int, int)
     * @since 1.0
     */
    public int y;
    
    ...//remainder omitted
```

对于不可变的属性，可以暴露出去，危害比较小

```java
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;
    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
           throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
           throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    }

    ... // Remainder omitted
}
```

## 17 最小化可变性

就是在说，设计类的时候，多考虑设计成为不可变类。

不可变类，实际上是说，其实例是不能被修改的类。

Java平台类库有很多不可变类：String，基本类型包装类，BigInteger，BigDecimal

总而言之，不可变类比可变类更易于设计、实现和使用，不容易出错，而且更安全。

### 不可变类需要遵守的五条规则

1. 不要提供修改对象状态的方法

2. 确保这个类不能被继承

3. 所有字段设置为final

4. 所有字段设置为private

5. 确保对任何可变组件的互斥访问

   > 如果你的类有任何引用可变对象的字段，请确保该类的客户端无法获得对这些对象的引用。 切勿将这样的属性初始化为客户端提供的对象引用，或从访问方法返回属性。 在构造方法，访问方法和 `readObject` 方法（详见第 88 条）中进行防御性拷贝（详见第 50 条）。

### 不可变类的方法与函数式编程

举例来说

```java
// Immutable complex number class
public final class Complex {

    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }

        if (!(o instanceof Complex)) {
            return false;
        }

        Complex c = (Complex) o;

        // See page 47 to find out why we use compare instead of ==
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

上面例子中的方法，没有对入参做修改，而是返回了一个新的实例，这种模式是函数式编程

> 因为方法返回将操作数应用于函数的结果，而不修改它们。 与其对应的过程式的（procedural）或命令式的（imperative）的方法相对比，在这种方法中，将一个过程作用在操作数上，导致其状态改变。 请注意，方法名称是介词（如 plus）而不是动词（如 add）。 这强调了方法不会改变对象的值的事实。

### 不可变类的优缺点

优点

- 不可变对象很简单，使用方面

- 不可变对象本质上是线程安全的，它们不需要同步

  > 被多个线程同时访问它们时，不会遭到破坏。 这是实现线程安全的最简单方法。 由于没有线程可以观察到另一个线程对不可变对象的影响，所以**不可变对象可以被自由地共享。** 因此，不可变类应鼓励客户端尽可能重用现有的实例。 一个简单的方法是为常用的值提供公共的静态 final 常量。 例如，`Complex` 类可能提供这些常量：
  >
  > ```java
  > public static final Complex ZERO = new Complex(0, 0);
  > public static final Complex ONE  = new Complex(1, 0);
  > public static final Complex I    = new Complex(0, 1);
  > ```
  >
  > 这种方法可以更进一步。 一个不可变的类可以提供静态的工厂（详见第 1 条）来缓存经常被请求的实例，以避免在现有的实例中创建新的实例。 所有基本类型的包装类和 `BigInteger` 类都是这样做的。 使用这样的静态工厂会使客户端共享实例而不是创建新实例，从而减少内存占用和垃圾回收成本。 在设计新类时，选择静态工厂代替公共构造方法，可以在以后增加缓存的灵活性，而不需要修改客户端。

  自由分享意味着不要提供clone方法，直接用就是。String有clone方法，是历史原因导致的，不要使用。

- **不仅可以共享不可变的对象，而且可以共享内部信息。** 例如，`BigInteger` 类在内部使用符号数值表示法。 符号用 `int` 值表示，数值用 `int` 数组表示。 `negate` 方法生成了一个数值相同但符号相反的新 `BigInteger` 实例。 即使它是可变的，也不需要复制数组；新创建的 `BigInteger` 指向与原始相同的内部数组。

- **不可变对象为其他对象提供了很好的构件（building blocks）**，无论是可变的还是不可变的。 如果知道一个复杂组件的内部对象不会发生改变，那么维护复杂对象的不变性就容易多了。不可变对象是优秀的映射键和集合元素是这一原则的重要例子: 一旦不可变对象作为 `Map` 的键或 `Set` 里的元素，你就不需要担心`Map` 或 `Set` 的不变性被这些对象的值的变化所破坏。

- **不可变对象无偿地提供了的原子失败机制（详见第 76 条）。** 它们的状态永远不会改变，所以不可能出现临时的不一致。

缺点

- **不可变类的主要缺点是对于每个不同的值都需要一个单独的对象。** 

  > 创建这些对象可能代价很高，特别是是大型的对象下。 例如，假设你有一个百万位的 `BigInteger` ，你想改变它的低位。`flipBit` 方法创建一个新的 `BigInteger` 实例，也是一百万位长，与原始位置只有一位不同。 该操作需要与 `BigInteger` 大小成比例的时间和空间。 将其与 `java.util.BitSet` 对比。 像 `BigInteger` 一样，`BitSet` 表示一个任意长度的位序列，但与 `BigInteger` 不同，`BitSet` 是可变的。 `BitSet` 类提供了一种方法，允许你在固定时间内更改百万位实例中单个位的状态.
  >
  > 如果你可以准确预测客户端要在你的不可变类上执行哪些复杂的操作，那么包级私有可变伙伴类的方式可以正常工作。如果不是的话，那么最好的办法就是提供一个公开的可变伙伴类。 这种方法在 Java 平台类库中的主要例子是 `String` 类，它的可变伙伴类是 `StringBuilder`（及其过时的前身 `StringBuffer` 类）。

### 最佳实践

```java
public class FinalBestPractice {
    private final int age;
    private final String name;

    private FinalBestPractice(int age,String name){
        this.age = age;
        this.name = name;
    }

    public static FinalBestPractice getFinalTrue(int age, String name){
        return new FinalBestPractice(age, name);
    }
}
```

这里注意类本身没有标记为final，但是因为没有提供private级别以上的构造方法，所以不能够被继承。

### 特别说明

- BigInteger和BigDecimal实际上是可以继承的，历史原因导致的，不要继承使用

- 对于任何类，每一个属性是否是可变的，都应该做慎重考虑

- 几点重要原则需要实践证明之

  > - **除非有充分的理由使类成为可变类，否则类应该是不可变的。**
  > - **如果一个类不能设计为不可变类，那么也要尽可能地限制它的可变性** 。
  > - **除非有充分的理由不这样做，否则应该把每个属性声明为私有 final 的。**
  > - **构造方法应该创建完全初始化的对象，并建立所有的不变性。** 

- 学习下CountDownLatch的代码

- 学习了解函数式编程与命令式编程，混合使用之

## 参考资料

1. [[Effective Java (高效 Java) 第三版](https://www.bookstack.cn/books/effective-java-3rd-chinese)](https://www.bookstack.cn/books/effective-java-3rd-chinese)
2. https://blog.csdn.net/cbmljs/article/details/103870881
3. https://www.jianshu.com/p/7b595ddd9d99
4. [[关于 Java 内部类的小抄](https://blog.jrwang.me/2016/java-inner-class/)](https://blog.jrwang.me/2016/java-inner-class/)
5. [[Java 中的内部类与匿名内部类详解](https://xie.infoq.cn/article/956c85d1cbec5ae5787084a46)](https://xie.infoq.cn/article/956c85d1cbec5ae5787084a46)
6. [[一篇文章看懂函数式编程与命令式编程](https://blog.51cto.com/u_10941874/5343800)](