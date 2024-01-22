# Spring Boot的核心注解

启动类上面的注解是 @SpringBootApplication，它便是SpringBoot的核心注解，主要组合了以下 3 个注解：

* **@SpringBootConfiguration**：组合了 **@Configuration** 注解，实现配置文件的功能；
* **@EnableAutoConfiguration**：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置的功能：@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})；
* **@ComponentScan**：Spring 组件扫描。
