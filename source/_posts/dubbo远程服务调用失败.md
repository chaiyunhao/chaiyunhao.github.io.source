---
title: dubbo远程服务调用失败
date: 2017-10-11 18:44:24
tags:
    - Dubbo
    - Java
---

## Dubbo是什么
Dubbo是阿里巴巴公司开源的一个高性能优秀的服务框架，使得应用可通过高性能的RPC实现服务的输出和输入功能，以及SOA服务治理方案，和spring框架无缝集成。

## 最近存在的问题
最近公司测试环境进行正常测试的时候，远程调用服务总是有时能成功有时又失败，刚开始的时候问题不是很严重成功几率还是比较高的，一直没能定位的问题的原因，但是已经影响测试人员的正常测试工作。

## 解决思路
1. 刚开始的时候我认为可能是测试环境zookeeper的缓存存在问题，将所有服务进行重新注册应该可以解决问题，所以进行了zookeeper服务的重启，但是没有任何效果。
2. 搜索了一些资料，有一篇文章告诉我服务器的磁盘爆满也可能出现远程服务调用失败的问题，但是我查询了我们的测试服务器，发现磁盘空间还是有很大冗余的,而且如果是这个原因，应该是一直调用失败。[CSDN文章](http://blog.csdn.net/bruce128/article/details/50669053)
3. 随后我发现有一个页面查询，查询结果时而是上一个版本旧的查询结果，时而是新版本的新的查询结果，也就是说可能还有一个老版本的服务还在运行着。
因为本地一直没有搭建dubbo的服务管理平台，今天终于抽出时间从[Github](https://github.com/alibaba/dubbo)上拉取了相关的代码，进行了搭建。搭建完成后，配置了测试环境zookeeper服务地址，服务起来后查看上面注册的服务，因为测试环境使用的是单机，所有不可能同一个服务注册两次。最终发现是由于之前尝试搭建一个额外的测试环境，在pass平台的进行简单的搭建之后，发现可行，但是没有完全搭建完毕，启动了部分服务后就搁置了，但是这部分服务的注册地址错误的使用的之前测试环境的zookeeper地址。所以也就导致的服务偶尔调用失败，以及一些接口使用老的服务的结果，将无关的服务停掉之后，测试环境恢复正常。