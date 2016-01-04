---
title: Java 8之Collector
date: 2016-01-04 21:17:25
tags: [Java8, Java]
categories: Java
description: Java 8之Collector
---

Java 8中有一个`Collector`接口，主要是为Stream中的元素转换成其他值提供的一个接口，在`Collectors`工具类中有很多`Collector`接口的实现方式。

Collector的定义为`Collector<T, A, R>`, 其中泛型参数`T`表示Stream中元素的类型， `A`表示定义一个初始容器的类型，`R`表示最终转换的类型。

Collector接口主要定义了如下几个方法：

  * supplier：这个方法主要是生成一个初始容器，用于存放转换的数据。它返回一个`Supplier<A>`类型，用Lambda表示为`() -> A`

  * accumulator: 这个方法是将初始容器与Stream中的每个元素进行计算，它返回一个`BiConsumer<A, T>`类型，用Lambda表示为`(A, T) -> void`

  * combiner: 这个方法用于在并发Stream中，将多个容器组合成一个容器，它返回一个`BinaryOperator<A>`类型，用Lambda表示为`(A, A) -> A`

  * finisher：这个方法用于将初始容器转换成最终的值，它返回一个`Function<A, R>`类型，用Lambda表示为`A -> R`

  * characteristics: 这个方法返回该Collector具有的哪些特征，返回的是一个`Set`, 分别是`CONCURRENT`(并发), `UNORDERED`(未排序)，`IDENTITY_FINISH`(`finisher`方法直接返回初始容器)等特征的组合。

<!-- more -->


在`Collectors`中有一个`joining`方法，它是将`Stream`中的字符序列类型的元素串接在一起，下面是一个例子：


```java
// 输出 abcd
System.out.println(Stream.of("a", "b", "c", "d").collect(Collectors.joining()));
```

下面通过自己实现joining方法来理解如何实现`Collector`接口。


定义自己实现的`JoinCollector`类：

```java
public class JoinCollector implements Collector<CharSequence, StringBuilder, String> {
    @Override
    public Supplier<StringBuilder> supplier() {
        return StringBuilder::new;
    }

    @Override
    public BiConsumer<StringBuilder, CharSequence> accumulator() {
        return StringBuilder::append;
    }

    @Override
    public BinaryOperator<StringBuilder> combiner() {
        return StringBuilder::append;
    }

    @Override
    public Function<StringBuilder, String> finisher() {
        return StringBuilder::toString;
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.emptySet();
    }
}
```

其中，`supplier`方法返回一个`StringBuilder`对象，用于表示初始容器。

`accumulator`方法返回的是`StringBuilder`的`append`方法的方法引用， 用于将`Stream`中的元素加入到初始容器`StringBuilder`中。

`combiner`方法用于将多个`StringBuilder`容器通过`append`方法组合成一个`StringBuilder`对象。

`finisher`方法是将`StringBuilder`对象通过`toString`方法最终转换成`String`对象。


使用自定义的`Collector`的例子如下：

```java
// 输出 abcd
System.out.println(Stream.of("a", "b", "c", "d").collect(new JoinCollector()));
```

得到的结果与`Collectors.joining`方法是一样的。





