# 解析JWT中的字段

我们将引入一个JWT解析库并将其添加到Gateway服务器的pom.xml文件中。有多个令牌解析器可用，但我们选择了Apache Commons Codec和 org.json 包来解析JSON体。

```xml
<dependency>
     <groupId>commons-codec</groupId>
     <artifactId>commons-codec</artifactId>
</dependency>
<dependency>
     <groupId>org.json</groupId>
     <artifactId>json</artifactId>
     <version>20190722</version>
</dependency>
```

一旦库被添加，我们就可以添加一个名为 getUsername() 的新方法到 TrackingFilter 中。以下清单显示了这个新方法：

```java
private String getUsername(HttpHeaders requestHeaders){
	String username = "";
	if (filterUtils.getAuthToken(requestHeaders)!=null){
		String authToken = filterUtils.getAuthToken(requestHeaders)
			.replace("Bearer ","");
        JSONObject jsonObj = decodeJWT(authToken);
        try {
        	username = jsonObj.getString("preferred_username");
        }catch(Exception e) {
        	logger.debug(e.getMessage());
        }
	}
	return username;
}


private JSONObject decodeJWT(String JWTToken) {
	String[] split_string = JWTToken.split("\\.");
	String base64EncodedBody = split_string[1];
	Base64 base64Url = new Base64(true);
	String body = new String(base64Url.decode(base64EncodedBody));
	JSONObject jsonObj = new JSONObject(body);
	return jsonObj;
}
```

为了使这个示例工作，我们需要确保 **FilterUtils** 中的 **AUTH\_TOKEN** 变量被设置为 **Authorization**。一旦我们实现了 getUsername() 函数，就可以在 TrackingFilter 的 filter() 方法中添加System.out.println，以打印从流经网关的JWT中解析出的 **preferred\_username**。

如果一切成功，您应该在控制台日志中看到以下内容：

```log
tmx-correlation-id found in tracking filter: 26f2b2b7-51f0-4574-9d84-07e563577641. 
The authentication name from the token is : admin
```
