---
title: CORS跨域资源共享总结
date: 2016-12-13 21:56:00
tags: [Http, CORS]
category: http
---
使用`XMLHttpRequest`对象和`Fetch`发起HTTP请求就必须遵守[同源策略](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy).
而CORS就是W3C推出的一个标准,用来解决跨域问题.

### CORS

CORS,全称是"跨域资源共享"(Cross-origin resource sharing).这种方式让Web应用服务器能支持跨站访问控制,从而使得安全地进行跨站数据传输成为可能.

跨源资源共享标准通过新增一系列 HTTP 头,让服务器能声明哪些来源可以通过浏览器访问该服务器上的资源.另外,对那些会对服务器数据造成破坏性影响的 HTTP 请求方法(特别是 GET 以外的 HTTP 方法,或者搭配某些MIME类型的POST请求),标准强烈要求浏览器必须先以 OPTIONS 请求方式发送一个预请求(preflight request),从而获知服务器端对跨源请求所支持 HTTP 方法.在确认服务器允许该跨源请求的情况下,以实际的 HTTP 请求方法发送那个真正的请求.服务器端也可以通知客户端,是不是需要随同请求一起发送信用信息(包括 Cookies 和 HTTP 认证相关数据).

浏览器将CORS请求分成两类: 简单请求(simple request) 和非简单请求(not-so-simple request);

<!-- more -->

### 简单请求

所谓的简单,是指:
* 只使用`GET`,`HEAD`或者`POST`请求方法.如果使用`POST`向服务器端传送数据,则数据类型(`Content-Type`)只能是` application/x-www-form-urlencoded`,`multipart/form-data`或`text/plain`中的一种.
* 不使用自定义请求头(例:`X-Requested-With`)

> 这些跨站请求与以往浏览器发出的跨站请求并无异同.并且,如果服务器不给出适当的响应头,则不会有任何数据返回给请求方.因此,那些不允许跨站请求的网站无需为这一新的 HTTP 访问控制特性担心.

CORS简单请求发起时,会自动在头部信息添加一个`Origin`字段:
```
// 请求
GET /cors HTTP/1.1
Origin: http://api.qiutc.me
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
表示本次请求来自哪个源(协议+域名+端口),服务端会获取到这个值,然后判断是否同意这个请求.

如果服务端允许请求,则会在返回的头信息中多出几个字段:
```
// 返回
Access-Control-Allow-Origin: http://api.qiutc.me
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: Info
Content-Type: text/html;charset=utf-8
```
其中以`Access-Control`开头的字段就是CORS请求,服务端返回的相关字段:
* `Access-Control-Allow-Origin`
  `required`.它的值是请求时`Origin`字段的值或者是`*`,如果是`*`,则表示接受任意域名的请求.
* `Access-Control-Allow-Credentials`
   可选,它的值是一个布尔值,表示是否允许发送Cookie.默认情况下,Cookie不包括在CORS请求之中.设为true,即表示服务器明确许可,Cookie可以包含在请求中,一起发给服务器.

   需要发送`cookie`的时候,还需要注意将`XMLHttpRequest`的`withCredentials`标志设置为true,
   从而使得Cookies可以随着请求发送:
   ```javascript
   var xhr = new XMLHttpRequest();
   xhr.withCredentials = true;
   ```
   **需要注意的是,如果要发送`Cookie`,`Access-Control-Allow-Origin`就不能设为`*`,必须指定明确的、与请求网页一致的域名.同时,`Cookie`依然遵循同源政策,只有用服务器域名设置的`Cookie`才会上传,其他域名的`Cookie`并不会上传,且原网页代码中的`document.cookie`也无法读取服务器域名下的`Cookie`.**

* `Access-Control-Expose-Headers`
   可选,设置浏览器允许访问的服务器的头信息的白名单.
   ```
   Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header
   ```
   这样,`X-My-Custom-Header` 和 `X-Another-Custom-Header `这两个头信息,都可以被浏览器得到.




### 非简单请求

如果浏览器发送的请求不满足上述的简单请求的条件,那么浏览器在进行CORS跨域资源共享的请求就是非简单请求:
* 请求以 `GET`, `HEAD` 或者 `POST` 以外的方法发起请求.或者,使用 `POST`，但请求数据为 `application/x-www-form-urlencoded`, `multipart/form-data` 或者 `text/plain` 以外的数据类型.比如说,用 `POST` 发送数据类型为 `application/xml` 或者 `text/xml` 的 `XML `数据的请求.
* 使用自定义请求头(比如添加诸如 `X-PINGOTHER`)

>  从Gecko 2.0开始,`text/plain`,`application/x-www-form-urlencoded` 和 `multipart/form-data`类型的数据都可以直接用于跨站请求,而不需要先发起“预请求”了.之前,只有 `text/plain` 可以不用先发起“预请求”,进行跨站请求.

非简单请求,会在正式通信之前,发送一次`预请求`,即`OPTIONS`请求,也称为`预检请求(preflight)`.预请求的目的是浏览器询问服务器,当前网页所在的域名是否在服务器的许可名单之中,以及可以使用哪些HTTP动词和头信息字段.只有得到肯定答复,浏览器才会发起正式的`XMLHttpRequest`请求,否则就会报错.

预检请求发送的请求:
```
OPTIONS /cors HTTP/1.1
Origin: http://api.qiutc.me
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
Host: api.qiutc.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
```
预检请求用的请求方法是`OPTIONS`,表示这个请求是用来询问的.头信息里面,关键字段是`Origin`,表示请求来自哪个源.
除了`Origin`字段,预检请求的头信息包括两个特殊字段.

* `Access-Control-Request-Method`
必选字段,用来列出浏览器的CORS请求会用到哪些HTTP方法,上例是`PUT`.

* `Access-Control-Request-Headers`
该字段是一个逗号分隔的字符串,指定浏览器CORS请求会额外发送的头信息字段,上例是`X-Custom-Header`.

预检请求的返回:
```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://api.qiutc.me
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Content-Type: text/plain
```
* `Access-Control-Allow-Methods`
`required`,它的值是逗号分隔的一个字符串,表明服务器支持的所有跨域请求的方法.注意,返回的是所有支持的方法,而不单是浏览器请求的那个方法.这是为了避免多次”预检”请求.

* `Access-Control-Allow-Headers`
如果浏览器请求包括`Access-Control-Request-Headers`字段,则`Access-Control-Allow-Headers`字段是必需的.它也是一个逗号分隔的字符串,表明服务器支持的所有头信息字段,不限于浏览器在”预检”中请求的字段.

* `Access-Control-Max-Age`
该字段可选,用来指定本次预检请求的有效期,单位为秒.上面结果中,有效期是20天(1728000秒),即允许缓存该条回应1728000秒(即20天),在此期间,不用发出另一条预检请求.

### 小记
就兼容性来说,ie10+ 和现代浏览器都是支持的.
!['兼容'](https://qiutc.me/img/cross-domain-cors.png)
CORS与JSONP的使用目的相同,但是比JSONP更强大.
JSONP只支持`GET`请求,CORS支持所有类型的HTTP请求.JSONP的优势在于支持老式浏览器,以及可以向不支持CORS的网站请求数据.


参考资料:
* [跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
* [HTTP访问控制(CORS)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS#)
