---
layout: post
title:  "python进阶：sqlchemy介绍及使用"
date:   2018-04-20 13:31:01 +0800
categories: python进阶
tag: sqlchemy
---

* content
{:toc}

### 介绍
    SQLAlchemy是一个基于Python实现的ORM框架。该框架建立在 DB API之上，使用关系对象映射进行数据库操作，简言之便是：将类和对象转换成SQL，然后使用数据API执行SQL并获取执行结果。
### 安装
    pip3 install sqlalchemy
### 架构

![](http://blogdata.zhaolibin.com/Fnr2URf7oy3RiB3pMR2L_GB3c76Z)

 - Engine，框架的引擎
 - Connection Pooling ，数据库连接池
 - Dialect，选择连接数据库的DB API种类
 - Schema/Types，架构和类型
 - SQL Exprression Language，SQL表达式语言
 
 SQLAlchemy本身无法操作数据库，其必须以来pymsql等第三方插件，Dialect用于和数据API进行交流，根据配置文件的不同调用不同的数据库API，从而实现对数据库的操作，如：
 
 ```
 MySQL-Python
    mysql+mysqldb://<user>:<password>@<host>[:<port>]/<dbname>
    
pymysql
    mysql+pymysql://<username>:<password>@<host>/<dbname>[?<options>]
    
MySQL-Connector
    mysql+mysqlconnector://<user>:<password>@<host>[:<port>]/<dbname>
    
cx_Oracle
    oracle+cx_oracle://user:pass@host:port/dbname[?key=value&key=value...]
 ```
 
 https://www.cnblogs.com/ctztake/p/8276246.html
 
 ### 