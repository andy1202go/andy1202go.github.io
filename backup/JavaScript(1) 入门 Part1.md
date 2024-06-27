> 简单地说，JavaScript是一种运行在浏览器中的解释型的编程语言。
>
> 为什么起名叫JavaScript？原因是当时Java语言非常红火，所以网景公司希望借Java的名气来推广，但事实上JavaScript除了语法上有点像Java，其他部分基本上没啥关系。

只要有网页的地方，基本都有js实现的交互逻辑。

## 1 最常用的是

- 怎么嵌入html：直接script标签中写入，或者script标签用src属性引入js文件；推荐后者

  ```javascript
  <script src="scripts/abbey/15.js"></script>
  或者
  <script type="text/javascript">
    ...
  </script>
  ```

- 如何编写：推荐vscode

- 如何运行：要有html页面

  > 你也许会想，直接在我的硬盘上创建好HTML和JavaScript文件，然后用浏览器打开，不就可以看到效果了吗？
  >
  > 这种方式运行部分JavaScript代码没有问题，但由于浏览器的安全限制，以`file://`开头的地址无法执行如联网等JavaScript代码，最终，你还是需要架设一个Web服务器，然后以`http://`开头的地址来正常执行所有JavaScript代码。

  就使用python自带的server服务吧

  ```python
  python -m http.server
  ```

- helloworld:alter()

  ```html
  <html>
  <head>
    <script>
      alert('Hello, world');
    </script>
  </head>
  <body>
    ...
  </body>
  </html>
  ```

## 2 基本语法

- 最好以分号结尾
- 不强制要求缩进
- 单行注释//，多行注释/**/
- 严格区分大小写

## 3 数据类型和变量

### Number

- JavaScript不区分整数和浮点数，统一用Number表示

  ```javascript
  123; // 整数123
  0.456; // 浮点数0.456
  1.2345e3; // 科学计数法表示1.2345x1000，等同于1234.5
  -99; // 负数
  NaN; // NaN表示Not a Number，当无法计算结果时用NaN表示
  Infinity; // Infinity表示无限大，当数值超过了JavaScript的Number所能表示的最大值时，就表示为Infinity
  0xa5b4c3d2; //16进制表达，0x开头
  ```

- 正常的四则运算，注意计算得到NaN的情况

  ```javascript
  1 + 2; // 3
  (1 + 2) * 5 / 2; // 7.5
  2 / 0; // Infinity
  0 / 0; // NaN
  10 % 3; // 1
  10.5 % 3; // 1.5
  ```

### 字符串

- 单引号双引号都行

### 布尔值

```javascript
true && true
true || false
!false
```

### 比较运算符

除了一般的大于小于，几个比较特殊的要重点注意

- 等于的比较，请严格使用===！==是设计缺陷，不同类型比较会自动转换类型进行比较，得到不可预计的结果

- NaN不等于任何值，甚至不等于NaN自己；只能用isNaN(NaN)来判断

- 浮点数的比较，受限于计算机对无限小数的精度问题，只能特殊处理如下

  ```javascript
  2 > 5; // false
  5 >= 2; // true
  7 == 7; // true
  false == 0; // true
  false === 0; // false
  NaN === NaN; // false
  isNaN(NaN); // true
  1 / 3 === (1 - 2 / 3); // false
  Math.abs(1 / 3 - (1 - 2 / 3)) < 0.0000001; // true
  ```

### null和undifined

- null表示空值，和0和‘’都不通

- > JavaScript的设计者希望用`null`表示一个空的值，而`undefined`表示值未定义。事实证明，这并没有什么卵用，区分两者的意义不大。大多数情况下，我们都应该用`null`。`undefined`仅仅在判断函数参数是否传递的情况下有用。

### 数组

- 是一组按顺序排列的集合，集合的每个值被称为元素
- 数组中的元素可以是任意数据类型，且一个数组中的都不用完全一致
- []中，逗号分隔
- 强烈建议直接使用`[]`初始化，而不是new Array()

```javascript
[1, 2, 3.14, 'Hello', null, true];
new Array(1, 2, 3); // 创建了数组[1, 2, 3]
var arr = [1, 2, 3.14, 'Hello', null, true];
arr[0]; // 返回索引为0的元素，即1
arr[5]; // 返回索引为5的元素，即true
arr[6]; // 索引超出了范围，返回undefined
```

### 对象

- 一组由键值对组成的无需集合
- 获取对象的属性是对象变量.属性名

```javascript
var person={
    name:'Shit',
    age:20,
    zipCode:null,
    tags:['js','java']
}
person.name;
person.tags;
person.tags;
```

### 变量

- 动态语言，不需要在初始化时候指定类型
- 变量名是大小写英文、数字、`$`和`_`的组合，且不能用数字开头
- strict模式下，必需有var才能申明变量

```javascript
'use strict'; //启用strict模式
var a; // 申明了变量a，此时a的值为undefined
var $b = 1; // 申明了变量$b，同时给$b赋值，此时$b的值为1
var s_007 = '007'; // s_007是一个字符串
var Answer = true; // Answer是一个布尔值true
var t = null; // t的值是null
abc = 'Hello, world'; //报错
```

### 类型转换

https://www.runoob.com/js/js-type-conversion.html

- typeof判断类型
- NaN 的数据类型是 number
- 数组(Array)的数据类型是 object
- 日期(Date)的数据类型为 object
- null 的数据类型是 object
- 未定义变量的数据类型为 undefined

javaScript 变量可以转换为新变量或其他数据类型：

- 通过使用 JavaScript 函数
- 通过 JavaScript 自身自动转换
- **String()** 可以将数字、布尔值转换为字符串。
- Date() 返回字符串。
- 全局方法 **Number()** 可以将字符串、布尔值、日期转换为数字。
- 运算符+可用于将变量转换为数字

## 4 字符串

- ASCII字符可以以`\x##`形式的十六进制表示

  ```javascript
  '\x41'; // 完全等同于 'A'
  ```

- 还可以用`\u####`表示一个Unicode字符

```javascript
'\u4e2d\u6587'; // 完全等同于 '中文''\x41'
```

### 多行字符串

用反引号，注意有换行符

```javascript
var strLong = `fd
asd
dfd`;
```

### 模板字符串

- 几个字符串可以用加号连接
- 更多的可以用模板字符串连接

```javascript
var name = '小明';
var age = 20;
var message = '你好, ' + name + ', 你今年' + age + '岁了!';
alert(message);

var name = '小明';
var age = 20;
var message = `你好, ${name}, 你今年${age}岁了!`;
alert(message);
```

### 常用函数

```javascript
var s = 'Hello, world!';
s.length;
s[0];
s.toUpperCase();
s.toLowerCase();
s.indexOf("H");
s.subString(0,3);
s.includes("sdfa");
```

- *需要特别注意的是*，字符串是不可变的，如果对字符串的某个索引赋值，不会有任何错误，但是，也没有任何效果

## 5 数组

### 长度

- 数组的长度，是.length
- 可以直接改变长度，不会报错，但不建议这么做；访问到没有值的时候，报undefined
- 可以直接修改某个位置的值

```javascript
var arr = [1, 2, 3];
arr.length; // 3
arr.length = 6;
arr; // arr变为[1, 2, 3, undefined, undefined, undefined]
arr.length = 2;
arr; // arr变为[1, 2]
var arr = [1, 2, 3];
arr[5] = 'x';
arr; // arr变为[1, 2, 3, undefined, undefined, 'x']
```

### indexOf

可以通过`indexOf()`来搜索一个指定的元素的位置：即获取索引

```javascript
var arr = [10, 20, '30', 'xyz'];
arr.indexOf(10); // 元素10的索引为0
arr.indexOf(20); // 元素20的索引为1
arr.indexOf(30); // 元素30没有找到，返回-1
arr.indexOf('30'); // 元素'30'的索引为2
```

### slice

- 就是String的subString的数组版本，截取数组的一部分返回；
- 也是左闭右开

```javascript
var arr = ['A', 'B', 'C', 'D', 'E', 'F', 'G'];
arr.slice(0, 3); // 从索引0开始，到索引3结束，但不包括索引3: ['A', 'B', 'C']
arr.slice(3); // 从索引3开始到结束: ['D', 'E', 'F', 'G']
```

### push和pop

- 队尾的操作
- pop()是有返回的

```javascript
var arr = [1, 2];
arr.push('A', 'B'); // 返回Array新的长度: 4
arr; // [1, 2, 'A', 'B']
arr.pop(); // pop()返回'B'
arr; // [1, 2, 'A']
arr.pop(); arr.pop(); arr.pop(); // 连续pop 3次
arr; // []
arr.pop(); // 空数组继续pop不会报错，而是返回undefined
arr; // []
```

### unshift和shift

- 直译是“移位”，就是平移
- 所以是在头部的增删动作
- shift()是有返回的

```javascript
var arr = [1, 2];
arr.unshift('A', 'B'); // 返回Array新的长度: 4
arr; // ['A', 'B', 1, 2]
arr.shift(); // 'A'
arr; // ['B', 1, 2]
arr.shift(); arr.shift(); arr.shift(); // 连续shift 3次
arr; // []
arr.shift(); // 空数组继续shift不会报错，而是返回undefined
arr; // []
```

### sort

- 正序排序
- 修改了原数组

### reverse

和sort的排序顺序正相反，其他一致

### splice

- 没有见过的方法，直译是“接合”
- 可以从指定的索引开始删除若干元素，然后再从该位置添加若干元素：

```javascript
var arr = ['Microsoft', 'Apple', 'Yahoo', 'AOL', 'Excite', 'Oracle'];
// 从索引2开始删除3个元素,然后再添加两个元素:
arr.splice(2, 3, 'Google', 'Facebook'); // 返回删除的元素 ['Yahoo', 'AOL', 'Excite']
arr; // ['Microsoft', 'Apple', 'Google', 'Facebook', 'Oracle']
// 只删除,不添加:
arr.splice(2, 2); // ['Google', 'Facebook']
arr; // ['Microsoft', 'Apple', 'Oracle']
// 只添加,不删除:
arr.splice(2, 0, 'Google', 'Facebook'); // 返回[],因为没有删除任何元素
arr; // ['Microsoft', 'Apple', 'Google', 'Facebook', 'Oracle']
```

- 直观感觉是，真tm灵活

### concat

- 数组连接
- 注意没有修改原数组，而是返回了新数组

```javascript
var arr = ['A', 'B', 'C'];
var added = arr.concat([1, 2, 3]);
added; // ['A', 'B', 'C', 1, 2, 3]
arr; // ['A', 'B', 'C']
```

### join

把当前array的每个元素用指定的字符串连接，并返回连接后的字符串

```javascript
var arr = ['A', 'B', 'C', 1, 2, 3];
arr.join('-'); // 'A-B-C-1-2-3'
```

### 多维数组

```javascript
var arr = [[1, 2, 3], [400, 500, 600], '-'];
var x = arr[1][1];
```

## 