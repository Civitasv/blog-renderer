---
title: "接口、lambda表达式与内部类"
summary: "接口用于描述类应该做什么，但不指定具体应该怎么做，lambda 表达式与**函数式接口**共同使用时能够大大简化程序的代码量，内部类定义在另一类的内部，在设计具有相互协作关系的类集合中很有用处"
date: "2021-01-21"
author: "Civitasv"
categories:
  - CoreJAVAVolume I
tags:
  - Java核心技术卷I
---

> 接口用于描述类应该做什么，但不指定具体应该怎么做，lambda 表达式与**函数式接口**共同使用时能够大大简化程序的代码量，内部类定义在另一类的内部，在设计具有相互协作关系的类集合中很有用处。

## 6.1 接口

在 Java 程序设计语言中，接口不是类，而是对希望符合这个接口的类的一组**需求**。

- 接口中的方法自动是`public`方法，因此声明方法时不必提供方法修饰符。

- 接口绝不会拥有实例，但可以声明接口的变量。

- 接口中可以包含常量（自动是`public static final`）。

- 接口可以提供多重继承的大多数好处，同时还能避免多重继承的复杂性和低效性。

- 接口中可以定义静态方法，但通常做法都是将静态方法放在伴随类中，如`Collection/Colletions或Path/Paths`，认为最好不要在接口中定义具体的方法，这样会破坏接口的抽象性。

### 6.1.1 默认方法

目前，接口中定义的方法可以有默认实现。

```java
public interface A{
    default int size(){return 0};
}
```

默认方法的一个重要用法是**接口演化（interface evolution）**，假设有接口 A 定义如下：

```java
public interface A{
    int size();
}
```

类 B 实现了接口 A：

```java
public class B implements A{
    public int size(){
        return 0;
    }
}
```

但不久后，接口 A 增加了一个`name`方法：

```java
public interface A{
    int size();
    String name();
}
```

这时，类 B 将不能通过编译，因为它并没有实现`name`方法，想一想，如果软件厂商已经上线了该服务，后面迁移到新的库时，连编译都无法通过的话，将多么打击程序员的积极性。这时有两种方案：

1. 不重新编译类 B，而是使用原来的一个包含这个类的 JAR 文件，这个类就可以正常加载了，但如果试图使用 B 实例调用`name`方法，就会抛出异常`AbstractMethodError`；

2. 将新添加的`name`方法实现为一个`default`方法，这不要求实现 A 的类必须实现`name`方法，如果使用 B 实例调用`name`方法，将调用`A.name`方法。

虽然使用默认方法有一定的好处，但为了接口的抽象性，我认为还是不宜在接口中定义或实现具体的方法。而且，使用默认方法还可能遭遇二义性冲突。

### 6.1.2 默认方法冲突

观察下面的代码：

```java
interface A {
    default String size() {
        return "A";
    }
}

interface B {
    default String size() {
        return "B";
    }
}

class C implements A, B {
    // 必须实现该方法
    @Override
    public String size() {
        return null;
    }
}
```

上面的代码无法编译，因为类 C 实现了接口 A 和 B，而这两个接口都具有默认的`size`函数，如果 C 中不实现自己的`size`，编译器不知道它将执行谁的`size`方法。也就造成了二义性冲突。

事实上，只要接口 A 或 B 中有一个定义了默认方法且二者方法重名，同时实现这两个接口的类都必须实现该重名方法以解决二义性冲突。

而且，如果超类和接口命名发生冲突，则会采取“类优先”原则，接口中的默认方法将被忽略。这可以确保与 Java 7 的兼容性。

### 6.1.3 接口与回调

Java 常采用接口实现回调的设计模式，具体实现及讲解可查看：
[回调函数](../vary/回调函数.md)

## 6.2 lambda 表达式

lambda 表达式采用一种十分简洁的语法定义代码块。

引入 lambda 表达式的目的是简洁的**传递代码块**。

## 6.2.1 函数式接口

对于只有一个抽象方法的接口，需要这种接口的对象时，就可以提供一个 lambda 表达式。这种接口称为函数式接口（functional interface）。

**最好将 lambda 表达式看作是一个函数，而不是一个对象，而它能够传递到函数式接口。**

但可以使用函数式接口变量保存函数表达式。

除此之外，还可以定义**方法引用**和**构造器引用**，简化 lambda 表达式。

常用函数式接口：

|    函数式接口    | 参数类型 | 返回类型 | 抽象方法名 |             描述             | 其他方法 |
| :--------------: | :------: | :------: | :--------: | :--------------------------: | :------: |
|     Runnable     |    无    |   void   |    run     | 作为无参数或返回值的动作执行 |
|   Supplier<T>    |    无    |    T     |    get     |     提供一个 T 类型的值      |
|   Consumer<T>    |    T     |   void   |   accept   |     处理一个 T 类型的值      | andThen  |
| BiConsumer<T, U> |   T, U   |   void   |   accept   |     处理 T 和 U 类型的值     | andThen  |
|       ...        |   ...    |   ...    |    ...     |             ...              |   ...    |

`@FunctionalInterface`: 函数式接口注解

## 6.3 内部类

内部类是定义在另一个类中的类，使用内部类主要有两个原因：

1. 内部类可以对同一个包中的其他类隐藏。

2. 内部类方法可以访问定义这个类的作用域中的数据，包括原本私有的数据。

实际上，在 lambda 表达式未出现之前，实现回调也是内部类的重要应用之一。