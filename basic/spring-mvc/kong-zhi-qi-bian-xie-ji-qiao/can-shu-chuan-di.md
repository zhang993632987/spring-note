# 参数传递

Spring MVC 允许以多种方式将客户端中的数据传送到控制器的处理器方法中，包括：

## 查询参数（Query Parameter）

```java
@GetMapping("query")
public String queryParam(
    @RequestParam int page,
    @RequestParam int count) {
    return page + " : " + count;
}
```

```java
@Test
public void queryParam() throws Exception {
      mockMvc.perform(
          MockMvcRequestBuilders.get("/query?page=1&count=5")
      ).andExpect(MockMvcResultMatchers.content().string("1 : 5"));
}
```

## 表单参数（Form Parameter）

```java
@Data
public class PageBounds {
    private int page;
    private int count;
}

@GetMapping("form")
public String form(PageBounds pageBounds) {
  return pageBounds.getPage() + " : " + pageBounds.getCount();
}
```

```java
@Test
public void form() throws Exception {
      mockMvc.perform(
          MockMvcRequestBuilders.get("/form")
          .queryParam("page", "1")
          .queryParam("count", "5")
      ).andExpect(MockMvcResultMatchers.content().string("1 : 5"));
}
```

## 路径变量（Path Variable）

```java
@GetMapping("path/{page}/{count}")
public String pathVariable(
    @PathVariable int page,
    @PathVariable int count) {
    return page + " : " + count;
}
```

```java
@Test
public void pathVariable() throws Exception {
    mockMvc.perform(
        MockMvcRequestBuilders.get("/path/1/5")
    ).andExpect(MockMvcResultMatchers.content().string("1 : 5"));
}
```
