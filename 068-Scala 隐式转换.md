# Scala 隐式转换

隐式转换是在Scala编译器进行类型匹配时，如果找不到合适的类型，那么隐式转换会让编译器在作用范围内自动推导出来合适的类型。

## 1.隐式值与隐式参数

隐式值是指在定义参数时前面加上implicit。隐式参数是指在定义方法时，方法中的部分参数是由implicit修饰【必须使用柯里化的方式，将隐式参数写在后面的括号中】。隐式转换作用就是：当调用方法时，不必手动传入方法中的隐式参数，Scala会自动在作用域范围内寻找隐式值自动传入。
隐式值和隐式参数注意：

- 1). 同类型的参数的隐式值只能在作用域内出现一次，同一个作用域内不能定义多个类型一样的隐式值。
- 2). implicit 关键字必须放在隐式参数定义的开头
- 3). 一个方法只有一个参数是隐式转换参数时，那么可以直接定义implicit关键字修饰的参数，调用时直接创建类型不传入参数即可。
- 4). 一个方法如果有多个参数，要实现部分参数的隐式转换,必须使用柯里化这种方式,隐式关键字出现在后面，只能出现一次

```scala
object Lesson_ImplicitValue {

  def Student(age:Int)(implicit name:String,i:Int)= {
    println( s"student :$name ,age = $age ,score = $i")
  }
  def Teacher(implicit name:String) ={
    println(s"teacher name is = $name")
  }

  def main(args: Array[String]): Unit = {
    implicit val zs = "zhangsan"
    implicit val sr = 100

    Student(18)
    Teacher
  }
}

```

## 2.隐式转换函数

隐式转换函数是使用关键字implicit修饰的方法。当Scala运行时，假设如果A类型变量调用了method()这个方法，发现A类型的变量没有method()方法，而B类型有此method()方法，会在作用域中寻找有没有隐式转换函数将A类型转换成B类型，如果有隐式转换函数，那么A类型就可以调用method()这个方法。

   隐式转换函数注意：隐式转换函数只与函数的参数类型和返回类型有关，与函数名称无关，所以作用域内不能有相同的参数类型和返回类型的不同名称隐式转换函数。

```scala
  def canFly(): Unit ={
    println(s"$name can fly...")
  }
}
class Rabbit(xname:String){
    val name = xname
}
object Lesson_ImplicitFunction {

  implicit def rabbitToAnimal(rabbit:Rabbit):Animal = {
      new Animal(rabbit.name)
  }

  def main(args: Array[String]): Unit = {
    val rabbit = new Rabbit("RABBIT")
    rabbit.canFly()
  }
}
```

## 3.隐式类

使用implicit关键字修饰的类就是隐式类。若一个变量A没有某些方法或者某些变量时，而这个变量A可以调用某些方法或者某些变量时，可以定义一个隐式类，隐式类中定义这些方法或者变量，隐式类中传入A即可。
隐式类注意：

1).隐式类必须定义在类，包对象，伴生对象中。

2).隐式类的构造必须只有一个参数，同一个类，包对象，伴生对象中不能出现同类型构造的隐式类。

```scala
class Rabbit(s:String){
  val name = s
}

object Lesson_ImplicitClass {

  implicit class Animal(rabbit:Rabbit){
    val tp = "Animal"
    def canFly() ={
      println(rabbit.name +" can fly...")
    }
  }

  def main(args: Array[String]): Unit = {
    val rabbit = new Rabbit("rabbit")
    rabbit.canFly()
    println(rabbit.tp)
  }
}
```

