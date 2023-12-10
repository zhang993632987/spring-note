# 在切点中选择 bean

Spring 引入了一个新的 **bean() 指示器**，它允许我们在切点表达式中使用 **bean 的 ID** 来标识 bean。bean() 使用 bean ID 或 bean 名称作为参数来限制切点只匹配特定的 bean。

例如，考虑如下的切点：

```java
execution(* concert.Performance.perform()) and bean('woodstock')
```

在这里，我们希望在执行 Performance 的 perform() 方法时应用通知，但限定 bean 的 ID 为 woodstock。
