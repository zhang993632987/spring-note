# Spring 对 AOP 的支持

<mark style="color:orange;">**Spring AOP 构建在动态代理基础之上，因此，Spring 对 AOP 的支持局限于方法拦截。**</mark>

*   [x] Spring 所创建的通知都是用标准的 Java 类编写的。

    而且，定义通知所应用的切点通常会使用注解或在 Spring 配置文件里采用 XML 来编写。
* [x] **Spring在运行时通知对象**
  *   ### 通过在代理类中包裹切面，Spring 在运行期把切面织入到 Spring 管理的 bean 中。

      如图所示，代理类封装了目标类，并拦截被通知方法的调用，再把调用转发给真正的目标 bean。当代理拦截到方法调用时， 在调用目标 bean 方法之前，会执行切面逻辑。

      <figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>
  *   直到应用需要被代理的 bean 时，Spring 才创建代理对象。

      如果使用的是 ApplicationContext 的话，在 ApplicationContext 从 BeanFactory 中加载所有 bean 的时候，Spring 才会创建被代理的对象。
  * 因为 Spring 运行时才创建代理对象，所以我们不需要特殊的编译器来织入 Spring AOP 的切面。
* [x] **Spring 只支持方法级别的连接点**
  * 因为 Spring 基于动态代理，所以 Spring 只支持方法连接点。
  * Spring 缺少对字段连接点的支持，无法让我们创建细粒度的通知，例如拦截对象字段的修改。
  * 而且它不支持构造器连接点，我们就无法在 bean 创建时应用通知。
