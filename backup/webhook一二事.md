告警配置通知过程中，使用了企微的群消息通知，获得webhookUrl之后，取其中的key或者整个链接进行了配置。想到，好像一直没有仔细了解过这个机制，为什么单独一个webhook URL就能直接实现web应用-->APP应用内群消息的推送呢？

## webhook的定义和来源

> Webhook 是一种事件驱动的轻量级通信，可通过 HTTP 在应用之间自动发送数据。Webhook 由特定事件触发，可自动实现 [应用编程接口（API）](https://www.redhat.com/zh-cn/topics/api/what-are-application-programming-interfaces)之间的通信，并可用于激活工作流，例如在 [ GitOps](https://www.redhat.com/zh-cn/topics/devops/what-is-gitops) 环境中。
>
> Webhook 可以将事件源连接到自动化解决方案，因此，它可以用来启动[事件驱动型自动化](https://www.redhat.com/zh-cn/topics/automation/what-is-event-driven-automation)以便在发生特定事件时执行各种 IT 操作。

### 几个要点

- web层面的东西
- 通信手段
- 事件驱动

### 最重要的理解

- Webhook 通常被称为*“反向 API”* 或*“推送 API”*

- 最开始是作为轮询的反面引入的，也就是说，为了解决客户端长期轮询服务端获取事件情况的各种弊端

  > 如果你正在构建一个与数据流合作的程序，你必须编写一个轮询系统。这意味着要处理通常难以调试的 crontab，或者编写和管理一个完整的守护进程。然后你必须担心缓存和解析。数据流似乎是为浏览器设计的，因为浏览器为你做了很多这些工作，但如果你在后台不断轮询数据流或 API，情况就不同了。那时，用数据流和 API 做任何简单的事情几乎都变得太费事了。

- 具体看：

  > Jeff Lindsay 在 2007 年发表的一篇名为[《Webhook 革新 Web 世界》（Web hooks to revolutionize the web）的博客文章推动了这一概念的普及。](https://progrium.github.io/blog/2007/05/03/web-hooks-to-revolutionize-the-web/)

### 更底层的理解

1. webhook的设计，目的是，成为某种基础设施
2. 能够媲美UNIX中简单命令+管道组合实现复杂功能的能力；web场景下，各种web资源是简单命令，管道是API+webhook
3. 不同点在于，UNIX中，是线性的命令组合；web场景是无状态的网状结构。

## 企微的webhook实现和安全风险探讨

看到仅使用固定URL进行webhook通信，并进行群内通知，产生两个疑问：

1. 如何实现的群内通知
2. 安全性怎么保证

简单AI调用和官方文档，大致了解了一下：

### 实现群内通知

1. 验证key有效性
2. key拦截校验
3. 解析消息体
4. key和企业、群的映射
5. 消息推送

应该是非常常规的流程了，没有什么花活吧，也许。

### 安全性分析

1. 主要风险

- 垃圾消息攻击：如果webhook地址泄露，攻击者可以利用该地址向群聊发送垃圾消息[1](https://cloud.tencent.com/document/product/1263/71731)[4]
- 信息泄露：可能泄露企业内部敏感信息
- 服务滥用：恶意用户可能利用该接口进行大规模消息推送

2. 企业微信官方的安全警告

   企业微信官方文档特别强调："特别特别要注意：一定要保护好消息推送的webhook地址，避免泄漏！不要分享到github、博客等可被公开查阅的地方，否则坏人就可以用你的消息推送来发垃圾消息了。"

所以就是，使用方（其实是作为服务端的用户们）自己负责，出现安全问题，也是用户受到影响；企微自己是不会有事的。

## 参考

- [什么是 Webhook？](https://www.redhat.com/zh-cn/topics/automation/what-is-a-webhook)
- [《Webhook 革新 Web 世界》（Web hooks to revolutionize the web）](https://progrium.github.io/blog/2007/05/03/web-hooks-to-revolutionize-the-web/)
- [消息推送配置说明](https://developer.work.weixin.qq.com/document/path/91770)



