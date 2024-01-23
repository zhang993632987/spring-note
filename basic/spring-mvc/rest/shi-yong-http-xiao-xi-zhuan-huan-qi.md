# 使用 HTTP 消息转换器

<mark style="color:blue;">**消息转换（message conversion）**</mark>提供了一种更为直接的方式，它能够<mark style="color:blue;">**将控制器产生的数据转换为服务于客户端的表述形式**</mark>。当使用消息转换功能时，DispatcherServlet 不再需要那么麻烦地将模型数据传送到视图中。实际上，这里根本就没有模型，也没有视图，只有控制器产生的数据，以及消息转换器（message converter）转换数据之后所产生的资源表述。

Spring 自带了各种各样的转换器，这些转换器满足了最常见的将对象转换为表述的需要。

| 信息转换器                                | 描述                                                                                                                                                             |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| AtomFeedHttpMessageConverter         | <p>Rome Feed 对象和 Atom feed（媒体类型 application/atom+xml）之间的互相转换。 <br>如果 Rome 包在类路径下将会进行注册</p>                                                                     |
| BufferedImageHttpMessageConverter    | BufferedImages 与图片二进制数据之间互相转换                                                                                                                                  |
| ByteArrayHttpMessageConverter        | 读取/写入字节数组。从所有媒体类型 （_/_）中读取，并以 application/octetstream 格式写入                                                                                                     |
| FormHttpMessageConverter             | 将 application/x-www-form-urlencoded 内容读入到 MultiValueMap 中，也会将 MultiValueMap 写入到 application/x-www-form- urlencoded 中或将 MultiValueMap 写入到 multipart/form-data 中 |
| Jaxb2RootElementHttpMessageConverter | <p>在 XML（text/xml 或 application/xml）和使用 JAXB2 注解的对象间互相读取和写入。<br>如果 JAXB v2 库在类路径下，将进行注册</p>                                                                    |
| MappingJacksonHttpMessageConverter   | <p>在 JOSN 和类型化的对象或非类型化的 HashMap 间互相读取和写入。<br>如果 Jackson JSON 库在类路径下，将进行注册</p>                                                                                  |
| MappingJackson2HttpMessageConverter  | <p>在 JSON 和类型化的对象或非类型化的 HashMap 间互相读取和写入。<br>如果 Jackson 2 JSON 库在类路径下，将进行注册</p>                                                                                |
| MarshallingHttpMessageConverter      | 使用注入的编排器和解排器（marshaller 和 unmarshaller）来读入和写入 XML。支持的编排器和解排器包括 Castor、 JAXB2、JIBX、XMLBeans 以及 Xstream                                                          |
| ResourceHttpMessageConverter         | 读取或写入 Resource                                                                                                                                                 |
| RssChannelHttpMessageConverter       | <p>在 RSS feed 和 Rome Channel 对象间互相读取或写入。<br>如果 Rome 库在类路径下，将进行注册</p>                                                                                           |
| SourceHttpMessageConverter           | <p>在 XML 和 javax.xml.transform.Source 对象间互相读取和写入。<br>默认注册</p>                                                                                                  |
| StringHttpMessageConverter           | 将所有媒体类型（_/_）读取为 String。将 String 写入为 text/plain                                                                                                                 |
| XmlAwareFormHttpMessageConverter     | FormHttpMessageConverter 的扩展，使用 SourceHttp MessageConverter 来支持基于 XML 的部分                                                                                      |

## **@ResponseBody**

正常情况下，当处理方法返回 Java 对象（除 String 外或 View 的实现以外）时，这个对象会放在模型中并在视图中渲染使用。但是，<mark style="color:blue;">**如果使用了消息转换功能的话，我们需要告诉 Spring 跳过正常的模型/视图流程，并使用消息转换器**</mark>。

有不少方式都能做到这一点，但是最简单的方法是为控制器方法添加 <mark style="color:blue;">**@ResponseBody**</mark> 注解：

```java
@RequestMapping(method=RequestMethod.GET, produces="application/json")
public @ResponseBody List<Spittle> spittles(
    @RequestParam(value="max", defaultValue=MAX_LONG_AS_STRING) long max,
    @RequestParam(value="count", defaultValue="20") int count) {
    return spittleRepository.findSpittles(max, count);
}
```

> 请注意 spittles() 的 @RequestMapping 注解。
>
> 在这里，我使用了 <mark style="color:blue;">**produces 属性表明这个方法只处理预期输出为 JSON 的请求**</mark>。也就是说，这个方法只会处理 Accept 头部信息包含 “application/json” 的请求。<mark style="color:blue;">**其他任何类型的请求，即使它的 URL 匹配指定的路径并且是 GET 请求也不会被这个方法处理**</mark>。
>
> 这样的请求会被其他的方法来进行处理（如果存在适当方法的话），或者返回客户端 HTTP 406（Not Acceptable）响应。

<mark style="color:blue;">**@ResponseBody 注解会告知 Spring，我们要将返回的对象作为资源发送给客户端，并将其转换为客户端可接受的表述形式**</mark><mark style="color:blue;">。</mark>更具体地讲，<mark style="color:blue;">**DispatcherServlet 将会考虑到请求中 Accept 头部信息，并查找能够为客户端提供所需表述形式的消息转换器。**</mark>

举例来讲，假设客户端的 Accept 头部信息表明它接受 “application/json”，并且 Jackson JSON 库位于应用的类路径下，那么将会选择 MappingJacksonHttpMessageConverter 或 MappingJackson2HttpMessageConverter（这取决于类路径下是哪个版本的 Jackson）。消息转换器会将控制器返回的 Spittle 列表转换为 JSON 文档，并将其写入到响应体中。

```json
[
    {
        "id":42,
        "latitude":28.419489,
        "longitude":-81.581184,
        "message":"Hello World",
        "time":140038920000
    },
    {
        "id":43,
        "latitude":28.419136,
        "longitude":-81.577225,
        "message":"Blast off!",
        "time":140047560000
    }
]
```

> 在默认情况下，Jackson JSON库在将返回的对象转换为 JSON 资源表述时，会使用反射。对于简单的表述内容来讲，这没有什么问题。但是如果你重构了 Java 类型，比如添加、移除或重命名属性，那么所产生的 JSON 也将会发生变化（如果客户端依赖这些属性的话，那客户端有可能会出错）。
>
> 但是，<mark style="color:blue;">**可以在 Java 类型上使用 Jackson 的映射注解，从而改变产生 JSON 的行为**</mark>。这样就能更多地控制所产生的 JSON，从而防止它影响到 API 或客户端。

## **@RequestBody**

@ResponseBody 能够告诉 Spring 在把数据发送给客户端的时候，要使用某一个消息器，与之类似，<mark style="color:blue;">**@RequestBody 也能告诉 Spring 查找一个消息转换器，将来自客户端的资源表述转换为对象**</mark><mark style="color:blue;">。</mark>

```java
@RequestMapping(method=RequestMethod.POST, consumes="application/json")
public @ResponseBody Spittle saveSpittle(@RequestBody Spittle spittle) {
    return spittleRepository.save(spittle);
}
```

@RequestMapping 表明它只能处理 “/spittles”（在类级别的 @RequestMapping 中进行了声明）的 POST 请求。POST 请求体中预期要包含一个 Spittle 的资源表述。<mark style="color:blue;">**因为 Spittle 参数上使用了 @RequestBody，所以 Spring 将会查看请求中的 Content-Type 头部信息，并查找能够将请求体转换为 Spittle 的消息转换器**</mark><mark style="color:blue;">。</mark>

例如，如果客户端发送的 Spittle 数据是 JSON 表述形式，那么 **Content-Type 头部**信息可能就会是 “application/json”。在这种情况下，DispatcherServlet 会查找能够将 JSON 转换为 Java 对象的消息转换器。如果 Jackson 2 库在类路径中，那么 MappingJackson2HttpMessageConverter 将会担此重任，将 JSON 表述转换为 Spittle，然后传递到 saveSpittle() 方法中。

> 注意，@RequestMapping 有一个 <mark style="color:blue;">**consumes 属性**</mark>，我们将其设置为 “application/json”。
>
> **consumes 属性的工作方式类似于 produces，不过它会关注请求的 Content-Type 头部信息**。**它会告诉 Spring 这个方法只会处理对 “/spittles” 的 POST 请求，并且要求请求的 Content-Type 头部信息为 “application/json”**。
>
> **如果无法满足这些条件的话，会由其他方法（如果存在合适的方法的话）来处理请求**。

## **@RestController**

Spring 4.0 引入了 @RestController 注解，**如果在控制器类上使用 **<mark style="color:blue;">**@RestController**</mark>** 来代替 @Controller 的话，Spring 将会为该控制器的所有处理方法应用消息转换功能**。我们不必为每个方法都添加 **@ResponseBody** 了。

## **ResponseEntity**

作为@ResponseBody 的替代方案，控制器方法可以返回一个 **ResponseEntity** 对象。

* ResponseEntity 中可以包含响应相关的元数据（如头部信息和状态码）以及要转换成资源表述的对象。
* <mark style="color:blue;">**除了包含响应头信息、状态码以及负载以外，ResponseEntity 还包含了 @ResponseBody 的语义**</mark>，因此负载部分将会渲染到响应体中，就像之前在方法上使用 @ResponseBody 注解一样。<mark style="color:orange;">**如果返回 ResponseEntity 的话，那就没有必要在方法上使用 @ResponseBody 注解了**</mark><mark style="color:orange;">。</mark>
