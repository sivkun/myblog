---
title: http协议
date: 2017-03-22 15:09:41
tags: [http]
categories: [协议]
---

> 可以[参考]( https://developer.mozilla.org/zh-CN/docs/Web/HTTP)，以及《图解HTTP》,进行学习


绝对URI格式
`http://user:pass@www.example.com:80/dir/index.html/uid=1#ch1`
http:或https:等协议方案名获取访问资源是要指定协议类型。不区分字母大小写，最后附一个冒号(:)

`http://`:协议方案名
`user:pass`:登录信息（认证）（可选）
`www.example.com`：服务器地址
`80`:服务器端口号
`/dir/index.html`:带层次的文件路径
`uid`：查询字符串
`ch1`:片段标识符

`RFC`:Request for Comments 征求修正意见书
通常，应用程序会遵照RFC确定的标准实现。可以说，RFC是互联网的设计文档，要是不按照RFC标准执行，就有可能导致无法通信的状况。

请求报文的构成
```js
POST(方法)  /form/entry(URI)  HTTP/1.1(协议版本) 

//请求首部字段
Host: example.com
Connection:keep-alive
Content-Type:appllication/x-www-form-urlencoded
Content-Length:16
//请求首部字段结束

//内容实体
name=sfqq&age=25
```

响应报文：
```js
HTTP/1.1 200 OK(状态码的原因短语)

Date: Wed, 22 Mar 2017 07:42:52 GMT
Content-Length:455
Content-Type:text/html

//主体
<html>
.... 
```

HTTP是一种不保存状态的协议，HTTP协议自身不对请求和响应之间的通信状态进行保存。这是为了更快的处理大量事务，确保协议可伸缩性，而特意把HTTP协议设计成如此简单。但为了事项期望的保持状态的功能，于是引入了Cookie技术。有了Cookie再用HTTP协议通信，就可以管理状态了。
<!-- more -->

HTTP/1.0和HTTP/1.1支持的方法

| 方法      | 说明          | 支持的HTTP协议版本 |
| :------ | :---------- | :---------- |
| GET     | 获取资源        | 1.0、1.1     |
| POST    | 传输实体主体      | 1.0、1.1     |
| PUT     | 传输文件        | 1.0、1.1     |
| HEAD    | 获得报文首部      | 1.0、1.1     |
| DELETE  | 删除文件        | 1.0、1.1     |
| OPTIONS | 询问支持的方法     | 1.1         |
| TRACE   | 追踪路径        | 1.1         |
| CONNECT | 要求用隧道协议连接代理 | 1.1         |
| LINK    | 简历和资源之间的联系  | 1.0         |
| UNLINK  | 断开连接关系      | 1.0         |


`持久链接keep-alive`
http协议的初始版本中，没进行一次HTTP通信就要断开一次客户端和服务器的TCP链接。
当页面复杂时，一个html页面中含有其他资源，因此每次的请求都会造成TCP连接的建立和断开，增加通信量的开销。
为了解决这个问题。http/1.1和一部分http/1.0想出了持久链接，特点是，只要任意一段没有明确提出断开连接，则保持TCP连接状态。
在HTTP/1.1中，所有的连接默认都是持久链接，但在HTTP/1.0内并未标准化。毫无疑问，除了服务器端，客户端也需要支持持久链接。

`管线化pipelining`
持久链接是的多数请求以管线化方式发送成为可能。从前发送请求后需要等待并收到响应，才能发送下一个请求。管线化技术出现后，不用等待响应亦可直接发送下一个请求。这样就能够做到同时并行发送多个请求，而不需要一个接一个地等待响应了。


# http报文内的http信息

用于HTTP协议交互的信息被称为HTTP报文。

## 请求报文及响应报文的结构

1. 请求报文
* 报文首部
    具体：请求行、请求首部字段、通用首部字段、实体首部字段、其它
* 空行（CR+LF）
* 报文主体
2. 响应报文
* 报文首部
    具体：状态行、响应首部字段、通用首部字段、实体首部字段、其它
* 空行（CR+LF）
* 报文主体

请求行：包含用于请求的方法，请求URI和HTTP版本
状态行：包含表明响应结果的状态码，原因短语和HTTP版本。
首部字段：包含便是请求和响应的各种条件和属性的各类首部。
其它：可能包含HTTP的RFC里未定义的首部（Cookie等）

## 编码提升传输速率

http在传输数据时可以按照数据原貌直接传输，但也可以在传输过程中通过编码提升传输速率。编码的操作需要计算机来完成，因此会消耗更多的CPU等资源。

### 报文主体和实体主体的差异
* 报文（message）
   是HTTP通信的基本单位，由8位组字节流组成，通过HTTP通信传输
* 实体（entity）
   作为请求或响应的有限载荷数据被传输，其内容由实体首部和实体主体组成。

 HTTP报文的主体用于传输请求或响应的实体主体。
 同城报文主体等于实体主体。只有当（传输中进行编码）操作时，（实体主体的内容发生变化），才导致他和报文主体产生差异。

内容编码致命应用在实体内容上的编码格式，并保持实体信息原样压缩。内容编码后的实体由客户端接受并负责解码。
常用内容编码格式：gzip（GNU zip），compress（unix系统的标准压缩），deflate（zlib），identity（不进行编码）
### 分割发送的分块传输编码
在HTTP通信过程中，请求的编码实体资源尚未全部传输完成之前，浏览器无法显示请求的页面。在传输大容量数据时，通过把数据分割成多块，能够让浏览器逐步显示页面。
这种把实体分块的功能成为分块传输编码（Chunked transfer coding）

## 发送多种数据的多部分对象集合
    发送邮件时，我们可以在邮件里写入文字并添加多份附件。这是因为采用了MIME（Multipurpose Internet Mail Extensions，多用途因特
网邮件扩展）机制。它允许邮件处理文本、图片、视频、等多各不同类型的数据。
相应地，HTTP协议中也采纳了多部分对象集合（Multipart），发送的一份报文内可含有多类型实体。通常是在图片或文本文件上传时使用。
* multipart/form-data 在web表单文件上传时使用
* multipart/byteranges
  状态码206（partial content，部分内容）响应报文包含了多个范围的内容时使用。

一份报文中含有多类型实体，在报文实体中，每个类型实体，要使用某种标记分割标识哪种类型的实体。 在报文首部使用boundary来指定字符串来划分多部分集合指明各类实体。多部分对象集合的每个部分类型中，都可以含有首部字段。

## 获取部分内容的范围请求
    断点续传。
    对一份10000字节大小的资源，如果适用范围请求，可以只请求5001~10000字节的资源。
```js
    //请求
    GET /tip.jpg HTTP/1.1
    Host:www.example.com
    Range:bytes = 5001-10000
    //响应
    HTTP/1.1 206 Partial Content
    Date:Wed, 22 Mar 2017 07:42:52 GMT
    Content-Range:bytes 5001-10000/10000
    Content-Length:5000
    Content-Type:image/jpeg
```
* 从5001字节后的全部
  `Range:bytes=5001-`
* 从一开始到3000字节和5000-7000字节的多重范围
  `Range:bytes=-3000,5000-7000`
  如果服务器端无法响应范围请求，则会返回状态码200 OK和完整的实体内容。

## 内容协商返回最合适的内容
    同一个web网站有可能存在多份相同内容的页面。比如英文版和中文版的web页面，他们内容虽然相同，但使用的语言却不同。
当浏览器的默认语言为英文或中文，访问相同的URI的web页面时，则会显示对应的英文版或中文版的web页面。这种机制为内容协商。
内容协商机制是指客户端和服务端就响应的资源内容进行交涉，然后提供客户端最为合适的资源。内容协商会以响应资源的语言、字符集、编码方式等作为判断基准。
请求头：Accept、Accept-Charset、Accept-Encoding、Accept-Language、Content-Language
内容协商技术三种类型：
* 服务器驱动协商
    由服务器端进行内容协商。以请求的首部字段为参考，在服务器端处理
* 客户端驱动协商
    由客户端进行内容协商，用户从浏览器显示的可选列表中手动选择。
* 透明协商
    是服务器驱动和客户端驱动的结合体，是由服务器端和客户端各自进行内容协商的一种方法。

# 返回结果的HTTP状态码
    状态码的职责就是当客户端向服务器端发送请求时，描述返回的请求结果。借助状态码，用户可以知道服务器端是正常处理了请求，还是出现了
错误。
状态码的类别

|      | 类别                     | 原因短语          |
| :--- | :--------------------- | :------------ |
| 1xx  | Informational(信息性状态码)  | 接受的请求正在处理     |
| 2XX  | Success(成功状态码)         | 请求处理完毕        |
| 3XX  | Redirection（重定向状态码）    | 需要进行附加操作以完成请求 |
| 4XX  | ClientError（客户端错误状态码）  | 服务器无法处理请求     |
| 5XX  | ServerError（服务器端错误状态码） | 服务器处理请求出错     |
200 请求成功
204 No Content 请求处理成功，但没有资源可返回。
206 Partial Content 对资源某一部分请求。
301 Moved Permanently 永久性重定向
302 Found 临时性重定向
303 see other
    该状态码便是由于请求对应的资源存在着另一个URI，应使用GET方法重定向获取请求的资源，303状态码和302Found状态码有着相同的功能，
但303状态码明确便是客户端应当采用GET方法获取资源，这点与302状态码有区别。
    当301/302/303响应状态码返回时，几乎所有的浏览器都会把POST改为GET，并删除请求报文内的主体，之后请求会自动再次发送。
301/302标准是禁止将POST方法改变为GET方法的，但实际上使用时大家都这么做了。
304 Not Modified
    该状态码表示客户端发送附带条件的请求（指请求报文还包含If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since中任一首部）时，服务器端允许请求访问资源，但未满足条件的情况。304状态码返回时，布包好任何响应的
主体
部分。304虽然别划分在3XX类别中，但是和重定向没有关系。
307 Temporary Redirect 临时重定向
 该状态码与302 Found有着相同的含义。307会遵照浏览器标准，不会从POST变为GET。但是，对于处理响应时的行为，每种浏览器有可能
 出现不同的情况。
400 Bad Request
该状态码便是报文中存在语法错误。浏览器回想200OK一样对待该状态码。
401 Unauthorized
403 Forbidden
404 Not Found
500 Internal Server Error
503 Service Unavailable
# 与HTTP协作的Web服务器
    一台Web服务器可搭建多个独立域名的Web站点，也可作为通信路径上的中转服务器提升传输效率。
## 通信数据转发程序：代理、网管、隧道
* 代理：
    代理是一种有转发功能的应用程序，它扮演了位于服务器和客户端‘中间人’的角色，接收由客户端发送的请求并转发给服务器，同时接收服务器
    返回的响应并转发给客户端。

* 网关
    网关是转发其他服务器通信数据的服务器，接收从客户端发送来的请求时，它就像自己拥有资源的服务器一样对请求进行处理。有时客户端可能都不会察觉，自己通信目标是一个网关。
* 隧道
    隧道是在相隔甚远的客户端和服务器两者之间进行中转，并支持双方通信连接的应用程序。

### 代理
    代理服务器的基本行为就是接收客户端发送的请求后转发给其它服务器。
使用代理服务器的理由有：利用缓存技术减少网络带宽的流量，阻止内部针对特定网站的访问控制，以获取访问日志为主要目的，等等。
代理有多重使用方法，按两种基准分类。一种是是否使用缓存，另一种是是否会修改报文。
* 缓存代理
  代理转发响应时，缓存代理会预先将资源的副本保存在代理服务器上。当代理再次接收到相同资源的请求时，就可以不从原服务器那里获取资源。而是将之前缓存的资源作为响应返回。
* 透明代理
  转发请求或响应时，不对报文做任何加工的代理类型被称为透明代理。反之称为非透明代理。
### 网关
利用网关可以由HTTP请求转化为其它协议通信。

## 保存资源的缓存
    缓存是指代理服务器或客户端本地磁盘内保存的资源副本。利用缓存可减少对资源服务器的访问，因此也就节省了通信流量和通信时间。
缓存服务器是代理服务器的一种。
缓存服务器的优势在于利用缓存可避免多次从源服务器转发资源。因此客户端可就近从缓存服务器上获取资源，而源服务器也不必多次处理相同
的请求了。
### 缓存的有效期限
    即便缓存服务器内有缓存，也不能保证每次都会返回对资源的请求。因为这关系到被缓存资源的有效性问题。
即时存在缓存，也会因为客户端的要求、缓存的有效期等因素，向原服务器确认资源的有效性。若判断缓存失效，缓存服务器将会再次从源服务
器上获取‘新’资源。
### 客户端的缓存
    缓存不紧可以存在于缓存服务器内，还可以存在客户端浏览器中。以IE为例，客户端缓存称为临时网络文件。
浏览器缓存如果有效，就不必再向服务器请求相同的资源了，可以直接从本地磁盘内读取。当判定缓存过期后，
会向源服务器确认资源的有效性。若判断浏览器缓存失效，浏览器会再次请求资源。
# HTTP首部
 在报文众多的字段中，HTTP首部字段包含的信息最为丰富。首部字段同时存在于请求和响应报文内，并涵盖HTTP报文相关的内容信息。
## HTTP首部字段
HTTP首部字段传递重要的信息
### HTTP首部字段结构
`首部字段名：字段值`
* 若HTTP首部字段重复了会如何？
    这种情况在规范内尚未明确，根据浏览器内部处理逻辑的不同，结果可能并不一致。有的优先处理第一次出现的首部字段，有的有限处理最后出现的首部字段等。
### 4中HTTP首部字段类型
1. 通用首部字段（general header fields）
   请求报文和响应报文两方都会使用的首部。
2. 请求首部字段（request header fields）
3. 响应首部字段（response headerfields）
4. 实体首部字段（entity header fields）

HTTP/1.1和非HTTP/1.1首部
非HTTP/1.1首部字段：Cookie、Set-Cookie和Content-Disposition等

### End-to-End 首部和 Hop-by-hop首部
* 端到端首部
    该类首部会转发给请求/响应对应的最终接收目标，切必须保存在由缓存生成的响应中，另外规定它必须被转发
* 逐跳首部
    该类首部只对单次转发有效，会因通过缓存或代理而不再转发。在HTTP/1.1和之后版本中，如果使用hop-to-hop首部，需提供Connection首
    部字段。有如下逐跳首部字段：
1. Connection
2. Keep-Alive
3. Proxy-Authenticate
4. Proxy-Authorization
5. Trailer
6. TE
7. Transfer-Encoding
8. Upgrade
## HTTP/1.1通用首部字段
### Cache-Control
    通过指定该字段，可以操作缓存的工作机制。
指令格式，例如：
`Cache-Control:private,max-age=0,no-cache`
缓存请求指定：

| 指令              | 参数   | 说明              |
| :-------------- | :--- | :-------------- |
| no-cache        | 无    | 强制向源服务器再次验证     |
| no-store        | 无    | 不缓存请求或响应的任何内容   |
| max-age=[秒]     | 必须   | 响应的最大Age值       |
| max-stale(=[秒]) | 可省略  | 接受已经过期的响应       |
| min-fresh=[秒]   | 必须   | 期望在指定时间内的响应仍然有效 |
| no-transform    | 无    | 代理不可改变媒体类型      |
| only-if-cached  | 无    | 从缓存获取资源         |
| cache-extension | -    | 新指令标记（token）    |

缓存响应指令

| 指令               | 参数   | 说明                      |
| :--------------- | :--- | :---------------------- |
| public           | 无    | 可向任一方提供响应的缓存            |
| private          | 可省略  | 仅向特定用户返回响应              |
| no-cache         | 可省略  | 缓存前必须先确认其有效性            |
| no-store         | 无    | 不缓存请求或响应的任何内容           |
| no-transform     | 无    | 代理不可更改媒体类型              |
| must-revalidate  | 无    | 可缓存必须向源服务器进行确认          |
| proxy-revalidate | 无    | 要求中间缓存服务器对缓存的响应有效性再进行确认 |
| max-age=[秒]      | 必须   | 响应的最大Age值               |
| s-maxage=[秒]     | 必须   | 公共缓存服务器响应的最大Age值        |
| cache-extension  | -    | 新指令标记（token）            |


**表示是否能缓存的指令**
public指令
`Cache-Control:public`,表示其它用户可以利用缓存
private指令
`Cache-Control:private`,表示缓存服务器会对特定用户提供资源缓存的服务，对于其它用户发来的请求，代理服务器则不会返回缓存。
no-cache指令
`Cache-Control:no-cache`，表示不缓存过期的资源，缓存会向源服务器进行有效性确认后处理资源。
**控制可执行缓存的对象的指令**
no-store指令
`Cache-Control:no-store`,暗示请求或响应中包含机密信息，不进行缓存。
**指定缓存期限和认证的指令**
s-maxage指令
`Cache-Control:s-maxage=604800`（单位：秒）一周
    该指令的功能和max-age指令相同，它们的不同点是s-maxage指令只适用于供多为用户使用的公共缓存服务器。也就是说对于向同一用户重复
返回响应服务器来说，这个指令没有任何作用。当使用s-maxage指令后，则直接忽略对Expires首部字段及max-age指令的处理。
max-age指令
`Cache-Control:max-age=604800`
    客户端发送的请求中包含max-age指令时，如果判定缓存资源的缓存时间数值比指定时间的数值更小，那么客户端就接受缓存的资源。另外，当
max-age=0，那么缓存服务器通常需要将请求转发给源服务器。
    服务器返回的响应中包含max-age时，缓存服务器将不对资源的有效性再做确认，而max-age数值代表资源保存为缓存的最长时间。
    应用HTTP/1.1版本的缓存服务器遇到同时存在Expires首部字段的情况时，会优先处理max-age指令，而忽略掉Expires首部字段。
而HTTP/1.0版本的缓存服务器的情况却相反，max-age指令会被忽略。
min-fresh指令
`Cache-Control:min-fresh=60`
    客户端询问缓存服务器，再过60秒这个资源还是新的吗（过期了吗）？不是的话，则不能用缓存做响应。
max-stale指令
`Cache-Control:max-stale=3600`
使用max-stale可只是缓存资源，即时过期也照常接受。

### Connection
    两个作用：
* 控制不再转发给代理的首部字段
```js
//客户端发送请求
    GET / HTTP/1.1
    Upgrade: HTTP/1.1
    Connection:Upgrade
//经过一个代理转发后
    GET / HTTP/1.1
//首部字段Upgrade被删除后再转发
```
* 管理持久链接
  `Connection:close`
  HTTP/1.1默认链接都是持久化链接。为此，客户端会在持久化链接上连续发送请求。当服务器端向明确断开连接时，则指定Connection首部字段为Close。
  `Connection:Keep-Alive`
  HTTP/1.1之前的HTTP版本的默认链接都是非持久链接。为此想在旧版本的HTTP协议上维持持续连接，则需要指定Connection首部字段值为Keep-Alive.
### Date
首部字段Date标明创建HTTP报文的日期和时间。
### Trailer
    首部字段Trailer会事先说明咋报文主体后记录了哪些首部字段。该首部子弹壳应用在HTTP/1.1版本分块传输编码时。
### Transfer-Encoding
    首部字段Transfer-Encoding规定了传输报文主体时采用的编码方式。
    HTTP/1.1的传输编码方式仅对分块传输编码有效
### Via
    使用首部字段via是为了追踪客户端与服务器端之间的请求和响应报文的传输路径。
    是偶不字段Via不紧用于追踪报文的转发，还可以避免请求回环的发生。所以必须在经过代理是附加该首部字段内容。
## 请求首部字段
    请求首部子弹是从客户端往服务器端发送请求报文所使用的字段，用于补充请求的附加信息，客户端信息，对响应内容相关的优先级等内容。
### Accept
`Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8`
Accept首部字段可通知服务器，用户代理能够处理的媒体类型及媒体类型的优先级。可以使用type/subtype这种形式，一次指定多种媒体类型。
例子：
* 文本文件
  text/html,text/plain,text/css...
  application/xhtml+xml,application/xml...
* 图片文件
  image/jpeg,image/gif,image/png...
* 视频文件
  video/mpeg,video/quicktime...
* 应用程序使用的二进制文件
  application/octet-stream,application/zip...

若想要给显示的媒体类型增加优先级，则使用q=来二外表示权重值，用分号（;）进行分割。

### Accept-Charset
`Accept-Charset:iso-8859-5,unicode-1-1;q=0.8`
该字段可用来通知服务器用户代理支持的字符集及字符集的相对有限顺序。另外可一次性指定多种字符集。与首部字段Accept相同的是可用权重
q值来表示相对优先级。
    该首部字段应用于内容协商机制的服务器驱动协商。
### Accept-Encoding
`Accept-Encoding:gzip,deflate`
Accept-Encoding首部告知服务器用户代理支持的内容编码及内容编码的优先级顺序，可一次性指定多种内容编码。
### Accept-Language
`Accept-Language:zh-cn,zh;q=0.7,en-us,en;q=0.3`
客户端在服务器有中文版资源的情况下，会请求其返回中文版对应的响应，没有中文版，则请求返回英文版响应。
### Authorization
`Authorization:Basic ddsfkdasfldasfkasdf=`
    首部字段Authorization是用来告诉服务器，用户代理的认证信息。

### Expect
`Expect:100-continue`
客户端使用首部字段Expect来告知服务器，期望出现的某种特定行为。因服务器无法理解客户端的期望做出回应而发生错误时，会返回状态码417 Expectation Failed
### From
    首部字段From用来告知服务器使用用户代理的用户的电子邮件地址。
### Host
`Host: www.example.com`
首部字段Host会会告知服务器，请求的资源所处的互联网主机名和端口号。Host首部字段在HTTP/1.1规范内是唯一一个必须被包含在请求内的首部字段。首部字段Host和以单台服务器分配多个域名的虚拟主机的工作主机有着很密切的关联，这是首部字段Host必须存在的意义。虚拟主机运行在同一个IP上，因此使用首部字段Host加以区分。
### If-Match
形如If-XXX这种样式的请求首部字段，都可称为条件请求。服务器接收到附带条件的请求后，只有判断指定条件为真时，才会执行请求。
`If-Match:'123456'`
发送请求含有If-Match，服务器端接收到请求后，If-Match的值和实体标记比对，如果值相等返回200OK，如果不相等返回412（Precondition Failed）实体标记（ETag）是与特定资源关联的确定值。资源更新后ETag也会随之更新。
还可以使用星号（*）指定If-Match的字段值。针对这种情况，服务器将会忽略ETag的值，只要资源存在就会处理请求。
### If-Modified-Since

`If-Modified-Since:Fri, 24 Mar 2017 07:22:58 GMT`
如果在If-Modified-Since字段指定的日期时间后，资源发生了更新，服务器则会接受请求
对请求做出相应返回200OK，否则表示资源未更新，返回304 Not Modified。
获取资源的更新日期时间，可通过确认首部字段Last-Modified来确定。

![If-Modified-Since](http协议/If-Modified-Since.png)


### If_None_Match
​	只有在If-None-Match的字段值与Etag值不一致时，可处理该请求。与If-Match首部字段的作用相反。
### If-Range
​	If-Range字段值若是跟ETag值或更新的日期时间匹配一致，那么久作为范围请求处理。

```
GET /index.html
If-Range:"123456"
Range:bytes=5001-10000
//响应，如果服务器端index.html实体标记（ETag）为123456
206 Partial Content
Content-Range:bytes 5001-10000/10000
Content-Length:5000
//响应。若不一致，则忽略范围请求，返回全部资源
200 OK
ETag:"567890"
//如果不使用首部字段If-Range则需要进行两次处理
GET /
If-Match:"123456"
Range:5001-10000
//响应
412 Precondition Failed
GET /
//响应
200 OK
ETag:"567890"
```

###  If-Unmodified-Since

`If-Unmodified-Since:Thu,03 Jul 2017 00:00:00 GMT`

​	首部字段If-UnModified-Since和If-Modified-Since作用相反。它的作用是告知服务器，指定的资源请求只有在字段值内指定的日期时间之后，未发生更新的情况下，才能处理请求。如果在指定日期时间之后发生了更新，则以状态码412 Precondition Failed作为响应返回。

### Max-Forwards

​	通过TRACE方法或OPTIONS方法，发送包含首部字段Max-Forwards请求时，该字段以十进制整数形式指定可经过的服务器最大数目。服务器再往下一个服务器转发请求之前，Max-Forwards的值减1后重新复制。当服务器接收到Max-Forwards值为0的请求时，则不再转发，而是直接返回响应。

​	使用HTTP协议通信时，请求可能经过代理等多台服务器。途中，如果代理服务器有与某些原因导致请求转发失败，客户端也就等不到服务器返回的响应了。对此，我们无从可知。（代理可能产生循环）

​	可以灵活使用Max-Forwards，针对以上问题产生的原因展开调查。由于Max-Forwards字段值为0时，服务器就会立即返回响应，因此我们至少可以对以那台服务器为终点的传输路径的通信状况有所把握。

### Proxy-Authorization

`Proxy-Authorization:Basic dGlOdkdlJdfdd8`

​	接受到从代理服务器发来的认证质询时，客户端会发送包含首部字段Proxy-Authorization的请求，以告知服务器认证所需的信息。这个行为和客户端与服务器之间的HTTP访问相类似的，客户端与服务器之间的认证，使用首部字段Authorization可起到相同作用。

### Range

`Range:bytes=5001-10000`

​	对于只需获取部分资源的范围的请求，包含首部子弹Range即可告知服务器资源的指定范围。

### Referer

```
GET / 
Referer:http://www.example.com/index.php
//只要查看Referer就知道请求的URI是从哪个Web页面发起的。
```

客户端一般会发送Referer首部字段给服务器，但当直接在浏览器的气质蓝输入URI，或处于安全性的考虑时，也可以不发送该首部字段。因为原始资源URI的查询字符串可能包含有ID和密码等信息，如果发送，可能导致保密信息泄露。

Referer的正确拼写应该是Referrer，不知为何大家一直沿用这个错误的拼写。

### TE

`TE:gzip,deflate;q=0.5`

该字段告知服务器客户端能够处理响应的传输编码方式以及相对优先级。和Accept-Encoding的功能很像，但是用于传输编码。

### User-Agent

首部字段User-Agent会创建请求的浏览器和用户代理名称信息传达给服务器。

##  响应首部字段

​	响应首部字段是由服务器端向客户端返回响应报文中所使用的的字段，用于补充响应的附加信息，服务器信息，以及对客户端的附加要求信息等

### Accept-Ranges

​	该字段是用来告知服务器是否能处理范围请求，以指定获取服务器端某个部分的资源。可指定的字段值有两种，可处理范围请求时指定其为bytes，反之则指定其为none。

### Age

`Age:600`：如果是缓存服务器的响应，表示：这个缓存向源服务器确认过，现在已经过去了10分钟。

​	首部字段Age能告知客户端，源服务器在多久前创建了响应。字段值得单位为妙。若创建该响应的服务器是缓存服务器，Age值是指缓存后的响应再次发起认证到认证完成的时间值。代理创建响应时必须加上首部字段Age。

### ETag

​	ETag实体标识。可将资源以字符串形式做唯一性标识的方式。服务器会为每份资源分配对应的ETag值。当资源更新时，ETag值也需要更新。生成ETag时，并没有统一的算法规则，而仅仅是由服务器来分配。

​	资源被缓存时，就会被分配唯一性标识。仅凭URI指定缓存的资源是相当困难的。

**强ETag值和弱ETag值**

**强ETag**

强Etag值，不论实体发生多么细微的变化都会改变其值

`ETag:"usagi-1234"`

**弱ETag值**

弱ETag值只用于提示资源是否相同。只有资源发生了根本改变，产生差异时才会改变ETag值。这时，会在字段值最开始处附加W/

`ETag:W/"usagi-1234"`

### Location

​	使用首部字段Location可以将响应接收方引导至某个请求URI位置不同的资源。基本上，该字段会配合3XX：Redirection的响应提供重定向的URI。几乎所有的浏览器在接收到包含首部字段Location的响应后，都会强制性地尝试对已提示的重定向资源的访问。

### Proxy-Authenticate

`Proxy-Authenticate: Basic realm="usagidesign Auth" `

​	首部字段Proxy-Authenticate会把代理服务器所要求的认证信息发送给客户端。它与客户端和服务器之间（WWW-Authorization）的HTTP认证访问的行为相似，不同之处在于其认证行为是在客户端与代理之间进行的。

### Retry-After

`Retry-After:120`

​	首部字段Retry-After告知客户端应该在多久之后再次发送请求。主要配合状态码503 Service Unavailable，或3XXRedirect响应一起使用。字段值可以指定为具体的日期时间，也可以使创建响应后的数秒。

### Server

`Server:Apache/2.2.17(Unix)`

​	首部字段Server告知客户端当前服务器上安装的HTTP服务器应用程序的信息。不会单单会标出服务器上的软件应用名称，还有可能包括版本号和安装时启用的可选项。

`Server:Apache/2.2.6(Unix)PHP/5.2.5` 

### Vary

```
/**
*GET /Sample.html      |    |GET /Sample.html     |     |			  |
*Accept-Language:en-us |--->|Accept-Language:en-us|---->|------------> | 源服务器
*客户端			   |	| 代理服务器			  |     |<------------|
*													  Vary：Accept-Language
* Vary：Accept-Language，告知代理服务器，只能对具有相同语言的请求返回Sample.html缓存，反之需要重新从源服
* 务器获取资源。
**/
```

### WWW-Authenticate

`WWW-Authenticate:Basic realm="Usagidesign Auth"`用于HTTP认证

## 实体首部字段

实体首部字段是包含在请求报文和响应报文中的实体部分使用的首部，用于补充内容的更新时间等于实体相关的信息。

### Allow

`Allow:GET,HEAD`

该字段用于通知客户端服务器能够支持Request-URI指定资源的所有HTTP方法。当服务器收到不支持的HTTP方法时，会以状态码405Method Not Allowed作为响应返回。与此同时，还会把所有能支持的HTTP方法写入首部字段Allow后返回。

### Content-Encoding

`Content-Encoding:gzip`

