# Spring框架中用到了哪些设计模式？

1. 工厂模式：BeanFactor 和 ApplicationContext 利用工厂模式创建 Bean。
2. 代理模式：Spring AOP 使用动态代理使用。
3. 单例模式：spring 中的 bean 的默认都是单例的。
4. 模板方法模式：spring 中的 jdbcTemplate、restTemplate 等使用了模板方法模式。
5. 观察者模式：spring 中的事件驱动模型是观察者模式的经典应用。
6. 包装器模式：
7. 适配器模式：Spring AOP 的增强或通知使用到了适配器模式；Spring MVC 中使用适配器模式适配 Controller。
