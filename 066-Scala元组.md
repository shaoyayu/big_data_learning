# Scala 元组

1. 元组定义

与列表一样，与列表不同的是元组可以包含不同类型的元素。元组的值是通过将单个的值包含在圆括号中构成的。

2. 创建元组与取值

- val tuple = new Tuple（1） 可以使用new
- val tuple2 = Tuple（1,2） 可以不使用new，也可以直接写成val tuple3 =（1,2,3） 
-  取值用”._XX” 可以获取元组中的值

> 注意：tuple最多支持22个参数

```scala
package icu.shaoyayu.scala

/**
 * @author shaoyayu
 * @date 2020/7/30 17:58
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 *        元组
 */
object Lesson8 {
  def main(args: Array[String]): Unit = {
    //一个元组
    val tuple1: Tuple1[String] = Tuple1[String]("s")
    val tuple2: (String, Int) = Tuple2[String, Int]("a", 123)
    val tuple3: (String, Boolean, Int) = Tuple3[String, Boolean, Int]("a", false, 3)
    val tuple4: (String, String, Boolean, Int) = Tuple4[String, String, Boolean, Int]("a", "b", true, 10)
    val tuple6: (Int, Int, Int, String, String, Boolean) = (1, 2, 3, "a", "b", true)
    val tuples = (1,2,3,4,5,"a",false,56);  //这样的定义也是可以的
    //遍历的第一步是获取迭代器
    val iterator: Iterator[Any] = tuples.productIterator
    //第二不在使用迭代器进行遍历
    iterator.foreach(println)
    //Tuple2独有的反转方法
    println(tuple2.swap)  //(123,a)
  }
}
```