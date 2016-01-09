---
title: Play框架学习之Action组合
date: 2016-01-09 23:21:03
tags: [Play2, Play, Scala]
categories: Play
description: Play框架学习之Action组合
---

在Play2中，可以组合多个`Action`

## 自定义Action

可以通过继承`ActionBuilder`来定义一个Action，Play2中自带的Action就是一个默认实现，如下是自定义一个`Action`，用于对每个请求打印信息。

```scala
object LoggingAction extends ActionBuilder[Request] {
   override def invokeBlock[A](request: Request[A], block: (Request[A]) => Future[Result]) = {
     Logger.info("Calling action")
     block(request)
   }
}
```
可以通过如下的方法来使用这个自定义的`Action`:

```scala
def custom() = LoggingAction {
    Ok("custom action")
}
```

这样，在每次请求时就会打印“Calling action”信息。

<!-- more -->

### 组合Action

在开发时，经常遇到会对每一个`Action`进行一些通用的处理，如日志， 权限等，在Play2中，可以将它们定义成不同的`Action`，然后将它们组合起来。

可以通过继承`Action`来创建新的`Action`，它是一个接收其他`Action`的`case class`。

```scala
case class Logging[A](action: Action[A]) extends Action[A] {
    lazy val parser: BodyParser[A] = action.parser

    override def apply(request: Request[A]): Future[Result] = {
      Logger.info("Calling action")
      action(request)
    }
}
```

也可以通过`Action.async`方法来定义一个接收其他`Action`的参数，如

```scala
def logging[A](action: Action[A]) = Action.async(action.parser) { request =>
    Logger.info("Calling action")
    action(request)
}
```

也可以通过`ActionBuilder`的`composeAction`方法来将它本身的`Action`组装到其他`Action`中，如

```scala
object LoggingAction extends ActionBuilder[Request] {
    override def invokeBlock[A](request: Request[A], block: (Request[A]) => Future[Result]) = {
      block(request)
    }

    override protected def composeAction[A](action: Action[A]): Action[A] = Logging[A](action)
}
```
可以通过以下方法来使用`logging`方法：

```scala
def custom() = logging {
    Action {
      Ok("custom action")
    }
}
```

### 更复杂的Action

可以通过组合`Action`来修改request对象，如：

```scala
def xForwardedFor[A](action: Action[A]) = Action.async(action.parser) { request =>
  val newRequest = request.headers.get(X_FORWARDED_FOR).map { xff =>
    new WrappedRequest[A](request) {
      override def remoteAddress = xff
    }
  } getOrElse request
  action(newRequest)
}
```

也可以修改返回的结果，如：

```scala
def addUaHeader[A](action: Action[A]) = Action.async(action.parser) { request =>
  action(request).map(_.withHeaders("X-UA-Compatible" -> "Chrome=1"))
}
```

## 不同的请求类型

`ActionFunction`是一个作用域request的方法，它可以用于权限，认证，查询数据库对象等。有以下内置的`ActionFunction`：

  * `ActionTransformer`：可以用来修改request，如增加一些额外的信息。

  * `ActionFilter`：可以用来拦截某些不符合条件的请求，不用来改变请求的值。

  * `ActionRefiner`：是上面2个的结合体

  * `ActionBuilder`：以`Request`作为输入参数，生成Action。


### 认证

以下是一个认证的例子，通过组合以上的`ActionFunction`。

首先，自定义一个请求类，并且定义Action

```scala
class UserRequest[A](val username: Option[String], request: Request[A]) extends WrappedRequest[A](request)

object UserAction extends
    ActionBuilder[UserRequest] with ActionTransformer[Request, UserRequest] {
  def transform[A](request: Request[A]) = Future.successful {
    new UserRequest(request.session.get("username"), request)
  }
}
```

添加一些额外的信息，生成另一个请求对象

```scala
class ItemRequest[A](val item: Item, request: UserRequest[A]) extends WrappedRequest[A](request) {
  def username = request.username
}
```

然后对请求信息进行转换

```scala
def ItemAction(itemId: String) = new ActionRefiner[UserRequest, ItemRequest] {
  def refine[A](input: UserRequest[A]) = Future.successful {
    ItemDao.findById(itemId)
      .map(new ItemRequest(_, input))
      .toRight(NotFound)
  }
}
```

验证请求内容

```scala
object PermissionCheckAction extends ActionFilter[ItemRequest] {
  def filter[A](input: ItemRequest[A]) = Future.successful {
    if (!input.item.accessibleByUser(input.username))
      Some(Forbidden)
    else
      None
  }
}
```

最后将所有的转换组合在一起：

```scala
def tagItem(itemId: String, tag: String) =
  (UserAction andThen ItemAction(itemId) andThen PermissionCheckAction) { request =>
    request.item.addTag(tag)
    Ok("User " + request.username + " tagged " + request.item.id)
}
```
