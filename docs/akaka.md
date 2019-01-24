---

---

##### Akaka介绍

1. akka是java虚拟机jVM平台上构建的高并发，分布式和容错应用的工具包--Akka是编写并发程序的框架

2. akka用scala语言携程，同时提供scala和java的开发接口

3. 主要解决的问题：可以轻松写出高效稳定的并发程序，程序员不用过多的考虑线程，锁和资源竞争细节

  ![image](D:\GitHubSpace\document\docs\other\1.png)

##### Akka中Actor模型
actor简化了并发编程，提升程序性能

##### actor模型

- 所有事物都是actor，一切皆actor
- actor模型作为一个并发模型设计和架构，actor与actor之间只能通过消息通信（消息的发送必须通过actorRef发送）
- actor和actor之间只能用消息进行通信，当一个actor给另一个actor发送消息，消息是有序的
- 消息的处理方式由接收方的actor决定，发送消息的actor可以阻塞等待或异步处理
- actorSystem的职责负责管理并创建actor，actorsystem是一个单例的[工厂模式],一个JVM进程中有一个即可，而actor是可以有多个的

  ![image](D:\GitHubSpace\document\docs\other\2.png)

##### 示意图

###### actor模型工作机制说明
1. actorsystem创建Actor

2. ActorRef：可以理解为Actor的代理或引用，消息是通过actorRef来发送，通过哪个ActorRef就表示要将消息发送给哪个actor

3. 消息通过actorRef发送后，会首先发送给Dispatcher Message(消息分发器),Dispatcher Message得到消息后，会将消息发送到对应的MailBox。dispatcher Message可以理解为一个线程池，mailbox可以理解成是消息队列，可以缓冲多个消息，遵循FIFO

   ![image](D:\GitHubSpace\document\docs\other\3.png)




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

