# Chapter 1. Getting Started

---

__Table of Contents__

1. Installation
2. [Architectural Fundamentals](#architectural-fundamentals)
3. [Creating a Database](#creating-a-database)
4. [Accessing a Database](#accessing-a-database)

## Architectural Fundamentals

In database jargon, PostgreSQL uses a client/server model. A PostgreSQL session consists of the following cooperating processes:
- A server process, which manages the database files, accepts connections to the database from client applications, and performs database actions on behalf of the clients. The database server program is called `postgres`
- The user's client (frontend) application that wants to perform database operations. Client applications can be very diverse in nature: a client could be a text-oriented tool, a graphical application, a web server that accesses the database to display web pages, or a specialized database maintenance tool. Some client applications are supplied with the PostgreSQL distribution; most are developed by users.

As is typical of client/server applications, the client and the server can be on different hosts. In that case they communicate over a TCP/IP network connection. 

## Creating a Database

The first test to see whether you can access the database server is to try to create a database. A running PostgreSQL server can manage many databases. Typically, a separate database is used for each project or for each user.

To create a new database, in this example named mydb, you use the following command:

```sh
$ createdb mydb
```

If you do not want to use your database anymore you can remove it. For example, if you are the owner (creator) of the database mydb, you can destroy it using the following command:

```sh
$ dropdb mydb
```

## Accessing a Database

Once you have created a database, you can access it by:
- Running the PostgreSQL interactive terminal program, called psql, which allows you to interactively enter, edit, and execute SQL commands.
- Using an existing graphical frontend tool like pgAdmin or an office suite with ODBC or JDBC support to create and manipulate a database. These possibilities are not covered in this tutorial.
- Writing a custom application, using one of the several available language bindings. These possibilities are discussed further in Part IV.

You probably want to start up psql to try the examples in this tutorial. It can be activated for the mydb database by typing the command:

```sh
$ psql mydb
```

In psql, you will be greeted with the following message:

```sh
psql (14.3)
Type "help" for help.

mydb=>
```

The last line could also be:

```sh
mydb=#
```

That would mean you are a database superuser, which is most likely the case if you installed the PostgreSQL instance yourself. Being a superuser means that you are not subject to access controls. 

The psql program has a number of internal commands that are not SQL commands. They begin with the backslash character, “\”. For example, you can get help on the syntax of various PostgreSQL SQL commands by typing:

```sh
mydb=> \h
```

To get out of psql, type:

```sh
mydb=> \q
```
and psql will quit and return you to your command shell. 


