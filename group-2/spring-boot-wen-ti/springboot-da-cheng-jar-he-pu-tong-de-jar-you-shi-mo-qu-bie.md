# SpringBoot 打成 jar 和普通的 jar 有什么区别？

Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过 java -jar xxx.jar 命令运行，**这种 jar 不可以作为普通的 jar 被其他项目依赖**，即使依赖了也无法使用其中的类。

Spring Boot 的 jar 无法被其他项目依赖，主要还是它和普通 jar 的结构不同：

* 普通的 jar 包，解压后直接就是包名，包里就是我们的代码；
* 而 Spring Boot 打包成的可执行 jar 解压后，在 \BOOT-INF\classes 目录下才是我们的代码，因此无法被直接引用。

如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。

> **一般来说，Spring Boot 直接打包成可执行 jar 就可以了，不建议将 Spring Boot 作为普通的 jar 被其他的项目所依赖。如果有这种需求，建议将被依赖的部分，单独抽出来做一个普通的 Maven 项目，然后在 Spring Boot 中引用这个 Maven 项目。**
