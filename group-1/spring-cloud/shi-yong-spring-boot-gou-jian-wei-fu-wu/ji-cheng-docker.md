# 集成docker

## 1. 添加 docker 的  Maven插件

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>1.4.13</version>
    <configuration>
        <repository>${docker.image.prefix}/${project.artifactId}</repository>
        <tag>${project.version}</tag>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

## 2. 添加 docker.image.prefix 属性值

```xml
<properties>
    <docker.image.prefix>192.168.10.110:5000</docker.image.prefix>
</properties>
```

除了直接在 pom.xml 中定义外，还可以在执行 maven 命令时通过 **-d** 传递 docker.image.prefix 属性值。

## 3. 编写 Dockerfile

{% hint style="warning" %}
Dockerfile 文件的存放路径与 pom.xml 同级！

<img src="../../../.gitbook/assets/image (5) (1) (1) (1).png" alt="" data-size="original">
{% endhint %}

```docker
FROM eclipse-temurin:17-jre as builder

WORKDIR /opt/java/openjdk/lib/security
COPY server.pem .
RUN keytool -import -alias keycloak -file server.pem -keystore cacerts \
        -storepass changeit -noprompt

WORKDIR /application
# 应用程序的JAR文件
ARG JAR_FILE=target/*.jar
# 将应用程序的JAR文件添加到容器中
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM eclipse-temurin:17-jre

WORKDIR /opt/java/openjdk/lib/security
COPY --from=builder /opt/java/openjdk/lib/security/cacerts ./

WORKDIR /application

COPY --from=builder /application/dependencies/ ./
COPY --from=builder /application/spring-boot-loader/ ./
COPY --from=builder /application/snapshot-dependencies/ ./
COPY --from=builder /application/application/ ./

# 开放端口
EXPOSE 8080 8081
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

## 4. 构建镜像

执行 **mvn clean package dockerfile:build** 命令构建镜像。

## 5. 生成容器

```bash
$ docker tag 192.168.10.110:5000/cloud-learning:0.0.1-SNAPSHOT cloud-learning:latest
$ docker run -d --name cloud -p 8080:8080 -p 8081:8081 cloud-learning
```
