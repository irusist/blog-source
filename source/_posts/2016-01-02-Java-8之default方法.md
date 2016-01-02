---
title: Java 8之default方法
date: 2016-01-02 22:26:14
tags:  [Java8, Java]
categories: Java8
description: Java 8之default方法
---

在Java 8中可以在接口定义方法的实现， 称为default方法， 类似用于Scala中的`trait`， 在`Iterable`中有类似以下的定义

```java
default void forEach(Consumer<? super T> action) {
    Objects.requireNonNull(action);
    for (T t : this) {
        action.accept(t);
    }
}
```

`Iterable`中的default方法的主要目的是为Java 8的Lambda表达式提供支持， 如果将这个方法定义成普通接口方法，则会对现有的JDK中的其他使用`Iterable`接口造成影响， 因此提供了default方法的功能。

<!-- more -->

一个接口中可以定义多个default方法。 下面写个简单的例子说明下default方法的使用

存在一个父接口， 定义了一个default方法

```java
public interface Parent {
    public default String doIt() {
        return "Parent";
    }
}
```

有一个类实现该接口，使用默认的default方法

```java
public class ParentImpl implements Parent {

}
```

`ParentImpl`有一个实现类

```java
public class ParentImpl2 extends ParentImpl {
    @Override
    public String doIt() {
        return "ParentImpl2";
    }
}
```

`ParentImpl2`有一个实现类

```java
public class ChildImpl2 extends ParentImpl2 implements Child {
    
}
```

有一个接口继承`Parent`接口

```java
public interface Child extends Parent {
    /**
     * 重写父接口的default方法
     *
     * @return String
     */
    @Override
    default String doIt() {
        return "Child";
    }
}
```

`Child`有一个实现类，使用`Child`的default方法

```java
public class ChildImpl implements Child {
    
}
```

以下为测试代码

```java
public class DefaultMethodTest {

    public static void main(String[] args) {

        Parent parent = new ParentImpl();
        // default方法的实现类可以直接调用default方法
        // 输出 Parent
        System.out.println(parent.doIt());

        Child child = new ChildImpl();
        // Child接口重写了Parent接口的default方法
        // 输出 Child
        System.out.println(child.doIt());

        // ParentImpl2类重写了Parent接口的default方法
        // 输出 ParentImpl2
        Parent parent1 = new ParentImpl2();
        System.out.println(parent1.doIt());

        // ChildImpl2父类与接口都有default方法， 使用类中定义的default方法
        // 输出 ParentImpl2
        Child child1 = new ChildImpl2();
        System.out.println(child1.doIt());
    }
}
```

通过以上的例子中可以得出以下几点：

  * 实现类可以直接使用父接口中定义的default方法
  
  * 接口可以重写父接口中定义的default方法
  
  * 实现类可以重写父接口中定义的default方法

  * 当父类与父接口都存在default方法时， 使用父类中重写的default方法

如果一个类实现了2个接口，这2个接口有相同的default方法签名时， 此时会编译失败， 必须在子类中重写这个default方法。










