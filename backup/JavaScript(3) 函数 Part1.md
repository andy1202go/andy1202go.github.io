## 1 函数定义和调用

常规定义：

```javascript
function abs(x){
    ...
}
```

- function指出这是一个函数定义
- abs是函数名
- (x)内是函数的参数，多个以逗号分隔
- {}是函数体

需要注意：

- 一旦执行到return就结束函数
- 如果没有return，会返回undefined

由于JavaScript的函数也是一个对象，所以有另一种函数定义方法

```javascript
var abs = function(x){
    ...
};
```

需要注意：

- 这种方式要有分号结尾
- 使用上完全相同

另外，JavaScript对函数入参比较随意，需要格外注意

```javascript
abs(10, 'blablabla'); // 多个参数，只用了第一个；返回10
abs(-9, 'haha', 'hehe', null); // 返回9
abs(); // 返回NaN

//改良
function abs(x) {
    if (typeof x !== 'number') {
        throw 'Not a number';
    }
    if (x >= 0) {
        return x;
    } else {
        return -x;
    }
}
```

### arguments

> JavaScript还有一个免费赠送的关键字`arguments`，它只在函数内部起作用，并且永远指向当前函数的调用者传入的所有参数。`arguments`类似`Array`但它不是一个`Array`
>
> 实际上`arguments`最常用于判断传入参数的个数。你可能会看到这样的写法：

```javascript
// foo(a[, b], c)
// 接收2~3个参数，b是可选参数，如果只传2个参数，b默认为null：
function foo(a, b, c) {
    if (arguments.length === 2) {
        // 实际拿到的参数是a和b，c为undefined
        c = b; // 把b赋给c
        b = null; // b变为默认值
    }
    // ...
}
```

### rest参数

和arguments类似，这个主要是取指定参数以外的参数；

同样是ES6引入的。

和arguments不同的是，rest要在函数定义中体现

```javascript
function foo(a, b, ...rest) {
    console.log('a = ' + a);
    console.log('b = ' + b);
    console.log(rest);
}

foo(1, 2, 3, 4, 5);
// 结果:
// a = 1
// b = 2
// Array [ 3, 4, 5 ]

foo(1);
// 结果:
// a = 1
// b = undefined
// Array []
```

rest参数只能写在最后，前面用`...`标识，从运行结果可知，传入的参数先绑定`a`、`b`，多余的参数以数组形式交给变量`rest`，所以，不再需要`arguments`我们就获取了全部参数。

如果传入的参数连正常定义的参数都没填满，也不要紧，rest参数会接收一个空数组（注意不是`undefined`）。

```javascript
'use strict';
function sum(...rest) {
    var count = 0;
    for (var i of rest) {
        count = count + i;
    }
    console.log(count);
    return count;
}
// 测试:
var i, args = [];
for (i=1; i<=100; i++) {
    args.push(i);
}
if (sum() !== 0) {
    console.log('测试失败: sum() = ' + sum());
} else if (sum(1) !== 1) {
    console.log('测试失败: sum(1) = ' + sum(1));
} else if (sum(2, 3) !== 5) {
    console.log('测试失败: sum(2, 3) = ' + sum(2, 3));
} else if (sum.apply(null, args) !== 5050) {
    console.log('测试失败: sum(1, 2, 3, ..., 100) = ' + sum.apply(null, args));
} else {
    console.log('测试通过!');
}
```

#### 小心你的return语句

总之一句话，return语句尽量不要换行！

JavaScript引擎有一个在行末自动添加分号的机制，这可能让你栽到return语句的一个大坑

所以正确的多行写法是：

```javascript
function foo() {
    return { // 这里不会自动加分号，因为{表示语句尚未结束
        name: 'foo'
    };
}
```

#### 练习

定义一个计算圆面积的函数`area_of_circle()`，它有两个参数：

- r: 表示圆的半径；
- pi: 表示π的值，如果不传，则默认3.14

```javascript
'use strict';

function area_of_circle(r, pi) {
    if (null === pi || undefined === pi) {
        pi = 3.14;
    }
    var ret = pi * r * r;
    return ret;
}
// 测试:
if (area_of_circle(2) === 12.56 && area_of_circle(2, 3.1416) === 12.5664) {
    console.log('测试通过');
} else {
    console.log('测试失败');
}
```

## 2 变量作用域与解构赋值

在JavaScript中，用`var`申明的变量实际上是有作用域的。

#### 变量作用域

- 如果一个变量在函数体内部申明，则该变量的作用域为整个函数体，在函数体外不可引用该变量

  ```javascript
  'use strict';
  
  function foo() {
      var x = 1;
      x = x + 1;
  }
  
  x = x + 2; // ReferenceError! 无法在函数体外引用变量x
  ```

- 如果两个不同的函数各自申明了同一个变量，那么该变量只在各自的函数体内起作用

  ```javascript
  'use strict';
  java
  function foo() {
      var x = 1;
      x = x + 1;
  }
  
  function bar() {
      var x = 'A';
      x = x + 'B';
  }
  ```

- JavaScript中，函数可以嵌套，此时，内部函数可以访问外部函数定义的变量，反之不行

  ```javascript
  'use strict';
  
  function foo() {
      var x = 1;
      function bar() {
          var y = x + 1; // bar可以访问foo的变量x!
      }
      var z = y + 1; // ReferenceError! foo不可以访问bar的变量y!
  }
  ```

- 如果内外函数变量重名，会从“内”向“外”查找，即先从本函数开始找，找不到就往外层函数查找（后续会发现，全局变量也是遵从这个规则的）；抽象来看，应该说优先从本域查找，然后往外域查找

  ```javascript
  'use strict';
  function foo() {
      var x = 1;
      function bar() {
          var x = 'A';
          console.log('x in bar() = ' + x); // 'A'
      }
      console.log('x in foo() = ' + x); // 1
      bar();
  }
  
  foo();
  //x in foo() = 1
  //x in bar() = A
  ```

#### 变量提升

JavaScript的函数定义有个特点，它会先扫描整个函数体的语句，把所有申明的变量“提升”到函数顶部：

```javascript
'use strict';

function foo() {
    var x = 'Hello, ' + y;
    console.log(x);
    var y = 'Bob';
}

foo();
```

虽然是strict模式，但语句`var x = 'Hello, ' + y;`并不报错，原因是变量`y`在稍后申明了。但是`console.log`显示`Hello, undefined`，说明变量`y`的值为`undefined`。这正是因为JavaScript引擎自动提升了变量`y`的声明，但不会提升变量`y`的赋值。

对于上述`foo()`函数，JavaScript引擎看到的代码相当于：

```javascript
function foo() {
    var y; // 提升变量y的申明，此时y为undefined
    var x = 'Hello, ' + y;
    console.log(x);
    y = 'Bob';
}
```

由于JavaScript的这一怪异的“特性”，我们在函数内部定义变量时，请严格遵守“**在函数内部首先申明所有变量**”这一规则。

#### 全局作用域

- 不在任何函数内定义的变量就具有全局作用域。

- 实际上，JavaScript默认有一个全局对象window

- 全局作用域的变量实际上被绑定到window的一个属性

  ```javascript
  'use strict';
  
  var course = 'Learn JavaScript';
  alert(course); // 'Learn JavaScript'
  alert(window.course); // 'Learn JavaScript'
  ```

- 函数也是一样的

  ```javascript
  'use strict';
  window.alert('调用window.alert()');
  // 把alert保存到另一个变量:
  var old_alert = window.alert;
  // 给alert赋一个新函数:
  window.alert = function () {}
  alert('无法用alert()显示了!');
  // 恢复alert:
  window.alert = old_alert;
  alert('又可以用alert()了!');
  ```

- 这说明JavaScript实际上只有一个全局作用域。任何变量（函数也视为变量），如果没有在当前函数作用域中找到，就会继续往上查找，最后如果在全局作用域中也没有找到，则报`ReferenceError`错误。

#### 名字空间

- 因为只有一个全局作用域，所以不同js文件定义的全局变量，如果名称相同，会造成命名冲突，也很难排查。

- 解决方案是把自己所有的变量和函数全都绑定到一个全局变量中（许多著名的JavaScript库都是这么干的：jQuery，YUI，underscore等等。）

  ```javascript
  // 唯一的全局变量MYAPP:
  var MYAPP = {};
  
  // 其他变量:
  MYAPP.name = 'myapp';
  MYAPP.version = 1.0;
  
  // 其他函数:
  MYAPP.foo = function () {
      return 'foo';
  };
  ```

#### 局部作用域

由于JavaScript的变量作用域实际上是函数内部，我们在`for`循环等语句块中是无法定义具有局部作用域的变量的：

```javascript
'use strict';

function foo() {
    for (var i=0; i<100; i++) {
        //
    }
    i += 100; // 仍然可以引用变量i
}
```

ES6引入let字段，替代var声明一个块级作用域的变量

#### 常量

类似块级作用域，常量之前也没有，ES6增加了const关键字来替换var

```javascript
'use strict';

const PI = 3.14;
PI = 3; // 某些浏览器不报错，但是无效果！
PI; // 3.14
```

#### 解构赋值

从ES6开始，JavaScript引入了解构赋值，可以同时对一组变量进行赋值。

什么是解构赋值？我们先看看传统的做法，如何把一个数组的元素分别赋值给几个变量：

```javascript
var array = ['hello', 'JavaScript', 'ES6'];
var x = array[0];
var y = array[1];
var z = array[2];
```

现在，在ES6中，可以使用解构赋值，直接对多个变量同时赋值：

```javascript
'use strict';

// 如果浏览器支持解构赋值就不会报错:
var [x, y, z] = ['hello', 'JavaScript', 'ES6'];
// x, y, z分别被赋值为数组对应元素:
console.log('x = ' + x + ', y = ' + y + ', z = ' + z);
```

- 支持嵌套
- 支持忽略某些元素
- 支持对象的解构
- 支持解构的同时赋值
- 支持解构赋值的变量名不同

```javascript
var person = {
    name: '小明',
    age: 20,
    gender: 'male',
    passport: 'G-12345678',
    school: 'No.4 middle school',
    address: {
        city: 'Beijing',
        street: 'No.1 Road',
        zipcode: '100001'
    }
};
var {name, address: {city, zip}} = person;
name; // '小明'
city; // 'Beijing'
zip; // undefined, 因为属性名是zipcode而不是zip
// 注意: address不是变量，而是为了让city和zip获得嵌套的address对象的属性:
address; // Uncaught ReferenceError: address is not defined

// 把passport属性赋值给变量id:
let {name, passport:id} = person;
name; // '小明'
id; // 'G-12345678'
// 注意: passport不是变量，而是为了让变量id获得passport属性:
passport; // Uncaught ReferenceError: passport is not defined

// 如果person对象没有single属性，默认赋值为true:
var {name, single=true} = person;
name; // '小明'
single; // true
```

有些时候，如果变量已经被声明了，再次赋值的时候，正确的写法也会报语法错误：

```javascript
// 声明变量:
var x, y;
// 解构赋值:
{x, y} = { name: '小明', x: 100, y: 200};
// 语法错误: Uncaught SyntaxError: Unexpected token =
```

这是因为JavaScript引擎把`{`开头的语句当作了块处理，于是`=`不再合法。解决方法是用小括号括起来：

```javascript
({x, y} = { name: '小明', x: 100, y: 200});
```

使用场景：简化代码 

> 例如，交换两个变量`x`和`y`的值，可以这么写，不再需要临时变量：
>
> var x=1, y=2;
>
> [x, y] = [y, x]
>
> 快速获取当前页面的域名和路径：
>
> var {hostname:domain, pathname:path} = location;
>
> 如果一个函数接收一个对象作为参数，那么，可以使用解构直接把对象的属性绑定到变量中。例如，下面的函数可以快速创建一个`Date`对象：
>
> function buildDate({year, month, day, hour=0, minute=0, second=0}) {
>
> ​    return new Date(year + '-' + month + '-' + day + ' ' + hour + ':' + minute + ':' + second);
>
> }
>
> 它的方便之处在于传入的对象只需要`year`、`month`和`day`这三个属性：
>
> buildDate({ year: 2017, month: 1, day: 1 });
>
> // Sun Jan 01 2017 00:00:00 GMT+0800 (CST)
>
> 也可以传入`hour`、`minute`和`second`属性：
>
> buildDate({ year: 2017, month: 1, day: 1, hour: 20, minute: 15 });
>
> // Sun Jan 01 2017 20:15:00 GMT+0800 (CST)
>
> 使用解构赋值可以减少代码量，但是，需要在支持ES6解构赋值特性的现代浏览器中才能正常运行。目前支持解构赋值的浏览器包括Chrome，Firefox，Edge等