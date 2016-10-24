---
layout: post
title: MySQL Cookbook 第2章 编写基于MySQL的程序
date: 2016-04-05 21:32:29
categories: MySQL
excerpt: 本章介绍如何编写基于MySQL的程序
---

# 2.0. 介绍

## MySQL Client API Architecture

Each MySQL programming interface covered in this book uses a two-level architecture:

* The upper level provides database-independent methods that implement database
access in a portable way that’s the same whether you use MySQL, PostgreSQL, Ora‐
cle, or whatever.
* The lower level consists of a set of drivers, each of which implements the details for
a single database system.

# 2.1. Connecting, Selecting a Database, and Disconnecting

Problem

You need to establish a connection to the database server and shut down the connection
when you’re done.

Solution

Each API provides routines for connecting and disconnecting. The connection routines
require that you provide parameters specifying the host on which the MySQL server is
running and the MySQL account to use. You can also select a default database.

Discussion

Establishing a connection to the MySQL server
    Every program that uses MySQL does this, no matter which API you use. The details
    on specifying connection parameters vary between APIs, and some APIs provide
    more flexibility than others. However, there are many common parameters, such
    as the host on which the server is running, and the username and password of the
    MySQL account to use for accessing the server.

Selecting a database
    Most MySQL programs select a default database.

Disconnecting from the server
    Each API provides a way to close an open connection. It’s best to do so as soon as
    you’re done using the server. If your program holds the connection open longer
    than necessary, the server cannot free up resources allocated to servicing the con‐
    nection. It’s also preferable to close the connection explicitly. If a program simply
    terminates, the MySQL server eventually notices, but an explicit close on the user
    end enables the server to perform an immediate orderly close on its end.

This section includes example programs that show how to use each API to connect to
the server, select the cookbook database, and disconnect. The discussion for each API
also indicates how to connect without selecting any default database. This might be the
case if you plan to execute a statement that doesn’t require a default database, such as
SHOW VARIABLES or SELECT VERSION() . Or perhaps you’re writing a program that enables
the user to specify the database after the connection has been made.

# 2.2. Checking for Errors

Problem

Something went wrong with your program, and you don’t know what.

Solution

Everyone has problems getting programs to work correctly. But if you don’t anticipate
problems by checking for errors, the job becomes much more difficult. Add some error-
checking code so your programs can help you figure out what went wrong.

Discussion

It’s
also a good idea to know how to check for errors and how to retrieve specific error
information from the API

When an error occurs, MySQL provides three values:

* A MySQL-specific error number
* A MySQL-specific descriptive text error message
* A five-character SQLSTATE error code defined according to the ANSI and ODBC
standards



