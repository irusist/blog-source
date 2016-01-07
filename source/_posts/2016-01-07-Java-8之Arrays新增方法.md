---
title: Java 8之Arrays新增方法
date: 2016-01-07 22:43:46
tags: [Java8, Java]
categories: Java
description: Java 8之Arrays新增方法
---

在Java 8中，在Arrays工具类中增加了一些对数组相关的并行和流的操作。

## parallelSort 方法

`parallelSort`方法是并行地对数组进行排序，它有很多个重载方法，可以对基本类型， `Comparable`类型对象，或者提供一个`Comparator`接口对一般对象进行排序。

如果数组的大小小于或等于8192(1 << 13)，会直接排序，不会并行。并行排序使用的是ForkJoin框架。

<!-- more -->

## parallelPrefix 方法

`parallelPrefix`方法需要提供一个操作方法，通过遍历数组的元素，将当前索引位置的值与它之前索引的值进行操作，然后将操作后的值覆盖当前索引位置的值。下面用一个简单说明一下

```java
int[] array = new int[]{2, 3, 1, 0, 5};
// [2, 3, 1, 0, 5]
// [2, 5, 1, 0, 5]
// [2, 5, 6, 0, 5]
// [2, 5, 6, 6, 5]
Arrays.parallelPrefix(array, (left, right) ->{
    System.out.println(Arrays.toString(array));
    return left + right;
});

// 输出 [2, 5, 6, 6, 11]
System.out.println(Arrays.toString(array));
```

上面将每一步操作都输出结果，首先，从索引为1开始循环，将索引为0的值2, 与索引为1的值3进行相加，得到5覆盖索引为1的值，依此直到达到数组的最后一个元素。


## setAll 方法

`setAll`方法提供一个 `int -> T`的函数接口，通过该接口对数组的索引进行操作，然后将指定数组当前索引位置的值赋值为操作后的值，以下是例子

```java
int[] array = new int[10];
Arrays.setAll(array, i -> i * 10);
// 输出 [0, 10, 20, 30, 40, 50, 60, 70, 80, 90]
System.out.println(Arrays.toString(array));
```
`parallelSetAll`方法是 `setAll`的并行操作版本。


## spliterator 方法

`spliterator`方法返回一个`Spliterator`, 关于`Spliterator`的内容，后续再详细说明下。


## stream 方法

`stream` 方法是将数组转换为`Stream`的操作，可以是任意类型的数组， 并且可以指定数组的范围。




