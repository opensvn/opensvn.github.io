---
layout: post
title: Boost.Asio网络编程 第1章
date: 2016-03-13 10:55:46
categories: C++
excerpt: Boost.Asio简介
---

# 什么是Boost.Asio #

简单来说，Boost.Asio是一个跨平台的C++库，主要是为了网络编程和一些其它低级输入/输出编程。

Boost.Asio成功地抽象出输入和输出的概念，不仅仅是网络，还有COM串行端口，文件等。在此之上，你可以同步或异步进行输入或输出编程：

{% highlight cpp %}
read(stream, buffer [, extra options])
async_read(stream, buffer [, extra options], handler)
write(stream, buffer [, extra options])
async_write(stream, buffer [, extra options], handler)
{% endhighlight %}

正如你在上一个代码片段看到，这些函数接受一个stream实例，它可以是任何东西（不仅仅是socket，只要我们能够读取或写入它）。

# 依赖 #

Boost.Asio依赖以下库：

* Boost.System：这个库为Boost库提供操作系统支持。
* Boost.Regex：可选，如果你使用带有boost::regex参数的read_until()或async_read_until()的重载版本。
* Boost.DateTime：可选，如果你使用Boost.Asio计时器。
* OpenSSL：可选，如果你决定使用Boost.Asio提供的SSL支持。

# 编译Boost.Asio #

Boost.Asio是只有头文件的库。然而，取决于你的编译器和程序的大小，你可以选择作为源文件嵌入Boost.Asio。可以使用下述方式减少编译次数：

* 只在一个文件#include <boost/asio/impl/src.hpp>（如果使用SSL，也要包含<boost/asio/ssl/impl/src.hpp>）
* 在所有文件中使用#define BOOST_ASIO_SEPARATE_COMPILATION

## 重要的宏 ##

BOOST_ASIO_DISABLE_THREADS如果被定义，它会禁止Boost.Asio中线程支持，不管Boost是否编译了线程支持。

# 同步VS异步 #

首先，异步编程与同步编程非常不同。在同步编程中，顺序执行操作，比如从socket读取请求，然后把回复写到socket。每一个操作都是阻塞的。因为操作是阻塞的，为了在读或写一个socket时不打断主程序，通常需要创建一个或多个线程处理socket的输入/输出。因此同步server/client通常是多线程的。

相反，异步编程是事件驱动的。你开始一个操作，但是不知道它什么时候结束；你提供一个回调函数，它会在操作结束时被API调用，连同操作结果。因此，在异步编程中，你没必要需要一个线程以上。

以下是一个基本的同步客户端例子：

{% highlight cpp %}
using boost::asio;
io_service service;
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
ip::tcp::socket sock(service);
sock.connect(ep);
{% endhighlight %}

首先，程序需要至少一个io_service实例。Boost.Asio使用io_service与操作系统输入/输出服务对话。通常一个io_service实例就足够了。

以下是一个简单的同步服务器例子：

{% highlight cpp %}
typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
io_service service;
ip::tcp::endpoint ep(ip::tcp::v4(), 2001)); // listen on 2001
ip::tcp::acceptor acc(service, ep);
while (true) {
    socket_ptr sock(new ip::tcp::socket(service));
    acc.accept(*sock);
    boost::thread(boost::bind(client_session, sock));
}
void client_session(socket_ptr sock) {
    while (true) {
        char data[512];
        size_t len = sock->read_some(buffer(data));
        if (len > 0)
            write(*sock, buffer("ok", 2));
    }
}
{% endhighlight %}

创建一个异步客户端，类似下面这样：

{% highlight cpp %}
using boost::asio;
io_service service;
ip::tcp::endpoint ep(ip::address::from_string("127.0.0.1"), 2001);
ip::tcp::socket sock(service);
sock.async_connect(ep, connect_handler);
service.run();
void connect_handler(const boost::system::error_code & ec) {
    // here we know we connected successfully
    // if ec indicates success
}
{% endhighlight %}

注意service.run()循环会一直运行，只要还有异步操作加入。在前一个例子中，只有一个async_connect异步操作。在此之后，service.run()退出。

每一个异步操作都有一个完成处理器，一个在操作完成时被调用的函数。

以下是一个基本异步服务器：

{% highlight cpp %}
using boost::asio;
typedef boost::shared_ptr<ip::tcp::socket> socket_ptr;
io_service service;
ip::tcp::endpoint ep(ip::tcp::v4(), 2001)); // listen on 2001
ip::tcp::acceptor acc(service, ep);
socket_ptr sock(new ip::tcp::socket(service));
start_accept(sock);
service.run();
void start_accept(socket_ptr sock) {
    acc.async_accept(*sock, boost::bind(handle_accept, sock, _1));
}
void handle_accept(socket_ptr sock, const boost::system::error_code &err) {
    if (err) return;
    // at this point, you can read/write to the socket
    socket_ptr sock(new ip::tcp::socket(service));
    start_accept(sock);
}
{% endhighlight %}

handle_accept里面，在socket使用后，创建一个新的socket，并再次调用start_accept()，它添加另一个async_accept操作，保持service.run()忙碌下去。

# 异常VS错误码 #

Boost.Asio允许异常或错误码。所有同步函数重载了抛出异常或返回错误码的版本。这些函数抛出boost::system::system_error错误。

{% highlight cpp %}
using boost::asio;
ip::tcp::endpoint ep;
ip::tcp::socket sock(service);
sock.connect(ep); // Line 1
boost::system::error_code err;
sock.connect(ep, err); // Line 2
{% endhighlight %}

在前面的代码中，sock.connect(ep)在发生错误时抛出异常，而sock.connect(ep, err)返回错误码。

所有异步函数返回一个错误码，你可以在回调中检查它。异步函数从不抛出异常，因为这不合理，谁来捕捉异常？

所有Boost.Asio错误码在命名空间boost::asio::error。

# Boost.Asio中的线程 #

当提到Boost.Asio中的线程，我们会讨论：

* io_service：io_service类是线程安全的。多个线程可以调用io_service::run()。大多数时候你可能从单个线程调用io_service::run()，它一直阻塞直到所有异步操作完成。然而你可以从多个线程里面调用io_service::run()，这会阻塞所有调用了io_service::run()的线程。所有回调都将在线程自己的上下文环境中被调用。
* socket：socket类不是线程安全的。因此你应该避免在一个线程中读，然后写到另一个线程中去。
* utility：utility类不是线程安全的，通常在多个线程中使用它不合理。它们中的大多数都是短时间使用，然后被回收。

Boost.Asio库自己可以使用多个非用户的线程，它保证在这些线程里面不会调用任何你的代码。这意味着回调只会被调用了io_service::run()的线程调用。

# 不仅仅是网络 #

除了网络，Boost.Asio提供其他输入/输出设施。

Boost.Asio允许等待信号，比如SIGTERM，SIGINT，SIGSEGV等：

{% highlight cpp %}
void signal_handler(const boost::system::error_code & err, int signal)
{
    // log this, and terminate application
}
boost::asio::signal_set sig(service, SIGINT, SIGTERM);
sig.async_wait(signal_handler);
{% endhighlight %}

使用Boost.Asio可以很容易地连接串行端口：

{% highlight cpp %}
io_service service;
serial_port sp(service, "COM7");
// serial_port sp(service, "/dev/ttyS0");
serial_port::baud_rate rate(9600);
sp.set_option(rate);
char data[512];
read(sp, buffer(data, 512));
{% endhighlight %}

Boost.Asio也允许连接Windows文件：

{% highlight cpp %}
HANDLE h = ::OpenFile(...);
windows::stream_handle sh(service, h);
char data[512];
read(h, buffer(data, 512));
{% endhighlight %}

你可以同样操作POSIX文件描述符，比如管道，标准I/O，各种设备：

{% highlight cpp %}
posix::stream_descriptor sd_in(service, ::dup(STDIN_FILENO));
char data[512];
read(sd_in, buffer(data, 512));
{% endhighlight %}

# 计时器 #

一些I/O操作可以有一个完成的最后期限，这个概念只能用到异步操作。

{% highlight cpp %}
bool read = false;
void deadline_handler(const boost::system::error_code &) {
    std::cout << (read ? "read successfully" : "read failed") <<
    std::endl;
}
void read_handler(const boost::system::error_code &) {
    read = true;
}
ip::tcp::socket sock(service);
…
read = false;
char data[512];
sock.async_read_some(buffer(data, 512));
deadline_timer t(service, boost::posix_time::milliseconds(100));
t.async_wait(&deadline_handler);
service.run();
{% endhighlight %}

Boost.Asio同样允许同步计时器，但它们通常等价于sleep操作。以下2个等价：

{% highlight cpp %}
boost::this_thread::sleep(500);
deadline_timer t(service, boost::posix_time::milliseconds(500));
t.wait();
{% endhighlight %}

# io_service类 #

io_service是Boost.Asio最重要的类，它跟操作系统打交道，等待一个异步操作结束并调用相应的回调函数。

你可以以几种方式使用io_service：

* 单线程，1个io_service和单处理线程：
{% highlight cpp %}
io_service service_;
// all the socket operations are handled by service_
ip::tcp::socket sock1(service_);
// all the socket operations are handled by service_
ip::tcp::socket sock2(service_);
sock1.async_connect(ep, connect_handler);
sock2.async_connect(ep, connect_handler);
deadline_timer t(service_, boost::posix_time::seconds(5));
t.async_wait(timeout_handler);
service_.run();
{% endhighlight %}
* 多线程，单个io_service和多个处理线程：
{% highlight cpp %}
io_service service_;
ip::tcp::socket sock1(service_);
ip::tcp::socket sock2(service_);
sock1.async_connect(ep, connect_handler);
sock2.async_connect(ep, connect_handler);
deadline_timer t(service_, boost::posix_time::seconds(5));
t.async_wait(timeout_handler);
for ( int i = 0; i < 5; ++i)
    boost::thread(run_service);
void run_service() {
    service_.run();
}
{% endhighlight %}
* 多个线程，多个io_service和多个处理线程：
{% highlight cpp %}
io_service service_[2];
ip::tcp::socket sock1(service_[0]);
ip::tcp::socket sock2(service_[1]);
sock1.async_connect(ep, connect_handler);
sock2.async_connect(ep, connect_handler);
deadline_timer t(service_[0], boost::posix_time::seconds(5));
t.async_wait(timeout_handler);
for (int i = 0; i < 2; ++i)
    boost::thread(boost::bind(run_service, i));
void run_service(int idx) {
    service_[idx].run();
}
{% endhighlight %}

首先，需要注意在一个线程里面不能有多个io_service实例。以下代码毫无意义：

{% highlight cpp %}
for ( int i = 0; i < 2; ++i)
    service_[i].run();
{% endhighlight %}

以下是从前面例子中你应该学到的：

* 第一种情况对最基本的程序有用。如果多个处理函数需要同时调用，将遇到瓶颈，因为它们以顺序方式被调用。如果一个处理函数花太长时间结束，所有后续处理函数都将等待。
* 第二种情况适用于大多数程序。如果多个处理函数被同时调用，它们会在自己的线程被调用。唯一的瓶颈是如果所有处理线程都在忙碌，而新的处理函数被调用。
* 第三种情况最复杂最灵活。你应该在第二种情况不够的时候使用它。你可以认为每一个处理线程（运行io_service::run()的线程）有自己的select/epoll循环。

最后记住如果没有更多的操作需要处理，run()将会终止。如果想要run()继续运行，你必须把更多工作给它。一种方法是在处理函数中开始另一个异步操作。另一种方法是模拟一些工作给它：

{% highlight cpp %}
typedef boost::shared_ptr<io_service::work> work_ptr;
work_ptr dummy_work(new io_service::work(service_));
{% endhighlight %}

上述代码将使得service_.run()不会停止除非你使用service_.stop()或dummy_work.reset(0)销毁dummy_work。

