## 请简述zookeeper的选举机制

- 半数机制（Paxos协议):集群中只要半数以上的机器存活，集群就能使用,所以zookeeper适合在基数台的机器上

- zookeeper 在配置中没有指定leader和follower,是在启动时，进行选举出来的

- 以三台服务器的选举过程为例：

  ```
  1.服务器1启动，进行一次选举，服务器1投自己一票，此时只有一票，不能达到半数以上要求，服务器1进入looking状态
  2. 服务器2启动，进行一次选举，服务器1投票自己，服务器2投票自己，然后交换选票信息，服务器1发现服务器2的ID比自己大，就该选为服务器2，此时服务器2两票，达到半数以上，服务器2为leader，服务器1为follower
  3.服务器3启动，发起一次选举，服务器1,2，都不是looking 状态，不会更改选票信息，服务器3服从多数，更改选票信息为服务器3，并更改状态为following
  ```

  ## Zookeeper的监听原理是什么

  1. 首先有一个main()线程
  2. 在main线程中创建zookeeper客户端，此时会创建两个线程，一个负责网络通信(connect)--sendthread，一个负责监听(listener)--enventThread
  3. 通过connect线程将注册的监听事件发送给zookeeper
  4. 在zookeeper的监听器列表中将注册的监听事件添加到列表中
  5. zookeeper监听到有数据或路径变化，就会将这个消息发送给listener线程
  6. listenner线程内部调用了process()方法

  ## zookeeper的部署方式有几种？集群中的角色有哪些？集群最少需要几台机器

  1. 单机模式，伪集群模式和集群模式
  2. leader和follower
  3. 3

## Hadoop

- 集群的最主要瓶颈：磁盘IO

###### 列举几个hadoop生态圈的组件并做简要描述

1.  zookeeper：开源的分布式应用程序协调服务，基于zookeeper可以实现同步服务，配置维护，命名服务

2. Flume:高可用的高可靠的，分布式的海量日志采集，聚合和传输的系统

3. Hbase：分布式的，面向列的开源数据库，利用hadoop HDFS作为其存储系统

4. Hive:基于Hadoop的一个数据仓库工具，可以将结构化的数据映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换成MapReduce任务进行运行

5. sqoop:将一个关系型数据库中的数据到进到Hadoop的HDFS中，可以将HDFS的数据到进到关系型数据库中   

   ## 简要描述如何安装hadoop                                            

1. 使用root账户登录
2. 修改IP
3. 修改host主机名
4. 配置ssh免密登录
5. 关闭防火墙
6. 安装jdk
7. 解压hadoop安装包
8. 配置hadoop核心文件：hadoop-env.sh core-site.xml mapred-site.xml hdfs-site.xml
9. 配置hadoop环境变量
10. 格式化hadoop namenode-format
11. 启动节点start-all.sh

