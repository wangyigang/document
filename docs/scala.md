#### scala概述

###### Scala的

###### 	Scala执行流程分析

> .scala文件 经过scalac 编译生成.class文件，执行scala命令运行在jvm虚拟机上

![1548419774939](assets/1548419774939.png)



###### Scala语言输出的三种方式

- 字符串通过+ 号连接
- printf 用法(类似c语言)字符串通过%传值(格式化输出)
- 字符串通过$引用(类似php)

## 变量

##### 变量的介绍

> 基本语法：var | val 变量名 [:变量类型] = 变量值

注意事项：

- scala支持类型推断，类型可以省略
- 类型确定后，就不能修改，说明scala是强数据类型语言
- var 修改的变量可改变，val修饰的变量不可改

##### 数据类型

- scala与java有相同的数据类型，在Scala中数据类型都是对象
- Scala中数据分为两大类：AnyVal(值类型)和AnyRef(引用类型)==>都是对象

![1548421238498](assets/1548421238498.png)

1. Any是所有类的父类
2. Scala中分为两个大的类型AnyVal(值类型) 和AnyRef(引用类型)
3. Scala中有两个特别的类型Null,还有一个Nothing
4. Null类型只有一个实例null,他是bottom class,是AnyRef的子类
5. Nothing是所有类型的子类
6. Scala中，Unit类型比较特殊，这个类型也只有一个实例()

##### 数据类型

###### scala数据类型列表



| 数据类型 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| Byte     | 8位有符号补码整数。数值区间为 -128 到 127                    |
| Short    | 16位有符号补码整数。数值区间为 -32768 到 32767               |
| Int      | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647     |
| Long     | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 |
| Float    | 32 位, IEEE 754标准的单精度浮点数                            |
| Double   | 64 位 IEEE 754标准的双精度浮点数                             |
| Char     | 16位无符号Unicode字符, 区间值为 U+0000 到 U+FFFF             |
| String   | 字符序列                                                     |
| Boolean  | true或false                                                  |
| Unit     | 表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()。 |
| Null     | null                                                         |
| Nothing  | Nothing类型在Scala的类层级的最低端；它是任何其他类型的子类型。 |
| Any      | Any是所有其他类的超类                                        |
| AnyRef   | AnyRef类是Scala里所有引用类(reference class)的基类           |

### 运算符

###### 运算符介绍

- 算术运算符
- 赋值运算符
- 比较运算符(关系运算符)
- 逻辑运算符(与或非)
- 位运算符(~| ^ >> << >>>)

###### 算术运算符

| **运算符** | **运算**   | **范例**   | **结果** |
| ---------- | ---------- | ---------- | -------- |
| **+**      | 正号       | +3         | 3        |
| **-**      | 负号       | b=4; -b    | -4       |
| **+**      | 加         | 5+5        | 10       |
| **-**      | 减         | 6-4        | 2        |
| *****      | 乘         | 3*4        | 12       |
| **/**      | 除         | 5/5        | 1        |
| **%**      | 取模(取余) | 7%5        | 2        |
| **+**      | 字符串相加 | “He”+”llo” | “Hello”  |

scala中没有 ++ ， --

###### 关系运算符

> 关系运算符的结果都是boolean，要么true,要么false

| **运算符** | **运算                                 范例                                         结果** |
| ---------- | ------------------------------------------------------------ |
| **==**     | 相等于                               4==3                                                false |
| **!=**     | 不等于                               4!=3                                                 true |
| **<**      | 小于                                   4<3                                                   false |
| **>**      | 大于                                   4>3                                                   true |
| **<=**     | 小于等于                           4<=3                                                false |
| **>=**     | 大于等于                           4>=3                                                true |

###### 逻辑运算符

| 运算符 | 描述                      | 实例                       |
| ------ | ------------------------- | -------------------------- |
| &&     | 逻辑与 【同样遵守短路与】 | (A && B) 运算结果为 false  |
| \|\|   | 逻辑或  【遵守短路或】    | (A \|\| B) 运算结果为 true |
| !      | 逻辑非                    | !(A && B) 运算结果为 true  |

###### 赋值运算符

> 赋值运算符就将某个运算后的值，赋值给指定的变量

| 运算符 | 描述                                           | 实例                                  |
| ------ | ---------------------------------------------- | ------------------------------------- |
| =      | 简单的赋值运算符，将一个表达式的值赋给一个左值 | C = A + B 将 A + B 表达式结果赋值给 C |
| +=     | 相加后再赋值                                   | C += A 等于 C = C + A                 |
| -=     | 相减后再赋值                                   | C -= A 等于 C = C - A                 |
| *=     | 相乘后再赋值                                   | C *= A 等于 C = C * A                 |
| /=     | 相除后再赋值                                   | C /= A 等于 C = C / A                 |
| %=     | 求余后再赋值                                   | C %= A 等于 C = C % A                 |

###### 位运算符

| 运算符 | 描述           | 实例                                                         |
| ------ | -------------- | ------------------------------------------------------------ |
| &      | 按位与运算符   | (a & b) 输出结果 12 ，二进制解释： 0000 1100                 |
| \|     | 按位或运算符   | (a \| b) 输出结果 61 ，二进制解释： 0011 1101                |
| ^      | 按位异或运算符 | (a ^ b) 输出结果 49 ，二进制解释： 0011 0001                 |
| ~      | 按位取反运算符 | (~a ) 输出结果 -61 ，二进制解释： 1100 0011， 在一个有符号二进制数的补码形式。 |
| <<     | 左移动运算符   | a << 2 输出结果 240 ，二进制解释： 1111 0000                 |
| >>     | 右移动运算符   | a >> 2 输出结果 15 ，二进制解释： 0000 1111                  |
| >>>    | 无符号右移     | A >>>2 输出结果 15, 二进制解释: 0000 1111                    |

> scala不支持三目运算符,在Scala中使用if ..else..方式实现

###### 运算符优先级

> 运算符的优先级就是表达式运算中的运算顺序

- ()[]可以直接提高优先级
- 弹幕运算符(!~)
- 算术运算符
- 位移运算符 
- 关系运算符
- 位运算
- 逻辑运算
- 赋值运算

###### scala键盘输入

```scala
StdIn.readLine() /readInt() ...
```

### 流程控制





### 函数式编程介绍

- 函数式编程是一种编程范式
- 它属于结构化编程的一种，主要思想是把运算过程尽量携程一系列嵌套的函数调用
- 函数式编程中，将函数当做数据类型，因此可以接受函数当做输入(参数)和输出(返回值)
- 函数式编程中，最重要的就是函数

##### 函数/方法的定义

###### 基本语法

```
def 函数名([参数名:参数类型]...)[[:返回值类型]=]{
	语句...
	返回值  //如果没有return ,默认以执行到最后一行的结果作为返回值
}
```

###### 过程

将函数的返回类型为Unit的函数成为过程(procedure)





## 数据结构上

##### 数据结构特点

- Scala同时支持不可变集合和可变集合

- 两个主要的包：
  - 不可变集合：scala.collection.immutable
  - 可变集合：scala.collection.mutable

- scala默认采用不可变集合，几乎所有的集合类，都同时提供了可变和不可变的两种版本

- Scala的集合有三大类：序列Seq, 集Set,映射Map，所有集合都扩展自Iterable特质

  

  ####  数组

   ##### 定义**定长**数组方式(Array)

  1. 方式一：

    	 ```
    	 val arr= new Array[Int](10)
    	 ```

     

  2. 方式二：定义数组时，直接赋值--使用apply方法创建数组对象

     ```
     val arr= Array(1,2,"hello")
     ```

  ##### 定义变长数组(ArrayBuffer)

  

  1. 方式一

     ```
     //定义，声明
     val arr = ArrayBuffer[Int]()
     //追加元素
     arr.append(7)
     //重新赋值
     arr(0)=6 //使用()的方式进行访问
     //删除元素
     arr.remove(0) //删除第一个元素
     //遍历
     for(item <- arr){
         println(item)
     }
     ```



   ###### 		定长数组与变长数组的转换

```
arr.toBuffer //转为可变数组
arr.toArray //转为定长数组
```



###### 多维数组

```
Array.ofDim[Double](3,4)
```

##### 数组的转换

- 使用for 推导式

  ```
  object ArrayDemo04 {
    def main(args: Array[String]): Unit = {
      var arr = Array(1,2,3,4,5,6).toBuffer
      //使用关键字yield产生一个新的数组缓冲--yield 将产生的数组累加到一个新的集合中
      var arr2 = for(ele <- arr) yield ele* ele
      println(arr2)
    }
  }
  ```

> 如果只想对特定的元素进行处理，可以使用守卫

- 使用守卫

  ```
    def main(args: Array[String]): Unit = {
      var arr = Array(1,2,3,4,5,6).toBuffer
      var arr2 = for (ele <- arr if(ele %2 == 0)) yield ele*ele
      println(arr2)//ArrayBuffer(4, 16, 36)
    }
    //这些操作产生的结果不会影响原有的容器
  ```

 ###### 常用算法API

```
 def main(args: Array[String]): Unit = {
    var arr = Array(10,21,32,4,15,46,17)
    println(arr.sum)  //求和
    println(arr.max)//求取最大值
    println(arr.min)//求取最小值

    println(arr.mkString) //tostring(),转为字符串类型
    println(arr.mkString(",")) //以,为分隔，调用toString()

    println("===========排序================")
    //排序
    val sorted = arr.sorted //调用arr.sorted方法
    println(sorted.mkString(","))

    println("=========降序排序==============")
    //降序  -- _ > _ 降序    _< _升序
    val ints = arr.sortWith(_ > _) //使用arr.sortWich  >左边大于右边，降序
    println(ints.mkString(","))
  }


```

###### Java数组和Scala数组之间的相互转换

- Scala数组转Java数组(List)

```
  def main(args: Array[String]): Unit = {
    val arrScala = ArrayBuffer("a","b","c")
    val listJava: util.List[String] = arrScala.asJava
    println(listJava)
  }
```

- Java数组转Scala数组

  ```
    def main(args: Array[String]): Unit = {
      val arrScala = ArrayBuffer("a","b","c")
      val listJava:util.List[String]= arrScala.asJava
      println(listJava)
      //javaList 转为scala List 需要调用asScala 
      var buf:mutable.Buffer[String] = listJava.asScala
      println(buf.mkString(","))
    }
  ```

  

- 其他操作方法

   - 可变和不可变共同有用的

     1. ++连接两个数组
     2. ++: 连接两个数组
     3. :+ 一个数组连接一个元素
     4. +: 一个元素连接一个数组
     5. /: 左折叠
     6. :\ 右折叠
     7. head :第一个元素
     8. tail : 出去第一个元素的其他元素组成的数组
     9. last : 最后一个元素
     10. max: 找到最大值
     11. min : 最小值

   - 可变数组拥有

     ![1548320595759](assets/1548320595759.png)

##### 元祖（tuple）

	##### 定义

> 将多个无关的数据封装成一个整体，成为元祖，元素最大的特点灵活，对数据没有过多的约束

注： 元祖中最大只有22个元素

##### 元祖的创建

```
  def main(args: Array[String]): Unit = {
    //第一种：
    val t1:(String,Int,String,Boolean)=("a",1,"2",true)
    //第二种：
    val t2:Tuple4[String, Int,String,Boolean]=("a",1,"2",true)
    println(t1)
    println(t2)
    //访问元祖中的数据
    println(t1._1)
    println(t1._2)
    println(t1._3)
    println(t1._4)
    //遍历元祖
    //for循环遍历
    for (ele<- t1.productIterator){
      println(ele)
    }
  }
```



##### 列表List

###### 列表List-元素的追加

```
  //方式一
  var list1 = List(1, 2, 3, "abc")
    val list2 = list1 :+ 4 //在列表后面添加新元素
    println(list2)
  //方式二
    //在列表前面增加元素
    val list3 = 100 +: list1
    println("list3=" + list3)
 //方式三
 	//::表示想集合中，新建集合添加元素
    //运算时，集合对象一定要放置在最右边
    //运算规则：从右向左
    val list4=List(1,2,3,"abc")
    val list5= 4::5::6::list4::Nil//从右向左，先有list4,
    //然后6 list  =>4 5 6 list4=> 4 5 6 list4
    println("list5="+list5)
```

##### ListBuffer

```
   //++= 
    list0++= list1 //++表示加入的是集合中的各个元素
    println(list0) //将list1中的每一个元素加入到list0中
```

##### Queue

> 基本介绍：
>
> 队列是一个有序列表，在底层可以用数组或是链表来实现
>
> 输入和输出要遵循陷入先出的原则，即：先存入队列的数组，要先取出

1. 队列Queue的创建

   new mutable.Queue[Int]

2. 出队

   - dequeue()
3. 入队

   - enqueue
4. head

   - head --返回队列首元素
5. tail

   - tail -- 返回队列除了head头元素之外组成的队列

​    def main(args: Array[String]): Unit = {
​       //创建队列
​       val q1 = new mutable.Queue[Int]
​       println(q1)
​       //追加元素
​       q1+=1
​       q1+=2
​       //将List中的元素添加到队列中
​       q1 ++= List(1,2,3)
​       println(q1)

​       //添加数组中的元素
​       q1++= Array(5,6,7)
​       println(q1)
​       //使用方法进行添加元素
​       q1.enqueue(100,200)// 参数时* ，可以添加无限个
​       println("======= head ======")
​       //返回队列头部元素
​       println(q1.head) //返回头结点信息
​       //删除队列头节点信息
​       val head  = q1.dequeue()
​       println(q1)

​       println("======  tail =====")
​       //返回队尾元素--除了队列头结点的所有元素组成的队列
​       println("queue.tail = "+ q1.tail)
​     }

   

##### 映射Map

​	Scala中的Map和Java类似，也是一个散列表，它存储的内容是键值对key-value映射，scala中不可变Map是有序的，可变的Map是无序的



###### 	构建不可变Map

```
val map = Map("Alice" -> 10, "Bob" -> 20, "Kotlin" -> "北京")
```

- 输出顺序和声明顺序一致

- 构建Map集合中，集合中的元素其实是Tuple2类型

- 默认情况下，Map是不可变map

  ###### 构建可变map

  ```
    val map = scala.collection.mutable.Map("Alice" -> 10, "Bob" -> 20, "Kotlin" -> 30)
  ```

  ###### 创建空的映射Map

  ```
      val map2 = new mutable.HashMap[String, Int]()
      println(map2)
  ```

  ###### 利用对欧元组创建

  ```
   val map4 = mutable.Map( ("A", 1), ("B", 2), ("C", 3),("D", 30) )
  ```

  ##### 映射map--取值

  ###### 	方式一：

  ```
  map(key)
  ```

  ###### 	方式二：contains--检查是否存在key

  ```
  map.contains("A")
  ```

  ###### 	方式三：

  ```
  map.get(key).get // 通过映射.get(key) 调用返回的一个Option对象，要么是some,要么是None
  ```

  ###### 	方式四：

  ```
  getOrElse //如果存在，返回key对应的值，如果不存在，返回默认值
  ```

##### 映射Map--增删改查

###### 		

```
   val map4 = mutable.Map( ("A", 1), ("B", 2), ("C", 3),("D", 30) )
    println("map4=" + map4)
    println(map4("A"))

    //map更新
    map4("A")=20 //map 必须是可修改的，否则会报错
    println(map4)//如果key不存在，会进行添加

    //添加
    map4+=("D"->40)
    println(map4) //如果已经存在，会进行修改
    //删除map元素
    map4-= ("B","A") //通过可以进行删除i
    println(map4)

    //遍历
    for ((k,v)<- map4){
      println(k+" "+v)
    }
```

TODO--Set





## 数据结构下-集合应用操作

##### 集合元素的映射-map映射操作

> ###### 将集合中的每一个元素通过制定功能映射成新的结果集 //将函数作为参数传递给另外一个函数，这就是函数式编程的特点



```
  def main(args: Array[String]): Unit = {
    //需求：将list中的所有元素全部*2
    val list  = List(1,3,5,7)
    val ints = list.map(f1)
    println(ints)
  }
  //map映射函数
  def f1(n:Int): Int={
    n*2
  }
```

```
/*
请将 val names = List("Alice", "Bob", "Nick") 中的所有单词，全部转成字母大写，返回到新的List集合中.
 */
object MapTest01 {
  def main(args: Array[String]): Unit = {
    val names=List("Alice","Bob","Nick")
    val names2 = names.map(upper)
    println("names2="+names2)
  }
  def upper(str:String):String={
    str.toUpperCase
  }
}

```

##### flatmap映射：flat即压扁，亚平，扁平化映射

###### 基本介绍

> 效果是将集合中的每个元素的子元素映射到某个函数并返回新的集合

```
object FlatMapTest01 {
  def main(args: Array[String]): Unit = {
    val names = List("Alice","Bob","Nick")
    //作用于集合中每个元素的子元素--映射到新的集合中
    println(names.flatMap(upper))
  }
  def upper(s:String) ={
    s.toUpperCase
  }
}
```

##### 集合元素的过滤-filter

> 基本介绍：filter，将符合要求的数据(筛选)放置到新的集合中

案例：将集合首字母为"C"的筛选到新的集合中

```
/*
将  val names = List("pangdi", "wangyg", "Com") 集合中首字母为'C'的筛选到新的集合
 */
object FilterTest {
  def main(args: Array[String]): Unit = {
    val names = List("pangdi", "wangyg", "Com")
    val list = names.filter(startC)
    println("list="+list)
  }
  def startC(str:String) :Boolean={
    if(str.startsWith("C"))  true else false
  }
}
```

##### 化简

###### 基本定义：将二元函数引用于集合中的函数

需求：求出list的和

```
object ReduceLeftTest {
  def main(args: Array[String]): Unit = {
    val list = List(1, 2, 3, 4, 5)
    //方式一：
    // val res = list.reduceLeft(f1)
    //方式二：
    val res = list.reduceLeft(_ + _)
    println("res=" + res)
  }

  def f1(n1: Int, n2: Int): Int = {
    n1 + n2
  }
}
```



##### 综合案例：WordCount





## Akka

##### Akaka介绍

1. akka是java虚拟机jVM平台上构建的高并发，分布式和容错应用的工具包--Akka是编写并发程序的框架
2. akka用scala语言携程，同时提供scala和java的开发接口
3. 主要解决的问题：可以轻松写出高效稳定的并发程序，程序员不用过多的考虑线程，锁和资源竞争细节

  ![image](other/1.png)

##### Akka中Actor模型

actor简化了并发编程，提升程序性能

##### actor模型

- 所有事物都是actor，一切皆actor

- actor模型作为一个并发模型设计和架构，actor与actor之间只能通过消息通信（消息的发送必须通过actorRef发送）

- actor和actor之间只能用消息进行通信，当一个actor给另一个actor发送消息，消息是有序的

- 消息的处理方式由接收方的actor决定，发送消息的actor可以阻塞等待或异步处理

- actorSystem的职责负责管理并创建actor，actorsystem是一个单例的[工厂模式],一个JVM进程中有一个即可，而actor是可以有多个的

  ![image](other/2.png)

##### 示意图

###### actor模型工作机制说明

1. actorsystem创建Actor

2. ActorRef：可以理解为Actor的代理或引用，消息是通过actorRef来发送，通过哪个ActorRef就表示要将消息发送给哪个actor

3. 消息通过actorRef发送后，会首先发送给Dispatcher Message(消息分发器),Dispatcher Message得到消息后，会将消息发送到对应的MailBox。dispatcher Message可以理解为一个线程池，mailbox可以理解成是消息队列，可以缓冲多个消息，遵循FIFO

4. Actor可以通过receive方法来获取消息，然后进行处理

   ```
   1. actorsystem创建Actor，返回该actor的引用
   2. AActor通过actorRef来发送message,将消息发送给Dispatcher Message
   3. Dispatcher（消息分发器 --线程池）Dispatcher拥有所有Actor的引用
   4. Dispatcher将message发送给BActor邮箱,mailBox是一个消息队列，可以缓存多个消息，实现了runable,是一个runable线程
   5. BActor通过receive()方法接收和处理消息，通过sender()给发送者发送消息
   ```

   ![image](other/3.png)



##### 代码1

需求：编写一个Actor，可以给自己发送消息

```
import akka.actor.{Actor, ActorSystem, Props}

/*
1. 继承Actor
2. 实现receive方法
 */
class TalklSelfActor extends  Actor{
  override def receive: Receive = {
    case "start"=> println("actor 开始运行...")
    case "hello"=> println("hello too...")
    case "fish" =>  println("<・)))><< 鱼")
    case "cat" => println("(>^ω^<)喵..")
    case  "exit"=>{
      println("准备退出...")
      context.stop(self)  //self 就是一个akka.actor.actorRef--
      context.system.terminate()//停止一切
    }
  }
}

object TalklSelfActorTest{
  def main(args: Array[String]): Unit = {
    //创建一个aActorsystem
    val actorFactory = ActorSystem("actorFactory")
    //通过actorFactory创建需要的actor
    val talklSelfActor = actorFactory.actorOf(Props[TalklSelfActor], "TalklSelfActor")
    talklSelfActor ! "start"
    talklSelfActor ! "hello"
    talklSelfActor ! "fish"
    talklSelfActor ! "cat"
    talklSelfActor ! "exit"
  }
}

```

##### ! 剖析

```
def !(message: Any)(implicit sender: ActorRef = Actor.noSender) = underlying.sendMessage(message, sender)
```

- 实质上调用了sendMessage方法，进行发送消息

##### Actor模型应用实例-Actor间通讯

- 需求：编写两个Actor，AActor，BActor

- AActor和BActor之间可以相互发送消息

  

```
//AActor
package TwoActorTalk

import akka.actor.{Actor, ActorRef}

/*
AActor需要拥有BActor的引用，所以在创建时，最好先创建BActor ,在创建AActor
 */
class AActor(iBActorRef: ActorRef) extends Actor {
  //AActor最好有BActor的引用
  var bActorRef = iBActorRef
  var count = 0

  override def receive: Receive = {
    case "start" => {
      println("AActor启动")
      println("start Ok~")
      println("我打")
      bActorRef ! "我打"
    }
    case "我打" => {
      count += 1
      println(s"AActor(黄飞鸿) 挺猛 看我佛山无影脚 第${count}脚")
      Thread.sleep(1000)
      bActorRef ! "我打"
    }
  }
}

//BAtor
package TwoActorTalk

import akka.actor.Actor

class BActor extends  Actor{
  var count =0

  //重写receive函数
  override def receive: Receive = {
    case "我打"=>{
      count+=1//计数器+1
      println(s"BActor(乔峰) 厉害，看我降龙十八掌第 ${count}掌")
      Thread.sleep(1000)
      sender() ! "我打"//发送消息
    }
  }
}
//入口函数
package TwoActorTalk

import akka.actor.{ActorRef, ActorSystem, Props}

object ActorTalk extends App {
  //先创建BActor的实例
  private val actorfactory = ActorSystem("actorfactory")
  val bActor: ActorRef = actorfactory.actorOf(Props[BActor], "BActor")
  val aactor: ActorRef = actorfactory.actorOf(Props(new AActor(bActor)), "Aactor")

  //在将BActor的实例的引用在创建AActor的时候赋值给A
  aactor ! "start"
}

```

#### Akka网络编程

###### OSI与Tcp/Ip参考模型

> OSI: 应用层，表示层，会话层，传输层，网络层，网络链路层, 物理层
>
> TCP/Ip ：应用层，传输层，网络层，链路层



##### Akka网络编程-demo客服

> 需求：服务端进行监听某端口，客户端可以通过键盘输入，发送咨询问题，服务端回答客户的问题

```
//消息协议体
package yellowchickenServer.common

/*
客户端发送的消息体
 */
case class ClientMessage(msg:String)
//服务端发送到的消息体
case class ServerMessage(msg:String)

```

```
//客户端code
package yellowchickenServer.client
import akka.actor.{Actor, ActorRef, ActorSelection, ActorSystem, Props}
import com.typesafe.config.ConfigFactory
import yellowchickenServer.common.{ClientMessage, ServerMessage}

import scala.io.StdIn
class CustomerActor extends Actor {
  //我们这里需要持有Server的Ref
  var yellowChickenServerRef: ActorSelection = _

  //preStart , 在启动Actor之前会先运行，因此变量,初始化写入preStart

  override def preStart(): Unit = {
    //println("preStart")
    //说明
    //1. 在AKKA 的Actor模型中， 认为 每个Actor都是一个资源（角色），通过一个Path来定位一个actor
    //2. path 的组成 akka.tcp://Server的actorfactory名字@ServerIp:Server的port/user/ServerActor名字
    yellowChickenServerRef = context.actorSelection("akka.tcp://Server@127.0.0.1:9999/user/YellowChickenServer")
  }

  override def receive: Receive = {
    case "start" => {
      println("客户端启动，可以咨询问题~~")
    }
    case mes: String => {
      //将mes 发送给Server
      yellowChickenServerRef ! ClientMessage(mes)
    }
    case ServerMessage(mes) => {
      println("收到客服回复的消息: " + mes)
    }

  }
}

object CustomerActor extends App {

  //编写必要的配置信息
  val serverHost = "127.0.0.1"
  val serverPort = 9999
  val clientHost = "127.0.0.1"
  val clientPort = 10000

  val config = ConfigFactory.parseString(
    s"""
       |akka.actor.provider="akka.remote.RemoteActorRefProvider"
       |akka.remote.netty.tcp.hostname=$clientHost
       |akka.remote.netty.tcp.port=$clientPort
       """.stripMargin)

  //创建CustomerActor
  val clientActorSystem = ActorSystem("Client", config)

  val customerActorRef: ActorRef = clientActorSystem.actorOf(Props[CustomerActor], "CustomerActor")

  customerActorRef ! "start"

  println("可以咨询问题了")
  while (true) {
    val mes = StdIn.readLine()
    customerActorRef ! mes //先发给自己，然后让  CustomerActor 发

  }
}
```

```
//服务端code
package yellowchickenServer.server

import akka.actor.{Actor, ActorRef, ActorSystem, Props}
import com.typesafe.config.ConfigFactory
import yellowchickenServer.common.{ClientMessage, ServerMessage}

class YellowChickenServer extends Actor{
  override def receive:Receive = {
    case "start" => {
      println("小妹开始监听程序，可以咨询问题~~")

    }
    case ClientMessage(mes) => {
      //怎么匹配他的内容
      println("客户咨询的问题是" + mes)
      mes match {
        case "姓名" => {
          sender() ! ServerMessage("wangyg")
        }
        case "地址" => {
          sender() ! ServerMessage("昌平区龙锦苑东一区")
        }
        case "工作" => {
          sender() ! ServerMessage("程序猿...")
        }
        case _ => {
          sender() ! ServerMessage("啥也不是~")
        }
      }
    }
  }
}
  object YellowChickenServer extends App{

    //创建ActorSystem
    //因为这时，我们需要监听网络，所以使用如下方法创建工厂
    //Config 就是我们的网络配置 ip , port..
    //def apply(name: String, config: Config): ActorSystem = apply(name, Option(config), None, None)

    val host = "127.0.0.1" //ip4
    val port = 9999
    //Config 就是我们的网络配置 ip , port..
    //
    val config = ConfigFactory.parseString(
      s"""
         |akka.actor.provider="akka.remote.RemoteActorRefProvider"
         |akka.remote.netty.tcp.hostname=$host
         |akka.remote.netty.tcp.port=$port
       """.stripMargin)

    val serverActorSystem = ActorSystem("Server",config)

    val yellowChickenServerRef: ActorRef = serverActorSystem.actorOf(Props[YellowChickenServer],"YellowChickenServer")

    //akka.tcp://Server@127.0.0.1:9999  就是Actor 路径
    yellowChickenServerRef ! "start"

  }
```

##### 扩展项目-spark Master Worker进程通讯项目

###### 需求分析

> 1. worker注册到Master，Master完成祖册，并回复worker注册成功--注册功能
>
> 2. worker 定时并发送心跳--3s一次，并在Master接收到
>
> 3.  Master接收到worker心跳后，要更新该worker的最近一次发送心跳的时间
> 4.  给Master启动定时任务，定时检测注册的worker有哪些没有更新心跳,并将其从hashmap中删除



```
//协议类
package SparkMasterWorker.common

//具体协议
//样例类--注册的协议：保存id,cpu,ram内存
case class RegisterWorkerInfo(id:String,cpu:Int,ram:Int)
//实体类
class WorkerInfo(val id:String, val cpu:Int, val ram:Int){
  //保存默认的心跳时间--最后的心跳时间
  var lastHeartBeatTime:Long = System.currentTimeMillis()
}

//最后一个类，用于返回注册成功后的信息--object类
case object RegisteredWorkerInfo //单例类
```



```
//SparkMaster Code
package SparkMasterWorker.master

import SparkMasterWorker.common.{RegisterWorkerInfo, RegisteredWorkerInfo, WorkerInfo}
import akka.actor.{Actor, ActorRef, ActorSystem, Props}
import com.typesafe.config.{Config, ConfigFactory}

import scala.collection.mutable

class SparkMaster extends Actor {
  var registMap = mutable.HashMap[String, WorkerInfo]()

  //实现receive方法
  override def receive: Receive = {
    //case--模式匹配，匹配各种情况，然后针对各种情况进行处理
    case "start" => {
      println("spark master启动，开始监控...")
    }
    //匹配worker向master发送注册时间的情况
    case RegisterWorkerInfo(id, cpu, ram) => {
      //向hashMap中添加信息
      if (!registMap.contains(id)) {
        // 如果不存在，原容器中不存在
        registMap += (id -> new WorkerInfo(id, cpu, ram))
        println("添加成功。。。。。。")
        //容器中添加成功后，向原发送者发送成功消息
        sender() ! RegisteredWorkerInfo
      }
    }
  }
}

//程序入口
object SparkMaster extends App {
  val masterHost = "127.0.0.1"
  val masterPort = 10000

  //设置配置
  val config: Config = ConfigFactory.parseString(
    s"""
akka.actor.provider="akka.remote.RemoteActorRefProvider"
akka.remote.netty.tcp.hostname=$masterHost
akka.remote.netty.tcp.port=$masterPort
     """.stripMargin)

  //获取actorsystem
  val sparkmasterFactory = ActorSystem("SparkMaster", config)//第一个参数：指定actorSystem的名称，第二个参数：指定配置
  //获取创建sparkMaster和引用
  val sparkMasterRef: ActorRef = sparkmasterFactory.actorOf(Props[SparkMaster], "SparkMaster01")//第一个参数：使用给定的名称作为上下文鉴定
  //发送消息
  sparkMasterRef ! "start" //sendmessage 发送消息

}

```



```
//SparkWorker code
package SparkMasterWorker.worker
import java.util.UUID

import SparkMasterWorker.common.{RegisterWorkerInfo, RegisteredWorkerInfo}
import akka.actor.{Actor, ActorRef, ActorSelection, ActorSystem, Props}
import com.typesafe.config.ConfigFactory

//第一步:继承Actor
class SparkWorker(masterHost: String, masterPort: Int) extends Actor { //通过主构造器将主机和端口号进行初始化
  //第三步：需要持有对方的引用--spark master
  var masterProxy: ActorSelection = _
  var id = UUID.randomUUID().toString //产生一个随机的UUID

  //第四步：在preStart()中对master的代理有一个初始化
  override def preStart(): Unit = {
    masterProxy = context.actorSelection(s"akka.tcp://SparkMaster@${masterHost}:${masterPort}/user/SparkMaster01") //传入一个路径
  }

  //第二部：实现receive方法
  override def receive: Receive = {
    case "start" => {
      println("spark worker启动...")
      //启动后，发送注册的请求
      masterProxy ! RegisterWorkerInfo(id, 8, 8 * 1024) //进行注册，发送RegisterWorkerInfo--id随机产生，只是模拟
    }
    //发送注册请求后，服务端会回发一个成功消息，在这里进行接收
    case RegisteredWorkerInfo => {
      println(s"收到回复，${id}已成功注册...")
    }
  }
}

object SparkWorker extends App {
  val (masterHost, masterPort, workerHost, workerPort) =
    ("127.0.0.1", 10000, "127.0.0.1", 10001)

  //创建config
  val config = ConfigFactory.parseString(
    s"""
       |akka.actor.provider="akka.remote.RemoteActorRefProvider"
       |akka.remote.netty.tcp.hostname=$workerHost
       |akka.remote.netty.tcp.port=$workerPort
       """.stripMargin)
  //创建actorsystem
  val sparkWorker = ActorSystem("sparkWorker",config)
  //创建actor
  val sparkWorkerActorRef: ActorRef = sparkWorker.actorOf(Props(new SparkWorker(masterHost,masterPort)),"SparkMaster01")
  //通过ref 进行发送消息
  sparkWorkerActorRef ! "start"

}
```

##### 	功能2：实现定时心跳

```
 //使用定时器机制，每隔3s给自己先发送一个信息，然后在发送信息给服务器端
 import context.dispatcher //需要导入一个包
      //注册成功后，进行发送3s心跳请求
      context.system.scheduler.schedule(0 millis,3000 millis,self,SendHeartBeat)
  
```

##### 	功能3：Master启动定时任务，定时检测注册的worker

> 需求：Master启动定时任务(**10**秒)，定时检测注册的worker有哪些没有更新心跳，已经超时的worker(**6秒**)，将其从hashmap中删除掉

```
//核心逻辑
case StartTimeOutWorker =>{
      //启动定时器
      import context.dispatcher //还需要import scala.concurrent.duration._
      context.system.scheduler.schedule(0 millis, 10000 millis, self, RemoveTimeOutWorker)
    }
    case RemoveTimeOutWorker =>{
      //定时清理超时6s的worker,scala
      //获取当前的时间
      val currentTime = System.currentTimeMillis()
      val workersInfo = registMap.values //获取到所有注册的worker信息
      //先将超时的一次性过滤出来，然后对过滤到的集合一次性删除
      workersInfo.filter( //进行过滤
        currentTime - _.lastHeartBeatTime > 6000
      ).foreach(workerInfo=>{
        registMap.remove(workerInfo.id)
      })

      printf("当前有%d个worker存活\n", registMap.size)
    }
```

##### 泛型

###### 基本介绍

> 如果我们要求函数的参数接收任意类型，有使用泛型，这个类型可以代表任意的数据类型, 

泛型案例：编写message类，可以构建Int类型的message，String类型的message

```
package day08

object GenericUst {
  def main(args: Array[String]): Unit = {
    val mesg = new StrMessage[String]("10")
    println("mesg="+mesg.get)
    val mesg2 = new IntMessage[Int](20)
    println("mesg2="+mesg2.get)
  }
}
//定义抽象父类
abstract class Message[T](s:T){
  def get:T = s
}
//子类继承抽象父类
class StrMessage[String](msg:String) extends Message(msg)

class IntMessage[Int](msg:Int) extends Message(msg)
```

泛型案例2：

```
package day08

import day08.SeasonEm.SeasonEm


object GenericUst2 {
  def main(args: Array[String]): Unit = {
    //[指定泛型]
    val cla01 = new EnglishClass[SeasonEm,String,String](SeasonEm.spring,"aaa","高级班")
    println(cla01.season + " "+cla01.name+" "+cla01.classType)
  }
}
/*设计一个EnglishClass(英语班级类),在创建Englishclass的一个实例
需要制定[班级开班季节spring,autumn,summer,winter] 班级名称，班级类型
开班季节只能是指定的，班级名称为String，班级类型是(字符串类型“高级班”，“初级班”..)
或是Int类型（1,2,3)
使用泛型来完成本案例
*/
object SeasonEm extends Enumeration{
  type SeasonEm = Value //自定义seasionEm,是value类型，这样才能使用
  val spring,summer,winter,autumn = Value
}

//根据业务需要，设计带有泛型的类
class EnglishClass[A,B,C](val season:SeasonEm,val name:B, val classType:C)
```

泛型案例3：

```
object GenericUst3 {
  def main(args: Array[String]): Unit = {
    val list = List("hello","world","scala")
    //获取list中间元素
    println(getEle[String](list)) //调用函数时，可以限定类型
  }
  //返回值类型自动类型推倒
  //T 使用泛型
  def getEle[T](l:List[T]) ={
    l(l.length/2) //返回list中间值
  }
}
```

##### 类型约束-上界/下界

###### scala中上界

> scala中表示某个类型是A类型的子类型，也称上界或上限，使用<: 关键字表示

```scala
[T<:A] 或使用通配符：[_<: A]
```

###### scala中下界

```
[T>:A]//A是T的下界，下限
[_>:A] //通配符形式
```

视图界定

基本介绍

> <%的意思是view bounds(视图)  它比<:使用的范围更广，除了所有的子类，还允许隐式类型转换

```scala
	<%  --支持隐式转换
```

视图界定demo1

```

object ViewBoundsTest01 {
  def main(args: Array[String]): Unit = {
    //支持隐式类型转换--11.f为scala中的Float,不需要手动转换成java.lang.Float
    val comparaComm = new CompareComm[java.lang.Float](11.1f, 10.3f)
    println("comparaComm="+comparaComm.greater)
  }
}
/*
T <% Comparable[T] 表示视图界定
T 是Comparable[T]子类型，同时也支持隐式类型转换
 */
class CompareComm[T<% Comparable[T]](obj1:T, obj2:T){
  def greater = if (obj1.compareTo(obj2)>0) obj1 else obj2
}
```

视图界定demo2

```
//package day08

//定义Person类，进行两个Person对象比较
class Person(val name: String, val age: Int) extends Ordered[Person] {
  //重写Compare方法
  override def compare(that: Person): Int = {
    //1 省略return
    println("compare方法被调用...")
    this.age - that.age
  }

  //重写tostring 方法
  override def toString: String = {
    this.name + "\t" + this.age
  }
}

//使用视图界定方式进行比较两个person类
class CompareComm2[T <% Ordered[T]](obj1: T, obj2: T) {
  def greater = if (obj1 > obj2) obj1 else obj2

  //使用compareTo方法
  def greater2 = if (obj1.compareTo(obj2) > 0) obj1 else obj2
}

object ViewBoundsTest02 {
  def main(args: Array[String]): Unit = {

    val jack = new Person("jack", 35)
    val tom = new Person("tom", 30)
    val compare = new CompareComm2(jack, tom)
    println(compare.greater)

  }
}

```

##### 类型约束-上下文界定

###### 基本介绍：

> context bounds(上下文界定)也是隐式参数的语法糖，为语法上的方便，引入了上下文界定

- 更加凸显了隐式值的价值

案例：使用上下文界定+隐式参数的方式，比较两个Person对象的年龄大小，要求：使用Ordering实现比较

```
ordered和ordering的区别：
Ordered继承了java中的Comparator接口，而Ordered继承了java中的Comparable接口，在Java中，Comparator是一个外部比较器(需要自己定义一个类实现比较器)，而Comparable是一个内部比较器，在类内部重载compareTO函数
```

上下文界定demo--要求：使用上下文界定+隐式参数的方式，比较大小

```
package com.atguigu.chapter17.contextbounds

object ContextBoundsDemo {

  //隐式参数
  implicit val personComparetor = new Ordering[Person] {
    override def compare(p1: Person, p2: Person): Int = {
      println("隐式值的 personComparetor 比较器被调用")
      p1.age - p2.age
    }
  }


  def main(args: Array[String]): Unit = {

    val p1 = new Person("mary", 30)
    val p2 = new Person("smith", 35)

    //完成p1 和 p2 的年龄的比较
    //上下文界定有三种写法

    val compareComm4 = new CompareComm4(p1, p2)
    println(compareComm4.greatter)

//    //方式2的使用方法
//    val compareComm5 = new CompareComm5(p1, p2)
//    println("compareComm5.greatter" + compareComm5.greatter)
//
//
//    //方式3的使用
//    val compareComm6 = new CompareComm6(p1,p2) //p1 =>Person
//    println("compareComm6.greatter=" + compareComm6.greatter)
  }
}

//一个普通的Person类
class Person(val name: String, val age: Int) {
  override def toString = this.name + "\t" + this.age
}

//方式1
//说明
//1. T: Ordering 表示 实现了 Ordering -> 实现 comparetor 接口
//2. implicit comparetor: Ordering[T] 隐式参数
class CompareComm4[T: Ordering](obj1: T, obj2: T)(implicit comparetor: Ordering[T]) {
  //编写了一个greatter 返回 较大的对象
  def greatter =
    if (comparetor.compare(obj1, obj2) > 0) obj1 else obj2
}

//方式2,将隐式参数放到方法内
class CompareComm5[T: Ordering](o1: T, o2: T) {
  def greatter = {
    //f1 是一个方法.
    def f1(implicit cmptor: Ordering[T]) = cmptor.compare(o1, o2)

    //使用f1
    if (f1 > 0) o1 else o2
  }
}


//方式3,使用implicitly语法糖，最简单(推荐使用)
class CompareComm6[T: Ordering](o1: T, o2: T) {
  def greatter = {

    //这句话就是会发生隐式转换，获取到隐式值 personComparetor
    //implicitly[Ordering[T]] 作用是进行上下文的匹配，如果有
    //Ordering[T] 类型 就会匹配到， 并赋给 comparetor
    val comparetor = implicitly[Ordering[T]] //Ordering[Person]

    if (comparetor.compare(o1, o2) > 0) o1 else o2
  }
}
```



##### 协变，逆变，不变

###### 基本介绍

> scala的协变(+),逆变(-),不变invariant
>
> 协变：如果A和子类型B，满足List[B]也是List[A]的子类型，就称为协变+
>
> 逆变：如果A和子类型B，满足List[A]是List[B]的子类型，就成为逆变，和原关系相反
>
> Java中，泛型类型都是不变invariant,比如List<String>不是List<Object>的子类型

```
package day08

object Convariant {
  def main(args: Array[String]): Unit = {
    //不变
    val t1: Temp3[Sub] = new Temp3[Sub]("hello")
    //    val t2:Temp3[Sub] = new Temp3[Super]("hello")
    //协变 super是sub的父类，Temp[super]是Temp[Sub]的父类
    val t4: Temp4[Super] = new Temp4[Sub]("hello")

    //逆变--super是sub的父类 temp[super]是super[sub]的子类
    val t5: Temp5[Sub] = new Temp5[Super]("hello")

  }
}

//逆变
class Temp5[-A](title: String) {
  override def toString: String = {
    title
  }
}

//协变
class Temp4[+A](title: String) {
  override def toString: String = {
    title
  }
}


//不变
class Temp3[A](title: String) {
  override def toString: String = {
    title
  }
}


class Super //父类
//sub是super的子类
class Sub extends Super
```

