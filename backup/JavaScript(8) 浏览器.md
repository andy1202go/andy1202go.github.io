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