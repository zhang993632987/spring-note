# 2 bean的生命周期

1. Spring 对 bean 进行**实例化**；
2. Spring 将值和 bean 的引用**注入**到 bean 对应的属性中；
3. 如果 bean 实现了 **BeanNameAware** 接口，Spring 将 bean 的 ID 传递给 **setBeanName**()方法；
4. 如果 bean 实现了 **BeanFactoryAware** 接口，Spring 将调用 **setBeanFactory**() 方法，将 BeanFactory 容器实例传入；
5. 如果 bean 实现了 **ApplicationContextAware** 接口，Spring 将调用 **setApplicationContext**() 方法，将 bean 所在的应用上下文的引用传入进来；
6. 如果 bean 实现了 **BeanPostProcessor** 接口，Spring 将调用它们的 **postProcessBeforeInitialization**() 方法；
7. 如果 bean 实现了 **InitializingBean** 接口，Spring 将调用它们的 **afterPropertiesSet**() 方法。类似地，如果 bean 使用 **init-method** 声明了初始化方法，该方法也会被调用；
8. 如果 bean 实现了 **BeanPostProcessor** 接口，Spring 将调用它们的 **postProcessAfterInitialization**() 方法；
9. 此时，bean 已经**准备就绪**，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁；
10. 如果 bean 实现了 **DisposableBean** 接口，Spring 将调用它的 **destroy()** 接口方法。同样，如果 bean 使用 **destroy-method** 声明了销毁方法，该方法也会被调用。
