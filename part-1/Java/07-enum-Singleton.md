# <font color="orange">利用 Java 枚举实现单例模式</font>

合格的单例模式的实现，至少要保证以下三点：

1. 实现单例功能；
2. 延迟加载；
3. 并发时不出错。 


## 1. 饿汉式

```java
public class Singleton {

    private static Singleton instance = new Singleton();

    private Singleton() {
    }

    public static Singleton getInstance() {
        return instance;
    }

}
```

- 饿汉模式的缺点：可能在还不需要此实例的时候就已经把实例创建出来了，没起到 lazy loading 效果。
- 饿汉模式的优点：实现简单，而且安全可靠。

## 2. 懒汉式

```java
public class Singleton {
    
    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

- 懒汉模式的优点：弥补了饿汉模式的缺点，起到了 lazy loading 的效果。
- 懒汉模式的缺点：多线程并发时有线程安全问题，有可能创建多个实例。

## 3. 双校验懒汉式

```java
public class SingletonSafe {

    private static volatile SingletonSafe singleton;

    private SingletonSafe() {
    }

    public static SingletonSafe getSingleton() {
        if (singleton == null) {
            synchronized (SingletonSafe.class) {
                if (singleton == null) {
                    singleton = new SingletonSafe();
                }
            }
        }
        return singleton;
    }
}
```

- 双校验懒汉模式的优点：弥补了懒汉模式的缺点，防止了并发问题。
- 双校验懒汉模式的缺点：因为涉及到锁，因此性能有损耗；代码变得更复杂。

除了上述三种实现单例的方式外，还有其它的方式，但是多少都会有些不尽如人意之处。

《Effective Java》：

    单元素的枚举类型已经成为实现 Singleton 的最佳方法。


## 4. Java 枚举

### 4.1 基本用法

首先，枚举类似类，一个枚举可以拥有成员变量，成员方法，构造方法。先来看枚举最基本的用法：

```java
enum Type {
    A, B, C, D;
}
```

创建 enum 时，编译器会自动为我们生成一个继承自 *`java.lang.Enum`* 的类，我们上面的 enum 可以简单看作：

```java
class Type extends Enum {
    public static final Type A;
    public static final Type B;
    ...
}
```

简单来说，上面的 Type 枚举相当于：

1. 有一个叫 Type 的类；
2. Type 类有且仅有 4 个对象；
3. 这 4 个对象分别叫<small>（也只能叫）</small>A，B，C，D 。

当然，这个构建 Type 的 A，B，C，D 四个实例的过程不是我们做的，一个 enum 的构造方法限制是 *`private`* 的，也就是不允许我们调用。

### 4.2 枚举的类方法和实例方法（了解、自学）

上面说到，我们可以把 Type 看作一个类，而把 A，B，C，D。看作 Type 的一个实例。同样，在 enum 中，我们可以定义类和实例的变量以及方法。

```java
enum Type {
    A, B, C, D;
 
    static int value;              // 静态属性 / 类属性
    public static int getValue() { // 静态方法 / 类方法
        return value;
    }
 
    String type;                   // 非静态属性 / 实例属性 
    public String getType() {      // 非静态方法 / 示例方法
        return type;
    }
}
```

1. 类属性被类方法使用；
2. 实例属性被实例方法使用；
3. 类方法通过：`Type.getValue()` 调用；
4. 实例方法通过：`Type.A.getType()` 调用。

如果 Type 的 A 实例像和 B，C，D 不一样，那么可以写成：

```java
// 静态属性、静态方法略。与此问题无关。
enum Type {
    A {
        public void sayHello() {
            System.out.println("I'm A");
        }
    }, B, C, D;

    public String sayHello() {
        System.out.println("I'm one of Type");
    }
}
```

A 实例后面的 `{…}` 就是属于 A 的实例方法，可以通过覆盖原本的方法，实现属于自己的定制。B，C，D 仍然『遵守』共同的 *`sayHello()`* 规则。

更极端一点，A，B，C，D 每个人都可以有自己的独特的 `sayHello()` 。

```java
enum Type {
    A { @Override public String sayHello() { ... } },
    B { @Override public String sayHello() { ... } },
    C { @Override public String sayHello() { ... } },
    D { @Override public String sayHello() { ... } };
 
    public abstract String sayHello();
}
```

## 5. 枚举单例

!FILENAME 内部类形式
```java
class Service {
    ...

    private Service() {
    }

    public static Service get() {
        return Singleton.INSTANCE.get();
    }

    private enum Singleton {
        INSTANCE;
        private Service instance;
        Singleton () {
            instance = new Service();
        }
        public Service get() {
            return instance;
        }
    }
}

```

如果不喜欢（或不习惯）内部类形式，那就把 `enum Singleton` 单独拿出来写成工具类形式。

