#### 数据结构上

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





### 数据结构下-集合应用操作

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







