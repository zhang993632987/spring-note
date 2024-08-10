# 编写切点

以下展示了一个切点表达式，这个表达式能够设置当 perform() 方法执行时触发通知的调用：

```java
execution(* concert.Performance.perform(..))
```

1. 使用 **execution()** 指示器选择 Performance 的 perform() 方法。
2. 方法表达式**以 "\*" 号开始**，表明了我们**不关心方法返回值的类型**。
3. 然后，我们指定了**全限定类名和方法名**。
4. 对于**方法参数列表**，使用**两个点号（..）表明切点要选择任意的 perform() 方法，无论该方法的入参是什么。**

现在假设我们需要配置的切点仅匹配 concert 包。在此场景下，可以使用 **within()** 指示器来限制匹配：

```java
execution(* concert.Performance.perform(..)) && within("concert.*")
```

> 逻辑运算符：
>
> * **"&&" 和 "and"**：表示与（and）关系。因为 "&" 在 XML 中有特殊含义，所以在 Spring 的 XML 配置里面描述切点时，应该使用 and 来代替 "&&"。
> * **"||" 和 "or"**：表示或（or）关系。
> * **"!" 和 "not"**：表示非（not）。
