# Windows

**Nacos 依赖 Java 环境运行，因此需要先保证安装了 Java 环境。**

## 1. 下载安装包包

地址：[https://github.com/alibaba/nacos/releases](https://github.com/alibaba/nacos/releases)

## 2. 修改配置文件

> **在 2.2.0.1 和 2.2.1 版本时，必须执行此变更，否则无法启动；其他版本为建议设置。**

**修改 conf 目录下的application.properties文件，**设置其中的 nacos.core.auth.plugin.nacos.token.secret.key 值。

## 3. 启动服务器

```properties
startup.cmd -m standalone
```

standalone 代表着单机模式运行。
