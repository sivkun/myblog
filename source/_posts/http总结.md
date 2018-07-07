---
title: http总结
date: 2017-07-01 21:52:52
tags: [http]
categories: [协议]
---
# Get和Post方法区别
Http协议定义了很多与服务器交互的方法，最基本的有4种，分别是GET,POST,PUT,DELETE. 一个URL地址用于描述一个网络上的资源，而HTTP中的GET, POST, PUT, DELETE就对应着对这个资源的查，改，增，删4个操作。 我们最常见的就是GET和POST了。GET一般用于获取/查询资源信息，而POST一般用于更新资源信息.
1. GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连。如userInfo.php?id=17065&name=test.POST方法是把提交的数据放在HTTP包的Body中。
2. GET提交的数据大小有限制（因为浏览器URL的长度有限制），而POST方法提交的数据没有限制。
3. GET方式提交数据会有安全问题。

# Cache-Control  

> http://mp.weixin.qq.com/s/tluGR6Xc2tCjtaOLWO9q6Q
> http://www.cnblogs.com/wakey/p/4241516.html

Cache-Control即可以用于请求头，也可以用于响应头。响应头是重点掌握内容。
它控制着两个缓存：本地缓存（私有缓存）和共享缓存。
本地缓存，是指在客户端本地机器中的缓存。站在开发者的角度，它并不完全受你的控制，通常浏览器会自己决定是否把某些内容放到缓存中，这意味着：不要依赖于本地缓存。
共享缓存，处于客户端和服务器之间的缓存。即 CDN。你对共享缓存拥有绝对的控制，应该好好地利用它。
<!--more-->
Cache-Control有三种属性：缓冲能力、过期时间和二次验证
首先是缓冲能力，它关注的是缓存到什么地方，和是否应该被缓存。他的几个重要的属性是：
* private:表示它只应该存在本地缓存
* public:表示它既可以存在共享缓存，也可以被存在本地缓存
* no-cache:表示不论是本地缓存还是共享缓存，在使用它以前必须用缓存里的值重新验证。
* no-store:表示不允许被缓存
第二个是过期时间，很显然它关注的是内容可以被缓存多久。它的几个重要的属性是：
* `max-age = <seconds>`:设置缓存时间，单位为秒。本地和共享可以使用。
* `s-maxage = <seconds>`:覆盖`max-age`属性。只在共享缓存中起作用。
最后一个是二次验证，表示精细控制。它的几个重要属性是：
* immutable:表示文档是不能更改的。
* must-revalidate:表示客户端必须检查代理服务器上是否存在，即时已经本地缓存了也要检查。
* proxy-revalidata:表示共享缓存必须要检查源是否存在，即使已经有缓存。
```js
Cache-Control: public max-age=3600
Cache-Control: private immutable
Cache-Control: no-cache
Cache-Control: public max-age=3600 s-maxage=7200
Cache-Control: public max-age=3600 proxy-revalidate
```
上面的意思：
1. 本地缓存和 CDN 缓存均缓存 1 小时；
2. 不能缓存在 CDN，只能缓存在本地。并且一旦被缓存了，则不能被更新；
3. 不能缓存。如果一定要缓存的话，确保对其进行了二次验证；
4. 本地缓存 1 小时，CDN 上缓存 2 小时；
5. 本地和 CDN 均缓存 1 小时。但是如果 CDN 收到请求，则尽管已经缓存了 1 小时，还是要检查源中文档是否已经被改变。

# ETag  
ETag是实体标签（Entity Tag）的缩写， 根据实体内容生成的一段hash字符串（类似于MD5或者SHA1之后的结果），可以标识资源的状态。 当资源发送改变时，ETag也随之发生变化。
ETag是Web服务端产生的，然后发给浏览器客户端。浏览器客户端是不用关心Etag是如何产生的。
为什么使用ETag呢？ 主要是为了解决Last-Modified 无法解决的一些问题。
1. 某些服务器不能精确得到文件的最后修改时间， 这样就无法通过最后修改时间来判断文件是否更新了。
2. 某些文件的修改非常频繁，在秒以下的时间内进行修改. Last-Modified只能精确到秒。
3. 一些文件的最后修改时间改变了，但是内容并未改变。 我们不希望客户端认为这个文件修改了。

# 浏览器不使用缓存
CTRL+F5强制刷新浏览器，或者设置IE。  可以让浏览器不使用缓存。
1. 浏览器发送Http request, 给Web 服务器， header中带有Cache-Control: no-cache.   明确告诉Web服务器，客户端不使用缓存。 
2. Web服务器将把最新的文档发送给浏览器客户端.
Pragma: no-cache的作用和Cache-Control: no-cache一模一样。 都是不使用缓存。 
Pragma: no-cache 是HTTP 1.0中定义的， 所以为了兼容HTTP 1.0. 所以会同时使用Pragma: no-cache和Cache-Control: no-cache



# 状态码
> http://www.cnblogs.com/wakey/p/4241526.html  

204	No Content(没有内容)	Response中包含一些Header和一个状态行， 但不包括实体的主题内容（没有response body）
302	Found（已找到）	与状态码301类似。但这里的移除是临时的。 客户端会使用Location中给出的URL，重新发送新的HTTP request。
304	Not Modified（未修改）	客户的缓存资源是最新的， 要客户端使用缓存
401	Unauthorized（未授权）	需要客户端对自己认证
403	Forbidden（禁止）	请求被服务器拒绝了	
404	Not Found（未找到）	未找到资源
407	Proxy Authentication Required(要求进行代理认证)	与状态码401类似， 用于需要进行认证的代理服务器  

## 206 Partial Content(部分内容)  
206状态码代表服务器已经成功处理了部分GET请求（只有发送GET 方法的request, web服务器才可能返回206），
应用场景：
1. FlashGet, 迅雷或者HTTP下载工具都是使用206状态码来实现断点续传
2. 将以个大文档分解为多个下载段同时下载 比如，在线看视频  

## 400 Bad Request（坏请求)  
发送的Request中的数据有错误(比如：表单有错误，Cookie有错误)，  这个我们也经常见到。   

## 411 Length Required（要求长度指示）  
服务器要求在Request中包含Content-Length。
当浏览器使用Post方法，发送数据给Web服务器时， 必须要有Content-Length。这样Web服务器才知道你要发送多少数据，否则Web服务器会返回411状态码
实例： 发送一个Post方法的Request 给www.google.com.   Request中没有Content-Length

## 413 Request Entity Too Large（请求实体太大）  
作用：客户端发送的实体主体部分比服务器能够或者希望处理的要大。  一般情况下我们看不到这个状态码。 因为浏览器不会发送太大的数据给网站，但是机器人可能会。
实例: 用post方法发送一个大文件(100MB以上)给www.google.com

## 414 Request URI Too Long(请求URI太长)  
就是说Request URI太长， 一般浏览器本身对URI的长度就会有限制，所以不会发送URI很长的Request. 我们平常是根本看不到414错误的。 但是机器人可以发送很长URI。  

# Cookie    
可以大致把Cookie分为2类： 回话cookie和持久cookie
会话cookie: 是一种临时的cookie，它记录了用户访问站点时的设置和偏好，关闭浏览器，会话cookie就被删除了
持久cookie: 存储在硬盘上，（不管浏览器退出，或者电脑重启，持久cookie都存在）， 持久cookie有过期时间

