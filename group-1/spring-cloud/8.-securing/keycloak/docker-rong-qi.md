# Docker 容器

{% code title="compose.yml" %}
```yaml
services:
  keycloak:    
    image: quay.io/keycloak/keycloak:22.0
    restart: always
    command: ["start-dev"]
    environment:
      KEYCLOAK_ADMIN: admin    
      KEYCLOAK_ADMIN_PASSWORD: 123456    
    ports:
      - "8073:8080"
```
{% endcode %}

> 可以将Keycloak与多个数据库一起使用，如H2、PostgreSQL、MySQL、Microsoft SQL Server、Oracle和MariaDB。在示例中，使用了默认的嵌入式H2数据库。&#x20;
>
> 如果想使用其他数据库，建议访问以下链接：
>
> [https://github.com/keycloak/keycloak-containers/blob/main/docker-compose-examples](https://github.com/keycloak/keycloak-containers/blob/main/docker-compose-examples).

## Keycloak管理控制台：

http://keycloak:8073/
