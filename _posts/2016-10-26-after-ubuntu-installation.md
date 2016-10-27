---
layout: post
title: 安装完Ubuntu后需要做的事
date: 2016-10-26 16:03:15
categories: Ubuntu
excerpt: 安装Ubuntu后，需要安装的开发工具（C++相关）。
---

## 需要安装的软件：

* g++，必须的，C++编译器
* emacs，编辑器之神，谁用谁知道，入门门槛稍微高了一点，我现在还是菜鸟hoho
* git，连接github开源代码
* vim，自从加入emacs阵营后，这个就用的少了
* subversion，公司代码还是用svn管理
* cmake，使用cmake自动生成Makefile，编译工程很方便
* global，emacs里面用来生成GTAGS，利于阅读代码

## Ubuntu美化

**安装主题numix(强烈推荐)**

{% highlight bash %}
sudo add-apt-repository ppa:numix/ppa
sudo apt update
sudo apt install numix-gtk-theme numix-icon-theme-circle
{% endhighlight %}

numix安装完成后，使用unity-tweak-tool将gtk主题换成Numix，图标换成Numix-Circle就大功告成了。

**安装Plank**

Plank是一个轻量的Dock工具。将Ubuntu侧边栏收起，Plank设置自动隐藏，可以获得一个很大的桌面空间，赏心悦目。

{% highlight bash %}
sudo apt install plank
{% endhighlight %}

## 安装jekyll(用于github page)步骤

1. 安装ruby和ruby-dev

{% highlight bash %}
sudo apt install ruby ruby-dev
{% endhighlight %}

2. 到[RubyGems](https://rubygems.org)网站下载最新的ruby-gem，解压安装

{% highlight bash %}
sudo ruby setup.rb
{% endhighlight %}

3. 由于ruby-gem官方源很慢，所以更换成国内源

{% highlight bash %}
gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
{% endhighlight %}

4. 安装jekyll，bundler

{% highlight bash %}
sudo gem install jekyll bundler
{% endhighlight %}

5. 更换bundle源

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
