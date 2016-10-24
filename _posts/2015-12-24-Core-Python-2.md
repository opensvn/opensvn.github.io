---
layout: post
title: Python核心编程 第2章
date: 2015-12-24 09:46:13
categories: Python
excerpt: 本章主要是Python语言的概要介绍
---

# 命令行选项

-d      提供调试输出
-O      生成优化字节码（产生.pyo文件）
-S      启动时不要运行导入地址查询Python路径
-v      详细输出（详细追踪import语句）
-m      mod 将模块当脚本运行
-Q      opt 除法选项
-c      cmd 将cmd字符串当Python脚本执行
file    从指定文件运行Python脚本

# 2.1 程序输出，print语句和"Hello World!"

Python的**print**语句（2.x）或函数（3.x）给用户显示程序输出的工具，类似C语言的**printf()**和shell脚本的**echo**。实际上，它也支持**printf()**风格的字符串替换用法：

    >>> print "%s is number %d!" % ("Python", 1)
    Python is number 1!

> **核心笔记：在交互解释器中打印变量内容**
> 在交互解释器中，你可以提供变量的名字直接打印变量的内容。print语句使用str()打印内容，而交互解释器调用repr()显示对象。下划线（_）在交互解释器中也有特殊含义：上一个计算过的表达式。

**print**语句也可以将输出重定向到文件。

    import sys
    print >> sys.stderr, 'Fatal error: invalid input!'

    logfile = open('/tmp/mylog.txt', 'a')
    print >> logfile, 'Fatal error: invalid input!'
    logfile.close()

**print**在Python 3.0变成了函数[print()]。从Python 2.6开始，你可以通过添加这句特殊的import语句使用print函数：

    from __future__ import print_function

新函数的语法是：

    print(*args, sep=' ', end='\n', file=None)

    print('Fatal error: invalid input!', file=sys.stderr)

# 2.2 程序输入和内置raw_input()函数

从命令行获取用户输入最简单的方法是使用内置的**raw_input()**函数。它从标准输入读取输入并将字符串赋值给你指定的变量。

    >>> user = raw_input('Enter login name: ')
    Enter login name: root
    >>> print 'Your login is:', user
    Your login is: root

    >>> num = raw_input('Now enter a number: ')
    Now enter a number: 1024
    >>> print 'Doubling your number: %d' % (int(num) * 2)
    Doubling your number: 2048

内置的**int()**函数将字符串转换成整数。

> **核心笔记：在交互解释器中寻求帮助**
> 在学习Python的过程中，如果需要对一个你不熟悉的新函数的帮助文档，你可以调用help()内置函数寻求帮助，比如help(raw_input)。
> **核心风格：用户交互放在函数外面**
> 我们建议函数应该保持干净，意思是函数应该只用来接收参数并返回值。

# 2.3 注释

和大多数脚本语言一样，井号（#）表示一个注释开始并持续到一行结尾。

    >>> # one comment
    ... print 'Hello World!' # another comment
    Hello World!

有一种特殊的注释被称为文档字符串。在一个模块，类或函数开头独立的字符串就是文档字符串。文档字符串可以在运行时访问并用来自动生成文档。可以使用object.__doc__访问注释文档。

# 2.4 操作符

你熟知的标准数学操作符在Python里面跟其它大多数语言一样工作。

    +   -   *   /   //  %   **

Python有2个除法操作符，一个斜杠用来做经典除法而2个斜杠用来做截断除法。经典除法的意思是如果操作数都是整数，则执行截断除法，而对于浮点型执行真实除法。如果真实除法生效，则除法操作符总是执行真实除法，不管操作数类型。两个星号（**）是指数操作符。

Python也提供了标准比较操作符，这些操作符返回布尔值：

    <   <=  >   >=  ==  !=  <>

Python当前支持两种不等操作符，!=和<>。后面这个正逐渐被淘汰，推荐使用前面的。

Python也提供表达式连接操作符：**and**，**or**，**not**。

    >>> 3 < 4 < 5
    True
    >>> 3 < 4 and 4 < 5
    True

> 核心风格：使用括号澄清
> 很多情况下，使用括号是个好主意。比如如果没有它们代码很难阅读，或如果没有它们容易造成混淆。

# 2.5 变量和赋值

Python中变量的规则和大多数其它高级语言一样。它们仅仅是以字母开头的标识符。Python是大小写敏感的。

Python是动态类型的，意味着变量类型的预先声明是不必要的。类型和值在赋值时初始化，使用等号进行赋值。

    >>> counter = 0
    >>> miles = 1000.0
    >>> name = 'Bob'
    >>> counter = counter + 1
    >>> kilometers = 1.609 * miles
    >>> print '%f miles is the same as %f km' % (miles, kilometers)
    1000.000000 miles is the same as 1609.000000 km

Python支持增量赋值，比如n *= 10。Python不支持自增和自减操作符。

# 2.6 数字

Python支持五种基本数字类型，其中三种是整形。
* **int**（有符号整数）
    * **long**（long整形）
    * **bool**（Boolean值）
* **float**（浮点型实数）
* **complex**（复数）

Python的long型没有范围限制，它只局限于系统的内存。如果你熟悉Java，Python的long型类似于BigInteger类型。未来int和long将被统一进一种整形。从2.3开始，溢出错误不再报告，其结果将自动转为long。Python 3中，int和long被统一为一个整形，且"L"不再是合法的Python语法。

Boolean值是整形的一个特殊情况。尽管由常量**True**和**False**表示，如果放进数值环境比如与其它数作加法，**True**被看作数字1，而**False**为0。

还有一种数值类型，decimal，代表十进制浮点数。但是它不是内置类型。你必须导入decimal模块才能使用。

# 2.7 字符串

字符串在Python中被表示为引号之间的一组连续的字符。Python允许使用一对单引号或一对双引号。三重引号（三个连续的单引号或双引号）可以用来转义特殊字符。可以使用下标（[]）和切片（[:]）获取子字符串。加号（+）是字符串连接操作符，星号（*）是重复操作符。

    >>> pystr = 'Python'
    >>> iscool = 'is cool!'
    >>> pystr[0]
    'P'
    >>> pystr[2:5]
    'tho'
    >>> iscool[:2]
    'is'
    >>> iscool[3:]
    'cool!'
    >>> iscool[-1]
    '!'
    >>> pystr + iscool
    'Pythonis cool!'
    >>> pystr + ' ' + iscool
    'Python is cool!'
    >>> pystr * 2
    'PythonPython'
    >>> '-' * 20
    '--------------------'
    >>> pystr = '''python
    ... is cool'''
    >>> pystr
    'python\nis cool'
    >>> print pystr
    python
    is cool
    >>>

# 2.8 列表和元组

列表和元组可以被认为是通用“数组”，使用它可以存放任意数量的任意Python对象，其元素顺序存放并通过下标访问。

列表和元组有一些主要的区别。列表由中括号（[]）括起来，其元素和大小可以改变。元组由括号（()）括起来且不能更新。元组可以认为是“只读”列表。像字符串一样可以使用切片操作符（[]和[:]）获取子集。

# 2.9 字典

字典是Python的映射类型，像Perl里面的关联数组或哈希一样工作，它由键值对组成。键可以是几乎任何Python类型，但是通常是数字或字符串。另一方面，值可以是任意Python对象。字典由花括号（{}）括起来。

    >>> aDict = {'host': 'earth'} # create dict
    >>> aDict['port'] = 80 # add to dict
    >>> aDict
    {'host': 'earth', 'port': 80}
    >>> aDict.keys()
    ['host', 'port']
    >>> aDict['host']
    'earth'
    >>> for key in aDict:
    ... print key, aDict[key]
    ...
    host earth
    port 80

# 2.10 代码块使用缩进

代码块由缩进指定而不是用像花括号这样的符号。没有额外的符号，程序更容易阅读。同样，缩进清晰地指出代码属于哪一个代码块。当然代码块可以由单个语句组成。

# 2.11 if语句

标准**if**条件语句遵循下面语法：

    if expression:
        if_suite

如果表达式非0或True，则if_suite被执行。

    if x < .0:
        print '”x” must be at least 0!'

Python支持**else**语句，和**if**一起使用：

    if expression:
        if_suite
    else:
        else_suite

Python有一个“else-if”书写为**elif**使用下面的语法：

    if expression1:
        if_suite
    elif expression2:
        elif_suite
    else:
        else_suite

# 2.12 while循环

标准while条件循环语句类似**if**，语法如下：

    while expression:
        while_suite

while_suite语句在循环里面一直被执行直到expression变为0或False。

    >>> counter = 0
    >>> while counter < 3:
    ...     print 'loop #%d' % (counter)
    ...     counter += 1
    loop #0
    loop #1
    loop #2

# 2.13 for循环和range()内置函数

Python中的for循环更像一个脚本语言中foreach迭代类型循环。Python的for接受一个iterable并遍历每一个元素。

    >>> for item in ['e-mail', 'net-surfing', 'homework',
    'chat']:
    ...     print item
    e-mail
    net-surfing
    homework
    chat

**print**语句默认在每一行结尾添加一个换行符。可以在**print**语句后面加一个逗号抑制这个默认行为。

    print 'I like to use the Internet for:'
    for item in ['e-mail', 'net-surfing', 'homework', 'chat']:
        print item,
    print

**print**语句中逗号分隔的元素当它们被显示时会自动包含一个分隔的空格。

Python提供**range()**内置函数为我们生成一个列表。

    >>> for eachNum in range(3):
    ...     print eachNum
    >>> for c in 'abc':
    ...     print c
    >>> for i in range(len('abc')):
    ...     print foo[i], '(%d)' % i
    >>> for i, ch in enumerate(foo):s
    ...     print ch, '(%d)' % i

# 2.14 列表推导式

列表推导式使用一个for循环生成一个列表：

    >>> squared = [x ** 2 for x in range(4)]
    >>> for i in squared:
    ...     print i

列表推导式甚至可以选择性的包含什么进新列表：

    >>> sqdEvens = [x ** 2 for x in range(8) if not x % 2]

# 2.15 文件和open()，file()内置函数

如何打开一个文件：

    handle = open(file_name, access_mode = 'r')

file_name变量包含字符串类型的文件名，access_mode可以是读（r），写（w），追加（a）。其它标志包括读写（+），二进制访问（b）。

如果open()成功，一个文件对象被返回。所有后续文件访问必须通过这个文件对象。

> **核心笔记：什么是属性？**
> 属性是关联到一块数据的项。属性可以是简单数据值或可执行对象比如函数或方法。通过点号访问属性：object.attribute。

    filename = raw_input('Enter file name: ')
    fobj = open(filename, 'r')
    data = fobj.readlines()
    fobj.close()
    for eachLine in data:
        print eachLine,

**file()**内置函数最近才添加进Python。它和**open()**是相同的，但是这种命名指示它是一个工厂函数（生产文件对象），类似于**int()**生产整数对象和**dict()**生产字典对象。

# 2.16 错误和异常

语法错误在编译时检查，但是Python也允许程序运行期间发现错误。当一个错误被检测，Python解释器抛出一个异常。使用**try-except**语句添加错误检测或异常处理到你的代码。

    try:
        filename = raw_input('Enter file name: ')
        fobj = open(filename, 'r')
        for eachLine in fobj:
            print eachLine,
        fobj.close()
    except IOError, e:
        print 'file open error:', e

程序员可以显式使用**raise**抛出一个异常。

# 2.17 函数

和其它语言一样，Python中的函数使用函数操作符（()）调用，函数必须在调用前声明。不需要声明函数返回类型或显式返回值，如果没有返回值，Python返回**None**。

Python可以被认为是“引用调用”。这意味着函数内任何对函数参数的改变会影响到原来的对象。然而在Python中，这实际上取决于传递的对象类型。如果这个对象允许更新，则它表现为我们预期的“引用调用”，但是如果对象的值不能改变，则它表现为“传值调用”。

## 如何定义函数

    def function_name([arguments]):
        "optional documentation string"
        function_suite

声明一个函数的语法由**def**关键字，跟着函数名和任意数量函数可能接收的参数组成。

    def addMe2Me(x):
        'apply + operation to argument'
        return (x + x)

# 默认实参

函数可以有具有默认值的参数。如果参数的值没有提供，将会使用默认值：

    >>> def foo(debug=True):
    ...     'determine if in debug mode with default argument'
    ...     if debug:
    ...         print 'in debug mode'
    ...     print 'done'
    >>> foo()
    in debug mode
    done
    >>> foo(False)
    done

# 2.18 类

类是面向对象编程的核心部分，并作为一个“容器”服务相关的数据和逻辑。

## 如何定义类

    class ClassName(base_class[es]):
        "optional documentation string"
        static_member_declarations
        method_declarations

类使用**class**关键字声明。基类或父类可选，如果没有，使用object作为基类。

    class FooClass(object):
        """my very first class: FooClass"""
        version = 0.1 # class (data) attribute

        def __init__(self, nm='John Doe'):
            """constructor"""
            self.name = nm # class instance (data) attribute
            print 'Created a class instance for', nm

        def showname(self):
            """display instance attribute and class name"""
            print 'Your name is', self.name
            print 'My name is', self.__class__.__name__

        def showver(self):
            """display class(static) attribute"""
            print self.version # references FooClass.version

        def addMe2Me(self, x): # does not use 'self'
            """apply + operation to argument"""
            return x + x

**__init__()**方法一个默认提供的函数，当类实例创建的时候被调用，类似一个构造器并在对象实例化后被调用。**self**基本上是实例自己本身的句柄，其它面向对象语言通常使用**this**。

## 如何创建实例

创建实例看起来像调用一个函数，并拥有一样的语法。

    >>> foo1 = FooClass()
    Created a class instance for John Doe
    >>> foo1.showname()
    Your name is John Doe
    My name is FooClass
    >>> foo1.showver()
    0.1
    >>> print foo1.addMe2Me(5)
    10
    >>> print foo1.addMe2Me('xyz')
    xyzxyz

# 2.19 模块

一个模块是物理组织和区分相关的Python代码到独立文件的逻辑方式。一个模块可以包含可执行代码，函数，类或所有任何以上的。

当你创建了一个Python源文件，模块名和文件名去除扩展名一样。一旦一个模块创建了，你可以使用**import**语句导入模块。

    import module_name

一旦导入模块，模块的属性（函数和变量等）可以用点号访问：

    module.function()
    module.variable

    >>> import sys
    >>> sys.stdout.write('Hello World!\n')
    Hello World!
    >>> sys.platform
    'win32'
    >>> sys.version
    '2.4.2 (#67, Sep 28 2005, 10:51:12) [MSC v.1310 32 bit
    (Intel)]'

> **核心笔记：PEP是什么？**
> PEP是Python Enhancement Proposal。它是新特性被引进到未来版本的Python的一种方式。[PEP链接](http://python.org/dev/peps)

# 2.20 有用的函数

|函数  |描述|
|------------|:----------|
|dir([obj])|显示obj的属性或全局变量的名字如果参数未提供|
|help([obj])|显示obj的帮助文档或进入交互帮助界面如果参数未提供|
|int(obj)|将obj转换为整形|
|len(obj)|返回obj的长度|
|open(fn, mode)|用mode打开文件fn（'r'读，'w'写）|
|range([start,]stop[,step])|返回一个整形列表，从start开始到stop（不包括stop），每次增量step；start默认为0，step默认为1|
|raw_input(str)|等待用户输入，str可选|
|str(obj)|将obj转换为字符串|
|type(obj)|返回obj的类型（本身是一个type对象）|