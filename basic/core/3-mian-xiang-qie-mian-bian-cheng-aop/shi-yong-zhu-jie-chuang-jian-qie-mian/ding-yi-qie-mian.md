# 定义切面

<details>

<summary><mark style="color:purple;">切面：Audience 类</mark></summary>

```java
package concert;

import org.aspect.lang.annotation.AfterReturning;
import org.aspect.lang.annotation.AfterThrowing;
import org.aspect.lang.annotation.Aspect;
import org.aspect.lang.annotation.Before;

@Aspect
public class Audience {

  @Before("execution(** concert.Performance.perform(..))")
  public void silenceCellPhones() {
    System.out.println("Silencing cell phones");
  }
  
  @Before("execution(** concert.Performance.perform(..))")
  public void takeSeats() {
    System.out.println("Taking seats");
  }
  
  @AfterReturning("execution(** concert.Performance.perform(..))")
  public void applause() {
    System.out.println("CLAP CLAP CLAP!!!");
  }
  
  @AfterThrowing("execution(** concert.Performance.perform(..))")
  public void demandRefund() {
    System.out.println("Demanding a refund");
  }
}
```

</details>

Audience 类使用 <mark style="color:blue;">**@AspectJ 注解**</mark>进行了标注。<mark style="color:blue;">**该注解表明 Audience 不仅仅是一个 POJO，还是一个切面**</mark>。Audience 类中的方法都使用注解来定义切面的具体行为。

**@Pointcut 注解能够在一个 @AspectJ 切面内定义可重用的切点。**

<details>

<summary><mark style="color:purple;">通过 <strong>@Pointcut 注解声明频繁使用的切点表达式</strong></mark></summary>

```java
package concert;

import org.aspect.lang.annotation.AfterReturning;
import org.aspect.lang.annotation.AfterThrowing;
import org.aspect.lang.annotation.Aspect;
import org.aspect.lang.annotation.Before;
import org.aspect.lang.annotation.Pointcut;

@Aspect
public class Audience {

  @Pointcut("execution(** concert.Performance.perform(..))")
  public void performce() { }

  @Before("performce()")
  public void silenceCellPhones() {
    System.out.println("Silencing cell phones");
  }
  
  @Before("performce()")
  public void takeSeats() {
    System.out.println("Taking seats");
  }
  
  @AfterReturning("performce()")
  public void applause() {
    System.out.println("CLAP CLAP CLAP!!!");
  }
  
  @AfterThrowing("performce()")
  public void demandRefund() {
    System.out.println("Demanding a refund");
  }
}
```

</details>

在 Audience 中，performance() 方法使用了 @Pointcut 注解。**为 @Pointcut 注解设置的值是一个切点表达式**，就像之前在通知注解上所设置的那样。

> 需要注意的是，**除了注解和没有实际操作的 performance() 方法，Audience 类依然是一个 POJO**。
>
> 我们**能够像使用其他的 Java 类那样调用它的方法，它的方法也能够独立地进行单元测试**，这与其他的 Java 类并没有什么区别。Audience 只是一个 Java 类，只不过它通过注解表明会作为切面使用而已。

像其他的 Java 类一样，它可以装配为 Spring 中的 bean：

```java
@Bean
public Audience audience() {
  return new Audience();
}
```

**如果就此止步的话，Audience 只会是 Spring 容器中的一个 bean。即便使用了 AspectJ 注解，但它并不会被视为切面，这些注解不会解析，也不会创建将其转换为切面的代理。**

如果使用了 **JavaConfig** 的话，可以<mark style="color:blue;">**在配置类的类级别上通过使用 EnableAspectJAutoProxy 注解启用自动代理功能。**</mark>

<details>

<summary><mark style="color:purple;">在 JavaConfig 中启用 AspectJ 注解的自动代理</mark></summary>

```java
package concert;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Component;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy
@Component
public class ConcertConfig {

  @Bean
  public Audience audience() {
    return new Audience();
  }
}
```

</details>

**AspectJ 自动代理会为使用 @Aspect 注解的 bean 创建一个代理，这个代理会围绕着所有该切面的切点所匹配的 bean**。在本例中，将会为 Concert bean 创建一个代理，Audience 类中的通知方法将会在 perform() 调用前后执行。

> ## 需要记住的是：
>
> Spring 的 AspectJ 自动代理仅仅使用 @AspectJ 作为创建切面的指导，切面依然是基于代理的。在本质上，它依然是 Spring 基于代理的切面。
