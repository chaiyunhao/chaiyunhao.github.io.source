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


#### 接口和映射文件的说明
1. 映射文件为xml格式的文件，以mapper为根元素，namespace的值为接口的全限定类名，同具体的接口映射绑定。
2. 映射文件中select|insert|update|delete代表的是Sql语句，每个Sql语句通过id同接口文件中的对应的方法进行绑定。
3. 映射文件中的每个Sql语句可以包含对应的入参和出参，用于完成和java对象的转换，入参和出参同接口中对应方法的参数和返回值对应。
4. 在Mybatis初始化的时候，每一个语句都会使用对应的MappedStatement代表，使用namespace+语句本身的id来代表这个语句。
5. 在Mybatis执行时，会进入对应接口的方法，通过类名加上方法名的组合生成id，找到需要的MappedStatement，交给执行器使用。


#### Mybatis 缓存
MyBatis将数据缓存设计成两级结构，分为一级缓存、二级缓存：
1. 一级缓存默认为开启状态，为SESSION级别，即在一个Mybatis会话中执行的所有语句，都会共享这一个缓存；还有一种是STATEMENT级别，可以理解为缓存只对当前执行的这一个statement有效，也就是不开启缓存。
