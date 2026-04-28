## 7 消除过期的对象引用

简单来说，就是对象过期了，但引用还在，就会占用内存不释放。久而久之会有各种问题，比如内存泄漏，导致性能下降，甚至磁盘分页，内存溢出。

给出一个自我实现的栈的case：

```java
// Can you spot the "memory leak"?
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    /**
     * Ensure space for at least one more element, roughly
     * doubling the capacity each time the array needs to grow.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

> 垃圾收集语言中的内存泄漏（更适当地称为无意的对象保留 unintentional object retentions）是隐蔽的。 如果无意中保留了对象引用，那么不仅这个对象排除在垃圾回收之外，而且该对象引用的任何对象也是如此。 即使只有少数对象引用被无意地保留下来，也可以阻止垃圾回收机制对许多对象的回收，这对性能产生很大的影响。
>
> 　　这类问题的解决方法很简单：一旦对象引用过期，将它们设置为 null。 在我们的 `Stack` 类的情景下，只要从栈中弹出，元素的引用就设置为过期。 `pop` 方法的修正版本如下所示：

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

特别注意的是：**清空对象引用应该是例外而不是规范**，也就是，一般不这么搞。

但以下几种情况需要格外注意有无内存泄漏，或者说需要手动清空对象：

- 当一个类自己管理内存时，如上面case所示。每当一个元素被释放时，元素中包含的任何对象引用都应该被清除。

- 缓存。

  > 一旦将对象引用放入缓存中，很容易忘记它的存在，并且在它变得无关紧要之后，仍然保留在缓存中。对于这个问题有几种解决方案。如果你正好想实现了一个缓存：只要在缓存之外存在对某个项（entry）的键（key）引用，那么这项就是明确有关联的，就可以用 `WeakHashMap` 来表示缓存；这些项在过期之后自动删除。记住，只有当缓存中某个项的生命周期是由外部引用到键（key）而不是值（value）决定时，`WeakHashMap` 才有用。
  >
  > 　　更常见的情况是，缓存项有用的生命周期不太明确，随着时间的推移一些项变得越来越没有价值。在这种情况下，缓存应该偶尔清理掉已经废弃的项。这可以通过一个后台线程 (也许是 `ScheduledThreadPoolExecutor`) 或将新的项添加到缓存时顺便清理。`LinkedHashMap` 类使用它的 `removeEldestEntry` 方法实现了后一种方案。对于更复杂的缓存，可能直接需要使用 `java.lang.ref`

- 监听器和其他回调。

  > 如果你实现了一个 API，其客户端注册回调，但是没有显式地撤销注册回调，除非采取一些操作，否则它们将会累积。确保回调是垃圾收集的一种方法是只存储弱引用（weak references），例如，仅将它们保存在 `WeakHashMap` 的键（key）中。

说实话，第一种遇到的少，可能看源码会有一些。后两种没有经历过，更多的是使用预算强行解决的。第三点可能压根没有考虑过，都是在使用成熟的配置、中间件这些东西。

后面关注学习吧。

## 8 避免使用Finalizer和Cleaner机制

第一次知道这两个东西，前者是Object中有的，后者应该是可以自定义使用的一种机制。

都是为了实例在不使用的时候关闭回收的。

原则就是：尽量不用！！！

因为不能保证什么时候触发，也就是说期望的“用完就关”，可能无法实现；但是这两个机制可以实现“用完会关”，时间不确定，但保证能关咯，所以就是一种“安全网”（就是蹦极啥的最下面有个安全网的含义）。

Finalizer的例子：FileInputStream

```java
    /**
     * Ensures that the <code>close</code> method of this file input stream is
     * called when there are no more references to it.
     *
     * @exception  IOException  if an I/O error occurs.
     * @see        java.io.FileInputStream#close()
     */
    protected void finalize() throws IOException {
        if ((fd != null) &&  (fd != FileDescriptor.in)) {
            /* if fd is shared, the references in FileDescriptor
             * will ensure that finalizer is only called when
             * safe to do so. All references using the fd have
             * become unreachable. We can call close()
             */
            close();
        }
    }
```

Cleaner的例子：java.util.logging.LogManager

```java
    // This private class is used as a shutdown hook.
    // It does a "reset" to close all open handlers.
    private class Cleaner extends Thread {

        private Cleaner() {
            /* Set context class loader to null in order to avoid
             * keeping a strong reference to an application classloader.
             */
            this.setContextClassLoader(null);
        }

        @Override
        public void run() {
            // This is to ensure the LogManager.<clinit> is completed
            // before synchronized block. Otherwise deadlocks are possible.
            LogManager mgr = manager;

            // If the global handlers haven't been initialized yet, we
            // don't want to initialize them just so we can close them!
            synchronized (LogManager.this) {
                // Note that death is imminent.
                deathImminent = true;
                initializedGlobalHandlers = true;
            }

            // Do a reset to close all active handlers.
            reset();
        }
    }
```

另外还要注意：

> 不要相信 `System.gc` 和 `System.runFinalization` 方法。 他们可能会增加 Finalizer 和 Cleaner 机制被执行的几率，但不能保证一定会执行

正确做法：

- 自己实现的资源类，需要实现AutoCloseable接口
- 使用资源类，try-with-resource

## 9 使用try-with-resource语句替代try-finally语句

对于使用要关闭的资源，一律使用try-with-resource，Java7及以上版本就行。

使用try-finally语句，对多个资源不友好

```java
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```

而且针对finally语句块的异常没有处理，针对其他异常的catch也比较复杂。

上面的case，用try-with-resource重写：

```java
// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

注意写法就是了。

另外注意，能这么写的资源类，都要求实现AutoCloseable接口才行。

## 总结

整个创建和销毁对象模块，主要讨论了几个重点问题

- 创建对象的较好实现（静态工厂方法，Builder模式）
- 创建对象时候要考虑的（单例模式，可读性，不可实例化，对象之间的关系）
- 哪些时候不要创建对象（避免创建不必要的对象，比如自动装箱，比如静态复用，比如Map.keySet）
- 销毁对象时，什么时候会造成不好排查的内存泄漏（对象自我管理内存时需要额外关注）
- 资源关闭型的对象，销毁时的最佳实践（try-with-resource，不用finalizer和cleaner机制）

指导工作实践：

创建对象时，能不能是单例的？能不能用静态工厂方法构建？参数多么？构造方法要不要是私有的？用的时候，对象是用一个新建一个还是就一两个就够了？有没有其他依赖，初始化时候能注入依赖么？

销毁时，有没有资源使用？有没有自我管理内存？

## 参考资料

1. [Effective Java (高效 Java) 第三版](https://www.bookstack.cn/books/effective-java-3rd-chinese)
2. [考虑使用静态工厂方法替代构造方法- 创建对象 - 博客园](https://www.cnblogs.com/chenpi/p/5981084.html)
3. [JavaBean](https://www.liaoxuefeng.com/wiki/1252599548343744/1260474416351680)
4. [03.使用私有构造方法或枚类实现 Singleton 属性](https://www.cnblogs.com/gongguowei01/p/12174260.html)
5. [Effective Javaa 第三版](