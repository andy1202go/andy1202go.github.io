> 尽管Object是一个具体的类，但设计它主要是为了拓展。它所有的非final方法（equals、hashCode、toString、clone和finalize）都有明确的通用约定（general contract），因为它们被设计成是要被覆盖的。任何一个类，它在覆盖这些方法的时候，都有责任遵守这些通用约定。
>
> 本章论述何时以及如何重写 `Object` 类的非 final 的方法。这一章省略了 finalize 方法，因为它在条目 8 中进行了讨论。`Comparable.compareTo` 方法虽然不是 `Object` 中的方法，因为具有很多的相似性，所以也在这里讨论

## 10 重写equals方法时遵守通用约定

Object的equals方法

```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

重写equals方法，看起来简单，但很容易出错，且出错后代价巨大，即出现问题后不好排查到是equals方法导致，而且可能很多方法都依赖equals方法。

### 不要重写equals方法

这一个条目最重要的一条就是：**避免此问题的最简单方法就是，不覆盖equals方法**
以下几种情况适用于不覆盖equals方法：

- 类的实例都是固有且唯一的。所以Object的equals方法就够用了；

- 类不需要提供另一个“逻辑相等”的测试功能，所以Object的equals方法比较对象是否是同一个就够了

  > 例如 `java.util.regex.Pattern` 可以重写 equals 方法检查两个是否代表完全相同的正则表达式 Pattern 实例，但是设计者并不认为客户需要或希望使用此功能。在这种情况下，从 Object 继承的 equals 实现是最合适的。

- 父类已经重写了equals方法，则父类的行为完全适合该子类。比如

  > 大多数 Set 从 AbstractSet 继承了 equals 实现、List 从 AbstractList 继承了 equals 实现，Map 从 AbstractMap 的 Map 继承了 equals 实现。

- 类是私有的或包级私有的，确保equals方法永远不被调用的。

  > - 如果你非常厌恶风险，可以重写 equals 方法，以确保不会被意外调用：
  >
  > ```java
  > @Override 
  > public boolean equals(Object o) {
  >     throw new AssertionError(); // Method is never called
  > }
  > ```

### 什么时候需要重写equals方法呢？

一句话来说，**值类**。

也就是表示值的类，比如String，比如Integer

```java
//Integer's    
public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```

特征在于

- 类包含逻辑相等的概念
- 且父类没有重写过equals方法

> 程序员使用 equals 方法比较值对象的引用，期望发现它们在逻辑上是否相等，而不是引用相同的对象。重写 equals 方法不仅可以满足程序员的期望，它还支持重写过 equals 的实例作为 Map 的键（key），或者 Set 里的元素，以满足预期和期望的行为。

其中有一种特殊的值类不需要重写equals方法，就是枚举Enum。对于这些类，逻辑相等与对象标识是一样的，所以 Object 的 equals 方法作用逻辑 equals 方法。

### 重写equals方法需要遵守的通用约定

Object 的规范如下：equals 方法实现了一个等价关系（equivalence relation）。它有以下这些属性:

- 自反性：对于任何非空引用x，`x.equals(x)`必须返回true
- 对称性：对于任何非空引用x和y，当且仅当`x.equals(y)`为true时，`y.equals(x)`返回true
- 传递性：对于任何非空引用x、y、z，如果`x.equals(y)`为true，`y.equals(z)`为true，那么`x.equals(z)`为true
- 一致性：对于任何非空引用 x 和 y，如果在 equals 比较中使用的信息没有修改，则 `x.equals(y)` 的多次调用必须始终返回 true 或始终返回 false。
- 非空性：对于任何非空引用 x，`x.equals(null)` 必须返回 false。

> 除非你喜欢数学，否则这看起来有点吓人，但不要忽略它！如果一旦违反了它，很可能会发现你的程序运行异常或崩溃，并且很难确定失败的根源。套用约翰·多恩（John Donne）的说法，没有哪个类是孤立存在的。一个类的实例常常被传递给另一个类的实例。许多类，包括所有的集合类，都依赖于传递给它们遵守 equals 约定的对象。

### 自反性

必须与自身相等，几乎不会违背的性质。

### 对称性

任何两个对象要在是否相等这个问题上达成一致，不能说从a出发，a与b相等，从b出发就不相等了。

比如说

```java
public final class CaseInsensitiveString {
    private final String s;
    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    // Broken - violates symmetry!
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String)  // One-way interoperability!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    ...// Remainder omitted
}
```

本义是好的，区分清楚不区分大小写String和String类型的比较，但如果下面这样子去比较呢

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
System.out.println(cis.equals(s)); // true
System.out.println(s.equals(cis)); // false
```

　　要消除这个问题，只需删除 equals 方法中与 String 类相互操作的恶意尝试。这样做之后，可以将该方法重构为单个返回语句:

```java
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString &&
            ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

### 传递性

equals 约定的第三个要求是，如果第一个对象等于第二个对象，第二个对象等于第三个对象，那么第一个对象必须等于第三个对象。

文中举了一个典型case，是继承父类导致的。

```java
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
    ...  // Remainder omitted
}

public class ColorPoint extends Point {
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    ...  // Remainder omitted
}
```

> 　　equals 方法应该是什么样子？如果完全忽略，则实现是从 Point 类上继承的，颜色信息在 equals 方法比较中被忽略。虽然这并不违反 equals 约定，但这显然是不可接受的。假设你写了一个 equals 方法，它只在它的参数是另一个具有相同位置和颜色的 ColorPoint 实例时返回 true：

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
    // If o is a normal Point, do a color-blind comparison
    if (!(o instanceof ColorPoint))
        return o.equals(this);
    // o is a ColorPoint; do a full comparison
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

　这种方法确实提供了对称性，但是丧失了传递性：

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

　　此外，这种方法可能导致无限递归：假设有两个 Point 的子类，比如 ColorPoint 和 SmellPoint，每个都有这种 equals 方法。 然后调用 `myColorPoint.equals(mySmellPoint)` 将抛出一个 StackOverflowError 异常。

> 里氏替代原则（Liskov substitution principle）指出，任何类型的重要属性都应该适用于所有的子类型，因此任何为这种类型编写的方法都应该在其子类上同样适用[Liskov87]。 这是我们之前声明的一个正式陈述，即 Point 的子类（如 CounterPoint）仍然是一个 Point，必须作为一个 Point 类来看待。 但是，假设我们将一个 CounterPoint 对象传递给 onUnitCircle 方法。 如果 Point 类使用基于 getClass 的 equals 方法，则无论 CounterPoint 实例的 x 和 y 坐标如何，onUnitCircle 方法都将返回 false。 这是因为大多数集合（包括 onUnitCircle 方法使用的 HashSet）都使用 equals 方法来测试是否包含元素，并且 CounterPoint 实例并不等于任何 Point 实例。 但是，如果在 Point 上使用了适当的基于 `instanceof` 的 equals 方法，则在使用 CounterPoint 实例呈现时，同样的 onUnitCircle 方法可以正常工作。

总结下就是，没有令人满意的方法来继承一个可实例化的类并添加一个值组件。

变通方法是，使用组合替代继承。

```java
// Adds a value component without violating the equals contract
public class ColorPoint {
    private final Point point;
    private final Color color;
    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
    /**
     * Returns the point-view of this color point.
     */
    public Point asPoint() {
        return point;
    }
    @Override 
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    ...    // Remainder omitted
}
```

java中有一些特殊的类，违反了上面的结论。

>  例如，`java.sql.Timestamp` 继承了 `java.util.Date` 并添加了一个 nanoseconds 字段。 Timestamp 的等价 equals 确实违反了对称性，并且如果 Timestamp 和 Date 对象在同一个集合中使用，或者以其他方式混合使用，则可能导致不稳定的行为。 Timestamp 类有一个免责声明，告诫程序员不要混用 Timestamp 和 Date。 虽然只要将它们分开使用就不会遇到麻烦，但没有什么可以阻止你将它们混合在一起，并且由此产生的错误可能很难调试。 Timestamp 类的这种行为是一个错误，不应该被仿效。

```java
/**
 * <P>A thin wrapper around <code>java.util.Date</code> that allows
 * the JDBC API to identify this as an SQL <code>TIMESTAMP</code> value.
 * It adds the ability
 * to hold the SQL <code>TIMESTAMP</code> fractional seconds value, by allowing
 * the specification of fractional seconds to a precision of nanoseconds.
 * A Timestamp also provides formatting and
 * parsing operations to support the JDBC escape syntax for timestamp values.
 *
 * <p>The precision of a Timestamp object is calculated to be either:
 * <ul>
 * <li><code>19 </code>, which is the number of characters in yyyy-mm-dd hh:mm:ss
 * <li> <code> 20 + s </code>, which is the number
 * of characters in the yyyy-mm-dd hh:mm:ss.[fff...] and <code>s</code> represents  the scale of the given Timestamp,
 * its fractional seconds precision.
 *</ul>
 *
 * <P><B>Note:</B> This type is a composite of a <code>java.util.Date</code> and a
 * separate nanoseconds value. Only integral seconds are stored in the
 * <code>java.util.Date</code> component. The fractional seconds - the nanos - are
 * separate.  The <code>Timestamp.equals(Object)</code> method never returns
 * <code>true</code> when passed an object
 * that isn't an instance of <code>java.sql.Timestamp</code>,
 * because the nanos component of a date is unknown.
 * As a result, the <code>Timestamp.equals(Object)</code>
 * method is not symmetric with respect to the
 * <code>java.util.Date.equals(Object)</code>
 * method.  Also, the <code>hashCode</code> method uses the underlying
 * <code>java.util.Date</code>
 * implementation and therefore does not include nanos in its computation.
 * <P>
 * Due to the differences between the <code>Timestamp</code> class
 * and the <code>java.util.Date</code>
 * class mentioned above, it is recommended that code not view
 * <code>Timestamp</code> values generically as an instance of
 * <code>java.util.Date</code>.  The
 * inheritance relationship between <code>Timestamp</code>
 * and <code>java.util.Date</code> really
 * denotes implementation inheritance, and not type inheritance.
 */
public class Timestamp extends java.util.Date {
     public boolean equals(java.lang.Object ts) {
      if (ts instanceof Timestamp) {
        return this.equals((Timestamp)ts);
      } else {
        return false;
      }
    }
    
    public boolean equals(Timestamp ts) {
        if (super.equals(ts)) {
            if  (nanos == ts.nanos) {
                return true;
            } else {
                return false;
            }
        } else {
            return false;
        }
    }
}
```

### 一致性

两个类，如果相等，除非有修改equals方法，否则会一直相等。

bad case是`java.net.URL`，它的equals方法，最深的地方会去做DNS解析；

> 不管一个类是不是不可变的，都不要写一个依赖于不可靠资源的 equals 方法。 如果违反这一禁令，满足一致性要求是非常困难的。 例如，`java.net.URL` 类中的 equals 方法依赖于与 URL 关联的主机的 IP 地址的比较。 将主机名转换为 IP 地址可能需要访问网络，并且不能保证随着时间的推移会产生相同的结果。 这可能会导致 URL 类的 equals 方法违反 equals 约定，并在实践中造成问题。 URL 类的 equals 方法的行为是一个很大的错误，不应该被效仿。 不幸的是，由于兼容性的要求，它不能改变。 为了避免这种问题，equals 方法应该只对内存驻留对象执行确定性计算。

```java
        /* If no entry in cache, then do the host lookup */
        if (addresses == null) {
            addresses = getAddressesFromNameService(host, reqAddr);
        }
```

这里已经尽力优化了，比如先比对容易不对的值，然后查ip地址的时候，也是优先本地查表，然后是查缓存，最后是查远端。但始终存在可能不一致的问题。

总结就是equals方法，不要依赖不可靠资源。

### 非空性

非官方要求，对象不能等于null。

专门加一个判断是不必要的。 为了测试它的参数是否相等，equals 方法必须首先将其参数转换为合适类型，以便调用访问器或允许访问的属性。 在执行类型转换之前，该方法必须使用 instanceof 运算符来检查其参数是否是正确的类型：

```java
@Override 
public boolean equals(Object o) {
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
    ...
}
```

### 实践配方

综合起来，以下是编写高质量 equals 方法的配方（recipe）：

1. 使用 == 运算符检查参数是否为该对象的引用。如果是，返回 true。这只是一种性能优化，但是如果这种比较可能很昂贵的话，那就值得去做。
2. 使用 `instanceof` 运算符来检查参数是否具有正确的类型。 如果不是，则返回 false。 通常，正确的类型是 equals 方法所在的那个类。 有时候，改类实现了一些接口。 如果类实现了一个接口，该接口可以改进 equals 约定以允许实现接口的类进行比较，那么使用接口。 集合接口（如 Set，List，Map 和 Map.Entry）具有此特性。
3. 参数转换为正确的类型。因为转换操作在 instanceof 中已经处理过，所以它肯定会成功。
4. 对于类中的每个「重要」的属性，请检查该参数属性是否与该对象对应的属性相匹配。如果所有这些测试成功，返回 true，否则返回 false。如果步骤 2 中的类型是一个接口，那么必须通过接口方法访问参数的属性;如果类型是类，则可以直接访问属性，这取决于属性的访问权限。

```java
package com.example.model;

import lombok.Data;

@Data
public class EqualsTest {
    private String valueStr;

    @Override
    public boolean equals(Object obj) {
        //step 1
        if (this == obj) {
            return true;
        }
        //step 2
        if (!(obj instanceof EqualsTest)) {
            return false;
        }
        //step 3
        EqualsTest e = (EqualsTest) obj;
        //step 4
        if (e.getValueStr().equals(this.getValueStr())) {
            return true;
        }
        return false;
    }
}

    static void testEquals(){
        EqualsTest e1 = new EqualsTest();
        e1.setValueStr("shit");
        EqualsTest e2 = new EqualsTest();
        e2.setValueStr("shit");
        System.out.println(e1.equals(e2));
        System.out.println(e2.equals(e1));
    }
```

其他需要注意的是

- 对于非float、非double的基本类型，使用==运算符进行比较；

- 对于对象引用属性，递归的调用equals方法

- float基本类型，使用静态`Float.compare(a,b)`

- double基本类型，使用静态`Double.compare(a,b)`

  > 由于存在 `Float.NaN`，`-0.0f` 和类似的 double 类型的值，所以需要对 float 和 double 属性进行特殊的处理；有关详细信息，请参阅 JLS 15.21.1 或 Float.equals 方法的详细文档。 虽然你可以使用静态方法 Float.equals 和 Double.equals 方法对 float 和 double 基本类型的属性进行比较，这会导致每次比较时发生自动装箱，引发非常差的性能。 对于数组属性，将这些准则应用于每个元素。 如果数组属性中的每个元素都很重要，请使用其中一个重载的 Arrays.equals 方法。

- 某些对象引用的属性可能合法地包含 null。 为避免出现 `NullPointerException` 异常，请使用静态方法 Objects.equals(Object, Object) 检查这些属性是否相等。

- equals 方法的性能可能受到属性比较顺序的影响。 为了获得最佳性能，你应该首先比较最可能不同的属性

- 当你完成编写完 equals 方法时，问你自己三个问题：它是对称的吗?它是传递吗?它是一致的吗?除此而外，编写单元测试加以排查

### 特别提醒

1. 重写equals的时候，也要重写hashcode，后面说

2. 不要把equals搞的太复杂/太聪明，我觉得URL就是这种

3. 声明要写对：

   ```java
   public boolean equals(Object obj){
       
   }
   ```

### 使用框架

使用谷歌 AutoValue 开源框架，该框架自动为你生成这些方法，只需在类上添加一个注解即可。在大多数情况下，AutoValue 框架生成的方法与你自己编写的方法本质上是相同的。

实践证明VsCode没成功，编译那里有问题，没有使用指定的插件啥的...

## 11 重写equals方法的同时也要重写hashcode方法

**在每个类中，在重写 equals 方法的时侯，一定要重写 hashcode 方法！**

这个是Object的规范约定。 如果不这样做，你的类违反了 hashCode 的通用约定，这会阻止它在 HashMap 和 HashSet 这样的集合中正常工作。

### 具体约定

1. 如果没有修改 equals 方法中用以比较的信息，在应用程序的一次执行过程中对一个对象重复调用 hashCode 方法时，它必须始终返回相同的值。在应用程序的多次执行过程中，每个执行过程在该对象上获取的结果值可以不相同。
2. 如果两个对象根据 equals(Object) 方法比较是相等的，那么在两个对象上调用 hashCode 就必须产生的结果是相同的整数。
3. 如果两个对象根据 equals(Object) 方法比较并不相等，则不要求在每个对象上调用 hashCode 都必须产生不同的结果。 但是，程序员应该意识到，为不相等的对象生成不同的结果可能会提高散列表（hash tables）的性能。

如果违反第二个关键条款，会造成，逻辑上相等的两个对象（equals方法），使用在一个hashMap中，取值是不同的；因为只看hashcode是否相同的。

解决方法中，有一种最讨巧的就是返回一个写死的hashcode，也就是这个类的所有实例，都返回同一个数值。合法，但是万一，万一有人使用这个类的对象做hash表的key，就会严重影响性能了。

### 实践配方

> 一个好的 hash 方法趋向于为不相等的实例生成不相等的哈希码。这也正是 hashCode 约定中第三条的表达。理想情况下，hash 方法为集合中不相等的实例均匀地分配 int 范围内的哈希码。
>
> 1. 声明一个 int 类型的变量 result，并将其初始化为对象中第一个重要属性 `c` 的哈希码，如下面步骤 2.a 中所计算的那样。（回顾条目 10，重要的属性是影响比较相等的领域。）
>
> 2. 对于对象中剩余的重要属性 `f`，请执行以下操作：
>    a. 比较属性 `f` 与属性 `c` 的 int 类型的哈希码：
>
>      -- i. 如果这个属性是基本类型的，使用 `Type.hashCode(f)` 方法计算，其中 `Type` 类是对应属性 `f` 基本类型的包装类。
>      -- ii. 如果该属性是一个对象引用，并且该类的 equals 方法通过递归调用 equals 来比较该属性，并递归地调用 hashCode 方法。 如果需要更复杂的比较，则计算此字段的“范式（“canonical representation）”，并在范式上调用 hashCode。 如果该字段的值为空，则使用 0（也可以使用其他常数，但通常来使用 0 表示）。
>       -- iii. 如果属性 `f` 是一个数组，把它看作每个重要的元素都是一个独立的属性。 也就是说，通过递归地应用这些规则计算每个重要元素的哈希码，并且将每个步骤 2.b 的值合并。 如果数组没有重要的元素，则使用一个常量，最好不要为 0。如果所有元素都很重要，则使用 `Arrays.hashCode` 方法。
>
>    b. 将步骤 2.a 中属性 c 计算出的哈希码合并为如下结果：`result = 31 * result + c;`
>
> 3. 返回 result 值。

文字描述有点晦涩难懂，直接看例子

```java
@Override 
public int hashCode() {
    //step 1
    int result = Short.hashCode(areaCode);
    //step 2
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    //step 3
    return result;
}
```

选用31的原因，一方面是它是素数，另一方面是JVM对它有优化

> 31 的一个很好的特性，是在一些体系结构中乘法可以被替换为移位和减法以获得更好的性能：`31 * i ==（i << 5） - i`。 现代 JVM 可以自动进行这种优化。

其他实现也有用其他素数的，比如URI的：

```java
  @Override
  public int hashCode() {
    int hash = 5;
    hash = 41 * hash + Objects.hashCode(this.m_scheme);
    hash = 41 * hash + Objects.hashCode(this.m_userinfo);
    hash = 41 * hash + Objects.hashCode(this.m_host);
    hash = 41 * hash + this.m_port;
    hash = 41 * hash + Objects.hashCode(this.m_path);
    hash = 41 * hash + Objects.hashCode(this.m_queryString);
    hash = 41 * hash + Objects.hashCode(this.m_fragment);
    return hash;
  }
```

这里，计算单一值的hashcode，使用了Objects.hashCode方法，个人更推荐这样写。

其中没有使用这个方法的一行，是因为是特殊值，写死了一个值。

### 特别提醒

- `Objects` 类有一个静态方法，它接受任意数量的对象并为它们返回一个哈希码。其质量与根据这个项目中的上面编写的方法相当。 不幸的是，它们的运行速度更慢，因为它们需要创建数组以传递可变数量的参数，以及如果任何参数是基本类型，则进行装箱和取消装箱。类似Objects.hash(lineNum, prefix, areaCode);

- 如果一个类是不可变的，并且计算哈希码的代价很大，那么可以考虑在对象中缓存哈希码，而不是在每次请求时重新计算哈希码。

  ```java
  //String
      public int hashCode() {
          int h = hash;
          //here is lazy
          if (h == 0 && value.length > 0) {
              char val[] = value;
  
              for (int i = 0; i < value.length; i++) {
                  h = 31 * h + val[i];
              }
              hash = h;
          }
          return h;
      }
  ```

- **不要试图从哈希码计算中排除重要的属性来提高性能。**

  > Java 类库中的许多类（例如 String 和 Integer）都将 hashCode 方法返回的确切值指定为实例值的函数。 这不是一个好主意，而是一个我们不得不忍受的错误：它妨碍了在未来版本中改进哈希函数的能力。 如果未指定细节并在散列函数中发现缺陷，或者发现了更好的哈希函数，则可以在后续版本中对其进行更改。

- 如果真的需要哈希函数而不太可能产生碰撞，请参阅 Guava 框架的的[[com.google.common.hash.Hashing](http://com.google.common.hash.hashing/)](http://com.google.common.hash.hashing/) [Guava] 方法。

### 使用框架

依然是AutoValue。