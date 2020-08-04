# Scala Trait+Match+Case class+偏函数

## trait	特性

1.	概念理解
Scala Trait(特征) 相当于 Java 的接口，实际上它比接口还功能强大。
与接口不同的是，它还可以定义属性和方法的实现。
一般情况下Scala的类可以继承多个Trait，从结果来看就是实现了多重继承。Trait(特征) 定义的方式与类类似，但它使用的关键字是 trait。
2.	举例：trait中带属性带方法实现
注意：
 继承的多个trait中如果有同名的方法和属性，必须要在类中使用“override”重新定义。
 trait中不可以传参数

```scala
package icu.shaoyayu.scala

/**
 * @author shaoyayu
 * @date 2020/8/1 8:44
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 *        trait
 */
trait Read{
  def read(name:String)={
    println(s"$name reading now...")
  }
}
trait Write{
  def write(name:String)={
    println(s"$name writing...")
  }
}

/**
 * 多继承的时候，第一个使用extends 第二个开始用with连接
 */
class FileAction() extends Read with Write {

}

object Lesson9 {
  def main(args: Array[String]): Unit = {
    val action = new FileAction()
    action.read("张三")
    action.write("李四")


  }
}
```



```scala
package icu.shaoyayu.scala

/**
 * @author shaoyayu
 * @date 2020/8/1 9:05
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 */
trait IsEqual{
  def isEqual(o:Any):Boolean  //可以定义没有实现的方法，有继承的类去实现
  def isNoEqual(o:Any):Boolean={  //可以定义实现的方法
    !isEqual(o)
  }
}

class Circle(x:Int,y:Int,radius:Double) extends IsEqual {
  val coordinateX = x;
  val coordinateY = y;
  val circleRadius = radius;
  override def isEqual(o: Any): Boolean = {
    o.isInstanceOf[Circle]&&o.asInstanceOf[Circle].coordinateX==this.coordinateX&&o.asInstanceOf[Circle].coordinateY==this.coordinateY&&o.asInstanceOf[Circle].circleRadius==this.circleRadius
  }
}

object Lesson10 {
  def main(args: Array[String]): Unit = {
    val c1 = new Circle(1,2,5)
    val c2 = new Circle(1,2,5)
    val c3 = new Circle(2,2,5)
    println(c1.isEqual(c2)) //true
    println(c1.isNoEqual(c2)) //false
    println(c1.isEqual(c3)) //false
    println(c1.isNoEqual(c3)) //true

  }

}
```

## 模式匹配match

1.	概念理解：
Scala 提供了强大的模式匹配机制，应用也非常广泛。
一个模式匹配包含了一系列备选项，每个都开始于关键字 case。
每个备选项都包含了一个模式及一到多个表达式。箭头符号 => 隔开了模式和表达式。
2.	代码及注意点
 模式匹配不仅可以匹配值还可以匹配类型
 从上到下顺序匹配，如果匹配到则不再往下匹配
 都匹配不上时，会匹配到case _ ,相当于default
 match 的最外面的”{ }”可以去掉看成一个语句



```scala
object Lesson_Match {
  def main(args: Array[String]): Unit = {
    val tuple = Tuple6(1,2,3f,4,"abc",55d)
    val tupleIterator = tuple.productIterator
    while(tupleIterator.hasNext){
      matchTest(tupleIterator.next())
    }
    
  }
  /**
   * 注意点：
   * 1.模式匹配不仅可以匹配值，还可以匹配类型
   * 2.模式匹配中，如果匹配到对应的类型或值，就不再继续往下匹配
   * 3.模式匹配中，都匹配不上时，会匹配到 case _ ，相当于default
   */
  def matchTest(x:Any) ={
    x match {
      case x:Int=> println("type is Int")
      case 1 => println("result is 1")
      case 2 => println("result is 2")
      case 3=> println("result is 3")
      case 4 => println("result is 4")
      case x:String => println("type is String")
//      case x :Double => println("type is Double")
      case _ => println("no match")
    }
  }
  
}
```

## 偏函数

如果一个方法中没有match 只有case，这个函数可以定义成PartialFunction偏函数。偏函数定义时，不能使用括号传参，默认定义PartialFunction中传入一个值，匹配上了对应的case,返回一个值。

```scala
/**
  * 一个函数中只有case 没有match ，可以定义成PartailFunction 偏函数
  */
object Lesson_PartialFunction {
  def MyTest : PartialFunction[String,String] = {
    case "scala" =>{"scala"}
    case "hello"=>{"hello"}
    case _=> {"no  match ..."}
  }
  def main(args: Array[String]): Unit = {
      println(MyTest("scala"))
  }
}

```

## 样例类(case classes)

1.	概念理解
使用了case关键字的类定义就是样例类(case classes)，样例类是种特殊的类。实现了类构造参数的getter方法（构造参数默认被声明为val），当构造参数是声明为var类型的，它将帮你实现setter和getter方法。
 样例类默认帮你实现了toString,equals，copy和hashCode等方法。
 样例类可以new, 也可以不用new
2.	例子：结合模式匹配的代码

```scala
case class Person1(name:String,age:Int)

object Lesson_CaseClass {
  def main(args: Array[String]): Unit = {
    val p1 = new Person1("zhangsan",10)
    val p2 = Person1("lisi",20)
    val p3 = Person1("wangwu",30)
    
    val list = List(p1,p2,p3)
    list.foreach { x => {
      x match {
        case Person1("zhangsan",10) => println("zhangsan")
        case Person1("lisi",20) => println("lisi")
        case _ => println("no match")
      }
    } }
    
  }
}
```







