# 配置自定义的用户服务

假设我们需要认证的用户存储在非关系型数据库中，如 Mongo 或 Neo4j，在这种情况下，我们需要提供一个自定义的 UserDetailsService 接口实现。

<mark style="color:blue;">**UserDetailsService**</mark> 接口非常简单：

```java
public interface UserDetailsService {
  UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

我们所需要做的就是实现 **loadUserByUsername()** 方法，**根据给定的用户名来查找用户**。loadUserByUsername() 方法会返回代表给定用户的 UserDetails 对象。

如下的程序清单展现了一个 UserDetailsService 的实现，它会从给定的 SpitterRepository 实现中查找用户：

```java
package spittr.security;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import spittr.Spitter;
import spittr.data.SpitterRepository;

public class SpitterUserService(SpitterRepository spitterRepository) {
  
  private final SpitterRepository spitterRepository;
  
  public SpitterUserService(SpitterRepository spitterRepository){
    this.spitterRepository = spitterRepository;
  }
  
  @Override
  public UserDetails loadUserByUsername(String username) 
        throws UsernameNotFoundException {  
    Spitter spitter = spitterRepository.findByUsername(username);
    if (spitter != null) {
      List<GrantedAuthority> authorities = new ArrayList<>();
      authorities.add(new SimpleGrantedAuthority("ROLE_SPITTER"));
      return new User(spitter.getUsername(), spitter.getPassword(), authorities);
    }
    
    throw new UsernameNotFoundException("User '" + username + "' not found.");
  }
  
}
```

为了使用 **SpitterUserService** 来认证用户，我们可以通过 <mark style="color:blue;">**userDetailsService()**</mark> 方法将其设置到安全配置中：

```java
@Autowired
SpitterRepository spitterRepository;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .userDetailsService(new SpitterUserService(spitterRepository));
}
```

**userDetailsService() 方法会配置一个用户存储**。不过，这里所使用的不是 Spring 所提供的用户存储，而是使用 UserDetailsService 的实现。
