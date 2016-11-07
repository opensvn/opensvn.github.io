---
layout: post
title: 安装完Ubuntu后需要做的事
date: 2016-10-26 16:03:15
categories: Ubuntu
excerpt: 安装Ubuntu后，需要安装的开发工具（C++相关）。
---

## 我会安装的工具

**开发相关**

* g++，必须的，C++编译器
* clang，据说是比gcc更好用的编译器，emacs的一些补全工具会用这个做后端
* emacs，编辑器之神，谁用谁知道，入门门槛稍微高了一点，我现在还是菜鸟hoho
* git，连接github开源代码
* vim，自从加入emacs阵营后，这个就用的少了
* subversion，公司代码还是用svn管理
* cmake，使用cmake自动生成Makefile，编译工程很方便
* global，emacs里面用来生成GTAGS，利于阅读代码
* meld，文件比较工具，支持整个目录的比较，以及版本控制系统，比如svn，git
* git cola，一个非常好用的git客户端工具
* LiteIDE，Go语言IDE
* CLion，JetBrains出品的C++IDE，很少用，最大的缺点耗内存，其他功能都不错
* curl，命令行抓取工具
* CodeBlocks，C++IDE，公司目前用的，比较少用，只用来编译发布。下面的操作安装最新的16.01版本。

{% highlight bash %}
sudo add-apt-repository ppa:damien-moore/codeblocks-stable
sudo apt-get update
sudo apt-get install codeblocks
{% endhighlight %}

**日常使用**

* ubuntu-tweak，该工具已经停止更新，16.04已经不能安装了，我是14.04升级到16.04，保留了该工具
* cryptomator，本地目录加密工具，加密指定目录所有文件。

{% highlight bash %}
sudo add-apt-repository ppa:sebastian-stenzel/cryptomator
sudo apt-get update
sudo apt-get install cryptomator
{% endhighlight %}

* keepassx，本地密码管理软件，再也不用担心忘记密码了，只需记住一个主密码即可，保险一点可以再加一个key文件。这里讲一个配合cryptomator和云盘（推荐坚果云）的靠谱方法：

    1. 首先用keepassx生成一个密码库的key文件
    2. 将key文件放入cryptomator加密的目录（称为保险箱吧）中
    3. 将保险箱同步到云盘
    4. 密码库文件不用放入保险箱，同步到云盘。

    这样就可以在不同机器上使用和修改密码库文件，云盘会自动同步。因为key文件放在保险箱里面，非常安全。没有key文件和主密码是打不开密码库的。当然理论上存在保险箱被破解的可能，如果你觉得不放心，可以将key文件保存在自己认为安全的地方（千万不能丢）。

* 坚果云（Nutstore），[下载地址](https://www.jianguoyun.com/s/downloads/linux)
    
{% highlight bash %}
sudo apt install default-jre-headless
sudo dpkg -i nautilus_nutstore_amd64.deb
{% endhighlight %}

* [蓝灯](https://getlantern.org/)，[下载地址](https://github.com/getlantern/lantern-binaries)

* 搜狗输入法，[下载地址](http://pinyin.sogou.com/linux/?r=pinyin)。安装完成后，设置系统输入法，通过**系统设置>语言支持>键盘输入方式系统**，选择**fcitx**，注销并重新登录即可

{% highlight bash %}
sudo apt install fcitx fcitx-libs fcitx-libs-qt
sudo dpkg -i sogoupinyin_2.1.0.0082_amd64.deb
{% endhighlight %}

* oh-my-zsh，首先要安装zsh。

{% highlight bash %}
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
{% endhighlight %}

## 你自己的Github-通过坚果云管理代码（[参见](http://blog.jianguoyun.com/?p=321)）

切换到用来管理代码的目录，初始化本地仓库

{% highlight bash %}
$ git init
$ git add .
$ git commit -m "first commit"
{% endhighlight %}

在坚果云同步目录中创建远程仓库

{% highlight bash %}
$ mkdir -p ~/Nutstore/git/project
$ cd ~/Nutstore/git/project
$ git init –bare
{% endhighlight %}

回到本地仓库目录，push代码到坚果云中

{% highlight bash %}
$ git remote add orig ~/Nutstore/git/project
$ git push orig master
{% endhighlight %}

这样在所有装了坚果云的电脑上都可以用git管理自己的代码。另一台电脑可以这样拉取代码：

{% highlight bash %}
$ git init
$ git remote add orig ~/Nutstore/git/project
$ git pull orig master
{% endhighlight %}

## Ubuntu美化

**安装主题numix（强烈推荐）**

{% highlight bash %}
sudo add-apt-repository ppa:numix/ppa
sudo apt update
sudo apt install numix-gtk-theme numix-icon-theme-circle
{% endhighlight %}

numix安装完成后，使用unity-tweak-tool将gtk主题换成Numix，图标换成Numix-Circle就大功告成了。

**安装Plank**

Plank是一个轻量的Dock工具。将Ubuntu侧边栏收起，Plank设置自动隐藏，可以获得一个很大的桌面空间，赏心悦目。在Dash里面搜索**startup**，打开**Startup Applications**，**命令**项填`sh -c "sleep 10 && plank"`将Plank添加为自启动。如果不设置延迟，则关机会变成注销，这是unity的一个bug。

{% highlight bash %}
sudo apt install plank
{% endhighlight %}

## 安装jekyll（用于github page）步骤

安装ruby和ruby-dev

{% highlight bash %}
sudo apt install ruby ruby-dev
{% endhighlight %}

到[RubyGems](https://rubygems.org)网站下载最新的ruby-gem，解压安装

{% highlight bash %}
sudo ruby setup.rb
{% endhighlight %}

由于ruby-gem官方源很慢，所以更换成国内源

{% highlight bash %}
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
{% endhighlight %}

安装jekyll，bundler

{% highlight bash %}
sudo gem install jekyll bundler
{% endhighlight %}

更换bundle源

{% highlight bash %}
bundle config mirror.https://rubygems.org https://gems.ruby-china.org
{% endhighlight %}

## 一些开发用到的库（请忽略）

* libboost-all-dev
* libxerces-c-dev
* libjsoncpp-dev
* libcurl4-gnutls-dev
* libmysqlcppconn-dev
* libssl-dev
* libmemcached-dev
* libiconv-1.14

编译libiconv-1.14代码会出错，需要改动srclib/stdio.in.h文件，将698行的代码：

{% highlight c %}
_GL_WARN_ON_USE (gets, "gets is a security hole - use fgets instead");
{% endhighlight %}

替换为：

{% highlight c %}
#if defined(__GLIBC__) && !defined(__UCLIBC__) && !__GLIBC_PREREQ(2, 16)
_GL_WARN_ON_USE (gets, "gets is a security hole - use fgets instead");
#endif
{% endhighlight %}
