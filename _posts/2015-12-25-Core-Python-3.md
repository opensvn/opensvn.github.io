---
layout: post
title: Python核心编程 第3章
date: 2015-12-25 13:28:23
categories: Python
excerpt: 本章介绍Python语言的基础
---

## 3.1 语句和语法

关于Python语句的一些规则和符号：
* 井号（#）指示Python注释。
* 换行（\n）是标准行分隔符（一个语句一行）
* 反斜杠（\）延续一行
* 分号（;）将2个语句连接在一行
* 冒号（:）分隔标题行和其单元
* 单元通过缩进界定
* Python文件组织成模块

### 3.1.1 注释（#）

Python注释以井号开始。一个注释可以在一行的任何地方开始，所有跟在井号后面直到行尾的字符被解释器忽略。明智审慎地使用它们。

### 3.1.2 延续（\）

Python语句通常由换行界定，意思是一个语句一行。单一语句可以通过反斜杠分成多行。反斜杠可以放在换行符前面使当前语句延续到下一行。

{% highlight python %}
# check conditions
if (weather_is_hot == 1) and \
   (shark_warnings == 0):
    send_goto_beach_mesg_to_pager()
{% endhighlight %}

有两种例外可以不使用反斜杠延续到下一行。被括号，方括号或花括号包围的单一语句和包围在三重引号的字符串。

{% highlight python %}
# display a string with triple quotes
print '''hi there, this is a long message for you
that goes over multiple lines... you will find
out soon that triple quotes in Python allows
this kind of fun! it is like a day on the beach!'''
# set some variables
go_surf, get_a_tan_while, boat_size, toll_money = (1,
    'windsurfing', 40.0, -2.00)
{% endhighlight %}

### 3.1.3 多个语句组成程序组（:）

多个语句组成一个代码块在Python中称为程序组（suites）。复合语句如if，while，def和class都需要一个标题行和一个程序组。标题行以关键字开始一行语句并以冒号结尾。

### 3.1.4 通过缩进界定程序组

Python利用缩进作为一种界定代码块的方法。内层的代码通过空格或制表符缩进。一个程序组的所有代码行必须处在同样的缩进级别上。

> **核心风格：使用4个空格缩进并避免使用制表符**

当缩进的数量增加时，一个新的代码块被识别。而一个代码块的结束由缩进的数量减少并匹配上一层的缩进。没有缩进的代码（最高层的代码）被认为是脚本的主要部分。

### 3.1.5 多个语句在一行（;）

分号（;）允许多个语句在一行，这些语句都不开始一个新的代码块。

{% highlight python %}
import sys; x = 'foo'; sys.stdout.write(x + '\n')
{% endhighlight %}

不要多行语句放在一行，因为它使得代码更难阅读。

### 3.1.6 模块
每一个Python脚本都是一个模块。模块物理表现为硬盘文件。当一个模块变得足够大时，将一些代码移到另一个模块更合理。

## 3.2 变量赋值

### 赋值操作符

等号是Python主要的赋值操作符，其它的是增量赋值操作符。

{% highlight python %}
anInt = -12
aString = 'cart'
aFloat = -3.1415 * (5.0 ** 2)
anotherString = 'shop' + 'ping'
aList = [3.14e10, '2nd elmt of a list', 8.82-4.371j]
{% endhighlight %}

注意赋值并没有显式将一个值赋给一个变量。在Python中，对象都是被引用的。因此当赋值时，一个对象的引用被赋值给变量，不管对象是刚刚创建还是已经存在。

如果你熟悉C，赋值被当做表达式。Python中不是这样，下面这样的语句在Python中非法：

{% highlight python %}
>>> x = 1
>>> y = (x = x + 1) # assignments not expressions!
  File "<stdin>", line 1
    y = (x = x + 1)
           ^
SyntaxError: invalid syntax
{% endhighlight %}

赋值可以链接在一起：

{% highlight python %}
>>> y = x = x + 1
>>> x, y
(2, 2)
{% endhighlight %}

### 增量赋值操作符

从Python 2.0开始，等号可以和算术操作符一起使用，且产生的结果再赋值给原来的变量。比如：

{% highlight python %}
x = x + 1 <=> x += 1
{% endhighlight %}

增量赋值操作符有，**+=**，**-=**，***=**，**/=**，**%=**，**\*\*=**，**<<=**，**>>=**，**&=**，**^=**，**|=**。

除了明显的语法改变，最显著的不同是第一个对象只被检查一次。可变对象在原地改变，而不可变对象和可变对象有一样的效果（一个新对象被分配）。

{% highlight python %}
>>> m = 12
>>> m %= 7
>>> m
5
>>> m **= 2
>>> m
25
>>> aList = [123, 'xyz']
>>> aList += [45.6e7]
>>> aList
[123, 'xyz', 456000000.0
{% endhighlight %}

Python不支持前/后自增操作符，也不支持前/后自减操作符。

### 多个赋值

{% highlight python %}
>>> x = y = z = 1
{% endhighlight %}

上面这个例子中，一个整形对象被创建，且x，y，z都引用到这个整形对象。Python也可以将多个对象赋值给多个变量。

### 元组赋值

{% highlight python %}
>>> x, y, z = 1, 2, 'a string'
{% endhighlight %}

上面这个例子中，2个整形对象和一个字符串对象被赋值给x，y，z。尽管括号是可选的，我们建议在任何使代码容易阅读的地方使用括号：

{% highlight python %}
>>> (x, y, z) = (1, 2, 'a string')
{% endhighlight %}

Python元组赋值的一个有趣副作用是我们不再需要一个临时变量来交换2个变量的值：

{% highlight python %}
>>> x, y = y, x
{% endhighlight %}

显然，Python在赋值前执行计算。

## 3.3 标识符

标识符是一组合法的允许作为计算机语言里的名字的字符串。从这个包罗万象的列表中，我们分离出一些形成语言结构的关键字。这些标识符是不应该用作其它意图的保留字，否则将发生语法错误。

Python也有一组额外的标识符集被称为built-ins。尽管它们不是保留字，不推荐使用这些特殊名字。

### 3.3.1 合法Python标识符

Python标识符的规则和其它来自C世界的高级语言一样：
* 第一个字符必须是字母或是下划线（_）。
* 任何其它的字符可以是字母，数字或下划线。
* 大小写敏感。

### 3.3.2 关键字

**keyword**模块包含关键字列表和一个**iskeyword()**函数。

{% highlight python %}
>>> import keyword
>>> keyword.kwlist
['and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del',
 'elif', 'else', 'except', 'exec', 'finally', 'for', 'from', 'global',
 'if', 'import', 'in', 'is', 'lambda', 'not', 'or', 'pass', 'print',
 'raise', 'return', 'try', 'while', 'with', 'yield']
{% endhighlight %}

### 3.3.3 Built-ins

除了关键字，Python还有一组内置名字集合。尽管不是关键字，内置名字应该被当做留给系统的，不应该用作任何其它意图。

内置名字都是**__builtins__**模块的成员，这个模块在程序开始前由解释器自动导入。

