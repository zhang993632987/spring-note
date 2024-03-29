# 客户端注册

## 1. 添加依赖

首先需要做的是将Spring Eureka依赖项添加到组织服务和许可证服务的pom.xml文件中。

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

## 2. 增加配置属性（在Config Server中）

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

* **eureka.instance.preferIpAddress**属性告诉Eureka，你想将服务的<mark style="color:blue;">**IP地址**</mark>而不是服务的**主机名**注册到Eureka。
* **eureka.client.registerWithEureka**属性是一个触发器，告知Spring Eureka客户端通过Eureka进行注册。
* **eureka.client.fetchRegistry**属性用于告知Spring Eureka客户端以获取注册表的本地副本。
  * 将此属性设置为 true将**在本地缓存注册表**，而不是每次查找服务都调用Eureka服务。
  * 每隔30秒，客户端软件就会重新联系Eureka服务，以便查看注册表是否有任何变化。
* **eureka.client.serviceUrl.defaultZone**包含了客户端用于解析服务位置的Eureka服务的**列表**，该列表以逗号进行分隔。

> 每个通过Eureka注册的服务都会有两个与之相关的组件：<mark style="color:blue;">**应用程序ID**</mark>和<mark style="color:blue;">**实例ID**</mark>。
>
> * **应用程序ID**用于表示一组服务实例。在Spring Boot微服务中，应用程序ID始终是由**spring. application.name**属性设置的值。
> * **实例ID**是一个随机生成的数字，用于代表单个服务实例。

> ## <mark style="color:blue;">**为什么偏向于IP地址?**</mark>
>
> <mark style="color:orange;">**在默认情况下，Eureka注册通过主机名联系它的服务。**</mark>这种方式在基于服务器的环境中运行良好，在这样的环境中，服务会被分配一个DNS支持的主机名。但是，**在基于容器的部署（如Docker）中，容器将以随机生成的主机名启动，并且该主机名不存在对应的DNS条目。**如果你没有将eureka.instance. preferIpAddress设置为true，那么你的客户端应用程序将无法正确地解析主机名的位置，因为该容器不存在DNS条目。设置preferIpAddress属性将通知Eureka服务，客户端想要通过IP地址进行通告。
>
> **基于云的微服务应该是短暂的和无状态的。它们可以随意启动和关闭，所以IP地址更适合这些类型的服务。**

## 3. 查看注册的服务

### Eureka 的REST API

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

> Eureka服务返回的默认格式是XML。
>
> Eureka可以将数据作为JSON净荷返回，但是必须将HTTP首部**Accept**设置为**application/json**。

### Eureka的仪表盘

一旦Eureka服务启动完毕，我们就可以用浏览器访问**http://localhost:8070**来查看Eureka仪表板。Eureka仪表板允许我们查看服务的注册状态。

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
## <mark style="color:orange;">注意</mark>

<mark style="color:blue;">**当服务通过Eureka注册时，Eureka将在30秒内等待3次连续的健康检查，然后服务才能变得可用。**</mark>

这一点在我们在**Docker环境中**运行的代码示例中很明显，因为Eureka服务和应用程序服务都是在同一时间启动的。请注意，<mark style="color:orange;">**在启动应用程序后，尽管服务本身已经启动，但你可能会收到关于未找到服务的404错误。在这种情况下，请等待30秒，然后再尝试调用服务。**</mark>

**在生产环境中，你的Eureka服务已经在运行。如果你正在部署现有的服务，那么旧服务仍然可以用于接收请求。**
{% endhint %}
