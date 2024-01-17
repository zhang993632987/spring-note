# Spring Cloud服务发现

通过<mark style="color:blue;">**Spring Cloud服务发现**</mark>，开发人员可以**从消费服务的客户端中抽象出部署服务器的物理位置**（IP地址或服务器名称）。服务消费者通过<mark style="color:orange;">**逻辑名称**</mark>而不是物理位置来调用服务器的业务逻辑。

Spring Cloud服务发现也处理服务实例在启动和关闭时的注册和注销。

Spring Cloud服务发现可以使用以下服务来实现：

* **Consul**；
* **ZooKeeper**；
* **Eureka**。
