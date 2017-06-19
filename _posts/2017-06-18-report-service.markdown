---
layout:     post
title:      "Report Service"
date:       2017-06-17 23:00:00
author:     "Travis"
header-img: "img/post-bg-2015.jpg"
catalog:    true
tags:
    - 缓存
    - 新浪
---

## 缓存策略
> 因服务是负载的，不能保证客户请求落在相同服务实例上，所以缓存应是共享的
> 每个服务实例也应有自己的本地缓存。
 
![cache](/img/private-sina/cache.png)
    
### 服务共享缓存：Redis
* 存储类型：String, List
* 以服务名，查询条件为基础生成key值（service-name + method-name + paramMap）
   - key值最大长度为512 MB
* 首次排序查询从引擎拿全量数据
* 首次加载获取部分数据（首次加载一般不会有排序要求，默认排序即可）

### 单服务实例（进程间）缓存：caffeine
* 高命中率，出色的并发能力
* 驱逐策略：缓存的驱逐策略是为了预测哪些数据在短期内最可能被再次用到，从而提升缓存的命中率。

> LRU 是最近最少使用页面置换算法(Least Recently Used),也就是首先淘汰最长时间未被使用的页
LFU 是最近最不常用页面置换算法(Least Frequently Used),也就是淘汰一定时期内被访问次数最少的页。

   + LRU（Least Recently Used）：实现简洁，运行高效，但 LRU 通过历史数据来预测未来是局限的，它会认为最后到来的数据是最可能被再次访问的，从而给与它最高的优先级。
   + Window TinyLfu：LRU -> TinyLFU -> Segmented LRU,LRU是为了应对突发流量，LFU处理常见情况。
![W-TinyLfu](/img/private-sina/window-tinylfu.png)
其中，过滤层的TinyLfu采用了CountMin Sketch 算法：
![CountMin-Sketch](/img/private-sina/sketch.png)
* 过期策略：
```properties
spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s
```
    + 根据缓存数目
    + 根据时间
       * 写后过期
       * 读后过期
* 并发：
Caffeine采用了类似日志的方式尽可能避免锁的操作：写入操作放到日志里然后异步批处理更新。
具体而言，Caffeine利用ringbuffer存放写入数据，待buffer满之后做批量处理，并且为每个线程使用独立的ringbuffer进一步提升性能

* 基准测试：
![caffeine-test](/img/private-sina/caffeine-test.png)

---

## 业务功能
* 分页: 根据list进行分页，自定义page对象
* 排序
    - 曝光，点击，点击率，消耗
        + page传入相应sort对象

---



## 高可用

借助 **Eureka** 以及 **Ribbon**

--- 
## 南北方网络问题

  南方机器 APP Gateway 访问北方的 eureka，再负载到北方的 report service，网络开销很大.
南方机房，以及北方机房，两个地区应该分别有一个 Eureka,或者每个地区分别有两个,实现一定的容灾功能。

![eureka zone and peer](/img/private-sina/eureka.png)

Eureka Server:
```yaml
server:
  port: ${PORT:8761}
---
spring:
  profiles: south
eureka:
  instance:
    hostname: south
  client:
    serviceUrl:
      south: http://south:8761/eureka/
      north: http://north:8761/eureka/
---
spring:
  profiles: north
eureka:
  instance:
    hostname: north
  client:
    serviceUrl:
      north: http://north:8761/eureka/
      south: http://south:8761/eureka/
      
```

APPs:
```yaml

eureka:
  client:
    preferSameZoneEureka: true
    region: china
  instance:
    metadataMap:
      instanceId: ${spring.application.name}:${server.port}

---
spring:
  profiles: south
eureka:
  instance:
    metadataMap:
      zone: south
  client:
    serviceUrl:
      south: http://south:8761/eureka/
      north: http://north:8761/eureka/
    availabilityZones:
      china: south,north
      
---
spring:
  profiles: north
eureka:
  instance:
    metadataMap:
      zone: north
  client:
    serviceUrl:
      north: http://north:8761/eureka/
      south: http://south:8761/eureka/
    availabilityZones:
      china: north,south
```
---

## 监控
* 缓存监控：公司提供的监控系统
* 服务监控：
    - 接口监控：Hystrix
![hystrix](/img/private-sina/hystrix.png)

---

## 后续
* 缓存配置的动态更新
* 服务调用链监控

