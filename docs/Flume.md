

##  概述

##### Flume定义

Flume是一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。Flume基于流式架构，灵活简单。

##### Flume的优点

```
1.可以和任意存储框架集成
2. 当输入速率大于写入目的地速率时，flume会进行缓冲，减小hdfs的压力
3. flume中事务基于channel,使用两个事务模型(sender+receiver),确保消息被可靠发送,分别负责从source到channel 和从 channel 到sink的时间传递，一旦事务中的所有数据全部成功提交到channel,source才认为数据读取完成，同理，只有成功被sink写出去的数据，才会从channel中移除
```

#### Flume组成架构

![1551364798275](assets/1551364798275.png)

##### 各个组成部分功能

##### Agent

Agent是一个JVM进程，以事件形式从数据源传到目的地，是Flume传输的基本单位，Agent的三个组成部分：source, channel, sink

#####  Source

Source是接收数据的组件。Source组件可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、spooling directory、netcat、syslog。

##### Channel

​	Channel是缓冲区，Channel是线程安全的，可以同时处理多个source和多个sink读写操作

​	分类： Memory Channel(内存队列) 和File Channel(文件队列，将所有事件写入磁盘)

##### Sink 

Sink不断轮询Channel中事件, 将事件写入或存储或发送到另一个Flume

#####  Agent 

​	Flume传输的基本单位，Event包含Header 和body(byte array)

```
Header是一个key-value字符串的HashMap， body 是一个byte Array数组
```

##### 内部原理

```
接收数据-> source->Channel处理器->事件拦截器->channel选择器(多个channel时使用)->返回写入事件的channel列表-> channel选择器-> sink处理器(多个sink时使用)->写入
```

![1551431726095](assets/1551431726095.png)

#### 拓扑结构

##### 负载均衡模式

![1551412734344](assets/1551412734344.png)

##### 聚合模式

![1551412770145](assets/1551412770145.png)



#### Flume安装

```
官网: http://flume.apache.org/
文档地址：http://flume.apache.org/FlumeUserGuide.html
下载地址：http://archive.apache.org/dist/flume/
```

##### 安装部署

```
1.上传并解压：apache-flume-1.7.0-bin.tar.gz
2. flume/flume-env.sh.template修改为flume-env.sh并配置JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
```



##  开发案例demo

##### 监控端口数据官方案例

案例需求：
监听44444端口，通过netcat工具向44444发送消息,最后Flume将监听的数据显示在控制台

###### 步骤

```
1．安装netcat工具
	sudo yum install -y nc
2. 判断端口是否被占用
	sudo netstat -tunlp | grep 44444
```

```shell
需要指定 -c/ -n / -f
bin/flume-ng agent -c conf/ -n a1 -f jobs/flume-netcat-logger.conf -Dflume.root.logger=INFO,console

-Dflume.root.logger==INFO,console ：-D表示flume运行时动态修改flume.root.logger参数属性值，并将控制台日志打印级别设置为INFO级别。日志级别包括:log、info、warn、error。
```

###### netcat

​	功能描述：netstat命令是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的状态信息。 
基本语法：netstat [选项]
选项参数：
​	-t或--tcp：显示TCP传输协议的连线状况； 
-u或--udp：显示UDP传输协议的连线状况；
​	-n或--numeric：直接使用ip地址，而不通过域名服务器(数字方式)
​	-l或--listening：显示监控中的服务器的Socket； 
​	-p或--programs：显示pid



##### 实时读取本地文件到HDFS案例

案例需求：实时监控Hive日志，并上传到HDFS中

```
#整体的设置
a1.sources = r1
a1.channels= c1
a1.sinks= s1

#对source的描述
a1.sources.r1.type= exec
a1.sources.r1.command = tail -F /opt/module/hive/logs/hive.log
a1.sources.r1.shell = /bin/bash -c

#对channel描述
a1.channels.c1.type=memory
a1.channels.c1.capacity=1000
a1.channels.c1.transactionCapacity =100

#对sink进行描述
a1.sinks.s1.type=hdfs
#书写的hdfs路径--并以当前时间命名 %D 月日年 %H以小时方式
a1.sinks.s1.hdfs.path=hdfs://hadoop102:9000/flume/%D/%H
#文件前缀--生成的文件以logs-为前缀
a1.sinks.s1.hdfs.filePrefix = logs-
#滚动文件--定时进行滚动文件,将写的文件封闭掉，然后生成一个新的文件
a1.sinks.s1.hdfs.rollInterval = 60
#按照文件大小滚动
a1.sinks.s1.hdfs.rollSize = 134217700
#按照事件数进行滚动
a1.sinks.s1.hdfs.rollCount = 0
#批处理大小
a1.sinks.s1.hdfs.batchSize = 1000
#文件类型--datastream 普通文件格式，
a1.sinks.s1.hdfs.fileType = DataStream
#最小块副本--datanode启动时要向namenode报告块信息，每一块的信息报告后达到99%可以
a1.sinks.s1.hdfs.minBlockReplicas = 1
#文件夹滚动
a1.sinks.s1.hdfs.round = true
#滚动的数量
a1.sinks.s1.hdfs.roundValue = 1
#这两个合起来表示按照1小时进行滚动一个文件夹
a1.sinks.s1.hdfs.roundUnit = hour
#当时用转义序列时 默认从header中获取时间数据--默认为false
a1.sinks.s1.hdfs.useLocalTimeStamp = true

a1.sources.r1.channels = c1
a1.sinks.s1.channel =c1
```

###### 启动命令

```
bin/flume-ng agent -c conf/ -n a1 -f jobs/flume-exec-hdfs.conf 
```



##### 实时读取目录文件到HDFS案例

需求：使用Flume监听整个目录的文件

要点：使用Spooldir Source监控文件夹变化

```
a3.sources = r3
a3.sinks = k3
a3.channels = c3

# Describe/configure the source
a3.sources.r3.type = spooldir
#spoolDir监听的路径
a3.sources.r3.spoolDir = /opt/module/flume/upload
#完成上传后，进行修改后缀
a3.sources.r3.fileSuffix = .COMPLETED
#是否向header中写入绝对路径
a3.sources.r3.fileHeader = true
#忽略所有以.tmp结尾的文件，不上传
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)

# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = hdfs://hadoop102:9000/flume/upload/%Y%m%d/%H
a3.sinks.k3.hdfs.filePrefix = upload-
a3.sinks.k3.hdfs.round = true
a3.sinks.k3.hdfs.roundValue = 1
a3.sinks.k3.hdfs.roundUnit = hour
a3.sinks.k3.hdfs.useLocalTimeStamp = true
a3.sinks.k3.hdfs.batchSize = 100
a3.sinks.k3.hdfs.fileType = DataStream
a3.sinks.k3.hdfs.rollInterval = 60
a3.sinks.k3.hdfs.rollSize = 134217700
a3.sinks.k3.hdfs.rollCount = 0

a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```



```
bin/flume-ng agent -c conf/ -n a3 -f jobs/flume-dir-hdfs.conf
//监控的文件夹中如果出现错误，任务就会停止
```

##### 单数据源多出口案例(选择器)

![1551439562632](assets/1551439562632.png)

需求：使用Flume-1监控文件变动，Flume-1将变动内容传递给Flume-2，Flume-2负责存储到HDFS。同时Flume-1将变动内容传递给Flume-3，Flume-3负责输出到Local FileSystem。



```
#一个source 两个channel 两个sink
a1.sources = r1
a1.channels = c1 c2
a1.sinks = s1 s2

#配置source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/module/hive/logs/hive.log
a1.sources.r1.shell=/bin/bash -c

#配置channel
a1.channels.c1.type= memory
a1.channels.c1.capacity=1000
a1.channels.c1.transactionCapacity=100

a1.channels.c2.type= memory
a1.channels.c2.capacity=1000
a1.channels.c2.transactionCapacity=100

#配置sink属性为avro属性
#相当于数据的发送者
a1.sinks.s1.type = avro
a1.sinks.s1.hostname=hadoop102
a1.sinks.s1.port=4141

a1.sinks.s2.type=avro
a1.sinks.s2.hostname=hadoop102
a1.sinks.s2.port=4142

#对接起来
a1.sources.r1.channels=c1 c2
a1.sinks.s1.channel=c1
a1.sinks.s2.channel=c2
```

```
#进行命名
a2.sources=r1
a2.channels=c1
a2.sinks=k1

#组装source
#bind port 
a2.sources.r1.type=avro
a2.sources.r1.bind=hadoop102
a2.sources.r1.port=4141

#配置channel
a2.channels.c1.type= memory
a2.channels.c1.capacity=1000
a2.channels.c1.transactionCapacity=100

#配置sink--hdfs类型
a2.sinks.k1.type = hdfs
a2.sinks.k1.hdfs.path = hdfs://hadoop102:9000/flume2/upload/%Y%m%d/%H
a2.sinks.k1.hdfs.filePrefix = upload-
a2.sinks.k1.hdfs.round = true
a2.sinks.k1.hdfs.roundValue = 1
a2.sinks.k1.hdfs.roundUnit = hour
a2.sinks.k1.hdfs.useLocalTimeStamp = true
a2.sinks.k1.hdfs.batchSize = 100
a2.sinks.k1.hdfs.fileType = DataStream
a2.sinks.k1.hdfs.rollInterval = 60
a2.sinks.k1.hdfs.rollSize = 134217700
a2.sinks.k1.hdfs.rollCount = 0

#进行拼接绑定
a2.sources.r1.channels=c1
a2.sinks.k1.channel=c1
```

```
#进行命名
a3.sources=r1
a3.channels=c1
a3.sinks=k1

#组装source
#bind port 
a3.sources.r1.type=avro
a3.sources.r1.bind=hadoop102
a3.sources.r1.port=4142

#配置channel
a3.channels.c1.type= memory
a3.channels.c1.capacity=1000
a3.channels.c1.transactionCapacity=100

#配置sink--本地生成的文件夹必须存在
a3.sinks.k1.type=file_roll
a3.sinks.k1.sink.diretory= /opt/module/data/flume3

#进行拼接绑定
a3.sources.r1.channels=c1
a3.sinks.k1.channel=c1
```

```
bin/flume-ng agent -c conf/ -n a1 -f jobs/oup1/flume-exec-avro.conf 

bin/flume-ng agent -c conf/ -n a2 -f jobs/oup1/flume-avro-hdfs.conf 

bin/flume-ng agent -c conf/ -n a3 -f jobs/oup1/flume-avro-file.conf 
```

提示：输出的本地目录必须是已经存在的目录，如果该目录不存在，并不会创建新的目录。

```
 bin/hive
 启动hive时会在监控文件下生成日志log
```

##### sink选择器设置

```
replicating(默认):将数据拷贝给所有channel
	a1.sources.r1.selector.type=replicating
Multiplexing: 根据配置发送到特定的channel
```

###### Multiplexing

```
a1.sources = r1

a1.sources.source1.selector.type = multiplexing
a1.sources.source1.selector.header = validation # 以header中的validation对应的值作为条件
a1.sources.source1.selector.mapping.SUCCESS = c2 # 如果header中validation的值为SUCCESS，使用c2这个channel
a1.sources.source1.selector.mapping.FAIL = c1 # 如果header中validation的值为FAIL，使用c1这个channel
a1.sources.source1.selector.default = c1 # 默认使用c1这个channel
//要使用拦截器对数据进行处理，改变头部信息
```



#####  单数据源多出口案例(Sink组)

案例需求：使用Flume-1监控文件变动，Flume-1将变动内容传递给Flume-2，Flume-2负责存储到HDFS。同时Flume-1将变动内容传递给Flume-3，Flume-3也负责存储到HDFS 

![1551446400481](assets/1551446400481.png)



```
#一个source 一个channel 多个sink，这时就有组的概念了
a1.sources = r1
a1.channels=c1
a1.sinkgroups=g1
a1.sinks=k1 k2

#配置sink组的成员
a1.sinkgroups.g1.sinks=k1 k2
a1.sinkgroups.g1.processor.type=load_balance
#指数级的退出
a1.sinkgroups.g1.processor.backoff=true
a1.sinkgroups.g1.processor.selector=random
a1.sinkgroups.g1.processor.selector.maxTimeout=10000

#配置source--绑定端口号，进行输出
a1.sources.r1.type=netcat
a1.sources.r1.bind=localhost
a1.sources.r1.port=44444

#一个channel对应多个sink，需要添加sink处理器
#故障转移  / 负载均衡load balnacing

#配置sink
a1.sinks.k1.type=avro
a1.sinks.k1.hostname=hadoop102
a1.sinks.k1.port=4141

a1.sinks.k2.type=avro
a1.sinks.k2.hostname=hadoop102
a1.sinks.k2.port=4142


#配置channel
a1.channels.c1.type=memory
a1.channels.c1.capacity=1000
a1.channels.c1.transactionCapacity =100

#进行绑定
a1.sources.r1.channels=c1
a1.sinks.k1.channel=c1
a1.sinks.k2.channel=c1
```

```
a2.sources = r1
a2.sinks = k1
a2.channels = c1

#配置sources
a2.sources.r1.type = avro
a2.sources.r1.bind = hadoop102
a2.sources.r1.port = 4141

# 配置sink
a2.sinks.k1.type = logger

# 配置channel
a2.channels.c1.type = memory
a2.channels.c1.capacity = 1000
a2.channels.c1.transactionCapacity = 100

#进行连接
a2.sources.r1.channels = c1
a2.sinks.k1.channel = c1
```

```
a3.sources = r1
a3.sinks = k1
a3.channels = c2

# Describe/configure the source
a3.sources.r1.type = avro
a3.sources.r1.bind = hadoop102
a3.sources.r1.port = 4142

# Describe the sink
a3.sinks.k1.type = logger

# Describe the channel
a3.channels.c2.type = memory
a3.channels.c2.capacity = 1000
a3.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a3.sources.r1.channels = c2
a3.sinks.k1.channel = c2
```

```
#启动命令
bin/flume-ng agent -c conf/ -n a1 -f jobs/group2/flume-netcat-avro.conf 

 bin/flume-ng agent -c conf/ -n a2 -f jobs/oup2/flume-avro-console1.conf -Dflume.root.logger=INFO,console

bin/flume-ng agent -c conf/ -n a3 -f jobs/oup2/flume-avro-console2.conf -Dflume.root.logger=INFO,console

```



##### 多数据源汇总案例

![1551454779756](assets/1551454779756.png)

案例需求：
hadoop103上的Flume-1监控文件/opt/module/group.log，
hadoop102上的Flume-2监控某一个端口的数据流，
Flume-1与Flume-2将数据发送给hadoop104上的Flume-3，Flume-3将最终数据打印到控制台。




配置Source用于监控hive.log文件，配置Sink输出数据到下一级Flume。
在hadoop103上创建配置文件并打开
[atguigu@hadoop103 group3]$ touch flume1-logger-flume.conf
[atguigu@hadoop103 group3]$ vim flume1-logger-flume.conf 
添加如下内容


4．执行配置文件
分别开启对应配置文件：flume3-flume-logger.conf，flume2-netcat-flume.conf，flume1-logger-flume.conf。
[atguigu@hadoop104 flume]$ bin/flume-ng agent --conf conf/ --name a3 --conf-file job/group3/flume3-flume-logger.conf -Dflume.root.logger=INFO,console

[atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name a2 --conf-file job/group3/flume2-netcat-flume.conf

[atguigu@hadoop103 flume]$ bin/flume-ng agent --conf conf/ --name a1 --conf-file job/group3/flume1-logger-flume.conf
5．在hadoop103上向/opt/module目录下的group.log追加内容
[atguigu@hadoop103 module]$ echo 'hello' > group.log
6．在hadoop102上向44444端口发送数据
[atguigu@hadoop102 flume]$ telnet hadoop102 44444
7.检查hadoop104上数据





##  Flume监控之Ganglia

4.1 Ganglia的安装与部署
1) 安装httpd服务与php
[atguigu@hadoop102 flume]$ sudo yum -y install httpd php
2) 安装其他依赖
[atguigu@hadoop102 flume]$ sudo yum -y install rrdtool perl-rrdtool rrdtool-devel
[atguigu@hadoop102 flume]$ sudo yum -y install apr-devel
3) 安装ganglia
[atguigu@hadoop102 flume]$ sudo rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
[atguigu@hadoop102 flume]$ sudo yum -y install ganglia-gmetad 
[atguigu@hadoop102 flume]$ sudo yum -y install ganglia-web
[atguigu@hadoop102 flume]$ sudo yum install -y ganglia-gmond
Ganglia由gmond、gmetad和gweb三部分组成。
gmond（Ganglia Monitoring Daemon）是一种轻量级服务，安装在每台需要收集指标数据的节点主机上。使用gmond，你可以很容易收集很多系统指标数据，如CPU、内存、磁盘、网络和活跃进程的数据等。
gmetad（Ganglia Meta Daemon）整合所有信息，并将其以RRD格式存储至磁盘的服务。
gweb（Ganglia Web）Ganglia可视化工具，gweb是一种利用浏览器显示gmetad所存储数据的PHP前端。在Web界面中以图表方式展现集群的运行状态下收集的多种不同指标数据。
4) 修改配置文件/etc/httpd/conf.d/ganglia.conf
[atguigu@hadoop102 flume]$ sudo vim /etc/httpd/conf.d/ganglia.conf
修改为红颜色的配置：


尖叫提示：selinux本次生效关闭必须重启，如果此时不想重启，可以临时生效之：
[atguigu@hadoop102 flume]$ sudo setenforce 0
5) 启动ganglia
[atguigu@hadoop102 flume]$ sudo service httpd start
[atguigu@hadoop102 flume]$ sudo service gmetad start
[atguigu@hadoop102 flume]$ sudo service gmond start
6) 打开网页浏览ganglia页面
http://192.168.1.102/ganglia
尖叫提示：如果完成以上操作依然出现权限不足错误，请修改/var/lib/ganglia目录的权限：
[atguigu@hadoop102 flume]$ sudo chmod -R 777 /var/lib/ganglia
4.2 操作Flume测试监控
1) 修改/opt/module/flume/conf目录下的flume-env.sh配置：
JAVA_OPTS="-Dflume.monitoring.type=ganglia
-Dflume.monitoring.hosts=192.168.1.102:8649
-Xms100m
-Xmx200m"
2) 启动Flume任务
[atguigu@hadoop102 flume]$ bin/flume-ng agent \
--conf conf/ \
--name a1 \
--conf-file job/flume-netcat-logger.conf \
-Dflume.root.logger=INFO,console \
-Dflume.monitoring.type=ganglia \
-Dflume.monitoring.hosts=192.168.1.102:8649
3) 发送数据观察ganglia监测图
[atguigu@hadoop102 flume]$ nc localhost 44444
样式如图：

图例说明：
字段（图表名称）	字段含义
EventPutAttemptCount	source尝试写入channel的事件总数量
EventPutSuccessCount	成功写入channel且提交的事件总数量
EventTakeAttemptCount	sink尝试从channel拉取事件的总数量。这不意味着每次事件都被返回，因为sink拉取的时候channel可能没有任何数据。
EventTakeSuccessCount	sink成功读取的事件的总数量
StartTime	channel启动的时间（毫秒）
StopTime	channel停止的时间（毫秒）
ChannelSize	目前channel中事件的总数量
ChannelFillPercentage	channel占用百分比
ChannelCapacity	channel的容量
第5章 自定义Source
5.1 介绍
Source是负责接收数据到Flume Agent的组件。Source组件可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy。官方提供的source类型已经很多，但是有时候并不能满足实际开发当中的需求，此时我们就需要根据实际需求自定义某些source。
官方也提供了自定义source的接口：
https://flume.apache.org/FlumeDeveloperGuide.html#source根据官方说明自定义MySource需要继承AbstractSource类并实现Configurable和PollableSource接口。
实现相应方法：
getBackOffSleepIncrement()//暂不用
getMaxBackOffSleepInterval()//暂不用
configure(Context context)//初始化context（读取配置文件内容）
process()//获取数据封装成event并写入channel，这个方法将被循环调用。
使用场景：读取MySQL数据或者其他文件系统。
5.2 需求
使用flume接收数据，并给每条数据添加前缀，输出到控制台。前缀可从flume配置文件中配置。

5.2 分析

5.3 编码
导入pom依赖
<dependencies>
    <dependency>
        <groupId>org.apache.flume</groupId>
        <artifactId>flume-ng-core</artifactId>
        <version>1.7.0</version>
</dependency>

</dependencies>

package com.atguigu;

import org.apache.flume.Context;
import org.apache.flume.EventDeliveryException;
import org.apache.flume.PollableSource;
import org.apache.flume.conf.Configurable;
import org.apache.flume.event.SimpleEvent;
import org.apache.flume.source.AbstractSource;

import java.util.HashMap;

public class MySource extends AbstractSource implements Configurable, PollableSource {

    //定义配置文件将来要读取的字段
    private Long delay;
    private String field;
    
    //初始化配置信息
    @Override
    public void configure(Context context) {
        delay = context.getLong("delay");
        field = context.getString("field", "Hello!");
    }
    
    @Override
    public Status process() throws EventDeliveryException {
    
        try {
            //创建事件头信息
            HashMap<String, String> hearderMap = new HashMap<>();
            //创建事件
            SimpleEvent event = new SimpleEvent();
            //循环封装事件
            for (int i = 0; i < 5; i++) {
                //给事件设置头信息
                event.setHeaders(hearderMap);
                //给事件设置内容
                event.setBody((field + i).getBytes());
                //将事件写入channel
                getChannelProcessor().processEvent(event);
                Thread.sleep(delay);
            }
        } catch (Exception e) {
            e.printStackTrace();
            return Status.BACKOFF;
        }
        return Status.READY;
    }
    
    @Override
    public long getBackOffSleepIncrement() {
        return 0;
    }
    
    @Override
    public long getMaxBackOffSleepInterval() {
        return 0;
    }
}
5.4 测试
1）打包
将写好的代码打包，并放到flume的lib目录（/opt/module/flume）下。
2）配置文件
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = com.atguigu.MySource
a1.sources.r1.delay = 1000
#a1.sources.r1.field = atguigu

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
3）开启任务
[atguigu@hadoop102 flume]$ pwd
/opt/module/flume
[atguigu@hadoop102 flume]$ bin/flume-ng agent -c conf/ -f job/mysource.conf -n a1 -Dflume.root.logger=INFO,console
4）结果展示

第6章 自定义Sink
6.1 介绍
Sink不断地轮询Channel中的事件且批量地移除它们，并将这些事件批量写入到存储或索引系统、或者被发送到另一个Flume Agent。
Sink是完全事务性的。在从Channel批量删除数据之前，每个Sink用Channel启动一个事务。批量事件一旦成功写出到存储系统或下一个Flume Agent，Sink就利用Channel提交事务。事务一旦被提交，该Channel从自己的内部缓冲区删除事件。
Sink组件目的地包括hdfs、logger、avro、thrift、ipc、file、null、HBase、solr、自定义。官方提供的Sink类型已经很多，但是有时候并不能满足实际开发当中的需求，此时我们就需要根据实际需求自定义某些Sink。
官方也提供了自定义source的接口：
https://flume.apache.org/FlumeDeveloperGuide.html#sink根据官方说明自定义MySink需要继承AbstractSink类并实现Configurable接口。
实现相应方法：
configure(Context context)//初始化context（读取配置文件内容）
process()//从Channel读取获取数据（event），这个方法将被循环调用。
使用场景：读取Channel数据写入MySQL或者其他文件系统。
6.2 需求
使用flume接收数据，并在Sink端给每条数据添加前缀和后缀，输出到控制台。前后缀可在flume任务配置文件中配置。
流程分析：

6.3 编码
package com.atguigu;

import org.apache.flume.*;
import org.apache.flume.conf.Configurable;
import org.apache.flume.sink.AbstractSink;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MySink extends AbstractSink implements Configurable {

    //创建Logger对象
    private static final Logger LOG = LoggerFactory.getLogger(AbstractSink.class);
    
    private String prefix;
    private String suffix;
    
    @Override
    public Status process() throws EventDeliveryException {
    
        //声明返回值状态信息
        Status status;
    
        //获取当前Sink绑定的Channel
        Channel ch = getChannel();
    
        //获取事务
        Transaction txn = ch.getTransaction();
    
        //声明事件
        Event event;
    
        //开启事务
        txn.begin();
    
        //读取Channel中的事件，直到读取到事件结束循环
        while (true) {
            event = ch.take();
            if (event != null) {
                break;
            }
        }
        try {
            //处理事件（打印）
            LOG.info(prefix + new String(event.getBody()) + suffix);
    
            //事务提交
            txn.commit();
            status = Status.READY;
        } catch (Exception e) {
    
            //遇到异常，事务回滚
            txn.rollback();
            status = Status.BACKOFF;
        } finally {
    
            //关闭事务
            txn.close();
        }
        return status;
    }
    
    @Override
    public void configure(Context context) {
    
        //读取配置文件内容，有默认值
        prefix = context.getString("prefix", "hello:");
    
        //读取配置文件内容，无默认值
        suffix = context.getString("suffix");
    }
}
6.4 测试
1）打包
将写好的代码打包，并放到flume的lib目录（/opt/module/flume）下。
2）配置文件
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = com.atguigu.MySink
#a1.sinks.k1.prefix = atguigu:
a1.sinks.k1.suffix = :atguigu

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
3）开启任务
[atguigu@hadoop102 flume]$ pwd
/opt/module/flume
[atguigu@hadoop102 flume]$ bin/flume-ng agent -c conf/ -f job/mysink.conf -n a1 -Dflume.root.logger=INFO,console
[atguigu@hadoop102 ~]$ nc localhost 44444
hello
OK
atguigu
OK
4）结果展示

第7章 知识扩展
7.1 常见正则表达式语法
元字符	描述
^	匹配输入字符串的开始位置。如果设置了RegExp对象的Multiline属性，^也匹配“\n”或“\r”之后的位置。
$	匹配输入字符串的结束位置。如果设置了RegExp对象的Multiline属性，$也匹配“\n”或“\r”之前的位置。
*	匹配前面的子表达式任意次。例如，zo*能匹配“z”，“zo”以及“zoo”。*等价于{0,}。
+	匹配前面的子表达式一次或多次(大于等于1次）。例如，“zo+”能匹配“zo”以及“zoo”，但不能匹配“z”。+等价于{1,}。
  [a-z]	字符范围。匹配指定范围内的任意字符。例如，“[a-z]”可以匹配“a”到“z”范围内的任意小写字母字符。
  注意:只有连字符在字符组内部时,并且出现在两个字符之间时,才能表示字符的范围; 如果出字符组的开头,则只能表示连字符本身.
  7.2 自定义MySQLSource
  7.2.1 自定义Source说明
  实时监控MySQL，从MySQL中获取数据传输到HDFS或者其他存储框架，所以此时需要我们自己实现MySQLSource。
  官方也提供了自定义source的接口：
  官网说明：https://flume.apache.org/FlumeDeveloperGuide.html#source
  7.2.2 自定义MySQLSource组成
  图6-1 自定义MySQLSource组成
  7.2.3 自定义MySQLSource步骤
  根据官方说明自定义mysqlsource需要继承AbstractSource类并实现Configurable和PollableSource接口。
  实现相应方法：
  getBackOffSleepIncrement()//暂不用
  getMaxBackOffSleepInterval()//暂不用
  configure(Context context)//初始化context
  process()//获取数据（从mysql获取数据，业务处理比较复杂，所以我们定义一个专门的类——SQLSourceHelper来处理跟mysql的交互），封装成event并写入channel，这个方法被循环调用
  stop()//关闭相关的资源

PollableSource：从source中提取数据，将其发送到channel。
Configurable：实现了Configurable的任何类都含有一个context，使用context获取配置信息。
7.2.4 代码实现
1、导入pom依赖
<dependencies>
    <dependency>
        <groupId>org.apache.flume</groupId>
        <artifactId>flume-ng-core</artifactId>
        <version>1.7.0</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.27</version>
    </dependency>
</dependencies>
2、添加配置信息
在classpath下添加jdbc.properties和log4j.properties
jdbc.properties:
dbDriver=com.mysql.jdbc.Driver
dbUrl=jdbc:mysql://hadoop102:3306/mysqlsource?useUnicode=true&characterEncoding=utf-8
dbUser=root
dbPassword=000000
log4j.properties:
#--------console-----------
log4j.rootLogger=info,myconsole,myfile
log4j.appender.myconsole=org.apache.log4j.ConsoleAppender
log4j.appender.myconsole.layout=org.apache.log4j.SimpleLayout
#log4j.appender.myconsole.layout.ConversionPattern =%d [%t] %-5p [%c] - %m%n

#log4j.rootLogger=error,myfile
log4j.appender.myfile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.myfile.File=/tmp/flume.log
log4j.appender.myfile.layout=org.apache.log4j.PatternLayout
log4j.appender.myfile.layout.ConversionPattern =%d [%t] %-5p [%c] - %m%n
3、SQLSourceHelper
1) 属性说明：
属性	说明（括号中为默认值）
runQueryDelay	查询时间间隔（10000）
batchSize	缓存大小（100）
startFrom	查询语句开始id（0）
currentIndex	查询语句当前id，每次查询之前需要查元数据表
recordSixe	查询返回条数
table	监控的表名
columnsToSelect	查询字段（*）
customQuery	用户传入的查询语句
query	查询语句
defaultCharsetResultSet	编码格式（UTF-8）
2) 方法说明：
方法	说明
SQLSourceHelper(Context context)	构造方法，初始化属性及获取JDBC连接
InitConnection(String url, String user, String pw)	获取JDBC连接
checkMandatoryProperties()	校验相关属性是否设置（实际开发中可增加内容）
buildQuery()	根据实际情况构建sql语句，返回值String
executeQuery()	执行sql语句的查询操作，返回值List<List<Object>>
getAllRows(List<List<Object>> queryResult)	将查询结果转换为String，方便后续操作
updateOffset2DB(int size)	根据每次查询结果将offset写入元数据表
execSql(String sql)	具体执行sql语句方法
getStatusDBIndex(int startFrom)	获取元数据表中的offset
queryOne(String sql)	获取元数据表中的offset实际sql语句执行方法
close()	关闭资源
3)代码实现：
package com.atguigu.source;

import org.apache.flume.Context;
import org.apache.flume.conf.ConfigurationException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.sql.*;
import java.text.ParseException;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

public class SQLSourceHelper {

    private static final Logger LOG = LoggerFactory.getLogger(SQLSourceHelper.class);
    
    private int runQueryDelay, //两次查询的时间间隔
            startFrom,            //开始id
            currentIndex,	     //当前id
            recordSixe = 0,      //每次查询返回结果的条数
            maxRow;                //每次查询的最大条数


    private String table,       //要操作的表
            columnsToSelect,     //用户传入的查询的列
            customQuery,          //用户传入的查询语句
            query,                 //构建的查询语句
            defaultCharsetResultSet;//编码集
    
    //上下文，用来获取配置文件
    private Context context;
    
    //为定义的变量赋值（默认值），可在flume任务的配置文件中修改
    private static final int DEFAULT_QUERY_DELAY = 10000;
    private static final int DEFAULT_START_VALUE = 0;
    private static final int DEFAULT_MAX_ROWS = 2000;
    private static final String DEFAULT_COLUMNS_SELECT = "*";
    private static final String DEFAULT_CHARSET_RESULTSET = "UTF-8";
    
    private static Connection conn = null;
    private static PreparedStatement ps = null;
    private static String connectionURL, connectionUserName, connectionPassword;
    
    //加载静态资源
    static {
        Properties p = new Properties();
        try {
            p.load(SQLSourceHelper.class.getClassLoader().getResourceAsStream("jdbc.properties"));
            connectionURL = p.getProperty("dbUrl");
            connectionUserName = p.getProperty("dbUser");
            connectionPassword = p.getProperty("dbPassword");
            Class.forName(p.getProperty("dbDriver"));
        } catch (IOException | ClassNotFoundException e) {
            LOG.error(e.toString());
        }
    }
    
    //获取JDBC连接
    private static Connection InitConnection(String url, String user, String pw) {
        try {
            Connection conn = DriverManager.getConnection(url, user, pw);
            if (conn == null)
                throw new SQLException();
            return conn;
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
    
    //构造方法
    SQLSourceHelper(Context context) throws ParseException {
        //初始化上下文
        this.context = context;
    
        //有默认值参数：获取flume任务配置文件中的参数，读不到的采用默认值
        this.columnsToSelect = context.getString("columns.to.select", DEFAULT_COLUMNS_SELECT);
        this.runQueryDelay = context.getInteger("run.query.delay", DEFAULT_QUERY_DELAY);
        this.startFrom = context.getInteger("start.from", DEFAULT_START_VALUE);
        this.defaultCharsetResultSet = context.getString("default.charset.resultset", DEFAULT_CHARSET_RESULTSET);
    
        //无默认值参数：获取flume任务配置文件中的参数
        this.table = context.getString("table");
        this.customQuery = context.getString("custom.query");
        connectionURL = context.getString("connection.url");
        connectionUserName = context.getString("connection.user");
        connectionPassword = context.getString("connection.password");
        conn = InitConnection(connectionURL, connectionUserName, connectionPassword);
    
        //校验相应的配置信息，如果没有默认值的参数也没赋值，抛出异常
        checkMandatoryProperties();
        //获取当前的id
        currentIndex = getStatusDBIndex(startFrom);
        //构建查询语句
        query = buildQuery();
    }
    
    //校验相应的配置信息（表，查询语句以及数据库连接的参数）
    private void checkMandatoryProperties() {
        if (table == null) {
            throw new ConfigurationException("property table not set");
        }
        if (connectionURL == null) {
            throw new ConfigurationException("connection.url property not set");
        }
        if (connectionUserName == null) {
            throw new ConfigurationException("connection.user property not set");
        }
        if (connectionPassword == null) {
            throw new ConfigurationException("connection.password property not set");
        }
    }
    
    //构建sql语句
    private String buildQuery() {
        String sql = "";
        //获取当前id
        currentIndex = getStatusDBIndex(startFrom);
        LOG.info(currentIndex + "");
        if (customQuery == null) {
            sql = "SELECT " + columnsToSelect + " FROM " + table;
        } else {
            sql = customQuery;
        }
        StringBuilder execSql = new StringBuilder(sql);
        //以id作为offset
        if (!sql.contains("where")) {
            execSql.append(" where ");
            execSql.append("id").append(">").append(currentIndex);
            return execSql.toString();
        } else {
            int length = execSql.toString().length();
            return execSql.toString().substring(0, length - String.valueOf(currentIndex).length()) + currentIndex;
        }
    }
    
    //执行查询
    List<List<Object>> executeQuery() {
        try {
            //每次执行查询时都要重新生成sql，因为id不同
            customQuery = buildQuery();
            //存放结果的集合
            List<List<Object>> results = new ArrayList<>();
            if (ps == null) {
                //
                ps = conn.prepareStatement(customQuery);
            }
            ResultSet result = ps.executeQuery(customQuery);
            while (result.next()) {
                //存放一条数据的集合（多个列）
                List<Object> row = new ArrayList<>();
                //将返回结果放入集合
                for (int i = 1; i <= result.getMetaData().getColumnCount(); i++) {
                    row.add(result.getObject(i));
                }
                results.add(row);
            }
            LOG.info("execSql:" + customQuery + "\nresultSize:" + results.size());
            return results;
        } catch (SQLException e) {
            LOG.error(e.toString());
            // 重新连接
            conn = InitConnection(connectionURL, connectionUserName, connectionPassword);
        }
        return null;
    }
    
    //将结果集转化为字符串，每一条数据是一个list集合，将每一个小的list集合转化为字符串
    List<String> getAllRows(List<List<Object>> queryResult) {
        List<String> allRows = new ArrayList<>();
        if (queryResult == null || queryResult.isEmpty())
            return allRows;
        StringBuilder row = new StringBuilder();
        for (List<Object> rawRow : queryResult) {
            Object value = null;
            for (Object aRawRow : rawRow) {
                value = aRawRow;
                if (value == null) {
                    row.append(",");
                } else {
                    row.append(aRawRow.toString()).append(",");
                }
            }
            allRows.add(row.toString());
            row = new StringBuilder();
        }
        return allRows;
    }
    
    //更新offset元数据状态，每次返回结果集后调用。必须记录每次查询的offset值，为程序中断续跑数据时使用，以id为offset
    void updateOffset2DB(int size) {
        //以source_tab做为KEY，如果不存在则插入，存在则更新（每个源表对应一条记录）
        String sql = "insert into flume_meta(source_tab,currentIndex) VALUES('"
                + this.table
                + "','" + (recordSixe += size)
                + "') on DUPLICATE key update source_tab=values(source_tab),currentIndex=values(currentIndex)";
        LOG.info("updateStatus Sql:" + sql);
        execSql(sql);
    }
    
    //执行sql语句
    private void execSql(String sql) {
        try {
            ps = conn.prepareStatement(sql);
            LOG.info("exec::" + sql);
            ps.execute();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    //获取当前id的offset
    private Integer getStatusDBIndex(int startFrom) {
        //从flume_meta表中查询出当前的id是多少
        String dbIndex = queryOne("select currentIndex from flume_meta where source_tab='" + table + "'");
        if (dbIndex != null) {
            return Integer.parseInt(dbIndex);
        }
        //如果没有数据，则说明是第一次查询或者数据表中还没有存入数据，返回最初传入的值
        return startFrom;
    }
    
    //查询一条数据的执行语句(当前id)
    private String queryOne(String sql) {
        ResultSet result = null;
        try {
            ps = conn.prepareStatement(sql);
            result = ps.executeQuery();
            while (result.next()) {
                return result.getString(1);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
    
    //关闭相关资源
    void close() {
        try {
            ps.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
    int getCurrentIndex() {
        return currentIndex;
    }
    
    void setCurrentIndex(int newValue) {
        currentIndex = newValue;
    }
    
    int getRunQueryDelay() {
        return runQueryDelay;
    }
    
    String getQuery() {
        return query;
    }
    
    String getConnectionURL() {
        return connectionURL;
    }
    
    private boolean isCustomQuerySet() {
        return (customQuery != null);
    }
    
    Context getContext() {
        return context;
    }
    
    public String getConnectionUserName() {
        return connectionUserName;
    }
    
    public String getConnectionPassword() {
        return connectionPassword;
    }
    
    String getDefaultCharsetResultSet() {
        return defaultCharsetResultSet;
    }
}
4、MySQLSource
代码实现：
package com.atguigu.source;

import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.EventDeliveryException;
import org.apache.flume.PollableSource;
import org.apache.flume.conf.Configurable;
import org.apache.flume.event.SimpleEvent;
import org.apache.flume.source.AbstractSource;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.text.ParseException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

public class SQLSource extends AbstractSource implements Configurable, PollableSource {

    //打印日志
    private static final Logger LOG = LoggerFactory.getLogger(SQLSource.class);
    //定义sqlHelper
    private SQLSourceHelper sqlSourceHelper;


    @Override
    public long getBackOffSleepIncrement() {
        return 0;
    }
    
    @Override
    public long getMaxBackOffSleepInterval() {
        return 0;
    }
    
    @Override
    public void configure(Context context) {
        try {
            //初始化
            sqlSourceHelper = new SQLSourceHelper(context);
        } catch (ParseException e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public Status process() throws EventDeliveryException {
        try {
            //查询数据表
            List<List<Object>> result = sqlSourceHelper.executeQuery();
            //存放event的集合
            List<Event> events = new ArrayList<>();
            //存放event头集合
            HashMap<String, String> header = new HashMap<>();
            //如果有返回数据，则将数据封装为event
            if (!result.isEmpty()) {
                List<String> allRows = sqlSourceHelper.getAllRows(result);
                Event event = null;
                for (String row : allRows) {
                    event = new SimpleEvent();
                    event.setBody(row.getBytes());
                    event.setHeaders(header);
                    events.add(event);
                }
                //将event写入channel
                this.getChannelProcessor().processEventBatch(events);
                //更新数据表中的offset信息
                sqlSourceHelper.updateOffset2DB(result.size());
            }
            //等待时长
            Thread.sleep(sqlSourceHelper.getRunQueryDelay());
            return Status.READY;
        } catch (InterruptedException e) {
            LOG.error("Error procesing row", e);
            return Status.BACKOFF;
        }
    }
    
    @Override
    public synchronized void stop() {
        LOG.info("Stopping sql source {} ...", getName());
        try {
            //关闭资源
            sqlSourceHelper.close();
        } finally {
            super.stop();
        }
    }
}
7.2.5 测试
1、jar包准备
1) 将mysql驱动包放入flume的lib目录下
[atguigu@hadoop102 flume]$ cp \
/opt/sorfware/mysql-libs/mysql-connector-java-5.1.27/mysql-connector-java-5.1.27-bin.jar \
/opt/module/flume/lib/
2) 打包项目并将jar包放入flume的lib目录下
2、配置文件准备
1）创建配置文件并打开
[atguigu@hadoop102 job]$ touch mysql.conf
[atguigu@hadoop102 job]$ vim mysql.conf 
2）添加如下内容
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = com.atguigu.source.SQLSource  
a1.sources.r1.connection.url = jdbc:mysql://192.168.1.102:3306/mysqlsource
a1.sources.r1.connection.user = root  
a1.sources.r1.connection.password = 000000  
a1.sources.r1.table = student  
a1.sources.r1.columns.to.select = *  
#a1.sources.r1.incremental.column.name = id  
#a1.sources.r1.incremental.value = 0 
a1.sources.r1.run.query.delay=5000

# Describe the sink
a1.sinks.k1.type = logger

# Describe the channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
3、mysql表准备
1) 创建mysqlsource数据库
CREATE DATABASE mysqlsource；
2) 在mysqlsource数据库下创建数据表student和元数据表flume_meta
CREATE TABLE `student` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(255) NOT NULL,
PRIMARY KEY (`id`)
);
CREATE TABLE `flume_meta` (
`source_tab` varchar(255) NOT NULL,
`currentIndex` varchar(255) NOT NULL,
PRIMARY KEY (`source_tab`)
);
3)向数据表中添加数据
1 zhangsan
2 lisi
3 wangwu
4zhaoliu
4、测试并查看结果
1)任务执行
[atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name a1 \
--conf-file job/mysql.conf -Dflume.root.logger=INFO,console
2)结果展示，如图所示：

7.3 练习
案例需求：
1）flume-1监控hive.log日志，flume-1的数据传送给flume-2，flume-2将数据追加到本地文件，同时将数据传输到flume-3。
2）flume-4监控本地另一个自己创建的文件any.txt，并将数据传送给flume-3。
3）flume-3将汇总数据写入到HDFS。
请先画出结构图，再开始编写任务脚本。
第8章 企业真实面试题（重点）
8.1 你是如何实现Flume数据传输的监控的
使用第三方框架Ganglia实时监控Flume。
8.2 Flume的Source，Sink，Channel的作用？你们Source是什么类型？
	1、作用
（1）Source组件是专门用来收集数据的，可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy
（2）Channel组件对采集到的数据进行缓存，可以存放在Memory或File中。
（3）Sink组件是用于把数据发送到目的地的组件，目的地包括Hdfs、Logger、avro、thrift、ipc、file、Hbase、solr、自定义。
2、我公司采用的Source类型为：
（1）监控后台日志：exec
（2）监控后台产生日志的端口：netcat
Exec  spooldir
8.3 Flume的Channel Selectors

8.4 Flume参数调优
1. Source
    增加Source个数（使用Tair Dir Source时可增加FileGroups个数）可以增大Source的读取数据的能力。例如：当某一个目录产生的文件过多时需要将这个文件目录拆分成多个文件目录，同时配置好多个Source 以保证Source有足够的能力获取到新产生的数据。
    batchSize参数决定Source一次批量运输到Channel的event条数，适当调大这个参数可以提高Source搬运Event到Channel时的性能。
2. Channel 
    type 选择memory时Channel的性能最好，但是如果Flume进程意外挂掉可能会丢失数据。type选择file时Channel的容错性更好，但是性能上会比memory channel差。
    使用file Channel时dataDirs配置多个不同盘下的目录可以提高性能。
    Capacity 参数决定Channel可容纳最大的event条数。transactionCapacity 参数决定每次Source往channel里面写的最大event条数和每次Sink从channel里面读的最大event条数。transactionCapacity需要大于Source和Sink的batchSize参数。
3. Sink 
    增加Sink的个数可以增加Sink消费event的能力。Sink也不是越多越好够用就行，过多的Sink会占用系统资源，造成系统资源不必要的浪费。
    batchSize参数决定Sink一次批量从Channel读取的event条数，适当调大这个参数可以提高Sink从Channel搬出event的性能。
    8.5 Flume的事务机制
    Flume的事务机制（类似数据库的事务机制）：Flume使用两个独立的事务分别负责从Soucrce到Channel，以及从Channel到Sink的事件传递。比如spooling directory source 为文件的每一行创建一个事件，一旦事务中所有的事件全部传递到Channel且提交成功，那么Soucrce就将该文件标记为完成。同理，事务以类似的方式处理从Channel到Sink的传递过程，如果因为某种原因使得事件无法记录，那么事务将会回滚。且所有的事件都会保持到Channel中，等待重新传递。
    8.6 Flume采集数据会丢失吗?
    不会，Channel存储可以存储在File中，数据传输自身有事务。





## 其他

##### 查看端口是否被占用

```
netstat  -anp  |grep   端口号
netstat   -nultp | grep 端口号 
```





# Kafka





## kafka架构

![1551824579376](assets/1551824579376.png)

```
1.生产者
		producer
2.broker
		topic(主题)
			partition(分区)
				replication(副本)
					leader--读写
					follower--数据备份
3.消费者
		consumer(消费者组(多个消费者))
	消费者组中的每一个消费者可以消费同一个topic下的不同分区的数据，提高消费数据能力
4.zookeeper
		broker--存储topic，partition的元数据信息
		consumer--0.9版本之后放在本地，将offset存放在zookeeper中
```

```
分区的原因：
1.分布式存储
2. 提高消费者能力
3. 负载均衡
```



#### kafka配置安装

```
1.解压jar包
tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module/
2.修改解压后的文件名
mv kafka_2.11-0.11.0.0/ kafka
3. 在kafka中创建logs文件夹
 mkdir logs //logs中不仅存放日志还有数据
4.修改配置文件
vim server.properties
#broker的全局唯一编号，不能重复
broker.id=0  //不同节点的broker.id一定要不同
#删除topic功能使能
delete.topic.enable=true
#kafka运行日志存放的路径	
log.dirs=/opt/module/kafka/logs
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop105:2181
```

##### kafka集群操作

```
#启动集群
bin/kafka-server-start.sh config/server.properties 
#关闭集群
bin/kafka-server-stop.sh stop
```



#### kafka命令行操作

##### topic主题相关操作

```
1. 主题help 
bin/kafka-topics.sh -help
2. 查看当前主题
bin/kafka-topics.sh --zookeeper hadoop102:2181 --list
3.创建主题 //保证kafka集群已开启
bin/kafka-topics.sh --zookeeper hadoop102:2181 --create --topic sec --partitions 3 --replication-factor 2
4. 查看某个主题
 bin/kafka-topics.sh  --zookeeper hadoop102:2181 --describe --topic sec
5.删除topic
bin/kafka-topics.sh --zookeeper hadoop102:2181 --delete --topic first
```

##### 生产者相关操作

```
 bin/kafka-console-producer.sh --broker-list hadoop102:9092 --topic sec
```

##### 消费者相关

```
1.#普通消费方式--默认消费latest.当前时间最新的
bin/kafka-console-consumer.sh --zookeeper hadoop102:2181 --topic sec
2.#--from-beginning,从头开始消费
bin/kafka-console-consumer.sh --zookeeperadoop102:2181 --topic sec --from-beginning
3. bootstrap-server方式，0.9版本之后存放本地，以主题方式保存
bin/kafka-console-consumer.sh --bootstrap-server hadoop102:9092 --topic sec
```



## Kafka工作流程分析

















## 其他

##### 显示全类名

```
#显示jps全类名
jps -l
```
