---
layout: post
title: 【慕课网笔记】Java高并发秒杀API（三）：Web层
author: 孟凡泽
date: 2016-05-23 19:00:40 +0800
tag: Java
---


注：续前文。本篇主要是Web层的开发，主要是使用Bootstrap进行页面的简单布置和用Javascript书写前端的交互逻辑。其中有很多细节值得我学习，比如Javascript代码的模块化。

看完这部视频后更加觉得，这个项目必须要自己做一遍。我打算趁着这几天比较闲，按照视频中的思路，将这个秒杀系统用PHP实现出来，因为自己太久没有用PHP写项目了，借此也可以学一些PHP中的MVC等框架，就像视频中使用的Java的SpringMVC那样。

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
        spring-web.xml
    mybatis-config.xml
--webapp
    --resources
    --WEB-INF
        web.xml
    index.jsp
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
        --web
            SeckillController
--sql

test
--java
--resources
```

## 设计Restful接口

### 前端交互流程设计

### 学习Restful接口设计

兴起于：Ruby on Rails，一种优雅的URI表述方式，是一种资源的状态和状态的转移。

GET /seckill/list
POST /seckill/{id}/execution
DELETE /seckill/{id}/delete

GET：查询
POST：添加/修改(非逆等)
PUT：修改
DELETE：删除

URL设计：/模块/资源/{标示}/集合1/...

实例：秒杀URL设计：
GET /seckill/list > 秒杀列表
GET /seckill/{id}/detail > 详情页
GET /seckill/time/now > 系统时间
POST /seckill/{id}/exposer > 暴露秒杀
POST /seckill/{id}/{md5}/execution > 执行秒杀

## SpringMVC集合spring

### 使用SpringMVC理论

围绕Handler开发：Handler产出Model和View。

#### SprintMVC运行流程

#### HTTP请求地址映射原理

使用注解：@RequestMapping。

#### 请求方法细节处理

- 请求参数绑定
- 请求方式限制
- 请求转发和重定向
- 数据模型赋值
- 返回Json数据
- cookie访问

![](/images/posts/14640003766491.jpg)


### 整合配置SpringMVC框架

- 配置DispatcherServlet：配置SpringMVC需要加载的配置文件：spring-dao.xml、spring-service.xml、spring-web.xml。
- 配置spring-web.xml：配置SpringMVC：
    - 开启SpringMVC注解模式。（简化配置：自动注册DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter；提供一系列功能：数据绑定、数字和日期的转换@NumberFormat、@DateTimeFormat，xml和json默认读写支持。）
    - 静态资源默认servlet配置（加入对静态资源的处理：js/git/png...，允许使用"/"做整体映射）
    - 配置jsp显示ViewResolver.
    - 扫描web相关的bean。

## 实现Restful接口

- 新建SeckillController。

```java
@Controller //类似于@Service @Component，目的是放入spring容器
@RequestMapping("/seckill") //url:/模块/资源/{id}/细分  
public class SeckillController{
    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private SeckillService seckillService;

    @RequestMapping(name="/list", method = RequestMethod.GET)
    public String list(Model model){
        //获取列表页
        List<Seckill> list = seckillService.getSeckillList();
        model.addAttribute("list", list);
        //list.jsp + model = ModelAndView
        return "list";  // /WEB-INF/jsp/"list".jsp
    }
    
    @RequestMapping(value = "/{seckillId}/detail", method = RequestMethod.GET)
    public String detail(@PathVariable("seckillId") Long seckillId, Model model){
        if(seckillId == null){
            return "redirect:/seckill/list";
        }
        Seckill seckill = seckillService.getById(seckillId);
        if(sekill == null){
            return "forward:/seckill/list";
        }
        model.addAttribute("seckill", seckill);
        return "detail";
    }
    
    //ajax json
    @RequestMapping(value = "/{seckillId}/exposer", method = RequestMethod.POST, produces = {"application/json;charset=UTF-8"})
    @ResponsBody
    public SeckillResult<Exposer> exposer(Long seckillId){
        SeckillResult<Exposer> result;
        Exposer exposer = seckillService.exportSeckillUrl(seckillId);
        //给result赋值，使用try catch
        return result;
    }
    
    @RequestMapping(value = "/{seckillId}/{md5}/execution", method = RequestMethod.POST, produces = {"application/json;charset=UTF-8"})
    @ResponsBody
    public SeckillResult<SeckillException> execute(.....){ //这里参数killPhone使用cookie
    
    }
    
    @RequestMapping(value = "/time/now", method = RequestMethod.GET)
    public SeckillResult<Long> time(){
        ...
    }
}
```

- 创建class SeckillResult<T>，包括boolean success、T data、String error，用于封装json结果，返回ajax结果。

## 基于bootstrap开发页面结构

- WEB-INFO/jsp下建立页面：detail.jsp、list.jsp。建立文件夹common放置公共的head.jsp，包括所有的<link>标签。
- 尽量使用CDN托管js库！！！
- 使用了JQuery cookie和JQuery countDown插件。
- 要用<script></script>，而不是<script />。

## Cookie登录交互、计时交互、秒杀交互

- javascript做到模块化，模拟包：

```js
    var seckill = {
        //封装秒杀相关ajax的url
        URL ： {
            now: function(){
                return "/seckill/time/now"
            }
        },
        //验证手机号
        validatePhone: function(phone){
        
        },
        //时间判断，为了减少detail代码和代码复用。
        countdown: function(seckillId, nowTime, startTime, endTime){
        
        },
        //获取秒杀地址，控制显示逻辑，执行秒杀
        handleSeckillKill: function(seckillId, node){
        
        },
        //详情页秒杀逻辑
        detail: {
            //详情页初始化
            init : function(params){
                //手机验证和登录，计时交互
                //规范我们的交互流程：params包括seckillId、startTime、endTime
                //在cookie中查找手机号（因为这里没有用到相关登录的后端，使用cookie替代）
                //验证手机，提出一个独立的方法validatePhone
                
                //已经登录
                //计时交互
                $.get(seckill.URL.now(), {}, function(result){
                    if(result && result['success']){
                        //时间判断
                        countdown(....);
                    }else{
                        
                    }
                });
            }
        }
    }
```

## web层课程总结

- 技术回顾：前端交互设计过程、Restful接口设计、SpringMVC使用技巧、Bootstrap和JS的使用
- 前端交互设计（前端、产品、后端）：

![](/images/posts/14640003907843.jpg)


- SpringMVC：配置和运行流程、DTO传递数据、注解映射驱动。
- SpringMVC运行流程：

![](/images/posts/14640003986865.jpg)



