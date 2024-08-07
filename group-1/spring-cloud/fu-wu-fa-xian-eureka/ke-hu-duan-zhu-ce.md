# 客户端注册

## 1. 添加依赖

首先需要做的是将 Spring Eureka 依赖项添加到组织服务和许可证服务的 pom.xml 文件中。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

## 2. 增加配置属性（在 Config Server 中）

```yaml
eureka:
  instance:
    prefer-ip-address: true
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://localhost:8070/eureka/
```

* **eureka.instance.preferIpAddress** 属性告诉 Eureka，将服务的 <mark style="color:blue;">**IP 地址**</mark>而不是服务的**主机名**注册到 Eureka。
* **eureka.client.registerWithEureka** 属性是一个触发器，告知 Spring Eureka 客户端通过 Eureka 进行注册。
* **eureka.client.fetchRegistry** 属性用于告知 Spring Eureka 客户端以获取注册表的本地副本。
  * 将此属性设置为 true 将**在本地缓存注册表**，而不是每次查找服务都调用 Eureka 服务。
  * 每隔 30 秒，客户端软件就会重新联系 Eureka 服务，以便查看注册表是否有任何变化。
* **eureka.client.serviceUrl.defaultZone** 包含了客户端用于解析服务位置的 Eureka 服务的**列表**，该列表以逗号进行分隔。

{% hint style="success" %}
每个通过 Eureka 注册的服务都会有两个与之相关的组件：<mark style="color:blue;">**应用程序 ID**</mark> 和<mark style="color:blue;">**实例 ID**</mark>。

* **应用程序 ID** 用于表示一组服务实例。在 Spring Boot 微服务中，应用程序 ID 始终是由**spring.application.name** 属性设置的值。
* **实例 ID** 是一个随机生成的数字，用于代表单个服务实例。
{% endhint %}

{% hint style="info" %}
<mark style="color:blue;">**为什么偏向于 IP 地址?**</mark>

<mark style="color:orange;">**在默认情况下，Eureka 注册通过主机名联系它的服务。**</mark>这种方式在基于服务器的环境中运行良好，在这样的环境中，服务会被分配一个 DNS 支持的主机名。

但是，**在基于容器的部署（如Docker）中，容器将以随机生成的主机名启动，并且该主机名不存在对应的DNS条目。**如果你没有将 eureka.instance.preferIpAddress 设置为 true，那么你的客户端应用程序将无法正确地解析主机名的位置，因为该容器不存在对应的 DNS 条目。

设置 preferIpAddress 属性将通知 Eureka 服务，客户端想要通过 IP 地址进行通告。

**基于云的微服务应该是短暂的和无状态的，它们可以随意启动和关闭，所以 IP 地址更适合这些类型的服务。**
{% endhint %}

## 3. 查看注册的服务

### Eureka 的 REST API

<details>

<summary><mark style="color:purple;">http://localhost:8070/eureka/apps</mark></summary>

{% code overflow="wrap" %}
```xml
<applications>
    <versions__delta>1</versions__delta>
    <apps__hashcode>UP_1_</apps__hashcode>
    <application>
        <name>LICENSE-SERVICE</name>
        <instance>
            <instanceId>localhost:license-service:8080</instanceId>
            <hostName>192.168.157.1</hostName>
            <app>LICENSE-SERVICE</app>
            <ipAddr>192.168.157.1</ipAddr>
            <status>UP</status>
            <overriddenstatus>UNKNOWN</overriddenstatus>
            <port enabled="true">8080</port>
            <securePort enabled="false">443</securePort>
            <countryId>1</countryId>
            <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo">
                <name>MyOwn</name>
            </dataCenterInfo>
            <leaseInfo>
                <renewalIntervalInSecs>30</renewalIntervalInSecs>
                <durationInSecs>90</durationInSecs>
                <registrationTimestamp>1702034771688</registrationTimestamp>
                <lastRenewalTimestamp>1702034771688</lastRenewalTimestamp>
                <evictionTimestamp>0</evictionTimestamp>
                <serviceUpTimestamp>1702034771082</serviceUpTimestamp>
            </leaseInfo>
            <metadata>
                <management.port>8080</management.port>
            </metadata>
            <homePageUrl>http://192.168.157.1:8080/</homePageUrl>
            <statusPageUrl>http://192.168.157.1:8080/actuator/info</statusPageUrl>
            <healthCheckUrl>http://192.168.157.1:8080/actuator/health</healthCheckUrl>
            <vipAddress>license-service</vipAddress>
            <secureVipAddress>license-service</secureVipAddress>
            <isCoordinatingDiscoveryServer>false</isCoordinatingDiscoveryServer>
            <lastUpdatedTimestamp>1702034771688</lastUpdatedTimestamp>
            <lastDirtyTimestamp>1702034772143</lastDirtyTimestamp>
            <actionType>ADDED</actionType>
        </instance>
    </application>
</applications>
```
{% endcode %}

</details>

<details>

<summary><mark style="color:purple;">http://localhost:8070/eureka/apps/license-service</mark></summary>

{% code overflow="wrap" %}
```xml
<application>
    <name>LICENSE-SERVICE</name>
    <instance>
        <instanceId>localhost:license-service:8080</instanceId>
        <hostName>192.168.157.1</hostName>
        <app>LICENSE-SERVICE</app>
        <ipAddr>192.168.157.1</ipAddr>
        <status>UP</status>
        <overriddenstatus>UNKNOWN</overriddenstatus>
        <port enabled="true">8080</port>
        <securePort enabled="false">443</securePort>
        <countryId>1</countryId>
        <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo">
            <name>MyOwn</name>
        </dataCenterInfo>
        <leaseInfo>
            <renewalIntervalInSecs>30</renewalIntervalInSecs>
            <durationInSecs>90</durationInSecs>
            <registrationTimestamp>1702034771688</registrationTimestamp>
            <lastRenewalTimestamp>1702035011633</lastRenewalTimestamp>
            <evictionTimestamp>0</evictionTimestamp>
            <serviceUpTimestamp>1702034771082</serviceUpTimestamp>
        </leaseInfo>
        <metadata>
            <management.port>8080</management.port>
        </metadata>
        <homePageUrl>http://192.168.157.1:8080/</homePageUrl>
        <statusPageUrl>http://192.168.157.1:8080/actuator/info</statusPageUrl>
        <healthCheckUrl>http://192.168.157.1:8080/actuator/health</healthCheckUrl>
        <vipAddress>license-service</vipAddress>
        <secureVipAddress>license-service</secureVipAddress>
        <isCoordinatingDiscoveryServer>false</isCoordinatingDiscoveryServer>
        <lastUpdatedTimestamp>1702034771688</lastUpdatedTimestamp>
        <lastDirtyTimestamp>1702034772143</lastDirtyTimestamp>
        <actionType>ADDED</actionType>
    </instance>
</application>
```
{% endcode %}

</details>

{% hint style="success" %}
Eureka 服务返回的默认格式是 XML。

Eureka 可以将数据作为 JSON 净荷返回，但是必须将 HTTP 请求中的首部字段 **Accept** 设置为**application/json**。
{% endhint %}

### Eureka 的仪表盘

一旦 Eureka 服务启动完毕，就可以通过访问 **http://localhost:8070** 来查看 Eureka 仪表板。Eureka 仪表板允许我们查看服务的注册状态。

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
<mark style="color:blue;">**当服务通过 Eureka 注册时，Eureka 将在 30 秒内等待 3 次连续的健康检查，然后服务才能变得可用。**</mark>这一点在 **Docker 环境中**运行我们的代码示例时表现非常明显，因为 Eureka 服务和应用程序服务都是在同一时间启动的。

请注意，<mark style="color:orange;">**在启动应用程序后，尽管服务本身已经启动，我们仍然可能收到关于未找到服务的 404 错误。在这种情况下，请等待 30 秒，然后再尝试调用服务。**</mark>
{% endhint %}
