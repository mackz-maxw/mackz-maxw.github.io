---
title: 代码随想录 | 八股-HTTP请求
categories: comp basic
lang: zh-CN
---

### HTTP请求报文和响应报文是怎样的，有哪些常见的字段？

HTTP报文分为请求报文和响应报文。

#### **请求报文** 
请求报文主要由请求行、请求头、空行、请求体构成。 
请求行包括如下字段：

* 方法（Method）：指定要执行的操作，如 GET、POST、PUT、DELETE 等。
* 资源路径（Resource Path）：请求的资源的URI（统一资源标识符）。
* HTTP版本（HTTP Version）：使用的HTTP协议版本，如 HTTP/1.1 或 HTTP/2.0。

**请求头**的字段较多，常使用的包含以下几个：

* Host：请求的服务器的域名。
* Accept：客户端能够处理的媒体类型。
* Accept-Encoding：客户端能够解码的内容编码。
* Authorization：用于认证的凭证信息，比如token数据。
* Content-Length：请求体的长度。
* Content-Type：请求体的媒体类型。（如 `application/json`、`text/html` 等）
* Cookie：存储在客户端的cookie数据。
* If-None-Match：与 `ETag` 配合使用，用于缓存控制。
* Connection：管理连接的选项，如 keep-alive。

方便记忆：

##### 1. **基本身份/环境信息**（我是谁、能接受什么）
* `Host`：我要访问哪个服务器（主机名）
* `User-Agent`：我是哪个浏览器（可选提）
* `Accept` / `Accept-Encoding`：我能接受的内容类型/压缩方式
    

##### 2. **身份认证/状态维持**
* `Authorization`：我的身份凭证（如 token）
* `Cookie`：我本地保存的 cookie，带回来给你看
    

##### 3. **请求体相关（仅 POST/PUT 用）**
* `Content-Type`：我发给你是什么格式（如 JSON）
* `Content-Length`：我发给你的内容多大
    

##### 4. **缓存控制（和上次请求有关）**
* `If-None-Match`：上次你的资源 ETag 是 XX，看看现在变没
* `If-Modified-Since`：上次的修改时间是 XX，资源有变吗？
* `Cache-Control`：我是否希望使用缓存？

**空行**是请求头部和请求主体之间的空行，用于分隔请求头部和请求主体。

**请求体**通常用于 POST 和 PUT 请求，包含发送给服务器的数据。

#### 响应报文

HTTP响应报文是服务器向客户端返回的数据格式，用于传达服务器对客户端请求的处理结果以及相关的数据。一个标准的HTTP响应报文通常包含状态行、响应头、空行、响应体。

状态行包含HTTP版本、状态码和状态消息。例如：HTTP/1.1 200 OK

响应头部也是以键值对的形式提供的额外信息，类似于请求头部，用于告知客户端有关响应的详细信息。一些常见的响应头部字段包括：

* Content-Type：指定响应主体的媒体类型。
* Content-Length：指定响应主体的长度（字节数）。
* Server：指定服务器的信息。
* Expires: 响应的过期时间，之后内容被认为是过时的。
* ETag: 响应体的实体标签，用于缓存和条件请求。
* Last-Modified： 资源最后被修改的日期和时间。
* Location：在重定向时指定新的资源位置。
* Set-Cookie：在响应中设置Cookie。
* Access-Control-Allow-Origin: 跨源资源共享（CORS）策略，指示哪些域可以访问资源。

方便记忆：

#### 1. **内容信息**
* `Content-Type`：我返回的是什么格式（如 HTML/JSON）
* `Content-Length`：内容有多大

#### 2. **服务器身份**
* `Server`：我是哪个服务器（如 nginx）

#### 3. **状态维持**
* `Set-Cookie`：设置 cookie，下次你带回来

#### 4. **缓存相关**
* `ETag`：这是资源的版本号
* `Last-Modified`：资源上次修改时间
* `Cache-Control` / `Expires`：多久前有效/过期时间

#### 5. **跨域和重定向**
* `Access-Control-Allow-Origin`：谁能访问我（CORS）
* `Location`：去这个新地址吧（重定向）

空行（Empty Line）在响应头和响应体之间，表示响应头的结束。

响应体是服务端实际传输的数据，可以是文本、HTML页面、图片、视频等，也可能为空。

### HTTP有哪些请求方式？
* GET：请求指定的资源。
* POST：向指定资源提交数据进行处理请求（例如表单提交）。
* PUT：更新指定资源。
* DELETE：删除指定资源。
* HEAD：获取GET请求相同响应的报文首部，不返回报文主体。
* OPTIONS：查询服务器支持的请求方法。
* PATCH：对资源进行部分更新

### GET请求和POST请求的区别
* 用途：GET请求通常用于获取数据，POST请求用于提交数据。
* 数据传输：GET请求将参数附加在URL之后，POST请求将数据放在请求体中。
* 安全性：GET请求由于参数暴露在URL中，安全性较低；POST请求参数不会暴露在URL中，相对更安全。
* 数据大小：GET请求受到URL长度限制，数据量有限；POST请求理论上没有大小限制。
* 幂等性：GET请求是幂等的，即多次执行相同的GET请求，资源的状态不会改变；POST请求不是幂等的，因为每次提交都可能改变资源状态。
* 缓存：GET请求可以被缓存，POST请求默认不会被缓存。