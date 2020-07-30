# Scala里面的Set与Map

## Set

```scala
package icu.shaoyayu.scala


/**
 * @author shaoyayu
 * @date 2020/7/26 11:15
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 *        set介绍
 *
 */
object Lesson6 {
  def main(args: Array[String]): Unit = {

  }

  /**
   * set 是有去重复的功能的 默认的是不可变长度的
   */
  val setList1: Set[Int] = Set[Int](1, 2, 3, 4, 4)
//  setList1.foreach(println) //1 2 3 4
  val setList2: Set[Int] = Set[Int](3, 4, 5, 6)
  //求交集
  val setList3: Set[Int] = setList1.intersect(setList2)
//  setList3.foreach(println) //3 4
//set1在set 2 中的差集
  val setList4: Set[Int] = setList1.diff(setList2)
//  setList4.foreach(println)  // 1 2
  // 求交集的另一种写法
  val setList5: Set[Int] = setList1 & setList2 // 3 4
  val setList6: Set[Int] = setList1 &~ setList2 //1 2
  setList6.foreach(println)



}
```

## Scala Set 常用方法

下表列出了 Scala Set 常用的方法：
序号	方法及描述
1	
def +(elem: A): Set[A]
为集合添加新元素，x并创建一个新的集合，除非元素已存在
2	
def -(elem: A): Set[A]
移除集合中的元素，并创建一个新的集合
3	
def contains(elem: A): Boolean
如果元素在集合中存在，返回 true，否则返回 false。
4	
def &(that: Set[A]): Set[A]
返回两个集合的交集
5	
def &~(that: Set[A]): Set[A]
返回两个集合的差集
6	
def +(elem1: A, elem2: A, elems: A*): Set[A]
通过添加传入指定集合的元素创建一个新的不可变集合
7	
def ++(elems: A): Set[A]
合并两个集合
8	
def -(elem1: A, elem2: A, elems: A*): Set[A]
通过移除传入指定集合的元素创建一个新的不可变集合
9	
def addString(b: StringBuilder): StringBuilder
将不可变集合的所有元素添加到字符串缓冲区
10	
def addString(b: StringBuilder, sep: String): StringBuilder
将不可变集合的所有元素添加到字符串缓冲区，并使用指定的分隔符
11	
def apply(elem: A)
检测集合中是否包含指定元素
12	
def count(p: (A) => Boolean): Int
计算满足指定条件的集合元素个数
13	
def copyToArray(xs: Array[A], start: Int, len: Int): Unit
复制不可变集合元素到数组
14	
def diff(that: Set[A]): Set[A]
比较两个集合的差集
15	
def drop(n: Int): Set[A]]
返回丢弃前n个元素新集合
16	
def dropRight(n: Int): Set[A]
返回丢弃最后n个元素新集合
17	
def dropWhile(p: (A) => Boolean): Set[A]
从左向右丢弃元素，直到条件p不成立
18	
def equals(that: Any): Boolean
equals 方法可用于任意序列。用于比较系列是否相等。
19	
def exists(p: (A) => Boolean): Boolean
判断不可变集合中指定条件的元素是否存在。
20	
def filter(p: (A) => Boolean): Set[A]
输出符合指定条件的所有不可变集合元素。
21	
def find(p: (A) => Boolean): Option[A]
查找不可变集合中满足指定条件的第一个元素
22	
def forall(p: (A) => Boolean): Boolean
查找不可变集合中满足指定条件的所有元素
23	
def foreach(f: (A) => Unit): Unit
将函数应用到不可变集合的所有元素
24	
def head: A
获取不可变集合的第一个元素
25	
def init: Set[A]
返回所有元素，除了最后一个
26	
def intersect(that: Set[A]): Set[A]
计算两个集合的交集
27	
def isEmpty: Boolean
判断集合是否为空
28	
def iterator: Iterator[A]
创建一个新的迭代器来迭代元素
29	
def last: A
返回最后一个元素
30	
def map[B](f: (A) => B): immutable.Set[B]
通过给定的方法将所有元素重新计算
31	
def max: A
查找最大元素
32	
def min: A
查找最小元素
33	
def mkString: String
集合所有元素作为字符串显示
34	
def mkString(sep: String): String
使用分隔符将集合所有元素作为字符串显示
35	
def product: A
返回不可变集合中数字元素的积。
36	
def size: Int
返回不可变集合元素的数量
37	
def splitAt(n: Int): (Set[A], Set[A])
把不可变集合拆分为两个容器，第一个由前 n 个元素组成，第二个由剩下的元素组成
38	
def subsetOf(that: Set[A]): Boolean
如果集合A中含有子集B返回 true，否则返回false
39	
def sum: A
返回不可变集合中所有数字元素之和
40	
def tail: Set[A]
返回一个不可变集合中除了第一元素之外的其他元素
41	
def take(n: Int): Set[A]
返回前 n 个元素
42	
def takeRight(n: Int):Set[A]
返回后 n 个元素
43	
def toArray: Array[A]
将集合转换为数组
44	
def toBuffer[B >: A]: Buffer[B]
返回缓冲区，包含了不可变集合的所有元素
45	
def toList: List[A]
返回 List，包含了不可变集合的所有元素
46	
def toMap[T, U]: Map[T, U]
返回 Map，包含了不可变集合的所有元素
47	
def toSeq: Seq[A]
返回 Seq，包含了不可变集合的所有元素
48	
def toString(): String
返回一个字符串，以对象来表示

## Map

```scala
package icu.shaoyayu.scala

import scala.collection.mutable

/**
 * @author shaoyayu
 * @date 2020/7/29 23:08
 * @E_Mail
 * @Version 1.0.0
 * @readme ：
 *        MAP
 */
object Lesson7 {

  def main(args: Array[String]): Unit = {
    //两种在构造的时候传值的方式 k->v或者(k,v)
    val case1: Map[String, Int] = Map[String, Int]("a" -> 2, "b" -> 3, ("c", 4))
    println(case1)  //Map(a -> 2, b -> 3, c -> 4)
    case1.foreach(println)
    /*
    (a,2)
    (b,3)
    (c,4)
     */
    //取到一个Option类型的值，当key不存在的时候，返回Node类型
    val maybeInt: Option[Int] = case1.get("aa")
    println(maybeInt) //Some(2)
    val value0: Int = case1.get("a").get  //当key不存在的时候会出现异常
    println(value0)
    val value1: Any = case1.get("aa").getOrElse("no valve")  //有值的时候返回对于的值，没有的时候返回 no valve
    println(value1)

    //获取所有的key
    val keys: Iterable[String] = case1.keys
    keys.foreach(key=>{
      val value: Int = case1.get(key).get
      println(s"key: $key ,valve: $value")
    })
    //key: a ,valve: 2
    //key: b ,valve: 3
    //key: c ,valve: 4

    case1.values.foreach(println) //获取所有的值

    println("--------------------------------------------------------------")

    val map1: Map[String, Int] = Map[String, Int](("a", 1), ("b", 2), ("c", 3))
    val map2: Map[String, Int] = Map[String, Int](("a", 3), ("b", 4), ("c", 5))

    val map3: Map[String, Int] = map1.++(map2)  //map2往map1里面合并，
    println(map3)

    //可变Map
    val mapBuf = mutable.Map[String, Int]("a"->1)
    mapBuf.put("b",3)
    mapBuf.foreach(println)
  }

}
```





## Scala Map 方法

下表列出了 Scala Map 常用的方法：
序号	方法及描述
1	
def ++(xs: Map[(A, B)]): Map[A, B]
返回一个新的 Map，新的 Map xs 组成
2	
def -(elem1: A, elem2: A, elems: A*): Map[A, B]
返回一个新的 Map, 移除 key 为 elem1, elem2 或其他 elems。
3	
def --(xs: GTO[A]): Map[A, B]
返回一个新的 Map, 移除 xs 对象中对应的 key
4	
def get(key: A): Option[B]
返回指定 key 的值
5	
def iterator: Iterator[(A, B)]
创建新的迭代器，并输出 key/value 对
6	
def addString(b: StringBuilder): StringBuilder
将 Map 中的所有元素附加到StringBuilder，可加入分隔符
7	
def addString(b: StringBuilder, sep: String): StringBuilder
将 Map 中的所有元素附加到StringBuilder，可加入分隔符
8	
def apply(key: A): B
返回指定键的值，如果不存在返回 Map 的默认方法

10	
def clone(): Map[A, B]
从一个 Map 复制到另一个 Map
11	
def contains(key: A): Boolean
如果 Map 中存在指定 key，返回 true，否则返回 false。
12	
def copyToArray(xs: Array[(A, B)]): Unit
复制集合到数组
13	
def count(p: ((A, B)) => Boolean): Int
计算满足指定条件的集合元素数量
14	
def default(key: A): B
定义 Map 的默认值，在 key 不存在时返回。
15	
def drop(n: Int): Map[A, B]
返回丢弃前n个元素新集合
16	
def dropRight(n: Int): Map[A, B]
返回丢弃最后n个元素新集合
17	
def dropWhile(p: ((A, B)) => Boolean): Map[A, B]
从左向右丢弃元素，直到条件p不成立
18	
def empty: Map[A, B]
返回相同类型的空 Map
19	
def equals(that: Any): Boolean
如果两个 Map 相等(key/value 均相等)，返回true，否则返回false
20	
def exists(p: ((A, B)) => Boolean): Boolean
判断集合中指定条件的元素是否存在
21	
def filter(p: ((A, B))=> Boolean): Map[A, B]
返回满足指定条件的所有集合
22	
def filterKeys(p: (A) => Boolean): Map[A, B]
返回符合指定条件的的不可变 Map
23	
def find(p: ((A, B)) => Boolean): Option[(A, B)]
查找集合中满足指定条件的第一个元素
24	
def foreach(f: ((A, B)) => Unit): Unit
将函数应用到集合的所有元素
25	
def init: Map[A, B]
返回所有元素，除了最后一个
26	
def isEmpty: Boolean
检测 Map 是否为空
27	
def keys: Iterable[A]
返回所有的key/p>
28	
def last: (A, B)
返回最后一个元素
29	
def max: (A, B)
查找最大元素
30	
def min: (A, B)
查找最小元素
31	
def mkString: String
集合所有元素作为字符串显示
32	
def product: (A, B)
返回集合中数字元素的积。
33	
def remove(key: A): Option[B]
移除指定 key
34	
def retain(p: (A, B) => Boolean): Map.this.type
如果符合满足条件的返回 true
35	
def size: Int
返回 Map 元素的个数
36	
def sum: (A, B)
返回集合中所有数字元素之和
37	
def tail: Map[A, B]
返回一个集合中除了第一元素之外的其他元素
38	
def take(n: Int): Map[A, B]
返回前 n 个元素
39	
def takeRight(n: Int): Map[A, B]
返回后 n 个元素
40	
def takeWhile(p: ((A, B)) => Boolean): Map[A, B]
返回满足指定条件的元素
41	
def toArray: Array[(A, B)]
集合转数组
42	
def toBuffer[B >: A]: Buffer[B]
返回缓冲区，包含了 Map 的所有元素
43	
def toList: List[A]
返回 List，包含了 Map 的所有元素
44	
def toSeq: Seq[A]
返回 Seq，包含了 Map 的所有元素
45	
def toSet: Set[A]
返回 Set，包含了 Map 的所有元素
46	
def toString(): String
返回字符串对象



