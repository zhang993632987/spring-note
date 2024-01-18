# Spring为什么不能解决prototype作用域循环依赖？

**因为 spring 不会缓存 prototype 作用域的 bean，而 spring 中循环依赖的解决是通过缓存来实现的。**
