# 创建环绕通知

环绕通知能够让你所编写的逻辑将被通知的目标方法完全包装起来。

<details>

<summary><mark style="color:purple;">使用环绕通知重新实现 Audience 切面</mark></summary>

```java
package concert;

import org.aspect.lang.annotation.ProceedingJoinPoint;
import org.aspect.lang.annotation.Around;
import org.aspect.lang.annotation.Aspect;
import org.aspect.lang.annotation.Pointcut;

@Aspect
public class Audience {

  @Pointcut("execution(** concert.Performance.perform(..))")
  public void performce() { }

  @Around("performce()")
  public void watchPerformance(ProceedingJoinPoint jp) {
    try {
      System.out.println("Silencing cell phones");
      System.out.println("Taking seats");
      jp.procee();
      System.out.println("CLAP CLAP CLAP!!!");
    } catch (Throwable e) {
      System.out.println("Demanding a refund");
    }
  }
}
```

</details>

<mark style="color:orange;">**这个新的通知方法接受 ProceedingJoinPoint 作为参数。这个对象是必须要有的，因为你要在通知中通过它来调用被通知的方法。**</mark>当要将控制权交给被通知的方法时，它需要调用 ProceedingJoinPoint 的 proceed() 方法。

> 需要注意的是，**别忘记调用 proceed() 方法**。**如果不调用这个方法的话，那么你的通知实际上会阻塞对被通知方法的调用。**
>
> **可以在通知中对 proceed() 方法进行多次调用。**要这样做的一个场景就是实现重试逻辑，也就是在被通知方法失败后，进行重复尝试。
