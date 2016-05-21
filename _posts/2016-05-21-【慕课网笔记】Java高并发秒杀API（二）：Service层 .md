---
layout: post
title: 【慕课网笔记】Java高并发秒杀API（二）：Service层
author: 孟凡泽
date: 2016-05-21 23:05:40 +0800
tag: Java
---

（续前文）

DAO层不写逻辑代码，主要内容：接口设计 + SQL编写。代码和SQL分离，方便Review。

DAO拼接等逻辑在Service层完成。

## Service接口设计

### 目录：

```
main
--resources
    --mapper
        SeckillDao.xml
        SuccessKilledDao.xml
    --spring
        spring-dao.xml
        spring-service.xml
    mybatis-config.xml
--webapp
--java
    --org.seckill
        --dao
        --entity
        --service
            --impl
                SeckillServiceImpl
            SeckillService
        --exception
        --dto           数据传输层
        --enums
            SeckillStateEnum
--sql

test
--java
--resources
```

设计业务接口：要站在“使用者”角度设计接口，而不是实现。包括三个方面：

- 方法定义粒度
- 参数
- 返回类型（return 类型/异常）

### 编码：

DTO：

```java
class Exposer{
    boolean exposed;    //是否开启秒杀
    String md5;
    long seckillId;
    long now;
    long start;
    long end;
}
class SeckillExecution{
    long seckillId;
    int state;
    String stateInfo;
    SuccessKilled successKilled;
}
```

Exception:

```java
// 重复秒杀异常（运行期异常）
public class RepeatKillException extends SeckillException{
    
}
// 秒杀关闭异常
public class SeckillCloseException extends SeckillException{

}
// 秒杀相关业务异常（通用异常）
public class SeckillException extends RuntimeException{
    //IDE自动生成
}
```

Service:

```java
interface SeckilService{
    List<Seckill> getSeckillList();
    Seckill getById(long seckillId);
    Exposer exportSeckillUrl(long seckillId);
    SeckillExecution executeSeckill(long seckillId, long userPhone, String md5) throws SeckillException, RepeatKillException, SeckillCloseException;
}
```

实现Service：

```java
class SeckillServiceImpl implements SeckillService{
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private SeckillDao seckillDao;
    private SuccessKilledDao successKilledDao;
    
    private final string salt = "rgafghasdjfas;dfjads;fj";  //盐值，混淆MD5。
    
    @Override
    List<Seckill> getSeckillList(){
    
    }
    
    Seckill getById(long seckillId){
    
    }
    
    Exposer exportSeckillUrl(long seckillId){
        //拿到Seckill对象
        //比较nowTime是否在startTime和endTime之间。返回相应的 new Exposer()。
    }
    
    // 多看看这个函数的实现，很有用！！(放在本文的最后了。)
    SeckillExecution executeSeckill(long seckillId, long userPhone, String md5) throws SeckillException, RepeatKillException, SeckillCloseException{
        //判断md5是否存在和是否被篡改
        //执行秒杀逻辑：减库存、记录购买行为。抛出异常或者返回new SeckillExecution()。注意这里的try...catch和throw异常的使用。
    }
}
```

可以把状态（数据字典）封装成枚举：SeckillStatEnum，尽量遵循枚举的开发规范。

## 使用Spring托管Service依赖理论

Spring IOC：依赖注入。对对象工厂和依赖管理可以获得一致的访问接口。

业务对象依赖图：
![](/images/posts/14638488325375.jpg)

为什么用IOC：

* 对象创建统一管理
* 规范的生命周期管理
* 灵活的依赖注入
* 一致的获取对象

Spring IOC注入方式和场景：

* XML：来自第三方类库，需要命名空间配置
* 注解：项目中自身开发使用的类，如@Service、@Controller
* Java配置类：通过代码控制对象创建逻辑，如自定义修改依赖类库

本项目IOC使用：

* XML配置
* package-scan（包扫描）
* Annotation注解

### 编码

* 创建spring-service.xml：扫描service包下所有使用注解的类型
* 对SeckillServiceImpl使用@Service注解，注入Service依赖（对成员变量使用@Autowired）。

> 扩展：@Component、@Service、@Dao、@Controller。@Autowired、@Resource、@Inject。

### 使用Spring声明式事务理论

声明式事务：解脱事务代码，交给第三方框架。

使用方式：

* 早期：ProxyFactoryBean + XML。
* tx:advice + aop命名空间：一次配置永久生效。（使用较多）
* 注解@Transactional：注解控制。（更推荐）

使用注解控制事务方法的优点：

* 开发团队达成一致约定，明确标注事务方法的编程风格
* 保证事务方法的执行时间尽可能短，不要穿插其它网络操作（RPC/HTTP请求）或者剥离到事务方法外部。
* 不是所有的方法都需要事务，如只有一条修改操作，只读操作不需要事务控制。

事务方法嵌套：声明式事务独有的概念，当有新的事务到来时：propagation_required等。

什么时候回滚事务：

* 抛出运行期异常（RuntimeException）
* 要小心不当的try-catch。

具体配置：

* 配置事务管理器（jdbc）：注入数据库连接池
* 配置基于注解的声明式事务，默认使用注解来管理事务行为

## 使用集成测试Service逻辑

- 使用Logback记录日志。
- 注意异常情况的处理。


