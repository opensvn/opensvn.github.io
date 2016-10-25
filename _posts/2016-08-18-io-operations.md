---
layout: post
title: Asio.Cookbook 第2章 IO操作
date: 2016-08-18 08:55:42
categories: C++
excerpt: 介绍IO操作相关的问题
---

## 介绍

IO操作是任何分布式应用的网络基础设施的关键操作。它们直接参与数据交换的过程。输入操作用来接收数据，输出操作用来发送数据。

### IO缓冲区

网络编程都是关于通过计算机网络进行进程间通信。像其他类型的IO操作一样，网络IO操作涉及使用内存缓冲区。

### 同步和异步IO操作

Boost.Asio支持两种类型的IO操作：同步和异步。同步操作阻塞当前线程的执行直到操作完成。异步操作在初始化时关联一个回调函数，当操作完成时，Boost.Asio会调用回调函数。

在一个或多个异步操作初始化后，程序使用一个线程来运行Boost.Asio库。Boost.Asio库使用这个线程运行事件循环并调用回调函数通知异步操作完成，异步操作的结果作为参数传递给回调函数。

### 附加操作

1. 取消异步操作。
取消之前初始化的异步操作的能力很重要。它允许程序声明之前初始化的异步操作不再有效。

2. shutdown套接字。
shutdown套接字很有用，如果需要通知另一个程序整个报文已发送。

3. close套接字。

## 使用固定长度IO缓冲区

固定长度的IO缓冲区通常用在报文长度已知的情况下。在Boost.Asio里面，固定长度IO缓冲区由asio::mutable_buffer或asio::const_buffer表示。asio::mutable_buffer表示可写缓冲区，asio::const_buffer表示只读缓冲区。

但是asio::mutable_buffer和asio::const_buffer并不能直接在Boost.Asio的IO函数中直接使用。相反MutableBufferSequence和ConstBufferSequence概念被引入。MutableBufferSequence指定一个对象表示asio::mutable_buffer对象的集合。相应地，ConstBufferSequence指定一个对象表示asio::const_buffer对象的集合。

asio::buffer()自由函数拥有28个重载形式，接收多种缓冲区表示形式并返回一个asio::mutable_buffers_1或asio::const_buffers_1对象。如果传给asio::buffer()函数的参数是只读类型，则返回asio::const_buffers_1对象，反之返回asio::mutable_buffers_1对象。asio::mutable_buffers_1和asio::const_buffers_1是asio::mutable_buffer和asio::const_buffer相应的适配。

### 为输出操作准备缓冲区

1. 分配一个缓冲区。
2. 将输出数据填入缓冲区。
3. 将缓冲区表示为满足ConstBufferSequence需求的对象。
4. 缓冲区已经可以用在Boost.Asio的输出函数或方法。

{% highlight cpp %}
std::string buf = "Hello";
asio::const_buffer_1 output_buf = asio::buffer(buf);
// output_buf can be used in Boost.Asio output operations
{% endhighlight %}
为了更好的理解为什么需要将缓冲区表示成满足ConstBufferSequence需求，可以看一下send()的声明：

{% highlight cpp %}
template <typename ConstBufferSequence>
std::size_t send(const ConstBufferSequence& buffers);
{% endhighlight %}
为了使用send()发送string对象，我们可以这样做：

{% highlight cpp %}
asio::const_buffer asio_buf(buf.c_str(), buf.size());
std::vector<asio::const_buffer> buffers_sequence;
buffers_sequence.push_back(asio_buf);
{% endhighlight %}

但是这样没有使用asio::buffer()方便。

### 为输入操作准备缓冲区

1. 分配一个缓冲区。缓冲区的大小必须足以保存接收到的数据。
2. 将缓冲区表示为满足MutableBufferSequence需求的对象。
3. 缓冲区已经可以用在Boost.Asio的输入函数或方法。

{% highlight cpp %}
const size_t BUF_SIZE_BYTES = 20;
std::unique_ptr<char[]> buf(new char[BUF_SIZE_BYTES]);
asio::mutable_buffers_1 input_buf =
    asio::buffer(static_cast<void*>(buf.get()), BUF_SIZE_BYTES);
// input_buf can be used in Boost.Asio input operations
{% endhighlight %}

> 缓冲区所有权
asio::mutable_buffer，asio::const_buffer，asio::mutable_buffers_1，asio::const_buffers_1等并不拥有原始缓冲区的所有权，它们只提供访问缓冲区的接口，不控制其生命周期。

## 使用可扩展面向流的IO缓冲区

可扩展缓冲区是当新数据写入时可以动态增长的缓冲区。它们经常用于从套接字读取未知大小的报文。

可扩展面向流的缓冲区在Boost.Asio中由asio::streambuf类表示：

{% highlight cpp %}
typedef basic_streambuf<> streambuf;
{% endhighlight %}

asio::basic_streambuf<>类继承自std::streambuf。

{% highlight cpp %}
asio::streambuf buf;
std::ostream output(&buf);

// Writing the message to the stream-based buffer.
output << "Message";

// Instantiate an input stream which uses our stream buffer.
std::istream input(&buf);

std::string message;
std::getline(input, message);
{% endhighlight %}

## 同步写TCP套接字

写套接字最基本的方法是使用Boost.Asio中的asio::ip::tcp::socket类的write_some()方法：

{% highlight cpp %}
template<typename ConstBufferSequence>
std::size_t write_some(const ConstBufferSequence& buffers);
{% endhighlight %}

如果方法成功执行，返回写入的字节数。需要强调的是这个方法可能不会发送参数指定的所有数据，它只保证如果错误不发生至少发送一个字节。这意味着为了发送缓冲区中所有数据需要调用write_some()多次。

1. 在客户端程序中，分配，打开并连接一个主动TCP套接字。在服务端程序中，通过接收器套接字接收一个主动TCP套接字的连接请求。
2. 分配缓冲区并将要写到套接字的数据填充进缓冲区。
3. 在一个循环中调用套接字的write_some()方法多次直到缓冲区中的数据都发送完。

{% highlight cpp %}
void writeToSocket(asio::ip::tcp::socket& sock) {
    std::string buf = "Hello";
    std::size_t total_bytes_written = 0;
    while (total_bytes_written != buf.length()) {
        total_bytes_written += sock.write_some(
            asio::buffer(buf.c_str() + total_bytes_written,
                buf.length() - total_bytes_written));
    }
}
{% endhighlight %}

### 替代品——send()方法

asio::ip::tcp::socket类包含另外一个同步写数据的方法send()。它有3种重载形式。其中一个和write_some()方法一样，有一样的签名并提供同样的功能。

第二个重载函数接收一个额外的参数：

{% highlight cpp %}
template<typename ConstBufferSequence>
std::size_t send(const ConstBufferSequence& buffers,
    socket_base::message_flags flags);
{% endhighlight %}

第三个重载函数和第二个一样，但是它不抛出异常，而是接收一个boost::system::error_code的参数。

使用write_some()方法向套接字写数据看起来非常复杂，因为必须使用一个循环，并且每一次迭代都需要正确构造缓冲区并跟踪已写的数据。幸运的是，Boost.Asio提供一个自由函数asio::write()简化向套接字写数据。asio::write()有8种重载形式。

{% highlight cpp %}
template<typename SyncWriteStream, typename ConstBufferSequence>
std::size_t write(SyncWriteStream& s, const ConstBufferSequence& buffers);

void WriteToSocketEnhanced(asio::ip::tcp::socket& sock) {
    std::string buf = "hello";
    asio::write(sock, asio::buffer(buf));
}
{% endhighlight %}

## 同步读TCP套接字

由Boost.Asio提供的从套接字读取数据最基本的方法是asio::ip::tcp::socket类的read_some()方法。

{% highlight cpp %}
template<typename MutableBufferSequence>
std::size_t read_some(const MutableBufferSequence& buffers);
{% endhighlight %}

需要注意的是没有办法控制read_some()将会读取多少数据。它只保证如果错误不发生至少读取一个字节。通常情况下，为了从套接字读取某一数量的数据需要调用read_some()多次。

1. 在客户端程序中，分配，打开并连接一个主动TCP套接字。在服务端程序中，通过接收器套接字接收一个主动TCP套接字的连接请求。
2. 分配足够大小的缓冲区使得能够装下预期的报文。
3. 在一个循环中调用套接字的read_some()方法多次直到读完数据。

{% highlight cpp %}
std::string readFromSocket(asio::ip::tcp::socket& sock) {
    const unsigned char MESSAGE_SIZE = 7;
    char buf[MESSAGE_SIZE];
    std::size_t total_bytes_read = 0;
    while (total_bytes_read != MESSAGE_SIZE) {
        total_bytes_read += sock.read_some(
            asio::buffer(buf + total_bytes_read,
                MESSAGE_SIZE - total_bytes_read));
    }
    return std::string(buf, total_bytes_read);
}
{% endhighlight %}

### 替代品——receive()方法

asio::ip::tcp::socket类包含另外一个同步读数据的方法receive()。它有3种重载形式。其中一个和read_some()方法一样，有一样的签名并提供同样的功能。

第二个重载函数接收一个额外的参数：

{% highlight cpp %}
template<typename MutableBufferSequence>
std::size_t receive(const MutableBufferSequence& buffers,
    socket_base::message_flags flags);
{% endhighlight %}

第三个重载函数和第二个一样，但是它不抛出异常，而是接收一个boost::system::error_code的参数。

使用read_some()方法向套接字写数据看起来非常复杂，因为必须使用一个循环，并且每一次迭代都需要正确构造缓冲区并跟踪已写的数据。幸运的是，Boost.Asio提供一组自由函数简化向套接字写数据。有3种读函数，每个有7种重载形式。

### asio::read()函数

{% highlight cpp %}
template<typename SyncReadStream, typename MutableBufferSequence>
std::size_t read(SyncReadStream& s, const MutableBufferSequence& buffers)

std::string readFromSocketEnhanced(asio::ip::tcp::socket& sock) {
    const unsigned char MESSAGE_SIZE = 7;
    char buf[MESSAGE_SIZE];
    asio::read(sock, asio::buffer(buf, MESSAGE_SIZE));
    return std::string(buf, MESSAGE_SIZE);
}
{% endhighlight %}

### read_until()函数

asio::read_until()函数从套接字读取数据直到遇到指定的模式，它有8种重载形式。

{% highlight cpp %}
template<typename SyncReadStream, typename Allocator>
std::size_t read_until(
    SyncReadStream& s,
    boost::asio::basic_streambuf<Allocator>& b,
    char delim);
{% endhighlight %}

asio::read_until从套接字s读取数据到缓冲区b直到遇到指定的字符delim。需要注意的是asio::read_until内部使用read_some()方法读取数据的。当函数返回时，缓冲区b中可能包含在delim后的数据。也就是说程序员需要处理这种情况。

{% highlight cpp %}
std::string readFromSocketDelim(asio::ip::tcp::socket& sock) {
    asio::streambuf buf;
    asio::read_until(sock, buf, '\n');
    std::string message;
    std::istream input_stream(&buf);
    std::getline(input_stream, message);
    return message;
}
{% endhighlight %}

### read_at()函数

asio::read_at()函数从特定位置开始读数据，参考Boost文档。

## 异步写TCP套接字

Boost.Asio提供的异步写数据最基本的工具是asio::ip::tcp::socket类的async_write_some()方法。

{% highlight cpp %}
template<typename ConstBufferSequence, typename WriteHandler>
void async_write_some(const ConstBufferSequence& buffers,
    WriteHandler handler);
{% endhighlight %}

async_write_some()方法初始化一个写操作并立即返回。第一个参数包含要写到套接字的数据，第二个参数是个回调函数，当初始化操作完成后由Boost.Asio调用。

回调应该具有下述的签名：

{% highlight cpp %}
void write_handler(const boost::system::error_code& ec,
    std::size_t bytes_transferred);
{% endhighlight %}

async_write_some()方法保证如果不发生错误，至少写一个字节数据。这意味着通常情况下，为了将所有数据写到套接字，需要调用async_write_some()多次。

1. 定义一个数据结构，包含指向套接字的指针，一个缓冲区和一个用来统计已写数据的变量
2. 定义一个回调函数
3. 在客户端程序中，分配，打开并连接一个主动TCP套接字。在服务端程序中，通过接收器套接字接收一个主动TCP套接字的连接请求。
4. 分配一个缓冲区，并填充要写到套接字的数据
5. 调用async_write_some()方法初始化一个写操作，指定步骤2的函数为回调函数
6. 调用asio::io_service类的run()方法
7. 在回调中，增加已写的数据。如果已写数据小于需要写的总数据，再初始化一个异步写操作

{% highlight cpp %}
struct Session {
    std::shared_ptr<asio::ip::tcp::socket> sock;
    std::string buf;
    std::size_t total_bytes_written;
};

void callback(const boost::system::error_code& ec,
    std::size_t bytes_transferred, std::shared_ptr<Session> s) {
    if (ec != 0) {
        std::cout << "Error code = " << ec.value()
        << ". Message: " << ec.message();
        return;
    }
    
    s->total_bytes_written += bytes_transferred;
    if (s->total_bytes_written == s->buf.length()) {
        return;
    }
    
    s->sock->async_write_some(
        asio::buffer(s->buf.c_str() + s->total_bytes_written,
            s->buf.length() - s->total_bytes_written),
        std::bind(callback, std::placeholders::_1,
            std::placeholders::_2, s));
}

void writeToSocket(std::shared_ptr<asio::ip::tcp::socket> sock) {
    std::shared_ptr<Session> s(new Session);
    
    s->buf = std::string("Hello");
    s->total_bytes_written = 0;
    s->sock = sock;
    
    s->sock->async_write_some(asio::buffer(s->buf),
        std::bind(callback, std::placeholders::_1,
            std::placeholders::_2, s));
}

int main() {
    std::string raw_ip_address = "127.0.0.1";
    unsigned short port_num = 3333;
    
    try {
        asio::ip::tcp::endpoint ep(
            asio::ip::address::from_string(raw_ip_address), port_num);
        asio::io_service ios;
        std::shared_ptr<asio::ip::tcp::socket> sock(
            new asio::ip::tcp::socket(ios, ep.protocol()));
        
        sock->connect(ep);
        writeToSocket(sock);
        
        ios.run();
    } catch (system::system_error &e) {
        return e.code().value();
    }
    
    return 0;
}
{% endhighlight %}

Boost.Asio提供一个自由函数asio::async_write()方法可以更方便地异步写到套接字：

{% highlight cpp %}
template<typename AsyncWriteStream, typename ConstBufferSequence,
    typename WriteHandler>
void async_write(AsyncWriteStream& s,
    const ConstBufferSequence& buffers, WriteHandler handler);
{% endhighlight %}

asio::async_write()函数是通过多次调用async_write_some()方法实现的。

## 异步读TCP套接字

