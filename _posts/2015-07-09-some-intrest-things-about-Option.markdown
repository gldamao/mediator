---
layout: post
title: "scala中处理null的终极解决方案----Option类型"
date: 2015-07-09 07:22:02 +0800
comments: true
categories: scala 读书笔记
---

### Scala支持更高级的null
> Scala在标准库里提供了scala.Option类,Option拥有两个子类,分别是None和Some.分别代表容器(Option)中是否包含元素.

- - - 

#### Scala的Option与java的null
Scala在Option的伴生对象里提供了工厂方法,这个方法能把java风格的引用转换为Option类型.

``` scala
var x: Option[String] = Option(null)
x: Option[String] = None
x = Option("Initialized")
x: Option[String] = Some(Initialized)
```
#### Option的高端使用技巧
Option的与众不同之处在于可以支持map,foreach,flatMap等方法.还可以用在for表达式里面.下面看Option能够方便的解决哪些问题.

* 创建对象或返回默认值.map调用可以自动略过对于None的处理,保证把方法应用在Some值上,并总体依然返回一个Option.同理filter也是一样的,自动解析Some或None,最终返回一个Option容器.避免了繁复的if结构.

  ``` scala
def getTemporaryDirectory(tmpArg: Option[String]): java.io.File = {
  tmpArg.map(d => new java.io.File(d)).
    filter(_.isDirectory).
    getOrElse(new java.io.File(System.getProperty("java.io.tmpdir")))
}

```

* 如果变量已初始化则执行代码块.利用foreach的特性,遍历Option里的所有值,因为Option值只有零或一个.所以代码块要么执行,要么不执行.
