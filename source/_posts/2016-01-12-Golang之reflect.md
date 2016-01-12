---
title: Golang之reflect
date: 2016-01-12 23:09:37
tags: [Go]
categories: Go
description: Golang之reflect
---

在Go语言中，使用`reflect`包可以对类型信息，类型的值进行获取和设置，分别用`Type`和`Value`表示。

可以定义一个类型，并且为类型声明一个方法

```go
type Person struct {
     Name string
     Id   int32
     Age  int32
}

func (person Person) Eat(food string) {
     fmt.Println("eat...", food)
}
```
<!-- more -->

`TypeOf`用于获取`Type`信息，如

```go
// Type
t := reflect.TypeOf(person)
// output: struct Person 3 1
fmt.Println(t.Kind(), t.Name(), t.NumField(), t.NumMethod())
```

`Kind()`方法用于表示是哪种类型的，如`struct`, `int`, `string`等。`Name`方法表示类名，`NumField`表示字段的个数，`NumMethod`表示方法的个数。

`ValueOf`用于获取`Value`信息，如

```go
// Value
v1 := reflect.ValueOf(person)
// output: struct {zhulx 1 28} 3 1 false
fmt.Println(v1.Kind(), v1.Interface(), v1.NumField(), v1.NumMethod(), v1.CanSet())
```

`Interface`方法用于获取具体的值，`CanSet`表示是否可以修改原始类型的值，如果返回false，则不能调用`CanXXX()`类型的方法用来设置字段的值，也不能调用`Call`方法用来调用方法。

```go
// 指针的Value
v2 := reflect.ValueOf(&person)
// output: ptr &{zhulx 1 28} 3 1 false true
fmt.Println(v2.Kind(), v2.Interface(), v2.Elem().NumField(), v2.NumMethod(), v2.CanSet(), v2.Elem().CanSet())
```

可以通过传入指针类型对象来对原始对象的字段进行修改，或者调用方法，如

```go
// Field Set
v3 := reflect.ValueOf(&person).Elem()
v3.FieldByName("Age").SetInt(30)
// output: 30
fmt.Println(person.Age)
```

也可以调用`Call`方法来调用具体某个方法，如

```go
// Method Call
v4 := reflect.ValueOf(&person).Elem()
// output: eat... apple
v4.MethodByName("Eat").Call([]reflect.Value{reflect.ValueOf("apple")})
```