# Parsing a custom field in a JWT

we’ll pull in a JWT parser library and add it to the Gateway server’s pom.xml file. Multiple token parsers are available, but we chose the Apache Commons Codec and the org.json package to parse the JSON body.

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

Once the libraries are added, we can then add a new method called getUsername() to the tracking filter. The following listing shows this new method.



To make this example work, we need to make sure the AUTH\_TOKEN variable in FilterUtils is set to Authorization.



