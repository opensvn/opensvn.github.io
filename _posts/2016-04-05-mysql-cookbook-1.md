---
layout: post
title: MySQL Cookbook 第1章 使用mysql客户端
date: 2016-04-05 15:56:30
categories: MySQL
excerpt: 介绍如何使用mysql客户端
---

# 1.0 介绍

本章描述了mysql的能力以使你能够更有效的使用它：

* 设置一个MySQL账户以使用cookbook数据库
* 指定连接参数和使用配置文件
* 交互模式和批量模式执行SQL语句
* 控制mysql输出格式
* 使用用户定义的变量保存信息

为了你自己试验本书中的例子，你需要一个MySQL用户和一个数据库。接下来假设：:

* MySQL服务器运行在本地系统中
* 你的MySQL用户和密码是cbuser和cbpass
* 你的数据库是cookbook

# 1.1 创建一个MySQL用户

**问题**

你需要一个账户来连接MySQL服务器

**解决方法**

使用**CREATE USER**和**GRANT**语句配置账户，然后使用账户名密码连接服务器

**讨论**

    % mysql -h localhost -u root -p
    Enter password: ******
    mysql> CREATE USER 'cbuser'@'localhost' IDENTIFIED BY 'cbpass';
    mysql> GRANT ALL ON cookbook.* TO 'cbuser'@'localhost';
    Query OK, 0 rows affected (0.09 sec)
    mysql> quit
    Bye

如果你想要连接到其他主机的服务器上，在**CREATE USER**和**GRANT**语句里面替换为其他主机，语句类似：

    mysql> CREATE USER 'cbuser'@'myhost.example.com' IDENTIFIED BY 'cbpass';
    mysql> GRANT ALL ON cookbook.* TO 'cbuser'@'myhost.example.com';

**CREATE USER**和**GRANT**只能被有管理权限的用户比如root使用。创建cbuser用户后，确认你能使用它连接MySQL：

    % mysql -h localhost -u cbuser -p
    Enter password: cbpass

# 1.2 创建一个数据库和一个样例表

**问题**

你想要创建一个数据库并在里面创建表

**解决方法**

使用**CREATE DATABASE**语句创建数据库，**CREATE TABLE**语句创建每一个表，**INSERT**语句添加行到表中。

**讨论**

如下创建一个数据库：

    mysql> CREATE DATABASE cookbook;

既然有了一个数据库，你可以在里面创建表。首先，选择cookbook作为默认数据库：

    mysql> USE cookbook;

然后创建一个简单的表：

    mysql> CREATE TABLE limbs (thing VARCHAR(20), legs INT, arms INT);

并用一些行填充它：

    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('human',2,2);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('insect',6,0);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('squid',0,10);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('fish',0,0);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('centipede',100,0);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('table',4,0);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('armchair',4,2);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('phonograph',0,1);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('tripod',3,0);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('Peg Leg Pete',1,2);
    mysql> INSERT INTO limbs (thing,legs,arms) VALUES('space alien',NULL,NULL);

执行**SELECT**语句确认这些行被添加进表：

    mysql> SELECT * FROM limbs;
    +--------------+------+------+
    | thing        | legs | arms |
    +--------------+------+------+
    | human        |    2 |    2 | 
    | insect       |    6 |    0 | 
    | squid        |    0 |   10 | 
    | fish         |    0 |    0 | 
    | centipede    |  100 |    0 | 
    | table        |    4 |    0 | 
    | armchair     |    4 |    2 | 
    | phonograph   |    0 |    1 | 
    | tripod       |    3 |    0 | 
    | Peg Leg Pete |    1 |    2 | 
    | space alien  | NULL | NULL | 
    +--------------+------+------+

# 1.3 如果mysql找不到要做什么

**问题**

当你从命令行调用mysql时，命令行解释器找不到mysql。

**解决方法**

添加mysql所在目录到**PATH**路径。

**讨论**

告诉命令解释器哪里查找mysql的一种方法是使用绝对路径。Unix下面命令可能看起来像：

    % /usr/local/mysql/bin/mysql

Windows下面看起来像：

    C:\> "C:\Program Files\MySQL\MySQL Server 5.6\bin\mysql"

一个更好的方案是修改你的**PATH**搜索路径环境变量，添加mysql所在目录，然后你就可以在任何地方调用mysql了。

可以从任何地方调用mysql有一个显著的额外的好处是，你不需要将数据文件放在mysql安装目录，你可以自由地以你认为合理的方式组织这些文件。

# 1.4. 指定mysql命令选项

**问题**

当你调用mysql程序不带命令选项时，程序返回一个“拒绝访问”的信息并立即退出。

**解决方法**

你必须指定连接参数，在命令行，配置文件或两者结合。

**讨论**

每一个选项是单破折号“短”形式：-h和-u指定主机名和用户名，-p提示输入密码。也有相应的双破折号“长”形式：--host，--user和--password。像这样使用：

    % mysql --host=localhost --user=cbuser --password
    Enter password: cbpass

使用这个命令了解mysql支持的所有选项：

    % mysql --help

为mysql指定的命令选项也适用于其他MySQL程序，比如mysqldump和mysqladmin。

    % mysqldump -h localhost -u cbuser -p cookbook > cookbook.sql
    Enter password: cbpass

某些操作需要一个MySQL管理员用户。mysqladmin可以执行MySQL root用户才能够的操作：

    % mysqladmin -h localhost -u root -p shutdown
    Enter password: ← enter MySQL root account password here

如果你使用的选项值和默认值一样，你可以忽略这个选项。没有默认密码。如果你喜欢，你可以直接在命令行用-ppassword或--password=password指定密码。不推荐这种方式。

因为默认主机就是本地主机，你可以忽略-h（--host）选项：

    % mysql -u cbuser -p

假设你不想指定任何选项，你怎么让mysql知道使用什么值呢？很简单，因为MySQL支持配置文件：

* 如果你在配置文件放入一个选项，你不需要在命令行指定它。
* 你可以混合命令行和配置文件的选项。这使你可以将最常使用的选项保存到配置文件并根据需要在命令行覆盖它们。

>   **MySQL中localhost的意义**
>
>Unix下面，MySQL程序表现不一样：按照惯例，他们将主机名hostname特殊处理并尝试用一个Unix socket文件连接本地服务器。为了强制使用TCP/IP连接到本地服务器，使用IP地址127.0.0.1（或::1）而不是localhost。
>
>TCP/IP连接的默认端口是3306。Unix域套接字的路径各不相同，虽然通常是/tmp/mysql.sock。使用-S file_name或--socket=file_name显示指定套接字路径。

## 使用配置文件指定连接参数

为了避免每次调用mysql都输入命令行选项，将它们放入配置文件，mysql会自动读取它们。配置文件是纯文本文件：

* 在Unix下，个人配置文件在个人目录，名为.my.cnf。也有管理员可以使用的指定给所有用户的全局参数的配置文件，在/etc或etc/mysql或MySQL安装目录下的etc目录。
* 在Windows下，你可以使用的文件包括安装目录下的my.ini或my.cnf文件.

下面这个例子展示了MySQL配置文件使用的格式：

    # general client program connection options
    [client]
    host = localhost
    user = cbuser
    password = cbpass
    # options specific to the mysql program
    [mysql]
    skip-auto-rehash
    pager="/usr/bin/less -E" # specify pager for interactive mode

使用刚刚展示的[client]组列出的连接参数，你可以不带参数调用mysql：

    % mysql

使用这个命令找出mysql程序从配置文件读取的默认参数：

    % mysql --print-defaults

你也可以使用my_print_defaults工具，它接受配置文件中的组名作为参数：

    % my_print_defaults client mysqldump

## 组合命令行和配置文件参数

MySQL程序首先读取配置文件中连接参数，然后检查命令行中额外的参数。这意味着你可以一种方式指定某些参数，以另一种方式指定其它参数：

    % mysql -p
    Enter password: ← enter your password here

命令行参数优先于配置文件中的参数，因此为了覆盖一个配置文件中的参数，只要在命令行指定就行：

    % mysql -u root -p
    Enter password: ← enter MySQL root account password here

当配置文件中有一个非空的密码选项，为了显示指定“无密码”，在命令行中使用--skip-password选项：

    % mysql --skip-password

## 从其他用户保护配置文件

在一个多用户的操作系统比如Unix中，保护你目录下配置文件以阻止其他用户阅读并发现如何使用你的账户连接MySQL。使用chmod将文件变为私有：

    % chmod 600 .my.cnf
    % chmod go-rwx .my.cnf

# 1.5. 交互执行SQL语句

**问题**

你已经启动了mysql。现在你想要发送SQL语句到MySQL服务器执行。

**解决方法**

输入语句，让mysql知道每一句在哪里结束，或者直接在命令行指定“单行”语句。

**讨论**

分号是最常见的终结符，你也可以使用\g（“go”）作为分号的同义词。

    mysql> SELECT NOW();
    +---------------------+
    | NOW()               |
    +---------------------+
    | 2016-04-05 20:20:31 |
    +---------------------+
    mysql> SELECT
        -> NOW()\g
    +---------------------+
    | NOW()               |
    +---------------------+
    | 2016-04-05 20:20:40 |
    +---------------------+

;和\g不是语言的一部分。它们是mysql程序使用的惯例，在发送语句到MySQL服务器前，mysql识别这些终结符并从语句中去除它们。

某些语句产生输出的行很长以致它们占据终端一行以上，使得查询结果很难读。为了避免这个问题，使用\G产生“垂直”输出：

    mysql> SHOW FULL COLUMNS FROM limbs LIKE 'thing'\G
    *************************** 1. row ***************************
         Field: thing
          Type: varchar(20)
     Collation: latin1_swedish_ci
          Null: YES
           Key: 
       Default: NULL
         Extra: 
    Privileges: select,insert,update,references
       Comment: 

为了在一个会话中所有语句执行的结果产生垂直输出，使用-E（或--vertical）选项。为了那些只超过终端宽度的结果产生垂直输出，使用--auto-vertical-output选项。

为了从命令行直接执行一个语句，指定-e（或--execute）选项：

    % mysql -u cbuser -p -e "SELECT COUNT(*) FROM limbs" cookbook
    Enter password: 
    +----------+
    | COUNT(*) |
    +----------+
    |       11 |
    +----------+

为了执行多个语句，用分号分隔它们：

    % mysql -u cbuser -p -e "SELECT COUNT(*) FROM limbs; SELECT NOW()" cookbook
    Enter password: 
    +----------+
    | COUNT(*) |
    +----------+
    |       11 |
    +----------+
    +---------------------+
    | NOW()               |
    +---------------------+
    | 2016-04-05 20:27:14 |
    +---------------------+

# 1.6. 从文件或程序输出中执行SQL语句

**问题**

你想要mysql读取保存在文件中的语句而不必手动输入它们。或者你想要mysql读取其他程序的输出。

**解决方法**

为了读一个文件，重定向mysql的输入，或使用source命令。为了读取程序输出，使用管道。

**讨论**

为了创建一个mysql批量执行的SQL脚本，将你的语句放进一个文本文件。然后调用mysql并重定向输入读取那个文件：

    % mysql cookbook < file_name

从输入文件读取的语句必须以;，\g或\G结尾。交互和批量模式的输出格式不一样。交互模式默认输出表格形式，批量模式默认以tab分隔。使用合适的命令选项可以覆盖默认行为。

为了在一个mysql会话中读取包含SQL语句的文件，使用source filename命令（或\. filename）：

    mysql> source limbs.sql;
    mysql> \. limbs.sql;

SQL脚本自己可以包含source或\.命令包含其它脚本。这样更具灵活性，但是要注意避免source循环。

SQL脚本不需要手写，它可以是程序生成的：

    % mysqldump cookbook > dump.sql
    % mysql -h other-host.example.com cookbook < dump.sql

任何程序输出产生包含正确结尾的SQL语句，都可以作为mysql的输入源：

    % mysqldump cookbook | mysql -h other-host.example.com cookbook
    % generate-test-data | mysql cookbook

# 1.7. 控制mysql输出的目的地和格式

**问题**

你想要mysql输出到别的地方而不是屏幕，且你不想要默认的输出格式。

**解决方法**

重定向输出到一个文件，或使用管道将输出发送给一个程序。你也可以控制mysql的输出，产生表格，tab分隔的，HTML或XML输出，去掉列头，或使mysql更加详细或更少信息。

**讨论**

使用shell的重定向能力保存mysql输出到文件：

    % mysql cookbook > outputfile
    % mysql cookbook < inputfile > outputfile
    % mysql cookbook < inputfile | mail paul

## 产生表格或tab分隔的输出

交互模式下，mysql默认使用表格输出：

    mysql> SELECT * FROM limbs WHERE legs = 0;
    +------------+------+------+
    | thing      | legs | arms |
    +------------+------+------+
    | squid      |    0 |   10 |
    | fish       |    0 |    0 |
    | phonograph |    0 |    1 |
    +------------+------+------+

非交互模式下，mysql默认使用tab分隔输出：

    % echo "SELECT * FROM limbs WHERE legs = 0" | mysql -u cbuser -p cookbook
    Enter password: 
    thing   legs    arms
    squid   0   10
    fish    0   0
    phonograph  0   1

非交互模式下，使用-t（或--table）选项产生表格输出：

    % echo "SELECT * FROM limbs WHERE legs = 0" | mysql -u cbuser -p cookbook -t
    Enter password: 
    +------------+------+------+
    | thing      | legs | arms |
    +------------+------+------+
    | squid      |    0 |   10 |
    | fish       |    0 |    0 |
    | phonograph |    0 |    1 |
    +------------+------+------+

使用-B或--batch在交互模式下产生tab分隔的输出。

## 产生HTML或XML输出

mysql从每一个查询结果里面产生一个HTML表，如果使用-H选项：

    % mysql -u cbuser -p -H -e "SELECT * FROM limbs WHERE legs = 0"  cookbook
    Enter password: 
    <TABLE BORDER=1><TR><TH>thing</TH><TH>legs</TH><TH>arms</TH></TR><TR><TD>squid</TD><TD>0</TD><TD>10</TD></TR><TR><TD>fish</TD><TD>0</TD><TD>0</TD></TR><TR><TD>phonograph</TD><TD>0</TD><TD>1</TD></TR></TABLE>

你可以将输出保存到一个文件，然后在浏览器中查看：

    % mysql -H -e "SELECT * FROM limbs WHERE legs=0" cookbook > limbs.html
    % open -a safari limbs.html

使用-X（或--xml）选项产生XML文件：

    % mysql -u cbuser -p -X -e "SELECT * FROM limbs WHERE legs = 0"  cookbook > limbs.xml

像这样使用转换：

    % mysql -X -e "SELECT * FROM limbs WHERE legs=0" cookbook \
    | xsltproc mysql-xml.xsl -
    Query: SELECT * FROM limbs WHERE legs=0
    Result set:
    squid, 0, 10
    fish, 0, 0
    phonograph, 0, 1

-H，--html，-X和--xml选项只对产生结果集的语句有用，对INSERT或UPDATE无效。

## 查询结果中丢弃列头

使用--skip-column-names选项创建输出只包含数据值，丢弃了头：

    % mysql --skip-column-names -e "SELECT arms FROM limbs" cookbook | summarize

指定“沉默”选项（-s或--silent）2次是一样的效果：

    % mysql -ss -e "SELECT arms FROM limbs" cookbook | summarize

## 指定输出列分隔符

在非交互模式下，mysql使用tab分隔输出列，没有指定输出分隔符的选项。可以处理mysql输出以使用不同的分隔符：

    % mysql cookbook < inputfile | sed -e "s/TAB/:/g" > outputfile
    % mysql cookbook < inputfile | tr "TAB" ":" > outputfile
    % mysql cookbook < inputfile | tr "\011" ":" > outputfile
    % mysql cookbook < inputfile \
    | sed -e 's/"/""/g' -e 's/TAB/","/g' -e 's/^/"/' -e 's/$/"/' > outputfile

## 控制mysql的详细级别

使用-v或--verbose告诉mysql变得更详细，指定选项多次可以增强详细度：

    % mysql -u cbuser -p -e "SELECT NOW()"
    % mysql -u cbuser -p -e "SELECT NOW()" -v
    % mysql -u cbuser -p -e "SELECT NOW()" -vv
    % mysql -u cbuser -p -e "SELECT NOW()" -vvv

-s和--silent也有一样的效果。

# 1.8. 在SQL语句中使用用户定义的变量

**问题**

你想要在一个语句中使用之前语句产生值。

**解决方法**

保存这个值在用户定义的变量里以供以后使用。

**讨论**

用户定义的变量使你在用一个会话中其他语句中使用它。用户变量是MySQL特定的扩展，不是标准SQL。

使用语法@var_name := val定义用户变量，变量可以在接下来的语句中使用：

    mysql> SELECT @max_limbs := MAX(arms + legs) FROM limbs;
    +--------------------------------+
    | @max_limbs := MAX(arms + legs) |
    +--------------------------------+
    |                            100 |
    +--------------------------------+
    mysql> SELECT * FROM limbs WHERE arms + legs = @max_limbs;
    +-----------+------+------+
    | thing     | legs | arms |
    +-----------+------+------+
    | centipede |  100 |    0 |
    +-----------+------+------+

变量的另一个用法是保存LAST_INSERT_ID()的结果在创建一个具有AUTO_INCREMENT列的表的新行后：

    mysql> SELECT @last_id := LAST_INSERT_ID();

LAST_INSERT_ID() 返回最新的AUTO_INCREMENT值。

用户变量保存单一值，如果一个语句返回多行，最后一行保存到变量中：

    mysql> SELECT @name := thing FROM limbs WHERE legs = 0;
    +----------------+
    | @name := thing |
    +----------------+
    | squid          |
    | fish           |
    | phonograph     |
    +----------------+
    mysql> SELECT @name;
    +------------+
    | @name      |
    +------------+
    | phonograph |
    +------------+

如果语句返回0行，赋值不会发生，且变量保存原来的值。如果变量之前没用过，则其值为NULL：

    mysql> SELECT @name2 := thing FROM limbs WHERE legs < 0;
    Empty set (0.00 sec)
    mysql> SELECT @name2;
    +--------+
    | @name2 |
    +--------+
    | NULL   |
    +--------+

为了显式设置某个值给一个变量，使用SET语句。SET语法可以使用:=或=作为赋值操作符：

    mysql> SET @sum = 4 + 7;
    mysql> SELECT @sum;
    +------+
    | @sum |
    +------+
    |   11 |
    +------+

你可以将SELECT结果赋值个一个变量，只要它是一个标量子查询（括号里面的只返回一个值的查询）：

    mysql> SET @max_limbs = (SELECT MAX(arms+legs) FROM limbs);

用户变量名大小写不敏感：

    mysql> SET @x = 1, @X = 2; SELECT @x, @X;
    +------+------+
    | @x   | @X   |
    +------+------+
    |    2 |    2 |
    +------+------+

用户变量只能出现在表达式允许出现的地方，不是常量或字符标识符必须提供。变量不能用了作为表名：

    mysql> SET @tbl_name = CONCAT('tmp_tbl_', CONNECTION_ID());
    mysql> CREATE TABLE @tbl_name (int_col INT);
    ERROR 1064: You have an error in your SQL syntax near '@tbl_name
    (int_col INT)'

SET也被用来给存储程序参数，本地变量和系统变量赋值。



