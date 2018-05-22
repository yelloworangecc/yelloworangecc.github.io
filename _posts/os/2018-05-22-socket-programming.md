---
layout: post
title:  "socket编程手册"
summary: 从操作系统角度讲,socket是存在于计算机网络节点（例如主机）上的用于发送或接收数据的一个端点，是网络软件（协议栈）的端点,也是系统资源的一种表现形式.从编程角度讲,socket本质是编程接口(API),是对TCP/IP的封装.本文基于freeBSD,整理了socket编程中常用函数的用法.
featured-img: os
# comments: true
---
从操作系统角度讲,socket是存在于计算机网络节点（例如主机）上的用于发送或接收数据的一个端点，是网络软件（协议栈）的端点,也是系统资源的一种表现形式.从编程角度讲,socket本质是编程接口(API),是对TCP/IP的封装.本文基于freeBSD,整理了socket编程中常用函数的用法.

P.S. 作为一名工作了3年的C++程序员,在工作期间都没有比较深入的了解socket编程,每次遭遇socket的时候都要查手册,过段时间又忘记了,反复学习和忘记的过程其实就是低效率的表现,因此我在这片博客中对socket常用函数进行罗列和总结,希望在以后遇到socket的时候代码能够"信手拈来".

## socket ##

**原型:**
```
int socket(int domain, int type, int protocol);
```

**说明:**
  * 函数创建一个通信的端节点并返回一个描述符
  * 参数domain指定了通信所在的域,也就是选定了要使用的协议族.在`<sys/socket.h>`中定义的协议族有:
    + **PF_LOCAL** 主机内部协议族,早前也叫`PF_UNIX`
	+ **PF_UNIX** 同上,过时的叫法
	+ **PF_INET** IPv4协议族
	+ **PF_ROUTE** 内部路由协议族
	+ **PF_KEY** 内部键管理功能
	+ **PF_INET6** IPv6协议族
	+ **PF_NDRV** 网络设备原始数据访问
  * 参数type指定了通信的语义,值有:
    + **SOCK_STREAM** 提供顺序的,可靠的,双向连接的字节流.可以支持超出频带的数据传输机制.
	+ **SOCK_DGRAM** 支持数据报服务(无连接,不可靠,固定最大长度).
	+ **SOCK_RAM** 提供对内部网络协议和接口的访问,只对超级用户可用.
  * 参数protocol指定socket要使用的协议,一般在一个协议族中只有一种与type对应的协议.然而也有可能存在多种协议的情况,此时就必须用这个参数来确定.协议值与通信所处的域有关.
  * `SOCK_STREAM`是全双工的字节流,与管道类似.一个流socket必须处于连接状态才能收发数据.与其他socket建立连接用函数`connect`或`connectx`,一旦连接建立,数据传输可以用函数`read`或`write`或这两个函数的变种函数,会话完成时调用函数`close`.带外数据也可以被传输(参考`send`和`recv`函数的说明).
  * 实现了`SOCK_STREAM`的通信协议保证数据不会丢失或出现重复.如果对端在一定时间长度内没有成功接收到数据,连接被视为断开,并且相关函数调用返回-1,全局变量errno会被设为`ETIMEDOUT`.在没有其他数据传输操作时,协议可以利用每分钟强制发送随意的数据来保持socket的活动状态.如果在一个较长的时间内(例如5分钟)没有在空闲连接上检测到保活数据,则返回一个错误.当进程在断开的流上发送数据会产生SIGPIPE信号,如果没有处理这个信号,那这个进程将会退出.
  * `SOCK_DGRAM`和`SOCK_RAW`允许用函数send向通信者发送数据报,数据报一般使用函数recvfrom来接收,函数返回数据报和通信者的地址.
  * 通过fcntl可以指定一个进程组在带外数据到达时接收SIGURG信号.函数还可以使能非阻塞IO以及通过SIGIO信号异步通告IO事件.
  * `setsockopt`和`getsockopt`可以设置和获取socket的选项参数.

**返回值:**
  * 错误发生时返回-1,否则返回socket的描述符.

**错误码:**
  * [EACCES] 没有足够的权限创建指定类型或协议的socket.
  * [EAFNOSUPPORT] 不支持的地址族.
  * [EMFILE] 进程文件描述符表满.
  * [ENFILE] 系统文件表满.
  * [ENOBUFS] 缓存不足.
  * [ENOMEM] 内存不足.
  * [EPROTONOSUPPORT] 域不支持的类型或协议.
  * [EPROTOTYPE] 协议不支持的socket类型.

## getsockopt setsockopt ##
**原型:**
```
int getsockopt(int socket, int level, int option_name,
         void *restrict option_value, socklen_t *restrict option_len);
int setsockopt(int socket, int level, int option_name,
         const void *option_value, socklen_t option_len);
```

**说明:**
  * 函数操作socket的选项.选项可能存在于多个协议层级,但总表现在最高层的socket层.
  * 当操作socket的选项时,必须指定选项所处的层级和选项的名字.在socket层操作时,参数`level`被制定为`SOL_SOCKET`.在其他层级操作时,需要提供选项所处协议的协议号.例如,为了明确一个选项是有TCP协议负责解释的,参数`level`应当被设置为TCP的协议号.
  * 参数`option_value`和`option_len`在`setsockopt`用来访问选项的值.对于`getsockopt`,他们用于保存所请求选项的缓冲区.`option_len`既是入参也是出参数,它应当被初始化为缓冲区的大小,在返回时修改为实际值的大小.没有选项需要提供或获取时,`option_value`可以指定为空.
  * `option_name`和其他任何选项不经过任何解释处理就转入到协议模块进行解析.在头文件<sys/socket.h>中有socket层级的选项定义.其他协议层级的选项格式和名字各不相同.
  * 大多数socket层级的`option_value`用一个int参数来实现.对于`setsockopt`,用非0值表示选项的开启,0表示禁用选项.
  * 以下的选项可以在socket层级被识别.除非另有说明,每个选项使用`getsockopt`来检测,使用`setsockopt`来设置.
    - `SO_DEBUG` 允许记录调试信息
	- `SO_REUSEADDR` 允许重用本地地址
	- `SO_REUSEPORT` 允许重复绑定地址和端口
	- `SO_KEEPALIVE` 开启保活
	- `SO_DONTROUTE` 对外发数据开启路由透传
	- `SO_LINGER` 有数据时延迟关闭
	- `SO_BROADCAST` 给予传播广播消息的权限
	- `SO_OOBINLINE` 允许接收带外数据
	- `SO_SNDBUF` 设置发送缓存大小
	- `SO_RCVBUF` 设置接收缓存大小
	- `SO_SNDLOWAT` 设置发送最小计数
	- `SO_RCVLOWAT` 设置接收最小技术
	- `SO_SNDTIMEO` 设置发送超时时间
	- `SO_RCVTIMEO` 设置接收超时时间
	- `SO_TYPE` 获取socket类型
	- `SO_ERROR` 获取或清空错误状态
	- `SO_NOSIGPIPE` 返回`EPIPE`错误码而不是产生`SIGPIPE`信号
	- `SO_NREAD` 获取可读数据个数
	- `SO_NWRITE` 获取待发数据个数
	- `SO_LINGER_SEC` 设置关闭等待时间

**返回值:**
  * 成功时返回0,否则返回-1并将错误码保存到全局变量.

**错误码:**
  * [EBADF] 参数`socket`不是合法的文件描述符.
  * [EFAULT] 参数`option_value`指定的地址不是合法的线程地址空间的地址.
  * [EINVAL] 在指定的层级上,该操作不合法.
  * [ENOBUFS] 系统没有足够资源来完成操作.
  * [ENOMEM] 没有足够内存来完成系统调用.
  * [ENOPROTOOPT] 在指定层级没有定义该选项.
  * [ENOTSOCK] 参数`socket`不是socket.
  * [EDOM] 参数`option_value`超出了带宽.
  * [EISCONN] 参数`socket`以处于连接状态,无法完成操作.
  * [EINVAL] socket已经被关闭.

## bind ##
**原型:**
```
int bind(int socket, const struct sockaddr *address, socklen_t address_len);
```

**说明:**
  * 函数将名称绑定到socket,刚创建时socket存在于一个名字空间(地址族)但是还未分配名称(地址),函数将参数`address`分配到socket.
  * UNIX域绑定名称会在文件系统中创建一个socket,这种情况下必须由调用者负责删除(使用`unlink`函数).
  * 不同域绑定名称的规则不同,具体请参考系统手册.
  
**返回值:**
  * 成功返回0,否则返回-1,且在全局变量`errno`指示错误码.
  
**一般错误码:**
  * [EACCES] 当前用户没有访问权限.
  * [EADDRINUSE] 指定的地址已经被占用.
  * [EADDRNOTAVAIL] 指定的地址当前不可获取.
  * [EAFNOSUPPORT] 参数`address`不符合参数`socket`的地址族.
  * [EBADF] 参数`socket`不是合法的文件描述符.
  * [EDESTADDRREQ] 参数`socket`是一个空指针.
  * [EFAULT] 参数`address`在用户地址空间中是一个非法值.
  * [EINVAL] 参数`socket`已经绑定了一个地址,并且协议不允许重新绑定新地址.或者参数`socket`已经被关闭.
  * [ENOTSOCK] 参数`socket`不是一个socket描述符.
  * [EOPNOTSUPP] 参数`socket`不是可以绑定地址的类型.

**unix域的错误码**
  * [EACCES] 路径前缀中的成员不允许搜索,或者节点的父目录没有写权限.
  * [EEXIST] 文件已经存在.
  * [EIO] 在建立目录或分配节点时发生IO错误.
  * [EISDIR] 路径为空.
  * [ELOOP] 在解析路径时遇到过多的符号链接(检测符号链接回路).
  * [ENAMETOOLONG] 路径中某个成员超出最大文件名长度,或者整个路径超出最大路径长度.
  * [ENOENT] 路径中的成员是不存在的文件.
  * [ENOTDIR] 路径中的成员是不存在的目录.
  * [EROFS] 只读文件系统.

## listen ##
**原型:**
```
int listen(int socket, int backlog);
```

**说明:**
  * 创建基于socket的连接需要若干步骤,首先,创建使用`socket`函数创建一个socket,然后,使用`listen`函数创建队列侦听即将到来的连接,最后,使用`accept`函数接受连接.`listen`函数仅用于`SOCK_STREAM`类型的socket.
  * 参数`backlog`定义了连接请求队列的最大长度.如果连接请求达到最大长度,客户端会收到错误码`ECONNERFUSED`.如果协议支持转发,连接请求也会被忽略.

**返回值:**
  * 成功返回0,否则返回-1,并在全局变量errno中保存错误码.

**错误码:**
  * [EACCES] 当前进程没有操作权限.
  * [EBADF] 参数`socket`,不是合法的文件描述符.
  * [EDESTADDREQ] socket没有绑定到本地地址并且协议不允许侦听未绑定地址的socket.
  * [EINVAL] 参数`socket`已经建立了连接.
  * [ENOTSOCK] 参数`socket`不是队socket的引用.
  * [EOPNOSUPP] socket的类型不支持`listen`函数.

## accept ##
**原型:**
```
int accept(int socket, struct sockaddr *restrict address, socklen_t *restrict address_len);
```

**说明:**
  * 参数`socket`是一个已经被创建,绑定了一个地址,且正在侦听连接的socket.`accept`函数从连接请求队列提取第一个连接请求,使用与`socket`一样的属性创建一个新的socket并分配一个文件描述符.如果队列中没有请求,并且socket没有被标记为非阻塞,`accept`函数会阻塞调用者知道一个连接请求到达.如果socket被标记为非阻塞并且队列为空,`accept`函数会返回一个错误码.被接受的socket不能用来接受更多连接,原始的socket保持打开状态.
  * 参数`address`是一个出参,会被填入通信层的连接实体的地址,格式由通信所在的域决定.参数`address_len`既是入参也是出参,传入时指定为参数`address`所指空间的大小,返回时指定为地址的实际大小.此函数用于面向连接的socket类型,目前既为`SOCK_STREAM`类型.
  * 可以用函数`select`函数来选择性接受连接并进行读写操作.
  * 对于像ISO或者DATAKIT这样的有明确的确认过程的协议,`accept`函数仅仅将连接请求从队列移除,而并没有实现确认过程.确认过程可以通过在文件描述符上操作普通的read或write来实现,并且拒绝过程可以通过关闭新生成的socket来实现.
  * 要想获取用户连接请求的数据而不确认接受连接可以通过`recvmsg`函数指定参数`msg_iovlen`为0,指定参数`msg_controllen`为非0值来实现,或者也可以通过`getsockopt`函数来请求数据.类似的,拒绝用户连接信息可以通过调用`sendmsg`发送控制信息或者调用`setsockopt`来实现.
  
**返回值:**
  * 出错返回-1,并将错误码保存到全局变量`errno`,成功返回被接受socket的描述符(非负整数).

**错误码:**
  * [EBADF] 参数`socket`不是合法的文件描述符.
  * [ECONNABORTED] 参数`socket`的连接已经断开.
  * [EFAULT] 参数`address`不是一个可写的用户地址空间.
  * [EINTR] `accept`系统调用被信号终止.
  * [EINVAL] 参数`socket`不愿意接受连接.
  * [EMFILE] 进程的描述符表已满.
  * [ENFILE] 系统文件表已满.
  * [ENOMEM] 内存不足以完成操作.
  * [ENOTSOCK] 参数`socket`引用的不是socket.
  * [EOPNOTSUPP] 参数`socket`不是`SOCK_STREAM`类型因此不接受连接.
  * [EWOULDBLOCK] 参数`socket`被标记为非阻塞且没有可被接受的连接.

## select ##
**原型:**
```
int select(int nfds, fd_set *restrict readfds, fd_set *restrict writefds,
           fd_set *restrict errorfds, struct timeval *restrict timeout);
```

**说明:**
  * 函数检查IO描述符集合,几个的地址由参数`readfds`,`writefds`和`errorfds`传入,用来检查描述符是准备好读写或者出现了异常.第一个参数`nfds`描述符会在每个集合中检查,举例来说,从0到`nfds`-1的所有描述符会被检查.(如果你的集合中有两个文件描述符4和17,`nfds`不应该为2而应该为18).在返回时,符合要求的一个描述符的子集替换传入的集合,并且返回集合中就绪的描述符的个数.
  * 描述符集合以比特位的方式存储在整数类型的数组中.以下是操作这样的描述符集合的函数:
    - `FD_ZERO(&fdset)` 初始化描述符集合`fdset`为空.
	- `FD_SET(fd, &fdset)` 在集合`fdset`中包含一个描述符`fd`.
	- `FD_CLR(fd, &fdset)` 从集合`fdset`中移除描述符`fd`.
	- `FD_ISSET(fd, &fdset)` 如果描述符`fd`在结合`fsdet`中返回非0值,否则返回0.
	- `FD_COPY(&fdset_orig, &fdset_copy)`使用集合`fdset_orig`的副本覆盖集合`fdset_copy`.
	描述符小于0,大于`FD_SETSIZE`时,这些函数的行为是未定义的.
  * 如果参数`timeout`是一个非空指针,它指定了函数等待完成的最长时间间隔.如果为空,函数将会无限期的阻塞.为了能轮询,参数`timeout`应该为非空,并且指向一个值为0的`timeval`结构体.参数`timeout`不会被`select`函数修改,并且可以被后续的滴啊用复用,然而,推荐每次调用`select`时都重新初始化.
  * 如果没有感兴趣的描述符,参数`readfds`,`writefds`和`errorfds`可以为空.

**返回值:**
  * 成功返回就绪描述符的数量,失败返回-1.如果超时,函数返回0.失败时,描述符集合不会被修改,并且在`errno`中设置了错误码.

**错误码:**
  * [EAGAIN] 内核无法分配资源.
  * [EBADF] 描述符集合中存在非法值.
  * [EINTR] 接收到信号.
  * [EINVAL] 超时时间不合法.
  * [EINVAL] 参数`ndfs`不和法.

## send sendmsg sendto ##
**原型:**
```
ssize_t send(int socket, const void *buffer, size_t length, int flags);
ssize_t sendmsg(int socket, const struct msghdr *message, int flags);
ssize_t sendto(int socket, const void *buffer, size_t length, int flags,
         const struct sockaddr *dest_addr, socklen_t dest_len);
```

**说明:**
  * 函数用于向另一个socket传输消息.`send`只能用于建立了连接的socket,`sendto`和`sendmsg`适用于任何情况.
  * 目标的地址由参数`dest_addr`和`dest_len`指定.消息的长度由`length`指定.如果消息的长度超过协议定义的范围,函数不会传输消息并且会返回错误码`EMSGSIZE`.
  * `send`不会发出任何失败通告,当发生错误时函数返回-1.
  * 如果没有可用的消息空间来存储要发送的消息,函数通常会阻塞,除非socket被标记为非阻塞模式.`select`函数可以用来检查何时可以发送更多消息.
  * 参数`flags`可以是以下值的组合:
    - `MSG_OOB` 用于传输带外数据,像`SOCK_STREAM`类型的socket支持这种操作,另外协议类型也需要支持.
	- `MSG_DONTROUTE` 用于路由程序的诊断.
  * `sendmsg`系统调用使用`msghdr`结构体来减少直接传递的参数.`msg_iov`和`msg_iovlen`字段用于指定0个或更多用户存储待发数据的缓冲区.`msg_iov`指向`iovec`的数组,`msg_iovlen`应该设置为这个数组的尺寸.在每一个`iovec`结构中,`iov_base`字段指定了一个存储空间,`iov_len`给出了空间的大小(以字节为单位,可以为0).在每一个`msg_iov`指定的存储空间中的数据将被轮流发送.

**返回值:**
  * 成功时返回已经发送的字节数,否则返回-1并且将错误码保存到全局变量`errno`.

**错误码:**
  * [EACCES] socket未设置`SO_BROADCAST`选项但使用了广播地址作为目标地址.
  * [EAGAIN] socket被标记为非阻塞,但请求操作仍然被阻塞.
  * [EBADF] 指定了非法的描述符.
  * [ECONNRESET] 连接被对端强制关闭.
  * [EFAULT] 指定了非法的用户空间地址.
  * [EHOSTUNREACH] 目标地址是不可达的主机.
  * [EINTR] 在数据被发出之前信号中断了系统调用.
  * [EMSGIZE] socket要求消息消息自动发出,但是消息的长度使自动发送失败.
  * [ENETDOWN] 用于与目的通信的本地网络接口关闭.
  * [ENOBUFS] 系统无法分配内部缓冲区.
  * [ENOBUFS] 网络接口的输出队列已满.
  * [ENOTSOCK] 参数`socket`不是一个socket.
  * [EOPNOTSUPP] 参数`socket`不支持参数`flags`中的值.
  * [EPIPE] socket在写入时被关闭,或者socket的连接不再存在,后者,调用线程会产生`SIGPIPE`信号.
  * [EAFNOSUPPORT] 指定地址空间中的地址不可用.
  * [EDESTADDRREQ] socket不是连接模式,且对端地址未指定.
  * [EISCONN] 指定了对端地址,但socket以处于连接状态.
  * [ENOENT] 路径不存在或指定了空字符串.
  * [ENOMEM] 内存不足.
  * [ENOTCONN] socket是连接模式,但未处于连接状态.
  * [ENOTDIR] socket地址中的路径名中的路径不是一个目录.
  * [EINVAL] `iov_len`的值溢出.
  * [EMSGSIZE] socket要求消息被自动发送,但消息的大小使自动发送失败,或则`msg_iovlen`小于等于0或大于`IOV_MAX`.

##recv recvfrom recvmsg##
**原型:**
```
ssize_t recv(int socket, void *buffer, size_t length, int flags);
ssize_t recvfrom(int socket, void *restrict buffer, size_t length, 
                 int flags, struct sockaddr *restrict address, 
				 socklen_t *restrict address_len);
ssize_t recvmsg(int socket, struct msghdr *message, int flags);
```

**描述:**
  * 系统调用`recvfrom`和`recvmsg`用来从socket接收消息,可以用于接收面向连接或无连接的数据.
  * 如果参数`address`不是一个空指针,并且socket是无连接的,函数会填入消息的源地址.参数`address_len`既是入参也是出参,应当初始化为参数`address`所指空间的大小,并在函数返回时修改为实际地址所占用的空间.
  * 函数`recv`通常只能用于已经建立了连接的socket,不同于`recvfrom`,函数不需要传入地址指针.由于功能重复,未来可能不在支持.
  * 所有函数在成功时返回消息的长度.如果消息的长度超过提供的缓冲区,超出的字节可能被丢弃,具体要看socket的类型.
  * 对于阻塞类型的socket,如果没有收到数据,函数等待消息到达.对于非阻塞类型的socket,函数返回-1,并且设置外部变量`errno`为`EAGAIN`.函数可能返回任何可用的数据,取决于请求的数量,而不是等待完整请求的数量被接收到.这一行为受socket级别选项`SO_RCVLOWAT`和`SO_RCVTIMEO`的影响.
  * select系统调用可以用来判定是否有更多数据到达.
  * 当没有数据可以接收并且对端的关闭事件,函数返回0.
  * 参数`flag`可以是以下值的组合:
    - `MSG_OOB` 处理不会在普通数据流中出现的带外数据.一些协议会在数据队列头部添加加急数据,这些协议不能使用此标识.
	- `MSG_PEEK` 窥视进入的消息,接收操作不会从接收队列移除数据,后面的接收操作将返回相同的数据.
	- `MSG_WAITALL` 等待完整的请求或错误,接收操作阻塞直到请求完整.然而,在信号被捕获,发生错误或连接释放,或者下一个数据是不同的类型时,调用仍然有可能返回不完整的请求
  * 系统调用`recvmsg`使用一个`msghdr`结构体来减少直接提供参数的个数,结构体定义如下:
```
struct msghdr {
             void            *msg_name;      /* optional address */
             socklen_t       msg_namelen;    /* size of address */
             struct          iovec *msg_iov; /* scatter/gather array */
             int             msg_iovlen;     /* # elements in msg_iov */
             void            *msg_control;   /* ancillary data, see below */
             socklen_t       msg_controllen; /* ancillary data buffer len */
             int             msg_flags;      /* flags on received message */
     };
```

**返回值:**
  * 成功返回接收的字节数,否则返回-1.
  * 对于TCP,返回0意味着对端单方面关闭了连接.

**错误码:**
  * [EAGAIN] socket标记为非阻塞单接收操作仍然会阻塞,或者设置了接收超时时间并且达到超时时间.
  * [EBADF] 参数`socket`不是合法的描述符.
  * [ECONNRESET] 连接被对端关闭.
  * [EFAULT] 缓冲区指针不再进程地址空间内.
  * [EINTR] 接收过程被信号中断.
  * [EINVAL] 设置了`MSG_OOB`但是没有带外数据可用.
  * [ENOBUFS] 缓冲区分配失败.
  * [ENOTCONN] 连接未建立.
  * [ENOTSOCK] 参数`socket`不是一个socket描述符.
  * [EOPNOTSUPP] socket的协议类型不支持参数`flags`的值.
  * [ETIMEDOUT] 连接超时.
  * [EINVAL] `iov_len`的值溢出.
  * [EMSGSIZE] 参数`msghdr`的成员`msg_iovlen`的值非法.
  * [ENOMEM] 内存不足.
