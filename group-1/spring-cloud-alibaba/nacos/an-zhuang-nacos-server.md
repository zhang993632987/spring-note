# 安装 Nacos Server

## Windows

**Nacos 依赖 Java 环境运行，因此需要先保证安装了 Java 环境。**

### 1. 下载安装包包

地址：[https://github.com/alibaba/nacos/releases](https://github.com/alibaba/nacos/releases)

### 2. 修改配置文件

> **在 2.2.0.1 和 2.2.1 版本时，必须执行此变更，否则无法启动；其他版本为建议设置。**

**修改 conf 目录下的application.properties文件，**设置其中的 nacos.core.auth.plugin.nacos.token.secret.key 值。

### 3. 启动服务器

```properties
startup.cmd -m standalone
```

standalone 代表着单机模式运行。

## Docker

### 1. Clone 项目

```bash
git clone https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
```

### 2. 启动服务

{% hint style="success" %}
单机模式 Derby：

```properties
docker compose -f example/standalone-derby.yaml up
```

单机模式 MySQL 5.7：

```properties
docker compose -f example/standalone-mysql-5.7.yaml up
```

单机模式 MySQL 8：

```properties
docker compose -f example/standalone-mysql-8.yaml up
```
{% endhint %}

{% hint style="success" %}
集群模式：

```
docker compose -f example/cluster-hostname.yaml up
```
{% endhint %}

## 容器可以指定的配置属性

| 属性名称                                          | 描述                                        | 选项                                                                                                                                                                                                                 |
| --------------------------------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| MODE                                          | 系统启动方式: 集群/单机                             | cluster/standalone默认 **cluster**                                                                                                                                                                                   |
| NACOS\_SERVERS                                | 集群地址                                      | p1:port1空格ip2:port2 空格ip3:port3                                                                                                                                                                                    |
| PREFER\_HOST\_MODE                            | 支持IP还是域名模式                                | hostname/ip 默认 **ip**                                                                                                                                                                                              |
| NACOS\_SERVER\_PORT                           | Nacos 运行端口                                | 默认 **8848**                                                                                                                                                                                                        |
| NACOS\_SERVER\_IP                             | 多网卡模式下可以指定IP                              |                                                                                                                                                                                                                    |
| SPRING\_DATASOURCE\_PLATFORM                  | 单机模式下支持MYSQL数据库                           | mysql / 空 默认:空                                                                                                                                                                                                     |
| MYSQL\_SERVICE\_HOST                          | 数据库 连接地址                                  |                                                                                                                                                                                                                    |
| MYSQL\_SERVICE\_PORT                          | 数据库端口                                     | 默认 : **3306**                                                                                                                                                                                                      |
| MYSQL\_SERVICE\_DB\_NAME                      | 数据库库名                                     |                                                                                                                                                                                                                    |
| MYSQL\_SERVICE\_USER                          | 数据库用户名                                    |                                                                                                                                                                                                                    |
| MYSQL\_SERVICE\_PASSWORD                      | 数据库用户密码                                   |                                                                                                                                                                                                                    |
| MYSQL\_SERVICE\_DB\_PARAM                     | 数据库连接参数                                   | default : **characterEncoding=utf8\&connectTimeout=1000\&socketTimeout=3000\&autoReconnect=true\&useSSL=false**                                                                                                    |
| MYSQL\_DATABASE\_NUM                          | 数据库编号                                     | 默认 :**1**                                                                                                                                                                                                          |
| JVM\_XMS                                      | -Xms                                      | 默认 :1g                                                                                                                                                                                                             |
| JVM\_XMX                                      | -Xmx                                      | 默认 :1g                                                                                                                                                                                                             |
| JVM\_XMN                                      | -Xmn                                      | 默认 :512m                                                                                                                                                                                                           |
| JVM\_MS                                       | -XX:MetaspaceSize                         | 默认 :128m                                                                                                                                                                                                           |
| JVM\_MMS                                      | -XX:MaxMetaspaceSize                      | 默认 :320m                                                                                                                                                                                                           |
| NACOS\_DEBUG                                  | 是否开启远程DEBUG                               | y/n 默认 :n                                                                                                                                                                                                          |
| TOMCAT\_ACCESSLOG\_ENABLED                    | server.tomcat.accesslog.enabled           | 默认 :false                                                                                                                                                                                                          |
| NACOS\_AUTH\_SYSTEM\_TYPE                     | 权限系统类型选择,目前只支持nacos类型                     | 默认 :nacos                                                                                                                                                                                                          |
| NACOS\_AUTH\_ENABLE                           | 是否开启权限系统                                  | 默认 :false                                                                                                                                                                                                          |
| NACOS\_AUTH\_TOKEN\_EXPIRE\_SECONDS           | token 失效时间                                | 默认 :18000                                                                                                                                                                                                          |
| NACOS\_AUTH\_TOKEN                            | token                                     | 默认 :SecretKey012345678901234567890123456789012345678901234567890123456789                                                                                                                                          |
| NACOS\_AUTH\_CACHE\_ENABLE                    | 权限缓存开关 ,开启后权限缓存的更新默认有15秒的延迟               | 默认 : false                                                                                                                                                                                                         |
| MEMBER\_LIST                                  | 通过环境变量的方式设置集群地址                           | 例子:192.168.16.101:8847?raft\_port=8807,192.168.16.101?raft\_port=8808,192.168.16.101:8849?raft\_port=8809                                                                                                          |
| EMBEDDED\_STORAGE                             | 是否开启集群嵌入式存储模式                             | `embedded` 默认 : none                                                                                                                                                                                               |
| NACOS\_AUTH\_CACHE\_ENABLE                    | nacos.core.auth.caching.enabled           | default : false                                                                                                                                                                                                    |
| NACOS\_AUTH\_USER\_AGENT\_AUTH\_WHITE\_ENABLE | nacos.core.auth.enable.userAgentAuthWhite | default : false                                                                                                                                                                                                    |
| NACOS\_AUTH\_IDENTITY\_KEY                    | nacos.core.auth.server.identity.key       | default : serverIdentity                                                                                                                                                                                           |
| NACOS\_AUTH\_IDENTITY\_VALUE                  | nacos.core.auth.server.identity.value     | default : security                                                                                                                                                                                                 |
| NACOS\_SECURITY\_IGNORE\_URLS                 | nacos.security.ignore.urls                | default : /,/error,/\*\*/\*.css,/\*\*/\*.js,/\*\*/\*.html,/\*\*/\*.map,/\*\*/\*.svg,/\*\*/\*.png,/\*\*/\*.ico,/console-fe/public/\*\*,/v1/auth/\*\*,/v1/console/health/\*\*,/actuator/\*\*,/v1/console/server/\*\* |

## Nacos 控制台

[http://127.0.0.1:8848/nacos/](http://127.0.0.1:8848/nacos/)
