# GET与POST的区别
1、
`GET`把参数包含在`URL`中
`POST`通过`request body`传递参数
2、
`GET`请求在`URL`中传送的参数是有长度限制的，一般2KB
`POST`没有限制
3、
`GET`请求会被浏览器主动缓存
`POST`不会缓存，除非手动设置
4、
`GET`请求只能进行`url`编码
`POST`支持多种编码方式
5、
`GET`比`POST`更不安全，因为参数直接暴露在`URL`上，所以不能用来传递敏感信息

GET和POST本质上是没有区别的。底层都是基于TCP/IP。

### 最重大区别：
#### GET产生一个TCP数据包
对于GET方式的请求，浏览器会把`http header`和`data`一并发送出去，服务器响应200（返回数据）；
#### POST产生两个TCP数据包
对于`POST`，浏览器先发送`header`，服务器响应`100 continue`，浏览器再发送`data`，服务器响应`200 ok`（返回数据）。


