在JavaScript的世界里，一切都是对象。

但是某些对象还是和其他对象不太一样。为了区分对象的类型，我们用`typeof`操作符获取对象的类型，它总是返回一个字符串：

```javascript
typeof 123; // 'number'
typeof NaN; // 'number'
typeof 'str'; // 'string'
typeof true; // 'boolean'
typeof undefined; // 'undefined'
typeof Math.abs; // 'function'
typeof null; // 'object'
typeof []; // 'object'
typeof {}; // 'object'
```

可见，`number`、`string`、`boolean`、`function`和`undefined`有别于其他类型。特别注意`null`的类型是`object`，`Array`的类型也是`object`，如果我们用`typeof`将无法区分出`null`、`Array`和通常意义上的object——`{}`。

### 包装对象

类似Java，JavaScript提供了基础类型的包装对象。

不一样的在于，JavaScript中string都有包装类。

先上结论：**不要使用包装类！！！**

```javascript
var n = new Number(123); // 123,生成了新的包装类型
var b = new Boolean(true); // true,生成了新的包装类型
var s = new String('str'); // 'str',生成了新的包装类型
typeof new Number(123); // 'object'
new Number(123) === 123; // false

typeof new Boolean(true); // 'object'
new Boolean(true) === true; // false

typeof new String('str'); // 'object'
new String('str') === 'str'; // false
```

如果我们在使用`Number`、`Boolean`和`String`时，没有写`new`会发生什么情况？

此时，`Number()`、`Boolean`和`String()`被当做普通函数，把任何类型的数据转换为`number`、`boolean`和`string`类型（注意不是其包装类型）

反正就是吐血...

### 数据转换总结

- 不要使用`new Number()`、`new Boolean()`、`new String()`创建包装对象；

- 用`parseInt()`或`parseFloat()`来转换任意类型到`number`；

- 用`String()`来转换任意类型到`string`，或者直接调用某个对象的`toString()`方法；

- 通常不必把任意类型转换为`boolean`再判断，因为可以直接写`if (myVar) {...}`；

- `typeof`操作符可以判断出`number`、`boolean`、`string`、`function`和`undefined`；

- 判断`Array`要使用`Array.isArray(arr)`；

- 判断`null`请使用`myVar === null`；

- 判断某个全局变量是否存在用`typeof window.myVar === 'undefined'`；

- 函数内部判断某个变量是否存在用`typeof myVar === 'undefined'`。

- null和undefined没有toString方法

- number的toString，要注意小数点问题

  ```javascript
  123.toString(); // SyntaxError
  123..toString(); // '123', 注意是两个点！
  (123).toString(); // '123'
  ```

## 1Date

不得不说，JavaScript的Date类型还是挺好用的，除了坑爹的月份，其他都好说。

1. 要获取系统当前时间，用：

   ```javascript
   var now = new Date();
   now; // Wed Jun 24 2015 19:49:22 GMT+0800 (CST)
   now.getFullYear(); // 2015, 年份
   now.getMonth(); // 5, 月份，注意月份范围是0~11，5表示六月
   now.getDate(); // 24, 表示24号
   now.getDay(); // 3, 表示星期三
   now.getHours(); // 19, 24小时制
   now.getMinutes(); // 49, 分钟
   now.getSeconds(); // 22, 秒
   now.getMilliseconds(); // 875, 毫秒数
   now.getTime(); // 1435146562875, 以number形式表示的时间戳
   ```

   注意，当前时间是浏览器从本机操作系统获取的时间，所以不一定准确，因为用户可以把当前时间设定为任何值。

2. 创建Date

   ```javascript
   var d = new Date(2015, 5, 19, 20, 15, 30, 123);
   d; // Fri Jun 19 2015 20:15:30 GMT+0800 (CST)
   var d = Date.parse('2015-06-24T19:49:22.875+08:00');
   d; // 1435146562875
   ```

   > 你可能观察到了一个*非常非常坑爹*的地方，就是JavaScript的月份范围用整数表示是0~11，`0`表示一月，`1`表示二月……，所以要表示6月，我们传入的是`5`！这绝对是JavaScript的设计者当时脑抽了一下，但是现在要修复已经不可能了。

3. 时区

   ```javascript
   var d = new Date(1435146562875);
   d.toLocaleString(); // '2015/6/24 下午7:49:22'，本地时间（北京时区+8:00），显示的字符串与操作系统设定的格式有关
   d.toUTCString(); // 'Wed, 24 Jun 2015 11:49:22 GMT'，UTC时间，与本地时间相差8小时
   ```

## 2 RegExp

正则表达式的实现。

### 基础或常用知识

在正则表达式中

- 如果直接给出字符，就是精确匹配
- \表示转义
- \d 匹配数字
- \w 匹配字母或数字
- . 匹配任意字符
- `*` 表示任意个字符
- `+` 表示至少一个字符
- ? 表示0个或1个字符
- {n} 表示n个字符
- {n,m} 表示n-m个字符
- \s 匹配空格
- [] 表示范围，就是范围内的匹配到就行
- A|B 可以匹配A或B
- ^ 表示行的开头
- $ 表示行的结束。如果是整行匹配，就属于必须精确匹配的

更多内容参考文章末尾链接，MDN的文章。

### RegExp

在JavaScript中使用正则表达式：

1. 创建

   两种方式：

   1. /正则表达式/

   2. new RegExp('正则表达式');

      ```javascript
      var re1 = /ABC\-001/;
      var re2 = new RegExp('ABC\\-001');
      
      re1; // /ABC\-001/
      re2; // /ABC\-001/
      ```

2. 判断字符串是否匹配

   test()方法，用RegExp的test方法

   ```javascript
   var re = /^\d{3}\-\d{3,8}$/;
   re.test('010-12345'); // true
   re.test('010-1234x'); // false
   re.test('010 12345'); // false
   ```

3. 匹配后提取信息

   除了简单地判断是否匹配之外，正则表达式还有提取子串的强大功能。用`()`表示的就是要提取的分组（Group）

   ```javascript
   var re = /^(\d{3})-(\d{3,8})$/;
   re.exec('010-12345'); // ['010-12345', '010', '12345']
   re.exec('010 12345'); // null
   ```

   exec方法在匹配失败时返回null

4. 切分字符串

   属于不务正业的用法

   ```javascript
   'a b   c'.split(' '); // ['a', 'b', '', '', 'c']
   'a b   c'.split(/\s+/); // ['a', 'b', 'c']
   'a,b, c  d'.split(/[\s\,]+/); // ['a', 'b', 'c', 'd']
   'a,b;; c  d'.split(/[\s\,\;]+/); // ['a', 'b', 'c', 'd']
   ```

5. 贪婪匹配

   默认是贪婪匹配，就是匹配到后会尽可能延长字符串。

   如果不想这么匹配，就要加上?

   ```javascript
   var re = /^(\d+)(0*)$/;
   re.exec('102300'); // ['102300', '102300', '']
   var re = /^(\d+?)(0*)$/;
   re.exec('102300'); // ['102300', '1023', '00']
   ```

6. 全局搜索

   JavaScript的正则表达式还有几个特殊的标志，最常用的是`g`，表示全局匹配。

   所谓全局匹配是多次匹配，或者说匹配到第一个之后，可以继续往后匹配下一个。

   ```javascript
   var r1 = /test/g;
   // 等价于:
   var r2 = new RegExp('test', 'g');
   
   ar s = 'JavaScript, VBScript, JScript and ECMAScript';
   var re=/[a-zA-Z]+Script/g;
   
   // 使用全局匹配:
   re.exec(s); // ['JavaScript']
   re.lastIndex; // 10
   
   re.exec(s); // ['VBScript']
   re.lastIndex; // 20
   
   re.exec(s); // ['JScript']
   re.lastIndex; // 29
   
   re.exec(s); // ['ECMAScript']
   re.lastIndex; // 44
   
   re.exec(s); // null，直到结束仍没有匹配到
   ```

   全局匹配类似搜索，因此不能使用`/^...$/`，那样只会最多匹配一次。

   正则表达式还可以指定`i`标志，表示忽略大小写，`m`标志，表示执行多行匹配。

### 练习

请尝试写一个验证Email地址的正则表达式。版本一应该可以验证出类似的Email：

```javascript
'use strict';
var re = /^[0-9a-zA-Z.]+@[0-9a-zA-Z]+\.[0-9a-zA-Z]+$/;
// 测试:
var
    i,
    success = true,
    should_pass = ['someone@gmail.com', 'bill.gates@microsoft.com', 'tom@voyager.org', 'bob2015@163.com'],
    should_fail = ['test#gmail.com', 'bill@microsoft', 'bill%gates@ms.com', '@voyager.org'];
for (i = 0; i < should_pass.length; i++) {
    if (!re.test(should_pass[i])) {
        console.log('测试失败: ' + should_pass[i]);
        success = false;
        break;
    }
}
for (i = 0; i < should_fail.length; i++) {
    if (re.test(should_fail[i])) {
        console.log('测试失败: ' + should_fail[i]);
        success = false;
        break;
    }
}
if (success) {
    console.log('测试通过!');
}
```

版本二可以验证并提取出带名字的Email地址：

```javascript
'use strict';
var re = /^<([A-Za-z]+\s[A-Za-z]+)>\s?([0-9a-zA-Z.]+@[0-9a-zA-Z]+\.[0-9a-zA-Z]+)$/;
// 测试:
var r = re.exec('<Tom Paris> tom@voyager.org');
if (r === null || r.toString() !== ['<Tom Paris> tom@voyager.org', 'Tom Paris', 'tom@voyager.org'].toString()) {
    console.log('测试失败!');
}
else {
    console.log('测试成功!');
}
```

## 3 JSON

### 创造JSON的历史

为了替代冗长复杂的xml来传递数据

> JSON是JavaScript Object Notation的缩写，它是一种数据交换格式。
>
> 在JSON出现之前，大家一直用XML来传递数据。因为XML是一种纯文本格式，所以它适合在网络上交换数据。XML本身不算复杂，但是，加上DTD、XSD、XPath、XSLT等一大堆复杂的规范以后，任何正常的软件开发人员碰到XML都会感觉头大了，最后大家发现，即使你努力钻研几个月，也未必搞得清楚XML的规范。
>
> 终于，在2002年的一天，道格拉斯·克罗克福特（Douglas Crockford）同学为了拯救深陷水深火热同时又被某几个巨型软件企业长期愚弄的软件工程师，发明了JSON这种超轻量级的数据交换格式。

> 道格拉斯同学长期担任雅虎的高级架构师，自然钟情于JavaScript。他设计的JSON实际上是JavaScript的一个子集。在JSON中，一共就这么几种数据类型：
>
> - number：和JavaScript的`number`完全一致；
> - boolean：就是JavaScript的`true`或`false`；
> - string：就是JavaScript的`string`；
> - null：就是JavaScript的`null`；
> - array：就是JavaScript的`Array`表示方式——`[]`；
> - object：就是JavaScript的`{ ... }`表示方式。
>
> 以及上面的任意组合。
>
> 并且，JSON还定死了字符集必须是UTF-8，表示多语言就没有问题了。为了统一解析，JSON的字符串规定必须用双引号`""`，Object的键也必须用双引号`""`。

### 序列化

JavaScript内置了JSON的解析。

- 把对象转为JSON并输出

  ```javascript
  'use strict';
  
  var xiaoming = {
      name: '小明',
      age: 14,
      gender: true,
      height: 1.65,
      grade: null,
      'middle-school': '\"W3C\" Middle School',
      skills: ['JavaScript', 'Java', 'Python', 'Lisp']
  };
  var s = JSON.stringify(xiaoming);
  console.log(s);
  ```

  要输出得好看一些，可以加上参数，按缩进输出：

  ```javascript
  JSON.stringify(xiaoming, null, '  ');
  ```

- 第二个参数可以控制输出内容的范围和格式

  比如只输出name和age字段

  ```javascript
  JSON.stringify(xiaoming,['name','age'],' ');
  ```

  比如string类型字段要大写

  ```javascript
  function convert(key, value) {
      if (typeof value === 'string') {
          return value.toUpperCase();
      }
      return value;
  }
  
  JSON.stringify(xiaoming, convert, '  ');
  ```

- 在对象中，用toJSON()方法来精确控制，返回name和skill并且skill大写

  ```javascript
  var xiaoming = {
      name: '小明',
      age: 14,
      gender: true,
      height: 1.65,
      grade: null,
      'middle-school': '\"W3C\" Middle School',
      skills: ['JavaScript', 'Java', 'Python', 'Lisp'],
      toJSON: function(){
          return {
              'Name':this.name,
              'Skills':this.skills
          }
      }
  };
  
  function convert(key, value) {
      if (typeof value === 'string') {
          return value.toUpperCase();
      }
      return value;
  }
  
  console.log(JSON.stringify(xiaoming, convert, '  '));
  ```

### 反序列化

我们直接用`JSON.parse()`把它变成一个JavaScript对象

```javascript
JSON.parse('[1,2,3,true]'); // [1, 2, 3, true]
JSON.parse('{"name":"小明","age":14}'); // Object {name: '小明', age: 14}
JSON.parse('true'); // true
JSON.parse('123.45'); // 123.45
```

`JSON.parse()`还可以接收一个函数，用来转换解析出的属性：

```javascript
var obj = JSON.parse('{"name":"小明","age":14}', function (key, value) {
    if (key === 'name') {
        return value + '同学';
    }
    return value;
});
console.log(JSON.stringify(obj)); // {name: '小明同学', age: 14}
```

### 练习

用浏览器访问OpenWeatherMap的[天气API](https://api.openweathermap.org/data/2.5/forecast?q=Beijing,cn&appid=800f49846586c3ba6e7052cfc89af16c)，查看返回的JSON数据，然后返回城市、天气预报等信息

```html
<script src="https://code.jquery.com/jquery-3.3.1.js" integrity="sha256-2Kok7MbOyxpgUVvAk/HJ2jigOSYS2auK4Pfzbm7uH60=" crossorigin="anonymous"></script>
```

```javascript
var url = 'https://api.openweathermap.org/data/2.5/forecast?q=Beijing,cn&appid=800f49846586c3ba6e7052cfc89af16c';
$.getJSON(url, function (data) {
    console.log(JSON.stringify(data,null,' '));
});
```

## 参考资料

1. [正则表达式- JavaScript - MDN Web Docs](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_expressions)

