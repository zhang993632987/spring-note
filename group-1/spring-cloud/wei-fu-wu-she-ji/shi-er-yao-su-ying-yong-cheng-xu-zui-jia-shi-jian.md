# 十二要素应用程序最佳实践

为了应对创建云原生微服务的挑战，我们将使用Heroku的最佳实践指南，称为<mark style="color:blue;">**十二要素应用程序（twelve-factor app）**</mark>，来构建高质量的微服务。可以将此方法视为开发和设计实践的集合，这些实践在构建分布式服务时侧重于**动态扩展**和**基本点**。

### <mark style="color:blue;">代码库</mark>

通过此实践，<mark style="color:orange;">**每个微服务都应该有一个单独的、源代码控制的代码库**</mark>。另外，必须要强调，<mark style="color:orange;">**服务器供应也应该处于版本控制中**</mark>。

代码库可以有多个部署环境实例（如**开发环境**、**测试环境**、**交付准备环境**、**生产环境**等），但它不与任何其他微服务共享。这是一条重要的指导原则，因为如果我们把代码库共享给所有微服务，我们最终将产生许多属于不同环境的不可变版本。

### <mark style="color:blue;">依赖</mark>

此最佳实践<mark style="color:orange;">**通过构建工具（如Maven或Gradle（Java））显式声明应用程序使用的依赖项**</mark>。

第三方JAR依赖项应该使用它们的特定版本号来声明。这允许你始终使用相同的库版本构建微服务。

### <mark style="color:blue;">配置</mark>

<mark style="color:orange;">**永远不要在源代码中添加嵌入式配置**</mark>！相反，<mark style="color:orange;">**最好将配置与可部署的微服务完全分离**</mark>。

### <mark style="color:blue;">后端服务</mark>

微服务通常会通过网络与数据库、API RESTful服务、其他服务器或消息传递系统进行通信。当这样做时，你<mark style="color:orange;">**应该确保可以在本地和第三方连接之间替换部署实现，而不需要对应用程序代码进行任何更改**</mark>。

### <mark style="color:blue;">构建、运行和发布</mark>

这条最佳实践提醒我们<mark style="color:orange;">**保持应用程序部署的构建、发布和运行阶段完全分离**</mark>。

我们应该能够构建独立于运行它们的环境的微服务。<mark style="color:orange;">**一旦构建了代码，任何运行时更改都需要回退到构建流程并重新部署**</mark>。已构建服务是不可变的，不能更改。

发布阶段负责将已构建服务与每个目标环境的特定配置相结合。如果不分离不同的阶段，则可能会导致代码中的问题和差异无法跟踪，或最好的情况下，难以跟踪。<mark style="color:orange;">**如果我们修改已经部署在生产环境中的服务，那么更改将不会记录在存储库中，并且可能出现两种情况：服务的新版本没有更改，或者我们被迫将更改复制到服务的新版本。**</mark>

### <mark style="color:blue;">进程</mark>

<mark style="color:orange;">**微服务应该始终是无状态的，并且应该只包含执行请求的事务所必需的信息**</mark>。

微服务可以随时终止和替换，而不必担心服务实例的丢失会导致数据丢失。

<mark style="color:orange;">**如果有存储状态的特定要求，则必须通过内存缓存（如Redis）或后端数据库来完成。**</mark>

### <mark style="color:blue;">端口绑定</mark>

<mark style="color:orange;">**端口绑定意味着通过特定端口发布服务**</mark>。

在微服务架构中，微服务在打包的时候应该是完全独立的，可运行的微服务中要包含一个运行时引擎。运行服务时不需要单独的Web或应用程序服务器。服务应该在命令行上自行启动，并且可通过公开的HTTP端口立即访问。

### <mark style="color:blue;">并发</mark>

并发最佳实践解释了云原生应用程序应该使用进程模型<mark style="color:orange;">**水平扩展**</mark>。

我们可以创建多个进程，然后在不同的进程之间分配服务的负载或应用程序，而不是让单个重要进程变得更大。

<mark style="color:orange;">**当需要扩展时，启动更多的微服务实例，进行横向扩展而不是纵向扩展。**</mark>

### <mark style="color:blue;">可任意处置</mark>

**微服务是可任意处置的，可以根据需要启动和停止，以促进弹性扩展，并快速部署应用程序代码和配置更改**。理想情况下，从启动命令执行到进程准备好接收请求，启动应该持续几秒钟。

<mark style="color:orange;">**可任意处置的意思是，我们可以用新实例移除失败实例，而不会影响任何其他服务。**</mark>如果某个微服务的实例由于底层硬件故障而失败，我们可以关闭该实例而不影响其他微服务，并在需要时在其他地方启动另一个实例。

### <mark style="color:blue;">开发环境/生产环境等同</mark>

这条最佳实践指的是<mark style="color:orange;">**尽可能让不同的环境（如开发环境、交付准备环境、生产环境）保持相似**</mark>。<mark style="color:orange;">**环境应始终包含部署代码的类似版本，基础设施和服务也一样**</mark>。

这可以通过**持续部署**来实现，它尽可能**自动化部署过程**，**允许在短时间内在环境之间部署微服务**。代码一提交就应该被测试，然后尽快从开发环境推进到生产环境。

如果我们想要避免部署错误，这条指导原则是必不可少的。拥有类似的开发环境和生产环境允许我们在部署和执行应用程序时掌控所有可能的场景。

### <mark style="color:blue;">日志</mark>

<mark style="color:orange;">**在写入日志时，应该通过Logstash或Fluentd等工具来管理日志，这些工具收集日志并将它们写到一个集中位置**</mark>。

微服务永远不应该关心这是如何发生的，它只需要专注于将日志条目写入标准输出（stdout）。

### <mark style="color:blue;">管理进程</mark>

开发人员通常不得不针对他们的服务执行<mark style="color:orange;">**管理任务**</mark>（如数据迁移或转换）。<mark style="color:orange;">**这些任务不应该是临时指定的，而应该通过源代码存储库管理和维护的脚本来完成**</mark>。这些脚本应该是可重复的，并且在每个运行的环境中都是不可变的（脚本代码不会针对每个环境进行修改）。

在运行微服务时，定义我们需要考虑的任务类型是很重要的，这样如果我们有多个带有这些脚本的微服务，就能够执行所有的管理任务，而不必手动执行这些任务。