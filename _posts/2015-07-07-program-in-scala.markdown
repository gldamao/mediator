---
layout: post
title: "Programe In Scala读书笔记"
date: 2015-07-09 14:32:59 +0800
comments: true
categories: scala 读书笔记
---

[Chapter.7 内建控制结构](#ch7)

[Chapter.8 函数和闭包](#ch8)

[Chapter.9 抽象控制](#ch9)

[Chapter.10 组合与继承](#ch10)

[Chapter.11 Scala的层级](#ch11)

[Chapter.12 特质](#ch12)

[Chapter.15 样本类和模式匹配](#ch15)

[Chapter.16 使用列表](#ch16)

[Chapter.17 集合类型](#ch17)

[Chapter.23 重访For表达式](#ch23)

[Chapter.24 抽取器Extractor](#ch24)

[Chapter.28 Scala的相等性方法浅析](#ch28)

[Chapter.30 Actor和并发](#ch30)

[Scala中处理null的终极解决方案--Option](#option)

[Scala类型系统](#classtype)

## <span id="ch7">Chapter.7 内建控制结构</span>

###等效推论(equational reasoning)
在表达式没有副作用的情况下，引入的变量等效于计算它的表达式。无论何时都可以用表达式来代替变量名。
###循环：
* while和do while的结构和用法同中java相同，在scala中被称作循环，并不是表达式，从而也不会产生有意义的结果，结果类型是Unit。
* 一般来说使用while的的实现都可以转化为递归函数的调用。
* Unit的有且唯一的实例写作 ()，这是scala与java很不同的一点，void是没有任何实例对象的。
* scala中赋值语句的返回值始终是Unit，Unit与任何值相比较都会返回true。
同时鼓励在代码中像质疑var那样质疑while，因为一般var和while都是成对出现的。因为while不产生值。为了让程序有所改变，通常不是更新var的值就是有其他的I/O操作。
* for可以和generator结合使用，可以兼容任何种类的集合，不仅仅是数组。或者说->符号右侧的表达式必须支持foreach方法。
* for语句中可以添加过滤条件，例如：

```scala
val files = java.io.File(".").listFiles
for (file <- files if file.getName.endsWith(".scala"))
  println(file)
```

*其中过滤器的个数不受限制*，但是在过滤条件不止一个的时候在每个条件之间需要用;来进行隔离。

```scala
for (
  file <- files
  if file.isFile;
  if file.endsWith(".scala")
) println(file)
```

为了简化语法，可以用大括号来包裹发生器和过滤器，同时在有括号中放置多个 **<-** 子句就表示嵌套的循环。

```scala
val fileHere = (new java.io.File(".")).listFiles

def fileLines(file: java.io.File) = {
  scala.io.Source.fromFile(file).getLines.toList
}

def grep(pattern: String) =
  for {
    file <- fileHere
    if (file.endsWith(".scala"))
    line <- fileLines(file)
    if line.trim.matches(pattern)
  } println(file + ": " + line.trim)

grep(".*gcd.*")
```

* mid-stream（流间）变量绑定，流间变量绑定的值都是val属性（关键字可省略）
* 利用for制造新集合  ```for {子句} yield  {循环体}```

```scala
val fileHere = (new java.io.File(".")).listFiles

def fileLength(file: java.io.File) = {
  scala.io.Source.fromFile(file).getLines.toList
}

val lineLength = for {
  file <- fileHere
  if (file.getName.endsWith(".scala"))
  line <- fileLength(file)
  trimedLine = line.trim
  if trimedLine.matches(".*for.*")
} yield trimedLine.length
```

###异常表达式的处理以及对控制流程的影响
* throw也是有结果类型的表达式。
* 把抛出的异常当作任何类型的值都是安全的，任何使用从throw返回值的尝试都不会起作用。
* throw表达式的结果类型是Nothing。
* if的一个分支计算值，另外一个分支抛出异常，那么整个if表达式的返回值就是实际计算分支的类型。
* try catch结构中使用模式匹配。在下面的例子中如果异常并不属于其中给定的两个类型，则异常会沿着调用堆栈继续向上传递。
```scala
try {
  val f = new FileReader("filename")
} catch {
  case ex: FileNotFoundException => // Handle missing file
  case ex: java.io.IOException => //Handle error I/O 
}
```
* finaly中尽量避免返回任何值，这种情况的处理方式和java有较大的差异。保证逻辑的简单。一般就是做一些资源清理的工作。


###match表达式：

* match表达式的强大之处在于它可以**支持任意模式的匹配**。
* 每个case分支都隐含了break功能。
* match表达式是可以生成值的！

- - - 

## <span id="ch8">Chapter.8 函数和闭包</span>
###本地方法
Scala中允许在一个方法内进行其他方法的定义，被包含的方法就叫做本地方法，**本地方法可以访问包含他们的方法的参数**。
一般本地方法是用来进行命名空间的隔离，功能相对独立和系统的Help方法可以考虑写成本地方法。
place holder: 被运行时传入的参数所代替，占位符需要足够的信息来明确自己将来要被什么类型的数据所代替，例如 _ + _ 是不合法的表达式。可以通过指定类型来改进上述表达式 (_:Int) + (_:Int) 
一个表达式中可以出现多个占位符，他们表达的意思是**有多个参数需要替换，而不是一个参数的多次替换**。

###偏应用函数（Partially applied function）
Scala的世界中，当带哦用函数时，传入任何需要的参数，就叫做把函数应用到参数上。而偏应用函数是一种表达式，不要求提供函数所需要的所有参数，仅仅给出其中一部分或者根本不提供所需参数。例如：
```Scala
scala > val a = sum _ 
a: (Int, Int , Int) => Int = <function>
```
对于上述代码，Scala编译器会片应用函数表达式 sum _ , 实例化一个*带三个缺失整形参数*的**函数值**，并赋值给变量a。当把新的到的函数值a应用到三个参数上时，就会去调用sum并传入这三个参数。

实际上变量a指向一个由Scala编译器依照偏应用函数表达式sum _ ，而实例化的一个函数值对象（实例），编译器生成的类中包含了一个apply方法，该方法的参数个数同sum _ 表达式缺少的参数数量。so... Scala编译器把表达式a(1, 2, 3)翻译成对函数值的apply方法调用并传入三个参数1, 2, 3. 所以上述调用就被转换为：

```Scala
scala> a.apply(1, 2, 3)
res1: Int = 6
```

以上这种一个下划线代表了全部参数列表的表达式的另一种用途，就是把它当作转化def为函数值的方式。

**尽管不能把方法或嵌套函数复制给变量或当作参数传递给其他方法，但是如果你把方法或嵌套函数通过在名称后面加一个下划线的方式包装在函数值中，就可以做到了。**

如果一个忽略所有参数的偏应用表达式，且代码在那个地方正需要一个函数，那就可以去掉下划线而另表达更简明。例如：```someNumbers.foreach(println _)```可以写成```someNumbers.foreach(println)``` 后一种格式仅在需要写函数的地方，**foreach需要一个函数作为参数传入。**如果在不需要函数的情况下， 尝试使用这种格式将引发一个编译错误。

> 注意：Scala需要制定显示省略的函数参数，尽管简单到只用一个‘_’。Scala仅允许在需要函数类型的地方才能省略这个'_'。

###闭包 Closure
通过“捕获”自由变量的绑定对函数文本执行的“关闭”行动。
针对闭包引出两个概念：

1. 封闭术语(closed term)： 在函数编写的时候就已经封闭了，没有任何带有自由变量的函数文本。
2. 开放术语(open term)： 在运行期间创建的函数值必须不或某些自由变量并绑定。

如果在闭包被创建之后，被封闭的元素发生了变化。Scala依然能够捕获到变化，可以说Scala封闭的是变量本身，而不是the value of variable。

如果闭包访问了某些在程序运行时有若干不同备份的变量时，在创建闭包的时刻活跃的值将会被封闭。

###重复参数
Scala支持方法的**最后一个参数**被指定为重复参数。只需在参数的类型后加一个“星号”。

```Scala
scala> def echo(args: String*) = 
  for (arg <- args) println(arg)
```
在方法内部是把重复参数的类型当作声明参数类型的数组来处理。*在作为参数时是**多个**,**重复**,**独立**的参数*，而在方法内部是Array[T]类型，所以如果想在参数调用时传递Array，则需要把Array元素展开：

```Scala
scala> echo(arr: _*)
```
把参数类型写成 “_*”可以展开Array对象。这个标注告诉编译器把Array的每个元素当作参数依次传入，而不是当作单一的参数传入。

###尾递归(Tail recursion)
说白了就是在方法的最后再次调用该方法本身，Scala编译器检测到尾递归就用新值更新函数参数，然后把它替换成一个回到函数开头的跳转。
**道义上不应该修与使用递归算法解决问题，递归经常是比基于循环的更优美和简明的方案。如果方案是为递归，就无需付出任何运行期开销。**尾递归的追踪过程会相应的变得繁琐，不过可以通过```-g:notailcalls```参数，来关闭尾递归优化。
####尾递归的局限性
由于VJM的指令集的关系，Scala只能优化直接递归调用使其返回同一个函数。

1.如果递归是间接的，例如：

```Scala
def isEven(x: Int): Boolean = 
  if (x == 0) true else isOdd(x - 1)
def isOdd(x: Int): Boolean = 
  if (x == 0) false else isEven(x - 1)
```
2.如果最后一个调用是一个**函数值**，那么也不能获得尾递归优化。例如：

```Scala
val funValue = nestedFun _
def nestedFun(x: Int) {
  if (x != 0) { println(x); funValue(x - 1) }
}
```
funValue变量实际上指向了一个包装了nestedFun的方法调用的函数值。当把这个函数值应用到参数上（一次方法调用过程，*方法应用到参数上*），它会转向把nestedFun应用到同一个参数，并返回结果。
但是这样Scala还是受限于方法或嵌套函数在最后一个操作调用本身，而没有转到某个函数值或什么其他的中间函数的情况而不能进行尾递归优化。


> 如果方法最后都是返回自身的**函数值**调用，则永远无法进行尾递归优化，另外也不能在函数调用后再附加其他操作，只能光秃秃的做一次自身调用！！！

---

## <span id="ch9">Chapter.9 控制抽象</span>
###高阶函数
高阶函数可以大幅度的减少代码重复，其实所有的函数都可以分割成两部分：

1. 通用部分：他们在每次函数调用中都相同，一般指函数体。
2. 非通用部分：在不同的函数调用中可能会变化。一般指函数参数。

> 当把函数值用作参数时，算法的非通用部分就是它(函数值)代表的某些其他算法。在这种函数的每一次调用中，都可以把不同的函数值作为参数传入，于是被调用函数将在每次选用参数的时候调用传入的函数值。这就是所谓的**高阶函数：higher-order function**。

###Curry化
用于制造“感觉像是原生语言支持“的控制抽象。如下例，curry化函数的调用在本质上是连续调用了两个传统函数。地一个函数调用带单个名为x的Int参数，并返回第二个函数的函数值。第二个函数带Int参数y。**也就是说curry化会依次应用多个传统函数到每个参数上。其中每一次应用都返回一个偏应用函数的函数值。**


```scala
scala> def curriedSum(x: Int)(y: Int) = x + y
curriedSum: (Int)(Int)Int
scala> curriedSum(1)(2)
res1: Int = 3
```

---

## <span id="ch10"> Chapter.10 组合与继承</span>
###抽象类

抽象类之前必须有abstract类型修饰,同时如果一个类中包含了未被实现的方法,那么他就必须被声明为抽象类.显然想要实例化一个抽象类对象是不允许的.**拥有抽象成员的类必须显式的用abstract指明是抽象类**.

**无参数方法**: 对于无参数的情况下有两种定义方式. *空括号*和*省略括号*.

> 省略括号(empty-paren method): 方法本身不接受参数, 方法仅仅读取变量值并不改变变量值.这个惯例支持统一访问原则(uniform access principle),*选用字段还是方法来实现属性,不应该影响到客户端代码.* 在性能上字段应该会比方法稍稍快一些,字段值在类初始化时预计算,而方法是在每次调用的时候都重新计算.
> 空括号: 如果方法调用中发生了I/O,或者修改了某些var变量,那么最好还是加上一对空括号.
> **简单的总结一下可以归纳为,如果方法调用没有副作用,推荐省略括号,否则最好加上一对空括号. 如果调用的函数执行了操作,就使用括号.如果仅仅提供对某个属性的访问,就省略括号.**

###重载方法和字段

在Scala中与java很大的一个区别是**字段和方法都在同一个name space中**.如此一来字段可以很轻松的重载无参数方法.相应的在同一个类中不允许使用同一个名字来定义方法和字段.

####Scala与java的命名空间对比:

> Scala中的命名空间: **值**(字段,方法,包,单例对象)  **类型**(类,特质) 
> Java中的命名空间:**字段**,**方法**,**类型**,**包**.

####定义参数化字段

在类的继承关系当中,如果子类要通过参数来实例化父类中的某些字段,可以通过参数化字段来避免重复.


``` scala
abstract class Element {
  def contents: Array[String]
  val height = contents.length
  val width = if (height == 0) 0 else contents(0).length
}

class ArrayElement( //注意,这里是小括号
  val contents: Array[String]
) extends Element
......

class ArrayElement(x123: Array[String]) extends Element {
  val contents: Array[String] = x123
}
```

在同一个时间使用相同的名称定义参数和字段的一种简写方式,现在ArrayElement有可以通过外部访问(不能重新赋值,val)的contents字段.

**继承inheritance,重载override,实现implement**.在子类中重载的如果是抽象方法,那么这个过程也叫做实现了该方法.**当实现某个父类的方法时,可以省略override关键字.同样的规律也可以应用到定义参数化字段上.**


++操作连接两个数组,Scala中的数据被表示为Java数组,但是会支持更多的方法.同时Scala中的数据继承自类scala.Seq.


```Scala
object Element {
  private val a = "123"

  private class ArrayElement(val contents: Array[String]) extends Element

  private class LineElement(s: String) extends Element {
    override def contents: Array[String] = Array(s)
  }

  private class UniformElement(ch: Char, override val length: Int, override val width: Int) extends Element {
    val line = ch.toString * width

    def con = for {
      i <- 0 until length
    } yield line

    def contents = con.toArray
  }

  def elem(s: Array[String]): Element = new ArrayElement(s)

  def elem(c: Char, length: Int, width: Int): Element = new UniformElement(c, length, width)

  def elem(s: String): Element = new LineElement(s)

}

abstract class Element {
  def contents: Array[String]

  def length = contents.length

  def width = if (length == 0) length else contents(0).length

  def above(other: Element) = Element.elem(this.contents ++ other.contents)

  def beside(other: Element) = {

  }

  override def toString(): String = {
    contents.mkString("\n")
  }

}

println(Element.elem('a', 2, 3).toString)
```

以上这段代码演示了以上几种用法,需要注意的几点:

1. 声明与调用的顺序,要求声明或定义必须出现在调用前.
2. toString是要返回一个String类型的.
3. 在一个类内部定义带有private属性的其他类,即使可以在该类内部进行实例化也不能把引用传递到外部,除非像上面这种有继承关系的,返回父类类型给外部.
4. 伴生对象与类之间可以互相访问私有成员,但是并没有共享命名空间.

---

## <span id="ch11"> Chapter.11 Scala的层级</span>
###Scala的类层级

Any是所有类的父类,内包含toString, hashCode, ==, !=, equals等方法,其中 ==与!=是带有final属性的,所以不能重载,但是可以通过重载equals来改变==和!=的含义.实际上Scala中的相等操作==被设计成对比较类型透明的,对于值类型而言是数学或布尔相等,对于引用类型而言,则被视为继承自Object的euqals方法的别名.默认被实现为判断引用相等.如果想要强制采用引用对比,**可以使用eq和ne操作符**.

所有的内建值类型都有一个公共的父类AnyVal,在运行时表示成Java的原始值,**不能用new创造这些类的实例,这些类都被定义为final和抽象的.** 最后,Unit有且仅有一个实例,被写作().

![Scala当中的继承关系](http://7tsz8v.com1.z0.glb.clouddn.com/classhierarchy.png)

Scala中的AnyRef是引用类的基类.在Java平台上AnyRef实际就是类java.lang.Object的别名.在Java平台上两者是可以交换使用,但是推荐在任何地方都使用AnyRef.

底层类型:

类Null是null类型的引用;它是每个引用类型的子类,所以Null不兼容任何值类型.类Nothing在整个继承链的最底端,它是任何其他类型的子类型.**然而这个类型并没有值**.他的最主要用途是可以和其他任何类型做兼容,例如在如下的if语句中,返回值可以始终看作是Int类型.

```Scala
def error(message:String): Nothing = throw new RuntimeException(message)

def divide(x: Int, y: Int):Int = 
  if (y != 0) x / y
  else error("can't divide zero!")
```

这样即使是通过else分支返回,Nothing也是Int的子类型,所以divide整体返回Int是没有问题的.

---

## <span id="ch12"> Chapter.12 特质</span>
###特质 trait

特质的定义除了关键字的差异之外,其他都和类定义无异.(如果特质没有声明超类,那么就有一个缺省的超类AnyRef),特质和java中的接口很类似,只不过特质中可以包含具体的方法.
混入特质的两种方式:

1. 使用extends关键字混入特质,**这种情况下已经隐式的继承了特质的超类**.
2. 使用with关键字混入特质.

特质与类的两个明显差异:

1. 特质不能有任何的"类"参数.不能传递任何参数给特质的主构造器. //todo Section20.5 有方法可以规避该限制
2. 在类中,super是静态绑定的.而在特质中是动态绑定的.调用的实现将在每一次特质被混入到具体的类中时才被确定.这种方式实现了特质的**可堆叠的改变**特性.

####胖 vs 瘦
特质最为重要的一个用途,是丰富现有的类.把相对独立的功能追加到某一个类上.丰富接口功能,小瘦子变身大胖子.

####特质用来做可堆叠的改变
在不改变原始基类的情况下,可以增加一些细节处理上的变化.

```Scala
import scala.collection.mutable.ArrayBuffer

abstract class IntQueue {
  def get(): Int
  def put(x: Int)
}

class BasicIntQueue extends IntQueue {
  private val buf = new ArrayBuffer[Int]
  def get() = buf.remove(0)
  def put(x: Int) { buf += x }
}

trait Doubling extends IntQueue {
  abstract override def put(x: Int) { super.put(2 * x) }
}

trait Incrementing extends IntQueue {
  abstract override def put(x: Int) { super.put(x + 1) }
}

trait Filtering extends IntQueue {
  abstract override def put(x: Int) {
    if (x >= 0) super.put(x)
  }
}
```

> 特质可以继承自某个类(也就是说特质拥有了一个显式指明的超类),这表示该特质只能被混入继承自该超类的子类中.
> **在特质中的super是动态绑定的**,Doubling中super的调用将直到被混入另外一个类或特质之后有了具体的方法定义时才工作.
> 为了告诉Scala编译器我们的目的,需要在这种需求的方法前加上```abstract override```的标志.**这种组合只能出现在特质中**.
> 最右侧的特质先起作用,如果那个方法调用了super,它调用其左侧的方法.

####why trait not multiple inheritance
最重要的差别在于对super的解释:

* 多重继承中,super的调用是静态绑定,具体调用哪个方法可以在定调用发生的地方明确决定.
* 对于特质而言,super的调用是由类和被混入到类中的特质的**线性化(linearization)**所决定的.

####如何确定选用是使用特质还是抽象类

1. 如果行为不会被重用,就使用**具体类**,具体类没有可重用的行为.
2. 如果要在多个不相关的类中重用,就选用**特质**,特质可以被混入到不同的类层级当中.
3. 如果你希望在java代码中继承它,就选用**抽象类**,特质没有特别近似的java代码相对应,另外**在java中继承特质是非常笨拙的**.
4. 只含有抽象方法的特质会被直接翻译成java接口,这种情况下两者兼容性非常棒.
5. 如果要追求效率上的极致,应该更倾向于使用类.


---

## <span id="ch15"> Chapter.15 样本类和模式匹配</span>
#### 样本类自动添加的属性,方法和功能
1. 首先会添加与类名一致的工厂方法.省略冗长的new关键字.
2. 样本类的参数列表中所有参数隐含获取了val前缀,会在样本类内部自动生成相应字段(普通类的参数要加上val字段才能被自动生成类内属性).
3. 编译器会为样本类自动添加*hashCode, toString, equals*三个方法的缺省实现.其中equals方法是可以进行结构化比较.
4. 最方便的是样本类支持模式匹配功能.

#### 模式匹配
* 通配模式, (_)一半用作最后一种default的匹配.
* 常量模式:任何字面值都可以作为常量.
* 变量模式:可以匹配任意对象,会把变量绑定到匹配的对象上.
* 构造器模式:对```case BinOp("*", e, Number(0)) => println("a deep match")```这种形式进行匹配.
* 序列模式:可以支持指定模式内任意数量的元素.也可以匹配不指定长度的序列.

```Scala
expr match {
  case List(0,_ ,_) => println("found it!")
  case _ => 
}

expr match {
  case List(0, _*) => println("found it!")
  case _ => 
}
```

* 元组模式:匹配元组同时可以把元组拆解.
* 类型模式:同时也可以在类型匹配后利用if语句做进一步的匹配(模式守卫).另外类型匹配可以融合类型测试和类型转换为一体,这两个操作在Scala当中是比较繁琐的.**对于泛型的处理同java，会在运行时擦出类型信息。所以在使用List等集合进行类型模式匹配式无效，一般直接写成```case a: List[_] => ...```，_表示任意类型。**
* **变量绑定**:非常有用的功能,相当于把变量模式与其他模式进行了组合, 变量名 + @ + 模式.这种做法的意义在于它能像通常的那样做模式匹配同时如果匹配成功又可以把变量设置成匹配的对象.

```Scala
expr match {
  case Unop("abs", e @ UnOp("abs", _)) => e
  case _ => 
}
```


* 模式守卫: 模式变量(并不是变量模式)仅允许在模式中出现一次,可以通过模式守卫来丰富匹配规则.

```Scala
def simplifyAdd(e: Expr) = e match {
  case BinOp("+",x, y) if x == y =>
    BinOp("*", x, Number(2))
  case _ => e
}
```

模式本身的功能非常强大,还可以支持如下几种:

* 模式还可以用来解构一些对象或元组等数据结构.

```Scala
val exp = new BinOp("+", Number(5), Number(1))
val BinOp(op, left, right) = exp
```

> 函数体样本,花括号内的样本序列(备选项)可以用在能够出现函数字面量的任何地方.**实际上样本序列就是函数字面量**.函数字面量只有一个入口点和参数列表,而样本序列可以有多个入口点,同时每个入口点拥有自己独特的参数列表.

```Scala
val withDefault: Option[Int] => Int = {
  case Some(x) => x
  case None => 0
}
```

上述这种情况在Actor库中使用的非常频繁.
**定义了一个val的函数字面值,接下来用函数体样本来完善函数体,里面提供了多个函数入口.这样就可以生成出不同种的多个函数了,具体执行的哪个入口是要通过对参数进行模式匹配来得出的**.

* 用作偏函数的样本序列,可以用来检查偏函数是否被定义. 偏函数有一个isDefinedAt方法,可以用来测试函数是否对某个特定值有定义.

```Scala
val second: List[Int] => Int = {
  case x :: y :: _ => y
}

val secondPartial: PartialFunction[List[Int], Int] = {
  case x :: y :: _ => y
}

```

#### 类型擦除:同java一样,scala在运行期不保存类型参数信息,**对于map等需要类型参数信息的匹配只能精确到是某种任意类型的map.唯一特殊的类型是数组,在Scala和java当中数组都被特殊处理了,数组元素类型和数组值保存在一起,因此数组可以用来做模式匹配.**

```Scala
//这么做是存在问题的,类型被擦除了
def isIntMap(x: Any) = x match {
  case m: Map[Int, Int] => true
  case _ => false
}

def isStringArray(x: Any) = x match {
  case a: Array[String] => "yes"
  case _ => "No"
}
```
####封闭类 sealed class
封闭类可以保证在超类定义文件之外的任何地点不能再添加子类.例如在模式匹配中无法保证有一个合理的default处理,一般来说这是不可能的,因为新的样本类可以定义在任何时刻和任何位置,而无法保证目前设定的模式匹配逻辑对于新定义样本类也是"合理合法的".
而如果使用封闭类,把所有子样本类都限定在超类定义的文件中,就可以focus在已知的样本类上.**同时如果使用继承自封闭类的样本类做匹配,编译器可以通过警告信息标识出缺失的模式组合.** 不能再赞有没有!!!
如果在明确目的的情况下可能要让编译器对于部分为做匹配的模式警告闭嘴,可以采用通配模式并抛出一个RuntimeException(为了以防万一)或为选择起的表达式加上注解.

```scala
def describe(e: Expr):String = (e: @unchecked) match { .....}
```

####option 可选值
option包含了两种形式的值,Some(x)和None,其中x是实际的值,None表示缺失.分离可选值最好的方式就是采用模式匹配来处理.

---

## <span id="ch16"> Chapter.16 使用列表</span>
#### 列表字面量
列表和数组不同,列表是不可变的.

1. 不能通过赋值来改变列表的元素.
2. 列表具有递归结构(说的是可以通过递归的方法来访问eg: LinkedList),而数组是连续的.

**Scala中的列表是协变的,对于每一对类型S和T来说,如果S是T的子类,那么List[S]是List[T]的子类型. 空列表List()的类型是List[Nothing],所以空列表可以赋值给任何类型的列表引用**.

``` Scala
def isort(xs: List[Int]): List[Int] = {
  if (xs.isEmpty) Nil
  else insert(xs.head, isort(xs.tail))
}

def insert(x: Int, xs: List[Int]): List[Int] = {
  if (xs.isEmpty || x <= xs.head) x :: xs
  else insert(xs.head, insert(x, xs.tail))
}
```

以上是使用Scala实现的插入排序,iosort方法接收一个参数类型为Int的列表,返回一个排序好的列表,而helper函数insert接受一个Int参数和一个List,含义是把该Int变量插入到一个已经排好序的列表中,并返回插入新元素的排序后列表.

####列表模式
列表和模式的结合使用,重点在于利用模式来解构列表.长度已知(如果长度不符则抛异常)和长度未知的两种常用结构方式:
   
    scala> val List(a, b, c) = fruit
    a: String = apples
    b: String = oranges
    c: String = pears

    scala> val a :: b :: rest = fruit
    a: String = apples
    b: String = oranges
    rest: List[String] = List[String](pears)

从列表的两个常用操作符(:: :::)上可以看出,列表的构建都是从后向前的.参考append方法的实现:

```Scala
def append[T](x: List[T], y: List[T]): List[T] = {
  x match {
    case Nil => y
    case xa :: xb => xa :: append(xb, y)
  }
}
```

> 与head,tail想对应的是init,last.head与tail是与列表长度无关的,总可以在常量时间内返回结果,而init和last需要遍历整个list,执行时长与list长度成正比.所以组织好结构,让常用的数据都放置在头部,而不是尾部.

####List中几个比较常用的操作:

* 列表的常规方法
  * apply
  * indices   返回列表所有有效的索引值组成的列表.
  * take  
  * drop
  * splitAt   xs splitAt n <=> (xs take n, xs drop n)
  * toArray <=> toList
  * copyToArray   把列表元素拷贝到指定Array中,第二个参数表示在Array中从指定下标后开始存数据,要保证Array有足够的长度!!
  * toString
  * mkString
  * elements  返回一个迭代器
* 列表间映射
  * map
  * flatMap   参数是一个能够返回元素列表的函数,对列表中的每个元素应用该方法,然后连接所有方法的结果并返回.(先map得出一坨List,而后flat,得到一个大的List)
  * foreach   参数是一个过程(返回Unit的方法),对每个元素应用一次,并不改变元素.
* 列表的过滤
  * filter    
  * partition
  * find  与filter方法类似,返回的是第一个满足*论断函数*的元素.而不是全部元素.
  * takeWhile 返回列表中最长能够满足*论断函数*的前缀.
  * dropWhile 移除掉最长能够满足*论断函数*的前缀.
  * span  xs span p <=> (xs takeWhile p, xs dropWhile p)
* 列表的论断
  * forall    对整个列表元素的论断
  * exists    同上
* 列表的折叠(斜杠的方向和操作树的倾斜方向是一致的,利用倾斜方向来区分左右折叠)
  * fold left   (z :/ xs)(op) z:初始值,xs:列表,op:二元操作,应用到初始值和每个相邻元素上. *z :/ List((a, b, c))(op)相当于op(op(op(z, a), b), c)*
  * fold right  (xs :\ z)(op) *(List(a, b, c) :\ z)(op) 等价与 op(a, (b, op(c, z)))*
  * **右折叠的效率要高于左折叠**
* 列表排序
  * sort    xs sort Compator    按照Compator来对xs列表进行排序.(内部的排序算法归类为归并排序)
* 创建列表
  * List.apply  通过元素创建列表
  * List.range  创建数值范围(可指定步长)
  * List.make   创建统一的列表
* 解除啮合列表
  * unzip   belong to Object List not Class List, unzip only can take List[Pair] as paramter. And Class method require the ability to apply to any parameter.
* 连接列表
  * flatten 以列表的列表作为参数(List[List[T]])     List.flatten(), belong to Object List bacause the same reason like List.unzip
  * concat  连接多个List,与flatten不同但很类似,flatten是一个List内部的打平,而concat是连接多个低维的List.

####一个递归的Sample代码,结合了curry,partial application function.

```Scala
def msort[T](less: (T, T) => Boolean)(xs: List[T]): List[T] = {
  def merge(xs: List[T], ys: List[T]): List[T] = {
    (xs, ys) match {
      case (_, Nil) => xs
      case (Nil, _) => ys
      case (x1 :: x2, y1 :: y2) =>
        if (less(x1, y1)) x1 :: merge(x2, ys)
        else y1 :: merge(xs, y2)
    }
  }

  val n = xs.length / 2
  n match {
    case 0 => xs
    case _ => merge(msort(less)(xs.take(n)), msort(less)(xs.drop(n)))
  }
}

val intSort = msort((a: Int, b: Int) => a < b) _
```

---

## <span id="ch17"> Chapter.17 集合类型</span>
####Iterator与Iterable

* 两者有很多相同的方法,不过两者并不在一个层级内,特质Iterator扩展了AnyRef.
* Iterable是可被枚举的类型,而特质Iterator是用来执行枚举操作的机制.
* Iterable可以被枚举若干次,而Iterator只能被枚举一次.

####更为高级的集合构建方法
* 列表缓存
  * List可以对列表的头部(而非尾部)进行快速的访问.
  * ListBuffer能够支持常量时间的添加和前缀操作.元素添加用+=,元素前缀用+:.
* 数组缓存
  * ArrayBuffer与数组类似,只是额外还允许在列表的开始和结束的地方添加和删除元素.
  * 由于实现中包装层导致执行得稍微有些慢,新的添加和删除操作平均为常量时间,但是偶尔会因为实现申请分配新的数组以保留缓存内容而需要**线性处理时间**.
* 队列(Queue)
  * 分为可变和不可变的两种Queue.
  * Immutable Queue通过enqueue(可支持列表作为参数而批量添加)和dequeue(返回由队列头部元素和移除后队列剩余部分组成的Tuple2)分别实现在队尾入队和在队首出队.
  * Mutable Queue使用+=*(添加一个元素)*和++=*(通过List来批量添加)*来添加元素,同时对于dequeue操作将只从队列移除头元素并返回.
* 栈(Stack)
  * 同样分为可变与不可变版本(不可变版本没弄明白怎么用,push多次后发现还是空的)
  * top, pop, push
* 字符串(经RichString隐式转换)
  * RichString的类型是Seq[Char],而Predef又包含了从String到RichString的隐式转换,所以任意字符串都可以看作是Seq[Char].

> 默认情况下直接使用Map和Set等集合类都是使用的Immutable版本,倾向于这种选择是由于Predef对象的支持.它被每个Scala源文件隐含引用.可变集里面一般都包含+=和++=操作符,在不可变集中是不支持这个操作的.

```Scala
object Predef {
  type Set[T] = scala.collection.immutable.Set[T]
  type Map[K, V] = scala.collection.immutable.Map[K, V]
  val Set = scala.collection.immutable.Set
  val Map = scala.collection.immutable.Map
  ...
}

```

####集合的排序
* List可以直接调用sorted方法进行排序.也可以使用sortWith方法来提供自定义的排序标准.
* 把任意一个自定义类混入特质Ordered就可以支持sorted操作.

```Scala
class Person (var name: String) extends Ordered [Person] {

  override def toString = name
  // return 0 if the same; negative if this < that; positive if this > that
  def compare (that: Person) = {
    if (this.name == that.name)
      0
    else if (this.name > that.name)
      1
    else
      −1
  }

}
```
> 不可变类型的befit: 存储结构上更为紧凑.一个空Mutable HashMap的大小为80字节左右,每添加一个元素就附加16字节.

####Immutable和Mutable之间的转换
带有等号的操作符在Immutable对象上是不可使用的,但是如果配合var的属性定义就可以被scala编译器所支持,属于一个两者之间转换的语法糖.

---

## <span id="ch23"> Chapter.23 重访For表达式</span>
### for循环的解析
先来看一个需求的两种不同实现,找出列表中所有妈妈和孩子结对的名称.

``` scala
case class Person(name: String, isMale: Boolean, children: Person*)

val lara = Person("Lara", false)
val bob = Person("Bob", true)
val julie = Person("Julie", false, lara, bob)

val persons = List(lara, bob, julie)

val s = persons.filter(p => !p.isMale) flatMap(p => p.children map (c => (p.name,c.name)))
- - - 
for {
  p <- persons
  if (!p.isMale)
  c <- p.children
} yield (p.name, c.name)

```
Scala编译器会把第二种实现方式的代码转译成地一个.*所有的能够yield(产生)结果的for表达式都会被编译器转译为高阶方法map, flatMap及filter的组合调用.所有不带yield的for循环都会被转译为仅对高阶函数filter和foreach的调用.*
看一个更复杂一点的N皇后问题

``` scala
def queens(n: Int): List[List[(Int, Int)]] = {
  def p3333laceQueens(k: Int): List[List[(Int, Int)]] =
    if (k == 0)
      List(List())
    else
      for {
        queens <- placeQueens(k - 1)
        column <- 1 to n
        queen = (k, column)
        if isSafe(queen, queens)
      } yield queen :: queens

  placeQueens(n)
}

def isSafe(queen: (Int, Int), queens: List[(Int, Int)]) = queens forall (q => !inCheck(queen, q))

def inCheck(q1: (Int, Int), q2: (Int, Int)) =
  q1._1 == q2._1 || 
  q1._2 == q2._2 ||
  (q1._1 - q2._1).abs == (q1._2 - q2._2).abs
```

* 检验某两个点是否可以兼容放置皇后的isCheck方法.
  1. 不能用同行皇后.
  2. 不能有同列皇后.
  3. 不能有同对脚线皇后.(两个点的横纵坐标之差相等表示两者在一条对脚线上)
* isSafe方法用来检测某个点对之前所有的皇后位置组合进行测试.
* 对于k皇后问题,利用递归回溯的策略,先解决k-1皇后问题,用k-1皇后的全体解组合来与第k个皇后放置列位置进行验证.
* 对列位置从1到n依次进行验证,如果合格就返回一种组合方案.

####给列表去重的一段代码
其中最后一行代码可以用注释掉的for来代替

``` scala
def removeDuplicates[A](xs:List[A]): List[A] = {
  if (xs.isEmpty) xs
  else
    xs.head :: removeDuplicates(
      xs.tail filter(x=> x!= xs.head)
      // for (x <- xs.tail if x != xs.head) yield x
    )
}
```

###for表达式的转译
每个for表达式都可以用三个高阶函数map, flatMap和filter来表达.
####转译带一个生成器的for表达式
```for(x <- expr1) yield expr2``` 可以转译成 ``` expr1.map(x => expr2)```

####转译以生成器和过滤器开始的for表达式
```for (x <- expr1 filter (x => expr2)) yield expr3``` 可以转译成 ```expr1 filter (x => expr2) map(x => expr3)```

####转译以两个生成器开始的for表达式

* seq是任意序列的生成器,定义及过滤器.
* seq也可能是空的,这是偶seq前的分号就可以省略了.

``` for(x <- expr1; y <- expr2; seq) yield expr3```  可以转译成 ```expr1.flatMap(x => for (y <- expr2;seq) yield expr3 )```

两 个生成器的转译例子,需求是找出写过两本书以上的作者:
``` scala
for {
  b1 <- books
  b2 <- books
  if b1 != b2
  a1 <- b1.authors
  a2 <- b2.authors
  if a1 == a2
} yield a1

//可以被转译为如下表达式

books.flatMap(b1 => 
  (books fliter (b2 => b1 != b2) flatMap(b2 =>
    b1.authors flatMap(a1 => 
      b2 filter (a2 => a1 == a2) map (a2 =>
        a1))))

```

####转译生成器中的元组和模式
如果生成器的左侧带有模式(直接生成一种模式的做法)或元素而不是简单的变量,情况就会稍微复杂一些
```for((x1, x2, x3, ..., xn) <- expr1) yield expr2```可以转译成```expr1.match { case(x1, x2, x3, ..., xn) => expr2 }```

``` scala
for(pat <- expr1) yield expr2
//会被转译为
expr1 filter {
  case pat => true
  case _ => false
} map (
  case pat => expr2
)
```
**生成器的条目首先经过过滤,保留匹配pat模式的元素,进而保证了模式匹配生成器不会抛出MatchError异常**.

####for中带有定义的转移
``` scala
for( x<- expr1; y = expr2; seq) yield expr3
//可以被转译为

for ((x, y) <- for(x <- expr1);seq) yield (x, expr2); seq)
yield expr3 
```

*每次产生新的x值时expr2都会被重新计算,防止expr2可能引用了x来改变自身的值,从而单独更新x的值会造成数值差异.*但是如果在生成器中定义了某些与循环无的变量则会大大降低运行效率,因为每次都会更新这个并不受循环新值x影响的变量.的变量则会大大降低运行效率,因为每次都会更新这个并不受循环新值x影响的变量.


####转译无生成器的for
```for(x <- expr1) body``` 可转译为 ``` expr1.foreach(x => bdoy)```


``` scala
for (x <- expr1; if expr2; y <- expr3) body
//可转译为 
expr1.filter(x => expr2) foreach(x => 
  expr3 foreach (y => body)) 
```

1. 由expr1中经过filter生成了expr3(由符合expr2条件的x构成).
2. 接下来又把每一个expr3中的元素应用到body方法上.

---

## <span id="ch24"> Chapter.24 抽取器Extractor</span>
##抽取器
####抽取器的来源和意义
在Scala中可以通过模式匹配(构造器模式)很方便简洁的来做数据的解构和分析,不过一直以来构造器模式还是和样本类相关联,实际当中可能会要求在在不创建关联的样本类的前提下写出类似的模式.或者更希望能够创造自己的模式类型.

例如要写一个Email地址的解析工具,代码如下:

``` Scala
def isEmail(s: String): Boolean
def domain(s: String): String
def user(s: String): String

if (isEmail(s)) println(user(s) + " AT " + domain(s))
else println("not an email address")

s match {
  case Email(user, domain) => println(user + " AT " + domain)
  case _ => println("not an email address")
}
```

字符串显然是无法满足上述模式匹配的要求，**抽取器可以为已存在的类型定义新模式，而这种模式不需要遵守类型的内部表达方式.**
抽取器是定义了unapply(匹配并分解值)成员方法的对象(object),也允许实现用于构建值的对偶方法apply.

``` scala
object Email {
    //注入方法,可选
    def apply(user: String, domain: String) = user + "@" + domain
    //抽取方法,必须
    def unapply(str: String): Optional[(String, String)] = {
        val parts = str split "@"
        if (parts.length == 2) Some((parts(0), parts(1))) else None
    }
}

```

> 这里还可以让Email继承Scala的函数类型,声明中的类型和Function2[String, String, String]是同意的,例如:

```object Email extends (String, String) => String ={ ... }```

抽取器由于含有了unapply而拥有了抽取的能力,于此同时也必须要考虑到传入的字符串可能并非email的情况,所以返回值**要求必须是Option和boolean(特殊情况,表示匹配成功或失败)**.
```selectorString match { case Email(user, domain) => ...}``` 将引发unapply调用
```Email.unapply(selectorString)```. 首先要判断参数是否符合unapply方法的参数类型,不符合则直接失败.

####变参抽取器
如果想要抽取不定数量的元素值,例如希望匹配代表域名的字符串,使得域的每一段都被拆分在不同的子模式中.
如下代码逆序匹配域名,参数列表尾部的序列同陪模式 _*, 匹配序列中所有剩下的元素.

``` scala
object Domain {
  def apply(parts: String*) {
    parts.reverse.mkString(".")
  }

  def unapply(whole: String): Option[Seq[String]] = {
    Some(whole.split("\\.").reverse)
  }
}

dom match {
  case Domain("org", "acm") => println("acm.org")
  case Domain("com", "sun", "java") => println("java.sun.com")
  case Domain("net", _*) => println("a .net domain")
}
```

### 样本类和抽取器之间的区别

* **抽取器可以把客户端代码独立开,**如果不见已经定义并导出了一套样本类,那么客户端就可能已经包含了对这些样本类的模式匹配.重命名某些样本类或修改类层级都将影响到客户端代码.
* 样本类的执行效率比抽取器更高,同时拥有静态的检查.
* 如果样本类继承自sealed基类,Scala编译器将采用穷据发检查模式匹配并在模式没有覆盖某种可能的组合值情况喜爱报错.

---

## <span id="ch28"> Chapter.28 scala的相等性方法浅析</span>
### Scala与java的equals方法

> "=="在java代码当中比较的是两个变量的引用(地址/对象),如果要在java中进行值比较则需要使用equals方法.

Scala中也有"=="操作符,只不过在Scala环境中表示的是值之间的比较.用来表示对象一致性的判断方法是eq.
**Scala中,==用来表示每个类型"自然的"相等性.对于值类型而言进行值比较,对于引用类型,相当与equals.**如果要改变==的默认行为可以在自定义类型中重写equals方法(==继承自Any,并且是final的),**继承的equals方法除非被重写,默认是像java那样判断对象是否一致.**因此equals(==)默认和eq是一样的.

###重写equals方法时的四个常见陷阱:
1. 重定义equals方法时采用了错误的方法签名.
2. 修改了equals方法但并没有同时修改hashCode.
3. 用可变字段定义equals方法.
4. 未能按同关系定义equals方法.

* equals方法接受的参数类型是Any,避免误写成自定义类型.否则在集合(范型擦除)这种范型类型中使用equals时调用的是Any里面的equals,结果就会比较诡异.一种稍稍优化的做法.

``` scala
override def equals(other: Any) = other match (
  case that: Point => this.x == that.x && this.y == that.y
  case _ => false
}
```
* 在Hash集合中进行相等性比较时,首先会根据元素的hash码分桶,例如contains检查首先决定要找的桶,然后再把给定元素同该桶中所有元素进行比较.而Any中定义的hashCode是根据已分配对象地址的某种转换.
* 关于Any中hashCode方法的契约:**如果两个对象equals方法相等,那么每一次调用hashCode方法都必须产出相同的整数结果.hashCode只能依赖equals所依赖的字段**
* 常用的hashCode方法```41 * (x + 41) + y```,同质数41进行运算,可以较为高效的获取到效果不错的hashCode方法.
* 使用var变量来定义equals方法,问题同样是出在配合集合使用时,构造对象P并加入到集合中,再改变由P的equals引用的var属性.这时P和集合中的P就"不相等"了.(已经不处在同一个hash桶,元素不可见且存在)
* equals的准则,必须对非null对象实现等同关系:
  * 必须是自反的.
  * 是对称的. x equals y 也就意味着 y equals x.
  * 它必须是可传递的. 
  * 它是一致的.对于任何非空值x和y,多次调用x.equals(y)的结果应该是一致的.
  * **对于任何非空值x, x equals null应返回false.

---

## <span id="ch30"> Chapter.30 Actor和并发</span>
### scala和java对于并发设计的不同之处

> java平台内置了基于共享内存和锁的线程模型,每个对象都关联了一个逻辑监控器,可以根据监控器来控制对数据的多线程访问.

对于将要被多线程共享的数据标记为*synchronized*来保证java runtime将应用一种锁机制来确保同一时间只有一个线程进入由同一个锁控制的同步代码段,以此来在多进程之间协同数据访问.

然而这种模型导致的问题就是控制复杂,编写代码时必须知道当下要申请哪些锁资源,以及正在修改或访问的数据可能会被其他线程修改或访问.频繁的锁操作以及竞争资源导致性能的大幅度下降.还有可能导致死锁.

Scala的actor提供了一种不共享任何数据,依赖消息传递的模型来实现并发.避免死锁.从根本上绕开了**共享数据+锁**所引入的问题.

#### actor和消息传递
actor当中的几个概念:

* 信箱: actor中接收并暂存发送给它的消息的空间.每个actor的信箱是独立的.
* 独立执行: actor是是一个类似线程的实体,每个actor使用一个独立的线程.
* actor的生命周期: 
  1. act方法返回.
  2. act方法由于异常被终止.
  3. actor调用exit方法.
* 消息投递是异步的,任何逻辑都不应依赖于消息到达信箱的顺序.
* 信箱中在receive方法被调用时如果没有消息,则该调用会阻塞,直到有消息抵达.
* 如果信箱中没有任何消息可以被偏函数处理,则对receive方法的调用也会阻塞,直到一个可以匹配的消息抵达.(为防止邮箱被沾满,需要有一个case _ 语句来处理任意的消息)

当actor发送消息是并不会组合,同样接收消息时也不会被打断,消息在邮箱中等待被处理.直到actor调用receive方法(每次处理一条消息).receive调用预先定义好的偏函数.

actor处理传给receive方法的片函数中的某个样本匹配的消息.邮箱中的每一个消息都要先调用传入片函数的isDefinedAt方法来决定是否与某个样本匹配.

####native thread vs actor

1. 如果使用的显示定义的actor则无需关心它和线程的对应关系是怎样的.Actor子系统会管理一个或多个原生线程供自己使用.
2. 如果反过来把原声线程当做actor使用则不能调用Thread.current这种方法(因为并不具备),需要用Actor.self来把当前线程作为actor来查看.
3. 一般都常用在交互式的repl环境中.

####通过重用(共享)线程获得更好的性能

actor是构建在普通的java线程之上的.**每个actor都会有一个独立的线程**.为了节约线程,scala提供了react(也带一个偏函数),react在找到并处理消息后并不返回.返回类型是Nothing.
考虑如果程序包含大量的actor,为每一个actor单独创建进程代价会很大,而且actor的大部分时间都是用于等待消息.**与其让每个actor在单独的线程中阻塞,不如用一个线程来执行多个actor的消息处理函数**.

> 由于react方法不需要返回,其实现不需要保留当前线程的调用栈.actor库可以在下一个被唤醒的线程中重用当前的线程.如果所有的actor都是react,理论上只需要一个线程就能够满足程序的全部actor的需要(如果计算机有多个core,actor子系统将在可能的情况下使用足够多的线程来充分利用所有core).

使用react避免频繁的线程切换开销.接收消息的消息处理器需要同时处理消息并执行actor所有余下的工作.

Actor线程模型可以这样理解：所有Actor共享一个线程池，总的线程个数可以配置，也可以根据CPU个数决定；当一个Actor启动之后，Scala分配一个线程给它使用，如果使用receive模型，这个线程就一直为该Actor所有，如果使用react模型，Scala执行完react方法后抛出异常，则该线程就可以被其它Actor使用。

``` scala
object LearnActor {

  def main(args: Array[String]): Unit = {

  }
}

object NameResolver extends Actor {
  override def act(): Unit = {
    react {
      case Some(host: String, actor: Actor) =>
        actor ! getIp(host)
        act()
      case "EXIT" =>
        println("Name resolver exiting.")
      case msg =>
        println("Unhandled message: " + msg)
        act()
    }
  }

  def getIp(name: String): Option[String] = {
    import java.net.{InetAddress, UnknownHostException}
    try {
      Some(InetAddress.getByName(name).toString)
    } catch {
      case _: UnknownHostException => None
    }

  }
}

// 使用loop的另一个版本

def act() {
  loop {
    react {
      case (name: String, actor: Actor) =>
        actor ! getIp(name)
      case msg =>
        println("Unhandled message: " + msg)
    }
  }
}
```

1. 不使用死循环.再执行完相应的消息处理操作后再次调用act()以此来实现持续的接收消息.
2. 如果不在case分支的最后调用act()方法,则可以把这个case分支认为是退出分支.
3. 也可以用**scala.actors.Actor.loop**函数重复执行一个代码段.这个actor将会循环相应消息,forever~~~~

eventloop方法也可以制作出一个无穷循环套react的简化版,不过要求偏函数不会再次调用react.

``` scala
def act()
  eventloop {
    case Withdraw(amount) => println("Withdrawing " + amount)
  }
```

###看看城里人怎么写actor

* Actor不应该阻塞,否则在actor阻塞时,另一个actor可能会对它发起一个它能够处理的请求.如果actor在首个请求时阻塞了,那么它将不会留意到第二个请求.最坏的情况是多个actor都在等待另一个阻塞的actor的响应.
* react的工作原理,返回Nothing表示永远不会正常返回的函数,总是以异常的方式完成.
  * 当调用一个actor的start时,start方法会保证最终有某个线程来调用那个actor的act方法.
  * 如果act当中调用了react,则react方法会在actor的邮箱中查找传递给偏函数的能够处理的消息.如果找到可处理消息,react会在卫莱某个时间处理该消息并在完成后抛出异常.
  * 如果找不到符合偏函数(isDefinedAt方法)的消息,会把actor置于"冷存储"状态,再下次邮箱收到更多消息时重新激活,并抛出异常.随之act方法也结束,调用act的线程会catch这个异常,忽略这个actor,并继续处理其他事务.
* 只通过消息与actor通信,如果一个Actor把指向自己的引用作为消息的一部分发给了另外一个Actor,那么强烈要求在下游Actor中除了通过!向其发送消息外,不通过引用主动调用任何方法.
* 通过创建一个"拥有"可变映射的actor并定义一系列允许其他actor访问这个映射的消息.来在多个actor中共享一份可变映射.这是比较actor style的做法,另外也可以通过消息向多个actor传递一个线程安全的映射(ConcurrentHashMap).让这些actor直接使用那个映射.
* 优先选择发送不可变的消息,每个actor的act方法都工作在单线程环境中,可以在act方法中使用任何非同步,可变的对象.但是用于在actor间发送消息的对象中的数据由多个actor"共享".

###同步消息和Future
####三种发送消息的方式:
1. ! 直接发送,异步请求.
2. !? 发送同步请求并等待处理结果的返回.
3. !! 发送请求并接收一个future(一个将在结果可用时产出结果的对象)

- - -
#**actor模式不建议使用同步请求,可能会造成死锁或大幅度降低应用性能**

1. 上游发送方会被自动的赋值给sender变量.
2. 也可以使用reply(some message to deliver)来回传消息.
3. 通过对future的函数调用表示法可以取回结果.**该操作可能会被阻塞,直到回复被发送**.(可以用isSet方法来检测结果是否已经可用.)
4. future的优势在于可以把结果一直传递下去,直到真正需要取出返回值的时候.

---

## <span id="option"> Scala中处理null的中级解决方案--Option</span>

### Scala支持更高级的null
> Scala在标准库里提供了scala.Option类,Option拥有两个子类,分别是None和Some.分别代表容器(Option)中是否包含元素.

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


- - - 

## <span id="classtype"> Scala类型系统</span>
###java的类型系统发展与scala类型比较

1. java在没有引入泛型之前(jdk1.5以前),每一种类型都对应了独立的class(一一映射).通过获取对象的class对象就可以准确的做出类型判断.
2. 引入泛型后,jvm在运行时进行了类型擦除,导致List<String>和List<Long>的class都是Class<List>.同时,java里增加了Type接口来表达更泛的类型.
3. scala自己定义了一个scala.reflect.runtime.universe.Type(2.10后).

在scala中获取类型信息是很方便的.同样要获取class信息也很方便.
``` Scala
import scala.reflect.runtime.universe._
class A
typeOf[A]
// res1:reflect.runtime.universe.Type = A

classOf[A]
//res2:Class[A] = class A

scala> val a  = new A
a: A = A@1d9e436a

scala> a.getClass
res23: Class[_ <: A] = class A

scala> trait T
defined trait T

scala> classOf[T]
res24: Class[T] = interface T

scala> typeOf[T]
res25: reflect.runtime.universe.Type = T

```

