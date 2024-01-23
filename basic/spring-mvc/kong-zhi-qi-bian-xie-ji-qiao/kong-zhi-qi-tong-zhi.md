# 控制器通知

控制器通知（controller advice）是任意带有 <mark style="color:blue;">**@ControllerAdvice**</mark> 注解的类，这个类会包含一个或多个如下类型的方法：

* <mark style="color:blue;">**@ExceptionHandler**</mark> 注解标注的方法；
* <mark style="color:blue;">**@InitBinder**</mark> 注解标注的方法；
* <mark style="color:blue;">**@ModelAttribute**</mark> 注解标注的方法。

<mark style="color:orange;">**在带有 @ControllerAdvice 注解的类中，以上所述的这些方法会运用到整个应用程序所有控制器中带有 @RequestMapping 注解的方法上。**</mark>

> @ControllerAdvice 注解本身已经使用了 <mark style="color:blue;">**@Component**</mark>，因此 @ControllerAdvice 注解所标注的类将会自动被组件扫描获取到，就像带有 @Component 注解的类一样。

@ControllerAdvice 最为实用的一个场景就是<mark style="color:blue;">**将所有的 @ExceptionHandler 方法收集到一个类中，这样所有控制器的异常就能在一个地方进行一致的处理。**</mark>

```java
@ControllerAdvice
public class ExceptionHandlerAdvice {

    @ResponseBody
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(ValidationException.class)
    public ValidationException handleException(ValidationException exception) {
        return exception;
    }
}
```

> <mark style="color:orange;">**异常处理器的返回值与控制器中方法的返回值并不要求保持一致！**</mark>
>
> 比如在我的实验中，Controller 中的方法返回一个 String 对象，而异常处理器返回的则是一个 Exception 对象！
