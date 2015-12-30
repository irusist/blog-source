---
title: Java 8之Lambda表达式
date: 2015-12-30 13:22:50
tags: [Lambda, Java8]
categories: Java
description: Java 8之Lambda表达式
---

## Lambda简介

Lambda表达式是Java 8引入的，具体的详细介绍可以查看[Lambda表达式](https://en.wikipedia.org/wiki/Anonymous_function)

举一个简单的例子来说明Java8中的Lambda表达式：在一个数字列表中选出大于10的数字列表。

存在一个过滤接口
```java
public interface Filter<T> {
    boolean f(T t);
}
```

<!-- more -->

有以下过滤器
```java
public class Filters {
    public static <T> List<T> filter(List<T> data, Filter<T> filter) {
        List<T> result = new ArrayList<>();
        for (T t : data) {
            if (filter.f(t)) {
                result.add(t);
            }
        }

        return result;
    }
}
```
在Java 7可以使用如下代码过滤数字

```java
public class Java7 {
    public static void main(String[] args) {
        Filter<Integer> filter = new Filter<Integer>() {
            @Override
            public boolean f(Integer data) {
                return data > 10;
            }
        };
        List<Integer> ages = Arrays.asList(9, 10, 23, 7, 12, 1, 13, 8);
        List<Integer> result = Filters.filter(ages, filter);

        // 输出: [23, 12, 13]
        System.out.println(result);
    }
}}
```

在Java 8中可以使用如下的代码

```java
public class Java8 {
    public static void main(String[] args) {
        List<Integer> ages = Arrays.asList(9, 10, 23, 7, 12, 1, 13, 8);
        List<Integer> result = Filters.filter(ages, data -> data > 10);
        // 输出: [23, 12, 13]
        System.out.println(result);
    }
}
```

其中，以上代码的

```java 
data -> data > 10```

就是一个Lambda表达式


## 类型推断

Java 8中的Lambda可以通过类型推断来省略一些类型信息，如

```java
BinaryOperator<Long> add = (x, y) -> x + y;
```

其中 BinaryOperator有以下方法
```java
T apply(T t, T u);
```

由BinaryOperator的类型Long，可以推断出以上的Lambda表达式的x，y都为Long类型。


## 函数接口

Java 8中的Lambda表达式需要依赖函数接口，它是一个接口，用于定义Lambda表达式需要的方法，如存在以下接口
```java
public interface Filter<T> {
    boolean f(T t);
}
```
则可以定义如下的Lambda表达式

```java
Filter<String> filter = data -> data.length() > 10;
```

在Java 8 存在很多的函数接口，例如

  * Predicate
  
  * Consumer

  * Function

  * Supplier

  * UnaryOperator

  * BinaryOperator


## Scala中的函数

在Scala中，函数是一等公民，不需要额外地依赖函数接口，如
```scala
val ages = List(9, 10, 23, 7, 12, 1, 13, 8)
val filtered = ages.filter(data => data > 10)
// 输出  List(23, 12, 13)
println(filtered)
```