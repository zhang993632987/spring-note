# 通过注解引入新功能

<mark style="color:blue;">**使用 Spring AOP，可以为 bean 引入新的方法。代理拦截对新方法的调用，并将调用委托给实现该方法的其他对象**</mark>，原理如下图所示：

<figure><img src="../../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

需要注意的是，**当引入接口的方法被调用时，代理会把此调用委托给实现了新接口的某个其他对象**。实际上，一个 bean 的实现被拆分到了多个类中。



为了验证该主意能行得通，我们
