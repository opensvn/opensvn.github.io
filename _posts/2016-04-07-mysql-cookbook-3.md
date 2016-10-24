---
layout: post
title: MySQL Cookbook 第3章 从表中查询数据
date: 2016-04-07 12:14:29
categories: MySQL
excerpt: 本章介绍如何查询数据
---

# 3.0. 介绍

本章专注使用SELECT语句从你的数据库获取信息。

# 3.1. 指定SELECT哪行哪列

**问题**

你想要从一张表中展示特定行和列。

**解决方法**

为了指示展示哪一列，在输出列表中指定它。为了指示展示哪一行，使用WHERE语句指定满足条件的行。

**讨论**

从一个表中显示列最简单的方式是使用SELECT * FROM tbl_name。*限定符意思是“所有列”：

    mysql> SELECT * FROM mail; 
    +---------------------+---------+---------+---------+---------+---------+
    | t                   | srcuser | srchost | dstuser | dsthost | size    |
    +---------------------+---------+---------+---------+---------+---------+
    | 2014-05-11 10:15:08 | barb    | saturn  | tricia  | mars    |   58274 | 
    | 2014-05-12 12:48:13 | tricia  | mars    | gene    | venus   |  194925 | 
    | 2014-05-12 15:02:49 | phil    | mars    | phil    | saturn  |    1048 | 
    | 2014-05-12 18:59:18 | barb    | saturn  | tricia  | venus   |     271 | 

显式命名列使你可以以任意顺序查询感兴趣的列：

    mysql> SELECT srcuser, srchost, t, size FROM mail;
    +---------+---------+---------------------+---------+
    | srcuser | srchost | t                   | size    |
    +---------+---------+---------------------+---------+
    | barb    | saturn  | 2014-05-11 10:15:08 |   58274 | 
    | tricia  | mars    | 2014-05-12 12:48:13 |  194925 | 
    | phil    | mars    | 2014-05-12 15:02:49 |    1048 | 
    | barb    | saturn  | 2014-05-12 18:59:18 |     271 | 

除非你以某种方式限制SELECT查询，它获取表中每一行。更确切地说，提供一个WHERE语句指定一个或多个条件行必须满足。

条件可以用来测试相等，不相等或相关的顺序。对于一些数据类型，比如字符串，可以使用模式匹配：

    mysql> SELECT t, srcuser, srchost FROM mail WHERE srchost = 'venus';
    +---------------------+---------+---------+
    | t                   | srcuser | srchost |
    +---------------------+---------+---------+
    | 2014-05-14 09:31:37 | gene    | venus   | 
    | 2014-05-14 14:42:21 | barb    | venus   | 
    | 2014-05-15 08:50:57 | phil    | venus   | 
    | 2014-05-16 09:00:28 | gene    | venus   | 
    | 2014-05-16 23:04:19 | phil    | venus   | 
    +---------------------+---------+---------+
    mysql> SELECT t, srcuser, srchost FROM mail WHERE srchost LIKE 's%';
    +---------------------+---------+---------+
    | t                   | srcuser | srchost |
    +---------------------+---------+---------+
    | 2014-05-11 10:15:08 | barb    | saturn  | 
    | 2014-05-12 18:59:18 | barb    | saturn  | 
    | 2014-05-14 17:03:01 | tricia  | saturn  | 
    | 2014-05-15 17:35:31 | gene    | saturn  | 
    | 2014-05-19 22:21:51 | gene    | saturn  | 
    +---------------------+---------+---------+

LIKE操作符执行一个模式匹配，%作为一个通配符匹配任意 字符串。WHERE语句可以测试多个条件且不同条件可以测试不同列：

    mysql> SELECT * FROM mail WHERE srcuser = 'barb' AND dstuser = 'tricia';
    +---------------------+---------+---------+---------+---------+-------+
    | t                   | srcuser | srchost | dstuser | dsthost | size  |
    +---------------------+---------+---------+---------+---------+-------+
    | 2014-05-11 10:15:08 | barb    | saturn  | tricia  | mars    | 58274 | 
    | 2014-05-12 18:59:18 | barb    | saturn  | tricia  | venus   |   271 | 
    +---------------------+---------+---------+---------+---------+-------+

输出列可以由表达式计算：

    mysql> SELECT t, CONCAT(srcuser,'@',srchost), size FROM mail;
    +---------------------+-----------------------------+---------+
    | t                   | CONCAT(srcuser,'@',srchost) | size    |
    +---------------------+-----------------------------+---------+
    | 2014-05-11 10:15:08 | barb@saturn                 |   58274 | 
    | 2014-05-12 12:48:13 | tricia@mars                 |  194925 | 
    | 2014-05-12 15:02:49 | phil@mars                   |    1048 | 
    | 2014-05-12 18:59:18 | barb@saturn                 |     271 | 

# 3.2. 命名查询结果列

**问题**

查询结果中的列名字不合适，丑陋或难以处理。

**解决方法**

使用别名选择你自己的列名字。

**讨论**

当你获取一个结果集，MySQL给每一个输出列一个名字。默认地，MySQL用创建表的列名赋值给输出列，你可以使用别名指定自己的列名。

如果你通过计算一个表达式生成一列，表达式本身就是列名：

    mysql> SELECT
        -> DATE_FORMAT(t, '%M %e, %Y'), CONCAT(srcuser, '@', srchost), size
        -> FROM mail;
    +-----------------------------+-------------------------------+---------+
    | DATE_FORMAT(t, '%M %e, %Y') | CONCAT(srcuser, '@', srchost) | size    |
    +-----------------------------+-------------------------------+---------+
    | May 11, 2014                | barb@saturn                   |   58274 | 
    | May 12, 2014                | tricia@mars                   |  194925 | 
    | May 12, 2014                | phil@mars                     |    1048 | 
    | May 12, 2014                | barb@saturn                   |     271 |

使用AS命名语句指定一列别名（关键字AS是可选的）：

    mysql> SELECT
        -> DATE_FORMAT(t,'%M %e, %Y') AS date_sent,
        -> CONCAT(srcuser,'@',srchost) AS sender,
        -> size FROM mail;
    +--------------+---------------+---------+
    | date_sent    | sender        | size    |
    +--------------+---------------+---------+
    | May 11, 2014 | barb@saturn   |   58274 | 
    | May 12, 2014 | tricia@mars   |  194925 | 
    | May 12, 2014 | phil@mars     |    1048 | 
    | May 12, 2014 | barb@saturn   |     271 | 

别名使列名更准确，更容易阅读和更有意义。别名受到一些限制。比如，如果它们是SQL关键字，数字或包含空格或其它特殊字符，则必须被引用：

    mysql> SELECT
        -> DATE_FORMAT(t,'%M %e, %Y') AS 'Date of message',
        -> CONCAT(srcuser,'@',srchost) AS 'Message sender',
        -> size AS 'Number of bytes' FROM mail;
    mysql> SELECT 1 AS INTEGER;
    You have an error in your SQL syntax near 'INTEGER'
    mysql> SELECT 1 AS 'INTEGER';
    +---------+
    | INTEGER |
    +---------+
    |       1 | 
    +---------+

你不能在WHERE语句中引用列的别名，因此下面这个语句非法：

    mysql> SELECT t, srcuser, dstuser, size/1024 AS kilobytes
        -> FROM mail WHERE kilobytes > 500;

# 3.3. 排序查询结果

**问题**

你的查询结果没有按照你想要的方式排序。

**解决方法**

使用ORDER BY语句告诉MySQL怎样排序结果行。

**讨论**

当你查询行时，MySQL服务器可以自由地以任意顺序返回它们，除非你通知怎么排序它们。有很多中排序的方式。简单地使用ORDER BY语句排序：

    mysql> SELECT * FROM mail WHERE dstuser = 'tricia'
        -> ORDER BY srchost, srcuser;
    +---------------------+---------+---------+---------+---------+--------+
    | t                   | srcuser | srchost | dstuser | dsthost | size   |
    +---------------------+---------+---------+---------+---------+--------+
    | 2014-05-15 10:25:52 | gene    | mars    | tricia  | saturn  | 998532 | 
    | 2014-05-14 11:52:17 | phil    | mars    | tricia  | saturn  |   5781 | 
    | 2014-05-19 12:49:23 | phil    | mars    | tricia  | saturn  |    873 | 
    | 2014-05-11 10:15:08 | barb    | saturn  | tricia  | mars    |  58274 | 
    | 2014-05-12 18:59:18 | barb    | saturn  | tricia  | venus   |    271 | 
    +---------------------+---------+---------+---------+---------+--------+

为了以相反的顺序排序列，在ORDER BY语句后面添加DESC关键字：

    mysql> SELECT * FROM mail WHERE size > 50000 ORDER BY size DESC;
    +---------------------+---------+---------+---------+---------+---------+
    | t                   | srcuser | srchost | dstuser | dsthost | size    |
    +---------------------+---------+---------+---------+---------+---------+
    | 2014-05-14 17:03:01 | tricia  | saturn  | phil    | venus   | 2394482 | 
    | 2014-05-15 10:25:52 | gene    | mars    | tricia  | saturn  |  998532 | 
    | 2014-05-12 12:48:13 | tricia  | mars    | gene    | venus   |  194925 | 
    | 2014-05-14 14:42:21 | barb    | venus   | barb    | venus   |   98151 | 
    | 2014-05-11 10:15:08 | barb    | saturn  | tricia  | mars    |   58274 | 
    +---------------------+---------+---------+---------+---------+---------+

# 3.4. 去除重复行

**问题**

查询结果包含重复行，你想要消除它们。

**解决方法**

使用DISTINCT。

**讨论**

某些查询产生包含重复行的结果：

    mysql> SELECT srcuser FROM mail;
    +---------+
    | srcuser |
    +---------+
    | barb    | 
    | tricia  | 
    | phil    | 
    | barb    | 
    | gene    | 
    | phil    | 
    | barb    | 
    | tricia  | 
    | gene    | 
    | phil    | 
    | gene    | 
    | gene    | 
    | gene    | 
    | phil    | 
    | phil    | 
    | gene    | 
    +---------+

为了删除重复行，产生唯一值的集合，添加DISTINCT到查询：

    mysql> SELECT DISTINCT srcuser FROM mail;
    +---------+
    | srcuser |
    +---------+
    | barb    | 
    | tricia  | 
    | phil    | 
    | gene    | 
    +---------+
    mysql> SELECT COUNT(DISTINCT srcuser) FROM mail;
    +-------------------------+
    | COUNT(DISTINCT srcuser) |
    +-------------------------+
    |                       4 | 
    +-------------------------+

DISTINCT也可以处理多列：

    mysql> SELECT DISTINCT YEAR(t), MONTH(t), DAYOFMONTH(t) FROM mail;
    +---------+----------+---------------+
    | YEAR(t) | MONTH(t) | DAYOFMONTH(t) |
    +---------+----------+---------------+
    |    2014 |        5 |            11 | 
    |    2014 |        5 |            12 | 
    |    2014 |        5 |            14 | 
    |    2014 |        5 |            15 | 
    |    2014 |        5 |            16 | 
    |    2014 |        5 |            19 | 
    +---------+----------+---------------+

# 3.5. 处理NULL值

**问题**

你尝试将列的值与NULL比较，但是不起作用。

**解决方法**

使用合适的比较操作符：IS NULL，IS NOT NULL或<=>。

**讨论**

Conditions that involve NULL are special because NULL means “unknown value.” Con‐
sequently, comparisons such as value = NULL or value <> NULL always produce a result
of NULL (not true or false) because it’s impossible to tell whether they are true or false.
Even NULL = NULL produces NULL because you can’t determine whether one unknown
value is the same as another.

    mysql> SELECT * FROM expt WHERE score = NULL;
    Empty set (0.01 sec)

    mysql> SELECT * FROM expt WHERE score <> NULL;
    Empty set (0.00 sec)

    mysql> SELECT * FROM expt WHERE score IS NULL;
    +---------+------+-------+
    | subject | test | score |
    +---------+------+-------+
    | Jane    | C    |  NULL | 
    | Jane    | D    |  NULL | 
    | Marvin  | D    |  NULL | 
    +---------+------+-------+
    mysql> SELECT * FROM expt WHERE score IS NOT NULL;
    +---------+------+-------+
    | subject | test | score |
    +---------+------+-------+
    | Jane    | A    |    47 | 
    | Jane    | B    |    50 | 
    | Marvin  | A    |    52 | 
    | Marvin  | B    |    45 | 
    | Marvin  | C    |    53 | 
    +---------+------+-------+

The MySQL-specific <=> comparison operator, unlike the = operator, is true even for
two NULL values:

    mysql> SELECT NULL = NULL, NULL <=> NULL;
    +-------------+---------------+
    | NULL = NULL | NULL <=> NULL |
    +-------------+---------------+
    |        NULL |             1 | 
    +-------------+---------------+

Sometimes it’s useful to map NULL values onto some other value that has more meaning
in the context of your application. For example, use IF() to map NULL onto the string
Unknown:

    mysql> SELECT subject, test, IF(score IS NULL,'Unknown',score) AS 'score'
        -> FROM expt;
    +---------+------+---------+
    | subject | test | score   |
    +---------+------+---------+
    | Jane    | A    | 47      | 
    | Jane    | B    | 50      | 
    | Jane    | C    | Unknown | 
    | Jane    | D    | Unknown | 
    | Marvin  | A    | 52      | 
    | Marvin  | B    | 45      | 
    | Marvin  | C    | 53      | 
    | Marvin  | D    | Unknown | 
    +---------+------+---------+

The preceding query can be written more concisely using IFNULL(), which tests its first
argument and returns it if it’s not NULL, or returns its second argument otherwise:

    SELECT subject, test, IFNULL(score,'Unknown') AS 'score'
    FROM expt;

In other words, these two tests are equivalent:

    IF(expr1 IS NOT NULL,expr1,expr2)
    IFNULL(expr1,expr2)

From a readability standpoint, IF() often is easier to understand than IFNULL(). From
a computational perspective, IFNULL() is more efficient because expr1 need not be
evaluated twice, as happens with IF().

# 3.6. Writing Comparisons Involving NULL in Programs

Problem

You’re writing a program that looks for rows containing a specific value, but it fails when
the value is NULL.

Solution

Choose the proper comparison operator according to whether the comparison value is
or is not NULL.

Discussion

A comparison of score = NULL is never true, so that statement returns no rows. To take
into account the possibility that $score could be undef, construct the statement using
the appropriate comparison operator like this:

    $operator = defined ($score) ? "=" : "IS";
    $sth = $dbh->prepare ("SELECT * FROM expt WHERE score $operator ?");
    $sth->execute ($score);

For inequality tests, set $operator like this instead:

    $operator = defined ($score) ? "<>" : "IS NOT";

# 3.7. Using Views to Simplify Table Access

Problem

You want to refer to values calculated from expressions without writing the expressions
each time you retrieve them.

Solution

Use a view defined such that its columns perform the desired calculations.

Discussion

To make the statement results easier to access, use a view, which is a virtual table
that contains no data. Instead, it’s defined as the SELECT statement that retrieves the data
of interest. The following view, mail_view, is equivalent to the SELECT statement just
shown:

    mysql> CREATE VIEW mail_view AS
        -> SELECT
        -> DATE_FORMAT(t,'%M %e, %Y') AS date_sent,
        -> CONCAT(srcuser,'@',srchost) AS sender,
        -> CONCAT(dstuser,'@',dsthost) AS recipient,
        -> size FROM mail;

To access the view contents, refer to it like any other table. You can select some or all of
its columns, add a WHERE clause to restrict which rows to retrieve, use ORDER BY to sort
the rows, and so forth. For example:

    mysql> SELECT date_sent, sender, size FROM mail_view
        -> WHERE size > 100000 ORDER BY size;
    +--------------+---------------+---------+
    | date_sent    | sender        | size    |
    +--------------+---------------+---------+
    | May 12, 2014 | tricia@mars   |  194925 | 
    | May 15, 2014 | gene@mars     |  998532 | 
    | May 14, 2014 | tricia@saturn | 2394482 | 
    +--------------+---------------+---------+

Stored programs provide another way to encapsulate calculations

# 3.8. Selecting Data from Multiple Tables

Problem

The answer to a question requires data from more than one table.

Solution

Use a join or a subquery.

Discussion

The queries shown so far select data from a single table, but sometimes you must retrieve
information from multiple tables. Two types of statements that accomplish this are joins
and subqueries. A join matches rows in one table with rows in another and enables you
to retrieve output rows that contain columns from either or both tables. A subquery is
one query nested within another, to perform a comparison between values selected by
the inner query against values selected by the outer query

    mysql> SELECT id, name, service, contact_name
        -> FROM profile INNER JOIN profile_contact ON id = profile_id;
    +----+---------+----------+--------------+
    | id | name    | service  | contact_name |
    +----+---------+----------+--------------+
    |  1 | Sybil   | Twitter  | user1-twtrid | 
    |  1 | Sybil   | Facebook | user1-fbid   | 
    |  2 | Nancy   | Twitter  | user2-fbrid  | 
    |  2 | Nancy   | Facebook | user2-msnid  | 
    |  2 | Nancy   | LinkedIn | user2-lnkdid | 
    |  4 | Lothair | LinkedIn | user4-lnkdid | 
    +----+---------+----------+--------------+
    mysql> SELECT * FROM profile_contact
        -> WHERE profile_id = (SELECT id FROM profile WHERE name = 'Nancy');
    +------------+----------+--------------+
    | profile_id | service  | contact_name |
    +------------+----------+--------------+
    |          2 | Twitter  | user2-fbrid  | 
    |          2 | Facebook | user2-msnid  | 
    |          2 | LinkedIn | user2-lnkdid | 
    +------------+----------+--------------+

# 3.9. Selecting Rows from the Beginning, End, or Middle of Query Results

Problem

You want only certain rows from a result set, such as the first one, the last five, or rows
21 through 40.

Solution

Use a LIMIT clause, perhaps in conjunction with an ORDER BY clause.

Discussion

MySQL supports a LIMIT clause that tells the server to return only part of a result set.
LIMIT is a MySQL-specific extension to SQL that is extremely valuable when your result
set contains more rows than you want to see at a time. It enables you to retrieve an
arbitrary section of a result set. Typical LIMIT uses include the following kinds of prob‐
lems:

• Answering questions about first or last, largest or smallest, newest or oldest, least
or most expensive, and so forth.
• Splitting a result set into sections so that you can process it one piece at a time. This
technique is common in web applications for displaying a large search result across
several pages. Showing the result in sections enables display of smaller, easier-to-
understand pages.

    mysql> SELECT * FROM profile LIMIT 1;
    +----+-------+------------+-------+----------------------+------+
    | id | name  | birth      | color | foods                | cats |
    +----+-------+------------+-------+----------------------+------+
    |  1 | Sybil | 1970-04-13 | black | lutefisk,fadge,pizza |    0 |
    +----+-------+------------+-------+----------------------+------+
    mysql> SELECT * FROM profile LIMIT 3;
    +----+-------+------------+-------+-----------------------+------+
    | id | name  | birth      | color | foods                 | cats |
    +----+-------+------------+-------+-----------------------+------+
    |  1 | Sybil | 1970-04-13 | black | lutefisk,fadge,pizza  |    0 |
    |  2 | Nancy | 1969-09-30 | white | burrito,curry,eggroll |    3 |
    |  3 | Ralph | 1973-11-02 | red   | eggroll,pizza         |    4 |
    +----+-------+------------+-------+-----------------------+------+

LIMIT n means “return at most n rows.” If you specify LIMIT 10 , and the result set has
only four rows, the server returns four rows.

    mysql> SELECT * FROM profile ORDER BY birth LIMIT 1;
    +----+--------+------------+-------+----------------+------+
    | id | name   | birth      | color | foods          | cats |
    +----+--------+------------+-------+----------------+------+
    |  7 | Joanna | 1952-08-20 | green | lutefisk,fadge |    0 |
    +----+--------+------------+-------+----------------+------+
    mysql> SELECT * FROM profile ORDER BY birth DESC LIMIT 1;
    +----+-------+------------+-------+---------------+------+
    | id | name  | birth      | color | foods         | cats |
    +----+-------+------------+-------+---------------+------+
    |  3 | Ralph | 1973-11-02 | red   | eggroll,pizza |    4 |
    +----+-------+------------+-------+---------------+------+
    mysql> SELECT name, DATE_FORMAT(birth,'%m-%d') AS birthday
        -> FROM profile ORDER BY birthday LIMIT 1;
    +-------+----------+
    | name  | birthday |
    +-------+----------+
    | Henry | 02-14    |
    +-------+----------+
    mysql> SELECT * FROM profile ORDER BY birth LIMIT 2,1;
    +----+---------+------------+-------+---------------+------+
    | id | name    | birth      | color | foods         | cats |
    +----+---------+------------+-------+---------------+------+
    |  4 | Lothair | 1963-07-04 | blue  | burrito,curry |    5 |
    +----+---------+------------+-------+---------------+------+
    mysql> SELECT * FROM profile ORDER BY birth DESC LIMIT 2,1;
    +----+-------+------------+-------+-----------------------+------+
    | id | name  | birth      | color | foods                 | cats |
    +----+-------+------------+-------+-----------------------+------+
    |  2 | Nancy | 1969-09-30 | white | burrito,curry,eggroll |    3 |
    +----+-------+------------+-------+-----------------------+------+

    SELECT * FROM profile ORDER BY name LIMIT 0, 3;
    SELECT * FROM profile ORDER BY name LIMIT 3, 3;
    SELECT * FROM profile ORDER BY name LIMIT 6, 3;

    SELECT SQL_CALC_FOUND_ROWS * FROM profile ORDER BY name LIMIT 4;
    SELECT FOUND_ROWS();

The keyword SQL_CALC_FOUND_ROWS in the first statement tells MySQL to calculate the
size of the entire result set even though the statement requests that only part of it be
returned. The row count is available by calling FOUND_ROWS() . If that function returns
a value greater than three, there are other rows yet to be retrieved.

LIMIT is useful in combination with RAND() to make random selections from a set of
items. See Recipe 15.8.
You can use LIMIT to restrict the effect of a DELETE or UPDATE statement to a subset of
the rows that would otherwise be deleted or updated, respectively. For more information
about using LIMIT for duplicate row removal, see Recipe 16.4.

# 3.10. What to Do When LIMIT Requires the “Wrong” Sort Order

Problem

LIMIT usually works best in conjunction with an ORDER BY clause that sorts rows. But
sometimes that sort order differs from what you want for the final result.

Solution

Use LIMIT in a subquery to retrieve the desired rows, then use the outer query to sort
them.

Discussion

If you want the last four rows of a result set, you can obtain them easily by sorting the
set in reverse order and using LIMIT 4 .What if you want the output rows to appear in ascending order instead?
Use the SELECT as a subquery of an outer statement that re-sorts the rows in the desired
final order:

    mysql> SELECT * FROM
        -> (SELECT name, birth FROM profile ORDER BY birth DESC LIMIT 4) AS t
        -> ORDER BY birth;
    +-------+------------+
    | name  | birth      |
    +-------+------------+
    | Aaron | 1968-09-17 |
    | Nancy | 1969-09-30 |
    | Sybil | 1970-04-13 |
    | Ralph | 1973-11-02 |
    +-------+------------+

AS t is used here because any table referred to in the FROM clause must have a name, even
a “derived” table produced from a subquery.

# 3.11. Calculating LIMIT Values from Expressions

Problem

You want to use expressions to specify the arguments for LIMIT.

Solution

Sadly, you cannot. LIMIT arguments must be literal integers—unless you issue the state‐
ment in a context that permits the statement string to be constructed dynamically. In
that case, you can evaluate the expressions yourself and insert the resulting values into
the statement string.

Discussion

Arguments to LIMIT must be literal integers, not expressions. Statements such as the
following are illegal:

    SELECT * FROM profile LIMIT 5+5;
    SELECT * FROM profile LIMIT @skip_count, @show_count;

The same “no expressions permitted” principle applies if you use an expression to cal‐
culate a LIMIT value in a program that constructs a statement string. You must evaluate
the expression first, and then place the resulting value in the statement. For example, if
you produce a statement string in Perl or PHP as follows, an error will result when you
attempt to execute the statement:

    $str = "SELECT * FROM profile LIMIT $x + $y";

To avoid the problem, evaluate the expression first:

    $z = $x + $y;
    $str = "SELECT * FROM profile LIMIT $z";

Or do this (don’t omit the parentheses or the expression won’t evaluate properly):

    $str = "SELECT * FROM profile LIMIT " . ($x + $y);

