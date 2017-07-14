---
title: mybatis缓存与源码分析
date: 2017-07-14 10:44:51
comments: true
tags:
      - Sql
      - Java
---

## Mybatis的基础概念

#### Mybatis是什么

Mybatis是支持定制化SQL、存储过程以及高级映射的优秀的持久层框架。Mybatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集。Mybatis可以对配置和原生Map使用简单的XML或注解，将接口和Java和POJOs（Plain Old Java Objects，普通Java对象）映射成数据库中的记录。

#### 几个核心的概念
1. SqlSession：代表和数据库是的一次回话，向用户提供了操作数据库的方法。
2. MappedStatement：代表要发往数据库执行的指令，可以理解为Sql的抽象表示。
3. Executor:具体用来和数据库交互的执行器，接受MappedStatement作为参数。
4. 映射接口：在接口中会将要执行的Sql用一个方法来表示，具体的Sql写在映射文件中。
5. 映射文件：可以理解为Mybatis编写Sql的地方，通常说每一张表都会对应一个映射文件，在该文件中会定义Sql语句的入参和出参。
