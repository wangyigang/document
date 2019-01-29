### 大数据概论

###### hadoop是什么

1. Hadoop是一个由Apache基金会所开发的分布式系统架构
2. 主要解决，海量数据的存储和海量数据的分析计算问题
3. 广义上来说，Hadoop通常指一个更广泛的概念--hadoop生态圈

##### hadoop组成

###### Hadoop1.x和hadoop2.x 区别

Hadoop1.x 组成= MapReduce(计算+资源调度) HDFS(数据存储), Common(辅助工具)

Hadoop2.x组成=MapReduce(计算),Yarn(资源调度)，HDFS(数据存储),Common(辅助工具)

> 将Mapreduce进行拆分，解耦，Mapreduce只负责计算，由yarn负责资源调度，

![1548662415317](assets/1548662415317.png)

```
数据来源层->数据传输层->数据存储层->资源管理层->数据计算层->任务调度层->业务模型层
```

##### Hadoop运行环境搭配

###### 虚拟机环境准备

1. 修改虚拟机的静态IP(静态IP不会改变，易于管理)

2. 修改主机名

3. 关闭防火墙

4. 创建用户(以具体用户名为主)

5. 配置用户具有root权限

6. /opt目录下创建两个module, software文件夹

7. 安装JDK

   ```
   先查询是否安装java:rpm-qa | grep java
   如果有，卸载     : sudo rpm -e java
   查看jdk的安装路径 : which java
   将jdk安装包放入到/opt/software路径下:利用Xftp等传输工具
   解压jdk :  tar -zxvf jdkXXX -C /opt/module 
   配置环境变量： sudo vim /etc/profile
   在/etc/profile最后添加#JAVA_HOME相关信息
   	#JAVA_HOME
   	export JAVA_HOME=/opt/module/jdk1.8.0_144
   	export PATH=$PATH:$JAVA_HOME/bin
   保存退出，并source /etc/profile 进行刷新，使修改生效
   测试JDK是否安装成功： java -version /java /javac 出现相应提示即可
   ```

8. 安装Hadoop：

   ```
   Hadoop下载地址：(以hadoop2.7.2为例)
   	https://archive.apache.org/dist/hadoop/common/hadoop-2.7.2/
   解压到/opt/module路径下
   	tar -zxvf hadoopXXX -C /opt/module
   添加环境变量
   	##HADOOP_HOME
   	export HADOOP_HOME=/opt/module/hadoop-2.7.2
   	export PATH=$PATH:$HADOOP_HOME/bin  //将bin和sbin两个路径全部加入path
   	export PATH=$PATH:$HADOOP_HOME/sbin
   保存并刷新: source /etc/profile
   测试是否成功： hadoop version
   ```

###### Hadoop运行模式

> 本地模式，伪分布式模式，完全分布式模式

###### 完全分布式运行模式

配置集群

