# 集群

## 服务端

```yaml
spring:
  application:
    name: eureka-server
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8091/eureka,
        http://localhost:8092/eureka, http://localhost:8093/eureka

---
server:
  port: 8091
spring:
  profiles: peer1

---
server:
  port: 8092
spring:
  profiles: peer2

---
server:
  port: 8093
spring:
  profiles: peer3
```

通过设定 <mark style="color:blue;">**spring.profiles.active**</mark> 参数启动三个 eureka 服务端构成一个三节点的集群。

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
* Eureka 服务端会将服务信息保存在内存之中
* 当以集群模式运行时，每一个 Eureka 服务端都会存在完整的服务信息
* Eureka 服务之间通过点对点的方式进行信息的同步。
{% endhint %}

## 客户端

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8091/eureka,
        http://localhost:8092/eureka, http://localhost:8093/eureka
```
