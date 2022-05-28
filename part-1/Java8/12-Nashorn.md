# <font color="orange">Java 8 Nashorn JavaScript</font>

从 JDK 1.8 开始，**Nashorn** 取代 Rhino（JDK 1.6, JDK1.7）成为 Java 的嵌入式 JavaScript 引擎。Nashorn 完全支持 ECMAScript 5.1 规范以及一些扩展。它使用基于 JSR 292 的新语言特性，其中包含在 JDK 7 中引入的 invokedynamic ，将 JavaScript 编译成 Java 字节码。

> Nashorn JavaScript Engine 在 Java 15 已经不可用了。

## jjs 交互式编程

**jjs** 是个基于 Nashorn 引擎的命令行工具。它接受一些 JavaScript 源代码为参数，并且执行这些源代码。


打开控制台，输入以下命令：

```js
$ jjs
jjs> print("Hello, World!")
Hello, World!
jjs> quit()
```

## 传递参数

打开控制台，输入以下命令：

```js
$ jjs -- a b c
jjs> print('字母: ' +arguments.join(", "))
字母: a, b, c
```

## jjs 执行 .js 文件

例如，我们创建一个具有如下内容的 sample.js 文件：

```js
print('Hello World!');
```

打开控制台，输入以下命令：

```js
$ jjs sample.js
```

以上程序输出结果为：

```
Hello World!
```


