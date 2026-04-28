## 3 方法

在一个对象中绑定函数，是这个对象的方法，举例

```javascript
var xiaoming = {
    name: '小明',
    birth: 1990,
    age: function () {
        var y = new Date().getFullYear();
        return y - this.birth;
    }
};

xiaoming.age; // function xiaoming.age()
xiaoming.age(); // 今年调用是25,明年调用就变成26了
```

#### this和that

- 方法中的this指当前对象

- 当方法中嵌套函数的时候，再使用this会有意想不到的结果...此时的this可能指向全局对象window，或者undefined

- 修复方法是使用一个that在函数一开始就捕获this，后续都使用that，利用嵌套函数变量作用域

  ```javascript
  'use strict';
  
  var xiaoming = {
      name: '小明',
      birth: 1990,
      age: function () {
          var that = this; // 在方法内部一开始就捕获this
          function getAgeFromBirth() {
              var y = new Date().getFullYear();
              return y - that.birth; // 用that而不是this
          }
          return getAgeFromBirth();
      }
  };
  
  xiaoming.age(); // 25
  ```

#### apply

函数本身都具有apply方法。apply方法可以接收两个参数，第一个参数是需要绑定的this变量，第二个是array，标识函数本身的参数。

上面的例子就可以写为

```javascript
function getAge() {
    var y = new Date().getFullYear();
    return y - this.birth;
}

var xiaoming{
    name:'小明',
    birth:1990,
    age:getAge
};

xiaoming.age(); 
getAge.apply(xiaoming,[]);
```

另一个和apply方法类似的是call，区别在于第二个入参

- `apply()`把参数打包成`Array`再传入；
- `call()`把参数按顺序传入。

```javascript
Math.max.apply(null, [3, 5, 4]); // 5
Math.max.call(null, 3, 5, 4); // 5
```

对普通函数调用，我们通常把`this`绑定为`null`。

#### 装饰器

利用`apply()`，我们还可以动态改变函数的行为。

比如在原函数执行前做些事情。

```javascript
'use strict';

var count = 0;
var oldParseInt = parseInt; // 保存原函数

window.parseInt = function () {
    // do sth before invocation
    count += 1;
    return oldParseInt.apply(null, arguments); // 调用原函数
};
// 测试:
parseInt('10');
parseInt('20');
parseInt('30');
console.log('count = ' + count); // 3
```

## 4 高阶函数

高阶函数英文叫Higher-order function。那么什么是高阶函数？

JavaScript的函数其实都指向某个变量。既然变量可以指向函数，函数的参数能接收变量，那么**一个函数就可以接收另一个函数作为参数**，这种函数就称之为高阶函数。

```javascript
function add(x, y, f) {
    return f(x) + f(y);
}
```

### 4.1 map/reduce

map/reduce是针对批量处理大数据的一种处理思路，就是对大数据，先拆解，然后组装；或者，按照英文名称含义，先做数据的匹配，然后整理输出。

#### map

把f(x)作用在Array的每一个元素并把结果生成一个新的Array

javaScript中map定义在Array中，含义如上。

也就是针对数组中的每个元素，做操作

```javascript
var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
arr.map(String); // ['1', '2', '3', '4', '5', '6', '7', '8', '9']
```

#### reduce

> Array的`reduce()`把一个函数作用在这个`Array`的`[x1, x2, x3...]`上，这个函数必须接收两个参数，`reduce()`把结果继续和序列的下一个元素做累积计算，其效果就是：
>
> ```javascript
> [x1, x2, x3, x4].reduce(f) = f(f(f(x1, x2), x3), x4)
> ```

```javascript
var arr = [1, 3, 5, 7, 9];
arr.reduce(function (x, y) {
    return x + y;
}); // 25
```

如果数组元素只有1个，那么还需要提供一个额外的初始参数以便至少凑够两：

```javascript
var arr = [123];
arr.reduce(function (x, y) {
    return x + y;
}, 0); // 123
```

#### 练习

请把用户输入的不规范的英文名字，变为首字母大写，其他小写的规范名字。输入：`['adam', 'LISA', 'barT']`，输出：`['Adam', 'Lisa', 'Bart']`。

```javascript
function normalize(arr) {
    let stand = function (str) {
        if (str.length === 1) {
            return str.toUpperCase();
        }
        let fir = str.substring(0, 1);
        let res = str.substring(1);
        return fir.toUpperCase() + res.toLowerCase();
    }

    return arr.map(stand);
}
// function normalize(arr) {
//     return arr.map(function get_norm(x) { return x.slice(0, 1).toUpperCase() + x.slice(1).toLowerCase(); });
// }
// 测试:
if (normalize(['adam', 'LISA', 'barT']).toString() === ['Adam', 'Lisa', 'Bart'].toString()) {
    console.log('测试通过!');
}
else {
    console.log('测试失败!');
}
```

小明希望利用`map()`把字符串变成整数，他写的代码很简洁：

```javascript
'use strict';

var arr = ['1', '2', '3'];
var r;
r = arr.map(parseInt);//应该使用Number，原因是parseInt实际是两个参数的函数方法，见下面链接
console.log(r);
```

结果竟然是`1, NaN, NaN`，小明百思不得其解，请帮他找到原因并修正代码。

提示：参考[[Array.prototype.map()的文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)。

### 4.2 filter

类似map函数，filter是Array的过滤函数，根据过滤条件对Array进行过滤。

例如，在一个`Array`中，删掉偶数，只保留奇数，可以这么写：

```javascript
var arr = [1, 2, 4, 5, 6, 9, 10, 15];
var r = arr.filter(function (x) {
    return x % 2 !== 0;
});
r; // [1, 5, 9, 15]
```

一般来说，filter函数就一个参数就够了，但是实际上可以有三个参数，后两个标识元素的位置和数组本身

```javascript
var arr = ['A', 'B', 'C'];
var r = arr.filter(function (element, index, self) {
    console.log(element); // 依次打印'A', 'B', 'C'
    console.log(index); // 依次打印0, 1, 2
    console.log(self); // self就是变量arr
    return true;
});
```

利用`filter`，可以巧妙地去除`Array`的重复元素：

```javascript
`use strict`
var
    r,
    arr = ['apple', 'strawberry', 'banana', 'pear', 'apple', 'orange', 'orange', 'strawberry'];
r = arr.filter(function (element, index, self) {
    return self.indexOf(element) === index;
});
console.log(r.toString());
```

#### 练习

请尝试用`filter()`筛选出素数：

```javascript
function get_primes(arr) {
    return arr.filter(function (x) {
        if (x === 1) {
            return false;
        }
        if (x === 2) {
            return true;
        }
        let i = 2;
        while (i < x) {
            if (x % i !== 0) {
                i++;
            } else {
                return false;
            }
        }
        return true;
    });
}

// 测试:
var
    x,
    r,
    arr = [];
for (x = 1; x < 100; x++) {
    arr.push(x);
}
r = get_primes(arr);
if (r.toString() === [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97].toString()) {
    console.log('测试通过!');
} else {
    console.log('测试失败: ' + r.toString());
}
```

### 4.3 sort

javaScript自带的Array的sort方法，会把元素都转成string，再按照ASCII码处理，所以对数字类型的排序很不友好。

解决办法就是sort中传入compare函数，比如

```javascript
var arr = [10, 20, 1, 2];
arr.sort(function (x, y) {
    return x - y;
}); // [1,2,10,20]
console.log(arr);
```

>  最后友情提示，`sort()`方法会直接对`Array`进行修改，它返回的结果仍是当前`Array`：
>
>  ```javascript
>  var a1 = ['B', 'A', 'C'];
>  var a2 = a1.sort();
>  a1; // ['A', 'B', 'C']
>  a2; // ['A', 'B', 'C']
>  a1 === a2; // true, a1和a2是同一对象
>  ```

### 4.4  Array

Array还提供了很多高阶函数

#### every()

可以判断数组的所有元素是否满足测试条件

```javascript
var arr = ['Apple', 'pear', 'orange'];
console.log(arr.every(function (s) {
    return s.length > 0;
})); // true, 因为每个元素都满足s.length>0

console.log(arr.every(function (s) {
    return s.toLowerCase() === s;
})); // false, 因为不是每个元素都全部是小写
```

#### find()

用于查找符合条件的第一个元素，如果找到了，返回这个元素，否则，返回`undefined`

```javascript
var arr = ['Apple', 'pear', 'orange'];
console.log(arr.find(function (s) {
    return s.toLowerCase() === s;
})); // 'pear', 因为pear全部是小写

console.log(arr.find(function (s) {
    return s.toUpperCase() === s;
})); // undefined, 因为没有全部是大写的元素
```

#### findIndex()

也是查找符合条件的第一个元素，不同之处在于`findIndex()`会返回这个元素的索引，如果没有找到，返回`-1`

#### forEach()

`forEach()`和`map()`类似，它也把每个元素依次作用于传入的函数，但不会返回新的数组。`forEach()`常用于遍历数组，因此，传入的函数不需要返回值

```javascript
var arr = ['Apple', 'pear', 'orange'];
arr.forEach(console.log); // 依次打印每个元素
```

##