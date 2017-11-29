### 主要内容 ###
- HTTP是什么
- URL详解
- HTTP的请求篇
- HTTP的相应篇

## HTTP是什么
### 1. 概述

> HTTP 是一个基于请求/响应模式的、无状态的协议
    
### 2. 特点

- 支持客户端/服务端模式
- 灵活：HTTP支持传输任意类型的数据对象，可以用过`Content-Type`来标示传输类型。
- 无连接:每次链接只接收一个请求。服务端处理完客户端的请求并且收到客户端的应答后，会断开链接。
- 无状态：无状态是指对事物的处理没有记忆状态，简单来说，就是后续如果需要前面的信息，则必须重传

## URL详解
### 1. URL的组成

> 通用的格式：schema://host[:port#]/path/…/[?query-string][#anchor]

| 名称 | 功能 |
|---   | ---  |
|schema|访问服务器以获取资源时要使用哪种协议，比如，http，https 和 FTP 等|
|host  |HTTP 服务器的 IP 地址或域名|
|port# |HTTP 服务器的默认端口是 80，这种情况下端口号可以省略，如果使用了别的端口，必须指明，例如www.cnblogs.com：8080|
|path  |访问资源的路径|
|query-string |发给 http 服务器的数据|
|anchor|锚    |

## HTTP的请求
### 1. 请求头
- 请求方法
> HTTP/1.1 协议中共定义了八种方法（也叫“动作”）来以不同的方式操作指定的资源

| 方法名 | 功能 | 
|---     |---   |
|GET     |向指定的资源发出“显示”请求，使用 GET 方法应该只用在读取数据上，而不应该用于产生“副作用”的操作中    |
|POST    |指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求文本中。这个请求可能会创建新的资源或者修改现有资源，或两者皆有。      |
|PUT     |向指定资源位置上传其最新内容      |
|DELET   |请求服务器删除 Request-URI 所标识的资源      |
|OPTIONS |使服务器传回该资源所支持的所有HTTP请求方法。用`*`来代替资源名称，向 Web 服务器发送 OPTIONS 请求，可以测试服务器功能是否正常运作      |
|HEAD    |与 GET 方法一样，都是向服务器发出指定资源的请求，只不过服务器将不传回资源的本文部分，它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中关于该资源的信息（原信息或称元数据）      |
|TRACE   |显示服务器收到的请求，主要用于测试或诊断      |
|CONNECT |HTTP/1.1 中预留给能够将连接改为通道方式的代理服务器。通常用于 SSL 加密服务器的链接（经由非加密的 HTTP 代理服务器）      |
### 2. 请求头
- 请求和响应常见通用的 Header

| 名称 | 作用 | 
|---   |---|
| Content-Type | 请求体/响应体的类型，如：text/plain、application/json |
|Accept|说明接收的类型，可以多个值，用`,`(英文逗号)分开|
|Content-length|请求体/响应体的长度，单位字节|
|Content-Encoding|请求体/响应体的编码格式，如 `gzip`、`deflate`|
|Accept-Encoding|告知对方我方接受的 `Content-Encoding`|
| ETag | 给当前资源的标识，和`Last-Modified`、`If-None-Match`、`If-Modified-Since`配合，用于缓存控制|
|Cache-Control|取值一般为`no-cache`、`max-age=xx`，`xx`为整数，表示资源缓存有效期（秒）|

- 常见的请求 Header

| 名称 | 作用 | 
|---   |---|
|Authorization|用于设置身份认证信息|
|User-Agent|用户标识，如：OS 和浏览器的类型和版本|
|If-Modified-Since|值为上一次服务器返回的Last-Modified值，用于确定某个资源是否被更改过，没有更改过就从缓存中读取|
|If-None-Match|值为上一次服务器返回的 ETag 值，一般会和If-Modified-Since|
|Cookie|已有的Cookie|
|Referer|标识请求引用自哪个地址，比如你从页面 A 跳转到页面 B 时，值为页面 A 的地址|
|Host|请求的主机和端口号|

## HTTP的响应
### 1. 状态码
| 状态码 | 对应的信息 | 
|---   |---|
|1XX|提示信息—表示请求已接收，继续处理|
|2XX|用于表示请求已被成功接收、理解、接收|
|3XX|用于表示资源（网页等）被永久转移到其它 URL，也就是所谓的重定向|
|4XX|客户端错误—请求有语法错误或者请求无法实现|
|5XX|服务器端错误—服务器未能实现合法的请求|

### 2. 常见的响应 Header

| 名称 | 作用 | 
|---   |---|
|Date|服务器的日期|
|Last-Modified|该资源最后被修改的时间|
|Transfer-Encoding|取值一般为 chunked，出现在 Content-Length 不能确定的情况下，表示服务器不知道响应板体的数据大小，一般同时出现Content-Encoding响应头|
|Set-Cookie|设置 Cookie|
|Location|重定向到另一个 URL，如输入浏览器就输入 baidu.com 回车，会自动跳转到www.baidu.com 就是通过这个响应头控制的|
|Server|后台服务器|


>作者：developerHaoz
链接：https://juejin.im/post/5a0ce1d95188253e24708454
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。