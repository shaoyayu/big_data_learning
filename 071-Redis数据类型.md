# Redis 数据类型

Redis不是*简单的*键值存储，它实际上是一个*数据结构服务器*，支持不同类型的值。这意味着在传统键值存储中，您将字符串键与字符串值相关联，而在Redis中，该值不仅限于简单的字符串，还可以容纳更复杂的数据结构。以下是Redis支持的所有数据结构的列表，本教程将分别进行介绍：

- 二进制安全字符串。
- 列表：根据插入顺序排序的字符串元素的集合。它们基本上是*链表*。
- 集：唯一，未排序的字符串元素的集合。
- 与Sets相似的有序sets，但每个字符串元素都与一个称为*score*的浮点值相关联。元素总是按它们的分数排序，因此与Sets不同，可以检索一系列元素（例如，您可能会问：给我前10名或后10名）。
- 哈希，是由与值关联的字段组成的映射。字段和值都是字符串。这与Ruby或Python哈希非常相似。
- 位数组（或简称为位图）：可以使用特殊命令像位数组一样处理字符串值：您可以设置和清除单个位，计数所有设置为1的位，找到第一个设置或未设置的位，等等。
- HyperLogLogs：这是一个概率数据结构，用于估计集合的基数。别害怕，它比看起来更简单...请参阅本教程的HyperLogLog部分。
- 流：提供抽象日志数据类型的类地图项的仅追加集合。在“ [Redis流简介”中对](https://redis.io/topics/streams-intro)它们进行了深入 [介绍](https://redis.io/topics/streams-intro)。

从[命令参考中](https://redis.io/commands)掌握这些数据类型的工作方式以及使用什么来解决给定问题并不总是那么容易，因此，本文档是有关Redis数据类型及其最常见模式的速成课程。

对于所有示例，我们将使用该`redis-cli`实用程序（一个简单但方便的命令行实用程序）对Redis服务器发出命令。

## Redis键

Redis密钥是二进制安全的，这意味着您可以使用任何二进制序列作为密钥，从“ foo”之类的字符串到JPEG文件的内容。空字符串也是有效的键。

有关密钥的其他一些规则：

- 太长的键不是一个好主意。例如，1024字节的密钥不仅在内存方面是一个坏主意，而且因为在数据集中查找密钥可能需要进行一些代价高昂的密钥比较。即使当手头的任务是匹配一个大值的存在时，对它进行散列（例如使用SHA1）也是一个更好的主意，尤其是从内存和带宽的角度来看。
- 非常短的键通常不是一个好主意。如果您可以改写“ user：1000：followers”，那么将“ u1000flw”作为密钥写的毫无意义。与键对象本身和值对象使用的空间相比，后者更具可读性，并且添加的空间较小。虽然短键显然会消耗更少的内存，但是您的工作是找到合适的平衡。
- 尝试坚持一个模式。例如，“ object-type：id”是一个好主意，例如“ user：1000”。点或破折号通常用于多字字段，例如“ comment：1234：reply.to”或“ comment：1234：reply-to”中。
- 允许的最大密钥大小为512 MB。



## String 类型

```shell
127.0.0.1:6379> help @string

  APPEND key value
  summary: Append a value to a key
  since: 2.0.0

  BITCOUNT key [start] [end]
  summary: Count set bits in a string
  since: 2.6.0

  BITOP operation destkey key [key ...]
  summary: Perform bitwise operations between strings
  since: 2.6.0

  BITPOS key bit [start] [end]
  summary: Find first bit set or clear in a string
  since: 2.8.7

  DECR key
  summary: Decrement the integer value of a key by one
  since: 1.0.0

  DECRBY key decrement
  summary: Decrement the integer value of a key by the given number
  since: 1.0.0

  GET key
  summary: Get the value of a key
  since: 1.0.0

  GETBIT key offset
  summary: Returns the bit value at offset in the string value stored at key
  since: 2.2.0

  GETRANGE key start end
  summary: Get a substring of the string stored at a key
  since: 2.4.0

  GETSET key value
  summary: Set the string value of a key and return its old value
  since: 1.0.0

  INCR key
  summary: Increment the integer value of a key by one
  since: 1.0.0

  INCRBY key increment
  summary: Increment the integer value of a key by the given amount
  since: 1.0.0

  INCRBYFLOAT key increment
  summary: Increment the float value of a key by the given amount
  since: 2.6.0

  MGET key [key ...]
  summary: Get the values of all the given keys
  since: 1.0.0

  MSET key value [key value ...]
  summary: Set multiple keys to multiple values
  since: 1.0.1

  MSETNX key value [key value ...]
  summary: Set multiple keys to multiple values, only if none of the keys exist
  since: 1.0.1

  PSETEX key milliseconds value
  summary: Set the value and expiration in milliseconds of a key
  since: 2.6.0

  SET key value [EX seconds] [PX milliseconds] [NX|XX]
  summary: Set the string value of a key
  since: 1.0.0

  SETBIT key offset value
  summary: Sets or clears the bit at offset in the string value stored at key
  since: 2.2.0

  SETEX key seconds value
  summary: Set the value and expiration of a key
  since: 2.0.0

  SETNX key value
  summary: Set the value of a key, only if the key does not exist
  since: 1.0.0

  SETRANGE key offset value
  summary: Overwrite part of a string at key starting at the specified offset
  since: 2.2.0

  STRLEN key
  summary: Get the length of the value stored in a key
  since: 2.2.0
```

