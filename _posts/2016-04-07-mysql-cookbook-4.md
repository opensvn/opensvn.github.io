---
layout: post
title: MySQL Cookbook 第4章 表管理
date: 2016-04-07 21:19:39
categories: MySQL
excerpt: 本章介绍如何管理表
---

# 4.0. Introduction

This chapter covers topics that relate to creating and populating tables:
• Cloning a table
• Copying from one table to another
• Using temporary tables
• Generating unique table names
• Determining what storage engine a table uses or converting it from one storage
engine to another

# 4.1. Cloning a Table

Problem

You want to create a table that has exactly the same structure as an existing table.

Solution

Use CREATE TABLE ... LIKE to clone the table structure. To also copy some or all of the
rows from the original table to the new one, use INSERT INTO ... SELECT .

Discussion

To create a new table that is just like an existing table, use this statement:

    CREATE TABLE new_table LIKE original_table;

The structure of the new table is the same as that of the original table, with a few ex‐
ceptions: CREATE TABLE ... LIKE does not copy foreign key definitions, and it doesn’t
copy any DATA DIRECTORY or INDEX DIRECTORY table options that the table might use.

The new table is empty. If you also want the contents to be the same as the original table,
copy the rows using an INSERT INTO ... SELECT statement:

    INSERT INTO new_table SELECT * FROM original_table;

To copy only part of the table, add an appropriate WHERE clause that identifies which
rows to copy. For example, these statements create a copy of the mail table named
mail2 , populated only with the rows for mail sent by barb :

    CREATE TABLE mail2 LIKE mail;
    INSERT INTO mail2 SELECT * FROM mail WHERE srcuser = 'barb';

# 4.2. Saving a Query Result in a Table

Problem

You want to save the result from a SELECT statement to a table rather than display it.

Solution

If the table exists, retrieve rows into it using INSERT INTO ... SELECT . If the table does
not exist, create it on the fly using CREATE TABLE ... SELECT .

Discussion

The MySQL server normally returns the result of a SELECT statement to the client that
executed the statement. For example, when you execute a statement from within the
mysql program, the server returns the result to mysql, which in turn displays it on the
screen. It’s possible to save the results of a SELECT statement in a table instead, which is
useful in several ways:

• You can easily create a complete or partial copy of a table. If you’re developing an
algorithm that modifies a table, it’s safer to work with a copy of a table so that you
need not worry about the consequences of mistakes. If the original table is large,
creating a partial copy can speed the development process because queries run
against it take less time.

• For a data-loading operation based on information that might be malformed, load
new rows into a temporary table, perform some preliminary checks, and correct
the rows as necessary. When you’re satisfied that the new rows are okay, copy them
from the temporary table to your main table.

• Some applications maintain a large repository table and a smaller working table
into which rows are inserted on a regular basis, copying the working table rows to
the repository periodically and clearing the working table.
• To perform summary operations on a large table more efficiently, avoid running
expensive summary operations repeatedly on it. Instead, select summary informa‐
tion once into a second table and use that for further analysis.

    INSERT INTO dst_tbl (i, s) SELECT val, name FROM src_tbl;

The number of columns to be inserted must match the number of selected columns,
with the correspondence between columns based on position rather than name. To copy
all columns, you can shorten the statement to this form:

    INSERT INTO dst_tbl SELECT * FROM src_tbl;

To copy only certain rows, add a WHERE clause that selects those rows:

    INSERT INTO dst_tbl SELECT * FROM src_tbl
    WHERE val > 100 AND name LIKE 'A%';

The SELECT statement can produce values from expressions, too. For example, the fol‐
lowing statement counts the number of times each name occurs in src_tbl and stores
both the counts and the names in dst_tbl :

    INSERT INTO dst_tbl (i, s) SELECT COUNT(*), name
    FROM src_tbl GROUP BY name;

    CREATE TABLE dst_tbl SELECT * FROM src_tbl;

MySQL creates the columns in dst_tbl based on the name, number, and type of the
columns in src_tbl . To copy only certain rows, add an appropriate WHERE clause. To
create an empty table, use a WHERE clause that selects no rows:

    CREATE TABLE dst_tbl SELECT * FROM src_tbl WHERE FALSE;

To create columns in an order different from that in which they appear in the source
table, name them in the desired order. If the source table contains columns a , b , and c
that should appear in the destination table in the order c , a , b , do this:

    CREATE TABLE dst_tbl SELECT c, a, b FROM src_tbl;

To create columns in the destination table in addition to those selected from the source
table, provide appropriate column definitions in the CREATE TABLE part of the statement.
The following statement creates id as an AUTO_INCREMENT column in dst_tbl and adds
columns a , b , and c from src_tbl :

    CREATE TABLE dst_tbl
    (
        id INT NOT NULL AUTO_INCREMENT,
        PRIMARY KEY (id)
    )
    SELECT a, b, c FROM src_tbl;

If you derive a column’s values from an expression, its default name is the expression
itself, which can be difficult to work with later. In this case, it’s prudent to give the column
a better name by providing an alias (see Recipe 3.2). Suppose that src_tbl contains
invoice information that lists items in each invoice. The following statement generates
a summary that lists each invoice named in the table and the total cost of its items, using
an alias for the expression:

    CREATE TABLE dst_tbl
    SELECT inv_no, SUM(unit_cost*quantity) AS total_cost
    FROM src_tbl GROUP BY inv_no;

CREATE TABLE ... SELECT is extremely convenient, but has some limitations that arise
from the fact that the information available from a result set is not as extensive as what
you can specify in a CREATE TABLE statement. For example, MySQL has no idea whether
a result set column should be indexed or what its default value is. If it’s important to
include this information in the destination table, use the following techniques:

• To make the destination table an exact copy of the source table, use the cloning
technique described in Recipe 4.1.
• To include indexes in the destination table, specify them explicitly. For example, if
src_tbl has a PRIMARY KEY on the id column, and a multiple-column index on
state and city , specify them for dst_tbl as well:

    CREATE TABLE dst_tbl (PRIMARY KEY (id), INDEX(state,city))
    SELECT * FROM src_tbl;

• Column attributes such as AUTO_INCREMENT and a column’s default value are not
copied to the destination table. To preserve these attributes, create the table, then
use ALTER TABLE to apply the appropriate modifications to the column definition.
For example, if src_tbl has an id column that is not only a PRIMARY KEY but also
an AUTO_INCREMENT column, copy the table and modify the copy:

    CREATE TABLE dst_tbl (PRIMARY KEY (id)) SELECT * FROM src_tbl;
    ALTER TABLE dst_tbl MODIFY id INT UNSIGNED NOT NULL AUTO_INCREMENT;

# 4.3. Creating Temporary Tables

Problem

You need a table only for a short time, after which you want it to disappear automatically.

Solution

Create a table using the TEMPORARY keyword, and let MySQL take care of removing it.

Discussion

Some operations require a table that exists only temporarily and that should disappear
when it’s no longer needed. You can, of course, execute a DROP TABLE statement explicitly
to remove a table when you’re done with it. Another option is to use CREATE TEMPORA
RY TABLE . This statement is like CREATE TABLE but creates a transient table that disappears
when your session with the server ends, if you haven’t already removed it yourself. This
is extremely useful behavior because MySQL drops the table for you automatically; you
need not remember to do it. TEMPORARY can be used with the usual table-creation meth‐
ods:

• Create the table from explicit column definitions:
CREATE TEMPORARY TABLE tbl_name (...column definitions...);
• Create the table from an existing table:
CREATE TEMPORARY TABLE new_table LIKE original_table;
• Create the table on the fly from a result set:
CREATE TEMPORARY TABLE tbl_name SELECT ... ;

A temporary table can have the same name as a permanent table. In this case, the tem‐
porary table “hides” the permanent table for the duration of its existence, which can be
useful for making a copy of a table that you can modify without affecting the original
by mistake. The DELETE statement in the following example removes rows from a tem‐
porary mail table, leaving the original permanent table unaffected:

mysql> CREATE TEMPORARY TABLE mail SELECT * FROM mail;
mysql> SELECT COUNT(*) FROM mail;
+----------+
| COUNT(*) |
+----------+
|       16 |
+----------+
mysql> DELETE FROM mail;
mysql> SELECT COUNT(*) FROM mail;
+----------+
| COUNT(*) |
+----------+
|        0 |
+----------+
mysql> DROP TEMPORARY TABLE mail;
mysql> SELECT COUNT(*) FROM mail;
+----------+
| COUNT(*) |
+----------+
|       16 |
+----------+
Although temporary tables created with CREATE TEMPORARY TABLE have the benefits just
discussed, keep the following caveats in mind:

• To reuse a temporary table within a given session, you must still drop it explicitly
before re-creating it. Attempting to create a second temporary table with the same
name results in an error.
• If you modify a temporary table that “hides” a permanent table with the same name,
be sure to test for errors resulting from dropped connections if you use a program‐
ming interface that has reconnect capability enabled. If a client program automat‐
ically reconnects after detecting a dropped connection, modifications affect the
permanent table after the reconnect, not the temporary table.
• Some APIs support persistent connections or connection pools. These prevent
temporary tables from being dropped as you expect when your script ends because
the connection remains open for reuse by other scripts. Your script has no control
over when the connection closes. This means it can be prudent to execute the fol‐
lowing statement prior to creating a temporary table, just in case it’s still in existence
from a previous execution of the script:
DROP TEMPORARY TABLE IF EXISTS tbl_name

# 4.4. Generating Unique Table Names

Problem

You need to create a table with a name guaranteed not to exist.

Solution

If you create a TEMPORARY table, it doesn’t matter whether a permanent table with that
name exists. Otherwise, try to generate a value that is unique to your client program
and incorporate it into the table name.

Discussion

If you cannot or do not want to use a TEMPORARY table, make sure that each invocation
of the script creates a uniquely named table and drops the table when it is no longer
needed.

Pro‐
cess ID (PID) values are a better source of unique values. PIDs are reused over time,
but never for two processes at the same time, so a given PID is guaranteed to be unique
among the set of currently executing processes.

Connection identifiers are another source of unique values. The MySQL server reuses
these numbers over time, but no two simultaneous connections to the server have the
same ID.

It’s possible to incorporate a connection ID into a table name within SQL by using
prepared statements. The following example illustrates this, referring to the table name
in the CREATE TABLE statement and a precautionary DROP TABLE statement:

    SET @tbl_name = CONCAT('tmp_tbl_', CONNECTION_ID());
    SET @stmt = CONCAT('DROP TABLE IF EXISTS ', @tbl_name);
    PREPARE stmt FROM @stmt;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
    SET @stmt = CONCAT('CREATE TABLE ', @tbl_name, ' (i INT)');
    PREPARE stmt FROM @stmt;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;

# 4.5. Checking or Changing a Table Storage Engine

Problem

You want to check which storage engine a table uses so that you can determine what
engine capabilities are applicable. Or you need to change a table’s storage engine because
you realize that the capabilities of another engine are more suitable for the way you use
the table.

Solution

To determine a table’s storage engine, you can use any of several statements. To change
the table’s engine, use ALTER TABLE with an ENGINE clause.

Discussion

MySQL supports multiple storage engines, which have differing characteristics. For
example, the InnoDB engine supports transactions, whereas MyISAM does not.

To determine the current engine for a table, check INFORMATION_SCHEMA or use the SHOW
TABLE STATUS or SHOW CREATE TABLE statement. For the mail table, obtain engine in‐
formation as follows:

    mysql> SELECT ENGINE FROM INFORMATION_SCHEMA.TABLES
        -> WHERE TABLE_SCHEMA = 'cookbook' AND TABLE_NAME = 'mail';
    +--------+
    | ENGINE |
    +--------+
    | InnoDB |
    +--------+

    mysql> SHOW TABLE STATUS LIKE 'mail'\G
    *************************** 1. row ***************************
               Name: mail
             Engine: InnoDB
            Version: 10
         Row_format: Compact
               Rows: 16
     Avg_row_length: 1024
        Data_length: 16384
    Max_data_length: 0
       Index_length: 16384
          Data_free: 0
     Auto_increment: NULL
        Create_time: 2016-04-07 21:23:27
        Update_time: NULL
         Check_time: NULL
          Collation: latin1_swedish_ci
           Checksum: NULL
     Create_options: 
            Comment: 
    mysql> SHOW CREATE TABLE mail\G
    *************************** 1. row ***************************
           Table: mail
    Create Table: CREATE TABLE `mail` (
      `t` datetime DEFAULT NULL,
      `srcuser` varchar(8) DEFAULT NULL,
      `srchost` varchar(20) DEFAULT NULL,
      `dstuser` varchar(8) DEFAULT NULL,
      `dsthost` varchar(20) DEFAULT NULL,
      `size` bigint(20) DEFAULT NULL,
      KEY `t` (`t`)
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1

To change the storage engine for a table, use ALTER TABLE with an ENGINE specifier. For
example, to convert the mail table to use the MyISAM storage engine, use this statement:

    ALTER TABLE mail ENGINE = MyISAM;

Be aware that converting a large table to a different storage engine might take a long
time and be expensive in terms of CPU and I/O activity.

To determine which storage engines your MySQL server supports, check the output
from the SHOW ENGINES statement or query the INFORMATION_SCHEMA ENGINES table.

# 4.6. Copying a Table Using mysqldump

Problem

You want to copy a table or tables, either among the databases managed by a MySQL
server, or from one server to another.

Solution

Use the mysqldump program.

Discussion

The mysqldump program makes a backup file that can be reloaded to re-create the
original table or tables:
% mysqldump cookbook mail > mail.sql
The output file mail.sql consists of a CREATE TABLE statement to create the mail table
and a set of INSERT statements to insert its rows. You can reload the file to re-create the
table should the original be lost:
% mysql cookbook < mail.sql

This method also makes it easy to deal with any triggers the table has. By default,
mysqldump writes the triggers to the dump file, so reloading the file copies the triggers
along with the table with no special handling.

## Copying tables within a single MySQL server

• Copy a single table to a different database:
% mysqldump cookbook mail > mail.sql
% mysql other_db < mail.sql
To dump multiple tables, name them all following the database name argument.
• Copy all tables in a database to a different database:
% mysqldump cookbook > cookbook.sql
% mysql other_db < cookbook.sql
When you name no tables after the database name, mysqldump dumps them all. To
also include stored routines and events, add the --routines and --events options
to the mysqldump command. (There is also a --triggers option, but it’s unneeded
because, as mentioned previously, mysqldump dumps triggers with their associated
tables by default.)
• Copy a table, using a different name for the copy:
• Dump the table:
% mysqldump cookbook mail > mail.sql
• Reload the table into a different database that does not contain a table with that
name:
% mysql other_db < mail.sql
• Rename the table:
% mysql other_db
mysql> RENAME mail TO mail2;
Or, to move the table into another database at the same time, qualify the new
name with the database name:
% mysql other_db
mysql> RENAME mail TO cookbook.mail2;

To perform a table-copying operation without an intermediary file, use a pipe to connect
the mysqldump and mysql commands:
% mysqldump cookbook mail | mysql other_db
% mysqldump cookbook | mysql other_db

## Copying tables between MySQL servers

Suppose that you want to copy the mail table from the cook
book database on the local host to the other_db database on the host other-
host.example.com. One way to do this is to dump the output into a file:

% mysqldump cookbook mail > mail.sql

Then copy mail.sql to other-host.example.com, and run the following command there
to load the table into that MySQL server’s other_db database:

% mysql other_db < mail.sql

To accomplish this without an intermediary file, use a pipe to send the output of mysql‐
dump directly over the network to the remote MySQL server. If you can connect to both
servers from your local host, use this command:
% mysqldump cookbook mail | mysql -h other-host.example.com other_db

If you cannot connect directly to the remote server using mysql from your local host,
send the dump output into a pipe that uses ssh to invoke mysql remotely on other-
host.example.com:
% mysqldump cookbook mail | ssh other-host.example.com mysql other_db

