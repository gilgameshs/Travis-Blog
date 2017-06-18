---
layout:     post
title:      "Report Service"
date:       2017-06-17 23:00:00
author:     "Travis"
header-img: "img/post-bg-2015.jpg"
tags:
    - 缓存
    - 新浪
---
## 高可用
> 借助 Eureka 以及 Ribbon

--- 
## 南北方网络问题

  南方机器 APP Gateway 访问北方的 eureka，再负载到北方的 report service，网络开销很大.
南方机房，以及北方机房，两个地区应该分别有一个 Eureka,或者每个地区分别有两个。
保证一个突然挂掉，不会出现问题。

![eureka zone and peer](/img/private-sina/eureka-server.png)

application.yml (Two Peer Aware Eureka Servers)

```

---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: http://peer2/eureka/

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: http://peer1/eureka/
```

Service 1 in Zone 1
```
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```
Service 1 in Zone 2
```
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

---

### 业务功能
* 分页: [spring data common page](https://github.com/spring-projects/spring-data-commons/tree/master/src/main/java/org/springframework/data/domain)系列工具
* 排序
    - 曝光，点击，点击率，消耗
        + page传入相应sort对象

---

## 缓存策略
> 因服务是负载的，不能保证客户请求落在相同服务实例上，所以缓存应是共享的
> 每个服务实例也应有自己的本地缓存。
 
![cache](/img/private-sina/cache.png)
    
### 服务共享缓存：Redis
* 以服务名，查询条件为基础生成key值（service-name + method-name + paramMap）
* key值最大长度为512 MB
* 首次排序查询从引擎拿全量数据
* 首次加载获取部分数据（首次加载一般不会有排序要求，默认排序即可）

### 单服务实例（进程间）缓存：caffeine
* 高命中率，出色的并发能力
* 驱逐策略：缓存的驱逐策略是为了预测哪些数据在短期内最可能被再次用到，从而提升缓存的命中率。
    + LRU（Least Recently Used）：实现简洁，运行高效，但 LRU 通过历史数据来预测未来是局限的，它会认为最后到来的数据是最可能被再次访问的，从而给与它最高的优先级。
    + Window TinyLfu：LRU -> TinyLFU -> Segmented LRU,LRU是为了应对突发流量，LFU处理常见情况。
* 过期策略：
    + 根据缓存数目
    + 根据时间
        * 写后过期
        * 读后过期
* 并发：
Caffeine采用了类似日志的方式尽可能避免锁的操作：写入操作放到日志里然后异步批处理更新。
具体而言，Caffeine利用ringbuffer存放写入数据，待buffer满之后做批量处理，并且为每个线程使用独立的ringbuffer进一步提升性能

---

## 监控
* 缓存监控：redis-live
* 服务监控：
    - 接口监控：hystrix

---

