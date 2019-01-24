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