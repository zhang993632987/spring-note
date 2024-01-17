# 云计算

<mark style="color:blue;">**云计算**</mark>是通过**互联网交付计算**和**虚拟化IT服务**——数据库、网络、软件、服务器、分析等，以提供灵活、安全、易于使用的环境。

最常见的云计算模型有以下几种：

* <mark style="color:blue;">**基础设施即服务**</mark>（Infrastructure as a Service，**IaaS**）
  * 供应商提供的基础设施，向用户提供服务器、存储和网络等计算资源的访问。
  * 在这个模型中，用户负责一切与基础设施维护和应用程序可伸缩性相关的东西。
* <mark style="color:blue;">**平台即服务**</mark>（Platform as a Service，**PaaS**）
  * 此模型提供平台和环境，让用户专注于应用程序的开发、执行和维护，甚至可以使用供应商提供的工具来创建这些应用程序（例如操作系统、数据库管理系统、技术支持、存储、托管、网络等）。
  * 用户不需要在物理基础设施上投入资金，也不需要花时间来管理它，这使得用户可以专注于应用程序的开发。
* <mark style="color:blue;">**容器即服务**</mark>（Container as a Service，**CaaS**）
  * 此模型介于IaaS和PaaS之间，它指的是一种基于容器的虚拟化形式。
  * 与IaaS模型不同，使用IaaS的开发人员必须管理部署服务的虚拟机，而使用CaaS则可以将微服务部署在一个轻量级、可移植的虚拟容器（如Docker）中，并将其部署到云供应商。云供应商不但运行容器运行所在的虚拟服务器，而且提供用于构建、部署、监控和缩扩容的综合工具。
* <mark style="color:blue;">**函数即服务**</mark>（Function as a Service，**FaaS**）
  * 也称为无服务器架构，尽管名字如此，但这个架构并不意味着在没有服务器的情况下运行特定的代码，它真正的意思是在云中执行功能的一种方式，供应商在云中提供所有必需的服务器。
  * 无服务器架构允许我们只关注服务的开发，而不必担心缩扩容、供应和服务器管理。相反，我们可以只专注于上传我们的函数，而不需要管理基础设施。
* <mark style="color:blue;">**软件即服务**</mark>（Software as a Service，**SaaS**）
  * 也称为按需软件，此模型使用户无须进行任何部署或维护就可以使用特定的应用程序。
  * 在大多数情况下，通过Web浏览器访问应用程序。
  * 在这个模型中，一切都由服务供应商管理：应用程序、数据、操作系统、虚拟化、服务器、存储和网络。用户只需租用服务并使用软件即可。

<figure><img src="../../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>