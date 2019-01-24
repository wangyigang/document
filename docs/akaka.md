#### Akaka介绍
1. akka是java虚拟机jVM平台上构建的高并发，分布式和容错应用的工具包--Akka是编写并发程序的框架
2. akka用scala语言携程，同时提供scala和java的开发接口
3. 主要解决的问题：可以轻松写出高效稳定的并发程序，程序员不用过多的考虑线程，锁和资源竞争细节
![image](https://note.youdao.com/yws/res/646/069B79A2FBD14DDD92691DFADEC3240B)

##### Akka中Actor模型
actor简化了并发编程，提升程序性能

##### actor模型
![image](https://note.youdao.com/yws/res/656/1EE8E2C4B1694719B75A1DD19EADB625)
- 所有事物都是actor，一切皆actor
- actor模型作为一个并发模型设计和架构，actor与actor之间只能通过消息通信（消息的发送必须通过actorRef发送）
- actor和actor之间只能用消息进行通信，当一个actor给另一个actor发送消息，消息是有序的
- 消息的处理方式由接收方的actor决定，发送消息的actor可以阻塞等待或异步处理
- actorSystem的职责负责管理并创建actor，actorsystem是一个单例的[工厂模式],一个JVM进程中有一个即可，而actor是可以有多个的



##### 示意图
![image](https://note.youdao.com/yws/res/683/8E69BCE85F574849B69E8FD744B25A57)
###### actor模型工作机制说明
1. actorsystem创建Actor
2. ActorRef：可以理解为Actor的代理或引用，消息是通过actorRef来发送，通过哪个ActorRef就表示要将消息发送给哪个actor
3. 消息通过actorRef发送后，会首先发送给Dispatcher Message(消息分发器),Dispatcher Message得到消息后，会