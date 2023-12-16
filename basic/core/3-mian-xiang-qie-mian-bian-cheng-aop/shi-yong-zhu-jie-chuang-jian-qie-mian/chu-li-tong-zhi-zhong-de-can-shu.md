# 处理通知中的参数

<details>

<summary><mark style="color:purple;">参数化的通知</mark></summary>

```java
package soundsystem;

import java.util.HashMap;
import java.util.Map;
import org.aspect.lang.annotation.Aspect;
import org.aspect.lang.annotation.Before;
import org.aspect.lang.annotation.Pointcut;

@Aspect
public class TrackCounter {

  private Map<Integer, Integer> trackCounts = new HashMap<>();
  
  @Pointcut("execution(* soundsystem.CompactDisc.playTrack(int) " +
            "&& args(trackNumber)")
  public void trackPlayed(int trackNumber) { }

  @Before("trackPlayed(trackNumber)")
  public void countTrack(int trackNumber) {
    int currentCount = getPlayCount(trackNumber);
    trackCounts.put(trackNumber, currentCount + 1);
  }
  
  public int getPlayCount(int trackNumber) {
    return trackCounts.containsKey(trackNumber) ? 
      trackCounts.get(trackNumber) : 0;
  }
}
```

</details>

这个切面使用 **@Pointcut** 注解定义命名的切点，并使用 **@Before** 将一个方法声明为前置通知。

下图将切点表达式进行了分解，以展现参数是在什么地方指定的：

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* <mark style="color:blue;">**切点表达式中的 args(trackNumber) 限定符表明传递给 playTrack() 方法的 int 类型参数也会传递到通知中去**</mark>。
* **参数的名称 trackNumber 也与切点方法签名中的参数相匹配。**这个参数会传递到通知方法中，这个通知方法是通过 @Before 注解和命名切点 trackPlayed(trackNumber) 定义的。**切点定义中的参数与切点方法中的参数名称是一样的**，这样就完成了从命名切点到通知方法的参数转移。
*
