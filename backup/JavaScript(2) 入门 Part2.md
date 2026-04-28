## 6 对象

- 访问属性是通过`.`操作符完成的，但这要求属性名必须是一个有效的变量名。如果属性名包含特殊字符，就必须用`''`括起来；`xiaohong`的属性名`middle-school`不是一个有效的变量，就需要用`''`括起来。访问这个属性也无法使用`.`操作符，必须用`['xxx']`来访问：
- 删除某一个属性，可以用delete
- 判断某个属性是否在对象中，可以用in
- 不过要小心，如果`in`判断一个属性存在，这个属性不一定是对象的，它可能是对象继承得到的
- 要判断一个属性是否是`xiaoming`自身拥有的，而不是继承得到的，可以用`hasOwnProperty()`方法：

```javascript
var xiaoming = {
    name: '小明'
};
xiaoming.age; // undefined
xiaoming.age = 18; // 新增一个age属性
xiaoming.age; // 18
delete xiaoming.age; // 删除age属性
xiaoming.age; // undefined
delete xiaoming['name']; // 删除name属性
xiaoming.name; // undefined
delete xiaoming.school; // 删除一个不存在的school属性也不会报错
'name' in xiaoming; // true
'grade' in xiaoming; // false
'toString' in xiaoming; // true
xiaoming.hasOwnProperty('name'); // true
xiaoming.hasOwnProperty('toString'); // false
```

## 7 条件判断

JavaScript使用`if () { ... } else { ... }`来进行条件判断，虽然某些情况下可以省略大括号，但建议是始终写{}

```java
var age = 3;
if (age >= 18) {
    alert('adult');
} else {
    if (age >= 6) {
        alert('teenager');
    } else {
        alert('kid');
    }
}
```

如果`if`的条件判断语句结果不是`true`或`false`怎么办？例如：

```javascript
var s = '123';
if (s.length) { // 条件计算结果为3
    //
}
```

JavaScript把`null`、`undefined`、`0`、`NaN`和空字符串`''`视为`false`，其他值一概视为`true`，因此上述代码条件判断的结果是`true`。

#### 练习

小明身高1.75，体重80.5kg。请根据BMI公式（体重除以身高的平方）帮小明计算他的BMI指数，并根据BMI指数

- 低于18.5：过轻
- 18.5-25：正常
- 25-28：过重
- 28-32：肥胖
- 高于32：严重肥胖

用`if...else...`判断并显示结果：

```javascript
'use strict';
var height = parseFloat(prompt('请输入身高(m):')); 
var weight = parseFloat(prompt('请输入体重(kg):'));
var bmi = weight/height/height;
if (bmi<18.5){
  console.log("过轻")
} else if (bmi<25){
  console.log("正常")
}else if (bmi<28){
  console.log("过重")
}else if (bmi<32){
  console.log("肥胖")
}else {
  console.log("严重肥胖")
}
```



## 8 循环

- 已知循环的开始和结束情况，用for最好；
- for循环还有for...in的变体，比较适用于对象的遍历；
- for可以不填写条件，for(;;)，需要使用break跳出循环；
- while和do{...}while都有，注意do...while至少执行一次

```javascript
for(int i=0;i<10;i++){
    
}

for(;;){
    break;
}

for(var key in obj){
    
}

while(true){}

do{}while(true);
```

`for`循环最常用的地方是利用索引来遍历数组：

```javascript
var arr = ['Apple', 'Google', 'Microsoft'];
var i, x;
for (i=0; i<arr.length; i++) {
    x = arr[i];
    console.log(x);
}
```

`for`循环的一个变体是`for ... in`循环，它可以把一个对象的所有属性依次循环出来：

```javascript
var o = {
    name: 'Jack',
    age: 20,
    city: 'Beijing'
};
for (var key in o) {
    console.log(key); // 'name', 'age', 'city'
}
```

要过滤掉对象继承的属性，用`hasOwnProperty()`来实现：

```javascript
var o = {
    name: 'Jack',
    age: 20,
    city: 'Beijing'
};
for (var key in o) {
    if (o.hasOwnProperty(key)) {
        console.log(key); // 'name', 'age', 'city'
    }
}
```

由于`Array`也是对象，而它的每个元素的索引被视为对象的属性，因此，`for ... in`循环可以直接循环出`Array`的索引：

```javascript
var a = ['A', 'B', 'C'];
for (var i in a) {
    console.log(i); // '0', '1', '2'
    console.log(a[i]); // 'A', 'B', 'C'
}
```

*请注意*，`for ... in`对`Array`的循环得到的是`String`而不是`Number`。

#### 练习

请利用循环遍历数组中的每个名字，并显示`Hello, xxx!`：

```javascript
'use strict';
var arr = ['Bart', 'Lisa', 'Adam'];
for (var i in arr){
  console.log(`Hello,${arr[i]}`);
}
```

## 9 Map和Set

js的对象基本类似Map结构，但是对象要求key必须为字符串；

为此引入了Map类型。

Map和Set是ES6规范，有可能浏览器不支持。

#### Map

```javascript
var m = new Map(); // 空Map
m.set('Adam', 67); // 添加新的key-value
m.set('Bob', 59);
m.has('Adam'); // 是否存在key 'Adam': true
m.get('Adam'); // 67
m.delete('Adam'); // 删除key 'Adam'
m.get('Adam'); // undefined
m.set('Adam', 67);
m.set('Adam', 88);
m.get('Adam'); // 88
```

初始化`Map`需要一个二维数组，或者直接初始化一个空`Map`

JS的问题就在于他有大量不规范的地方，你用不报错，但是结果诡异，硬要找原因就是在浪费时间，你觉得从结果能推导结论，很可能换个浏览器结果又变了。

永远只用规范的代码。

对于Map永远只用get()/set()/has()/delete()

参考：

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map

注意看MDN文档是如何劝你不要给Map加属性的：

> 请注意！为Map设置对象属性也是可以的，但是可能引起大量的混乱。

#### Set

`Set`和`Map`类似，也是一组key的集合，但不存储value。由于key不能重复，所以，在`Set`中，没有重复的key。

要创建一个`Set`，需要提供一个`Array`作为输入，或者直接创建一个空`Set`：

```javascript
var s = new Set([1, 2, 3, 3, '3']);
s; // Set {1, 2, 3, "3"}
s.add(4);
s; // Set {1, 2, 3, 4}
s.add(4);
s; // 仍然是 Set {1, 2, 3, 4}
var s = new Set([1, 2, 3]);
s; // Set {1, 2, 3}
s.delete(3);
s; // Set {1, 2}
```

set没有get方法，也不能索引直接取值，只能通过has()判断，或者遍历取值；遍历见下文，不能用上面的for...in...

for ... in循环由于历史遗留问题，它遍历的实际上是对象的属性名称。一个Array数组实际上也是一个对象，它的每个元素的索引被视为一个属性。

## 10 iterable

遍历Array可以采用下标循环，遍历Map和Set就无法使用下标。为了统一集合类型，ES6标准引入了新的iterable类型，Array、Map和Set都属于iterable类型。

具有iterable类型的集合可以通过新的for ... of循环来遍历。

就也是新的特性，有些浏览器可能不支持。

```javascript
var a = ['A', 'B', 'C'];
var s = new Set(['A', 'B', 'C']);
var m = new Map([[1, 'x'], [2, 'y'], [3, 'z']]);
for (var x of a) { // 遍历Array
    console.log(x);
}
for (var x of s) { // 遍历Set
    console.log(x);
}
for (var x of m) { // 遍历Map
    console.log(x[0] + '=' + x[1]);
}
```

#### `for ... of`循环和`for ... in`循环有何区别？

`for ... in`循环由于历史遗留问题，它遍历的实际上是对象的属性名称。一个`Array`数组实际上也是一个对象，它的每个元素的索引被视为一个属性。

当我们手动给`Array`对象添加了额外的属性后，`for ... in`循环将带来意想不到的意外效果：

```javascript
var a = ['A', 'B', 'C'];
a.name = 'Hello';
for (var x in a) {
    console.log(x); // '0', '1', '2', 'name'
}
```

`for ... in`循环将把`name`包括在内，但`Array`的`length`属性却不包括在内。

`for ... of`循环则完全修复了这些问题，它只循环集合本身的元素：

更好的方式是直接使用`iterable`内置的`forEach`方法，它接收一个函数，每次迭代就自动回调该函数。以`Array`为例，`Map`的回调函数参数依次为`value`、`key`和`map`本身：

```javascript
var s = new Set(['A', 'B', 'C']);
s.forEach(function (element, sameElement, set) {
    console.log(element);
});
var m = new Map([[1, 'x'], [2, 'y'], [3, 'z']]);
m.forEach(function (value, key, map) {
    console.log(value);
});
```

## 参考资料

https://www.liaoxuefeng.com/wiki/1022910821149312/1023020895584256