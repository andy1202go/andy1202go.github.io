## 12 始终重写toString方法

### 为什么

- Object提供的toString方法，不好用，是类名@十六进制的展示，看不懂啥意思

  ```java
  //    * It is recommended that all subclasses override this method.    
  public String toString() {
          return getClass().getName() + "@" + Integer.toHexString(hashCode());
      }
  ```

- 官方都说，强烈建议，所有类都重写这个方法

- 没有约定，想equals和hashcode那样，但良好的toString方法，对后续的使用大有益处。

- **当对象被传递到 println、printf、字符串连接操作符或断言，或者由调试器打印时，toString 方法会自动被调用**

### 怎么写

- 一般的类，参考源码去写，就把关键信息或者逻辑信息组织好就行了

  ```java
  //Timestamp
     /**
       * Formats a timestamp in JDBC timestamp escape format.
       *         <code>yyyy-mm-dd hh:mm:ss.fffffffff</code>,
       * where <code>ffffffffff</code> indicates nanoseconds.
       * <P>
       * @return a <code>String</code> object in
       *           <code>yyyy-mm-dd hh:mm:ss.fffffffff</code> format
       */
      @SuppressWarnings("deprecation")
      public String toString () {
  
          int year = super.getYear() + 1900;
          int month = super.getMonth() + 1;
          int day = super.getDate();
          int hour = super.getHours();
          int minute = super.getMinutes();
          int second = super.getSeconds();
          String yearString;
          String monthString;
          String dayString;
          String hourString;
          String minuteString;
          String secondString;
          String nanosString;
          String zeros = "000000000";
          String yearZeros = "0000";
          StringBuffer timestampBuf;
  
          if (year < 1000) {
              // Add leading zeros
              yearString = "" + year;
              yearString = yearZeros.substring(0, (4-yearString.length())) +
                  yearString;
          } else {
              yearString = "" + year;
          }
          if (month < 10) {
              monthString = "0" + month;
          } else {
              monthString = Integer.toString(month);
          }
          if (day < 10) {
              dayString = "0" + day;
          } else {
              dayString = Integer.toString(day);
          }
          if (hour < 10) {
              hourString = "0" + hour;
          } else {
              hourString = Integer.toString(hour);
          }
          if (minute < 10) {
              minuteString = "0" + minute;
          } else {
              minuteString = Integer.toString(minute);
          }
          if (second < 10) {
              secondString = "0" + second;
          } else {
              secondString = Integer.toString(second);
          }
          if (nanos == 0) {
              nanosString = "0";
          } else {
              nanosString = Integer.toString(nanos);
  
              // Add leading zeros
              nanosString = zeros.substring(0, (9-nanosString.length())) +
                  nanosString;
  
              // Truncate trailing zeros
              char[] nanosChar = new char[nanosString.length()];
              nanosString.getChars(0, nanosString.length(), nanosChar, 0);
              int truncIndex = 8;
              while (nanosChar[truncIndex] == '0') {
                  truncIndex--;
              }
  
              nanosString = new String(nanosChar, 0, truncIndex + 1);
          }
  
          // do a string buffer here instead.
          timestampBuf = new StringBuffer(20+nanosString.length());
          timestampBuf.append(yearString);
          timestampBuf.append("-");
          timestampBuf.append(monthString);
          timestampBuf.append("-");
          timestampBuf.append(dayString);
          timestampBuf.append(" ");
          timestampBuf.append(hourString);
          timestampBuf.append(":");
          timestampBuf.append(minuteString);
          timestampBuf.append(":");
          timestampBuf.append(secondString);
          timestampBuf.append(".");
          timestampBuf.append(nanosString);
  
          return (timestampBuf.toString());
      }
  ```

- 如果是抽象基类，可以把ToString方法写好，子类往往不需要专门覆盖了

  ```java
  //AbstractCollection
      /**
       * Returns a string representation of this collection.  The string
       * representation consists of a list of the collection's elements in the
       * order they are returned by its iterator, enclosed in square brackets
       * (<tt>"[]"</tt>).  Adjacent elements are separated by the characters
       * <tt>", "</tt> (comma and space).  Elements are converted to strings as
       * by {@link String#valueOf(Object)}.
       *
       * @return a string representation of this collection
       */
      public String toString() {
          Iterator<E> it = iterator();
          if (! it.hasNext())
              return "[]";
  
          StringBuilder sb = new StringBuilder();
          sb.append('[');
          for (;;) {
              E e = it.next();
              sb.append(e == this ? "(this Collection)" : e);
              if (! it.hasNext())
                  return sb.append(']').toString();
              sb.append(',').append(' ');
          }
      }
  ```

- 可以指定格式化的输出，虽然有说可能根据toString的格式来解析，但我个人觉得，还是不要这么做比较好...toString本来也不应该担此重任

### 特别提醒

> 在静态工具类（详见第 4 条）中编写 toString 方法是没有意义的。 你也不应该在大多数枚举类型（条目 34）中写一个 toString 方法，因为 Java 为你提供了一个非常好的方法。 但是，你应该在任何抽象类中定义 toString 方法，该类的子类共享一个公共字符串表示形式。 例如，大多数集合实现上的 toString 方法都是从抽象集合类继承的。

### 使用框架

依然AutoValue，但是对于非常特殊的类，可以自己写，来输出指定格式的，带有逻辑意味的输出

## 13 谨慎的重写clone方法

### 为什么

因为Object中的clone()方法，是受保护的，你不能，不借助反射 （详见第 65 条），仅仅因为它实现了 Cloneable 接口，就调用对象上的 clone 方法。

```java
    protected native Object clone() throws CloneNotSupportedException;
```

Cloneable 接口的目的，仅仅是做个标记

> 既然 Cloneable 接口不包含任何方法，那它用来做什么？ 它决定了 Object 的受保护的 clone 方法实现的行为：如果一个类实现了 Cloneable 接口，那么 Object 的 clone 方法将返回该对象的逐个属性（field-by-field）拷贝；否则会抛出 `CloneNotSupportedException` 异常。这是一个非常反常的接口使用，而不应该被效仿。 通常情况下，实现一个接口用来表示可以为客户做什么。但对于 Cloneable 接口，它会修改父类上受保护方法的行为。

而且，clone方法的通用规范薄弱，没有强约束，比如equals那种

> 创建并返回此对象的副本。 「复制（copy）」的确切含义可能取决于对象的类。 一般意图是，对于任何对象 x，表达式 `x.clone() != x` 返回 true，并且 `x.clone().getClass() == x.getClass()` 也返回 true，但它们不是绝对的要求，但通常情况下，`x.clone().equals(x)` 返回 true，当然这个要求也不是绝对的。
>
> 　　根据约定，这个方法返回的对象应该通过调用 `super.clone` 方法获得的。 如果一个类和它的所有父类（Object 除外）都遵守这个约定，情况就是如此，`x.clone().getClass() == x.getClass()`。
>
> 　　根据约定，返回的对象应该独立于被克隆的对象。 为了实现这种独立性，在返回对象之前，可能需要修改由 super.clone 返回的对象的一个或多个属性。

### 实践配方

如果一定要实现clone方法，真诚的建议是：参考已有的类的实现！！！比如DateFormat中的

```java
    /**
     * Overrides Cloneable
     */
    public Object clone()
    {
        DateFormat other = (DateFormat) super.clone();
        other.calendar = (Calendar) calendar.clone();
        other.numberFormat = (NumberFormat) numberFormat.clone();
        return other;
    }
```

如果一定有步骤的话，大致如下

1. 调用super.clone()，并转类型为当前类，这个是Object类承诺的；
2. 成员类依次重复上述步骤
3. 特殊类型，比如hash类型的，需要单独实现深拷贝（new 对象，赋值等）
4. 捕获CloneNotSupportedException

比如HashSet中

```java
    /**
     * Returns a shallow copy of this <tt>HashSet</tt> instance: the elements
     * themselves are not cloned.
     *
     * @return a shallow copy of this set
     */
    @SuppressWarnings("unchecked")
    public Object clone() {
        try {
            HashSet<E> newSet = (HashSet<E>) super.clone();
            newSet.map = (HashMap<E, Object>) map.clone();
            return newSet;
        } catch (CloneNotSupportedException e) {
            throw new InternalError(e);
        }
    }
```

### 特别提醒

谨慎的考虑提供clone方法，

1. 复制功能最好由构造方法或工厂提供

2. 数组的复制，最好使用clone 

   > 数组是 clone 机制的唯一有力的用途。

## 14 考虑实现Comparable接口

### 为什么

虽然不是Object的实现方法，但是强烈建议，所有有自然顺序的值类，都应该实现这个接口！

实现这个接口，就能享受很多好处，可以在集合类中sort，可以互相比较等。

### 约定

和equals类似

> 将此对象与指定的对象按照排序进行比较。 返回值可能为负整数，零或正整数，因为此对象对应小于，等于或大于指定的对象。 如果指定对象的类型与此对象不能进行比较，则引发 `ClassCastException` 异常。
>
> 　　下面的描述中，符号 sgn(expression) 表示数学中的 signum 函数，它根据表达式的值为负数、零、正数，对应返回-1、0 和 1。
>
> - 实现类必须确保所有 `x` 和 `y` 都满足 `sgn(x.compareTo(y)) == -sgn(y. compareTo(x))`。 （这意味着当且仅当 `y.compareTo(x)` 抛出异常时，`x.compareTo(y)` 必须抛出异常。）
> - 实现类还必须确保该关系是可传递的：`(x. compareTo(y) > 0 && y.compareTo(z) > 0)` 意味着 `x.compareTo(z) > 0`。
> - 最后，对于所有的 z，实现类必须确保 `x.compareTo(y) == 0` 意味着 `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`。
> - 强烈推荐 `(x.compareTo(y) == 0) == (x.equals(y))`，但不是必需的。 一般来说，任何实现了 `Comparable` 接口的类违反了这个条件都应该清楚地说明这个事实。 推荐的语言是「注意：这个类有一个自然顺序，与 `equals` 不一致」。

前三条就是自反性，对称性和传递性，必须要满足才行。

第四条是个强烈建议，翻译一下就是最好和equals的结果保持相同，如果不同，注意标记出来。

### 实践配方

编写方法也和equals类似，关键区别是对入参不用在意，允许抛出异常这样子。

如果有多个属性，应该注意比较的顺序。

```java
// Multiple-field `Comparable` with primitive fields
public int compareTo(PhoneNumber pn) {
    int result = Short.compare(areaCode, pn.areaCode);
    if (result == 0) {
        result = Short.compare(prefix, pn.prefix);
        if (result == 0)
            result = Short.compare(lineNum, pn.lineNum);
    }
    return result;
}
```

注意不要使用符号进行比较

> 　　　　在本书第二版中，曾经推荐如果比较整型基本类型的属性，使用关系运算符「<」和 「>」，对于浮点类型基本类型的属性，使用 `Double.compare` 和 `Float.compare` 静态方法。在 Java 7 中，静态比较方法被添加到 Java 的所有包装类中。 在 `compareTo` 方法中使用关系运算符「<」和「>」是冗长且容易出错的，不再推荐。

### 使用Comparator接口

类中使用，应先静态初始化Comparator，然后在方法中使用

```java
    private static final Comparator<EqualsTest> COMPARATOR = Comparator
    .comparingInt(EqualsTest::getValueInt);

    @Override
    public int compareTo(Object o) {
       try {
            return COMPARATOR.compare(this, (EqualsTest) o);
        } catch (Exception e) {
            // TODO: handle exception
            throw new UnsupportedOperationException("Unimplemented method 'compareTo'");
        }
    }
```

代码中直接使用的话，可以是静态构造，或者上面示例

```java
// Comparator based on static compare method
static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return Integer.compare(o1.hashCode(), o2.hashCode());
    }
};
// Comparator based on Comparator construction method
static Comparator<Object> hashCodeOrder =
        Comparator.comparingInt(o -> o.hashCode());

// BROKEN difference-based comparator - violates transitivity!

static Comparator<Object> hashCodeOrder = new Comparator<>() {
    public int compare(Object o1, Object o2) {
        return o1.hashCode() - o2.hashCode();
    }
};
```

尽量不要使用上面第三个示例，会破坏传递性

> 不要使用这种技术！它可能会导致整数最大长度溢出和 IEEE 754 浮点运算失真的危险[JLS 15.20.1,15.21.1]。 此外，由此产生的方法不可能比使用上述技术编写的方法快得多

## 总结

本章节讨论了Object提供的方法，该如何在新类中进行拓展的原则以及做法和注意事项。

其他都可以忘记，但最好形成一些肌肉记忆

- 谨慎考虑重写equals方法，能不重写就不要重写
- 重写equals的时候，必须重写hashcode
- toString始终重写
- 只有数组推荐使用clone进行复制，其他的类不重写clone方法
- 值类，特别考虑是否重写equals，尽量重写Comparator方法

## 参考资料

1. [[Effective Java (高效 Java) 第三版](https://www.bookstack.cn/books/effective-java-3rd-chinese)](https://www.bookstack.cn/books/effective-java-3rd-chinese)
2. [[Effective Javaa 第三版](https://ebook.qicoder.com/effective-java-3rd/#effective-javaa-%E7%AC%AC%E4%B8%89%E7%89%88)](https://ebook.qicoder.com/effective-java-3rd/#effective-javaa-%E7%AC%AC%E4%B8%89%E7%89%88)
3. [[Java反序列化漏洞 - 1.从URL类的一个bug 说起](https://juejin.cn/post/7115420084209713182)](https://juejin.cn/post/7115420084209713182)
4. [[AutoValue](https://github.com/google/auto/tree/main/value)](https://github.com/google/auto/tree/main/value)
5. https://blog.csdn.net/wangxiaotongfan/article/details/81486952
6. [[Guava hash](https://github.com/google/guava/wiki/HashingExplained#hashing)](https://github.com/google/guava/wiki/HashingExplained#hashing)
7. [[为什么Java String哈希乘数为31？](https://juejin.cn/post/6844903683361079309)](