多次受到CORS问题困扰，每次都是一知半解的解决了，不如这次来探寻下。

找到核心解释：MDN社区的解释。

然后最核心，是对CORS的理解：

> 跨源资源共享标准新增了一组 [[HTTP 标头](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Headers)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Headers)字段，允许服务器声明哪些源站通过浏览器有权限访问哪些资源。另外，规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 [`[GET](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/GET)`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/GET) 以外的 HTTP 请求，或者搭配某些 [[MIME 类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/MIME_types)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/MIME_types)的 [`[POST](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/POST)`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/POST) 请求），浏览器必须首先使用 [`[OPTIONS](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/OPTIONS)`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/OPTIONS) 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨源请求。服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（例如 [[Cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/Cookies)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/Cookies) 和 [[HTTP 认证](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/Authentication)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/Authentication)相关数据）。
>
> CORS 请求失败会产生错误，但是为了安全，在 JavaScript 代码层面*无法*获知到底具体是哪里出了问题。你只能查看浏览器的控制台以得知具体是哪里出现了错误。

## 关键点：

- 服务器声明的
- 基于HTTP标头的
- 应对跨域资源访问的
- 为了安全目的设立的

## 其他需要熟知的信息：

1. 简单请求

2. 非简单请求的预检请求，prefilght

3. 附带身份凭证的请求，也就是请求会发送的更多，响应声明也要更加严格

4. 一些Headers：

   > Origin
   >
   > Access-Control-Request-Method
   >
   > Access-Control-Request-Headers
   >
   >  
   >
   > Access-Control-Allow-Origin
   >
   > Vary
   >
   > Access-Control-Allow-Methods
   >
   > Access-Control-Allow-Headers
   >
   > Access-Control-Allow-Credentials
   >
   > Access-Control-Max-Age 预检请求缓存时长(s)
   >
   > Access-Control-Expose-Headers

5. 常见错误：[[跨源资源共享（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/CORS)

## 参考

[[跨源资源共享（CORS）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/CORS)

[[MIME 类型（IANA 媒体类型）](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/MIME_types)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Guides/MIME_types)

[[Content-Type](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Headers/Content-Type)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Headers/Content-Type)