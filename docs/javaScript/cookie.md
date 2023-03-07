# Cookie

## Name和Value

Name和Value是一个键值对。Name是Cookie的名称，Cookie一旦创建，名称便不可更改，一般名称不区分大小写；Value是该名称对应的Cookie的值，如果值为Unicode字符，需要为字符编码。如果值为二进制数据，则需要使用BASE64编码。

## Domain

Domain决定Cookie在哪个域是有效的，也就是决定在向该域发送请求时是否携带此Cookie，Domain的设置是对子域生效的，如Doamin设置为 .a.com,则b.a.com和c.a.com均可使用该Cookie，但如果设置为b.a.com,则c.a.com不可使用该Cookie。Domain参数必须以点(".")开始。

## Path

Path是Cookie的有效路径，和Domain类似，也对子路径生效，如Cookie1和Cookie2的Domain均为a.com，但Path不同，Cookie1的Path为 /b/,而Cookie的Path为 /b/c/,则在a.com/b页面时只可以访问Cookie1，在a.com/b/c页面时，可访问Cookie1和Cookie2。Path属性需要使用符号“/”结尾。

## Expires/Max-age

Expires和Max-age均为Cookie的有效期，Expires是该Cookie被删除时的时间戳，格式为GMT,若设置为以前的时间，则该Cookie立刻被删除，并且该时间戳是服务器时间，不是本地时间！若不设置则默认页面关闭时删除该Cookie。
Max-age也是Cookie的有效期，但它的单位为秒，即多少秒之后失效，若Max-age设置为0，则立刻失效，设置为负数，则在页面关闭时失效。Max-age默认为 -1。

## HttpOnly

HttpOnly值为 true 或 false,若设置为true，则不允许通过脚本document.cookie去更改这个值，同样这个值在document.cookie中也不可见，但在发送请求时依旧会携带此Cookie。

## Secure
Secure为Cookie的安全属性，若设置为true，则浏览器只会在HTTPS和SSL等安全协议中传输此Cookie，不会在不安全的HTTP协议中传输此Cookie。
 

## SameSite (CSRF安全问题)
SameSite用来限制跨站 Cookie，从而减少安全风险。它有3个属性，分别是：

* Scrict最为严格，完全禁止第三方Cookie，跨站点时，任何情况下都不会发送Cookie

* 网站可以选择显式关闭SameSite属性，将其设为None。不过，前提是必须同时设置Secure属性（Cookie 只能通过 HTTPS 协议发送），否则无效。

* 网站可以选择显式关闭SameSite属性，将其设为None。不过，前提是必须同时设置Secure属性（Cookie 只能通过 HTTPS 协议发送），否则无效。

`跨站：协议不同或者顶级域名或二级域名相同`

## 基本使用

```js
// key=value 逗号隔开属性
document.cookie="username=John Smith; expires=Thu, 18 Dec 2043 12:00:00 GMT; path=/";
```