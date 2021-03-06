# Socket
`Socket`是对`TCP/IP`协议的封装，`Socket`本身并不是协议，而是一个调用接口（`API`），它简化了程序员的操作，知道对方的`IP`以及`PORT`就可以给对方发送消息，再由服务器端来处理发送的这些消息。通过`Socket`，我们才能使用`TCP/IP`协议。
#### 建立Socket连接
建立`Socket`连接至少需要一对套接字，其中一个运行于客户端，称为`ClientSocket`，另一个运行于服务器端，称为`ServerSocket`。
套接字之间的连接过程分为三个步骤：

* 服务器监听
* 客户端请求
* 连接确认。

1. **服务器监听**：服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态，等待客户端的连接请求。
2. **客户端请求**：指客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求。
3. **连接确认**：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，双方就正式建立连接。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求。

#### Socket连接与TCP连接
创建`Socket`连接时，可以指定使用的传输层协议，`Socket`可以支持不同的传输层协议（TCP或UDP），当使用TCP协议进行连接时，该`Socket`连接就是一个TCP连接。

#### Socket连接与http连接
1. 通常情况下Socket连接就是TCP连接，因此Socket连接一旦建立，通信双方即可开始相互发送数据内容，直到双方连接断开
2. HTTP连接使用的是“请求—响应”的方式，不仅在请求时需要先建立连接，而且需要客户端向服务器发出请求后，服务器端才能回复数据。
3. 很多情况下，需要服务器端主动向客户端推送数据，保持客户端与服务器数据的实时与同步。此时若双方建立的是`Socket`连接，服务器就可以直接将数据传送给客户端；若双方建立的是`HTTP`连接，则服务器需要等到客户端发送一次请求后才能将数据传回给客户端

### Socket的长连接与短连接
**长连接**：整个通讯过程，客户端和服务端只用一个`Socket`对象，长期保持`Socket`的连接
**短连接**：每次请求，都新建一个`Socket`,处理完一个请求就直接关闭掉`Socket`
#### 如何实现一个长连接
开启`Socket`的同时对应的流也不能关闭！
那么每次要发消息就直接往流里面任进去数据，然后服务端调用`flush()`方法强制刷新就行了？这样肯定不行。客户端无法正常接收消息，一直在`read`方法那里阻塞。因为客户端无法知道消息什么时候接收完。所以`read`方法会一直循环等待读取。所以要求服务端每发送完一段消息，调用`flush()`方法刷新前就插入一个结束符合标志，这样客户端解析道结束符合标志就会推出`read`操作。这样就实现了一个长连接。

#### 关闭Socket和流时需要注意以下事情
虽然前面说了流关闭了，`Socket`就不可用了，但是，我们还是要显式的关闭`Socket`的，因为在`Socekt`中还有中状态：叫做半连接状态，当我们只是用到输出流的时候，我们关闭了输出流,并且不能直接调用`close`方法，只能调用`shutDown`对应方法（具体请查看java API），其实输入流还是连接着的（只是我们没有用到而已！），这时候，如果没有显式关闭`Soceket`，很容易导致内存泄露，所以，所有流`Socket`都要显式关闭

#### 长连接和短连接适用场景
**短连接**：适用于网页浏览等数据刷新频度较低的场景。
**长连接**：适用于客户端和服务端通信频繁的场景，例如聊天室，实时游戏等。

### Socket半包，粘包与分包的问题处理
#### 半包
指接受方没有接受到一个完整的包，只接受了部分。这种情况主要是由于`TCP`为提高传输效率，将一个包分配的足够大，导致接受方并不能一次接受完。（ 在长连接和短连接中都会出现）。 
#### 半包的处理
`Socket`内部默认的收发缓冲区大小大概是`8K`，如果一次性包的大小小于8K一般不会出现半包。
半包处理方式有：

1. 通过包头、包长、包体的协议形式，当服务器端获取到指定的包长时才说明获取完整。
2. 指定包的结束标识，这样当我们获取到指定的标识时，说明包获取完整。

#### 粘包
指发送方发送的若干包数据到接收方接收时粘成一包，从接收缓冲区看，后一包数据的头紧接着前一包数据的尾。
粘包现象的原因：

* 发送方引起的粘包是由TCP协议本身造成的，TCP为提高传输效率，发送方往往要收集到足够多的数据后才发送一包数据。若连续几次发送的数据都很少，通常TCP会根据优化算法把这些数据合成一包后一次发送出去，这样接收方就收到了粘包数据。
* 接收方引起的粘包是由于接收方用户进程不及时接收数据，从而导致粘包现象。这是因为接收方先把收到的数据放在系统接收缓冲区，用户进程从该缓冲区取数据，若下一包数据到达时前一包数据尚未被用户进程取走，则下一包数据放到系统接收缓冲区时就接到前一包数据之后，而用户进程根据预先设定的缓冲区大小从系统接收缓冲区取数据，这样就一次取到了多包数据。

#### 粘包的处理
1. 短连接的情况下，不用考虑粘包的情况。
2. 如果发送数据无结构，如文件传输，这样发送方只管发送，接收方只管接收存储就ok，也不用考虑粘包。

粘包的处理方式：

1. 接收方创建一预处理线程，对接收到的数据包进行预处理，将粘连的包分开。
#### 分包
出现粘包的时候我们的接收方要进行分包处理。


