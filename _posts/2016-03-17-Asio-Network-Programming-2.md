---
layout: post
title: Boost.Asio网络编程 第2章
date: 2016-03-17 10:55:52
categories: C++
excerpt: 这一章展示了使用Asio编写网络程序必须知道的内容
---

## Boost.Asio命名空间 ##

Boost.Asio所有东西都放在boost::asio命名空间或者子命名空间：

* boost::asio：核心类和函数放在这里。重要的类有io_service和streambuf。重要的自由函数，比如read，read_at，read_util，它们相应的异步版本，以及同步写和异步写函数。
* boost::asio::ip：网络部分属于这里。重要的类有address，endpoint，tcp，udp，icmp。重要的自由函数connect和async_connect。注意boost::asio::ip::tcp::socket只是一个boost::asio::ip::tcp中的typedef。
* boost::asio::error：包含调用I/O例程的错误码。
* boost::asio::ssl：包含处理SSL的类。
* boost::asio::local：包含POSIX相关的类。
* boost::asio::windows：包含Windows相关的类。

## IP地址 ##

为了处理IP地址，Boost.Asio提供ip::address，ip::address_v4和ip::address_v6类。以下是一些最重要的函数：

* ip::address(v4_or_v6_address)：这个函数转换一个v4或v6地址为ip::address。
* ip::address::from_string(str)：从一个IPv4或IPv6地址字符串创建地址。
* ip::address::to_string()：返回一个友好可读的地址字符串。
* ip::address_v4::broadcast([addr, mask])：创建一个广播地址。
* ip::address_v4::any()：返回一个代表任何地址的地址。
* ip::address_v4::loopback()，ip_address_v6::loopback()：返回回路地址
* ip::host_name()：返回当前主机名。

你可能最常用到ip::address::from_string：
{% highlight cpp %}
ip::address addr = ip::address::from_string("127.0.0.1");
{% endhighlight %}

## 端点 ##

端点是你要连接的地址和端口。不同类型有自己的endpoint类，比如ip::tcp::endpoint，ip::udp::endpoint和ip::icmp::endpoint。

如果你想连接本机80端口，这样：
{% highlight cpp %}
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 80);
{% endhighlight %}

创建端点有3种方法：

* endpoint()：默认构造函数，有时候可以用于UDP/ICMP套接字。
* endpoint(protocol, port)：通常用于服务端套接字接受新连接。
* endpoint(addr, port)：创建一个指定地址和端口的端点。

下面是例子：
{% highlight cpp %}
ip::tcp::endpoint ep1;
ip::tcp::endpoint ep2(ip::tcp::v4(), 80);
ip::tcp::endpoint ep3(ip::address::from_string("127.0.0.1"), 80);
{% endhighlight %}

如果想要连接到一个主机名，可以这样做：
{% highlight cpp %}
// outputs "87.248.122.122"
io_service service;
ip::tcp::resolver resolver(service);
ip::tcp::resolver::query query("www.yahoo.com", "80");
ip::tcp::resolver::iterator iter = resolver.resolve(query);
ip::tcp::endpoint ep = *iter;
std::cout << ep.address().to_string() << std::endl;
{% endhighlight %}

如果resolve()函数成功，它将返回至少一个入口。你可以使用第一个，也可以遍历所有的。

## 套接字 ##

Boost.Asio有3中socket类：ip::tcp，ip::udp和ip::icmp，并且可以扩展。你可以创建自己的socket类。

你可以认为ip::tcp，ip::udp和ip::icmp类是占位符，它们提供其他类和函数的简便访问：

* ip::tcp::socket, ip::tcp::acceptor, ip::tcp::endpoint, ip::tcp::resolver, ip::tcp::iostream
* ip::udp::socket, ip::udp::endpoint, ip::udp::resolver
* ip::icmp::socket, ip::icmp::endpoint, ip::icmp::resolver

socket类创建相应的套接字，创建时需要传递io_service实例：

{% highlight cpp %}
io_service service;
ip::udp::socket sock(service)
sock.set_option(ip::udp::socket::reuse_address(true));
{% endhighlight %}

每一个socket名字是一个typedef：

* ip::tcp::socket = basic_stream_socket\<tcp\>
* ip::udp::socket = basic_datagram_socket\<udp\>
* ip::icmp::socket = basic_raw_socket\<icmp\>

### 同步错误码 ###

所有的同步函数重载了抛出异常或返回错误码的版本，像下面的例子：
{% highlight cpp %}
sync_func(arg1, arg2 ... argN); // throws
boost::system::error_code ec;
sync_func(arg1 arg2, ..., argN, ec); // returns error code
{% endhighlight %}

### Socket成员函数 ###

不是所有的函数对每一种类型的socket都可用。注意所有的异步函数立刻返回，同步函数只有在操作完成才会返回。

#### 连接相关的函数 ####

* assign(protocol, socket)：这个函数将一个原始套接字赋给一个socket实例。使用它处理遗留代码。
* open(protocol)：这个函数使用指定IP协议（v4或v6）打开一个socket。这个函数主要用于UDP/ICMP套接字，或者服务器套接字。
* bind(endpoint)：绑定到指定端点。
* connect(endpoint)：同步连接到指定端点。
* async_connect(endpoint)：异步连接到指定端点。
* is_open()：如果socket已打开返回true。
* close()：关闭套接字。此套接字任何异步操作立刻取消并以error::operation_aborted错误码结束。
* shutdown(type_of_shutdown)：从现在开始禁止send，receive操作。
* cancel()：取消此套接字所有异步操作。所有异步操作立刻以error::operation_aborted错误码结束。

例子如下：
{% highlight cpp %}
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 80);
ip::tcp::socket sock(service);
sock.open(ip::tcp::v4());
sock.connect(ep);
sock.write_some(buffer("GET /index.html\r\n"));
char buff[1024]; sock.read_some(buffer(buff,1024));
sock.shutdown(ip::tcp::socket::shutdown_receive);
sock.close();
{% endhighlight %}

#### 读写函数 ####

对于所有异步读写函数，其处理函数的签名为：
{% highlight cpp %}
void handler(const boost::system::error_code &e, size_t bytes)
{% endhighlight %}

* async_receive(buffer, [flags,] handler)：从套接字开始异步receive操作。
* async_read_some(buffer, handler)：等价于async_receive(buffer, handler)。
* async_receive_from(buffer, endpoint[, flags], handler)：从指定端点开始异步receive操作。
* async_send(buffer[, flags], handler)：开始异步发送buffer中的数据。
* async_write_some(buffer, handler)：等价于async_send(buffer, handler)。
* async_send_to(buffer, endpoint, handler)：开始异步发送buffer中的数据到指定端点。
* receive(buffer[, flags])：同步读取数据到指定buffer。这个函数阻塞知道收到数据或一个错误发生。
* read_some(buffer)：等价于receive(buffer)。
* receive_from(buffer, endpoint[, flags])：同步地从指定端点读取数据到指定buffer。这个函数阻塞直到受到数据或错误发生。
* send(buffer[,  flags])：同步发送buffer中的数据。函数阻塞直到受到数据或发生错误。
* write_some(buffer)：等价于send(buffer)。
* send_to(buffer, endpoint[, flags])：同步发送buffer中的数据到指定端点。函数阻塞直到受到数据或发生错误。
* avaliable()：返回可以同步读取的字节数，不用阻塞。

flags的默认值是0，但可以是下列的组合：

* ip::socket_type::socket::message_peek：这个标志只是偷看消息。它返回这个消息，但是下一次调用会重读这个消息。
* ip::socket_type::socket::message_out_of_band：这个标志处理out-of-band（OOB）数据。OOB数据是标记为比普通数据更重要的数据。
* ip::socket_type::socket::message_do_not_route：这个标志指定消息应该不使用路由表发送。
* ip::socket_type::socket::message_end_of_record：这个标志指示数据标志一个记录的结尾。在Windows下不支持。

如果你使用下面这段代码，你最可能使用message_peek：
{% highlight cpp %}
char buff[1024];
sock.receive(buffer(buff), ip::tcp::socket::message_peek);
memset(buff,1024, 0);
// re-reads what was previously read
sock.receive(buffer(buff));
{% endhighlight %}

同步读写一个TCP套接字：
{% highlight cpp %}
ip::tcp::endpoint ep( ip::address::from_string("127.0.0.1"), 80);
ip::tcp::socket sock(service);
sock.connect(ep);
sock.write_some(buffer("GET /index.html\r\n"));
std::cout << "bytes available " << sock.available() << std::endl;
char buff[512];
size_t read = sock.read_some(buffer(buff));
{% endhighlight %}

同步读写一个UDP套接字：
{% highlight cpp %}
ip::udp::socket sock(service);
sock.open(ip::udp::v4());
ip::udp::endpoint receiver_ep("87.248.112.181", 80);
sock.send_to(buffer("testing\n"), receiver_ep);
char buff[512];
ip::udp::endpoint sender_ep;
sock.receive_from(buffer(buff), sender_ep);
{% endhighlight %}

注意为了使用receive_from从一个UDP套接字读取数据，你需要一个默认构造的端点。

异步读取一个UDP服务器套接字：
{% highlight cpp %}
using namespace boost::asio;
io_service service;
ip::udp::socket sock(service);
boost::asio::ip::udp::endpoint sender_ep;
char buff[512];
void on_read(const boost::system::error_code & err, std::size_t read_bytes) {
    std::cout << "read " << read_bytes << std::endl;
    sock.async_receive_from(buffer(buff), sender_ep, on_read);
}
int main(int argc, char* argv[]) {
    ip::udp::endpoint ep(ip::address::from_string("127.0.0.1"), 8001);
    sock.open(ep.protocol());
    sock.set_option(boost::asio::ip::udp::socket::reuse_address(true));
    sock.bind(ep);
    sock.async_receive_from(buffer(buff,512), sender_ep, on_read);
    service.run();
}
{% endhighlight %}

#### 套接字控制 ####

这些函数处理高级套接字选项：

* get_io_service()：返回构造时传递的io_service实例。
* get_option(option)：返回一个套接字选项。
* set_option(option)：设置套接字选项
* io_control(cmd)：在套接字上执行一个I/O命令。

以下是你可以读取或设置的套接字选项：

| 名字 | 描述 | 类型 |
|---|---|---|
| broadcast | 如果真，允许广播消息 | bool |
| debug | 如果真，使socket-level调试生效 | bool |
| enable_connection_aborted | 如果真，报告在accept()时连接被中止 | bool |
| receive_buffer_size | 套接字的接收缓冲区大小 | int |
| receive_low_watermark | 提供处理套接字输入的最小字节数 | int |
| reuse_address | 如果真，套接字可以绑定一个已经在使用的地址 | bool |
| send_buffer_size | 套接字发送缓冲区大小 | int |
| send_low_watermark | 提供处理套接字输出的最小字节数 | int |
| ip::v6_only | 如果真，只允许IPv6通信 | bool |

每一个名字代表内部socket的一个typedef或一个类。以下是例子：
{% highlight cpp %}
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 80);
ip::tcp::socket sock(service);
sock.connect(ep);
// TCP socket can reuse address
ip::tcp::socket::reuse_address ra(true);
sock.set_option(ra);
// get sock receive buffer size
ip::tcp::socket::receive_buffer_size rbs;
sock.get_option(rbs);
std::cout << rbs.value() << std::endl;
// set sock's buffer size to 8192
ip::tcp::socket::send_buffer_size sbs(8192);
sock.set_option(sbs);
{% endhighlight %}

v#### TCP vs UDP vs ICMP ####

如果一个成员函数不在下表，说明它对所有socket类可用：

| 名字 | TCP | UDP | ICMP |
|---|---|---|---|
| async_read_some | Yes | - | - |
| async_receive_from | - | Yes | Yes |
| async_write_some | Yes | - | - |
| async_send_to | - | Yes | Yes |
| read_some | Yes | - | - |
| receive_from | - | Yes | Yes |
| write_some | Yes | - | - |
| send_to | - | Yes | Yes |

#### 其它函数 ####

* local_endpoint()：返回本地地址。
* remote_endpoint()：返回远程地址。
* native_handle()：返回原始套接字的句柄。你只有在想要调用原始套接字API的时候需要这个函数。
* non_blocking()：如果套接字非阻塞返回true，否则false。
* native_non_blocking()：和non_blocking一样，但是它在原始套接字上调用原始的套接字API。
* at_mark()：如果套接字即将读取OOB数据，返回true。很少使用。

### 其它考虑 ###

最后需要注意的是，一个套接字实例不能被复制，因为复制构造函数和赋值操作符无法访问。如果你想要创建拷贝，使用共享指针：
{% highlight cpp %}
typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
socket_ptr sock1(new ip::tcp::socket(service));
socket_ptr sock2(sock1); // ok
socket_ptr sock3;
sock3 = sock1; // ok
{% endhighlight %}

#### Socket缓冲区 ####

当读取或写入一个套接字，需要一个缓冲区，它将保存进来和流出的数据。缓冲区的内存必须比I/O操作更长久。你必须保证I/O操作持续期间它不被释放或超出范围。

这个问题有下面几种解决方法：

* 使用全局缓冲区。
* 创建一个缓冲区，当操作完成时销毁它。
* 使用一个连接对象维护套接字和额外的数据，比如缓冲区。

#### 缓存包装器函数 ####

任何时候我们需要一个缓冲区用于读写操作时，用buffer()函数包装真实的缓冲区对象。它包装任何缓冲区为一个类，允许Boost.Asio函数迭代访问缓冲区。比如你使用下面的代码：
{% highlight cpp %}
sock.async_receive(some_buffer, on_read);
{% endhighlight %}

some_buffer实例需要满足一些条件，即ConstBufferSequence或MutableBufferSequence。使用buffer()函数就可以满足这些条件。

简单地说，你可以用buffer()函数包装下面的对象：

* char[] const数组
* void*指针和大小
* std::string
* POD[] const数组（POD代表plain old data，即构造函数和析构函数不用做任何事）
* std::vector\<POD\>
* boost::array\<POD\>
* std::array\<POD\>

## 读/写/连接自由函数 ##

Boost.Asio提供自由函数处理I/O。

### 连接函数 ###

连接socket到一个端点：

* connect(socket, begin[, end][, condition])：同步连接，尝试begin和end之间每一个端点。begin迭代器是socket_type::resolver::query调用返回的结果。你可以提供一个条件函数在每次连接尝试前调用。它的签名是Iterator connect_condition(const boost::system::error_code &ec, Iterator next)。
* async_connect(socket, begin[, end][, condition], handler)：执行异步连接，最后调用完成处理函数。处理函数的签名是void handler(const boost::system::error_code &ec, Iterator iterator)。

示例如下：
{% highlight cpp %}
using namespace boost::asio::ip;
tcp::resolver resolver(service);
tcp::resolver::iterator iter = resolver.resolve(
	tcp::resolver::query("www.yahoo.com",
	"80"));
tcp::socket sock(service);
connect(sock, iter);
{% endhighlight %}

一个主机名可以解析为多个地址，因此connect和async_connect将你从尝试每一个地址的负担中释放出来。

### 读/写函数 ###

* async_read(stream, buffer[, completion], handler)：从stream异步读取数据。完成后调用处理函数，其签名为void handler(const boost::system::error_code &ec, size_t bytes)。你可以指定一个完成函数。每一次成功读后都调用一次完成函数，并告诉Boost.Asio异步读操作是否完成，如果没有继续读。完成函数的签名为size_t completion(const boost::system::error_code &ec, size_t bytes_transfered)。当完成函数返回0时，我们认为读操作完成。如果返回非0值，它指示下一次async_read_some操作读取的最大字节数。
* async_write(stream, buffer[, completion], handler)：异步写数据到stream。参数的意义和async_read类似。
* read(stream, buffer[, completion])：从stream同步读取数据。参数的意义和async_read类似。
* write(stream, buffer[, completion])：同步写数据到stream。参数的意义和async_read类似。

每一个读或写操作在下面这些条件发生时结束：

* 提供的缓冲区已满（对于读）或在缓冲区的数据都被写完（对于写）
* 完成函数返回0，如果你提供了这样一个函数
* 一个错误发生

下面的代码会异步读知道发现'\n'：
{% highlight cpp %}
io_service service;
ip::tcp::socket sock(service);
char buff[512];
int offset = 0;
size_t up_to_enter(const boost::system::error_code &, size_t bytes) {
    for ( size_t i = 0; i < bytes; ++i)
        if (buff[i + offset] == '\n')
        return 0;
    return 1;
}
void on_read(const boost::system::error_code &, size_t) {}
...
async_read(sock, buffer(buff), up_to_enter, on_read);
{% endhighlight %}

Boost.Asio提供一些助手完成函数：

* transfer_at_least(n)
* transfer_exactly(n)
* transfer_all()

例子如下：
{% highlight cpp %}
char buff[512];
void on_read(const boost::system::error_code &, size_t) {}
// read exactly 32 bytes
async_read(sock, buffer(buff), transfer_exactly(32), on_read);
{% endhighlight %}

最后4个函数，使用继承std::streambuf的stream_buffer函数而不是通常的buffer。STL stream和stream_buffer非常灵活。
{% highlight cpp %}
io_service service;
void on_read(streambuf& buf, const boost::system::error_code &, size_t) {
    std::istream in(&buf);
    std::string line;
    std::getline(in, line);
    std::cout << "first line: " << line << std::endl;
}
int main(int argc, char* argv[]) {
    HANDLE file = ::CreateFile("readme.txt", GENERIC_READ, 0, 0,
        OPEN_ALWAYS, FILE_ATTRIBUTE_NORMAL | FILE_FLAG_OVERLAPPED, 0);
    windows::stream_handle h(service, file);
    streambuf buf;
    async_read(h, buf, transfer_exactly(256),
        boost::bind(on_read,boost::ref(buf),_1,_2));
    service.run();
}
{% endhighlight %}

#### read_until/async_read_util函数 ####

这些函数一直读直到一个条件满足：

* async_read_until(stream, stream_buffer, delim, handler)：开始一个异步读操作。当遇到一个delim时，读操作停止。delim可以是任意一个字符，std::string或boost::regex。处理函数的签名为void handler(const boost::system::error_code &ec, size_t bytes)。
* async_read_until(stream, stream_buffer, completion, handler)：这个函数和前一个一样，只是将delim替换为完成函数，其签名是pair\<iterator, bool\> completion(iterator begin, iterator end)。迭代器参数是buffers_iterator\<streambuf::const_buffers_type\>。你需要记住的是迭代器类型是随机访问迭代器。返回true说明读操作应该停止。
* read_until(stream, stream_buffer, delim)：同步读，参数意义与async_read_until一样。
* read_until(stream, stream_buffer, completion)：同步读，参数意义与async_read_until一样。

下面的代码将读到一个标点符号为止：
{% highlight cpp %}
typedef buffers_iterator<streambuf::const_buffers_type> iterator;
std::pair<iterator, bool> match_punct(iterator begin, iterator end) {
    while (begin != end)
        if (std::ispunct(*begin))
            return std::make_pair(begin,true);
    return std::make_pair(end,false);
}
void on_read(const boost::system::error_code &, size_t) {}
...
streambuf buf;
async_read_until(sock, buf, match_punct, on_read);
{% endhighlight %}

如果想读到一个空格，修改最后一行：
{% highlight cpp %}
async_read_until(sock, buff, ' ', on_read);
{% endhighlight %}

#### *_at函数 ####

这些函数在一个stream上做随机读写操作。你指定读写操作从哪开始（offset）：

* async_read_at(stream, offset, buffer[, completion], handler)：在指定流上从offset开始异步读操作。处理函数的签名是void handler(const boost::system::error_code &ec, size_t bytes)。缓冲区可以是通常的buffer()包装器或一个streambuf函数。如果指定了一个完成函数，它在每次成功读操作后被调用，并告诉Boost.Asio操作async_read_at是否完成，其签名为size_t completion(const boost::system::error_code &ec, size_t bytes)。当完成函数返回0时，我们认为读操作完成；如果返回非0值，它指示下一次async_read_some_at调用要读取的最大字节数。
* async_write_at(stream, offset, buffer[, completion], handler)：开始一个异步写操作。参数的意思和async_read_at一样。
* read_at(stream, offset, buffer[, completion])：同步读，参数的意思和async_read_at一样。
* write_at(stream, offset, buffer[, completion])：同步读，参数的意思和async_read_at一样。

这些函数不处理socket，它们处理随机访问流。
{% highlight cpp %}
io_service service;
int main(int argc, char* argv[]) {
    HANDLE file = ::CreateFile("readme.txt", GENERIC_READ, 0, 0,
        OPEN_ALWAYS, FILE_ATTRIBUTE_NORMAL | FILE_FLAG_OVERLAPPED,
        0);
    windows::random_access_handle h(service, file);
    streambuf buf;
    read_at(h, 256, buf, transfer_exactly(128));
    std::istream in(&buf);
    std::string line;
    std::getline(in, line);
    std::cout << "first line: " << line << std::endl;
}
{% endhighlight %}

# 异步编程 #

这部分将深入钻研进行异步编程会碰到的一些问题。

## 异步的需求 ##

尽管异步编程更难，你可能还是会选择它。比如说编写需要处理非常多并行客户端的服务器。并行客户端越多，异步编程比同步编程越容易。

## 异步run()，run_one()，poll()，poll_one() ##

为了实现监听循环，io_service类提供4个函数，run()，run_one()，poll()和poll_one()。

### 永久运行 ###

run()会一直运行，只要有追加的操作执行或者你手动调用io_service::stop()。为了保持io_service实例运行，通常添加一个或多个异步操作，并当它们执行时，保持继续添加异步操作：
{% highlight cpp %}
using namespace boost::asio;
io_service service;
ip::tcp::socket sock(service);
char buff_read[1024], buff_write[1024] = "ok";
void on_read(const boost::system::error_code &err, std::size_t bytes)
;
void on_write(const boost::system::error_code &err, std::size_t bytes)
{
    sock.async_read_some(buffer(buff_read), on_read);
}
void on_read(const boost::system::error_code &err, std::size_t bytes)
{
    // ... process the read ...
    sock.async_write_some(buffer(buff_write,3), on_write);
}
void on_connect(const boost::system::error_code &err) {
    sock.async_read_some(buffer(buff_read), on_read);
}
int main(int argc, char* argv[]) {
    ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
    sock.async_connect(ep, on_connect);
    service.run();
}
{% endhighlight %}

### run_one()，poll()，poll_one()函数 ###

run_one()函数会执行和分配最多一个异步操作：

* 如果没有追加操作，函数立刻返回0
* 如果有追加操作，函数阻塞知道第一个操作执行，并返回1

考虑下述等价代码：
{% highlight cpp %}
io_service service;
service.run(); // OR
while (!service.stopped()) service.run_once();
{% endhighlight %}

你可以使用run_once()开始一个异步操作，然后等待它完成：
{% highlight cpp %}
io_service service;
bool write_complete = false;
void on_write(const boost::system::error_code & err, size_t bytes)
{ write_complete = true; }
...
std::string data = "login ok";
write_complete = false;
async_write(sock, buffer(data), on_write);
do service.run_once() while (!write_complete);
{% endhighlight %}

poll_one函数运行最多一次准备运行的追加操作，非阻塞：

* 如果有至少一个准备运行的非阻塞追加操作，poll_one运行它并返回1
* 否则，立刻返回0

准备运行的非阻塞追加操作，通常意味着任何：

* 一个定时器超时了，且它的async_wait回调函数需要被调用。
* 一个I/O操作完成了（比如async_read），且其回调函数需要被调用。
* 一个之前添加到io_services队列的自定义的回调函数。

poll()函数运行所有添加的操作，不用阻塞。下面代码等价：
{% highlight cpp %}
io_service service;
service.poll(); // OR
while (service.poll_one());
{% endhighlight %}

所有之前的函数失败时抛出boost::system::system_error异常。这不应该发生，这里抛出的错误通常是致命的。每一个函数也有不抛异常而返回错误码的重载版本。
{% highlight cpp %}
io_service service;
boost::system::error_code err = 0;
service.run(err);
if (err) std::cout << "Error " << err << std::endl;
{% endhighlight %}

## 异步工作 ##

异步工作不仅仅是异步接受客户端连接，异步读取或写到套接字。它包含任何可以异步执行的操作。

默认情况下，你不知道异步处理函数的调用顺序。你可以使用service.post()抛出自定义的函数使其可以异步调用：
{% highlight cpp %}
using namespace boost::asio;
io_service service;
void func(int i) {
    std::cout << "func called, i= " << i << std::endl;
}
void worker_thread() {
    service.run();
}
int main(int argc, char* argv[]) {
    for (int i = 0; i < 10; ++i)
        service.post(boost::bind(func, i));
    boost::thread_group threads;
    for (int i = 0; i < 3; ++i)
        threads.create_thread(worker_thread);
    // wait for all threads to be created
    boost::this_thread::sleep(boost::posix_time::millisec(500));
    threads.join_all();
}
{% endhighlight %}

有的时候你想要某些异步处理函数按顺序调用。你可以使用io_service::strand，它将顺序调用你的异步处理函数。
{% highlight cpp %}
void func(int i) {
    std::cout << "func called, i= " << i << "/"
    << boost::this_thread::get_id() << std::endl;
}
void worker_thread() {
    service.run();
}
int main(int argc, char* argv[])
{
    io_service::strand strand_one(service), strand_two(service);
    for ( int i = 0; i < 5; ++i)
        service.post(strand_one.wrap( boost::bind(func, i)));
    for ( int i = 5; i < 10; ++i)
        service.post(strand_two.wrap( boost::bind(func, i)));
    boost::thread_group threads;
    for ( int i = 0; i < 3; ++i)
        threads.create_thread(worker_thread);
    // wait for all threads to be created
    boost::this_thread::sleep( boost::posix_time::millisec(500));
    threads.join_all();
}
{% endhighlight %}

## 异步post() vs dispatch() vs wrap() ##

Boost.Asio提供3种方法添加你的函数异步调用：

* service.post(handler)：这个函数保证在请求io_service添加处理函数后立刻返回。处理函数会在之后被某个调用了service.run()的线程调用。
* service.dispatch(handler)：请求io_service调用指定的处理函数，同时如果当前线程调用了service.run()，它可以在函数内执行处理函数。
* service.wrap(handler)：这个函数创建一个包装函数。当包装函数被调用时，会调用service.dispatch(handler)。

看看dispatch如何影响输出：
{% highlight cpp %}
void func(int i) {
    std::cout << "func called, i= " << i << std::endl;
}
void run_dispatch_and_post() {
    for ( int i = 0; i < 10; i += 2) {
        service.dispatch(boost::bind(func, i));
        service.post(boost::bind(func, i + 1));
    }
}
int main(int argc, char* argv[]) {
    service.post(run_dispatch_and_post);
    service.run();
}
{% endhighlight %}

wrap()函数返回一个函数对象，以供将来使用：
{% highlight cpp %}
void dispatched_func_1() {
    std::cout << "dispatched 1" << std::endl;
}
void dispatched_func_2() {
    std::cout << "dispatched 2" << std::endl;
}
void test(boost::function<void()> func) {
    std::cout << "test" << std::endl;
    service.dispatch(dispatched_func_1);
    func();
}
void service_run() {
    service.run();
}
int main(int argc, char* argv[]) {
    test(service.wrap(dispatched_func_2));
    boost::thread th(service_run);
    boost::this_thread::sleep( boost::posix_time::millisec(500));
    th.join();
}
{% endhighlight %}

io_service::strand类也包含成员函数poll()，dispatch()和wrap()。其意义和io_service的一样。然而大多数时间你只会使用io_service::strand::wrap()作为io_service::poll()或io_service::dispatch()的参数。

# 保持活着 #

当使用套接字缓冲区时，你可以用一个buffer实例度过一个异步调用。我们可以使用同样的原理创建一个类，内部保存套接字和读/写缓冲区。然后对于所有异步调用，传递一个共享指针给boost::bind函数：
{% highlight cpp %}
struct connection : boost::enable_shared_from_this<connection> {
    typedef boost::system::error_code error_code;
    typedef boost::shared_ptr<connection> ptr;
    connection() : sock_(service), started_(true) {}
    void start(ip::tcp::endpoint ep) {
        sock_.async_connect(ep,
                            boost::bind(&connection::on_connect, shared_from_this(),
                                        _1));
    }
    void stop() {
        if ( !started_) return;
        started_ = false;
        sock_.close();
    }
    bool started() { return started_; }
private:
    void on_connect(const error_code & err) {
        // here you decide what to do with the connection: read or write
        if (!err) do_read();
        else stop();
    }
    void on_read(const error_code & err, size_t bytes) {
        if (!started()) return;
        std::string msg(read_buffer_, bytes);
        if (msg == "can_login") do_write("access_data");
        else if (msg.find("data ") == 0) process_data(msg);
        else if (msg == "login_fail") stop();
    }
    void on_write(const error_code & err, size_t bytes) {
        do_read();
    }
    void do_read() {
        sock_.async_read_some(buffer(read_buffer_),
                              boost::bind(&connection::on_read, shared_from_this(),
                                          _1, _2));
    }
    void do_write(const std::string & msg) {
        if ( !started() ) return;
        // note: in case you want to send several messages before
        // doing another async_read, you'll need several write buffers!
        std::copy(msg.begin(), msg.end(), write_buffer_);
        sock_.async_write_some(buffer(write_buffer_, msg.size()),
                               boost::bind(&connection::on_write, shared_from_this(),
                                           _1, _2));
    }
    void process_data(const std::string & msg) {
        // process what comes from server, and then perform another write
    }
private:
    ip::tcp::socket sock_;
    enum { max_msg = 1024 };
    char read_buffer_[max_msg];
    char write_buffer_[max_msg];
    bool started_;
};
int main(int argc, char* argv[]) {
    ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"),
                          8001);
    connection::ptr(new connection)->start(ep);
}
{% endhighlight %}

在所有的异步调用中，我们传递一个boost::bind函数对象作为参数。这个函数对象内部保存一个共享指针指向connection实例。只要还有追加的异步操作，Boost.Asio将保存一个boost::bind函数对象，它又保存一个共享指针指向connection实例，因此保持连接活着。
