# bean的生命周期

<figure><img src="../../.gitbook/assets/spring-framework-ioc-source-102.png" alt=""><figcaption></figcaption></figure>

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

<details>

<summary><mark style="color:purple;">TestLifecycle</mark> </summary>

{% code overflow="wrap" %}
```java
/**
 * @author Zhang B H
 * @create 2024-01-19 14:34
 */
@Slf4j
@Configuration
public class TestLifecycle implements BeanNameAware, BeanFactoryAware,
        ApplicationContextAware, InitializingBean, DisposableBean {

    public TestLifecycle() {
        log.error("3. Bean Constructor");
    }

    @Override
    public void setBeanName(String name) {
        log.error("6. BeanNameAware.setBeanName()");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory)
            throws BeansException {
        log.error("7. BeanFactoryAware.setBeanFactory()");
    }


    @Override
    public void setApplicationContext(ApplicationContext applicationContext)
            throws BeansException {
        log.error("8. ApplicationContextAware.setApplicationContext()");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        log.error("10. InitializingBean.afterPropertiesSet()");
    }

    @Override
    public void destroy() throws Exception {
        log.error("12. InitializingBean.afterPropertiesSet()");
    }

    @Component
    public static class TestBeanFactorPostProcessor
            implements BeanFactoryPostProcessor {
        @Override
        public void postProcessBeanFactory(
                ConfigurableListableBeanFactory beanFactory)
                throws BeansException {
            log.error("1. BeanFactoryPostProcessor.postProcessBeanFactory()");
        }

    }

    @Component
    public static class TestInstantiationAwareBeanPostProcessor
            implements InstantiationAwareBeanPostProcessor {
        @Override
        public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName)
                throws BeansException {
            log.error("2. InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation()");
            return InstantiationAwareBeanPostProcessor.super.postProcessBeforeInstantiation(beanClass, beanName);
        }

        @Override
        public boolean postProcessAfterInstantiation(Object bean, String beanName)
                throws BeansException {
            log.error("4. InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation()");
            return InstantiationAwareBeanPostProcessor.super.postProcessAfterInstantiation(bean, beanName);
        }

        @Override
        public PropertyValues postProcessProperties(
                PropertyValues pvs, Object bean, String beanName)
                throws BeansException {
            log.error("5. InstantiationAwareBeanPostProcessor.postProcessProperties()");
            return InstantiationAwareBeanPostProcessor.super.postProcessProperties(pvs, bean, beanName);
        }
    }

    @Component
    public static class TestBeanPostProcessor implements BeanPostProcessor {
        @Override
        public Object postProcessBeforeInitialization(Object bean, String beanName)
                throws BeansException {
            log.error("9. BeanPostProcessor.postProcessBeforeInitialization()");
            return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
        }

        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName)
                throws BeansException {
            log.error("11. BeanPostProcessor.postProcessBeforeInitialization()");
            return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
        }
    }

    public static void main(String[] args) {
        new AnnotationConfigApplicationContext(TestLifecycle.class);
    }
}
```
{% endcode %}

</details>

输出：

<pre class="language-java"><code class="lang-java">1. BeanFactoryPostProcessor.postProcessBeanFactory()
<strong>2. InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation()
</strong>3. Bean Constructor
4. InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation()
7. BeanFactoryAware.setBeanFactory()
5. InstantiationAwareBeanPostProcessor.postProcessProperties()
6. BeanNameAware.setBeanName()
7. BeanFactoryAware.setBeanFactory()
8. ApplicationContextAware.setApplicationContext()
9. BeanPostProcessor.postProcessBeforeInitialization()
10. InitializingBean.afterPropertiesSet()
11. BeanPostProcessor.postProcessBeforeInitialization()
</code></pre>

> 不知为何，BeanFactoryAware.setBeanFactory() 被调用了两次。除此之外，整个生命周期的执行顺序与上述内容吻合。
