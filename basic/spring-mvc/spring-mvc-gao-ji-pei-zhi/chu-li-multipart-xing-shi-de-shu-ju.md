# 处理 multipart 形式的数据

一般表单提交所形成的请求结果是很简单的，就是以 & 符分割的多个 name-value 对。尽管这种编码形式很简单，并且对于典型的基于文本的表单提交也足够满足要求，但是对于传送二进制数据，如上传图片，就显得力不从心了。

与之不同的是，<mark style="color:blue;">**multipart 格式的数据会将一个表单拆分为多个部分（part），每个部分对应一个输入域**</mark>。在一般的表单输入域中，它所对应的部分中会放置文本型数据，但是如果上传文件的话，它所对应的部分可以是二进制，下面展现了 multipart 的请求体：

```http
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="firstName"

Charles
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="lastName"

Xavier
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="email"

charles@xmen.com
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
"Content-Disposition: form-data; name="username"

professorx
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="password"

letmeinO1
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW
Content-Disposition: form-data; name="profilePicture"; filename="me.jpg"

Content-Type: image/jpeg

  [[ Binary image data goes here ]]
-----------WebKitFormBoundaryqgkaBn8lHJCuNmiW--
```

在这个 multipart 的请求中，我们可以看到 <mark style="color:blue;">**profilePicture**</mark> 部分与其他部分明显不同。除了其他内容以外，它还有自己的 Content-Type 头，表明它是一个 JPEG 图片。尽管不一定那么明显，但 profilePicture 部分的请求体是二进制数据，而不是简单的文本。

## **1. 配置 multipart 解析器**

### **引入Jakarta Commons FileUpload 的相关依赖**

```xml
<dependency>
  <groupId>commons-fileupload</groupId>
  <artifactId>commons-fileupload</artifactId>
  <version>1.5</version>
</dependency>
```

### 配置 CommonsMultipartResolver

CommonsMultipartResolver 不会强制要求设置<mark style="color:blue;">**临时文件路径**</mark>。<mark style="color:blue;">**默认情况下，这个路径就是 Servlet 容器的临时目录**</mark>。不过，通过设置 uploadTempDir 属性，我们可以将其指定为一个不同的位置：

```java
@Bean
public MultipartResolver multipartResolver2() throws IOException {
    CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();
    multipartResolver.setUploadTempDir(
      new FileSystemResource("/tmp/study/uploads")
    );
    multipartResolver.setMaxUploadSize(2097152);
    multipartResolver.setMaxInMemorySize(0);
    return multipartResolver;
}
```

在这里，我们将最大的文件容量设置为 2MB，最大的内存大小设置为 0 字节。这两个属性表明不能上传超过 2MB 的文件，并且不管文件的大小如何，所有的文件都会写到磁盘中。

## **2. 控制器**

```java
@PostMapping("upload")
public String upload(
    @RequestPart MultipartFile picture,
    @Valid PageBounds pageBounds, @ApiIgnore Errors errors) {
    if (errors.hasErrors())
        return "error";
    return "success";
}
```

<mark style="color:blue;">**MultipartFile**</mark> 提供了获取上传文件 byte 的方式，但是它所提供的功能并不仅限于此，还能获得原始的文件名、大小以及内容类型。它还提供了一个 InputStream，用来将文件数据以流的方式进行读取。除此之外，MultipartFile 还提供了一个便利的 <mark style="color:blue;">**transferTo()**</mark> 方法，它能够帮助我们将上传的文件写入到文件系统中。
