---
title: HTTP保持登录状态的机制
date: 2019年8月31日 21:25:00
categories: HTTP
tags: [Session, Cookies]
---

## HTTP保持登录状态

### 1. 无状态的HTTP协议

HTTP无状态是指：HTTP协议对事物处理是没有记忆能力的，也就是说服务器不知道客户端是什么状态。当我们向服务器发送请求后，服务器解析此请求，然后返回对应的响应，服务器负责完成整个过程。这个过程是独立的的，**服务器不会记录前后状态的变化**，也就是缺失状态记录。这就是说如果后续处理需要前面的信息，就必须重传，这导致需要额外传递一些前面的重复请求，才能获取后续响应，然而这种效果显然太浪费资源了。
<!--more-->
于是两种用于保持HTTP连接状态的技术就出现了，即Session和Cookies。Session在服务端，也就是网站的服务器，用来**保存用户的会话信息**；Cookie在客户端，有了Cookies，浏览器在下次访问网站是会自动附带上它发送给服务器，服务器通过识别Cookies鉴别出是哪个用户，**判断是否是登录状态**，然后返回对应的响应。

Cookies保存了登录的凭证，有了它，只需要在下次请求中携带Cookies发送请求就不用重新输入用户名，密码等信息重新登录了。所以在爬虫中，一般会将登录成功后获取的Cookies放在请求头中直接请求，而不必重新模拟登录。

### 2. 原理剖析

#### Session

Session指有始有终的一系列动作/消息。比如，打电话时，从拿起电话拨号到挂断电话这一过程可以称为一个Session。

而Web中，Session对象用来存储特定用户Session所需的属性和配置信息。这样，当用户在各个Web网页之间跳转是，存储在Session对象的变量不会丢失，而是在整个Session中一直存在下去。当用户请求来自应用程序的web网页时，如果该用户还没有Session，则web服务器会自动创建一个Session对象。而当Session过期或被放弃的时候，服务器就会终止该Session。

#### Cookies

Cookies指某些网站为了辨别用户身份，进行Session跟踪而存储在本地终端上的数据。

当客户端第一次请求服务器是，服务器会返回一个请求头中带有Set-Cookie的字段相应给客户端，用来标记是哪个用户，客户端会把Cookies保存起来，Cookies携带了SessionID信息，服务器检查该Cookies即可查找到对应的Session是什么，然后再判断Session来一次辨别用户状态。

成功登录某个网站时，服务器会告诉客户端设置哪些Cookies信息，在后序访问客户端会把Cookies发送给服务器，服务器再找到对应的Session加以判断。如果Session中某些设置登录状态是有效的，就证明用户处于登录状态，此次返回登录之后才可以查看网页内容，浏览器进行解析，用户就可以看到内容了。

### 3. Cookies内容

1. name 该Cookies的名称，一旦创建，名称不可更改
2. Value 该Cookie的值，如果值是Unicode字符，需要为字符编码，如果是二进制数据，则需要BASE64编码
3. Domain：可以访问该Cookie的域名。如果设置为12306.com，则所有以12306.com结尾的域名都可以访问该Cookie。
4. Max age:该Cookie失效的时间，单位为秒, 常与Expires一起使用。Max age如果是正数，则该Cookie在Max Age秒后失效。如果为负数，则关闭浏览器时Cookie失效。浏览器也不会以任何形式保存该Cookie。
5. Path：该Cookie的使用路径。如果设置为/path/，则只有路径为/path/的页面可以访问该Cookie。如果设置为/，则本域名下所有页面都可以访问该Cookie。
6. Size：该Cookie的大小。
7. HTTP字段：Cookie的httponly属性。若此属性的值为true，则只用在HTTP头带有此Cookie信息，而不能通过document.cookie来访问此Cookie。
8. Secure:该Cookie是否仅被使用安全协议传输。安全协议有HTTP和SSL等，在网络上传输数据前先将数据加密。默认为false。

### 4.常见误区

#### 会话Cookie和持久Cookie

传说中会话Cookie是把Cookie放在浏览器内存中，浏览器关闭后该Cookie失效，持久Cookie则会保持到客户端的硬盘中，下次还可以继续使用，长久保持用户登录状态。

传说是假的，过期时间是由Cookie的Max Age或Expire决定的。持久化Cookie是把有效实际那设置的比较长，这样下次访问时仍然携带之前的cookie，就可以直接保持登录状态。

#### Session误区

对于Session来说，除非程序员通知服务器删除Session，否则服务器会一直保留。

但是在我们关闭浏览器后，浏览器不会主动关闭之前通知服务器它将要关闭，所以服务器不会有机会知道浏览器已经关闭。

大部分Session机制使用会话Cookis来保存SessionID信息，而关闭浏览器后，**Cookies消失了，再次连接服务器时，就无法找到原来的Session了**。如果服务器设置的Cookies被浏览器保存到硬盘上，或者用某种手段改写浏览器发出HTTP请求头把原来的Cookies发送给服务器，则当再次打开服务器是，仍然能够找到原来的SessionID，仍然可以保存登录状态。

由于关闭浏览器不会使Session被删除，这就需要服务器为Session设置一个失效时间，当距离客户端上一次使用Session的时间超过这个失效时间时，服务器就认为客户端停止了活动，会把Session删除以节省存储空间。

### 参考

[1] [Session和Cookies的原理及代码实现](https://www.imooc.com/article/38017)

