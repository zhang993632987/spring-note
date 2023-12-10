# 实例

为所有的 Performance 实现引入下面的 **Encoreable** 接口：

```java
package concert;

public interface Encoreable {
  void performEncore();
}
```

## 第一步：创建切面

为了实现该功能，我们要创建一个新的切面：

```java
package concert;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.DeclareParents;

@Aspect
public class EncodeableIntroducer {

  @DeclareParents(value="concert.Performce+",
                  defaultImpl=DefaultEncoreable.class)
  public static Encoreable encoreable;
}
```

**EncoreableIntroducer 是一个切面，通过 @DeclareParents 注解，将 Encoreable 接口引入到 Performance bean 中。**

@DeclareParents 注解由三部分组成：

* **value** 属性指定了**哪种类型的 bean 要引入该接口**。在本例中，也就是所有实现 Performance 的类型。（标记符后面的加号表示是 Performance 的所有子类型，而不是 Performance 本身。）
* **defaultImpl** 属性指定了**为引入功能提供实现的类**。在这里，我们指定的是 DefaultEncoreable 提供实现。
* @DeclareParents 注解所标注的**静态属性**指明了**要引入的接口**。在这里，我们所引入的是 Encoreable 接口。

## 第二步：将切面声明为一个 Bean

和其他的切面一样，需要在 Spring 应用中将 EncoreableIntroducer 声明为一个 bean：

<pre class="language-java"><code class="lang-java"><strong>@Bean
</strong><strong>public EncoreableIntroducer encoreableIntroducer() {
</strong>    return new EncoreableIntroducer();
}
</code></pre>

之后，Spring 的自动代理机制将会获取到它的声明，当 Spring 发现一个 bean 使用了 @Aspect 注解时，Spring 就会创建一个代理，然后将调用委托给被代理的 bean 或被引入的实现，这取决于**调用的方法属于被代理的 bean 还是属于被引入的接口**。
