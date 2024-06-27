## 5 闭包

高阶函数除了可以接受函数作为参数外，还可以把函数作为结果返回。

比如，同样是求和，一个是调用后返回结果，一个是返回函数结果，再用这个结果计算

```javascript
function sum(arr) {
    return arr.reduce(function (x, y) {
        return x + y;
    });
}

sum([1, 2, 3, 4, 5]); // 15

function lazy_sum(arr) {
    var sum = function () {
        return arr.reduce(function (x, y) {
            return x + y;
        });
    }
    return sum;
}

var f=lazy_sum([1, 2, 3, 4, 5]); 
f();

//调用的时候再传入
function lazy_sum2() {
    var sum = function (arr) {
        return arr.reduce(function (x, y) {
            return x + y;
        });
    }
    return sum;
}

var f=lazy_sum2(); 
console.log(f([1, 2, 3, 4, 5]));
```

请再注意一点，当我们调用`lazy_sum()`时，每次调用都会返回一个新的函数，即使传入相同的参数：

```javascript
var f1 = lazy_sum([1, 2, 3, 4, 5]);
var f2 = lazy_sum([1, 2, 3, 4, 5]);
f1 === f2; // false
```

另一个需要注意的问题是，返回的函数并没有立刻执行，而是直到调用了`f()`才执行。我们来看一个例子：

```javascript
function count() {
    var arr = [];
    for (var i=1; i<=3; i++) {
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}

var results = count();
var f1 = results[0];
var f2 = results[1];
var f3 = results[2];
```

这里results=count();的时候，results里面是array[3]，里面是3个return i*i;

问题是i为多少呢？

i执行到4停止了，因为是var i，所以现在i=4；

你可能认为调用`f1()`，`f2()`和`f3()`结果应该是`1`，`4`，`9`，但实际结果是：

```javascript
f1(); // 16
f2(); // 16
f3(); // 16
```

> 返回闭包时牢记的一点就是：返回函数不要引用任何循环变量，或者后续会发生变化的变量。

修改方法1：

```javascript
function count() {
    var arr = [];
    for (var i=1; i<=3; i++) {
        arr.push((function (n) {
            return function () {
                return n * n;
            }
        })(i));
    }
    return arr;
}
```

这里使用了“创建一个匿名函数并立刻执行”的语法：

```javascript
(function (x) {
    return x * x;
})(3); // 9
```

因为results中的函数改为了

```javascript
(function (n) {
            return function () {
                return n * n;
            }
        })(i);
```

return n*n;并且i=对应的数据

修改方法2：使用let

```javascript
function count() {
    var arr = [];
    for (let i=1; i<=3; i++) {
        arr.push(function () {
            return i * i;
        });
    }
    return arr;
}

var results = count();
var f1 = results[0];
var f2 = results[1];
var f3 = results[2];

console.log(f1());
console.log(f2());
```

改变var为let之后，i的作用域就是块级别的了，相当于每个返回的函数，都只是使用了当前作用域内的参数i的值。

### 闭包的作用

- 返回函数并延迟执行

  如上面示例

- 封装私有变量

  对标Java等的private修饰符。但JavaScript只有方法没有class，所以只有这种方法。

  ```javaScript
  'use strict';
  
  function create_counter(initial) {
      var x = initial || 0;
      return {
          inc: function () {
              x += 1;
              return x;
          }
      }
  }
  
  var c1 = create_counter();
  c1.inc(); // 1
  c1.inc(); // 2
  c1.inc(); // 3
  
  var c2 = create_counter(10);
  c2.inc(); // 11
  c2.inc(); // 12
  c2.inc(); // 13
  ```

  这里的x，就是生成的计数器的私有变量；从外部代码根本无法访问到变量`x`

  句话说，闭包就是携带状态的函数，并且它的状态可以完全对外隐藏起来。

- 把多参数的函数变成单参数的函数

  例如，要计算xy可以用`Math.pow(x, y)`函数，不过考虑到经常计算x2或x3，我们可以利用闭包创建新的函数`pow2`和`pow3`：

  ```javascript
  'use strict';
  
  function make_pow(n) {
      return function (x) {
          return Math.pow(x, n);
      }
  }
  // 创建两个新函数:
  var pow2 = make_pow(2);
  var pow3 = make_pow(3);
  
  console.log(pow2(5)); // 25
  console.log(pow3(7)); // 343
  ```

  就函数疯狂嵌套，毕竟函数可以做入参和出参。

## 6 箭头函数

就是类似Java中的function的lambda表达式写法

```javascript
x => x * x
```

上面的箭头函数相当于：

```javascript
function (x) {
    return x * x;
}
```

x=>x*x，这种属于最省略的写法。

如果参数有多个，就要()；如果返回逻辑复杂，就不能少了{}和return

```javascript
// 可变参数:
(x, y, ...rest) => {
    var i, sum = x + y;
    for (i=0; i<rest.length; i++) {
        sum += rest[i];
    }
    return sum;
}
```

箭头函数看上去是匿名函数的一种简写，但实际上，箭头函数和匿名函数有个明显的区别：箭头函数内部的`this`是词法作用域，由上下文确定。

换言之，this在箭头函数中就是直观理解的对象。

由于`this`在箭头函数中已经按照词法作用域绑定了，所以，用`call()`或者`apply()`调用箭头函数时，无法对`this`进行绑定，即传入的第一个参数被忽略：

```javascript
var obj = {
    birth: 1990,
    getAge: function (year) {
        var b = this.birth; // 1990
        var fn = (y) => y - this.birth; // this.birth仍是1990
        return fn.call({birth:2000}, year);
    }
};
obj.getAge(2015); // 25
```

### 练习

请使用箭头函数简化排序时传入的函数：

```javascript
'use strict'
var arr = [10, 20, 1, 2];
arr.sort((x, y) => {
    return x-y;
});
console.log(arr); // [1, 2, 10, 20]
```

## 7 generator

generator（生成器）是ES6标准引入的新的数据类型。一个generator看上去像一个函数，但可以返回多次。

generator跟函数很像，定义如下：

```javascript
function* foo(x) {
    yield x + 1;
    yield x + 2;
    return x + 3;
}
```

generator和函数不同的是，generator由`function*`定义（注意多出的`*`号），并且，除了`return`语句，还可以用`yield`返回多次。

存在即合理！或者说，生成器是解决了什么问题？

> generator和普通函数相比，有什么用？
>
> 因为generator可以在执行过程中多次返回，所以它看上去**就像一个可以记住执行状态的函数**，利用这一点，写一个generator就可以实现需要用面向对象才能实现的功能。

generator还有另一个巨大的好处，就是把异步回调代码变成“同步”代码。这个好处要等到后面学了AJAX以后才能体会到。

总结来说，2点作用

1. 实现面向对象编程
2. 异步变“同步”，代码可读性大大提高

使用上，要使用function*来定义，用yield a返回数据，用f.next()来请求下一次输出，或者使用for(let i of f())遍历获得输出。

同一个函数，使用生成器前后的写法如下所示（斐波那契数列）

```javascript
function fib(max) {
    var
        t,
        a = 0,
        b = 1,
        arr = [0, 1];
    while (arr.length < max) {
        [a, b] = [b, a + b];
        arr.push(b);
    }
    return arr;
}

function* fib(max) {
    var
        t,
        a = 0,
        b = 1,
        n = 0;
    while (n < max) {
        yield a;
        [a, b] = [b, a + b];
        n ++;
    }
    return;
}
for (var x of fib(10)) {
    console.log(x); // 依次输出0, 1, 1, 2, 3, ...
}
```

### 练习

要生成一个自增的ID，可以编写一个`next_id()`函数：

```javascript
var current_id = 0;

function next_id() {
    current_id ++;
    return current_id;
}
```

由于函数无法保存状态，故需要一个全局变量`current_id`来保存数字。

不用闭包，试用generator改写：

```javascript
'use strict';
function* next_id() {
    let n = 1;
    while (n > 0) {
        yield n;
        n++;
    }
}

// 测试:
var
    x,
    pass = true,
    g = next_id();
for (x = 1; x < 100; x++) {
    if (g.next().value !== x) {
        pass = false;
        console.log('测试失败!');
        break;
    }
}
if (pass) {
    console.log('测试通过!');
}
```



## 参考资料

1. [[廖雪峰的JavaScript教程](https://www.liaoxuefeng.com/wiki/1022910821149312/1023021053637728)](https://www.liaoxuefeng.com/wiki/1022910821149312/1023021053637728)
2. [[MapReduce: Simplified Data Processing on Large Clusters](https://research.google/pubs/pub62/)](https://research.google/pubs/pub62/)