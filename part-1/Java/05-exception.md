# <font color="orange">异常的作用</font>

## 1. 方法作者面对的一个问题

虽然我们面前的代码都是我们一个人编写的，但是我们需要清楚的是，在编码过程中涉及到两个角色：『**类、方法的作者**』和『**类、方法的使用者**』。

只不过，**我们一个人同时扮演了这两个角色**。<small>认识清这一点很重要。</small>

方法的使用者调用某个方法，逻辑上，相当于方法的使用者『要求』方法的作者去执行某个操作，造成某种效果或得到某些数据。但是方法的作者去执行这项任务时有可能执行成功，也有可能执行失败。例如：

> 你调用一个 DAO 方法去删除数据库中的某条数据，但是在执行期间，数据库所在的服务器可能断电，也可能断网，从而导致客户端<small>（你的代码）</small>和数据库服务器连接断开，那么，这个 DAO 方法的执行当然就不会成功。

简单来说就是，你去调用一个方法，没有人能保证这个方法的执行一定就会成功的！就算被调方法的代码是 100% 正确的，你调用方法最后的结果也不一定是执行成功。例如上例就是。

那方法的作者如何告知方法的使用者：**你要我去执行的事情没能成功完成，我执行失败了，并且失败原因是 xxx** ？


## 2. 没有异常的编程世界是怎么玩的

上述问题不单单是 Java 程序员要面对的问题，在 Java 之前，甚至是在 C++ 之前，程序员就要面对并解决这个问题。

> 有没有更偏门的、小众的编程语言早于 C++ 提出异常的概念这？个我没有深入研究，但是在通用语言中，C++ 是第一个提出『异常』语法特性的编程语言。C++ 之前的『著名语言』中，都没有异常的概念。

### 2.1 方案一

对于『告知调用者调用方法失败』，最简单最容易想到的办法是返回一个 boolean 值，用 boolean 值的 true 和 false 来表示『调用成功』和『调用失败』：

```java
boolean doSomething() {
    ...

    if (...)    // 情况 1 导致的失败
        return false;
    ...

    if (...)    // 情况 2 导致的失败
        return false;
    ...

    if (...)    // 情况 3 导致的失败
        return false;
    ...

    return true;    // 成功
}
```

上面的代码很好理解，但是『返回 boolean 值』这种方案很明显地有一个缺陷：可以表示成功和失败，**但是无法表示失败的原因**。

`doSomething()` 方法的调用者在获得返回值 false 之后，能得知该方法执行失败，但不知道具体的失败原因。


### 2.2 方案二

为了解决方案一的缺陷，程序员想出了方案二：返回一个 int 值，0 表示成功，非 0 值表示失败，非 0 值又可称**错误码**。

上面的代码就会改造成如下形式：

```java
int doSomething() {
    ...

    if (...)    // 情况 1 导致的失败
        return 1;
    ...

    if (...)    // 情况 2 导致的失败
        return 2;
    ...

    if (...)    // 情况 3 导致的失败
        return 3;
    ...

    return 0;    // 成功
```

方法的调用者可以通过 `doSomething()` 方法的返回值来得知成功与否，以及失败的原因。

但是方案二仍有两个缺陷：

1. 返回的错误码是**无语义**的。即，错误码只是一个代号，如果没有配套的文档进行说明，方法调用者单凭错误码无法得知错误的原因。

2. 返回 0 或错误码会『占用』返回值类型。如果方法本身在执行正常时需要返回一个字符串，那么这个方法**不可能有时返回字符串，有时返回 int**，因为方法返回值类型只能有一个。

### 2.3 方案三

方案二的缺陷不是不能解决：

- 可以通过枚举来让错误码语义化，解决方案一的第一个缺陷；

- 可以通过结果参数，或者是联合体/结构体/类来包装返回值，解决方案二的第二个缺陷。

但是上述的改进都是程序员层面“强行”想出的解决办法，而并非编程语言本身所提供的“便利”，需要程序员进行“额外”的编码才能达到预期的必要的效果。

在发现『告知方法调用者方法调用失败，并且失败原因是 xxx』是编程必要需求时，开始要求从编程语言本身的语法层面上解决这个需求。


## 3. 从语法层面解决上述问题：异常

异常概念的提出就是要从编程语言的语法层面上解决『返回错误信息』的问题。

相较于上述的方案三，异常从语法层面上弥补了方案二的缺陷：

- 通过自定义异常来代替错误码，实现错误码所没有的语义话；

- 异常是从被调方法中抛出，而不是返回，因此异常不会占用方法返回值类型。

上述的代码在引入异常概念后将会变成如下形式：

```java
String doSomething() {
    ...

    if (...)    // 情况 1 导致的失败
        throw new XxxException;
    ...

    if (...)    // 情况 2 导致的失败
        throw new YyyException;
    ...

    if (...)    // 情况 3 导致的失败
        throw new ZzzException;
    ...

    return "该返回啥返回啥";    // 成功
```

## 4. 异常堆栈 

在 Java 中，异常类本身代表着一种错误原因/种类，在异常对象中还包含着整个异常链。

即，a 方法执行不成功是因为其中的代码在调用 b 方法执行失败；而 b 方法执行失败，是因为它在调用 c 方法时 c 方法执行失败；而 c 方法执行失败的原因又是它在执行 d 方法时调用 e 时 e 方法执行失败 ...，直到最终的最底层原因。
