# <font color="orange">Lambda 表达式</font>

在 Java 编程语言中，方法是『二等公民』。一个典型的现象就是：你无法将一个方法（或者说一段代码）直接作为参数传给一个方法。

> 想实现这样的目的（在 Java 8 之前）你只能采用间接的方式：将方法（一段代码）定义为一个类的实例方法，给目标方法传入这个类的一个对象，再在目标方法内再来调用这个方法。

Java 8 的一个核心升级就是 lambda 表达式。Sun（Oracle）公司借助于『接口』，很巧妙地实现了 lambda 表达式语法。这样，就给初学者提供了一个更简单的了解、学习 lambda 表达式的途径：将它当作接口的升级、缩写。

我们以 Runnable 接口为例来讲述从接口到 lambda 表达式的演变过程。

## 1. 接口的原始形式

这是 Runnbale 接口的一个实现类。使用它时，我们需要创建 Hello 类的对象，将其传给目标方法。

```java
class Hello implements Runnable {

    @Override
    public void run() {
        System.out.println("hello world");
    }
}
```

```java
Hello hello = new Hello();
Thread thread = new Thread(hello);
thread.start();
```

## 2. 改造 1：接口的匿名实现类

注意，这一步改造和 lambda 表达式无关，这里所涉及的语法是『接口』本身就有就存在的语法。

如果我们不会重复利用 Hello 这个类，那么我们可以不把 Hello 这个类的定义独立写成一个 `.java` 文件，而是直接『内嵌』到它所使用的那个地方（既然不重复利用它，那自然也就只有那一处地方）。

```java
// Hello hello = new Hello();
Runnable hello = new Runnable() {
    @Override
    public void run() {
        System.out.println("hello world");
    }
};
```

Hello 类不以独立的类定义的形式存在，此时，上述代码就称为 Runnable 接口的匿名实现类。

## 3. 改造 2：省略引用变量

由于 `hello` 变量也没有被重复利用，定义后仅出现在了 `Thread thread = new Thread(hello);` 这一行，因此，我们可以将两行整合在一起，省略掉 `hello` 变量。

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("hello world");
    }
});
```

## 4. 改造 3：lambda 表达式的初级形式

改造到上一步，我们就可以开始进入到 lambda 表达式的范畴。

Runnable 接口中所定义的方法只有一个，所有它具有唯一性。因此，理论上我们完全可以省略掉 `run()` 方法的声明，因为我们只要一谈论『 Runnable 接口的实现类』，大家都知道我们要实现的方法是怎样的。

```java
Thread thread = new Thread(() -> {
    System.out.println("hello world");
});
```

由于 lambda 表达式是嵌在 `new Thread()` 中的，所以，将 lambda 表达式单独提取出来就是这样的：

```java
() -> {
    System.out.println("hello world");
}
```

这里的 `()` 就是 run 方法的参数列表的那个 `()`，`->` 后面跟的代码片段（`{}`）就是原来 run 方法的方法体。

由于 Runnable 接口的 run 方法是无参的，所以上面的 `()` 中是空的。如果接口的方法是有参数的，那么 `()` 中的内容给九是方法的形式参数泪飙。

以 Predicate 接口为例，Predicate 接口的 test 方法的原型是：`boolean test(String s)` ，那么它的 lambda 表达式的形式就是：

```java
(String s) -> {
    ...
}
```

## 5. 改造 4：省略 lambda 表达式的参数类型

对于有参数的接口的方法的 lambda 表达式的参数，参数的类型声明不是必须的，可省略。以 **Predicate** 接口的 `test` 方法为例：

```java
(String s) -> {
    ...
}
```

可以改造为：

```java
(s) -> {
    ...
}
```

## 6. 改造 5：一行代码的 lambda 表达式的缩写

如果 lambda 表达式的方法中有且仅有一行，那么 `{}` 以及这一行代码后的 `;` 是可以省略的。

还是回到上述的 Runnable 接口的 run 方法：

```java
() -> {
    System.out.println("hello world");
}
```

可以简写为：

```java
() -> System.out.println("hello world")
```

这样，`new Thread()` 就写成了：

```java
Thread thread = new Thread(() -> System.out.println("hello world"));
```

后续，还可以再进一步的简写一点，不过那就是非必要部分的。

## 7. 变量作用域

不少人在使用 Lambda 表达式的尝鲜阶段，可能都遇到过一个错误提示：

    Variable used in lambda expression should be final or effectively final

以上报错，就涉及到外部变量在 Labmda 表达式中的作用域，且有以下几个语法规则。

### 7.1 变量作用域的规则

『**局部变量**』是指在我们普通的方法内部，且在 Lambda 表达式外部声明的变量。

- 规则 1：**局部变量不可变，域变量或静态变量是可变的**

  在 Lambda 表达式内使用局部变量时，该局部变量必须是不可变的。

  ```java
  public class AClass {
    private Integer num1 = 1;
    private static Integer num2 = 10;

    public void testA() {
        int a = 1;
        int b = 2;
        int c = 3;
        a++;
        new Thread(() -> {
            System.out.println("a=" + a); // 在 Lambda 表达式使用前有改动，编译报错
            b++; // 在 Lambda 表达式中更改，报错
            System.out.println("c=" + c); // 在 Lambda 表达式使用之后有改动，编译报错

            System.out.println("num1=" + this.num1++); // 对象变量，或叫域变量，编译通过
            AClass.num2 = AClass.num2 + 1;
            System.out.println("num2=" + AClass.num2); // 静态变量，编译通过
        }).start();
        c++;
    }
  }
  ```
  
- 规则 2：**表达式内的变量名不能与局部变量重名，域变量和静态变量不受限制**

  ```java
  public class AClass {
    private Integer num1 = 1;
    private static Integer num2 = 10;

    public void testA() {
        int a = 1;
        new Thread(() -> {
            int a = 3; // 与外部的局部变量重名，编译报错
            Integer num1 = 232; // 虽与域变量重名，允许，编译通过
            Integer num2 = 11; // 虽与静态变量重名，允许，编译通过
        }).start();
    }
  }
  ```

  虽然域变量和静态变量可以重名，从可读性的角度考虑，最好也不用重复，养成良好的编码习惯。

- 规则 3：**可使用 this、super 关键字，等同于在普通方法中使用**

  ```java
  public class AClass extends ParentClass {
    @Override
    public void printHello() {
        System.out.println("subClass: hello budy!");
    }

    @Override
    public void printName(String name) {
        System.out.println("subClass: name=" + name);
    }

    public void testA() {
        this.printHello();  // 输出：subClass: hello budy!
        super.printName("susu"); // 输出：ParentClass: name=susu

        new Thread(() -> {
            this.printHello();  // 输出：subClass: hello budy!
            super.printName("susu"); // 输出：ParentClass: name=susu
        }).start();

    }
  }

  class ParentClass {
    public void printHello() {
        System.out.println("ParentClass: hello budy!");
    }

    public void printName(String name) {
        System.out.println("ParentClass: name=" + name);
    }
  }
  ```

- 规则 4：**不能使用接口中的默认方法（default 方法）**

  ```java
  public class AClass implements testInterface {
    public void testA() {
        new Thread(() -> {
            String name = super.getName(); // 编译报错：cannot resolve method 'getName()'
        }).start();
    }
  }

  interface testInterface {
    // 默认方法
    default public String getName() {
        return "susu";
    }
  }
  ```



### 7.2 为何要 final？

不管是 Lambda 表达式，还是匿名内部类，编译器都要求了『**变量必须是 final 类型的，即使不显式声明，也要确保没有修改**』。

为何编译器要强制设定变量为 final 或 effectively final 呢？

1. 引入的局部变量是副本，改变不了原本的值。

2. 局部变量存于栈中，多线程中使用有问题。

3. 线程安全问题。


## 8. 方法引用

一句话介绍：

> 方法引用<small>（Method Reference）</small>是在 Lambda 表达式的基础上引申出来的一个功能。

### 8.1 示例

```java
List<Integer> list = Arrays.asList(1, 2, 3);
list.forEach(num -> System.out.println(num));
```

上面是一个很普通的 Lambda 表达式：*遍历打印列表的元素* 。

相比 JDK 8 版本以前的 *for* 循环或 *Iterator* 迭代器方式，这种 Lambda 表达式的写法已经是一种很精简且易读的改进。

不过它还有进一步精简的余地：使用『**方法引用**』：

```java
list.forEach(System.out::println);
```

上述代码中间的两个冒号 `::` ，就是 Java 语言中方法引用的特有标志，出现它，就说明使用到了方法引用。

> 上述代码中『省』了一个变量，如果把省掉的变量『补』回来，那么上述代码实际上是下面这个样子：
> 
> ```java
> Consumer<Integer> consumer = System.out::print;
> list.forEach(consumer);
> ```
> 
> **forEach** 方法的参数是 **Consumer\<T>** 接口的实现类的对象。

从编译器的角度来理解，等号右侧的语句是一种方法引用，那么编译器会认为该语句引用的是 **Consumer\<T>** 接口的 **accept(T t)** 抽象方法。

方法引用解决了代码功能复用的问题，使得表达式更为紧凑，可读性更强，借助已有方法来达到传统方式下需多行代码才能达到的目的。

### 8.2 方法引用的语法

方法引用的语法很简单。

使用一对冒号 `::` 来完成，分为左右两个部分，左侧为类名或对象名，右侧为方法名或 new 关键字。有以下 4 种主要情况：

1. `对象::实例方法`

2. `类::静态方法`

3. `类::实例方法`

4. `类::new`

在前两种情况中，方法引用等同于提供方法参数的 lambda 表达式。例如，

- `System.out::println` 等同于 

  `System.out.println(x)`

- `Math::pow` 等同于 

  `(x, y) -> Math.pow(x, y)`


在第三种情况中，第一个参数会成为执行方法的对象。例如：

- `String::compareToIgnoreCase` 等同于 

  `(x, y) -> x.compareToIgnoreCase(y)`

第四种情况<small>（构造器引用）</small>和方法引用类似，不同的是在构造器引用中方法名是 **new** 。

你还可以捕获方法引用中的 this 参数。例如

- `this::equals` 就等同于

  `x -> this.equals(x)`

你也可以使用 super 对象：`super::实例方法` 。

