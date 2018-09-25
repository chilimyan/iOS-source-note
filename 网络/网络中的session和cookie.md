# 网络中的session和cookie
* `cookie`一般是通过服务端生成，然后响应返回给客户端，客户端保存起来，用来记录用户色状态，区分用户。
 iOS的`Cookie`内容如下：

 ```
 <NSHTTPCookie
 	version:0
 	name:JSESSIONID
 	value:4C965991B6E31A4006E36A9AD63729A0
 	expiresDate:'(null)'
 	created:'2018-09-25 07:53:05 +0000'
 	sessionOnly:TRUE
 	domain:114.255.63.244
 	partition:none
 	path:/RedseaPlatform
 	isSecure:FALSE
 	isHTTPOnly: YES
  path:"/RedseaPlatform" isSecure:FALSE isHTTPOnly: YES>
 ```
* `session`是存储在服务端，服务端生成`session`后会返回给客户端。`sessionId`一般是存储在`cookie`里面的，然后客户端把`cookie`传给服务端，服务端解析出`sessionId`，然后根据`sessionId`判断当前用户。

* iOS有个`NSHTTPCookieStorage`类对`cookie`进行管理，并且在`UIWebView`可以进行共享。相同 Key 值得新的 `Cookie` 会覆盖旧的 `Cookie`，所以要修改`cookie`可以直接给它赋新值就好了。
  WKWebView的cookie有点麻烦。可以参考这里[WKWedView的坑](https://mp.weixin.qq.com/s/rhYKLIbXOsUJC_n6dt9UfA)
#### 如何保证cookie的安全
* 对 `Cookie` 进行加密处理
* 只在 `Https` 上携带 `Cookie`。
* 设置 `Cookie` 为 `httpOnly`，防止跨站脚本攻击。

