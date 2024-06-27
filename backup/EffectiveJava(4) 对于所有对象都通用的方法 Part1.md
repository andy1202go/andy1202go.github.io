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