# 自定义扩展的 Data Id 配置

<mark style="color:blue;">**通过自定义扩展的 Data Id 配置，既可以解决多个应用间配置共享的问题，又可以支持一个应用有多个配置文件。**</mark>

一个完整的配置案例如下所示：

```properties
spring.application.name=opensource-service-provider
spring.cloud.nacos.config.server-addr=127.0.0.1:8848

# config external configuration
# 1、Data Id 在默认的组 DEFAULT_GROUP,不支持配置的动态刷新
spring.cloud.nacos.config.extension-configs[0].data-id=ext-config-common01.properties

# 2、Data Id 不在默认的组，不支持动态刷新
spring.cloud.nacos.config.extension-configs[1].data-id=ext-config-common02.properties
spring.cloud.nacos.config.extension-configs[1].group=GLOBALE_GROUP

# 3、Data Id 既不在默认的组，也支持动态刷新
spring.cloud.nacos.config.extension-configs[2].data-id=ext-config-common03.properties
spring.cloud.nacos.config.extension-configs[2].group=REFRESH_GROUP
spring.cloud.nacos.config.extension-configs[2].refresh=true
```

* 通过 **spring.cloud.nacos.config.extension-configs\[n].data-id** 的配置方式来支持**多个 Data Id** 的配置。
* 通过 **spring.cloud.nacos.config.extension-configs\[n].group** 的配置方式自定义 Data Id 所在的组，不明确配置的话，默认是 DEFAULT\_GROUP。
* 通过 **spring.cloud.nacos.config.extension-configs\[n].refresh** 的配置方式来控制该 Data Id 在配置变更时，是否支持应用中可动态刷新， 感知到最新的配置值。默认是不支持的。

{% hint style="success" %}
<mark style="color:blue;">**多个 Data Id 同时配置时，他的优先级关系是 spring.cloud.nacos.config.extension-configs\[n].data-id 其中 n 的值越大，优先级越高。**</mark>
{% endhint %}

{% hint style="success" %}
**spring.cloud.nacos.config.extension-configs\[n].data-id 的值必须带文件扩展名，文件扩展名既可支持 properties，又可以支持 yaml/yml。**&#x20;

此时 spring.cloud.nacos.config.file-extension 的配置对自定义扩展配置的 Data Id 文件扩展名没有影响。
{% endhint %}

## 共享 Data Id

为了更加清晰的在多个应用间配置共享的 Data Id ，可以通过以下的方式来配置：

```properties
# 配置支持共享的 Data Id
spring.cloud.nacos.config.shared-configs[0].data-id=common.yaml

# 配置 Data Id 所在分组，缺省默认 DEFAULT_GROUP
spring.cloud.nacos.config.shared-configs[0].group=GROUP_APP1

# 配置Data Id 在配置变更时，是否动态刷新，缺省默认 false
spring.cloud.nacos.config.shared-configs[0].refresh=true
```

* 通过 spring.cloud.nacos.config.shared-configs\[n].data-id 来支持多个共享 Data Id 的配置。
* 通过 spring.cloud.nacos.config.shared-configs\[n].group 来配置自定义 Data Id 所在的组，不明确配置的话，默认是 DEFAULT\_GROUP。
* 通过 spring.cloud.nacos.config.shared-configs\[n].refresh 来控制该 Data Id 在配置变更时，是否支持应用中动态刷新，默认 false。
