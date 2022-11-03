
## 伴生对象

### 单例对象
scala 创建单例对象非常简单， 直接 object 关键字修饰的类就是单例对象  
单例对象名就是类名，直接拿着类名去调用其属性或方法就行，很像静态static

### 伴生对象
将单例关联到一个类。单例的名字和类的名字一致，这样的object 就是伴生对象，这样的类就是伴生类  
```scala
class Market private (val color: String) {           // 私有构造函数，外部不可调用
  println("Create Market")
}

object Market {
  private val markets = mutable.Map(
    "red"  -> new Market("red"),
    "blue" -> new Market("blue")
  )
  
  def getMarket(color: String): Market = {
    markets.getOrElseUpdate(color, new Market(color))
  }
}
```

伴生对象和伴生类没有边界，他们可以互相访问彼此  
这样实现的话，伴生类不能被随意构造，只能通过伴生对象来搞

### 静态static
scala 中没有 static  
如何实现？  
>object 单例对象其实就可以认为是静态方式的调用  

object 修饰的话都是静态的，想像java 一样, 类中既有静态又有非静态，如何实现?
>伴生对象的方式，将静态字段放入伴生对象中，其余的放入class 中

## Scala 基础类型
scala 可以使用java的所有类型，但也有自己的类型  
### Any
所有类型的超类型，有两个子类 引用类型AnyRef 值类型AnyVal, AnyRef 对应java 的 Object类型，AnyVal 为基础类型

### Nothing
Any 为所有类型的父类，而 Nothing 为所有类型的子类  
奇怪，要所有类型的子类是什么操作？ 
>很强大，说是可以作为 Exception 的返回值，用于类型推断的

例子  
```scala
def someOp(number: Int) =
  if (number < 10)
    number * 2
  else
    throw new RuntimeException("Invalid argument")
```
此处类型该如何推断? Any吧范围太大了，等于没推断，此处应该被推断为Int的，但是Exception.   
解释不清
### Null
Null 是所有 AnyRef 的子类，只有一个实例 null, 处于scala 类型的次底端（Nothing 是他的子类）

### Nil
Nil就是一个空的 `List[Nothing]`，即一个可以封装任何类型元素但又没有元素的容器。  
这里就利用了上述Nothing可以作为异常类型的特性。从某种意义上讲，方法抛出异常表示没有按照既定的返回值类型进行返回，所以Nothing的语义正合适。

### Option
有个人 Joshua Bloch 在 Effective Java 中有这样的一个合理建议，返回空集合而不是 null  
当然，如果遵循这个建议我们就不必忍受 NullPointerException  
但是，java 中大量返回了null, 如果你不检查null pointer, 也不容易被发现，这样就很容易出现 NullPointerException  
Ok, scala Option就出现了，他在编译是就强制你进行检查  
Some 和 None 是 Option 子类  

Scala中的Option与Java 8引入的java.util.Optional语义相同

### None
None的本质就是 `Option[Nothing]`

### Unit
同 Void

### Either
一个函数的调用结果可能存在也可能不存在时, 用 Option  
当一个函数想要返回两种不同类型的值之一， 那就  Either  

假设一个 `compute()` 方法，他只对正数有效，对于无效的输入，我们可以优雅的返回错误消息，而不是抛出异常。  
Either类型有两种值: 左值(通常被认为是错误的) 和 右值(通常被认为是符合预期的)  
```scala
def compute(input: Int) = {
  if (input > 0) 
    Right(math.sqrt(input))                   // Right 和 Left 都是单例对象
  else
    Left("Error computing, invalid input")
}
```
当接受到 Either 类型的值时， 使用模式匹配处理
```scala
def display(result: Either[String, Double]): Unit = {
  result match {
    case Right(value) => println(value)
    case Left(err) => println(err)
  }
}
```
