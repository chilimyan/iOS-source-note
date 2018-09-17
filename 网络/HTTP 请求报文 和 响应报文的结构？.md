# HTTP 请求报文 和 响应报文的结构？
### 请求报文
#### 请求头
对客户端的环境描述,客户端请求的主机地址等信息：

```Accept: text/html ( 客户端所能接收的数据类型 )
Accept-Language: zh-cn ( 客户端的语言环境 )
Accept-Encoding: gzip( 客户端支持的数据压缩格式 )
Host: m.baidu.com( 客户端想访问的服务器主机地址 ) 
User-Agent: Mozilla/5.0(Macintosh;Intel Mac OS X10.10 rv:37.0) Gecko/20100101Firefox/37.0( 客户端的类型,客户端的软件环境 )
```
#### 请求行
请求方法,请求资源路径,`http`协议版本. `"GET /resources/images/ HTTP/1.1"`

#### 请求体
客户端发给服务器的具体数据,比如文件/图片等
### 响应报文
#### 响应头
包含了对服务器的描述,对返回数据的描述.

```
Content-Encoding: gzip(服务器支持的数据压缩格式) Content-Length: 1528(返回数据的长度)
Content-Type:application/xhtml+xml;charset=utf-8(返回数据的类型)
Date: Mon,15Jun201509:06:46GMT(响应的时间) 
Server: apache (服务器类型)
```
#### 响应行
包含了http协议版本,状态吗,状态英文名称.
`"HTTP/1.1 200 OK"`
#### 响应内容
服务器返回给客户端的具体数据(图片/html/文件...).

