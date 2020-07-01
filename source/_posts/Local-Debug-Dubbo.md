---
title: 无代码侵入调试dubbo服务
date: 2020-06-28 18:58:48
tags: dubbo
categories: 
- 效率

---

### 1.提供者（Provider-service）

​	dubbo.xml中添加**register="false"** 

```xml
<dubbo:registry protocol="zookeeper" address="${zookeeper.cluster}" register="false"/>
```

### 2.消费者（Consumer-service）

​        在本地任意位置新建配置文件:dubbo-local-consumer.properties：

```properties
# 以下是你们DubboServer.xml中配置的需要Export Service，
# 建议你有几个要Export Service都配置在这里，后面是请求本地的地址
# 地址格式：dubbo://ip:port，这里需要注意的是，需要修改为自己dubbo服务的端口
com.xxxService=dubbo://localhost:20880

```

​		设置启动参数指向该配置文件：

```properties
# 启动参数：
-Ddubbo.resolve.file=/C:\workspace\dubbo-local-consumer.properties
```

