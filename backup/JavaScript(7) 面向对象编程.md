JavaScript虽然所有数据都是对象，但不是就能原生实现面向对象编程的。

因为JavaScript里面是没有类的，都是实例...

曲线救国的方案是通过原型来实现。

原型是指当我们想要创建`xiaoming`这个具体的学生时，我们并没有一个`Student`类型可用。那怎么办？恰好有这么一个现成的对象：

```
var robot = {
    name: 'Robot',
    height: 1.6,
    run: function () {
        console.log(this.name + ' is running...');
    }
};
```

我们看这个`robot`对象有名字，有身高，还会跑，有点像小明，干脆就根据它来“创建”小明得了！
于是我们把它改名为`Student`，然后创建出`xiaoming`：

```javascript
var Student = {
    name: 'Robot',
    height: 1.2,
    run: function () {
        console.log(this.name + ' is running...');
    }
};

var xiaoming = {
    name: '小明'
};

xiaoming.__proto__ = Student;
```

注意最后一行代码把`xiaoming`的原型指向了对象`Student`，看上去`xiaoming`仿佛是从`Student`继承下来的：

```javascript
xiaoming.name; // '小明'
xiaoming.run(); // 小明 is running...
```

就是先实例化父类实例，然后使用 is a ，表示需要的类是啥。

在JavaScrip代码运行时期，你可以把`xiaoming`从`Student`变成`Bird`，或者变成任何对象。

*请注意*，上述代码仅用于演示目的。在编写JavaScript代码时，不要直接用`obj.__proto__`去改变一个对象的原型，并且，低版本的IE也无法使用`__proto__`。`Object.create()`方法可以传入一个原型对象，并创建一个基于该原型的新对象，但是新对象什么属性都没有，因此，我们可以编写一个函数来创建

```javascript
// 原型对象:
var Student = {
    name: 'Robot',
    height: 1.2,
    run: function () {
        console.log(this.name + ' is running...');
    }
};

function createStudent(name) {
    // 基于Student原型创建一个新对象:
    var s = Object.create(Student);
    // 初始化新对象:
    s.name = name;
    return s;
}

var xiaoming = createStudent('小明');
xiaoming.run(); // 小明 is running...
xiaoming.__proto__ === Student; // true
```

## 1 创建对象

JavaScript对每个创建的对象都会设置一个原型，指向它的原型对象。

当我们用`obj.xxx`访问一个对象的属性时，JavaScript引擎先在当前对象上查找该属性，如果没有找到，就到其原型对象上找，如果还没有找到，就一直上溯到`Object.prototype`对象，最后，如果还没有找到，就只能返回`undefined`。

例如，创建一个`Array`对象：

```javascript
var arr = [1, 2, 3];
```

其原型链是：

```javascript
arr ----> Array.prototype ----> Object.prototype ----> null
```

函数也是对象，函数也是一个对象，它的原型链是：

```javascript
foo ----> Function.prototype ----> Object.prototype ----> null
```

之前用过的Array的一些方法，就是定义在Array.prototype里面的；apply()是定义在Function.prototype里面的。

### 构造函数

JavaScript提供了除了直接{}创建对象的方式，构造函数。

先定义一个构造函数

```javascript
function Student(name) {
    this.name = name;
    this.hello = function () {
        alert('Hello, ' + this.name + '!');
    }
}
```

使用new来生成一个Student的对象

```javascript
var xiaoming = new Student('小明');
xiaoming.name; // '小明'
xiaoming.hello(); // Hello, 小明!
```

几点注意事项：

- 构造函数，也是函数，也就是对象

- 构造函数没有return，如果直接用，会返回undefined

- xiaoming的原型链是：

  ```javascript
  xiaoming ----> Student.prototype ----> Object.prototype ----> null
  ```

- 用`new Student()`创建的对象还从原型上获得了一个`constructor`属性，它指向函数`Student`本身：

  ```javascript
  xiaoming.constructor === Student.prototype.constructor; // true
  Student.prototype.constructor === Student; // true
  
  Object.getPrototypeOf(xiaoming) === Student.prototype; // true
  
  xiaoming instanceof Student; // true
  ```

  乱是真的乱...

如果新建一个对象xiaohong，xiaoming和xiaohong虽然都有同一个方法，但是是不同的，虽然函数名称和代码都是相同的！

```javascript
xiaoming.name; // '小明'
xiaohong.name; // '小红'
xiaoming.hello; // function: Student.hello()
xiaohong.hello; // function: Student.hello()
xiaoming.hello === xiaohong.hello; // false
```

要想让创建的对象共享某些方法，应该把方法写在prototype中，Array和Function就是这么做的吧

```javascript
function Student(name) {
    this.name = name;
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
};
```

### 不要忘记写new

如果直接用了Student，后果很严重

> 在strict模式下，`this.name = name`将报错，因为`this`绑定为`undefined`，在非strict模式下，`this.name = name`不报错，因为`this`绑定为`window`，于是无意间创建了全局变量`name`，并且返回`undefined`，这个结果更糟糕。
>
> 所以，调用构造函数千万不要忘记写`new`。为了区分普通函数和构造函数，按照约定，构造函数首字母应当大写，而普通函数首字母应当小写，这样，一些语法检查工具如[[jslint](http://www.jslint.com/)](http://www.jslint.com/)将可以帮你检测到漏写的`new`。

构造函数，约定俗成，**首字母大写！！！**

### 最佳实践

```javascript
function Student(props) {
    this.name = props.name || '匿名'; // 默认值为'匿名'
    this.grade = props.grade || 1; // 默认值为1
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
};

function createStudent(props) {
    return new Student(props || {})
}

var xiaoming = createStudent({
    name: '小明'
});

xiaoming.grade; // 1
```

1. 工厂模式，create封装所有new操作
2. props参数，可以传入自定义参数，也可以有默认值

### 练习

请利用构造函数定义`Cat`，并让所有的Cat对象有一个`name`属性，并共享一个方法`say()`，返回字符串`'Hello, xxx!'`：

```javascript
'use strict';
function Cat(name) {
    this.name = name;
}

Cat.prototype.say = function () {
    return "Hello, " + this.name + "!";
}

function createCat(name) {
    return new Cat(name);
}

// 测试:
var kitty = new Cat('Kitty');
var doraemon = new Cat('哆啦A梦');
if (kitty && kitty.name === 'Kitty'
    && kitty.say
    && typeof kitty.say === 'function'
    && kitty.say() === 'Hello, Kitty!'
    && kitty.say === doraemon.say
) {
    console.log('测试通过!');
} else {
    console.log('测试失败!');
}
```

## 2 原型继承

一笔乱账...

直接记住最佳实践吧

> JavaScript的原型继承实现方式就是：
>
> 1. 定义新的构造函数，并在内部用`call()`调用希望“继承”的构造函数，并绑定`this`；
> 2. 借助中间函数`F`实现原型链继承，最好通过封装的`inherits`函数完成；
> 3. 继续在新的构造函数的原型上定义新方法。

```javascript
// 2 借助中间函数`F`实现原型链继承，最好通过封装的`inherits`函数完成；
function inherits(Child, Parent) {
    var F = function () {};
    F.prototype = Parent.prototype;
    Child.prototype = new F();
    Child.prototype.constructor = Child;
}

function Student(props) {
    this.name = props.name || 'Unnamed';
}

Student.prototype.hello = function () {
    alert('Hello, ' + this.name + '!');
}

// 1 定义新的构造函数，并在内部用`call()`调用希望“继承”的构造函数，并绑定`this`；
function PrimaryStudent(props) {
    Student.call(this, props);
    this.grade = props.grade || 1;
}

// 实现原型继承链:
inherits(PrimaryStudent, Student);

// 绑定其他方法到PrimaryStudent原型:
// 3 继续在新的构造函数的原型上定义新方法。
PrimaryStudent.prototype.getGrade = function () {
    return this.grade;
};
```

## 3 class继承

es6引入了class关键字，对类的定义还有继承都是极大的利好。

```javascript
class Student {
    constructor(name) {
        this.name = name;
    }

    hello() {
        alert('Hello, ' + this.name + '!');
    }
}
```

继承

```javascript
class PrimaryStudent extends Student {
    constructor(name, grade) {
        super(name); // 记得用super调用父类的构造方法!
        this.grade = grade;
    }

    myGrade() {
        alert('I am at grade ' + this.grade);
    }
}
```

### 练习

请利用`class`重新定义`Cat`，并让它从已有的`Animal`继承，然后新增一个方法`say()`，返回字符串`'Hello, xxx!'`：

```javascript
'use strict';
class Animal {
    constructor(name) {
        this.name = name;
    }
}

class Cat extends Animal {
    constructor(name) {
        super(name);
    }

    say() {
        return "Hello, " + this.name + "!";
    }
}

// 测试:
var kitty = new Cat('Kitty');
var doraemon = new Cat('哆啦A梦');
if ((new Cat('x') instanceof Animal)
    && kitty
    && kitty.name === 'Kitty'
    && kitty.say
    && typeof kitty.say === 'function'
    && kitty.say() === 'Hello, Kitty!'
    && kitty.say === doraemon.say) {
    console.log('测试通过!');
} else {
    console.log('测试失败!');
}
```

## 参考资料

1. [[面向对象编程](https://www.liaoxuefeng.com/wiki/1022910821149312/1023022126220448)](https://www.liaoxuefeng.com/wiki/1022910821149312/1023022126220448)