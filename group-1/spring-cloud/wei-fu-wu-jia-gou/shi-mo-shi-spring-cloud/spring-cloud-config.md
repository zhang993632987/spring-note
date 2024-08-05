# Spring Cloud Config

<mark style="color:blue;">**Spring Cloud Config**</mark> 通过**集中式服务**来管理应用程序的配置数据，因此**应用程序配置数据（特别是环境特定的配置数据）与部署的微服务是完全分离的**。这确保了无论启动多少个微服务实例，这些微服务实例始终具有相同的配置。

Spring Cloud Config 拥有自己的属性管理存储库，但也可以与以下开源项目集成：

* **Git**：Spring Cloud Config 集成了 Git 后端存储库，能从存储库中读出应用程序的配置数据。
* **Consul**：Consul 包括键值存储数据库，Spring Cloud Config 可以用它来存储应用程序的配置数据。
* **Eureka**：Eureka 同样也有一个可以被 Spring Cloud Config 使用的键值数据库。
