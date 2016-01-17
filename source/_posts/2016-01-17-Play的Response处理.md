---
title: Play的Response处理
date: 2016-01-17 13:22:54
tags: [Play, Scala]
categories: Play
description: Play的Response处理
---

默认情况下，不需要指定`Content-Length`这个响应头，Play框架会根据响应内容自动计算出来，如
```scala
def normal = Action {
    Ok("Hello, Response")
}
```

会自动生成如下的响应头

```html
Content-Length:15
Content-Type:text/plain; charset=utf-8
Date:Sun, 17 Jan 2016 05:26:14 GMT
```

<!-- more -->

可以使用`play.api.libs.iteratee.Enumerator`来生成响应内容，如

```scala
def normal = Action {
    Result(
      header = ResponseHeader(200),
      body = Enumerator("Hello, Enumerator".getBytes())
    )
}
```
这时也会生成`Content-Length`响应头，如下

```html
Content-Length:17
Date:Sun, 17 Jan 2016 05:31:15 GMT
```
在Play框架中，会把所有的相应内容都加载到内存，再计算出响应内容的长度。如果响应内容很多，就会占用很多内容，这时就需要自己来指定`Content-Length`来解决，如：

```scala
def normal = Action {
    import play.api.libs.concurrent.Execution.Implicits._
    val file = new File("d:\\a.pdf")
    Result(
      header = ResponseHeader(200, Map(CONTENT_LENGTH -> file.length().toString)),
      body = Enumerator.fromFile(file)
    )
}
```

如果要处理文件，可以通过`Ok.sendFile`方法，如

```scala
def header = Action {
    Ok.sendFile(content = new File("d:\\a.pdf"))
}
```

这时会生成如下的响应头：

```html
Content-Disposition:attachment; filename="a.pdf"
Content-Length:54503
Content-Type:application/pdf
Date:Sun, 17 Jan 2016 05:40:49 GMT
```

在浏览器访问时，会自动下载这个文件，文件名为`a.pdf`

也可以指定其他的下载文件名，如:

```scala
def header = Action {
    Ok.sendFile(content = new File("d:\\a.pdf"), fileName = _ => "custom.pdf")
}
```
这时生成的响应头如下：

```html
Content-Disposition:attachment; filename="custom.pdf"
Content-Length:54503
Content-Type:application/pdf
Date:Sun, 17 Jan 2016 05:42:32 GMT
```

也可以通过指定`inline`参数，让浏览器直接打开这个文件，如

```scala
def header = Action {
    Ok.sendFile(content = new File("d:\\a.pdf"), fileName = _ => "custom.pdf", inline = true)
}
```

这时生成的响应头如下：

```html
Content-Length:54503
Content-Type:application/pdf
Date:Sun, 17 Jan 2016 05:43:44 GMT
```

浏览器会直接显示支持的文件格式的内容。


在Play中，支持`Chunked`传输形式，如

```scala
def chunked = Action {
    import play.api.libs.concurrent.Execution.Implicits._
    val stream = new FileInputStream("d:\\a.pdf")
    Ok.chunked(Enumerator.fromStream(stream))
}
```

也可以通过`Enumerator`的`apply`方法生成chunked的内容，如

```scala
def chunked = Action {
    Ok.chunked(Enumerator("Hello", "World", "zhulx").andThen(Enumerator.eof))
}
```

在Play中，可以通过chunked机制来实现comet，如

```scala
def comet = Action {
    val events = Enumerator(
      """<script>console.log('hello')</script>""",
      """<script>console.log('World')</script>"""
    )

    Ok.chunked(events).as(HTML)
}
```

这样，在浏览器的控制台就会打印日志。

可以通过`play.api.libs.iteratee.Enumeratee`来转换`Enumerator`对象，如

```scala
import play.api.libs.concurrent.Execution.Implicits._
def toCometMessage = Enumeratee.map[String] {
  data => Html( """<script>console.log('""" + data + """')</script>""")
}

def comet = Action {
  val events = Enumerator("Hello", "world")
  Ok.chunked(events &> toCometMessage)
}
```

也可以使用`play.api.libs.Comet`帮助类

```scala
def comet = Action {
   val events = Enumerator("Hello2", "World2")
   Ok.chunked(events &> Comet("console.log"))
}
```

