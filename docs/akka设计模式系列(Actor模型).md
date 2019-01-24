> 谈到akka就必须介绍Actor并发模型，而谈到Actor就必须看一篇叫做《A Universal Modular Actor Formalism for Artificial Intelligence 》的论文，他最早发表于1973年，提出了一种并发计算的理论模型，Actor就源于该模型



##### 简介

在Actor模型中，actor是一个并发原语，简单的说，一个actor就是一个工人，与进程或线程一样都能工作或处理任务，其实这还有点不好理解，我们可以把它想象成面向对象编程语言中的一个对象实例。在OOP中一个对象可以访问或修改另一个对象的属性，也可以直接调用另一个对象的方法

```
  private var name: String = ""
  def this (name: String) {
    this ()
    this.name = name
  }
  def getName: String = {
    return this.name
  }
  def sayHello (to: HelloWorld, msg: String): Unit = {
    System.out.println (to.getName + " 收到 " + name + " 的消息：" + msg
  }
```

sayHello在一个线程中执行基本没有问题，但是多个线程执行中，就可能出问题了，因为在执行sayHello的时候person2的name可能会被其他线程修改，导致线程不安全

​	actor和对象的不同之处在于，actor的状态不能直接读取，修改，actor的方法不能直接调用，acotr只能通过消息传递的方式与外界通信，每个对象都有一个this指针，代表对象的地址，可以通过该地址调用方法或存取状态，与此类似，actor也有一个代表本身的地址，但只能向改地址发送消息

简单点说，**actor通过消息传递的方式与外界通信。消息传递是异步的。每个actor都有一个邮箱，该邮箱接收并缓存其他actor发过来的消息，actor一次只能同步处理一个消息，处理消息过程中，除了可以接收消息，不能做任何其他操作 **



> actor通过消息传递的方式与外界通信，消息传递是异步的，每个actor都有一个邮箱，该邮箱接收并缓存其他actor发过来的消息，actor一次只能同步处理一个消息，处理消息过程中，处理可以接收消息，不能做任何其他操作

##### 好处

> acotor模型的另一个好处就是可以消除共享状态，因为它每次只能处理一条消息，所以actor内部可以安全的处理状态，而不用考虑锁机制

![img](D:\GitHubSpace\document\docs\other\img_1aeb3f16f0f7045930299c586806561a.png)

如上图所示actor之间是有层级关系的，子actor如果出现了异常会抛给父actor，父actor会根据情况重新构建子actor，子actor从出现异常，到恢复之后正常运行，这段时间内的所有消息都不会丢失，等恢复之后又可以处理下一个消息。也就是说如果一个actor抛出了异常，除了导致发生异常的消息外，任何消息都不会丢失。这容错性当然好了。当然了，为了实现这种特性，akka或者Erlang需要做很多工作的。