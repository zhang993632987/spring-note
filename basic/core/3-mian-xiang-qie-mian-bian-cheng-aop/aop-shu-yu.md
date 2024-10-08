# AOP 术语

## 通知（Advice）

在 AOP 中，切面的工作被称为通知（Advice）。

<mark style="background-color:blue;">**通知定义了切面是**</mark><mark style="color:blue;background-color:blue;">**什么**</mark><mark style="background-color:blue;">**以及**</mark><mark style="color:blue;background-color:blue;">**何时**</mark><mark style="background-color:blue;">**使用。除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。**</mark>

Spring 切面可以应用 5 种类型的通知：

* **前置通知（Before）**：在目标方法被调用之前调用通知功能；
* **后置通知（After）**：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
* **返回通知（After-returning）**：在目标方法成功执行之后调用通知；
* **异常通知（After-throwing）**：在目标方法抛出异常后调用通知；
* **环绕通知（Around）**：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

## **连接点（Join point）**

我们的应用可能有数以千计的时机应用通知，这些时机被称为连接点。

<mark style="background-color:blue;">**连接点是在应用执行过程中能够插入切面的一个点**</mark>，这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。

切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。

## **切点（Poincut）**

一个切面并不需要通知应用的所有连接点，**切点有助于缩小切面所通知的连接点的范围**。

如果说通知定义了切面的“什么”和“何时”的话，那么切点就定义了“何处”。<mark style="background-color:blue;">**切点的定义会匹配通知所要织入的一个或多个连接点**</mark><mark style="background-color:blue;">。</mark>通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。

## **切面（Aspect）**

<mark style="background-color:blue;">**切面是通知和切点的结合**</mark><mark style="background-color:blue;">。</mark>

通知和切点共同定义了切面的全部内容 —— 它是什么，在何时和何处完成其功能。

## **引入（Introduction）**

**引入允许我们向现有的类添加新方法或属性。**

例如，可以创建一个 Auditable 通知类，该类记录了对象最后一次修改时的状态。这很简单，只需一个方法，setLastModified(Date)，和一个实例变量来保存这个状态。然后，这个新方法和实例变量就可以被引入到现有的类中，从而可以在无需修改这些现有的类的情况下，让它们具有新的行为和状态。

## **织入（Weaving）**

<mark style="background-color:blue;">**织入是把切面应用到目标对象并创建新的代理对象的过程。**</mark>切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入：

* **编译期**：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ 的织入编译器就是以这种方式织入切面的。
* **类加载期**：切面在目标类加载到 JVM 时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5 的加载时织入（load-time weaving，LTW）就支持以这种方式织入切面。
* <mark style="color:blue;">**运行期：**</mark>切面在应用运行的某个时刻被织入。一般情况下，<mark style="color:blue;">**在织入切面时，AOP 容器会为目标对象动态地创建一个代理对象。Spring AOP 就是以这种方式织入切面的。**</mark>
