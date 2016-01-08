---
title: 'Play框架学习之Action, Controller, Result'
date: 2016-01-08 22:34:46
tags: [Play2, Play, Scala]
categories: Play
description: Play框架学习之Action, Controller, Result
---

折腾了2个晚上，终于把Play2框架编译成功搭建起来，跟着官网的文档体验一下。

## Action

Action是用于请求的地方，最简单的Action就是返回一些文本或页面的Result，如

```scala
Action {
  Ok("Hello world")
}
```

其中OK是一个`Status`对象，它接收一个值为200的`Int`类型代表响应头的状态码，同时`Status`类有一个`apply`方法，它有一个参数，代表响应的内容。


Action也接收一个`Request => Result`类型的参数， 可以对请求信息进行处理，如

```scala
def add = Action { request =>
  Ok("request is: " + request)
}
```

<!-- more -->

request参数可以定义成隐式参数的形式

```scala
def add = Action { implicit request =>
   Ok("request is: " + request)
}
```

Action还可以有一个`BodyParser`参数，用于解析请求body的内容，如

```scala
def add = Action(parse.json) { implicit request =>
  Ok("request is: " + request)

```

表示请求的body可以通过json来解析，因此body的格式必须为`text/json`或者`application/json`的。


## Controller

Controller是一个用来定义`Action`的一个单例对象。最简单的`Controller`的方法是没有参数的

```scala
def add = Action { request =>
    Ok("request is: " + request)
}
```

方法也可以有参数，这些参数可以被`Action`中的代码块使用，如

```scala
def add(name : String) = Action { request =>
    Ok("parameter is: " + name)
}
```

## Result

`Result`是`Action`的返回类型，它可以设置响应头和响应体，如：

```scala
def add = Action {
   Result(
     header = ResponseHeader(200, Map(CONTENT_TYPE -> "text/plain")),
     body = Enumerator("response".getBytes)
   )
}
```

也可以设置各种响应码，如404，500等，一些常用的如下

```scala
/** Generates a ‘403 FORBIDDEN’ result. */
val Forbidden = new Status(FORBIDDEN)

/** Generates a ‘404 NOT_FOUND’ result. */
val NotFound = new Status(NOT_FOUND)

/** Generates a ‘500 INTERNAL_SERVER_ERROR’ result. */
val InternalServerError = new Status(INTERNAL_SERVER_ERROR)
```

`Result`也可以返回一个重定向的结果，如

```scala
def redirect = Action { request =>
   Redirect("/add", MOVED_PERMANENTLY)
}
```

重定向时如果不指定第2个参数，则默认响应头为`val SEE_OTHER = 303`


`Action`也支持一个默认的`TODO`页面，如

```scala
def index(name:String) = TODO
```