# UriComponentBuilder

**当创建新资源的时候，将资源的 URL 放在响应的 Location 头部信息中，并返回给客户端是一种很好的方式。**

Spring 提供了 <mark style="color:blue;">**UriComponentsBuilder**</mark>，它<mark style="color:blue;">**是一个构建类，通过逐步指定 URL 中的各种组成部分（如 host、端口、路径以及查询），我们能够使用它来构建 UriComponents 实例**</mark>。借助 UriComponentsBuilder 所构建的 UriComponents 对象，我们就能获得适合设置给 Location 头部信息的 URI。

为了使用 UriComponentsBuilder，我们需要做的就是<mark style="color:blue;">**在处理器方法中将其作为一个参数**</mark>，如下面的程序清单所示。

```java
@RequestMapping(method=RequestMethod.POST, consumes="application/json")
@ResponseStatus(HttpStatus.CREATED)
public ResponseEntity<Spittle> saveSpittle(
  @RequestBody Spittle spittle, 
  UriComponentsBuilder ucb) {
  Spittle saved = spittleRepository.save(spittle);
    
  HttpHeaders headers = new HttpHeaders();
  URI locationUri = ucb.path("/spittles/")
      .path(String.valueOf(saved.getId()))
      .build()
      .toUri();
  headers.setLocation(locationUri);
    
  ResponseEntity<Spittle> responseEntity = new ResponseEntity<Spittle>(
          saved, headers, HttpStatus.CREATED);
  return responseEntity;
}
```

**在处理器方法所得到的 UriComponentsBuilder 中，会预先配置已知的信息如 host、端口以及 Servlet 内容**。它<mark style="color:blue;">**会从处理器方法所对应的请求中获取这些基础信息**</mark>。基于这些信息，代码会通过设置路径的方式构建 UriComponents 其余的部分。
