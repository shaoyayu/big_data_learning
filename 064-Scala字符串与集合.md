# Scala字符串与集合

```scala
package icu.shaoyayu.scala

import scala.collection.mutable.{ArrayBuffer, ListBuffer}

/**
 * @author shaoyayu
 * @date 2020/7/25 16:26
 * @E_Mail
 * @Version 1.0.0
 * @readme ：https://www.scala-lang.org/api/2.11.12/
 *        https://www.scala-lang.org/api/2.11.12/#scala.collection.immutable.List
 *        字符串与集合
 */
object Lesson5 {

  def main(args: Array[String]): Unit = {
    // type String = java.lang.String 所以说，java里面String有的方法，在Scala里面的String也是有的
    val str1 = "abc"
    val str2 = "ABC"
    println(str1.equals(str2))
    println(str1.equalsIgnoreCase(str2))  //忽略大小写的比较

    println(str1.indexOf(98)) //可以传递一个阿斯克码的位置，返回字符第一个对于的位置，没有的话，返回一个 -1

    //创建一个集和
    val arr = Array[String]("a","b","c","d")
    arr.foreach(println)  //这是在Scala里面的一种遍历方法
    for (str<-arr){ //遍历
      println(str)
    }

    val intArr = new Array[Int](5) //[Int] 代表的是集和里面的泛型，5代表的是集和的大小 Int类型的初始值是0
    intArr(0) = 1
    intArr(1) = 2
    intArr(2) = 3
    intArr.foreach(println)

    //创建一个二维数组
    val twoDimensionalArray = new Array[Array[Int]](3)
    twoDimensionalArray(0) = Array(0,1,2,3)
    twoDimensionalArray(1) = Array(0,1,2)
    twoDimensionalArray(2) = Array(0,1)
    println("==================================")
    for (oneDimensionalArray<- twoDimensionalArray){
      oneDimensionalArray.foreach(println)
    }
    println("==================================")
    twoDimensionalArray.foreach(array=>{array.foreach(println)})
    println("==================================")
    for (one<-twoDimensionalArray;v<-one){
      println(v)
    }

    println("==================================")
    val arr1 = Array[String]("1","2","3","4","5")
    val arr2 = Array[String]("a","b","c","d","e")
    val combination = Array.concat(arr1, arr2)
    combination.foreach(println)



    println("==================================")
    val klc = Array.fill(5)("hello")  //初始化5个Hello的元素到集和中
    klc.foreach(println)

    println("==================================")
    //可变集和
    val arrayBuffer = ArrayBuffer[Int](1, 2, 3)
    arrayBuffer.+=(4) //把元素追加到集和后面
    arrayBuffer.+=:(0)  //把元素追加到前面
    arrayBuffer.append(5,6) //添加到集和后面
    arrayBuffer.insert(2,7) //替换每个2下标的值
    arrayBuffer.foreach(println)

    /*========================================List====================================================*/

    println("==================================")
    val ints = List[Int](12, 13)  //创建的时候需要指定类型
    ints.foreach(println)
    println("==================================")

    val strList = List[String]("hello scala", "hello java", "hello spark")
    val splitStr:List[Array[String]] = strList.map(s => { //切分里面的每一个元素，把相对于结果返回一个新的Array集和组成的List
      s.split(" ")
    })
    splitStr.foreach(s=>{
      println("---")
      s.foreach(println)
    })
    /*
    ---
    hello
    scala
    ---
    hello
    java
    ---
    hello
     */

    /**
     * flatMap是在Map处理结果上的到的Array集和把每一个Array里面的元素 填写到一个新的List里面返回
     */
    val list:List[String] = strList.flatMap(s => {
      s.split(" ")
    })
    println("==================================")
    list.foreach(println)
    /*
    hello
    scala
    hello
    java
    hello
    spark
     */
    println("==================================")
    val scalaStr: List[String] = List[String]("hello java", "hello scala", "hello spark", "hello hadoop")
    val comeBack: List[String] = scalaStr.filter(s => {   //把满足条件的值进行封装到一个行的集和里面
      s.equals("hello java")
    })
    comeBack.foreach(println) //hello java

    println("==================================")

    val numberOfEligible: Int = scalaStr.count(s => {  //统计满足条件的数目
      s.length > 10
    })
    println(numberOfEligible) //3个


    /**
     * 可变的list
     */
    println("==================================")

    val instance: ListBuffer[Int] = ListBuffer[Int](1, 2, 3)
    instance.append(4,5,6)
    instance.+=:(0)
    instance.foreach(println)
  }

}
```



## 字符串常用方法

String 方法
	
char charAt(int index)
返回指定位置的字符  从0开始
	
int compareTo(Object o)
比较字符串与对象
	
int compareTo(String anotherString)
按字典顺序比较两个字符串
	
int compareToIgnoreCase(String str)
按字典顺序比较两个字符串，不考虑大小写
	
String concat(String str)
将指定字符串连接到此字符串的结尾
	
boolean contentEquals(StringBuffer sb)
将此字符串与指定的 StringBuffer 比较。
	
static String copyValueOf(char[] data)
返回指定数组中表示该字符序列的 String
	
static String copyValueOf(char[] data, int offset, int count)
返回指定数组中表示该字符序列的 String
	
boolean endsWith(String suffix)
测试此字符串是否以指定的后缀结束
	
boolean equals(Object anObject)
将此字符串与指定的对象比较
	
boolean equalsIgnoreCase(String anotherString)
将此 String 与另一个 String 比较，不考虑大小写
	
byte getBytes()
使用平台的默认字符集将此 String 编码为 byte 序列，并将结果存储到一个新的 byte 数组中
	
byte[] getBytes(String charsetName
使用指定的字符集将此 String 编码为 byte 序列，并将结果存储到一个新的 byte 数组中
	
void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)
将字符从此字符串复制到目标字符数组
	
int hashCode()
返回此字符串的哈希码
16	
int indexOf(int ch)
返回指定字符在此字符串中第一次出现处的索引（输入的是ascii码值）
	
int indexOf(int ch, int fromIndex)
返返回在此字符串中第一次出现指定字符处的索引，从指定的索引开始搜索
	
int indexOf(String str)
返回指定子字符串在此字符串中第一次出现处的索引
	
int indexOf(String str, int fromIndex)
返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
	
String intern()
返回字符串对象的规范化表示形式
	
int lastIndexOf(int ch)
返回指定字符在此字符串中最后一次出现处的索引
	
int lastIndexOf(int ch, int fromIndex)
返回指定字符在此字符串中最后一次出现处的索引，从指定的索引处开始进行反向搜索
	
int lastIndexOf(String str)
返回指定子字符串在此字符串中最右边出现处的索引
	
int lastIndexOf(String str, int fromIndex)
返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索
	
int length()
返回此字符串的长度
	
boolean matches(String regex)
告知此字符串是否匹配给定的正则表达式
	
boolean regionMatches(boolean ignoreCase, int toffset, String other, int ooffset, int len)
测试两个字符串区域是否相等
28	
boolean regionMatches(int toffset, String other, int ooffset, int len)
测试两个字符串区域是否相等
	
String replace(char oldChar, char newChar)
返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所有 oldChar 得到的
	
String replaceAll(String regex, String replacement
使用给定的 replacement 替换此字符串所有匹配给定的正则表达式的子字符串
	
String replaceFirst(String regex, String replacement)
使用给定的 replacement 替换此字符串匹配给定的正则表达式的第一个子字符串
	
String[] split(String regex)
根据给定正则表达式的匹配拆分此字符串
	
String[] split(String regex, int limit)
根据匹配给定的正则表达式来拆分此字符串
	
boolean startsWith(String prefix)
测试此字符串是否以指定的前缀开始
	
boolean startsWith(String prefix, int toffset)
测试此字符串从指定索引开始的子字符串是否以指定前缀开始。
	
CharSequence subSequence(int beginIndex, int endIndex)
返回一个新的字符序列，它是此序列的一个子序列
	
String substring(int beginIndex)
返回一个新的字符串，它是此字符串的一个子字符串
	
String substring(int beginIndex, int endIndex)
返回一个新字符串，它是此字符串的一个子字符串
	
char[] toCharArray()
将此字符串转换为一个新的字符数组
	
String toLowerCase()
使用默认语言环境的规则将此 String 中的所有字符都转换为小写
	
String toLowerCase(Locale locale)
使用给定 Locale 的规则将此 String 中的所有字符都转换为小写
	
String toString()
返回此对象本身（它已经是一个字符串！）
	
String toUpperCase()
使用默认语言环境的规则将此 String 中的所有字符都转换为大写
	
String toUpperCase(Locale locale)
使用给定 Locale 的规则将此 String 中的所有字符都转换为大写
	
String trim()
删除指定字符串的首尾空白符
	
static String valueOf(primitive data type x)
返回指定类型参数的字符串表示形式



## 常用的数组方法

序号	方法和描述
1	
def apply( x: T, xs: T* ): Array[T]
创建指定对象 T 的数组, T 的值可以是 Unit, Double, Float, Long, Int, Char, Short, Byte, Boolean。
2	
def concat[T]( xss: Array[T]* ): Array[T]
合并数组
3	
def copy( src: AnyRef, srcPos: Int, dest: AnyRef, destPos: Int, length: Int ): Unit
复制一个数组到另一个数组上。相等于 Java's System.arraycopy(src, srcPos, dest, destPos, length)。
4	
def empty[T]: Array[T]
返回长度为 0 的数组
5	
def iterate[T]( start: T, len: Int )( f: (T) => T ): Array[T]
返回指定长度数组，每个数组元素为指定函数的返回值。
以上实例数组初始值为 0，长度为 3，计算函数为a=>a+1：
scala> Array.iterate(0,3)(a=>a+1)
res1: Array[Int] = Array(0, 1, 2)
6	
def fill[T]( n: Int )(elem: => T): Array[T]
返回数组，长度为第一个参数指定，同时每个元素使用第二个参数进行填充。
7	
def fill[T]( n1: Int, n2: Int )( elem: => T ): Array[Array[T]]
返回二数组，长度为第一个参数指定，同时每个元素使用第二个参数进行填充。
8	
def ofDim[T]( n1: Int ): Array[T]
创建指定长度的数组
9	
def ofDim[T]( n1: Int, n2: Int ): Array[Array[T]]
创建二维数组
10	
def ofDim[T]( n1: Int, n2: Int, n3: Int ): Array[Array[Array[T]]]
创建三维数组
11	
def range( start: Int, end: Int, step: Int ): Array[Int]
创建指定区间内的数组，step 为每个元素间的步长
12	
def range( start: Int, end: Int ): Array[Int]
创建指定区间内的数组
13	
def tabulate[T]( n: Int )(f: (Int)=> T): Array[T]
返回指定长度数组，每个数组元素为指定函数的返回值，默认从 0 开始。
以上实例返回 3 个元素：
scala> Array.tabulate(3)(a => a + 5)
res0: Array[Int] = Array(5, 6, 7)
14	
def tabulate[T]( n1: Int, n2: Int )( f: (Int, Int ) => T): Array[Array[T]]
返回指定长度的二维数组，每个数组元素为指定函数的返回值，默认从 0 开始。





## 常用的List方法

1	def +(elem: A): List[A]
前置一个元素列表
2	def ::(x: A): List[A]
在这个列表的开头添加的元素。
3	def :::(prefix: List[A]): List[A]
增加了一个给定列表中该列表前面的元素。
4	def ::(x: A): List[A]
增加了一个元素x在列表的开头
5	def addString(b: StringBuilder): StringBuilder
追加列表的一个字符串生成器的所有元素。
6	def addString(b: StringBuilder, sep: String): StringBuilder
追加列表的使用分隔字符串一个字符串生成器的所有元素。
7	def apply(n: Int): A
选择通过其在列表中索引的元素
8	def contains(elem: Any): Boolean
测试该列表中是否包含一个给定值作为元素。
9	def copyToArray(xs: Array[A], start: Int, len: Int): Unit
列表的副本元件阵列。填充给定的数组xs与此列表中最多len个元素，在位置开始。
10	def distinct: List[A]
建立从列表中没有任何重复的元素的新列表。
11	def drop(n: Int): List[A]
返回除了第n个的所有元素。
12	def dropRight(n: Int): List[A]
返回除了最后的n个的元素
13	def dropWhile(p: (A) => Boolean): List[A]
丢弃满足谓词的元素最长前缀。
14	def endsWith[B](that: Seq[B]): Boolean
测试列表是否使用给定序列结束。
15	def equals(that: Any): Boolean
equals方法的任意序列。比较该序列到某些其他对象。
16	def exists(p: (A) => Boolean): Boolean
测试谓词是否持有一些列表的元素。
17	def filter(p: (A) => Boolean): List[A]
返回列表满足谓词的所有元素。
18	def forall(p: (A) => Boolean): Boolean
测试谓词是否持有该列表中的所有元素。
19	def foreach(f: (A) => Unit): Unit
应用一个函数f以列表的所有元素。
20	def head: A
选择列表的第一个元素
21	def indexOf(elem: A, from: Int): Int
经过或在某些起始索引查找列表中的一些值第一次出现的索引。
22	def init: List[A]
返回除了最后的所有元素
23	def intersect(that: Seq[A]): List[A]
计算列表和另一序列之间的多重集交集。
24	def isEmpty: Boolean
测试列表是否为空
25	def iterator: Iterator[A]
创建一个新的迭代器中包含的可迭代对象中的所有元素
26	def last: A
返回最后一个元素
27	def lastIndexOf(elem: A, end: Int): Int
之前或在一个给定的最终指数查找的列表中的一些值最后一次出现的索引
28	def length: Int
返回列表的长度
29	def map[B](f: (A) => B): List[B]
通过应用函数以g这个列表中的所有元素构建一个新的集合
30	def max: A
查找最大的元素
31	def min: A
查找最小元素
32	def mkString: String
显示列表的字符串中的所有元素
33	def mkString(sep: String): String
显示的列表中的字符串中使用分隔串的所有元素
34	def reverse: List[A]
返回新列表，在相反的顺序元素
35	def sorted[B >: A]: List[A]
根据排序对列表进行排序
36	def startsWith[B](that: Seq[B], offset: Int): Boolean
测试该列表中是否包含给定的索引处的给定的序列
37	def sum: A
概括这个集合的元素
38	def tail: List[A]
返回除了第一的所有元素
39	def take(n: Int): List[A]
返回前n个元素
40	def takeRight(n: Int): List[A]
返回最后n个元素
41	def toArray: Array[A]
列表以一个数组变换
42	def toBuffer[B >: A]: Buffer[B]
列表以一个可变缓冲器转换
43	def toMap[T, U]: Map[T, U]
此列表的映射转换
44	def toSeq: Seq[A]
列表的序列转换
45	def toSet[B >: A]: Set[B]
列表到集合变换
46	def toString(): String
列表转换为字符串