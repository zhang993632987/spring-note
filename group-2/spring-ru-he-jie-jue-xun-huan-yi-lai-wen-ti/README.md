# Spring 如何解决循环依赖问题

{% hint style="warning" %}
## <mark style="color:orange;">注意</mark>

<mark style="color:orange;">**Spring 只解决了单例模式下属性依赖的循环问题；**</mark>

为了解决单例的循环依赖问题，Spring 使用了三级缓存。
{% endhint %}

## 三级缓存

{% code overflow="wrap" %}
```java
/** Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
 
/** Cache of early singleton objects: bean name --> bean instance */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** Cache of singleton factories: bean name --> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);

```
{% endcode %}

* **第一层缓存（singletonObjects）**：单例对象缓存池，**已经实例化并且属性已经完成赋值**，这里的对象是**成熟对象**；
* **第二层缓存（earlySingletonObjects）**：单例对象缓存池，**已经实例化但尚未完成属性赋值**，这里的对象是**半成品对象**；
* **第三层缓存（singletonFactories）**: 单例工厂的缓存

如下是获取单例 Bean 对象的方法：

{% code overflow="wrap" %}
```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
  // Spring首先从singletonObjects（一级缓存）中尝试获取
  Object singletonObject = this.singletonObjects.get(beanName);
  // 若是获取不到而且对象正在建立中，则尝试从earlySingletonObjects(二级缓存)中获取
  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    synchronized (this.singletonObjects) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        // 若仍然获取不到对象，而且容许从singletonFactories经过getObject获取，
        // 则经过singletonFactory.getObject()(三级缓存)获取
        if (singletonObject == null && allowEarlyReference) {
          ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
          if (singletonFactory != null) {
              singletonObject = singletonFactory.getObject();
              // 若是获取到了则将singletonObject放入到earlySingletonObjects,
              // 也就是将三级缓存提高到二级缓存中
              this.earlySingletonObjects.put(beanName, singletonObject);
              this.singletonFactories.remove(beanName);
          }
        }
    }
  }
  return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```
{% endcode %}

getSingleton() 方法的逻辑如下：

1. 首先从一级缓存 singletonObjects 中获取 Bean 对象；
2. 若获取不到且对象正在建立中，再从二级缓存 earlySingletonObjects 中进行获取；
3. 若是仍是获取不到且容许通过 singletonFactory.getObject() 获取，就从三级缓存 singletonFactory (三级缓存)获取单例工厂对象，调用工厂对象的 getObject 方法构造对象，并将构造完成的对象放入二级缓存中。

Spring 解决循环依赖的关键就在于 singletonFactories 这个三级缓存，缓存中保存的对象是 ObjectFactory 接口类型，该接口定义如下：

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```

在 Bean 建立过程中，调用了以下方法填充第三层缓存：

```java
addSingletonFactory(beanName, new ObjectFactory<Object>() {
   @Override   
   public Object getObject() throws BeansException {
      return getEarlyBeanReference(beanName, mbd, bean);
   }
});
```

## 循环依赖的解决方法

假设：A 对象通过 setter 方法注入了一个 B 对象，同时 B 对象通过 setter 方法注入了一个 A 对象，A 对象和 B 对象构成了循环依赖。

1. A 首先完成初步的初始化（未注入依赖项），而且将本身提早曝光到 singletonFactories 中，此时进行初始化的第二步（注入依赖项），发现依赖对象 B，此时就尝试去 get(B)，发现 B 尚未被 create，因此走 B 的 create 流程；
2. 同样，B 在初始化第二步的时候发现本身依赖了对象 A，因而尝试 get(A)：尝试一级缓存 singletonObjects (确定没有，由于A还没初始化彻底)，尝试二级缓存 earlySingletonObjects（也没有），尝试三级缓存 singletonFactories，因为 A 经过 ObjectFactory 将本身提早曝光了，因此 B 可以经过 ObjectFactory.getObject 拿到 A 对象(半成品)，B 拿到 A 对象后顺利完成了初始化，彻底初始化以后将本身放入到一级缓存 singletonObjects 中。
3. 接着继续 A 的初始化流程中，此时 A 能拿到依赖对象 B，顺利完成自身的初始化，最终 A 也完成了初始化，被放入到了一级缓存 singletonObjects 中；
4. 因为 B 注入了 A 的对象引用，因此 B 如今拥有的 A 对象变成了一个完成了初始化的对象，而不是一个半成品。
