---
title: Java 8之Optional
date: 2016-01-02 23:25:07
tags: [Java8, Java]
categories: Java
description: Java 8之Optional
---

由于`NullPointerException`是Java开发中发生次数最多的一个异常， 所以在Java 8中提供了一个`Optional`类， 用来避免该异常的发生， 这个类类似于Scala中的`Option`。

`Optional`类的所有构造方法都是`private`的，因此可以使用它的静态方法来初始化，如：

```java
// 用来生成一个空值的Optional
Optional empty = Optional.empty();

// 用来生成一个非空值的Optional
Optional<String> notNull = Optional.of("optional");

// 可以根据传入的值生成空或非空的Optional
Optional<String> nullValue = Optional.ofNullable(null);
```

<!-- more -->

可以使用`isPresent` 方法来判Optional内的值是否存在，如：

```java
// 返回 false
System.out.println(Optional.empty().isPresent());

// 返回 true
System.out.println(Optional.of(1).isPresent());
```

使用`get`方法获取Optional内的值，如果是`empty`，则抛出`NoSuchElementException`异常。

使用`ifPresent` 方法可以先判断这个值是否存在，如果存在，则 执行一个操作。

`filter`, `map`, `flatMap`等方法与`Stream`的操作类似， 这里不再说明。

`orElse` 方法接收一个与Optional类型一致的参数，表示如果这个`Optional`是empty，则返回这个传入的值。

`orElseGet`方法接受一个`Supplier`接口，表示当`Optional`内的值不存在时，通过`Supplier`获取到这个值。

`orElseThrow` 方法接受一个 `Supplier`接口，表示当`Optional`内的值不存在时，通过`Supplier` 获取到一个异常值，并抛出这个异常。



