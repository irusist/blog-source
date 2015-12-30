---
title: Java 8之Stream API
date: 2015-12-30 22:07:36
tags: [Java8]
categories: Java
description: Java 8之Stream API
---

## Stream API简介

Java 8中定义了一个Stream的接口，与容器类类似，主要存放着一些元素，但与集合不同，它主要是对它内部的数据进行过滤，聚合等一些操作。它内部必须要有集合，数组等数据结构。


## Stream对象的生成


Stream对象可以通过集合，文件, String, Stream, StreamBuilder等来构建

### 通过集合来构建

```java
Stream<Integer> stream = Arrays.asList(1, 2, 3, 4).stream()
```

### 通过文件来构建

```java
// 获取文件内容
Stream<String> lines = Files.lines(Paths.get("/etc/hosts"));
// 获取目录下的文件列表
Stream<Path> paths = Files.list(Paths.get("/etc/init.d"));
```

<!-- more -->

### 通过String来构建
```java
IntStream stream = "Stream".chars();
```

### 通过Stream来构建

```java
Stream<String> stream = Stream.of("me", "you", "i");
```

### 通过StreamBuilder来创建

```java
Stream.Builder<String> builder = Stream.builder();
Stream<String> stream = builder.add("a").add("b").add("c").build();	
```

## filter方法

filter方法主要用来过滤Stream中的元素，示例代码如下
```java
List<String> string = Arrays.asList("nanjing", "hanmei", "shangrao", "shanghai", "nanchang");
// 输出
// shangrao
// shanghai
// nanchang
string.stream().filter(x -> x.length() > 7).forEach(System.out::println);
```
以上的例子代码中将过滤出Stream中长度大于7的元素


## map方法

map方法主要用来将Stream中的元素转换成其他的值，示例代码如下

```java
// 输出
// NANJING
// BEIJING
// SHANGHAI
Stream.of("nanjing", "beijing", "shanghai").map(String::toUpperCase).forEach(System.out::println);
```

以上的例子代码中将Stream中的每个元素转换成大写的


## mapToInt方法

mapToInt方法是将Stream中的元素转换成int类型的，示例代码如下

```java
// 输出
// 10
// 20
// 30
Stream.of(1, 2, 3).mapToInt(data -> data * 10).forEach(System.out::println);
```

以上的例子代码中将Stream的每个元素都乘以10，类似的方法还有 `mapToLong`, `mapToDouble`等


## flatMap方法

flatMap是将Stream中的每个元素都转换成Stream类型，然后将每个Stream的元素提出取来，聚合成一个最外围的Stream。示例代码如下：

```java
// 输出
// 1
// 2
// 3
// 4
Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4))
        .flatMap(List::stream)
        .forEach(System.out::println);
```

以上的例子代码将Stream两个List组合成一个Stream


## distinct方法

distinct方法是将Stream中的相等元素去除，通过equals方法来判断元素是否相等，示例代码如下：

```java
// 输出
// 1
// 2
// 3
// 4
Stream.of(1, 2, 3, 4, 2, 3, 1).distinct().forEach(System.out::println);
```

## sorted方法

sorted方法是将Stream中的元素进行排序，它内部的元素必须实现Comparable， 示例代码如下：

```java
// 输出
// beijing
// nanjing
// nantong
// shangrao
Stream.of("nanjing", "beijing", "nantong", "shangrao").sorted().forEach(System.out::println);
```

sorted方法还可以接收一个Comparator的参数

## forEach方法

forEach方法主要是对Stream中每个元素进行操作，示例如下

```java
Stream.of(1, 2, 3).forEach(System.out::println);
```

## reduce方法

reduce方法是一个聚合方法，是将Stream中的元素进行某种操作，得到一个元素，示例代码如下

```java
// 输出 6
System.out.println(Stream.of(1, 2, 3).reduce(0, (acc, element) -> acc + element));
```

以上的代码是将Stream的每个元素相加，并加上初始值`0`。reduce还有2个重载的方法，后续再详细学习下。


## collect方法

collect方法是将Stream的通过一个收集器，转换成一个具体的值，是一个聚合方法，示例代码如下：

```java
// 输出 [abc, def, efg]
System.out.println(Stream.of("abc", "def", "efg").collect(Collectors.toList()));
```

上面的代码将Stream转换成一个List。


## min， max方法

min，max方法是获取Stream中最小或最大的元素的值，需要传递一个Comparator比较器参数，是聚合方法，示例代码如下

```java
// 输出 haimen
List<String> cities = Arrays.asList("nanjing", "beijing", "nantong", "haimen", "shangrao");
System.out.println(cities.stream().min(Comparator.comparing(String::length)).get());
```

## count方法

count方法是获取Stream中元素的数量，是聚合方法，示例代码如下

```java
// 输出 3
System.out.println(Stream.of(1, 2, 3).count());
```


## anyMatch方法

anyMatch方法用来判断Stream中是否存在指定条件的元素，示例代码如下

```java
// 输出 true
System.out.println(Stream.of(1, 2, 3).anyMatch(data -> data > 2));
// 输出 false
System.out.println(Stream.of(1, 2, 3).anyMatch(data -> data > 3));
```

## allMatch

allMatch方法用来判断Stream中的元素是否都满足指定的条件，示例代码如下

```java
// 输出 false
System.out.println(Stream.of(1, 2, 3).allMatch(data -> data > 2));
// 输出 true
System.out.println(Stream.of(1, 2, 3).allMatch(data -> data > 0));
```

## noneMatch

noneMatch方法用来判断Stream中是否所有元素都不满足指定条件，示例代码如下

```java
// 输出 false
System.out.println(Stream.of(1, 2, 3).noneMatch(data -> data > 2));
// 输出 true
System.out.println(Stream.of(1, 2, 3).noneMatch(data -> data > 3));
```

## findFirst方法

findFirst方法返回Stream中的第一个元素，返回的类型为 `Optional`, 示例代码如下

```java
// 输出 1
System.out.println(Stream.of(1, 2, 3).findFirst().get());
```






