---
layout: post
title: 【慕课网】Java高并发秒杀API（一）：业务分析与DAO层
author: 孟凡泽
date: 2016-05-21 22:05:40 +0800
tag: [Java, Web后台]
categories: 笔记 @慕课网
---

注：这是我在看完慕课网上的视频后记的笔记，从零开始介绍了如何开发一个高并发的商品秒杀网站。

我觉得这是我在慕课网上看得最有价值的视频之一了，从中可以学习到很多项目构建的思路和编码技巧。唯一遗憾的是，这是用Java写的，而我没有用Java写过网站，所以对其中的很多Java依赖库丝毫不了解。不过没有关系，语言只是工具，思想都是相通的。把它放在PHP或者ASP.Net上一定会有类似的框架和方案可以实现同样的功能。也正因如此，若直接看我的这篇博客，可能什么都学不到。这篇博客是为我日后回顾做记录的。想学习相关的内容还是从看慕课网上yijun zhang老师的视频开始吧。

传送门：[http://www.imooc.com/u/2145618/courses?sort=publish](http://www.imooc.com/u/2145618/courses?sort=publish)

## 课程介绍

基于SpringMVC + Spring + MyBatis。

秒杀业务：

- 具有典型“事务”特性
- 秒杀/红包类应用越来越多
- 面试常问

## 相关技术介绍

* MySQL：表设计、SQL技巧、事务和行级锁
* MyBatis：DAO层设计与开发、合理使用MyBatis、与Spring的整合
* Spring：Spring IOC整合Service、声明式事务运用
* SpringMVC：Restful接口设计和使用、框架运作流程、Controller开发技巧
* 前端：交互设计、Bootstrap、JQuery
* 高并发：高并发点和分析、优化思路和实现。

## 创建项目

从零开始创建、从官网获取资源、使用Maven创建项目然后导入到IntelliJ IDEA工程。

注意使用更高的Servlet版本。

目录：

```
main
--resources
    --mapper
        SeckillDao.xml
        SuccessKilledDao.xml
    --spring
        spring-dao.xml
    mybatis-config.xml
--webapp
--java
    --org.seckill
        --dao
        --entity
--sql

test
--java
--resources
```

## 配置依赖：

0. junit 4.11

1. 日志：java常用：slf4j，log4j，logback，common-logging。
    - slf4j(接口) + logback(实现)

2. 数据库相关：
    - 驱动：mysql-connector-java
    - 连接池：c3p0
    - DAO框架：MyBatis
    
3. Servlet Web相关依赖
    - 标签：taglibs
    - 默认标签库：jstl
    - jackson
    - javax.servlet-api

4. spring依赖：
    - 核心依赖：spring-core
    - IOC：spring-beans
    - 扩展：spring-context
    - DAO层依赖：spring-jdbc
    - 事务相关：spring-tx
    - Spring Web相关依赖：spring-web
    - spring-webmvc
    - spring test相关依赖：spring-test

## 秒杀业务的分析

核心：用户和商家对库存的处理。

用户：减库存、记录购买明细 => 事务 => 数据落地（MySQL Vs NoSQL）

事务机制依然是目前最可靠的数据落地方案。

难点：“竞争”。MySQL：事务 + 行级锁。

## 实现哪些秒杀功能

- 秒杀接口暴露
- 执行秒杀
- 相关查询

开发阶段：

- DAO设计编码
- Service设计编码
- Web设计编码

## 编码

### 数据库设计与编码

#### DDL

- 秒杀库存表：`seckill(seckill_id, name, number, start_time, end_time, create_time)`

MySQL使用InnoDB引擎（支持事务）。
在start_time、end_time、create_time上建立索引。

- 秒杀成功明细表：`success_killed(seckill_id, user_phone, state, create_time)`

使用联合主键：seckill_id, user_phone
在create_time上建立索引。
state: -1无效，0成功，1已付款....

养成手写DDL的习惯。

#### DAO实体和接口编码

- 编写Entity类。（技巧：IDE生成getter/setter/tostring）

- 设计DAO接口：实体的CRUD。
    - SeckillDao：
        - `int reduceNumber(long seckillId, Data killTime)`
        - `Seckill queryById(long seckillId)`
        - `List<Seckill> queryAll(int offset, int limit)`
    - SuccessKilledDao:
        - `int insertSuccessKilled(SuccessKilled successKilled)`（没必要传整个实体，long seckillId, long userPhone即可）
        - `SuccessKilled queryByIdWithSeckill(long seckillId)`
    
- 基于myBatis（或HiberNate）实现DAO。写mapper，为DAO接口方法提供SQL语句配置。

- myBatis整合Spring：
    - 更少的编码：只写接口，不写实现
    - 更少的配置：别名（省去包名）、配置扫描、DAO实现 
    - 足够的灵活性。（自己定制SQL、自由传参、结果集自动复制）

    1. 配置数据库相关参数。
    2. 数据库连接池c3p0。（连接池属性：数据库连接参数、maxPoolSize、minPoolSize、autoCommitOnClose、checkoutTimeout、acquireRetryAttempts）
    3. 配置sqlSessionFactory对象：注入数据库连接池、配置MyBatis全局配置文件、扫描entity包并使用别名、扫描sql配置文件(mapper/*.xml)。
    4. 配置扫描DAO接口包，动态实现DAO接口，注入到spring容器中：注入sqlSessionFactory、给出扫描DAO接口包。

#### DAO层单元测试和问题排查

* 使用IDE自动生成SeckillDaoTest()方法。
* 配置spring和junit整合，junit启动时加载springIOC容器。告诉junit spring配置文件。
* 注入DAO实现类依赖。（就是声明个成员变量…）
* 编写测试方法。

涨姿势：Java没有保存形参的记录，要对形参使用@params注解。



