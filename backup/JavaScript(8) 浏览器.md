JavaScript的出现就是为了能在浏览器中运行。

流行的浏览器，除了ie10之前的，都支持ES6。

国内的换皮浏览器需要注意的是内核是什么。

JavaScript代码，我们撰写的原则就是，尽量一份代码，可以在多种浏览器中运行。

## 1 浏览器对象

要明白，浏览器也是程序，js是在浏览器这个容器中运行的。

所以浏览器提供了一些对象，js可以获取到，可以操作。

### window

window对象，既是全局作用域，又是浏览器窗口。

有innerWidth，innerHeight属性，表示浏览器窗口的内宽和内高，内部表示去除菜单栏，工具栏，边框等占位元素之后的空间。

对应的有outerWidth和outerHeight属性，浏览器窗口的外宽和外高。

```javascript
console.log('window inner size: ' + window.innerWidth + ' x ' + window.innerHeight);
```

调整浏览器大小看数值变动。

### navigator

表示浏览器信息。常用属性有

- navigator.appName：浏览器名称
- navigator.appVersion：浏览器版本
- navigator.language：浏览器设置的语言
- navigator.platform：操作系统类型
- navigator.userAgent：浏览器设定的user-agent字符串

```javascript
console.log('appName = ' + navigator.appName);
console.log('appVersion = ' + navigator.appVersion);
console.log('language = ' + navigator.language);
console.log('platform = ' + navigator.platform);
console.log('userAgent = ' + navigator.userAgent);
/*
appName = Netscape
appVersion = 5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36
language = zh-CN
platform = Win32
userAgent = Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36
*/
```

**注意**，不要通过navigator判断浏览器版本，因为比较好篡改，这个信息一般不可信。

> 但这样既可能判断不准确，也很难维护代码。正确的方法是充分利用JavaScript对不存在属性返回`undefined`的特性，直接用短路运算符`||`计算：
>
> ```javascript
> var width = window.innerWidth || document.body.clientWidth;
> ```

### screen

表示屏幕信息。

- screen.colorDepth：颜色位数
- screen.height：屏幕宽度，以像素为单位
- screen.width：屏幕高度，以像素为单位

```javascript
console.log('Screen size = ' + screen.width + ' x ' + screen.height);
console.log('Screen color = ' + screen.colorDepth);
console.log('window inner size: ' + window.innerWidth + ' x ' + window.innerHeight);
/*
Screen size = 1920 x 1080
Screen color = 24
window inner size: 2133 x 1034
*/
```

当电脑连接多块显示器时，同一个浏览器，挪动到不同显示器执行上述代码，可能有不同的结果。

### location

表示当前页面的URL信息，说白了就是获取当前URL的各个部分的能力

```javascript
console.log(location.href);
console.log(location.protocol);
console.log(location.host);
console.log(location.port);
console.log(location.pathname);
console.log(location.search);
console.log(location.hash);
/*
https://www.liaoxuefeng.com/wiki/1022910821149312/1023022272084160
https:
www.liaoxuefeng.com

/wiki/1022910821149312/1023022272084160



*/
```

可以看到没有在url中体现的，属性中也没有转换什么的，比如80端口。

另外

> 要加载一个新页面，可以调用`location.assign()`。如果要重新加载当前页面，调用`location.reload()`方法非常方便。

### document

document对象表示当前页面。

由于HTML在浏览器中以DOM形式表示为树形结构，所以document对象就是整个DOM树的根节点。

要查找DOM树的某个节点，需要从`document`对象开始查找。最常用的查找是根据ID和Tag Name。

用`document`对象提供的`getElementById()`和`getElementsByTagName()`可以按ID获得一个DOM节点和按Tag名称获得一组DOM节点：

```html
<dl id="drink-menu" style="border:solid 1px #ccc;padding:6px;">
    <dt>摩卡</dt>
    <dd>热摩卡咖啡</dd>
    <dt>酸奶</dt>
    <dd>北京老酸奶</dd>
    <dt>果汁</dt>
    <dd>鲜榨苹果汁</dd>
</dl>
```

```java
var menu = document.getElementById('drink-menu');
var drinks = document.getElementsByTagName('dt');
var i, s;

s = '提供的饮料有:';
for (i=0; i<drinks.length; i++) {
    s = s + drinks[i].innerHTML + ',';
}
console.log(s);
//提供的饮料有:摩卡,酸奶,果汁,
```

`document`对象还有一个`cookie`属性，可以获取当前页面的Cookie。

Cookie是由服务器发送的key-value标示符。因为HTTP协议是无状态的，但是服务器要区分到底是哪个用户发过来的请求，就可以用Cookie来区分。当一个用户成功登录后，服务器发送一个Cookie给浏览器，例如`user=ABC123XYZ(加密的字符串)...`，此后，浏览器访问该网站时，会在请求头附上这个Cookie，服务器根据Cookie即可区分出用户。

Cookie还可以存储网站的一些设置，例如，页面显示的语言等等。

JavaScript可以通过`document.cookie`读取到当前页面的Cookie；由于JavaScript能读取到页面的Cookie，而用户的登录信息通常也存在Cookie中，这就造成了巨大的安全隐患，这是因为在HTML页面中引入第三方的JavaScript代码是允许的：

```html
<!-- 当前页面在wwwexample.com -->
<html>
    <head>
        <script src="http://www.foo.com/jquery.js"></script>
    </head>
    ...
</html>
```

为了解决这个问题，服务器在设置Cookie时可以使用`httpOnly`，设定了`httpOnly`的Cookie将不能被JavaScript读取。这个行为由浏览器实现，主流浏览器均支持`httpOnly`选项，IE从IE6 SP1开始支持。

为了确保安全，服务器端在设置Cookie时，应该始终坚持使用`httpOnly`。

### history

`history`对象保存了浏览器的历史记录，JavaScript可以调用`history`对象的`back()`或`forward ()`，相当于用户点击了浏览器的“后退”或“前进”按钮。

这个对象属于历史遗留对象，对于现代Web页面来说，由于大量使用AJAX和页面交互，简单粗暴地调用`history.back()`可能会让用户感到非常愤怒。

**任何情况，你都不应该使用`history`这个对象了。**

## 2 操作DOM

**HTML，被浏览器解析之后，就是一颗DOM树，牢记这点。**

对DOM树的操作，就能改变HTML的结构。

对DOM树的操作，不外乎增删改查。

### 查

查，又或者说是获取某个DOM节点。

最常用的是document.getElementById()和document.getElementsByTagName()，还有CSS选择器document.getElementsByClassName()

由于ID在HTML中是唯一的，所以常用做法是，通过id查到唯一DOM节点，再精确查找他的子节点。

拿到父节点后，通常需要遍历获取所有子节点，可以通过迭代children属性来实现

```javascript
var
    i, c,
    list = document.getElementById('list');
for (i = 0; i < list.children.length; i++) {
    c = list.children[i]; // 拿到第i个子节点
}
```

举例

```html
<!-- HTML结构 -->
<div id="test-div">
<div class="c-red">
    <p id="test-p">JavaScript</p>
    <p>Java</p>
  </div>
  <div class="c-red c-green">
    <p>Python</p>
    <p>Ruby</p>
    <p>Swift</p>
  </div>
  <div class="c-green">
    <p>Scheme</p>
    <p>Haskell</p>
  </div>
</div>
```

```javascript
'use strict';
// 选择<p>JavaScript</p>:
var js = document.getElementById("test-p");

// 选择<p>Python</p>,<p>Ruby</p>,<p>Swift</p>:
var arr = document.getElementsByClassName("c-red c-green")[0].children;

// 选择<p>Haskell</p>:
var haskell = document.getElementsByClassName("c-green")[1].lastElementChild;

console.log(arr);
console.log(haskell);


// 测试:
if (!js || js.innerText !== 'JavaScript') {
    alert('选择JavaScript失败!');
} else if (!arr || arr.length !== 3 || !arr[0] || !arr[1] || !arr[2] || arr[0].innerText !== 'Python' || arr[1].innerText !== 'Ruby' || arr[2].innerText !== 'Swift') {
    console.log('选择Python,Ruby,Swift失败!');
} else if (!haskell || haskell.innerText !== 'Haskell') {
    console.log('选择Haskell失败!');
} else {
    console.log('测试通过!');
}
```

尤其注意各种get方法的使用。

getElementById，可以精确到具体的DOM节点；那DOM节点的常用方法都是可以直接用的，比如innerText；

但是getElementsByTagName，getElementsByClassName，拿到的都是一个DOM节点数组，所以要定位到具体的某个节点。

常用的方式方法，更多示例如下

```javascript
// 返回ID为'test'的节点：
var test = document.getElementById('test');

// 先定位ID为'test-table'的节点，再返回其内部所有tr节点：
var trs = document.getElementById('test-table').getElementsByTagName('tr');

// 先定位ID为'test-div'的节点，再返回其内部所有class包含red的节点：
var reds = document.getElementById('test-div').getElementsByClassName('red');

// 获取节点test下的所有直属子节点:
var cs = test.children;

// 获取节点test下第一个、最后一个子节点：
var first = test.firstElementChild;
var last = test.lastElementChild;
```

另外

> 严格地讲，我们这里的DOM节点是指`Element`，但是DOM节点实际上是`Node`，在HTML中，`Node`包括`Element`、`Comment`、`CDATA_SECTION`等很多种，以及根节点`Document`类型，但是，绝大多数时候我们只关心`Element`，也就是实际控制页面结构的`Node`，其他类型的`Node`忽略即可。根节点`Document`已经自动绑定为全局变量`document`。

### 增

这里首先涉及一个问题，如何创建一个节点。

```javascript
var ab = document.createElement("p");
ab.id = 'ab';
ab.innerText = 'AB';

var d = document.createElement('style');
d.setAttribute('type', 'text/css');
d.innerHTML = 'p { color: red }';
```

有了待新增的节点，理论上，有三种新增DOM节点的方法

1. 空白的DOM节点

   增加属性来充实这个节点，例如，`<div></div>`，那么，直接使用`innerHTML = '<span>child</span>'`就可以修改DOM节点的内容，相当于“插入”了新的DOM节点。

2. 在某个节点后面新增节点

   ref.appendChild(new);

3. 在某个节点之前新增节点

   parentElement.insertBefore(newElement, referenceElement);

   所以这里需要的信息更多，需要知道父节点，需要知道参考节点。

练习将有的HTML list重新排序

```html
<!-- HTML结构 -->
<ol id="test-list">
    <li class="lang">Scheme</li>
    <li class="lang">JavaScript</li>
    <li class="lang">Python</li>
    <li class="lang">Ruby</li>
    <li class="lang">Haskell</li>
</ol>
```

```javascript
// sort list:
var list = document.getElementById("test-list");
var i, j;
for (i = 0; i < list.children.length; i++) {
    if (i === 0) continue;
    for (j = i; j > 0; j--) {
        if (list.children[j - 1].innerText > list.children[j].innerText) {
            list.insertBefore(list.children[j], list.children[j - 1]);
        }
    }
}
// 测试:
; (function () {
    var
        arr, i,
        t = document.getElementById('test-list');
    if (t && t.children && t.children.length === 5) {
        arr = [];
        for (i = 0; i < t.children.length; i++) {
            arr.push(t.children[i].innerText);
        }
        if (arr.toString() === ['Haskell', 'JavaScript', 'Python', 'Ruby', 'Scheme'].toString()) {
            console.log('测试通过!');
        }
        else {
            console.log('测试失败: ' + arr.toString());
        }
    }
    else {
        console.log('测试失败!');
    }
})();
```

### 删

只需要知道父节点是哪个，然后parent.removeChild。

怎么拿到父节点呢？尤其是在不知道id什么的情况下。

```javascript
// 拿到待删除节点:
var self = document.getElementById('to-be-removed');
// 拿到父节点:
var parent = self.parentElement;
// 删除:
var removed = parent.removeChild(self);
removed === self; // true
```

self.parentElement;

**注意到删除后的节点虽然不在文档树中了，但其实它还在内存中，可以随时再次被添加到别的位置。**

另外就是注意remove会实时改变children属性的长度，遍历的时候尤其需要注意。

```html
<!-- HTML结构 -->
<ul id="test-list2">
    <li>JavaScript</li>
    <li>Swift</li>
    <li>HTML</li>
    <li>ANSI C</li>
    <li>CSS</li>
    <li>DirectX</li>
</ul>
```

把与Web开发技术不相关的节点删掉：

```javascript
var list = document.getElementById("test-list2");
var needRemove = [], i, j;
for (i = 0; i < list.children.length; i++) {
    if (list.children[i].innerText !== "JavaScript" && list.children[i].innerText !== "HTML" && list.children[i].innerText !== "CSS") {
        needRemove.push(i);
    }
}
for (j = needRemove.length - 1; j > -1; j--) {
    list.removeChild(list.children[needRemove[j]]);
}

// 测试:
;(function () {
    var
        arr, i,
        t = document.getElementById('test-list2');
    if (t && t.children && t.children.length === 3) {
        arr = [];
        for (i = 0; i < t.children.length; i ++) {
            arr.push(t.children[i].innerText);
        }
        if (arr.toString() === ['JavaScript', 'HTML', 'CSS'].toString()) {
            console.log('测试通过!');
        }
        else {
            console.log('测试失败: ' + arr.toString());
        }
    }
    else {
        console.log('测试失败!');
    }
})();
```

### 改

对于DOM节点，可以修改文本，可以修改属性，可以修改CSS样式

最强大的是innerHTML属性的修改，可以直接改变DOM节点甚至是子节点的情况。谨慎使用（比如XSS攻击）。

参考

- [[前端安全系列（一）：如何防止XSS攻击？](https://tech.meituan.com/2018/09/27/fe-security.html)](https://tech.meituan.com/2018/09/27/fe-security.html)
- [[innerHTML 的漏洞与安防](https://juejin.cn/post/6844903909715116046)](https://juejin.cn/post/6844903909715116046)

```javascript
// 获取<p id="p-id">...</p>
var p = document.getElementById('p-id');
// 设置文本为abc:
p.innerHTML = 'ABC'; // <p id="p-id">ABC</p>
// 设置HTML:
p.innerHTML = 'ABC <span style="color:red">RED</span> XYZ';
// <p>...</p>的内部结构已修改
```

通过innerText可以修改文本。

通过.style属性，可以修改颜色，字体等样式

```javascript
// 获取<p id="p-id">...</p>
var p = document.getElementById('p-id');
// 设置文本:
p.innerText = '<script>alert("Hi")</script>';
// HTML被自动编码，无法设置一个<script>节点:
// <p id="p-id">&lt;script&gt;alert("Hi")&lt;/script&gt;</p>

// 获取<p id="p-id">...</p>
var p = document.getElementById('p-id');
// 设置CSS:
p.style.color = '#ff0000';
p.style.fontSize = '20px';
p.style.paddingTop = '2em';
```

## 参考资料

1. [[浏览器](https://www.liaoxuefeng.com/wiki/1022910821149312/1023022129105888)](