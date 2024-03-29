# Spring Cloud Config

<mark style="color:blue;">**Spring Cloud Config**</mark>通过**集中式服务**来处理应用程序配置数据的管理，因此**应用程序配置数据（特别是环境特定的配置数据）与部署的微服务是完全分离的**。这确保了无论启动多少个微服务实例，这些微服务实例始终具有相同的配置。

Spring Cloud Config拥有自己的属性管理存储库，但也可以与以下开源项目集成：

* **Git**：Spring Cloud Config集成了Git后端存储库，能从存储库中读出应用程序的配置数据。
* **Consul**：Consul包括键值存储数据库，Spring Cloud Config可以用它来存储应用程序的配置数据。
* **Eureka**：Eureka同样也有一个可以被Spring Cloud Config使用的键值数据库。
