---
layout: post
title: MySQL Cookbook 第5章 处理字符串
date: 2016-04-07 22:45:02
categories: MySQL
excerpt: 本章介绍MySQL的字符串问题
---

# 5.0. Introduction

* A string can be binary or nonbinary. Binary strings are used for raw data such as
images, music files, or encrypted values. Nonbinary strings are used for character
data such as text and are associated with a character set and collation (sort order).
* A character set determines which characters are legal in a string. You can choose
collations according to whether you need comparisons to be case sensitive or case
insensitive, or to use the rules of a particular language.
* Data types for binary strings are BINARY, VARBINARY, and BLOB. Data types for
nonbinary strings are CHAR, VARCHAR, and TEXT, each of which permits CHARACTER
SET and COLLATE attributes.
* You can convert a binary string to a nonbinary string and vice versa, or convert a
nonbinary string from one character set or collation to another.
* You can use a string in its entirety or extract substrings from it. Strings can be
combined with other strings.
* You can apply pattern-matching operations to strings.
* Full-text searching is available for efficient queries on large collections of text.

# 5.1. String Properties

One string property is whether it is binary or nonbinary:

* A binary string is a sequence of bytes. It can contain any type of information, such
as images, MP3 files, or compressed or encrypted data. A binary string is not as‐
sociated with a character set, even if you store a value such as abc that looks like
ordinary text. Binary strings are compared byte by byte using numeric byte values.
* A nonbinary string is a sequence of characters. It stores text that has a particular
character set and collation. The character set defines which characters can be stored
in the string. The collation defines the character ordering, which affects comparison
and sorting operations.

To see which character sets are available for nonbinary strings, use this statement:

    mysql> SHOW CHARACTER SET;
    +----------+-----------------------------+---------------------+--------+
    | Charset  | Description                 | Default collation   | Maxlen |
    +----------+-----------------------------+---------------------+--------+
    | big5     | Big5 Traditional Chinese    | big5_chinese_ci     |      2 | 
    | dec8     | DEC West European           | dec8_swedish_ci     |      1 | 
    | cp850    | DOS West European           | cp850_general_ci    |      1 | 
    | hp8      | HP West European            | hp8_english_ci      |      1 | 

The default character set in MySQL is latin1. If you must store characters from several
languages in a single column, consider using one of the Unicode character sets (such as
utf8 or ucs2) because they can represent characters from multiple languages.

To determine whether a given string contains multibyte characters, use the LENGTH()
and CHAR_LENGTH() functions, which return the length of a string in bytes and charac‐
ters, respectively. If LENGTH() is greater than CHAR_LENGTH() for a given string, multibyte
characters are present:
* The utf8 Unicode character set has multibyte characters, but a given utf8 string
might contain only single-byte characters, as in the following example:
```
    mysql> SET @s = CONVERT('abc' USING utf8);
    mysql> SELECT LENGTH(@s), CHAR_LENGTH(@s);
    +------------+-----------------+
    | LENGTH(@s) | CHAR_LENGTH(@s) |
    +------------+-----------------+
    |          3 |               3 | 
    +------------+-----------------+
```
* For the ucs2 Unicode character set, all characters are encoded using two bytes, even
if they are single-byte characters in another character set such as latin1. Thus,
every ucs2 string contains multibyte characters:
```
    mysql> SET @s = CONVERT('abc' USING ucs2);
    mysql> SELECT LENGTH(@s), CHAR_LENGTH(@s);
    +------------+-----------------+
    | LENGTH(@s) | CHAR_LENGTH(@s) |
    +------------+-----------------+
    |          6 |               3 | 
    +------------+-----------------+
```
Another property of nonbinary strings is collation, which determines the sort order of
characters in the character set. Use SHOW COLLATION to see all available collations; add a
LIKE clause to see the collations for a particular character set:
```
    mysql> SHOW COLLATION LIKE 'latin1%';
    +-------------------+---------+----+---------+----------+---------+
    | Collation         | Charset | Id | Default | Compiled | Sortlen |
    +-------------------+---------+----+---------+----------+---------+
    | latin1_german1_ci | latin1  |  5 |         | Yes      |       1 | 
    | latin1_swedish_ci | latin1  |  8 | Yes     | Yes      |       1 | 
    | latin1_danish_ci  | latin1  | 15 |         | Yes      |       1 | 
    | latin1_german2_ci | latin1  | 31 |         | Yes      |       2 | 
    | latin1_bin        | latin1  | 47 |         | Yes      |       1 | 
    | latin1_general_ci | latin1  | 48 |         | Yes      |       1 | 
    | latin1_general_cs | latin1  | 49 |         | Yes      |       1 | 
    | latin1_spanish_ci | latin1  | 94 |         | Yes      |       1 | 
    +-------------------+---------+----+---------+----------+---------+
```
In contexts where no collation is specified explicitly, strings in a given character set use
the collation with Yes in the Default column. As shown, the default collation for lat
in1 is latin1_swedish_ci. (Default collations are also displayed by SHOW CHARACTER
SET.)

A collation can be case sensitive (a and A are different), case insensitive (a and A are the
same), or binary (two characters are the same or different based on whether their nu‐
meric values are equal). A collation name ending in _ci, _cs, or _bin is case insensitive,
case sensitive, or binary, respectively.

The following example illustrates how collation affects sort order. Suppose that a table
contains a latin1 string column and has the following rows:

    mysql> CREATE TABLE t (c CHAR(3) CHARACTER SET latin1);
    mysql> INSERT INTO t (c) VALUES('AAA'),('bbb'),('aaa'),('BBB');
    mysql> SELECT c FROM t;
    +------+
    | c    |
    +------+
    | AAA  | 
    | bbb  | 
    | aaa  | 
    | BBB  | 
    +------+

By applying the COLLATE operator to the column, you can choose which collation to use
for sorting and thus affect the order of the result:

* A case-insensitive collation sorts a and A together, placing them before b and B.
However, for a given letter, it does not necessarily order one lettercase before an‐
other, as shown by the following result:
```
    mysql> SELECT c FROM t ORDER BY c COLLATE latin1_swedish_ci;
    +------+
    | c    |
    +------+
    | AAA  | 
    | aaa  | 
    | bbb  | 
    | BBB  | 
    +------+
```
* A case-sensitive collation puts A and a before B and b, and sorts uppercase before
lowercase:
```
    mysql> SELECT c FROM t ORDER BY c COLLATE latin1_general_cs;
    +------+
    | c    |
    +------+
    | AAA  | 
    | aaa  | 
    | BBB  | 
    | bbb  | 
    +------+
```
* A binary collation sorts characters using their numeric values. Assuming that up‐
percase letters have numeric values less than those of lowercase letters, a binary
collation results in the following ordering:
```
    mysql> SELECT c FROM t ORDER BY c COLLATE latin1_bin;
    +------+
    | c    |
    +------+
    | AAA  | 
    | BBB  | 
    | aaa  | 
    | bbb  | 
    +------+
```
If you require that comparison and sorting operations use the sorting rules of a particular
language, choose a language-specific collation. For example, if you store strings using
the utf8 character set, the default collation (utf8_general_ci) treats ch and ll as twocharacter strings. To use the traditional Spanish ordering that treats ch and ll as single
characters that follow c and l, respectively, specify the utf8_spanish2_ci collation. The
two collations produce different results, as shown here:

```
    mysql> CREATE TABLE t (c CHAR(2) CHARACTER SET utf8);
    mysql> INSERT INTO t (c) VALUES('cg'),('ch'),('ci'),('lk'),('ll'),('lm');
    mysql> SELECT c FROM t ORDER BY c COLLATE utf8_general_ci;
    mysql> SELECT c FROM t ORDER BY c COLLATE utf8_spanish2_ci;
```

# 5.2. Choosing a String Data Type

Problem

You want to store string data but aren’t sure which is the most appropriate data type.

Solution

Choose the data type according to the characteristics of the information to be stored
and how you need to use it. Consider questions such as these:
* Are the strings binary or nonbinary?
* Does case sensitivity matter?
* What is the maximum string length?
* Do you want to store fixed- or variable-length values?
* Do you need to retain trailing spaces?
* Is there a fixed set of permitted values?

Discussion

MySQL provides several binary and nonbinary string data types. These types come in
pairs as shown in the following table. The maximum length is in bytes, whether the type
is binary or nonbinary. For nonbinary types, the maximum number of characters is less
for strings that contain multibyte characters:


| Binary data type | Nonbinary data type | Maximum length |
| -- | -- | -- |
| BINARY | CHAR | 255 |
| VARBINARY | VARCHAR | 65,535 |
| TINYBLOB | TINYTEXT | 255 |
| BLOB | TEXT | 65,535 |
| MEDIUMBLOB | MEDIUMTEXT | 16,777,215 |
| LONGBLOB | LONGTEXT | 4,294,967,295 |


For the BINARY and CHAR data types, MySQL stores column values using a fixed width.
For example, values stored in a BINARY(10) or CHAR(10) column always take 10 bytes
or 10 characters, respectively. Shorter values are padded to the required length as nec‐essary when stored. For BINARY, the pad value is 0x00 (the zero-valued byte, also known
as ASCII NUL). CHAR values are padded with spaces for storage and trailing spaces are
stripped upon retrieval.
For VARBINARY, VARCHAR, and the BLOB and TEXT types, MySQL stores values using only
as much storage as required, up to the maximum column length. No padding is added
or stripped when values are stored or retrieved.
To preserve trailing pad values that are present in the original strings that are stored,
use a data type for which no stripping occurs. For example, if you store character (non‐
binary) strings that might end with spaces, and want to preserve them, use VARCHAR or
one of the TEXT data types. The following statements illustrate the difference in trailingspace handling for CHAR and VARCHAR columns:

    mysql> CREATE TABLE t (c1 CHAR(10), c2 VARCHAR(10));
    mysql> INSERT INTO t (c1,c2) VALUES('abc ','abc ');
    mysql> SELECT c1, c2, CHAR_LENGTH(c1), CHAR_LENGTH(c2) FROM t;
    +------+------+-----------------+-----------------+
    | c1   | c2   | CHAR_LENGTH(c1) | CHAR_LENGTH(c2) |
    +------+------+-----------------+-----------------+
    | abc  | abc  |               3 |               4 | 
    +------+------+-----------------+-----------------+

A table can include a mix of binary and nonbinary string columns, and its nonbinary
columns can use different character sets and collations. When you declare a nonbinary
string column, use the CHARACTER SET and COLLATE attributes if you require a particular
character set and collation. For example, if you need to store utf8 (Unicode) and sjis
(Japanese) strings, you might define a table with two columns like this:

    CREATE TABLE mytbl
    (
        utf8str VARCHAR(100) CHARACTER SET utf8 COLLATE utf8_danish_ci,
        sjisstr VARCHAR(100) CHARACTER SET sjis COLLATE sjis_japanese_ci
    );

The CHARACTER SET and COLLATE clauses are each optional in a column definition:
* If you specify CHARACTER SET and omit COLLATE, the default collation for the char‐
acter set is used.
* If you specify COLLATE and omit CHARACTER SET, the character set implied by the
collation name (the first part of the name) is used. For example, utf8_danish_ci
and sjis_japanese_ci imply utf8 and sjis, respectively. This means that the
CHARACTER SET attributes could have been omitted from the preceding CREATE TABLE
statement.

* If you omit both CHARACTER SET and COLLATE, the column is assigned the table
default character set and collation. A table definition can include those attributes
following the closing parenthesis at the end of the CREATE TABLE statement. If
present, they apply to columns that have no explicit character set or collation of
their own. If omitted, the table defaults are taken from the database defaults. You
can specify the database defaults when you create the database with the CREATE
DATABASE statement. The server defaults apply to the database if they are omitted.

The server default character set and collation are latin1 and latin1_swedish_ci, so
strings by default use the latin1 character set and are not case sensitive. To change this,
set the character_set_server and collation_server system variables at server start‐
up 

MySQL also supports ENUM and SET string types, which are used for columns that have
a fixed set of permitted values. The CHARACTER SET and COLLATE attributes apply to these
data types as well.

# 5.3. Setting the Client Connection Character Set

Problem

You’re executing SQL statements or producing query results that don’t use the default
character set.

Solution

Use SET NAMES or an equivalent method to set your connection to the proper character
set

Discussion

To deal with this problem, configure your connection to use the appropriate character
set. You have several ways to do this:

* Issue a SET NAMES statement after you connect:
```
    mysql> SET NAMES 'utf8';
```
SET NAMES permits the connection collation to be specified as well:
```
    mysql> SET NAMES 'utf8' COLLATE 'utf8_general_ci';
```
* If your client program supports the --default-character-set option, you can use
it to specify the character set at program invocation time. mysql is one such program.
Put the option in an option file so that it takes effect each time you connect to the
server:
```
    [mysql]
    default-character-set=utf8
```
* If you set the environment for your working environment using the LANG or LC_ALL
environment variable on Unix, or the code page setting on Windows, MySQL client
programs automatically detect which character set to use. For example, setting
LC_ALL to en_US.UTF-8 causes programs such as mysql to use utf8.

* Some programming interfaces provide their own method of setting the character
set. For example, MySQL Connector/J for Java clients detects the character set used
on the server side automatically when you connect, but you can specify a different
set explicitly using the characterEncoding property in the connection URL. The
property value should be the Java-style character-set name. To select utf8, you
might use a connection URL like this:
```
    jdbc:mysql://localhost/cookbook?characterEncoding=UTF-8
```
This is preferable to SET NAMES because Connector/J performs character-set con‐
version on behalf of the application, but is unaware of which character set applies
if you use SET NAMES. Similar principles apply to programs written for other APIs.
For PDO, use a charset option in your data source name (DSN) string (this works
in PHP 5.3.6 or later):
```
    $dsn = "mysql:host=localhost;dbname=cookbook;charset=utf8";
```
For Connector/Python, specify a charset connection parameter:
```
    conn_params = {
    "database": "cookbook",
    "host": "localhost",
    "user": "cbuser",
    "password": "cbpass",
    "charset": "utf8",
    }
```
Some APIs may also provide a parameter to specify the collation.

# 5.4. Writing String Literals

Problem

You need to write literal strings in SQL statements.

Solution

Learn the syntax rules that govern string values.

Discussion

You can write strings several ways:

* Enclose the text of the string within single quotes or double quotes:
'my string'
"my string"
When the ANSI_QUOTES SQL mode is enabled, you cannot use double quotes for
quoting strings: the server interprets double quote as the quoting character for
identifiers such as table or column names, and not for strings

* Use hexadecimal notation. Each pair of hex digits produces one byte of the string.
abcd can be written using any of these formats:
0x61626364
X'61626364'
x'61626364'
MySQL treats strings written using hex notation as binary strings. Not coinciden‐
tally, it’s common for applications to use hex strings when constructing SQL state‐
ments that refer to binary values:

INSERT INTO t SET binary_col = 0xdeadbeef;

* To specify a character set for interpretation of a literal string, use an introducer
consisting of a character-set name preceded by an underscore:
_latin1 'abcd'
_ucs2 'abcd'
An introducer tells the server how to interpret the string that follows it. For _lat
in1 'abcd', the server produces a string consisting of four single-byte characters.
For _ucs2 'abcd', the server produces a string consisting of two two-byte characters
because ucs2 is a double-byte character set.

A quoted string that includes the same quote character produces a syntax error:

    mysql> SELECT 'I'm asleep';
    ERROR 1064 (42000): You have an error in your SQL syntax near 'asleep''

You have several ways to deal with this:
* Enclose a string containing single quotes within double quotes (assuming that
ANSI_QUOTES is disabled), or enclose a string containing double quotes within single
quotes:
```
    mysql> SELECT "I'm asleep", 'He said, "Boo!"';
    +------------+-----------------+
    | I'm asleep | He said, "Boo!" |
    +------------+-----------------+
    | I'm asleep | He said, "Boo!" | 
    +------------+-----------------+
```
* To include a quote character within a string quoted by the same kind of quote,
double the quote or precede it with a backslash. When MySQL reads the statement,
it strips the extra quote or the backslash:
```
    mysql> SELECT 'I''m asleep', 'I\'m wide awake';
    +------------+----------------+
    | I'm asleep | I'm wide awake |
    +------------+----------------+
    | I'm asleep | I'm wide awake | 
    +------------+----------------+
    mysql> SELECT "He said, ""Boo!""", "And I said, \"Yikes!\"";
    +-----------------+----------------------+
    | He said, "Boo!" | And I said, "Yikes!" |
    +-----------------+----------------------+
    | He said, "Boo!" | And I said, "Yikes!" | 
    +-----------------+----------------------+
```
A backslash turns off any special meaning of the following character, including
itself. To write a literal backslash within a string, double it:
```
    mysql> SELECT 'Install MySQL in C:\\mysql on Windows';
    +--------------------------------------+
    | Install MySQL in C:\mysql on Windows |
    +--------------------------------------+
    | Install MySQL in C:\mysql on Windows | 
    +--------------------------------------+
```
* Write the string as a hex value:
```
    mysql> SELECT 0x49276D2061736C656570;
    +------------------------+
    | 0x49276D2061736C656570 |
    +------------------------+
    | I'm asleep             | 
    +------------------------+
```

# 5.5. Checking or Changing a String’s Character Set or Collation

Problem

You want to know the character set or collation of a string, or change a string to some
other character set or collation.

Solution

To check a string’s character set or collation, use the CHARSET() or COLLATION() func‐
tion. To change its character set, use the CONVERT() function. To change its collation,
use the COLLATE operator.

Discussion

To determine a string’s character set or collation, use the CHARSET() or COLLATION()
function. For example, did you know that the USER() function returns a Unicode string?

    mysql> SELECT USER(), CHARSET(USER()), COLLATION(USER());
    +------------------+-----------------+-------------------+
    | USER()           | CHARSET(USER()) | COLLATION(USER()) |
    +------------------+-----------------+-------------------+
    | cbuser@localhost | utf8            | utf8_general_ci   | 
    +------------------+-----------------+-------------------+

String values that take their character set and collation from the current configuration
may change properties if the configuration changes. This is true for literal strings:

    mysql> SET NAMES 'latin1';
    mysql> SELECT CHARSET('abc'), COLLATION('abc');
    +----------------+-------------------+
    | CHARSET('abc') | COLLATION('abc')  |
    +----------------+-------------------+
    | latin1         | latin1_swedish_ci | 
    +----------------+-------------------+
    mysql> SET NAMES utf8 COLLATE 'utf8_bin';
    mysql> SELECT CHARSET('abc'), COLLATION('abc');
    +----------------+------------------+
    | CHARSET('abc') | COLLATION('abc') |
    +----------------+------------------+
    | utf8           | utf8_bin         | 
    +----------------+------------------+

For a binary string, the CHARSET() or COLLATION() functions return a value of binary,
which means that the string is compared and sorted based on numeric byte values, not
character collation values.
To convert a string from one character set to another, use the CONVERT() function:

    mysql> SET @s1 = _latin1 'my string', @s2 = CONVERT(@s1 USING utf8);
    mysql> SELECT CHARSET(@s1), CHARSET(@s2);
    +--------------+--------------+
    | CHARSET(@s1) | CHARSET(@s2) |
    +--------------+--------------+
    | latin1       | utf8         | 
    +--------------+--------------+

To change the collation of a string, use the COLLATE operator:

    mysql> SET @s1 = _latin1 'my string', @s2 = @s1 COLLATE latin1_spanish_ci;
    mysql> SELECT COLLATION(@s1), COLLATION(@s2);
    +-------------------+-------------------+
    | COLLATION(@s1)    | COLLATION(@s2)    |
    +-------------------+-------------------+
    | latin1_swedish_ci | latin1_spanish_ci | 
    +-------------------+-------------------+

The new collation must be legal for the character set of the string. For example, you can
use the utf8_general_ci collation with utf8 strings, but not with latin1 strings:

    mysql> SELECT _latin1 'abc' COLLATE utf8_bin;
    ERROR 1253 (42000): COLLATION 'utf8_bin' is not valid for
    CHARACTER SET 'latin1'

To convert both the character set and collation of a string, use CONVERT() to change the
character set, and apply the COLLATE operator to the result:

    mysql> SET @s1 = _latin1 'my string';
    mysql> SET @s2 = CONVERT(@s1 USING utf8) COLLATE utf8_spanish_ci;
    mysql> SELECT CHARSET(@s1), COLLATION(@s1), CHARSET(@s2), COLLATION(@s2);
    +--------------+-------------------+--------------+-----------------+
    | CHARSET(@s1) | COLLATION(@s1)    | CHARSET(@s2) | COLLATION(@s2)  |
    +--------------+-------------------+--------------+-----------------+
    | latin1       | latin1_swedish_ci | utf8         | utf8_spanish_ci | 
    +--------------+-------------------+--------------+-----------------+

The CONVERT() function can also convert binary strings to nonbinary strings and vice
versa. To produce a binary string, use binary; any other character-set name produces
a nonbinary string:

    mysql> SET @s1 = _latin1 'my string';
    mysql> SET @s2 = CONVERT(@s1 USING binary);
    mysql> SET @s3 = CONVERT(@s2 USING utf8);
    mysql> SELECT CHARSET(@s1), CHARSET(@s2), CHARSET(@s3);
    +--------------+--------------+--------------+
    | CHARSET(@s1) | CHARSET(@s2) | CHARSET(@s3) |
    +--------------+--------------+--------------+
    | latin1       | binary       | utf8         | 
    +--------------+--------------+--------------+

Alternatively, produce binary strings using the BINARY operator, which is equivalent to
CONVERT(str USING binary):

    mysql> SELECT CHARSET(BINARY _latin1 'my string');
    +-------------------------------------+
    | CHARSET(BINARY _latin1 'my string') |
    +-------------------------------------+
    | binary                              | 
    +-------------------------------------+

# 5.6. Converting the Lettercase of a String

Problem

You want to convert a string to uppercase or lowercase.

Solution

Use the UPPER() or LOWER() function. If they don’t work, you’re probably trying to
convert a binary string. Convert it to a nonbinary string that has a character set and
collation and is subject to case mapping.

Discussion

The UPPER() and LOWER() functions convert the lettercase of a string:

    mysql> SELECT thing, UPPER(thing), LOWER(thing) FROM limbs;
    +--------------+--------------+--------------+
    | thing        | UPPER(thing) | LOWER(thing) |
    +--------------+--------------+--------------+
    | human        | HUMAN        | human        | 
    | insect       | INSECT       | insect       | 
    | squid        | SQUID        | squid        | 
    | fish         | FISH         | fish         | 

But some strings are “stubborn” and resist lettercase conversion:

    mysql> CREATE TABLE t (b BLOB) SELECT 'aBcD' AS b;
    mysql> SELECT b, UPPER(b), LOWER(b) FROM t;
    +------+----------+----------+
    | b    | UPPER(b) | LOWER(b) |
    +------+----------+----------+
    | aBcD | aBcD     | aBcD     | 
    +------+----------+----------+

This problem occurs for strings that have a BINARY or BLOB data type. These are binary
strings that have no character set or collation. Lettercase does not apply, and UPPER()
and LOWER() do nothing.

To map a binary string to a given lettercase, convert it to a nonbinary string, choosing
a character set that has uppercase and lowercase characters. The case-conversion func‐
tions then work as you expect because the collation provides case mapping

    mysql> SELECT b,
        -> UPPER(CONVERT(b USING latin1)) AS upper,
        -> LOWER(CONVERT(b USING latin1)) AS lower
        -> FROM t;
    +------+-------+-------+
    | b    | upper | lower |
    +------+-------+-------+
    | aBcD | ABCD  | abcd  | 
    +------+-------+-------+

To convert the lettercase of only part of a string, break it into pieces, convert the relevant
piece, and put the pieces back together. Suppose that you want to convert only the initial
character of a string to uppercase. The following expression accomplishes that:

CONCAT(UPPER(LEFT(str,1)),MID(str,2))

But it’s ugly to write an expression like that each time you need it. For convenience,
define a stored function:

    mysql> CREATE FUNCTION initial_cap (s VARCHAR(255))
        -> RETURNS VARCHAR(255) DETERMINISTIC
        -> RETURN CONCAT(UPPER(LEFT(s,1)),MID(s,2));

Then you can capitalize initial characters more easily:

    mysql> SELECT thing, initial_cap(thing) FROM limbs;
    +--------------+--------------------+
    | thing        | initial_cap(thing) |
    +--------------+--------------------+
    | human        | Human              | 
    | insect       | Insect             | 
    | squid        | Squid              | 
    | fish         | Fish               | 
    | centipede    | Centipede          | 
    | table        | Table              | 
    | armchair     | Armchair           | 
    | phonograph   | Phonograph         | 
    | tripod       | Tripod             | 
    | Peg Leg Pete | Peg Leg Pete       | 
    | space alien  | Space alien        | 
    +--------------+--------------------+
