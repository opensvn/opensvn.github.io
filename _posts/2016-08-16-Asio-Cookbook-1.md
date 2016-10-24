---
layout: post
title: Asio.Cookbook 第1章 基础
date: 2016-08-16 08:45:19
categories: C++
excerpt: 基础介绍
---

## 介绍

TCP协议是具有下列特性的传输层协议：

- 它是可靠的。这意味着TCP协议保证报文以正确的顺序传输，或者通知报文没有传输成功。TCP协议包含错误处理机制。
- 它假定建立逻辑连接。在一个程序通过TCP协议与另一个程序通信之前，它必须根据标准通过交换服务报文建立一个逻辑连接。
- 它假定点对点通信模型。也就是在单个连接上只有两个程序可以通信，不支持多播消息。
- 它是面向数据流的。这意味着一个程序发送给另一个程序的数据会被TCP协议解释为字节流。

UDP协议是一个与TCP协议不同的传输层协议，它具有下列特性：

- 它是不可靠的。这意味着如果一个发送者通过UDP协议发送一个报文，不保证报文会被发送。UDP协议不会尝试检查或修复任何错误。开发者负责所有的错误处理。
- 它是无连接的。这意味着在程序通信之前不需要建立连接。
- 它支持一对一或一对多通信模型。UDP协议支持多播消息。
- 它是面向数据报的。这意味着UDP协议将数据解释为特定大小的报文并且会尝试将报文作为一个整体发送。数据报文要么被作为一个整体发送，要么当发送失败时根本不会发送。

因为UDP协议是不可靠的，它通常被用在可靠的局域网。如果想在因特网使用UDP协议，开发者必须实现错误处理机制。当需要在因特网传输时，因为可靠性，TCP协议通常是最好的选择。

## 创建一个端点

端点服务两个目标：

- 客户端程序使用端点指定一个想要通信的特定的服务端程序。
- 服务端程序使用端点指定一个本地的IP地址和端口，在此之上接收来自客户端的报文。

### 客户端创建端点

1. 获取服务器程序的IP地址和端口。IP地址应该是点号隔开的字符串（IPv4）或者16进制的字符串（IPv6）。
2. 使用一个asio::ip::address对象代表IP地址。
3. 用address对象和端口初始化一个asio::ip::tcp::endpoint对象。

```c++
#include <boost/asio.hpp>

std::string raw_ip_address = "127.0.0.1";
unsigned short port_num = 3333;

boost::system::error_code ec;
asio::ip::address ip_address = asio::ip::address::from_string(raw_ip_address, ec);
if (ec.value() != 0) {
    // handle error
}

asio::ip::tcp::endpoint ep(ip_address, port_num);
```

### 服务端创建端点

1. 获取一个服务器将要监听请求的端口。
2. 创建一个特殊的asio::ip::address对象表示这个服务器所有可用的IP地址。
3. 使用address对象和端口初始化一个asio::ip::tcp::endpoint对象。

```c++
#include <boost/asio.hpp>

unsigned short port_num = 3333;

// IPv6
asio::ip::address ip_address = asio::ip::address_v6::any();
asio::ip::tcp::endpoint ep(ip_address, port_num);
```

Boost.Asio提供3个类用来表示IP地址：

- asio::ip::address_v4，代表IPv4地址
- asio::ip::address_v6，代表IPv6地址
- asio::ip::address，代表IPv4或IPv6地址

from_string()是asio::ip::address的静态方法，它有4个重载形式：

```c++
static asio::ip::address from_string(
    const char* str);
    
static asio::ip::address from_string(
    const char* str,
    boost::system::error_code& ec);
    
static asio::ip::address from_string(
    const std::string& str);
    
static asio::ip::address from_string(
    const std::string& str,
    boost::system::error_code& ec);
```

为了表示主机可用的所有IP地址，asio::ip::address_v4和asio::ip::address_v6提供了一个静态方法any()。

asio::ip::tcp::endpoint其实是basic_endpoint<>模板的一个特例。

```c++
typedef basic_endpoint<tcp> endpoint;
```

同样asio::ip::udp::endpoint也是basic_endpoint<>模板的一个特例。

```c++
typedef basic_endpoint<udp> endpoint;
```

## 创建一个主动套接字

基本上，套接字有两种类型。主动套接字用来发起一个连接建立过程。被动套接字用来被动地等待传入的连接请求。

1. 创建一个asio::io_service实例，或者使用之前创建的。
2. 创建一个代表传输层协议（TCP或UDP）和下层IP协议（IPv4或IPv6）的类对象。
3. 创建一个代表套接字的对象，传递asio::io_service给它的构造函数。
4. 调用open()方法，传递协议参数给它。

```c++
asio_io_service ios;
asio::ip::tcp protocol = asio::ip::tcp::v4();
asio::ip::tcp::socket sock(ios);
boost::system::error_code ec;
sock.open(protocol, ec);
if (ec.value() != 0) {
    // handle error
    std::cerr << "Error code = " << ec.value()
        << " Message: " << ec.message();
    return ec.value();
}
```

asio::ip::tcp类代表TCP协议，它没有提供功能，而是像一个数据结构包含一组描述协议的值。

asio::ip::tcp没有公开的构造函数。相反它提供了2个静态方法（v4()和v6()）返回该类对象。

```c++
class tcp {
public:
    typedef basic_endpoint<tcp> endpoint;
    typedef basic_stream_socket<tcp> socket;
    typedef basic_socket_acceptor<tcp> acceptor;
};
```

使用UDP协议创建主动套接字和TCP类似：

```c++
asio::io_service ios;
asio::ip::udp protocol = asio::ip::udp::v6();
try {
    asio::ip::udp::socket sock(ios, protocol);
} catch (boost::system::system_error& e) {
    std::cerr << e.code() << " " << e.what();
}
```

## 创建一个被动套接字

一个被动套接字或者接收器套接字是一种用来等待建立连接请求的套接字。它有2个重要的暗示：

- 被动套接字只用在服务端程序或者既是服务端又是客户端的程序。
- 被动套接字只被TCP协议定义。

在Boost.Asio里面一个被动套接字由asio::ip::tcp::acceptor类来表示。

1. 创建一个asio::io_service实例或使用之前创建的。
2. 创建一个asio::ip::tcp类对象代表TCP协议和所需底层的IP协议（IPv4和IPv6）。
3. 创建一个asio::ip::tcp::acceptor类对象表示接收器套接字，传递asio::io_service类对象给它的构造函数。
4. 调用接收器套接字的open()函数，传递第2步创建的对象给它。

```c++
asio::io_service ios;
asio::ip::tcp protocol = asio::ip::tcp::v6();
asio::ip::tcp::acceptor acceptor(ios);
boost::system::error_code ec;
acceptor.open(protocol, ec);
if (ec.value() != 0) {
    std::cerr << ec.value() << " " << ec.message();
}
```

## 解析域名

域名系统是一个分布式的命名系统，它允许将人类友好的名字和计算机网络中的设备关联起来。一个域名是一个表示计算机网络设备名字的字符串。更确切地说，一个域名是一个或多个IP地址的别名。

域名系统作为一个分布式的数据库，存储了域名到相应IP地址的映射，并提供一个接口，允许查询一个特定域名所映射的IP地址。将域名转换为相应IP地址的过程称为域名解析。

1. 获取DNS域名和端口，并用字符串表示。
2. 创建一个asio::io_service实例或使用之前创建的。
3. 创建一个resolver::query类对象。
4. 创建一个适合必要协议的DNS解析对象。
5. 调用解析器的resolve()方法，用query对象作为参数。

```c++
std::string host = "samplehost.com";
std::string port_num = "3333";

asio::io_service ios;

asio::ip::tcp::resolver::query query(host, port_num,
    asio::ip::tcp::resolver::query::numeric_service);
    
asio::ip::tcp::resolver resolver(ios);

boost::system::error_code ec;
asio::ip::tcp::resolver::iterator it = 
    resolver.resolve(query, ec);
if (ec != 0) {
    // handle error
}
```

asio::ip::tcp::resolver::iterator类是一个迭代器，它指向解析结果的第一个元素，元素类型是asio::ip::basic_resolver_entry&lt;tcp&gt;。每一个结果包含一个endpoint对象，可以通过asio::ip::basic_resolver_entry&lt;tcp&gt;::endpoint()获取。

```c++
asio::ip::tcp::resolver::iterator it =
    resolver.resolve(query, ec);
asio::ip::tcp::resolver::iterator it_end;
for (; it != it_end; ++it) {
    asio::ip::tcp::endpoint = it->endpoint();
}
```

当域名被映射到多个IP地址，并且有些是IPv4，有些是IPv6时，则结果集合中包含2种endpoint。

通过UDP协议解析域名也是类似的。

```c++
std::string host = "www.samplehost.com";
std::string port_num = "3333";

asio::io_service ios;

asio::ip::udp::resolver::query query(host, port_num,
    asio::ip::udp::resolver::query::numeric_service);

asio::ip::udp::resolver resolver(ios);

boost::system::error_code ec;
asio::ip::udp::resolver::iterator it = 
    resolver.resolve(query, ec);
if (ec != 0) {
    // handle error
}
```

## 绑定套接字到端点

有些操作隐式绑定未绑定套接字。比如主动套接字连接服务器，客户端程序不需要特定的端点与服务器通信，因此它通常将选择绑定IP地址和端口的权利委托给操作系统。服务端程序通常需要显示将被动套接字绑定到特定的端点。

1. 获取服务器监听连接请求的端口。
2. 创建一个表示主机所有可用IP地址的端点。
3. 创建并打开一个接收器套接字。
4. 调用接收器套接字的bind()方法，将断点作为参数传给它。

```c++
unsigned short port_num = 3333;
asio::ip::tcp::endpoint ep(asio::ip::address_v4::any(), port_num);
asio::ip::tcp::acceptor acceptor(ios, ep.protocol());

boost::system::error_code ec;
acceptor.bind(ep, ec);
if (ec != 0) {
    // handle error
}
```

UDP服务器不建立连接，而且使用主动套接字等待到来的请求。

```c++
unsigned short port_num = 3333;
asio::ip::udp::endpoint ep(asio::ip::address_v4::any(), port_num);
asio::io_service ios;
asio::ip::udp::socket sock(ios, ep.protocol());

boost::system::error_code ec;
sock.bind(ep, ec);
if (ec != 0) {
    // handle error
}
```

## 连接一个套接字

1. 获取服务器程序的IP地址和端口。
2. 用IP地址和端口创建一个endpoint对象。
3. 创建并打开一个主动套接字。
4. 用endpoint对象作参数调用connect()方法。
5. 如果方法成功，套接字被认为已连接并可以用来发送和接收来自服务器的数据。

```c++
std::string raw_ip_address = "127.0.0.1";
unsigned short port_num = 3333;
try {
    asio::ip::tcp::endpoint ep(
        asio::ip::address::from_string(raw_ip_address), port_num);
    asio::io_service ios;
    asio::ip::tcp::socket sock(ios, ep.protocol());
    sock.connect(ep);
    // At this point socket 'sock' is connected to the server
    // connect()方法会将客户端IP地址和操作系统选好端口绑定到'sock'
} catch (boost::system::system_error& e) {
    std::cerr << e.code() << " " << e.what();
}
```

自由函数asio::connect()接收一个主动套接字对象和一个asio::ip::tcp::resolver::iterator对象作为参数，遍历所有端点。

1. 获取DNS域名和端口，并用字符串表示。
2. 使用asio::ip::tcp::resolver类解析域名。
3. 创建一个主动套接字，不打开它。
4. 调用asio::connect()函数。

```c++
std::string host = "samplehost.com";
std::string port_num = "3333";

asio::ip::tcp::resolver::query query(host, port_num
    asio::ip::tcp::resolver::query::numeric_service);
    
asio::io_service ios;

asio::ip::tcp::resolver resolver(ios);

try {
    asio::ip::tcp::resolver::iterator it = 
        resolver.resolve(query);
    
    asio::ip::tcp::socket sock(ios);

    asio::connect(sock, it);
    // At this point 'sock' is connected to the server
} catch (boost::system::system_error& e) {
    std::cerr << e.code() << " " << e.what();
}
```

## 接受连接

1. 获取服务器监听连接请求的端口。
2. 创建一个端点。
3. 初始化并打开一个接收器套接字。
4. 绑定接收器套接字到端点。
5. 调用listen()方法开始监听端点上到来的连接请求。
6. 初始化一个主动套接字对象。
7. 当准备好处理连接请求时，用主动套接字作为参数，调用接收器套接字的accept()方法。
8. 如果调用成功，主动套接字和客户端程序就连上了，可以用来通信了。

```c++
const int BACKLOG_SIZE = 30;
unsigned short port_num = 3333;
asio::ip::tcp::endpoint ep(asio::ip::address_v4::any(), port_num);

asio::io_service ios;

try {
    asio::ip::tcp::acceptor acceptor(ios, ep.protocol());
    acceptor.bind(ep);

    acceptor.listen(BACKLOG_SIZE);

    asio::ip::tcp::socket sock(ios);
    acceptor.accept(sock);
    // At this point 'sock' is connected to the client
} catch (boost::system::system_error& e) {
    std::cerr << e.code() << " " << e.what();
}
```

注意UDP服务器不使用接收器套接字，因为UDP协议不需要建立连接。相反，主动套接字被使用来绑定到一个端点并监听到来的报文，而且同一个主动套接字也用来通信。