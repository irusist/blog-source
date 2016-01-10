---
title: Play框架学习之Content negotiation
date: 2016-01-10 13:42:44
tags: [Play2, Play, Scala]
categories: [Play]
description: Play框架学习之Content negotiation
---

内容协商可以通过指定请求头以`Accept*`形式，对同一个请求地址，可以响应不同的内容。这在对一个服务支持xml或json格式的情况下很有用。

## Language

在Play中可以通过`play.api.mvc.RequestHeader#acceptLanguages`方法获取支持的语言类型，它返回一个列表，通过解析请求的`Accept-Language`请求头，根据他们的quality value进行排序，如`zh-CN,zh;q=0.8`, 它的quality value为0.8。在Play中有一个隐式转换，转成`Lang`类型，如：

```scala
implicit def request2lang(implicit request: RequestHeader): Lang = {
    play.api.Play.maybeApplication.map(app => play.api.i18n.Messages.messagesApiCache(app).preferred(request).lang)
      .getOrElse(request.acceptLanguages.headOption.getOrElse(play.api.i18n.Lang.defaultLang))
}
```

它会找出排序过后列表中第一个支持的语言。

<!-- more -->

## Content

类似地，`play.api.mvc.RequestHeader#acceptedTypes`方法会返回请求支持的MIME类型，它是通过解析请求头的`Accept`参数得到的，并根据quality因子进行排序。可以通过以下的方法来处理：

```scala
def custom() = Action { implicit request =>
    render {
      case Accepts.Html() => Ok(views.html.index("html accepts"))
      case Accepts.Json() => Ok(Json.toJson("json accepts"))
    }
}
```

它解析`Accept`请求头，如果包含`text/html`，则返回index页面，如果包含`application/json`，则返回JSON格式的内容，如果请求头指定的是`application/xml`, 则会返回`val NOT_ACCEPTABLE = 406`。


## 请求抽取器

可以通过`play.api.mvc.Accepting`这个case class来自定义请求头抽取器，如：

```scala
val AcceptsMp3 = Accepting("audio/mp3")

render {
    case AcceptsMp3() => ???
}
```



