# 处理异常

Spring 提供了多种方式将异常转换为响应：

* **特定的 Spring 异常将会自动映射为指定的 HTTP 状态码；**
* **异常上可以添加 @ResponseStatus 注解，从而将其映射为某一个 HTTP 状态码；**
* **在方法上可以添加 @ExceptionHandler 注解，使其用来处理异常。**

## **1. 自动映射**

<mark style="color:blue;">**在默认情况下，Spring 会将自身的一些异常自动转换为合适的状态码。**</mark>

| Spring 异常                               | HTTP 状态码                     |
| --------------------------------------- | ---------------------------- |
| BindException                           | 400 - Bad Request            |
| ConversionNotSupportedException         | 500 - Internal Server Error  |
| HttpMediaTypeNotAcceptableException     | 406 - Not Acceptable         |
| HttpMediaTypeNotSupportedException      | 415 - Unsupported Media Type |
| HttpMessageNotReadableException         | 400 - Bad Request            |
| HttpMessageNotWritableException         | 500 - Internal Server Error  |
| HttpRequestMethodNotSupportedException  | 405 - Method Not Allowed     |
| MethodArgumentNotValidException         | 400 - Bad Request            |
| MissingServletRequestParameterException | 400 - Bad Request            |
| MissingServletRequestPartException      | 400 - Bad Request            |
| NoSuchRequestHandlingMethodException    | 404 - Not Found              |
| TypeMismatchException                   | 400 - Bad Request            |

## **2. 在异常类上添加 @ResponseStatus**

在自定义<mark style="color:blue;">**异常类**</mark>时，可以在类上可以<mark style="color:blue;">**添加 @ResponseStatus 注解**</mark>，从而将其映射为某一个 HTTP 状态码。

```java
@ResponseStatus(
    value = HttpStatus.BAD_REQUEST,
    reason = "参数校验不通过"
)
public class ValidationException extends RuntimeException {
}
```

## **3. @ExceptionHandler**

在 Controller 中增加一个标注了 <mark style="color:blue;">**@ExceptionHandler**</mark> 注解的方法，它会自动处理该控制器中所有方法抛出的指定异常。

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
@ExceptionHandler(ValidationException.class)
public String handleException(ValidationException exception) {
    return exception.getMessage();
}
```

<mark style="color:blue;">**与该 handleException 方法处于同一个 Controller 中的所有方法，当它们抛出 ValidationException 时，其处理逻辑都会委托给该方法进行处理。**</mark>

如果多个控制器类中都会抛出某个特定的异常，那么你可能会发现要在所有的控制器方法中重复相同的 @ExceptionHandler 方法。或者，为了避免重复，我们会<mark style="color:blue;">**创建一个基础的控制器类，所有控制器类要扩展这个类，从而继承通用的 @ExceptionHandler 方法。**</mark>还可以通过控制器通知 ControllerAdvice 来实现统一的异常处理。
