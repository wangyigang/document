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

- 虚拟机环境准备(见上)

###### 编写集群分发脚本xsync

- scp 安全拷贝

  定义：scp可以实现服务器之间的数据拷贝

  基本语法： scp -r  $dir/$fname   $user@$host:$pdir/$fname

  -  -r(递归)
  - $pdir/$fname 要拷贝的文件路径/名称
  - $user ： 远端用户@主机：目的路径/名称

```shell
scp -r /opt/module root@hadoop102:/opt/module
```

1. 如果是以root用户传输，需要修改文件的拥有者，所属组:sudo chown XXX:XXX -R  路径

###### rsync 远程同步工具

> rsync只对差异文件做更新，scp是将所有文件都赋值过去

1. 基本语法

   ```shell
   rsync -rvl $pdir/$fname  $user@$host:$pfir/$fname
   //-r 递归  -v 显示详细过程  -l 拷贝符号链接
   ```

 ###### 集群配置

|      | hadoop102         | hadoop103                   | hadoop104                  |
| ---- | ----------------- | --------------------------- | -------------------------- |
| HDFS | NameNode DataNode | DataNode                    | SecondaryNameNode DataNode |
| YARN | NodeManager       | ResourceManager NodeManager | NodeManager                |

###### 配置集群

1. 核心配置文件

   配置core-site.xml

   ```xml
   <!-- 指定HDFS中NameNode的地址 -->
   <property>
   		<name>fs.defaultFS</name>
         <value>hdfs://hadoop102:9000</value>
   </property>
   
   <!-- 指定Hadoop运行时产生文件的存储目录 -->
   <property>
   		<name>hadoop.tmp.dir</name>
   		<value>/opt/module/hadoop-2.7.2/data/tmp</value>
   </property>
   ```

2. HDFS配置文件

      配置hadoop-env.sh

      ```shell
      export JAVA_HOME=/opt/module/jdk1.8.0_144 //配置环境变量
      ```

      配置hdfs-site.xml

      ```xml
      <!-- 指定Hadoop副本个数，默认就是3 -->
      <property>
      		<name>dfs.replication</name>
      		<value>3</value>
      </property>
      
      <!-- 指定Hadoop辅助名称节点主机配置 -->
      <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>hadoop104:50090</value>
      </property>
      ```

3. YARN配置文件

     配置yarn-env.sh

     ```
     	vi yarn-env.sh
     	export JAVA_HOME=/opt/module/jdk1.8.0_144  //导出环境变量
     ```

     配置yarn-site.xml

     ```xml
     <!-- Reducer获取数据的方式 -->
     <property>
     		<name>yarn.nodemanager.aux-services</name>
     		<value>mapreduce_shuffle</value>
     </property>
     
     <!-- 指定YARN的ResourceManager的地址 -->
     <property>
     		<name>yarn.resourcemanager.hostname</name>
     		<value>hadoop103</value>
     </property>
     ```

4. MapReduce配置文件

     配置mapred-env.sh

     ```
     vi mapred-env.sh
     export JAVA_HOME=/opt/module/jdk1.8.0_144
     ```

     配置mapred-site.xml

     ```
     <!-- 指定MR运行在Yarn上 -->
     <property>
     		<name>mapreduce.framework.name</name>
     		<value>yarn</value>
     </property>
     ```

> 在集群上分发配置好的Hadoop配置文件

###### SSH无密码配置

配置ssh

基本语法： ssh 另一台机器的IP地址

1. 生成公钥和私钥：ssh-keygen -t rsa : 三次回车，就会生成公钥和私钥

2. 将公钥拷贝到免密登录的机器上 //或者直接拷贝整个文件也可以，不过较为危险



##### 群起集群

###### 配置salves

```
/opt/module/hadoop-2.7.2/etc/hadoop //在此路径下，建立slaves文件夹
vim slaves
```

添加以下内容(不允许有空格，或空行) --配置完成后，进行分发

```
hadoop102
hadoop103
hadoop104
```

###### 启动集群

1. 如果是第一次启动，需要格式化NameNode(1.注意进程 2. 删除data和log数据3. 再进行格式化)

   ```
   bin/hdfs namenode -format
   ```

2. 启动HDFS

   ```shell
   sbin/start-dfs.sh
   ```

3. 启动YARN

   ```shell
   start-yarn.sh
   ```

4. Web端进行查看

   ```
   http://hadoop201:50070
   ```

   

##### 集群启动/停止方式总结

###### 各个服务分别启动/停止

```shell
hadoop-daemon.sh start/stop namenode/datanode/secondarynamenode
```

###### 启动、停止YARN

```shell
yarn-daemon.sh start/stop resourcemanager/nodemanager
```

###### 各个模块启动/停止-- 常用

整体启动/停止HDFS

```shell
start-dfs.sh /stop-dfs.sh
```

整体启动/停止YARN

```shell
start-yarn.sh/ stop-yarn.sh
```



##### 集群事件同步

###### 配置时间同步具体步骤(root用户)

1. 检查ntp是否安装

   ```shell
   rpm -qa | grep ntp
   ```

2. 修改ntp配置文件

   修改1：授权XXX网段上的所有机器都可以从这台机器上查询和同步时间

   ```shell
   #restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap为
   restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
   ```

   修改2： 集群在局域网中，不适用其他互联网上的时间

   ```shell
   server 0.centos.pool.ntp.org iburst
   server 1.centos.pool.ntp.org iburst
   server 2.centos.pool.ntp.org iburst
   server 3.centos.pool.ntp.org iburst为
   #server 0.centos.pool.ntp.org iburst
   #server 1.centos.pool.ntp.org iburst
   #server 2.centos.pool.ntp.org iburst
   #server 3.centos.pool.ntp.org iburst //将这四个部分注释掉
   ```

   修改3： 当该节点丢失网络连接，扔可以采用本地时间作为时间服务器为其他节点提供时间同步

   ```
   server 127.127.1.0
   fudge 127.127.1.0 stratum 10
   ```

3. 修改/etc/sysconfig/ntpd文件

   ```
   vim /etc/sysconfig/ntpd
   增加内容如下（让硬件时间与系统时间一起同步）
   SYNC_HWCLOCK=yes
   ```

   

4. 重新启动ntpd服务

   ```
   service ntpd start
   ```

5. 设置ntpd服务开机启动

   ```
   chkconfig ntpd on
   ```

###### 其他机器配置(必须root用户)

​	其他机器每10分钟与时间服务器同步一次

```
crontab -e
*/10 * * * * /usr/sbin/ntpdate hadoop102  //每隔10分钟进行时间同步
```





## Hadoop之HDFS

###### HDFS产出背景

> 随着数据量的越来越大，一个操作系统中存不下所有的数据，那么就分配到更多的操作系统磁盘中，为了方便管理，需要一种系统来管理多台机器上的文件，这就是分布式文件管理系统，HDFS只是分布式文件管理系统中的一种

###### HDFS定义

> HDFS(hadoop Distributed File System) ,分布式的文件系统，用于存储文件，多台服务器联合实现其功能

**使用场景**: 适合一次写入，多次读出的场景，且不支持文件的修改,适合做数据分析



##### HDFS优缺点

###### 优点

- 高容错性: 数据自动保存多个副本，通过增加副本的方式，提高容错性
- 适合处理大数据
- 可构建在廉价的机器上，通过多副本机制，提高可靠性

###### 缺点

- 不适合低延时数据访问：效率偏低
- 无法高效的对小文件进行存储
- 不支持并发写入，文件随机修改

##### HDFS组成架构

![1549094434671](assets/1549094434671.png)

- NameNode(nn):  Master
  - 管理HDFS的名称空间
  - 配置副本策略
  - 管理数据块(Block)映射信息
  - 处理客户端读写请求

- DataNode: Slave

  - 存储实际的数据块

  - 执行数据块的读/写操作

- Client : 客户端
  - 文件切分，文件上传HDFS时，客户端将文件切分成一个一个的Block，然后进行上传
  - 与NameNode交互，获取文件的位置信息
  - 与DataNode交互，读取或写入数据
  - Client提供一些命令管理HDFS， ex: namenode格式化
  - Client通过一些命令访问HDFS，比如HDFS增删改查等刀座
- Secondary NameNode： 非NameNode热备
  - 辅助NamNode，分担工作： 定时合并FsImage和Edits,并推送给NamNode
  - 紧急情况下，辅助修复NamNode



##### HDFS文件块大小

​	HDFS中文件在物理上是分块存储(Block)，块的大小可以通过配置参数(dfs.blocksize)来规定，默认大小Hadoop2.x版本中128M

![1549096200149](assets/1549096200149.png)

###### 问什么块的大小不能设置太小，也不能太大?

1. HDFS块设置太小，会增加寻址时间，增加寻找块的开始位置时间

2. 如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间，导致处理数据时，相对较慢

   **总结：HDFS块的大小设置主要取决于磁盘传输速率**

   

##### HDFS的shell 操作

###### 基本语法

```
bin/hadoop fs XX命令 或   bin/hdfs dfs XXX命令
```

###### 常用命令

- 启动Hadoop集群

```shell
sbin/start-dfs.sh
sbin/start-yarn.sh
```

- `-`help : 输出这个命令参数

```
hadoop fs -help rm
```

- `-`ls: 显示目录信息

```
hadoop fs -ls /
```

- `-`mkdir: 在HDFS上创建目录

```
hadoop fs -mkdir -p /wangyg/love/pangdi
```

- `-`moveFromLocal : 从本地剪切到HDFS上

```
hadoop fs -moveFromLoacl ./1.txt /wangyg
```

- `-`appendToFile: 追加一个文件到已经存在的文件末尾

```
hadoop fs -appendToFile 1.txt /wangyg/1.txt
```

- `-`cat: 显示文件内容
- `-` chgrp `-`chmod `-`chown:  修改文件所属权限
- `-`copyFromLocal : 从本地拷贝到HDFS中
- `-`copyToLocal : 从HDFS拷贝到本地
- `-`cp : 从HDFS的一个路径拷贝到另一个路径
- `-`mv : 在HDFS目录中移动文件
- `-`get: 等同于copyToLoacal : 从HDFS下载
- `-`getmerget: 合并下载多个文件
- `-`put: 等同于copuFromLocal : 上传
- `-`rm : 删除文件或文件夹
- `-`rmdir : 删除空目录
- `-`du: 统计文件夹的大小信息
- `-`setrep :设置HDFS中文件的副本数量



##### HDFS客户端操作

##### HDFS客户端环境准备：

1. 将编译后的相应版本的Hadoop jar包放在一个位置中(无中文，空格)

![1549098907950](assets/1549098907950.png)

2. 配置HADOOP_HOME环境变量

![1549098966957](assets/1549098966957.png)

3. 再将HADOOP_HOME添加到path中

4. 创建一个maven工程
5. 添加maven依赖

```xml
<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.8.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.hadoop</groupId>
			<artifactId>hadoop-common</artifactId>
			<version>2.7.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.hadoop</groupId>
			<artifactId>hadoop-client</artifactId>
			<version>2.7.2</version>
		</dependency>
		<dependency>
			<groupId>org.apache.hadoop</groupId>
			<artifactId>hadoop-hdfs</artifactId>
			<version>2.7.2</version>
		</dependency>
		<dependency>
			<groupId>jdk.tools</groupId>
			<artifactId>jdk.tools</artifactId>
			<version>1.8</version>
			<scope>system</scope>
			<systemPath>${JAVA_HOME}/lib/tools.jar</systemPath>
		</dependency>
</dependencies>
```

- 创建目录--mkdir

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.junit.Test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

public class HdfsClient {
    @Test
    public void testMkdir() throws URISyntaxException, IOException, InterruptedException {
        //获取文件系统
        Configuration conf = new Configuration();
        //配置在集群上运行
        conf.set("fs.defaultFS", "hdfs://hadoop201:9000");

        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop201:9000"), conf, "wangyg");

        // 2 创建目录
        fs.mkdirs(new Path("/wangyg/pangdi"));

        // 3 关闭资源
        fs.close();
    }
}
```



##### HDFS的API操作

###### HDFS文件上传

```
@Test
    public void testCopyFromLocalFile() throws IOException, URISyntaxException, InterruptedException {
        //获取文件系统
        Configuration conf = new Configuration();
        conf.set("dfs.replication", "2"); //设置副本个数
        //获取文件系统
        FileSystem fileSystem = FileSystem.get(new URI("hdfs://hadoop201:9000"),conf,"wangyg");
        //上传文件
        fileSystem.copyFromLocalFile(new Path("C:\\Users\\wangyg\\Desktop\\TODO.txt"),
                new Path("/wangyg"));

        //上传完毕后，关闭资源
        fileSystem.close();

        System.out.println("上传完毕...");
    }
```



> 参数优先级: 客户端代码中的值 > ClassPath下的配置> 服务器的默认配置



###### HDFS文件下载

```
    @Test
    public void testCopyToLocalFile() throws URISyntaxException, IOException, InterruptedException {
        //获取文件系统
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop201:9000"),
                conf, "wangyg");
        fs.copyToLocalFile(false,new Path("/wangyg/TODO.txt")
                , new Path("D:\\"));

        System.out.println("下载完成...");

    }
```

###### HDFS文件夹删除

```
 @Test
    public void testDelete() throws URISyntaxException, IOException, InterruptedException {
        //获取配置
        Configuration conf = new Configuration();
        //获取文件系统
        FileSystem fs = FileSystem.get(new URI("hdfs://hadoop201:9000"), conf, "wangyg");
        boolean delete = fs.delete(new Path("/wangyg/"), true);
        if (delete) {
            System.out.println("删除成功...");
        }
        //关闭资源
        fs.close();

    }
```

###### HDFS文件名更改

```java
  @Before
    public void before() throws URISyntaxException, IOException, InterruptedException {
        //获取配置
        Configuration conf = new Configuration();
        //获取文件系统
        fs = FileSystem.get(new URI("hdfs://hadoop201:9000"), conf, "wangyg");
    }
    //HDFS文件名更改
    @Test
    public void testRename() throws IOException {
        boolean rename = fs.rename(new Path("/1.txt"), new Path("/2.txt"));

        if(rename){
            System.out.println("修改名称成功...");
        }
        //关闭资源
        fs.close();
    }
```

###### HDFS文件详情查看

```
 //HDFS文件详情查看
    @Test
    public void testListFiles() throws IOException {
        RemoteIterator<LocatedFileStatus> listFiles = fs.listFiles(new Path("/"), true);
        while (listFiles.hasNext()) {
            LocatedFileStatus next = listFiles.next();
            //文件名称--输出详情
            System.out.println(next.getPath().getName());
            //长度
            System.out.println(next.getLen());
            //权限
            System.out.println(next.getPermission());
            //获取分组
            System.out.println(next.getGroup());

            //获取存储的块信息
            BlockLocation[] blockLocations = next.getBlockLocations();
            for (BlockLocation blockLocation : blockLocations) {
                //获取块存储的主节点
                String[] hosts = blockLocation.getHosts();
                for (String host : hosts) {
                    System.out.println(host);
                }
            }
        }
        //关闭资源
        fs.close();

    }
```

###### HDFS文件和文件夹判断

```java
  //hdfs文件和文件夹判断
    @Test
    public void testListStatus() throws IOException {
        FileStatus[] listStatus = fs.listStatus(new Path("/"));
        for (FileStatus fileStatus : listStatus) {
            //如果是文件
            if(fileStatus.isFile()){
                System.out.println("f:"+fileStatus.getPath().getName());
            }else{//文件夹
                System.out.println("d:"+fileStatus.getPath().getName());
            }
        }
    }
```



### HDFS的数据流

##### HDFS写数据流程

![1549115215477](assets/1549115215477.png)

1. 客户端通过Distributed FileSystem模块向Namenode请求上传文件，NameNode检查目标文件是否存在
2. NamNode返回是否可以上传
3. 客户端请求第一个Block上传到那几个DataNode上
4. NamNode返回三个DataNode节点，分别是dn1, dn2,dn3
5. 客户端通过FsDataOutputStream模块请求dn1 上传数据，dn1收到请求继续调用dn2,dn2调用dn3，将通信管道建立完成
6. dn1,dn2,dn3逐级应答客户端
7. 客户端开始向dn1 上传第一个Block，以Packet为单位，dn1收到一个packet(64K)就会传给dn2,dn2传给dn3,dn1每传一个packet会让如一个应答队列等待应答
8. 当一个Block传输完成之后，客户端再次请求NamNode上传第二个Block的服务器(重复操作)



###### 网络拓扑-节点距离计算

> 节点距离： 两个节点到达最近的共同祖先的距离总和

###### 机架感知(副本存储节点选择)

```
1. 第一个副本在client所处节点上(本机)--目的是效率方面考虑
2. 第二个副本在同机架不同节点
3. 第三个是不同机架的不同节点上
```

###### HDFS读数据流程

![1549116069280](assets/1549116069280.png)

1. 客户端通过Distributed FIleSystem 向NamNode请求下载文件，NameNode通过查询元数据，吵到文件所在的DataNode地址
2. 挑选一台DataNode(就近原则)服务器，请求读取数据
3. DataNode开始传输数据给客户端(以Packet为单位来做校验)
4. 客户端以Packet为单位接收，先在本地缓存，然后写入目标文件

##### NamNode和SecondaryNameNode

###### NN和2NN工作机制

> 如果元数据只放入内存中，一旦断电，元数据就会丢失==》 引入FsImage
>
> 当元数据更新时，更新FsImage，效率过低 ==》引入Edits log(只进行追加操作，效率高)
>
> 引入一个新的节点SecondaryNamNode，专门用于FsImage和Edits合并

![1549116504151](assets/1549116504151.png)

###### 第一阶段： NamNode启动

1. 第一次启动NamNode格式化后，创建Fsimage和Edits文件，如果不是第一次启动，直接加载编辑日志和镜像文件到内存
2. 客户端对元数据进行增删改的请求
3. NamNode记录操作日志，更新滚动日志Edits
4. namenode对内存中的数据进行增删改操作

###### 第二阶段： secondary NameNode工作

1.  2NN 询问NameNode是否需要CheckPoint，直接带回namenode是否检查结果
2. 2NN请求执行checkpoin
3. NameNode滚动正在写的Edits日志
4. 将滚动前的边记日志和镜像文件(Fsimage) 拷贝到2NN中
5. 2NN加载边记日志和镜像文件，并进行合并
6. 生成新的镜像文件Fsimage.chkpoint
7. 拷贝Fsimage.chkpoint到NamNode
8. NamNode将fsimage.chkpoint重新命名成fsimage



###### Fsimage和Edits解析

> oiv查看Fsimage文件

```
基本语法： hdfs oiv -p 文件类型 -i 镜像文件 -o 转换后的文件输出路径
hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-2.7.2/fsimage.xml
```

> oev查看Edits文件

```
基本语法：hdfs oev -p 文件类型 -i 边记日志 -o 转换后文件输出路径
hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-2.7.2/edits.xml
```

###### CheckPoint时间设置

[hdfs-default.xml]

- 2nn每隔一小时执行一次

```xml
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>3600</value>
</property>
```

- 一分钟检查一次操作次数  :当操作次数达到一百万时，2nn执行一次

```xml
<property>
  <name>dfs.namenode.checkpoint.txns</name>
  <value>1000000</value>
<description>操作动作次数</description>
</property>

<property>
  <name>dfs.namenode.checkpoint.check.period</name>
  <value>60</value>
<description> 1分钟检查一次操作次数</description>
</property >
```



##### NameNode故障处理

###### 方法一：

> 将2nn中数据拷贝到Namenode存储目录

1. kill -9 namenode进程
2. 删除namenode存储的数据(/opt/module/hadoop-2.7.2/data/tmp/dfs/name)

```
rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/name/*
```

3. 拷贝2NN中数据到原Namenode存储数据目录

```shell
scp -r wangyg@hadoop201:/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary/* ./name/
```

4. 重新启动Namenode

```shell
sbin/hadoop-daemon.sh start namenode
```

###### 方法二： 

> 使用-importCheckpoint选项启动NameNode守护进程，从而将2NN中数据拷贝到Namenode目录中

1. 修改hdfs-site.xml

```xml
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>120</value>
</property>

<property>
  <name>dfs.namenode.name.dir</name>
  <value>/opt/module/hadoop-2.7.2/data/tmp/dfs/name</value>
</property>
```

2. kill -9 NameNode进程
3. 删除NameNode存储的数据

```
rm -rf /opt/module/hadoop-2.7.2/data/tmp/dfs/name/*
```

4. 如果SecondaryNameNode不和NameNode在一个主机节点上，需要将SecondaryNameNode存储数据的目录拷贝到NameNode存储数据的平级目录，并删除in_use.lock文件

```
scp -r wangyg@hadoop104:/opt/module/hadoop-2.7.2/data/tmp/dfs/namesecondary ./
 rm -rf in_use.lock
```

5. 导入检查点数据(等待一会ctrl +c结束掉)

```
bin/hdfs namenode -importCheckpoint
```

6. 启动NameNode

```
sbin/hadoop-daemon.sh start namenode
```



##### 集群安全模式

![1549156642052](assets/1549156642052.png)

###### 基本语法

集群处于安全模式，不能执行重要操作(写操作)，集群启动完成后，自动退出安全模式

- bin/hdfs dfsadmin -safemode get : 查看安全模式状态
- bin/hdfs dfsadmin -safemode enter: 进入安全模式
- bin/hdfs dfsadmin -safemode leave: 离开安全模式
- bin/hdfs dfsadmin -safemode wait : 等待安全模式状态



##### DataNode工作机制

![1549156820769](assets/1549156820769.png)

1. 一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个数据本身，一个是元数据包括数据块的长度，块数据校验和，以及时间戳
2. DataNode启动后，向Namenode进行注册，通过后，周期性(1小时)向Namenode上报所有的块信息
3. 心跳是每3秒一次，心跳返回结果带有NamNode给该DataNode的命令 ，如赋值数据到另一台机器，或删除某个数据块，如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用
4. 集群运行中可以安全加入和退出一些机器

###### DataNode掉线时限参数设置

```
TimeOut = 2* dfs.namenode.heartbeat.recheck-interval +10*dfs.heartbeat.interval // 10分钟 30s
```

```xml
<property>
    <name>dfs.namenode.heartbeat.recheck-interval</name>
    <value>300000</value>
</property>
<property>
    <name>dfs.heartbeat.interval</name>
    <value>3</value>
</property>
```

###### 服役新节点

1. 环境准备

   一个新的主机hadoop105

   修改IP地址和主机名称

   格式化(删除原有残留文件): /opt/module/hadoop/data/log

   source /etc/profile

2. 服役新节点具体步骤

   1. 直接启动DataNode，即可关联到集群

   ```shell
   sbin/hadoop-daemon.sh start datanode
   sbin/hadoop-daemon.sh start nodemanager
   ```

   2. 如果数据不平衡，可用命令实现集群的再平衡

   ```
   ./start-balancer.sh 
   ```

   ##### 退役旧数据节点

   ###### 添加白名单

   > 添加白名单的主机节点，都允许访问namenode,不在白名单的主机节点，都会被退出

   1. 在NameNode的/opt/module/hadoop-2.7.2/etc/hadoop目录下创建dfs.hosts文件

   ```shell
   pwd            #/opt/module/hadoop-2.7.2/etc/hadoop
   touch dfs.hosts
   vim dfs.hosts
   ```

   ​	添加如下主机名称

   ```
   hadoop201
   hadoop202
   hadoop203
   ```

   2. 在NamNode的hdfs-site.xml配置文件中增加dfs.hosts属性

   ```xml
   <property>
   <name>dfs.hosts</name>
   <value>/opt/module/hadoop-2.7.2/etc/hadoop/dfs.hosts</value>
   </property>
   ```

   3. 配置文件分发

   ```
   scp分发
   ```

   4. 刷新NameNode

   ```
   hdfs dfsadmin -refreshNodes
   ```

   5. 更新ResourceManager节点

   ```
   yarn rmadmin -refreshNodes
   ```

   6. 在web浏览器上查看

   7. 如果数据不均衡，可用命令时限集群的在平衡

   ```
   ./start-balancer.sh
   ```

   ###### 黑名单退役

   > 黑名单上面的主机都会被强制退出

   1. 在NameNode的 /opt/module/hadoop-2.7.2/etc/hadoop 目录下创建dfs.hosts.exclude文件

   ```shell
   pwd        #/opt/module/hadoop-2.7.2/etc/hadoop
   touch dfs.hosts.exclude
   vim dfs.hosts.exclude
   ```

   2. 在namenode的hdfs-site.xml配置文件中增加dfs.hosts.exclude属性

   ```xml
   <property>
   <name>dfs.hosts.exclude</name>
   <value>/opt/module/hadoop-2.7.2/etc/hadoop/dfs.hosts.exclude</value>
   </property>
   ```

   3. 刷新namenode， 刷新ResourceManager

   ```shell
   hdfs dfsadmin -refreshNodes  //刷新namenode
   yarn rmadmin -refreshNodes  //刷新resoucemanager
   ```

   4. Web浏览器端检查--等待退役节点状态改为decommissioned 后

      ```shell
      sbin/hadoop-daemon.sh stop datanode
      sbin/hadoop-daemon.sh stop nodemanager
      ```

   5. 若数据不平衡，使用sbin/start-balancer.sh



##### HDFS HA高可用

> 双NamNode消除单点故障

- 内存中各自保存一份元数据、

- Edits日志只有Active状态的NameNode节点可以做写操作

  **TODO**





## Hadoop 之 MapReduce

###### MapReduce定义

> 分布式运算程序的编程框架，是用户开发 基于hadoop的数据分析应用的和新框架

> MapReduce核心功能是将用户编写的业务逻辑代码和自带默认组件 整合成一个完整的分布式运算程序，并发运行在一个Hadoop集群上

###### MapReduce优缺点

###### 优点：

1. MapReduce易于编程
2. 良好的 扩展性--计算资源不够可以增加机器
3. 高容错性： 一台机器挂了，可以把上面的计算任务转移到另外一个节点上运行，不至于把这个任务运行失败

4. 适合PB级以上海量数据的离线处理

###### 缺点：

1. 不擅长实时计算:
2. 不擅长流式计算: 流式计算的输入数据是动态的,mapreduce的输入数据是静态的
3. 不擅长DAG(有向图)计算



##### MapReduce核心思想

- 第一个阶段的MapTask并发实例，完全并行运行，互不相干
- 第二个阶段的ReduceTask并发实例不互相干，他们的数据依赖于上一个阶段的所有MapTask的输出

###### MapReduce进程

一个完整的MapReduce程序有三类实例进程

- MrAppMaster: 负责整个程序的过程调度及状态协调
- MapTask： 负责map阶段的数据处理流程
- ReduceTask： 负责Reduce阶段的数据处理流程



###### 常用数据序列化类型

| **Java类型** | **Hadoop Writable类型** |
| ------------ | ----------------------- |
| boolean      | BooleanWritable         |
| byte         | ByteWritable            |
| int          | IntWritable             |
| float        | FloatWritable           |
| long         | LongWritable            |
| double       | DoubleWritable          |
| String       | Text                    |
| map          | MapWritable             |
| array        | ArrayWritable           |

###### MapReduce编程规范

> Mapper  Reducer和Driver

Mapper阶段

```
1. 用户自定义的Mapper要继承自己的父类
2. Mapper的输入数据时K V对的形式(K V的类型可自定义)
3. Mapper中业务逻辑写在map()方法中
4.Mapper的输出类型kv对形式
5. map()方法对每一个<k,v>调用一次
```

###### Reducer阶段

```
1. 用户自定义的Reducer要继承自己的父类
2. Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
3. Reducer的业务逻辑写在reduce()方法中
4. ReduceTask进程对每一组相同的<KV>组调用一次reduce()方法
```

###### Driver阶段

​	封装MapReduce相关参数，job对象设置

###### WordCount案例实操

需求： 给定的文本文件中统计输出一个单词出现的总次数

```java
//Mapper阶段
public class WordCountMapper extends Mapper<LongWritable,Text,Text,IntWritable> {
    private Text word = new Text();
    private IntWritable one = new IntWritable(1);

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //获取一行
        String string = value.toString();
        //以空格进行切分
        String[] split = string.split(" ");
        //输出
        for (String word : split) {
            this.word.set(word);
            context.write(this.word,one);
        }
    }
}
```

编写Reducer类

```java

public class WordCountReducer extends Reducer<Text, IntWritable,Text, IntWritable >{
    private int sum=0;
    private IntWritable v = new IntWritable();
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        sum=0;
        for (IntWritable value : values) {
            sum+= value.get();
        }
        
        //输出
        v.set(sum);
        context.write(key, v);
    }
}
```

编写Driver驱动类

```java
public class WordCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        //获取job实例
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);

        //设置路径
        job.setJarByClass(WordCountDriver.class);
        //设置mapper
        job.setMapperClass(WordCountMapper.class);
        //设置reducer
        job.setReducerClass(WordCountReducer.class);
        //设置map阶段和reducer阶段的输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        //设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        //提交job等待运行
        boolean b = job.waitForCompletion(true);
        System.out.println(b);
        //三目运算符
        System.exit(b?0:1);
    }
}
```

添加编译

```xml
<build>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.3.2</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<artifactId>maven-assembly-plugin </artifactId>
				<configuration>
					<descriptorRefs>
						<descriptorRef>jar-with-dependencies</descriptorRef>
					</descriptorRefs>
					<archive>
						<manifest>
							<mainClass>XXXX.driver</mainClass>
						</manifest>
					</archive>
				</configuration>
				<executions>
					<execution>
						<id>make-assembly</id>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
```



##### Hadoop序列化

> 序列化就是把内存中的对象，转换成字节序列，以便存储，持久化和网络传输

> 反序列化就是把字节序列或磁盘的持久化数据，转换成内存中的对象

目的：便于将字节序列在网络中传输

Haoop序列化特点：

- 紧凑：高效实用存储空间
- 快速： 读写数据的额外开销小
- 可扩展： 随着通信协议的升级而升级
- 互操作： 支持多语言的交互

###### 自定义bean对象实现序列化接口(Writable)

1. 必须实现W日table接口

2. 反序列化，需要反射调用空参构造函数，所以必须有空参构造

   ```
   public FlowBean() {
   	super();
   }
   ```

3. 重写序列化方法

   ```
   @Override
   public void write(DataOutput out) throws IOException {
   	out.writeLong(upFlow);
   	out.writeLong(downFlow);
   	out.writeLong(sumFlow);
   }
   ```

4. 重写反序列化方法

   ```
   @Override
   public void readFields(DataInput in) throws IOException {
   	upFlow = in.readLong();
   	downFlow = in.readLong();
   	sumFlow = in.readLong();
   }
   ```

5. 注意反序列化的顺序和序列化的顺序要完全相同

6. 要想把结果显示在文件中，需要重写toString() , 可用"\t"分开

7. 如果自定义bean类型要放在key位置传输，需要实现Comparable接口，会对key进行排序

###### 序列化案例

> 需求：同一每一个手机号的总上行流量，下行流量，总流量

1. bean对象

```
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class FlowBean  implements Writable {
    private long upFlow;
    private long downFlow;
    private long sumFlow;

    //空参构造
    public FlowBean() {
        super();
    }

    public FlowBean(long upFlow, long downFlow) {
        this.upFlow = upFlow;
        this.downFlow = downFlow;
        this.sumFlow = upFlow+ downFlow;
    }
    //toString方法
    @Override
    public String toString() {
        return upFlow +
                "\t" + downFlow +
                "\t" + sumFlow ;
    }

    public long getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public long getDownFlow() {
        return downFlow;
    }

    public void setDownFlow(long downFlow) {
        this.downFlow = downFlow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }

    //序列化方法
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(upFlow);
        out.writeLong(downFlow);
        out.writeLong(sumFlow);
    }
    //反序列化方法
    @Override
    public void readFields(DataInput in) throws IOException {
        this.upFlow = in.readLong();
        this.downFlow = in.readLong();
        this.sumFlow = in.readLong();
    }
}
```

2. Mapper类

```java

public class FlowCountReducer extends Reducer<Text,FlowBean, Text,FlowBean> {

    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {
        long sumUpFlow =0;
        long sumDownFlow=0;
        for (FlowBean value : values) {
            sumUpFlow+= value.getUpFlow();
            sumDownFlow+= value.getDownFlow();
        }
        //封装对象
        FlowBean flowBean = new FlowBean(sumUpFlow, sumDownFlow);
        context.write(key, flowBean);
        
    }
}
```

3. 编写Driver驱动类

```java

public class FlowCountDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        //设置类路径
        job.setJarByClass(FlowCountDriver.class);
        //指定本业务job要使用的map
        job.setMapperClass(FlowCountMapper.class);
        //指定job的reduce类
        job.setReducerClass(FlowCountReducer.class);
        //设置map阶段输出类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(FlowBean.class);
        //设置reduce阶段输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);
        //设置job输入路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        //设置job的输出路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        //提交Job,并运行
        boolean b = job.waitForCompletion(true);
        System.out.println(b);
        
    }
}
```



#### MapReduce框架原理

##### 切片与MapTask并行决定机制

###### 切块与切片：

数据块：Block是HDFS物理上把数据分成一块一块

数据切片：数据切片只是在逻辑上对输入进行分片，并不会在磁盘上切分，只是逻辑意义

- 一个job的Map阶段并行度由客户端在提交job时切片数决定
- 每一个切片分配一个Maptask并行实例处理
- 默认情况下，切片大小=blocksize
- 切片不考虑数据集整体，而是针对每一个文件单独切片



###### Job提交流程源码和切片源码详解

```java
waitForCompletion()

submit();

// 1建立连接
	connect();	
		// 1）创建提交Job的代理
		new Cluster(getConfiguration());
			// （1）判断是本地yarn还是远程
			initialize(jobTrackAddr, conf); 

// 2 提交job
submitter.submitJobInternal(Job.this, cluster)
	// 1）创建给集群提交数据的Stag路径
	Path jobStagingArea = JobSubmissionFiles.getStagingDir(cluster, conf);

	// 2）获取jobid ，并创建Job路径
	JobID jobId = submitClient.getNewJobID();

	// 3）拷贝jar包到集群
copyAndConfigureFiles(job, submitJobDir);	
	rUploader.uploadFiles(job, jobSubmitDir);

// 4）计算切片，生成切片规划文件
writeSplits(job, submitJobDir);
		maps = writeNewSplits(job, jobSubmitDir);
		input.getSplits(job);

// 5）向Stag路径写XML配置文件
writeConf(conf, submitJobFile);
	conf.writeXml(out);

// 6）提交Job,返回提交状态
status = submitClient.submitJob(jobId, submitJobDir.toString(), job.getCredentials());
```

![1549202815569](assets/1549202815569.png)

###### FileInputFormat切片机制：

切片机制：

- 简单的按照文件的内容长度进行切分，切分大小，默认等于Block大小
- 切片时不考虑数据集整体，而是逐个对每一个文件单独切片



#####  CombineTextInputFormat切片机制

> 背景
>
> 框架默认的TextInputFormat切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个MapTask，这样如果有大量小文件，就会产生大量的MapTask，处理效率极其低下。

- 应用场景：
  **CombineTextInputFormat用于小文件过多的场景**，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个MapTask处理。

- 虚拟存储切片最大值设置

  ```
  CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m
  ```

  注意：虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

- 切片机制(分为两部分)

  - 虚拟存储过程
  - 切片过程二部分。

  ![1549606773116](assets/1549606773116.png)

- 虚拟存储过程：
  将输入目录下所有文件大小，依次和设置的setMaxInputSplitSize值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值2倍，此时将文件均分成2个虚拟存储块（防止出现太小切片）。
  例如setMaxInputSplitSize值为4M，输入文件大小为8.02M，则先逻辑上分成一个4M。剩余的大小为4.02M，如果按照4M逻辑划分，就会出现0.02M的小的虚拟存储文件，所以将剩余的4.02M文件切分成（2.01M和2.01M）两个文件。

- 切片过程：
  （a）判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片。
  （b）**如果不大于则跟下一个虚拟存储文件进行合并**，共同形成一个切片。
  （c）测试举例：有4个小文件大小分别为1.7M、5.1M、3.4M以及6.8M这四个小文件，则虚拟存储之后形成6个文件块，大小分别为：
  1.7M，（2.55M、2.55M），3.4M以及（3.4M、3.4M）
  最终会形成3个切片，大小分别为：
  （1.7+2.55）M，（2.55+3.4）M，（3.4+3.4）M

  

  #### CombineTextInputFormat案例实操

  > 需求：将输入的大量小文件合并成一个切片统一处理。

- 输入数据
  准备4个小文件

- 期望: 期望一个切片处理4个文件

  

  实现过程

  1. 不做任何处理，运行1.6节的WordCount案例程序，观察切片个数为4。

  2. 在WordcountDriver中增加如下代码，运行程序，并观察运行的切片个数为3。

     ```
     // 如果不设置InputFormat，它默认用的是TextInputFormat.class
     job.setInputFormatClass(CombineTextInputFormat.class);
     
     //虚拟存储切片最大值设置4m
     CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);
     
     运行如果为3个切片。
     ```

3.  在WordcountDriver中增加如下代码，运行程序，并观察运行的切片个数为1。

      （a）驱动中添加代码如下：
      // 如果不设置InputFormat，它默认用的是TextInputFormat.class
        job.setInputFormatClass(CombineTextInputFormat.class);

      //虚拟存储切片最大值设置20m
      CombineTextInputFormat.setMaxInputSplitSize(job, 20971520);
      （b）运行如果为1个切片。

#### FileInputFormat实现类

##### FileInputFormat常见的接口实现类：

```
TextInputFormat： 默认
KeyValueTextInputFormat
NLineInputFormat
CombineTextInputFormat
自定义InputFormat等
```

###### TextInputFormat:

> 默认的FileInputFormat实现类，按行读取每条记录
>
> key是存储行在整个文件中的起始字节偏移量，LongWritable类型
>
> value值是这行的内容，不包括终止符(回车换行符)， Text类型

###### KeyValueTextInputFormat

> 每一行为一条记录，通过分隔符分隔为key, value 可以通过Driver中设置conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR,'\t');设置分隔符，默认分隔符就是tab ('\t')



###### KeyValueTextInputFormat使用案例

需求:统计输入文件中每一行的第一个单词相同的行数。

```
//输入数据
banzhang ni hao
xihuan hadoop banzhang
banzhang ni hao
xihuan hadoop banzhang
```

```
//期望结果数据
banzhang	2
xihuan	2
```

- code

```java
package keyvalueTextInputFormat;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.KeyValueLineRecordReader;
import org.apache.hadoop.mapreduce.lib.input.KeyValueTextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class KVTextDriver {
    //主方法
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        args = new String[]{"",""};
        //设置conf
        Configuration conf = new Configuration();
        //设置分隔符
        conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR, " ");

        Job job = Job.getInstance(conf);

        //设置输入格式
        job.setInputFormatClass(KeyValueTextInputFormat.class);

        job.setJarByClass(KVTextDriver.class);

        job.setMapperClass(KVTextMapper.class);
        job.setReducerClass(KVTextReducer.class);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        boolean b = job.waitForCompletion(true);

        System.out.println(b);
    }
}
```



```java
package keyvalueTextInputFormat;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import java.io.IOException;
//
public class KVTextMapper extends Mapper<Text, Text,Text, IntWritable> {
    private IntWritable v = new IntWritable(1);

    @Override
    protected void map(Text key, Text value, Context context) throws IOException, InterruptedException {
        //1. 封装对象
        //什么也不需要做, 使用keyvalueTextInputFormat的话就key直接是第一个
        //写出
        context.write(key, v);

    }
}
```



```java
package keyvalueTextInputFormat;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class KVTextReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    private IntWritable v = new IntWritable();


    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum=0;
        for (IntWritable value : values) {
            sum+= value.get();
        }
        v.set(sum);
        context.write(key, v);
    }
}
```



###### NLineInputFormat

> 每个map进程处理的InputSplit不在按Block块去划分，而是按NLineInputFormat指定的行数N来划分

##### NLineInputFormat使用案例

> 需求：对每个单词进行个数统计，要求根据每个输入文件的行数来规定输出多少个切片。此案例要求每三行放入一个切片中。



- 输入数据

  ```
  banzhang ni hao
  xihuan hadoop banzhang
  banzhang ni hao
  xihuan hadoop banzhang
  banzhang ni hao
  xihuan hadoop banzhang
  banzhang ni hao
  xihuan hadoop banzhang
  banzhang ni hao
  xihuan hadoop banzhang banzhang ni hao
  xihuan hadoop banzhang
  ```

- 期望输出数据

  ```
  Number of splits:4
  ```

  ###### code代码实现

  ```
  package NLineInputFormat;
  
  import org.apache.hadoop.conf.Configuration;
  import org.apache.hadoop.fs.Path;
  import org.apache.hadoop.io.LongWritable;
  import org.apache.hadoop.io.Text;
  import org.apache.hadoop.mapreduce.Job;
  import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
  import org.apache.hadoop.mapreduce.lib.input.NLineInputFormat;
  import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
  
  import java.io.IOException;
  
  public class NLineInputDriver {
      public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
          // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
          args = new String[] { "e:/input/inputword", "e:/output1" };
  
          // 1 获取job对象
          Configuration configuration = new Configuration();
          Job job = Job.getInstance(configuration);
  
          //7设置每个切片InputSplit中划分三条记录
          NLineInputFormat.setNumLinesPerSplit(job, 3);
  
          // 8使用NLineInputFormat处理记录数
          job.setInputFormatClass(NLineInputFormat.class);
  
          // 2设置jar包位置，关联mapper和reducer
          job.setJarByClass(NLineInputDriver.class);
          job.setMapperClass(NLineInputMapper.class);
          job.setReducerClass(NLineInputReducer.class);
  
          // 3设置map输出kv类型
          job.setMapOutputKeyClass(Text.class);
          job.setMapOutputValueClass(LongWritable.class);
  
          // 4设置最终输出kv类型
          job.setOutputKeyClass(Text.class);
          job.setOutputValueClass(LongWritable.class);
  
          // 5设置输入输出数据路径
          FileInputFormat.setInputPaths(job, new Path(args[0]));
          FileOutputFormat.setOutputPath(job, new Path(args[1]));
  
          // 6提交job
          boolean b = job.waitForCompletion(true);
          System.out.println(b);
      }
  }
  ```



```
package NLineInputFormat;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class NLineInputMapper extends Mapper<LongWritable,Text,Text,LongWritable> {
    private Text k = new Text();
    private LongWritable v = new LongWritable(1);
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //获取一行
        String string = value.toString();
        //切割
        String[] split = string.split(" ");
        //循环写出
        for (String word : split) {
            k.set(word);
            context.write(k, v);
        }

    }
}

```



```java
package NLineInputFormat;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class NLineInputReducer extends Reducer<Text, IntWritable,Text, IntWritable > {
    private IntWritable v = new IntWritable();
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        //累加求和
        int sum=0;
        for (IntWritable value : values) {
            sum+= value.get();
        }
        v.set(sum);
        context.write(key,v );
    }
}
```

#### 自定义InputFormat

> Hadoop自带的inputFormat类型不能满足所有场景，需要自定义InputFormat来解决实际问题

- 自定义InputFormat步骤

```
1. 自定义一个类继承FileInputFormat
2. 改写RecordReader， 实现一次读取一个完整文件封装为KV
3. 输出时使用sequenceFileOutPutFormat输出合并文件
```

#### 自定义InputFormat案例

> 可以自定义InputFormat实现小文件的合并。
> 需求:将多个小文件合并成一个SequenceFile文件（SequenceFile文件是Hadoop用来存储二进制形式的key-value对的文件格式），SequenceFile里面存储着多个文件，存储的形式为文件路径+名称为key，文件内容为value。

- code

```
package WholeFileInputformat;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.SequenceFileOutputFormat;

import java.io.IOException;

public class WholeFileInputDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
        // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[] { "d:/input/inputinputformat", "d:/output1" };

        //获取job实例
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);
        //设置类路径
        job.setJarByClass(WholeFileInputDriver.class);
        //设置inputformat

        job.setInputFormatClass(WholeFileInputFormat.class);
        //设置outputformat
        job.setOutputFormatClass(SequenceFileOutputFormat.class);

        //设置mapper和Reducer
        job.setMapperClass(WholeFileInputMapper.class);
        job.setReducerClass(WholeFileInputReducer.class);

        //设置map和reduce输出格式
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(BytesWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(BytesWritable.class);
        //设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 6 提交job
        boolean result = job.waitForCompletion(true);
        System.out.println(result);
        System.exit(result ? 0 : 1);
    }
}

```



```
package WholeFileInputformat;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;

import java.io.IOException;

public class WholeFileInputFormat extends FileInputFormat<Text, BytesWritable> {

    @Override
    public RecordReader<Text, BytesWritable> createRecordReader(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
        WholeRecordReader wholeRecordReader = new WholeRecordReader();
        //调用
      //  wholeRecordReader.initialize(split, context);
        return wholeRecordReader;
    }

    /**
     * 是否切割的判断--重写后不进行切割
     * @param context
     * @param filename
     * @return
     */
    @Override
    protected boolean isSplitable(JobContext context, Path filename) {
        return false;
    }
}

```



```
package WholeFileInputformat;

import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/*
继承FileInputFormat
    重写isSplitable()方法，返回false 不可切割
    重写cretateRedordReader(),创建自定义的RecordReader对象

改写RedcordReader

Driver驱动设置为sewqunenceFIle
 */
public class WholeFileInputMapper extends Mapper<Text, BytesWritable, Text, BytesWritable> {

    @Override
    protected void map(Text key, BytesWritable value, Context context) throws IOException, InterruptedException {
        context.write(key, value);
    }
}

```



```
package WholeFileInputformat;

import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WholeFileInputReducer extends Reducer<Text, BytesWritable,Text, BytesWritable> {
    @Override
    protected void reduce(Text key, Iterable<BytesWritable> values, Context context) throws IOException, InterruptedException {
        for (BytesWritable value : values) {
            context.write(key, value);
        }
    }
}

```

```
package WholeFileInputformat;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

import java.io.IOException;

public class WholeRecordReader extends RecordReader<Text, BytesWritable> {
    //标记位--flag为true,表示文件没有去读，false-表示读完
    private boolean flag = true;
    private Configuration conf=null;
    private FileSplit split = null;
    private Text k = new Text();
    private BytesWritable v = new BytesWritable();
    private FSDataInputStream fs ;
    /**
     *  初始化方法--框架会自动调用一次
     * @param split
     * @param context
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public void initialize(InputSplit split, TaskAttemptContext context) throws IOException, InterruptedException {
        //获取文件切片--强转
        this.split = (FileSplit) split;
        //获取配置信息
        this.conf = context.getConfiguration();
        //读取数据--开流准备
        Path path =(this.split).getPath();
//        FileSystem fileSystem = FileSystem.get(context.getConfiguration());
        FileSystem fileSystem = path.getFileSystem(context.getConfiguration());
        this.fs = fileSystem.open(path);

    }

    /**
     * 核心方法--读取下一组key value值
     * @return 读到返回true,否则返回false
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public boolean nextKeyValue() throws IOException, InterruptedException {
        if(flag){
            //读取文件
            byte[] bytes = new byte[(int) split.getLength()];
            fs.read(bytes);
            String path = split.getPath().toString();
            v.set(bytes, 0, bytes.length);
            k.set(path);

            flag =false;
            return true;
        }

        return false;
    }

    /**
     *  获取当前key
     * @return
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public Text getCurrentKey() throws IOException, InterruptedException {
        return k;
    }

    @Override
    public BytesWritable getCurrentValue() throws IOException, InterruptedException {
        return v;
    }

    /**
     *  获取当前进度
     * @return
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public float getProgress() throws IOException, InterruptedException {
        return flag? 0:1;
    }

    /**
     * 关闭方法
     * @throws IOException
     */
    @Override
    public void close() throws IOException {
        IOUtils.closeStream(fs);
    }
}

```



### MapReduce工作流程

![1549695242721](assets/1549695242721.png)

- 先判断在哪个分区中，然后将数据进行序列化，写入到环形缓冲去中

![1549695295249](assets/1549695295249.png)

- reduce阶段：

  - 有几个分区启动几个ReduceTask,每个分区从全部的MapTask进行拷贝相应分区数据

  - 合并文件，归并排序

    

```
具体Shuffle过程详解，如下：
1. MapTask收集我们的map()方法输出的kv对，放到内存缓冲区中
2. 从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件
3. 多个溢出文件会被合并成大的溢出文件
4. 在溢出过程及合并的过程中，都要调用Partitioner进行分区和针对key进行排序
5. ReduceTask根据自己的分区号，去各个MapTask机器上取相应的结果分区数据
6. ReduceTask会取到同一个分区的来自不同MapTask的结果文件，ReduceTask会将这些文件再进行合并（归并排序）
7. 合并成大文件后，Shuffle的过程也就结束了，后面进入ReduceTask的逻辑运算过程（从文件中取出一个一个的键值对Group，调用用户自定义的reduce()方法）
```

- 注意

  ```
  Shuffle中的缓冲区大小会影响到MapReduce程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。
  缓冲区的大小可以通过参数调整，参数：io.sort.mb默认100M
  ```

  

- 4．源码解析流程

  ```
  context.write(k, NullWritable.get());
  output.write(key, value);
  collector.collect(key, value,partitioner.getPartition(key, value, partitions));
  	HashPartitioner();
  collect()
  	close()
  	collect.flush()
  sortAndSpill()
  	sort()   QuickSort
  mergeParts();
  	
  collector.close();
  ```

  


#### Shuffle机制

> Map方法之后，Reduce方法之前的数据处理过程称为Shuffle

![1549695955541](assets/1549695955541.png)

##### Partition分区

> 要求将统计结果按照条件输出到不同的文件中(分区中)

###### 默认Partition分区

```
public class HashPartitioner<K2, V2> implements Partitioner<K2, V2>{
	public int getPartition(K2 key, V2 value,
                          int numReduceTasks) {
    return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
  	}
  	//默认分区是根据key的hashCode对ReduceTasks个数取模得到的
```

> 默认分区是根据key的hashCode对ReduceTasks个数取模得到的



##### 自定义Partitioner分区

步骤

- 自定义类继承Partitioner ,重写getPartition()方法
- Job驱动中设置自定义Partitioner
- 自定义Partition后，根据自定义Partition的逻辑设置相应数量的ReduceTask
  - job.setNumReduceTasks(X)

###### Partition分区案例

> 需求:将统计结果按照手机归属地不同省份输出到不同文件中（分区）

> 期望输出数据:	手机号136、137、138、139开头都分别放到一个独立的4个文件中，其他开头的放到一个文件中。

![1549701316932](assets/1549701316932.png)

- code

```
package Partition;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;
@SuppressWarnings("all")
public class FlowDriver {
    public static void main(String[] args) {
        args = new String[]{"D:\\input\\phone.txt","D:\\output"};

        //设置job和configuration
        Configuration conf = new Configuration();
        try {

            Job job = Job.getInstance(conf);
            //设置jar路径
            job.setJarByClass(FlowDriver.class);
            //设置map和reduce
            job.setMapperClass(FlowMapper.class);
            job.setReducerClass(FlowReducer.class);
            //设置map阶段输出类型
            job.setMapOutputKeyClass(Text.class);
            job.setMapOutputValueClass(FlowBean.class);
            //设置reduce阶段输出类型
            job.setOutputKeyClass(Text.class);
            job.setOutputValueClass(FlowBean.class);
            //设置输入输出参数路径
            FileInputFormat.setInputPaths(job, new Path(args[0]));
            FileOutputFormat.setOutputPath(job, new Path(args[1]));

            ////增加自定义数据分区设置和reduceTask数量设置
            job.setPartitionerClass(ProvincePartitioner.class);
            job.setNumReduceTasks(5);

            //提交job
            boolean b = job.waitForCompletion(true);
            System.out.println(b);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

```

```
package Partition;

import org.apache.hadoop.io.Writable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class FlowBean implements Writable {
    private long upFlow;
    private long downFlkow;
    private long sumFlow;

    //无参构造
    public FlowBean() {
    }
    //设置方法
    public void set(long upFlow, long downFlkow){
        this.upFlow = upFlow;
        this.downFlkow = downFlkow;
        this.sumFlow = this.upFlow+ this.downFlkow;
    }

    public long getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public long getDownFlkow() {
        return downFlkow;
    }

    public void setDownFlkow(long downFlkow) {
        this.downFlkow = downFlkow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }

    @Override
    public String toString() {
        return  upFlow +
                " " + downFlkow +
                " " + sumFlow;
    }

    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(upFlow);
        out.writeLong(downFlkow);
        out.writeLong(sumFlow);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        upFlow = in.readLong();
        downFlkow=in.readLong();
        sumFlow = in.readLong();
    }
}

```

```
package Partition;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

@SuppressWarnings("all")
public class FlowMapper  extends Mapper<LongWritable, Text, Text, FlowBean> {
    Text k = new Text();
    FlowBean v = new FlowBean();

    //重写map方法
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {

        /**
         * 7 	13560436666	120.196.100.99		1116		 954			200
         * id	手机号码		网络ip			   上行流量     下行流量     网络状态码
         */
        //获取一行
        String line = value.toString();
        //以空格切割 获取手机号 上流量 下流量
        String[] strings = line.split("\t");
        //获取手机号，上流量，下流量
        String phone = strings[1];
        String upflow = strings[strings.length-3];//上行流量
        String downflow = strings[strings.length-2]; //下行流量

        //封装对象
        k.set(phone);
        v.set(Long.parseLong(upflow), Long.parseLong(downflow));

        //写出
        context.write(k, v);
    }
}

```

```
package Partition;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
@SuppressWarnings("all")
public class FlowReducer extends Reducer<Text, FlowBean, Text, FlowBean> {
    private FlowBean sumFlow = new FlowBean();
    @Override
    protected void reduce(Text key, Iterable<FlowBean> values, Context context) throws IOException, InterruptedException {
        long upSum=0;
        long downSum =0;
        for (FlowBean value : values) {
            upSum += value.getUpFlow();
            downSum+= value.getDownFlkow();

            sumFlow.set(upSum, downSum);
            context.write(key, sumFlow);
        }

    }
}

```



```
package Partition;

import io.netty.handler.codec.marshalling.ThreadLocalMarshallerProvider;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

public class ProvincePartitioner extends Partitioner<Text, FlowBean> {

    @Override
    public int getPartition(Text text, FlowBean flowBean, int numPartitions) {
        String preNum = text.toString().substring(0, 3);
        int partition=4;

        if("136".equals(preNum)){
            partition=0;
        }else if("137".equals(preNum)){
            partition=1;
        }else if("138".equals(preNum)){
            partition =2;
        }else if("139".equals(preNum)){
            partition=3;
        }
        return  partition;
    }
}

```

```
总结：
1. ReduceTaks个数为1时，自定义分区无效 
2. 设置的ReduceTask个数<分区的结果，报出IO异常
3. ReduceTask个数> 分区的数 ，会有一部分空数据文件
```



#### WritableComparable排序

> 排序是MapReduce框架中最重要的操作之一
>
> MapTask和ReduceTask都会对数据按照**key**进行排序，任何应用程序中的数据都会被排序，而不管逻辑上是否需要

- 默认按照**字典顺序排序**，且排序方法**快速排序**
- MapTask : 环形缓冲区达到阈值后，对缓冲区中数据**进行一次快速排序**，溢写到磁盘中后，会**对磁盘上所有数据进行归并排序**
- ReduceTask： **从每个MapTask上拷贝相应的数据文件，磁盘上的数量达到一定的阈值，进行一次归并排序生成更大的文件**，当**所有数据拷贝完后，ReduceTask统一对内存和磁盘上的所有数据进行一次归并排序**

###### 排序的分类

- 部分排序
  - MapReduce根据数据的 键key 对数据集排序，保证输出的每个文件内部有序
- 全排序
  - 最终的输出结果只有一个文件，且文件内部有序， 实现方式：只设置一个ReduceTask，缺点： 效率低
- 辅助排序(Grouping Comparator分组)
  - 对Reduce端对key进行分组

- 二次排序
  - 在自定义排序过程中，如果compareTo中判断条件为两个即为二次排序



##### 自定义排序WritableComparable

> 原理：bean对象做为key传输，需要实现**WritableComparable接口**重写**compareTo**方法，就可以实现排序。

```java
@Override
public int compareTo(FlowBean o) {

	int result;
		
	// 按照总流量大小，倒序排列
	if (sumFlow > bean.getSumFlow()) {
		result = -1;
	}else if (sumFlow < bean.getSumFlow()) {
		result = 1;
	}else {
		result = 0;
	}

	return result;
}
```



##### WritableComparable排序案例(全排序)

> 需求：FlowBean产生的结果再次对总流量进行排序。



```
期望输出数据--对数据结果进行降序排序
13509468723	7335	110349	117684
13736230513	2481	24681	27162
13956435636	132		1512	1644
13846544121	264		0		264
```

```
FlowBean实现WritebaleCOmparable接口重写compareTO方法:按照倒叙排列
Mapper类： 输出context.write(bean,手机号) //以bean对象作为key使排列
Reducer类：循环写出
```



```
//FlowBean
package FlowCountSorted;

import org.apache.hadoop.io.WritableComparable;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

public class FlowBean implements WritableComparable<FlowBean> {

    private long upFlow;
    private long downFLow;
    private long sumFlow;

    /**
     * 默认无参构造
     */
    public FlowBean() {
    }

    /**
     * 有参构造
     *
     * @param upFlow
     * @param downFLow
     */
    public FlowBean(long upFlow, long downFLow) {
        set(upFlow, downFLow);
    }

    public void set(long upFlow, long downFLow) {
        this.upFlow = upFlow;
        this.downFLow = downFLow;
        this.sumFlow = this.upFlow + this.downFLow;
    }

    /**
     * compareTo方法
     *
     * @param o
     * @return
     */
    @Override
    public int compareTo(FlowBean o) {
        //按照总流量的大小进行排序
        if (this.sumFlow > o.sumFlow) {
            return -1;
        } else if (sumFlow < o.sumFlow) {
            return 1;
        } else {
            return 0;
        }
    }


    public long getUpFlow() {
        return upFlow;
    }

    public void setUpFlow(long upFlow) {
        this.upFlow = upFlow;
    }

    public long getDownFLow() {
        return downFLow;
    }

    public void setDownFLow(long downFLow) {
        this.downFLow = downFLow;
    }

    public long getSumFlow() {
        return sumFlow;
    }

    public void setSumFlow(long sumFlow) {
        this.sumFlow = sumFlow;
    }

    @Override
    public String toString() {
        return upFlow +
                "\t" + downFLow +
                "\t" + sumFlow;
    }

    /**
     * 序列化方法
     *
     * @param out
     * @throws IOException
     */
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(upFlow);
        out.writeLong(downFLow);
        out.writeLong(sumFlow);
    }

    /**
     * 反序列化方法
     *
     * @param in
     * @throws IOException
     */
    @Override
    public void readFields(DataInput in) throws IOException {
        //顺数保持一致即可
        this.upFlow = in.readLong();
        this.downFLow = in.readLong();
        this.sumFlow = in.readLong();
    }
}
```

```
package FlowCountSorted;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import java.io.IOException;

public class FlowBeanSortDriver {
    public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {

        // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[]{"d:/output1","d:/output2"};

        // 1 获取配置信息，或者job对象实例
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration);

        // 2 指定本程序的jar包所在的本地路径
        job.setJarByClass(FlowBeanSortDriver.class);

        // 3 指定本业务job要使用的mapper/Reducer业务类
        job.setMapperClass(FlowBeanSortMapper.class);
        job.setReducerClass(FlowBeanSortReducer.class);

        // 4 指定mapper输出数据的kv类型
        job.setMapOutputKeyClass(FlowBean.class);
        job.setMapOutputValueClass(Text.class);

        // 5 指定最终输出的数据的kv类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(FlowBean.class);

        // 6 指定job的输入原始文件所在目录
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
        boolean result = job.waitForCompletion(true);
        System.out.println(result);
        System.exit(result ? 0 : 1);
    }
}
```

```
package FlowCountSorted;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class FlowBeanSortMapper extends Mapper<LongWritable,Text,FlowBean,Text> {
    private FlowBean k = new FlowBean();
    private Text v = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        //原始数据：手机号  上行流量  下行流量  总流量
        String line = value.toString();

        //切割
        String[] split = line.split("\t");

        //获取对应数据
        String phoneNum = split[0];

        long upFlow = Long.parseLong(split[1]);
        long downFlow = Long.parseLong(split[2]);
        long sumFlow = Long.parseLong(split[3]);

        //封装对象
        k.setDownFLow(downFlow);
        k.setUpFlow(upFlow);
        k.setSumFlow(sumFlow);

        v.set(phoneNum);

        context.write(k, v);

    }
}

```



```
package FlowCountSorted;


import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class FlowBeanSortReducer extends Reducer<FlowBean, Text, Text, FlowBean> {
    @Override
    protected void reduce(FlowBean key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        for (Text text : values) {
            context.write(text, key);
        }
    }
}
```



#### WritableComparable排序案例（区内排序）

> 需求：要求每个省份手机号输出的文件中按照总流量内部排序。

> 需求分析：基于前一个需求，增加自定义分区类，分区按照省份手机号设置。



- code

```
//增加自定义分区类
public class ProvincePartitioner extends Partitioner<FlowBean, Text> {

	@Override
	public int getPartition(FlowBean key, Text value, int numPartitions) {
		
		// 1 获取手机号码前三位
		String preNum = value.toString().substring(0, 3);
		
		int partition = 4;
		
		// 2 根据手机号归属地设置分区
		if ("136".equals(preNum)) {
			partition = 0;
		}else if ("137".equals(preNum)) {
			partition = 1;
		}else if ("138".equals(preNum)) {
			partition = 2;
		}else if ("139".equals(preNum)) {
			partition = 3;
		}

		return partition;
	}
}
```

- 在驱动类中设置分区类和ReduceTask个数

```
// 加载自定义分区类
job.setPartitionerClass(ProvincePartitioner.class);

// 设置Reducetask个数
job.setNumReduceTasks(5);
```



#### Combiner合并

> 需求：统计过程中对每一个MapTask的输出进行局部汇总，以减小网络传输量即采用Combiner功能。

期望：Combine输入数据多，输出时经过合并，输出数据降低。



- code

```
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordcountCombiner extends Reducer<Text, IntWritable, Text, IntWritable>{
IntWritable v = new IntWritable();
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        // 1 汇总
		int sum = 0;
		for(IntWritable value :values){
			sum += value.get();
		}
		v.set(sum);
		// 2 写出
		context.write(key, v);
	}
}
```

- 在WordcountDriver驱动中指定Combiner

```
// 指定需要使用combiner，以及用哪个类作为combiner的逻辑
job.setCombinerClass(WordcountCombiner.class);
```

##### 案例二

将WordcountReducer作为Combiner在WordcountDriver驱动类中指定

```
// 指定需要使用Combiner，以及用哪个类作为Combiner的逻辑
job.setCombinerClass(WordcountReducer.class);
```



#### GroupingComparator分组（辅助排序）

> 对Reduce阶段的数据根据某一个或几个字段进行分组。

- 分组排序步骤：

  ```
  自定义类继承WritableComparator
  重写compare()方法
  ```


  （3）创建一个构造将比较对象的类传给父类
  protected OrderGroupingComparator() {
  		super(OrderBean.class, true);
  }
  3.3.10 GroupingComparator分组案例实操
  1．需求
  有如下订单数据
  表4-2 订单数据
  订单id	商品id	成交金额
  0000001	Pdt_01	222.8
  	Pdt_02	33.8
  0000002	Pdt_03	522.8
  	Pdt_04	122.4
  	Pdt_05	722.4
  0000003	Pdt_06	232.8
  	Pdt_02	33.8
  现在需要求出每一个订单中最贵的商品。
  （1）输入数据

  ```
  1.自定义类继承WritableComparator
  2. 重写compare()方法
  @Override
  public int compare(WritableComparable a, WritableComparable b) {
  		// 比较的业务逻辑
  		return result;
  }
  3. 创建一个构造将比较对象的类传给父类
  protected OrderGroupingComparator() {
  		super(OrderBean.class, true);
  }
  ```

  

  


  1．需求
  有如下订单数据
  表4-2 订单数据
  订单id	商品id	成交金额
  0000001	Pdt_01	222.8
  	Pdt_02	33.8
  0000002	Pdt_03	522.8
  	Pdt_04	122.4
  	Pdt_05	722.4
  0000003	Pdt_06	232.8
  	Pdt_02	33.8
  现在需要求出每一个订单中最贵的商品。
  （1）输入数据

（2）期望输出数据
1	222.8
2	722.4
3	232.8
2．需求分析
（1）利用“订单id和成交金额”作为key，可以将Map阶段读取到的所有订单数据按照id升序排序，如果id相同再按照金额降序排序，发送到Reduce。
（2）在Reduce端利用groupingComparator将订单id相同的kv聚合成组，然后取第一个即是该订单中最贵商品，如图4-18所示。

图4-18 过程分析
3．代码实现
（1）定义订单信息OrderBean类
package com.atguigu.mapreduce.order;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.WritableComparable;

public class OrderBean implements WritableComparable<OrderBean> {

	private int order_id; // 订单id号
	private double price; // 价格
	
	public OrderBean() {
		super();
	}
	
	public OrderBean(int order_id, double price) {
		super();
		this.order_id = order_id;
		this.price = price;
	}
	
	@Override
	public void write(DataOutput out) throws IOException {
		out.writeInt(order_id);
		out.writeDouble(price);
	}
	
	@Override
	public void readFields(DataInput in) throws IOException {
		order_id = in.readInt();
		price = in.readDouble();
	}
	
	@Override
	public String toString() {
		return order_id + "\t" + price;
	}
	
	public int getOrder_id() {
		return order_id;
	}
	
	public void setOrder_id(int order_id) {
		this.order_id = order_id;
	}
	
	public double getPrice() {
		return price;
	}
	
	public void setPrice(double price) {
		this.price = price;
	}
	
	// 二次排序
	@Override
	public int compareTo(OrderBean o) {
	
		int result;
	
		if (order_id > o.getOrder_id()) {
			result = 1;
		} else if (order_id < o.getOrder_id()) {
			result = -1;
		} else {
			// 价格倒序排序
			result = price > o.getPrice() ? -1 : 1;
		}
	
		return result;
	}
}
（2）编写OrderSortMapper类
package com.atguigu.mapreduce.order;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class OrderMapper extends Mapper<LongWritable, Text, OrderBean, NullWritable> {

	OrderBean k = new OrderBean();
	
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		
		// 1 获取一行
		String line = value.toString();
		
		// 2 截取
		String[] fields = line.split("\t");
		
		// 3 封装对象
		k.setOrder_id(Integer.parseInt(fields[0]));
		k.setPrice(Double.parseDouble(fields[2]));
		
		// 4 写出
		context.write(k, NullWritable.get());
	}
}
（3）编写OrderSortGroupingComparator类
package com.atguigu.mapreduce.order;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.io.WritableComparator;

public class OrderGroupingComparator extends WritableComparator {

	protected OrderGroupingComparator() {
		super(OrderBean.class, true);
	}
	
	@Override
	public int compare(WritableComparable a, WritableComparable b) {
	
		OrderBean aBean = (OrderBean) a;
		OrderBean bBean = (OrderBean) b;
	
		int result;
		if (aBean.getOrder_id() > bBean.getOrder_id()) {
			result = 1;
		} else if (aBean.getOrder_id() < bBean.getOrder_id()) {
			result = -1;
		} else {
			result = 0;
		}
	
		return result;
	}
}
（4）编写OrderSortReducer类
package com.atguigu.mapreduce.order;
import java.io.IOException;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Reducer;

public class OrderReducer extends Reducer<OrderBean, NullWritable, OrderBean, NullWritable> {

	@Override
	protected void reduce(OrderBean key, Iterable<NullWritable> values, Context context)		throws IOException, InterruptedException {
		
		context.write(key, NullWritable.get());
	}
}
（5）编写OrderSortDriver类
package com.atguigu.mapreduce.order;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class OrderDriver {

	public static void main(String[] args) throws Exception, IOException {

// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
		args  = new String[]{"e:/input/inputorder" , "e:/output1"};

		// 1 获取配置信息
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
	
		// 2 设置jar包加载路径
		job.setJarByClass(OrderDriver.class);
	
		// 3 加载map/reduce类
		job.setMapperClass(OrderMapper.class);
		job.setReducerClass(OrderReducer.class);
	
		// 4 设置map输出数据key和value类型
		job.setMapOutputKeyClass(OrderBean.class);
		job.setMapOutputValueClass(NullWritable.class);
	
		// 5 设置最终输出数据的key和value类型
		job.setOutputKeyClass(OrderBean.class);
		job.setOutputValueClass(NullWritable.class);
	
		// 6 设置输入数据和输出数据路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		// 8 设置reduce端的分组
	job.setGroupingComparatorClass(OrderGroupingComparator.class);
	
		// 7 提交
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
3.4 MapTask工作机制
MapTask工作机制如图4-12所示。

图4-12  MapTask工作机制
	（1）Read阶段：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。
	（2）Map阶段：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。
	（3）Collect收集阶段：在用户编写map()函数中，当数据处理完成后，一般会调用OutputCollector.collect()输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
	（4）Spill阶段：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。
	溢写阶段详情：
	步骤1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号Partition进行排序，然后按照key进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照key有序。
	步骤2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件output/spillN.out（N表示当前溢写次数）中。如果用户设置了Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。
	步骤3：将分区数据的元信息写到内存索引数据结构SpillRecord中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过1MB，则将内存索引写到文件output/spillN.out.index中。
	（5）Combine阶段：当所有数据处理完成后，MapTask对所有临时文件进行一次合并，以确保最终只会生成一个数据文件。
	当所有数据处理完后，MapTask会将所有临时文件合并成一个大文件，并保存到文件output/file.out中，同时生成相应的索引文件output/file.out.index。
	在进行文件合并过程中，MapTask以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并io.sort.factor（默认10）个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。
	让每个MapTask最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。
3.5 ReduceTask工作机制
1．ReduceTask工作机制
ReduceTask工作机制，如图4-19所示。

图4-19 ReduceTask工作机制
	（1）Copy阶段：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
	（2）Merge阶段：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。
	（3）Sort阶段：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。
	（4）Reduce阶段：reduce()函数将计算结果写到HDFS上。
2．设置ReduceTask并行度（个数）
ReduceTask的并行度同样影响整个Job的执行并发度和执行效率，但与MapTask的并发数由切片数决定不同，ReduceTask数量的决定是可以直接手动设置：
// 默认值是1，手动设置为4
job.setNumReduceTasks(4);
3．实验：测试ReduceTask多少合适
（1）实验环境：1个Master节点，16个Slave节点：CPU:8GHZ，内存: 2G
（2）实验结论：
表4-3 改变ReduceTask （数据量为1GB）
MapTask =16
ReduceTask	1	5	10	15	16	20	25	30	45	60
总时间	892	146	110	92	88	100	128	101	145	104
4．注意事项

3.6 OutputFormat数据输出
3.6.1 OutputFormat接口实现类

3.6.2 自定义OutputFormat

3.6.3 自定义OutputFormat案例实操
1．需求
	过滤输入的log日志，包含atguigu的网站输出到e:/atguigu.log，不包含atguigu的网站输出到e:/other.log。
（1）输入数据

（2）期望输出数据

2．需求分析

3．案例实操
（1）编写FilterMapper类
package com.atguigu.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class FilterMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
	
		// 写出
		context.write(value, NullWritable.get());
	}
}
（2）编写FilterReducer类
package com.atguigu.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class FilterReducer extends Reducer<Text, NullWritable, Text, NullWritable> {

Text k = new Text();

	@Override
	protected void reduce(Text key, Iterable<NullWritable> values, Context context)		throws IOException, InterruptedException {
	
	   // 1 获取一行
		String line = key.toString();
	
	   // 2 拼接
		line = line + "\r\n";
	
	   // 3 设置key
	   k.set(line);
	
	   // 4 输出
		context.write(k, NullWritable.get());
	}
}
（3）自定义一个OutputFormat类
package com.atguigu.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class FilterOutputFormat extends FileOutputFormat<Text, NullWritable>{

	@Override
	public RecordWriter<Text, NullWritable> getRecordWriter(TaskAttemptContext job)			throws IOException, InterruptedException {
	
		// 创建一个RecordWriter
		return new FilterRecordWriter(job);
	}
}
（4）编写RecordWriter类
package com.atguigu.mapreduce.outputformat;
import java.io.IOException;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.RecordWriter;
import org.apache.hadoop.mapreduce.TaskAttemptContext;

public class FilterRecordWriter extends RecordWriter<Text, NullWritable> {

	FSDataOutputStream atguiguOut = null;
	FSDataOutputStream otherOut = null;
	
	public FilterRecordWriter(TaskAttemptContext job) {
	
		// 1 获取文件系统
		FileSystem fs;
	
		try {
			fs = FileSystem.get(job.getConfiguration());
	
			// 2 创建输出文件路径
			Path atguiguPath = new Path("e:/atguigu.log");
			Path otherPath = new Path("e:/other.log");
	
			// 3 创建输出流
			atguiguOut = fs.create(atguiguPath);
			otherOut = fs.create(otherPath);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
	@Override
	public void write(Text key, NullWritable value) throws IOException, InterruptedException {
	
		// 判断是否包含“atguigu”输出到不同文件
		if (key.toString().contains("atguigu")) {
			atguiguOut.write(key.toString().getBytes());
		} else {
			otherOut.write(key.toString().getBytes());
		}
	}
	
	@Override
	public void close(TaskAttemptContext context) throws IOException, InterruptedException {
	
		// 关闭资源
IOUtils.closeStream(atguiguOut);
		IOUtils.closeStream(otherOut);	}
}
（5）编写FilterDriver类
package com.atguigu.mapreduce.outputformat;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class FilterDriver {

	public static void main(String[] args) throws Exception {

// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
args = new String[] { "e:/input/inputoutputformat", "e:/output2" };

		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
	
		job.setJarByClass(FilterDriver.class);
		job.setMapperClass(FilterMapper.class);
		job.setReducerClass(FilterReducer.class);
	
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(NullWritable.class);
		
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);
	
		// 要将自定义的输出格式组件设置到job中
		job.setOutputFormatClass(FilterOutputFormat.class);
	
		FileInputFormat.setInputPaths(job, new Path(args[0]));
	
		// 虽然我们自定义了outputformat，但是因为我们的outputformat继承自fileoutputformat
		// 而fileoutputformat要输出一个_SUCCESS文件，所以，在这还得指定一个输出目录
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
3.7 Join多种应用
3.7.1 Reduce Join

3.7.2 Reduce Join案例实操
1．需求
表4-4 订单数据表t_order
id	pid	amount
1001	01	1
1002	02	2
1003	03	3
1004	01	4
1005	02	5
1006	03	6
表4-5 商品信息表t_product
pid	pname
01	小米
02	华为
03	格力
	将商品信息表中数据根据商品pid合并到订单数据表中。
表4-6 最终数据形式
id	pname	amount
1001	小米	1
1004	小米	4
1002	华为	2
1005	华为	5
1003	格力	3
1006	格力	6
2．需求分析
通过将关联条件作为Map输出的key，将两表满足Join条件的数据并携带数据所来源的文件信息，发往同一个ReduceTask，在Reduce中进行数据的串联，如图4-20所示。

图4-20 Reduce端表合并
3．代码实现
1）创建商品和订合并后的Bean类
package com.atguigu.mapreduce.table;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import org.apache.hadoop.io.Writable;

public class TableBean implements Writable {

	private String order_id; // 订单id
	private String p_id;      // 产品id
	private int amount;       // 产品数量
	private String pname;     // 产品名称
	private String flag;      // 表的标记
	
	public TableBean() {
		super();
	}
	
	public TableBean(String order_id, String p_id, int amount, String pname, String flag) {
	
		super();
	
		this.order_id = order_id;
		this.p_id = p_id;
		this.amount = amount;
		this.pname = pname;
		this.flag = flag;
	}
	
	public String getFlag() {
		return flag;
	}
	
	public void setFlag(String flag) {
		this.flag = flag;
	}
	
	public String getOrder_id() {
		return order_id;
	}
	
	public void setOrder_id(String order_id) {
		this.order_id = order_id;
	}
	
	public String getP_id() {
		return p_id;
	}
	
	public void setP_id(String p_id) {
		this.p_id = p_id;
	}
	
	public int getAmount() {
		return amount;
	}
	
	public void setAmount(int amount) {
		this.amount = amount;
	}
	
	public String getPname() {
		return pname;
	}
	
	public void setPname(String pname) {
		this.pname = pname;
	}
	
	@Override
	public void write(DataOutput out) throws IOException {
		out.writeUTF(order_id);
		out.writeUTF(p_id);
		out.writeInt(amount);
		out.writeUTF(pname);
		out.writeUTF(flag);
	}
	
	@Override
	public void readFields(DataInput in) throws IOException {
		this.order_id = in.readUTF();
		this.p_id = in.readUTF();
		this.amount = in.readInt();
		this.pname = in.readUTF();
		this.flag = in.readUTF();
	}
	
	@Override
	public String toString() {
		return order_id + "\t" + pname + "\t" + amount + "\t" ;
	}
}
2）编写TableMapper类
package com.atguigu.mapreduce.table;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class TableMapper extends Mapper<LongWritable, Text, Text, TableBean>{

String name;
	TableBean bean = new TableBean();
	Text k = new Text();
	
	@Override
	protected void setup(Context context) throws IOException, InterruptedException {
	
		// 1 获取输入文件切片
		FileSplit split = (FileSplit) context.getInputSplit();
	
		// 2 获取输入文件名称
		name = split.getPath().getName();
	}
	
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		
		// 1 获取输入数据
		String line = value.toString();
		
		// 2 不同文件分别处理
		if (name.startsWith("order")) {// 订单表处理
	
			// 2.1 切割
			String[] fields = line.split("\t");
			
			// 2.2 封装bean对象
			bean.setOrder_id(fields[0]);
			bean.setP_id(fields[1]);
			bean.setAmount(Integer.parseInt(fields[2]));
			bean.setPname("");
			bean.setFlag("order");
			
			k.set(fields[1]);
		}else {// 产品表处理
	
			// 2.3 切割
			String[] fields = line.split("\t");
			
			// 2.4 封装bean对象
			bean.setP_id(fields[0]);
			bean.setPname(fields[1]);
			bean.setFlag("pd");
			bean.setAmount(0);
			bean.setOrder_id("");
			
			k.set(fields[0]);
		}
	
		// 3 写出
		context.write(k, bean);
	}
}
3）编写TableReducer类
package com.atguigu.mapreduce.table;
import java.io.IOException;
import java.util.ArrayList;
import org.apache.commons.beanutils.BeanUtils;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class TableReducer extends Reducer<Text, TableBean, TableBean, NullWritable> {

	@Override
	protected void reduce(Text key, Iterable<TableBean> values, Context context)	throws IOException, InterruptedException {
	
		// 1准备存储订单的集合
		ArrayList<TableBean> orderBeans = new ArrayList<>();

// 2 准备bean对象
		TableBean pdBean = new TableBean();

		for (TableBean bean : values) {
	
			if ("order".equals(bean.getFlag())) {// 订单表
	
				// 拷贝传递过来的每条订单数据到集合中
				TableBean orderBean = new TableBean();
	
				try {
					BeanUtils.copyProperties(orderBean, bean);
				} catch (Exception e) {
					e.printStackTrace();
				}
	
				orderBeans.add(orderBean);
			} else {// 产品表
	
				try {
					// 拷贝传递过来的产品表到内存中
					BeanUtils.copyProperties(pdBean, bean);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}
	
		// 3 表的拼接
		for(TableBean bean:orderBeans){
	
			bean.setPname (pdBean.getPname());
			
			// 4 数据写出去
			context.write(bean, NullWritable.get());
		}
	}
}
4）编写TableDriver类
package com.atguigu.mapreduce.table;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class TableDriver {

	public static void main(String[] args) throws Exception {

// 0 根据自己电脑路径重新配置
args = new String[]{"e:/input/inputtable","e:/output1"};

// 1 获取配置信息，或者job对象实例
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 2 指定本程序的jar包所在的本地路径
		job.setJarByClass(TableDriver.class);
	
		// 3 指定本业务job要使用的Mapper/Reducer业务类
		job.setMapperClass(TableMapper.class);
		job.setReducerClass(TableReducer.class);
	
		// 4 指定Mapper输出数据的kv类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(TableBean.class);
	
		// 5 指定最终输出的数据的kv类型
		job.setOutputKeyClass(TableBean.class);
		job.setOutputValueClass(NullWritable.class);
	
		// 6 指定job的输入原始文件所在目录
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
4．测试
运行程序查看结果
1001	小米	1	
1001	小米	1	
1002	华为	2	
1002	华为	2	
1003	格力	3	
1003	格力	3	
5．总结

3.7.3 Map Join
1．使用场景
Map Join适用于一张表十分小、一张表很大的场景。
2．优点
思考：在Reduce端处理过多的表，非常容易产生数据倾斜。怎么办？
在Map端缓存多张表，提前处理业务逻辑，这样增加Map端业务，减少Reduce端数据的压力，尽可能的减少数据倾斜。
3．具体办法：采用DistributedCache
	（1）在Mapper的setup阶段，将文件读取到缓存集合中。
	（2）在驱动函数中加载缓存。
// 缓存普通文件到Task运行节点。
job.addCacheFile(new URI("file://e:/cache/pd.txt"));
3.7.4 Map Join案例实操
1．需求
表4-4 订单数据表t_order
id	pid	amount
1001	01	1
1002	02	2
1003	03	3
1004	01	4
1005	02	5
1006	03	6
表4-5 商品信息表t_product
pid	pname
01	小米
02	华为
03	格力
	将商品信息表中数据根据商品pid合并到订单数据表中。
表4-6 最终数据形式
id	pname	amount
1001	小米	1
1004	小米	4
1002	华为	2
1005	华为	5
1003	格力	3
1006	格力	6
2．需求分析
MapJoin适用于关联表中有小表的情形。

图4-21 Map端表合并
3．实现代码
（1）先在驱动模块中添加缓存文件
package test;
import java.net.URI;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class DistributedCacheDriver {

	public static void main(String[] args) throws Exception {

// 0 根据自己电脑路径重新配置
args = new String[]{"e:/input/inputtable2", "e:/output1"};

// 1 获取job信息
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);

		// 2 设置加载jar包路径
		job.setJarByClass(DistributedCacheDriver.class);
	
		// 3 关联map
		job.setMapperClass(DistributedCacheMapper.class);

// 4 设置最终输出数据类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);

		// 5 设置输入输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		// 6 加载缓存数据
		job.addCacheFile(new URI("file:///e:/input/inputcache/pd.txt"));
		
		// 7 Map端Join的逻辑不需要Reduce阶段，设置reduceTask数量为0
		job.setNumReduceTasks(0);
	
		// 8 提交
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
（2）读取缓存的文件数据
package test;
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;
import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class DistributedCacheMapper extends Mapper<LongWritable, Text, Text, NullWritable>{

	Map<String, String> pdMap = new HashMap<>();
	
	@Override
	protected void setup(Mapper<LongWritable, Text, Text, NullWritable>.Context context) throws IOException, InterruptedException {
	
		// 1 获取缓存的文件
		URI[] cacheFiles = context.getCacheFiles();
		String path = cacheFiles[0].getPath().toString();
		
		BufferedReader reader = new BufferedReader(new InputStreamReader(new FileInputStream(path), "UTF-8"));
		
		String line;
		while(StringUtils.isNotEmpty(line = reader.readLine())){
	
			// 2 切割
			String[] fields = line.split("\t");
			
			// 3 缓存数据到集合
			pdMap.put(fields[0], fields[1]);
		}
		
		// 4 关流
		reader.close();
	}
	
	Text k = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
	
		// 1 获取一行
		String line = value.toString();
		
		// 2 截取
		String[] fields = line.split("\t");
		
		// 3 获取产品id
		String pId = fields[1];
		
		// 4 获取商品名称
		String pdName = pdMap.get(pId);
		
		// 5 拼接
		k.set(line + "\t"+ pdName);
		
		// 6 写出
		context.write(k, NullWritable.get());
	}
}
3.8 计数器应用

3.9 数据清洗（ETL）
在运行核心业务MapReduce程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。清理的过程往往只需要运行Mapper程序，不需要运行Reduce程序。
3.9.1 数据清洗案例实操-简单解析版
1．需求
去除日志中字段长度小于等于11的日志。
（1）输入数据

（2）期望输出数据
每行字段长度都大于11。
2．需求分析
	需要在Map阶段对输入的数据根据规则进行过滤清洗。
3．实现代码
（1）编写LogMapper类
package com.atguigu.mapreduce.weblog;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class LogMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
	
	Text k = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		
		// 1 获取1行数据
		String line = value.toString();
		
		// 2 解析日志
		boolean result = parseLog(line,context);
		
		// 3 日志不合法退出
		if (!result) {
			return;
		}
		
		// 4 设置key
		k.set(line);
		
		// 5 写出数据
		context.write(k, NullWritable.get());
	}
	
	// 2 解析日志
	private boolean parseLog(String line, Context context) {
	
		// 1 截取
		String[] fields = line.split(" ");
		
		// 2 日志长度大于11的为合法
		if (fields.length > 11) {
	
			// 系统计数器
			context.getCounter("map", "true").increment(1);
			return true;
		}else {
			context.getCounter("map", "false").increment(1);
			return false;
		}
	}
}

（2）编写LogDriver类
package com.atguigu.mapreduce.weblog;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class LogDriver {

	public static void main(String[] args) throws Exception {

// 输入输出路径需要根据自己电脑上实际的输入输出路径设置
        args = new String[] { "e:/input/inputlog", "e:/output1" };

		// 1 获取job信息
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
	
		// 2 加载jar包
		job.setJarByClass(LogDriver.class);
	
		// 3 关联map
		job.setMapperClass(LogMapper.class);
	
		// 4 设置最终输出类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);
	
		// 设置reducetask个数为0
		job.setNumReduceTasks(0);
	
		// 5 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		// 6 提交
		job.waitForCompletion(true);
	}
}
3.9.2 数据清洗案例实操-复杂解析版
1．需求
对Web访问日志中的各字段识别切分，去除日志中不合法的记录。根据清洗规则，输出过滤后的数据。
（1）输入数据

（2）期望输出数据
都是合法的数据
2．实现代码
（1）定义一个bean，用来记录日志数据中的各数据字段
package com.atguigu.mapreduce.log;

public class LogBean {
	private String remote_addr;// 记录客户端的ip地址
	private String remote_user;// 记录客户端用户名称,忽略属性"-"
	private String time_local;// 记录访问时间与时区
	private String request;// 记录请求的url与http协议
	private String status;// 记录请求状态；成功是200
	private String body_bytes_sent;// 记录发送给客户端文件主体内容大小
	private String http_referer;// 用来记录从那个页面链接访问过来的
	private String http_user_agent;// 记录客户浏览器的相关信息

	private boolean valid = true;// 判断数据是否合法
	
	public String getRemote_addr() {
		return remote_addr;
	}
	
	public void setRemote_addr(String remote_addr) {
		this.remote_addr = remote_addr;
	}
	
	public String getRemote_user() {
		return remote_user;
	}
	
	public void setRemote_user(String remote_user) {
		this.remote_user = remote_user;
	}
	
	public String getTime_local() {
		return time_local;
	}
	
	public void setTime_local(String time_local) {
		this.time_local = time_local;
	}
	
	public String getRequest() {
		return request;
	}
	
	public void setRequest(String request) {
		this.request = request;
	}
	
	public String getStatus() {
		return status;
	}
	
	public void setStatus(String status) {
		this.status = status;
	}
	
	public String getBody_bytes_sent() {
		return body_bytes_sent;
	}
	
	public void setBody_bytes_sent(String body_bytes_sent) {
		this.body_bytes_sent = body_bytes_sent;
	}
	
	public String getHttp_referer() {
		return http_referer;
	}
	
	public void setHttp_referer(String http_referer) {
		this.http_referer = http_referer;
	}
	
	public String getHttp_user_agent() {
		return http_user_agent;
	}
	
	public void setHttp_user_agent(String http_user_agent) {
		this.http_user_agent = http_user_agent;
	}
	
	public boolean isValid() {
		return valid;
	}
	
	public void setValid(boolean valid) {
		this.valid = valid;
	}
	
	@Override
	public String toString() {
	
		StringBuilder sb = new StringBuilder();
		sb.append(this.valid);
		sb.append("\001").append(this.remote_addr);
		sb.append("\001").append(this.remote_user);
		sb.append("\001").append(this.time_local);
		sb.append("\001").append(this.request);
		sb.append("\001").append(this.status);
		sb.append("\001").append(this.body_bytes_sent);
		sb.append("\001").append(this.http_referer);
		sb.append("\001").append(this.http_user_agent);
		
		return sb.toString();
	}
}
（2）编写LogMapper类
package com.atguigu.mapreduce.log;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class LogMapper extends Mapper<LongWritable, Text, Text, NullWritable>{
	Text k = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
	
		// 1 获取1行
		String line = value.toString();
		
		// 2 解析日志是否合法
		LogBean bean = parseLog(line);
		
		if (!bean.isValid()) {
			return;
		}
		
		k.set(bean.toString());
		
		// 3 输出
		context.write(k, NullWritable.get());
	}
	
	// 解析日志
	private LogBean parseLog(String line) {
	
		LogBean logBean = new LogBean();
		
		// 1 截取
		String[] fields = line.split(" ");
		
		if (fields.length > 11) {
	
			// 2封装数据
			logBean.setRemote_addr(fields[0]);
			logBean.setRemote_user(fields[1]);
			logBean.setTime_local(fields[3].substring(1));
			logBean.setRequest(fields[6]);
			logBean.setStatus(fields[8]);
			logBean.setBody_bytes_sent(fields[9]);
			logBean.setHttp_referer(fields[10]);
			
			if (fields.length > 12) {
				logBean.setHttp_user_agent(fields[11] + " "+ fields[12]);
			}else {
				logBean.setHttp_user_agent(fields[11]);
			}
			
			// 大于400，HTTP错误
			if (Integer.parseInt(logBean.getStatus()) >= 400) {
				logBean.setValid(false);
			}
		}else {
			logBean.setValid(false);
		}
		
		return logBean;
	}
}
（3）编写LogDriver类
package com.atguigu.mapreduce.log;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class LogDriver {
	public static void main(String[] args) throws Exception {
		
// 1 获取job信息
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);

		// 2 加载jar包
		job.setJarByClass(LogDriver.class);
	
		// 3 关联map
		job.setMapperClass(LogMapper.class);
	
		// 4 设置最终输出类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(NullWritable.class);
	
		// 5 设置输入和输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		// 6 提交
		job.waitForCompletion(true);
	}
}
3.10 MapReduce开发总结
在编写MapReduce程序时，需要考虑如下几个方面：





第4章 Hadoop数据压缩
4.1 概述


4.2 MR支持的压缩编码
表4-7
压缩格式	hadoop自带？	算法	文件扩展名	是否可切分	换成压缩格式后，原来的程序是否需要修改
DEFLATE	是，直接使用	DEFLATE	.deflate	否	和文本处理一样，不需要修改
Gzip	是，直接使用	DEFLATE	.gz	否	和文本处理一样，不需要修改
bzip2	是，直接使用	bzip2	.bz2	是	和文本处理一样，不需要修改
LZO	否，需要安装	LZO	.lzo	是	需要建索引，还需要指定输入格式
Snappy	否，需要安装	Snappy	.snappy	否	和文本处理一样，不需要修改
为了支持多种压缩/解压缩算法，Hadoop引入了编码/解码器，如下表所示。
表4-8
压缩格式	对应的编码/解码器
DEFLATE	org.apache.hadoop.io.compress.DefaultCodec
gzip	org.apache.hadoop.io.compress.GzipCodec
bzip2	org.apache.hadoop.io.compress.BZip2Codec
LZO	com.hadoop.compression.lzo.LzopCodec
Snappy	org.apache.hadoop.io.compress.SnappyCodec
压缩性能的比较
表4-9
压缩算法	原始文件大小	压缩文件大小	压缩速度	解压速度
gzip	8.3GB	1.8GB	17.5MB/s	58MB/s
bzip2	8.3GB	1.1GB	2.4MB/s	9.5MB/s
LZO	8.3GB	2.9GB	49.3MB/s	74.6MB/s
http://google.github.io/snappy/
On a single core of a Core i7 processor in 64-bit mode, Snappy compresses at about 250 MB/sec or more and decompresses at about 500 MB/sec or more.
4.3 压缩方式选择
4.3.1 Gzip压缩

4.3.2 Bzip2压缩

4.3.3 Lzo压缩

4.3.4 Snappy压缩

4.4 压缩位置选择
压缩可以在MapReduce作用的任意阶段启用，如图4-22所示。

图4-22 MapReduce数据压缩
4.5 压缩参数配置
要在Hadoop中启用压缩，可以配置如下参数：
表4-10 配置参数
参数	默认值	阶段	建议
io.compression.codecs   
（在core-site.xml中配置）	org.apache.hadoop.io.compress.DefaultCodec, org.apache.hadoop.io.compress.GzipCodec, org.apache.hadoop.io.compress.BZip2Codec
	输入压缩	Hadoop使用文件扩展名判断是否支持某种编解码器
mapreduce.map.output.compress（在mapred-site.xml中配置）	false	mapper输出	这个参数设为true启用压缩
mapreduce.map.output.compress.codec（在mapred-site.xml中配置）	org.apache.hadoop.io.compress.DefaultCodec	mapper输出	企业多使用LZO或Snappy编解码器在此阶段压缩数据
mapreduce.output.fileoutputformat.compress（在mapred-site.xml中配置）	false	reducer输出	这个参数设为true启用压缩
mapreduce.output.fileoutputformat.compress.codec（在mapred-site.xml中配置）	org.apache.hadoop.io.compress. DefaultCodec	reducer输出	使用标准工具或者编解码器，如gzip和bzip2
mapreduce.output.fileoutputformat.compress.type（在mapred-site.xml中配置）	RECORD	reducer输出	SequenceFile输出使用的压缩类型：NONE和BLOCK
4.6 压缩实操案例
4.6.1 数据流的压缩和解压缩
测试一下如下压缩方式：
表4-11
DEFLATE	org.apache.hadoop.io.compress.DefaultCodec
gzip	org.apache.hadoop.io.compress.GzipCodec
bzip2	org.apache.hadoop.io.compress.BZip2Codec
package com.atguigu.mapreduce.compress;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.io.compress.CompressionCodec;
import org.apache.hadoop.io.compress.CompressionCodecFactory;
import org.apache.hadoop.io.compress.CompressionInputStream;
import org.apache.hadoop.io.compress.CompressionOutputStream;
import org.apache.hadoop.util.ReflectionUtils;

public class TestCompress {

	public static void main(String[] args) throws Exception {
		compress("e:/hello.txt","org.apache.hadoop.io.compress.BZip2Codec");
//		decompress("e:/hello.txt.bz2");
	}

	// 1、压缩
	private static void compress(String filename, String method) throws Exception {
		
		// （1）获取输入流
		FileInputStream fis = new FileInputStream(new File(filename));
		
		Class codecClass = Class.forName(method);
		
		CompressionCodec codec = (CompressionCodec) ReflectionUtils.newInstance(codecClass, new Configuration());
		
		// （2）获取输出流
		FileOutputStream fos = new FileOutputStream(new File(filename + codec.getDefaultExtension()));
		CompressionOutputStream cos = codec.createOutputStream(fos);
		
		// （3）流的对拷
		IOUtils.copyBytes(fis, cos, 1024*1024*5, false);
		
		// （4）关闭资源
		cos.close();
		fos.close();
fis.close();
	}

	// 2、解压缩
	private static void decompress(String filename) throws FileNotFoundException, IOException {
		
		// （0）校验是否能解压缩
		CompressionCodecFactory factory = new CompressionCodecFactory(new Configuration());
	
		CompressionCodec codec = factory.getCodec(new Path(filename));
		
		if (codec == null) {
			System.out.println("cannot find codec for file " + filename);
			return;
		}
		
		// （1）获取输入流
		CompressionInputStream cis = codec.createInputStream(new FileInputStream(new File(filename)));
		
		// （2）获取输出流
		FileOutputStream fos = new FileOutputStream(new File(filename + ".decoded"));
		
		// （3）流的对拷
		IOUtils.copyBytes(cis, fos, 1024*1024*5, false);
		
		// （4）关闭资源
		cis.close();
		fos.close();
	}
}
4.6.2 Map输出端采用压缩
即使你的MapReduce的输入输出文件都是未压缩的文件，你仍然可以对Map任务的中间结果输出做压缩，因为它要写在硬盘并且通过网络传输到Reduce节点，对其压缩可以提高很多性能，这些工作只要设置两个属性即可，我们来看下代码怎么设置。
1．给大家提供的Hadoop源码支持的压缩格式有：BZip2Codec 、DefaultCodec
package com.atguigu.mapreduce.compress;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.compress.BZip2Codec;	
import org.apache.hadoop.io.compress.CompressionCodec;
import org.apache.hadoop.io.compress.GzipCodec;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
	
		Configuration configuration = new Configuration();
	
		// 开启map端输出压缩
	configuration.setBoolean("mapreduce.map.output.compress", true);
		// 设置map端输出压缩方式
	configuration.setClass("mapreduce.map.output.compress.codec", BZip2Codec.class, CompressionCodec.class);
	
		Job job = Job.getInstance(configuration);
	
		job.setJarByClass(WordCountDriver.class);
	
		job.setMapperClass(WordCountMapper.class);
		job.setReducerClass(WordCountReducer.class);
	
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
	
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
	
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		boolean result = job.waitForCompletion(true);
	
		System.exit(result ? 1 : 0);
	}
}
2．Mapper保持不变
package com.atguigu.mapreduce.compress;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{

Text k = new Text();
	IntWritable v = new IntWritable(1);

	@Override
	protected void map(LongWritable key, Text value, Context context)throws IOException, InterruptedException {
	
		// 1 获取一行
		String line = value.toString();
	
		// 2 切割
		String[] words = line.split(" ");
	
		// 3 循环写出
		for(String word:words){
k.set(word);
			context.write(k, v);
		}
	}
}
3．Reducer保持不变
package com.atguigu.mapreduce.compress;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable>{

	IntWritable v = new IntWritable();
	
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,
			Context context) throws IOException, InterruptedException {
		
		int sum = 0;
	
		// 1 汇总
		for(IntWritable value:values){
			sum += value.get();
		}
		
	    v.set(sum);
	
	    // 2 输出
		context.write(key, v);
	}
}
4.6.3 Reduce输出端采用压缩
基于WordCount案例处理。
1．修改驱动
package com.atguigu.mapreduce.compress;
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.compress.BZip2Codec;
import org.apache.hadoop.io.compress.DefaultCodec;
import org.apache.hadoop.io.compress.GzipCodec;
import org.apache.hadoop.io.compress.Lz4Codec;
import org.apache.hadoop.io.compress.SnappyCodec;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountDriver {

	public static void main(String[] args) throws IOException, ClassNotFoundException, InterruptedException {
		
		Configuration configuration = new Configuration();
		
		Job job = Job.getInstance(configuration);
		
		job.setJarByClass(WordCountDriver.class);
		
		job.setMapperClass(WordCountMapper.class);
		job.setReducerClass(WordCountReducer.class);
		
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		// 设置reduce端输出压缩开启
		FileOutputFormat.setCompressOutput(job, true);
		
		// 设置压缩的方式
	    FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class); 
//	    FileOutputFormat.setOutputCompressorClass(job, GzipCodec.class); 
//	    FileOutputFormat.setOutputCompressorClass(job, DefaultCodec.class); 
	    
		boolean result = job.waitForCompletion(true);
		
		System.exit(result?1:0);
	}
}
2．Mapper和Reducer保持不变（详见4.6.2）
第5章 Yarn资源调度器
Yarn是一个资源调度平台，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而MapReduce等运算程序则相当于运行于操作系统之上的应用程序。
5.1 Yarn基本架构
 	YARN主要由ResourceManager、NodeManager、ApplicationMaster和Container等组件构成，如图4-23所示。

图4-23 Yarn基本架构
5.3 Yarn工作机制
1．Yarn运行机制，如图4-24所示。

图4-24  Yarn工作机制
2．工作机制详解
	（1）MR程序提交到客户端所在的节点。
	（2）YarnRunner向ResourceManager申请一个Application。
	（3）RM将该应用程序的资源路径返回给YarnRunner。
	（4）该程序将运行所需资源提交到HDFS上。
	（5）程序资源提交完毕后，申请运行mrAppMaster。
	（6）RM将用户的请求初始化成一个Task。
	（7）其中一个NodeManager领取到Task任务。
	（8）该NodeManager创建容器Container，并产生MRAppmaster。
	（9）Container从HDFS上拷贝资源到本地。
	（10）MRAppmaster向RM 申请运行MapTask资源。
	（11）RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。
	（12）MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。
（13）MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。
	（14）ReduceTask向MapTask获取相应分区的数据。
	（15）程序运行完毕后，MR会向RM申请注销自己。
5.4 作业提交全过程
1．作业提交过程之YARN，如图4-25所示。

图4-25 作业提交过程之Yarn
作业提交全过程详解
（1）作业提交
第1步：Client调用job.waitForCompletion方法，向整个集群提交MapReduce作业。
第2步：Client向RM申请一个作业id。
第3步：RM给Client返回该job资源的提交路径和作业id。
第4步：Client提交jar包、切片信息和配置文件到指定的资源提交路径。
第5步：Client提交完资源后，向RM申请运行MrAppMaster。
（2）作业初始化
第6步：当RM收到Client的请求后，将该job添加到容量调度器中。
第7步：某一个空闲的NM领取到该Job。
第8步：该NM创建Container，并产生MRAppmaster。
第9步：下载Client提交的资源到本地。
（3）任务分配
第10步：MrAppMaster向RM申请运行多个MapTask任务资源。
第11步：RM将运行MapTask任务分配给另外两个NodeManager，另两个NodeManager分别领取任务并创建容器。
（4）任务运行
第12步：MR向两个接收到任务的NodeManager发送程序启动脚本，这两个NodeManager分别启动MapTask，MapTask对数据分区排序。
第13步：MrAppMaster等待所有MapTask运行完毕后，向RM申请容器，运行ReduceTask。
第14步：ReduceTask向MapTask获取相应分区的数据。
第15步：程序运行完毕后，MR会向RM申请注销自己。
（5）进度和状态更新
YARN中的任务将其进度和状态(包括counter)返回给应用管理器, 客户端每秒(通过mapreduce.client.progressmonitor.pollinterval设置)向应用管理器请求进度更新, 展示给用户。
（6）作业完成
除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用waitForCompletion()来检查作业是否完成。时间间隔可以通过mapreduce.client.completion.pollinterval来设置。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。
2．作业提交过程之MapReduce，如图4-26所示

图4-26 作业提交过程之MapReduce
5.5 资源调度器
目前，Hadoop作业调度器主要有三种：FIFO、Capacity Scheduler和Fair Scheduler。Hadoop2.7.2默认的资源调度器是Capacity Scheduler。
具体设置详见：yarn-default.xml文件
<property>
    <description>The class to use as the resource scheduler.</description>
    <name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
</property>
1．先进先出调度器（FIFO），如图4-27所示
	图4-27 FIFO调度器
2．容量调度器（Capacity Scheduler），如图4-28所示
	图4-28容量调度器
3．公平调度器（Fair Scheduler），如图4-29所示

图4-29公平调度器
5.6 任务的推测执行
1．作业完成时间取决于最慢的任务完成时间
一个作业由若干个Map任务和Reduce任务构成。因硬件老化、软件Bug等，某些任务可能运行非常慢。
思考：系统中有99%的Map任务都完成了，只有少数几个Map老是进度很慢，完不成，怎么办？
2．推测执行机制
发现拖后腿的任务，比如某个任务运行速度远慢于任务平均速度。为拖后腿任务启动一个备份任务，同时运行。谁先运行完，则采用谁的结果。
3．执行推测任务的前提条件
（1）每个Task只能有一个备份任务
（2）当前Job已完成的Task必须不小于0.05（5%）
（3）开启推测执行参数设置。mapred-site.xml文件中默认是打开的。
<property>
  	<name>mapreduce.map.speculative</name>
  	<value>true</value>
  	<description>If true, then multiple instances of some map tasks may be executed in parallel.</description>
</property>

<property>
  	<name>mapreduce.reduce.speculative</name>
  	<value>true</value>
  	<description>If true, then multiple instances of some reduce tasks may be executed in parallel.</description>
</property>
4．不能启用推测执行机制情况
   （1）任务间存在严重的负载倾斜；
   （2）特殊任务，比如任务向数据库中写数据。
5．算法原理，如图4-20所示

图4-30 推测执行算法原理
第6章 Hadoop企业优化
6.1 MapReduce 跑的慢的原因

6.2 MapReduce优化方法
MapReduce优化方法主要从六个方面考虑：数据输入、Map阶段、Reduce阶段、IO传输、数据倾斜问题和常用的调优参数。
6.2.1 数据输入

6.2.2 Map阶段

6.2.3 Reduce阶段


6.2.4 I/O传输

6.2.5 数据倾斜问题

6.2.6 常用的调优参数
1．资源相关参数
（1）以下参数是在用户自己的MR应用程序中配置就可以生效（mapred-default.xml）
表4-12
配置参数	参数说明
mapreduce.map.memory.mb	一个MapTask可使用的资源上限（单位:MB），默认为1024。如果MapTask实际使用的资源量超过该值，则会被强制杀死。
mapreduce.reduce.memory.mb	一个ReduceTask可使用的资源上限（单位:MB），默认为1024。如果ReduceTask实际使用的资源量超过该值，则会被强制杀死。
mapreduce.map.cpu.vcores	每个MapTask可使用的最多cpu core数目，默认值: 1
mapreduce.reduce.cpu.vcores	每个ReduceTask可使用的最多cpu core数目，默认值: 1
mapreduce.reduce.shuffle.parallelcopies	每个Reduce去Map中取数据的并行数。默认值是5
mapreduce.reduce.shuffle.merge.percent	Buffer中的数据达到多少比例开始写入磁盘。默认值0.66
mapreduce.reduce.shuffle.input.buffer.percent	Buffer大小占Reduce可用内存的比例。默认值0.7
mapreduce.reduce.input.buffer.percent	指定多少比例的内存用来存放Buffer中的数据，默认值是0.0
（2）应该在YARN启动之前就配置在服务器的配置文件中才能生效（yarn-default.xml）
表4-13
配置参数	参数说明
yarn.scheduler.minimum-allocation-mb	  	给应用程序Container分配的最小内存，默认值：1024
yarn.scheduler.maximum-allocation-mb	  	给应用程序Container分配的最大内存，默认值：8192
yarn.scheduler.minimum-allocation-vcores		每个Container申请的最小CPU核数，默认值：1
yarn.scheduler.maximum-allocation-vcores		每个Container申请的最大CPU核数，默认值：32
yarn.nodemanager.resource.memory-mb   	给Containers分配的最大物理内存，默认值：8192
（3）Shuffle性能优化的关键参数，应在YARN启动之前就配置好（mapred-default.xml）
表4-14
配置参数	参数说明
mapreduce.task.io.sort.mb   	Shuffle的环形缓冲区大小，默认100m
mapreduce.map.sort.spill.percent   	环形缓冲区溢出的阈值，默认80%
2．容错相关参数(MapReduce性能优化)
表4-15
配置参数	参数说明
mapreduce.map.maxattempts	每个Map Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。
mapreduce.reduce.maxattempts	每个Reduce Task最大重试次数，一旦重试参数超过该值，则认为Map Task运行失败，默认值：4。
mapreduce.task.timeout	Task超时时间，经常需要设置的一个参数，该参数表达的意思为：如果一个Task在一定时间内没有任何进入，即不会读取新的数据，也没有输出数据，则认为该Task处于Block状态，可能是卡住了，也许永远会卡住，为了防止因为用户程序永远Block住不退出，则强制设置了一个该超时时间（单位毫秒），默认是600000。如果你的程序对每条输入数据的处理时间过长（比如会访问数据库，通过网络拉取数据等），建议将该参数调大，该参数过小常出现的错误提示是“AttemptID:attempt_14267829456721_123456_m_000224_0 Timed out after 300 secsContainer killed by the ApplicationMaster.”。
6.3 HDFS小文件优化方法
6.3.1 HDFS小文件弊端
HDFS上每个文件都要在NameNode上建立一个索引，这个索引的大小约为150byte，这样当小文件比较多的时候，就会产生很多的索引文件，一方面会大量占用NameNode的内存空间，另一方面就是索引文件过大使得索引速度变慢。
6.3.2 HDFS小文件解决方案
小文件的优化无非以下几种方式：
（1）在数据采集的时候，就将小文件或小批数据合成大文件再上传HDFS。
（2）在业务处理之前，在HDFS上使用MapReduce程序对小文件进行合并。
（3）在MapReduce处理时，可采用CombineTextInputFormat提高效率。


第7章 MapReduce扩展案例
7.1 倒排索引案例（多job串联）
1．需求
有大量的文本（文档、网页），需要建立搜索索引，如图4-31所示。
（1）数据输入
    
（2）期望输出数据
atguigu	c.txt-->2	b.txt-->2	a.txt-->3	
pingping	c.txt-->1	b.txt-->3	a.txt-->1	
ss	c.txt-->1	b.txt-->1	a.txt-->2	
2．需求分析

3．第一次处理
（1）第一次处理，编写OneIndexMapper类
package com.atguigu.mapreduce.index;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;

public class OneIndexMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	
	String name;
	Text k = new Text();
	IntWritable v = new IntWritable();
	
	@Override
	protected void setup(Context context)throws IOException, InterruptedException {
	
		// 获取文件名称
		FileSplit split = (FileSplit) context.getInputSplit();
		
		name = split.getPath().getName();
	}
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
	
		// 1 获取1行
		String line = value.toString();
		
		// 2 切割
		String[] fields = line.split(" ");
		
		for (String word : fields) {
	
			// 3 拼接
			k.set(word+"--"+name);
			v.set(1);
			
			// 4 写出
			context.write(k, v);
		}
	}
}
（2）第一次处理，编写OneIndexReducer类
package com.atguigu.mapreduce.index;
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class OneIndexReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
	
IntWritable v = new IntWritable();

	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,Context context) throws IOException, InterruptedException {
		
		int sum = 0;
	
		// 1 累加求和
		for(IntWritable value: values){
			sum +=value.get();
		}
		
	   v.set(sum);
	
		// 2 写出
		context.write(key, v);
	}
}
（3）第一次处理，编写OneIndexDriver类
package com.atguigu.mapreduce.index;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class OneIndexDriver {

	public static void main(String[] args) throws Exception {
	
	   // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
		args = new String[] { "e:/input/inputoneindex", "e:/output5" };
	
		Configuration conf = new Configuration();
	
		Job job = Job.getInstance(conf);
		job.setJarByClass(OneIndexDriver.class);
	
		job.setMapperClass(OneIndexMapper.class);
		job.setReducerClass(OneIndexReducer.class);
	
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
	
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		job.waitForCompletion(true);
	}
}
（4）查看第一次输出结果
atguigu--a.txt	3
atguigu--b.txt	2
atguigu--c.txt	2
pingping--a.txt	1
pingping--b.txt	3
pingping--c.txt	1
ss--a.txt	2
ss--b.txt	1
ss--c.txt	1
4．第二次处理
（1）第二次处理，编写TwoIndexMapper类
package com.atguigu.mapreduce.index;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class TwoIndexMapper extends Mapper<LongWritable, Text, Text, Text>{

	Text k = new Text();
	Text v = new Text();
	
	@Override
	protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
		
		// 1 获取1行数据
		String line = value.toString();
		
		// 2用“--”切割
		String[] fields = line.split("--");
		
		k.set(fields[0]);
		v.set(fields[1]);
		
		// 3 输出数据
		context.write(k, v);
	}
}
（2）第二次处理，编写TwoIndexReducer类
package com.atguigu.mapreduce.index;
import java.io.IOException;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
public class TwoIndexReducer extends Reducer<Text, Text, Text, Text> {

Text v = new Text();

	@Override
	protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
		// atguigu a.txt 3
		// atguigu b.txt 2
		// atguigu c.txt 2
	
		// atguigu c.txt-->2 b.txt-->2 a.txt-->3
	
		StringBuilder sb = new StringBuilder();
	
	    // 1 拼接
		for (Text value : values) {
			sb.append(value.toString().replace("\t", "-->") + "\t");
		}

v.set(sb.toString());

		// 2 写出
		context.write(key, v);
	}
}
（3）第二次处理，编写TwoIndexDriver类
package com.atguigu.mapreduce.index;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class TwoIndexDriver {

	public static void main(String[] args) throws Exception {
	
	   // 输入输出路径需要根据自己电脑上实际的输入输出路径设置
args = new String[] { "e:/input/inputtwoindex", "e:/output6" };

		Configuration config = new Configuration();
		Job job = Job.getInstance(config);

job.setJarByClass(TwoIndexDriver.class);
		job.setMapperClass(TwoIndexMapper.class);
		job.setReducerClass(TwoIndexReducer.class);

		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
	
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		boolean result = job.waitForCompletion(true);
System.exit(result?0:1);
	}
}
（4）第二次查看最终结果
atguigu	c.txt-->2	b.txt-->2	a.txt-->3	
pingping	c.txt-->1	b.txt-->3	a.txt-->1	
ss	c.txt-->1	b.txt-->1	a.txt-->2	
7.2 TopN案例
1．需求
对需求2.3输出结果进行加工，输出流量使用量在前10的用户信息
（1）输入数据			（2）输出数据
			
2．需求分析

3．实现代码
（1）编写FlowBean类
package com.atguigu.mr.top;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.WritableComparable;

public class FlowBean implements WritableComparable<FlowBean>{

	private long upFlow;
	private long downFlow;
	private long sumFlow;


​	
​	public FlowBean() {
​		super();
​	}
​	
​	public FlowBean(long upFlow, long downFlow) {
​		super();
​		this.upFlow = upFlow;
​		this.downFlow = downFlow;
​	}
​	
​	@Override
​	public void write(DataOutput out) throws IOException {
​		out.writeLong(upFlow);
​		out.writeLong(downFlow);
​		out.writeLong(sumFlow);
​	}
​	
​	@Override
​	public void readFields(DataInput in) throws IOException {
​		upFlow = in.readLong();
​		downFlow = in.readLong();
​		sumFlow = in.readLong();
​	}
​	
​	public long getUpFlow() {
​		return upFlow;
​	}
​	
​	public void setUpFlow(long upFlow) {
​		this.upFlow = upFlow;
​	}
​	
​	public long getDownFlow() {
​		return downFlow;
​	}
​	
​	public void setDownFlow(long downFlow) {
​		this.downFlow = downFlow;
​	}
​	
​	public long getSumFlow() {
​		return sumFlow;
​	}
​	
	public void setSumFlow(long sumFlow) {
		this.sumFlow = sumFlow;
	}
	
	@Override
	public String toString() {
		return upFlow + "\t" + downFlow + "\t" + sumFlow;
	}
	
	public void set(long downFlow2, long upFlow2) {
		downFlow = downFlow2;
		upFlow = upFlow2;
		sumFlow = downFlow2 + upFlow2;
	}
	
	@Override
	public int compareTo(FlowBean bean) {
		
		int result;
		
		if (this.sumFlow > bean.getSumFlow()) {
			result = -1;
		}else if (this.sumFlow < bean.getSumFlow()) {
			result = 1;
		}else {
			result = 0;
		}
		
		return result;
	}
}
（2）编写TopNMapper类
package com.atguigu.mr.top;

import java.io.IOException;
import java.util.Iterator;
import java.util.TreeMap;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class TopNMapper extends Mapper<LongWritable, Text, FlowBean, Text>{
	
	// 定义一个TreeMap作为存储数据的容器（天然按key排序）
	private TreeMap<FlowBean, Text> flowMap = new TreeMap<FlowBean, Text>();
	private FlowBean kBean;
	
	@Override
	protected void map(LongWritable key, Text value, Context context)	throws IOException, InterruptedException {
		
		kBean = new FlowBean();
		Text v = new Text();
		
		// 1 获取一行
		String line = value.toString();
		
		// 2 切割
		String[] fields = line.split("\t");
		
		// 3 封装数据
		String phoneNum = fields[0];
		long upFlow = Long.parseLong(fields[1]);
		long downFlow = Long.parseLong(fields[2]);
		long sumFlow = Long.parseLong(fields[3]);
		
		kBean.setDownFlow(downFlow);
		kBean.setUpFlow(upFlow);
		kBean.setSumFlow(sumFlow);
		
		v.set(phoneNum);
		
		// 4 向TreeMap中添加数据
		flowMap.put(kBean, v);
		
		// 5 限制TreeMap的数据量，超过10条就删除掉流量最小的一条数据
		if (flowMap.size() > 10) {
//		flowMap.remove(flowMap.firstKey());
			flowMap.remove(flowMap.lastKey());		
}
	}
	
	@Override
	protected void cleanup(Context context) throws IOException, InterruptedException {
		
		// 6 遍历treeMap集合，输出数据
		Iterator<FlowBean> bean = flowMap.keySet().iterator();
	
		while (bean.hasNext()) {
	
			FlowBean k = bean.next();
	
			context.write(k, flowMap.get(k));
		}
	}
}
（3）编写TopNReducer类
package com.atguigu.mr.top;

import java.io.IOException;
import java.util.Iterator;
import java.util.TreeMap;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class TopNReducer extends Reducer<FlowBean, Text, Text, FlowBean> {

	// 定义一个TreeMap作为存储数据的容器（天然按key排序）
	TreeMap<FlowBean, Text> flowMap = new TreeMap<FlowBean, Text>();
	
	@Override
	protected void reduce(FlowBean key, Iterable<Text> values, Context context)throws IOException, InterruptedException {
	
		for (Text value : values) {
	
			 FlowBean bean = new FlowBean();
			 bean.set(key.getDownFlow(), key.getUpFlow());
	
			 // 1 向treeMap集合中添加数据
			flowMap.put(bean, new Text(value));
	
			// 2 限制TreeMap数据量，超过10条就删除掉流量最小的一条数据
			if (flowMap.size() > 10) {
				// flowMap.remove(flowMap.firstKey());
flowMap.remove(flowMap.lastKey());
			}
		}
	}

	@Override
	protected void cleanup(Reducer<FlowBean, Text, Text, FlowBean>.Context context) throws IOException, InterruptedException {
	
		// 3 遍历集合，输出数据
		Iterator<FlowBean> it = flowMap.keySet().iterator();
	
		while (it.hasNext()) {
	
			FlowBean v = it.next();
	
			context.write(new Text(flowMap.get(v)), v);
		}
	}
}
（4）编写TopNDriver类
package com.atguigu.mr.top;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class TopNDriver {

	public static void main(String[] args) throws Exception {
		
		args  = new String[]{"e:/output1","e:/output3"};
		
		// 1 获取配置信息，或者job对象实例
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);
	
		// 6 指定本程序的jar包所在的本地路径
		job.setJarByClass(TopNDriver.class);
	
		// 2 指定本业务job要使用的mapper/Reducer业务类
		job.setMapperClass(TopNMapper.class);
		job.setReducerClass(TopNReducer.class);
	
		// 3 指定mapper输出数据的kv类型
		job.setMapOutputKeyClass(FlowBean.class);
		job.setMapOutputValueClass(Text.class);
	
		// 4 指定最终输出的数据的kv类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(FlowBean.class);
	
		// 5 指定job的输入原始文件所在目录
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
	
		// 7 将job中配置的相关参数，以及job所用的java类所在的jar包， 提交给yarn去运行
		boolean result = job.waitForCompletion(true);
		System.exit(result ? 0 : 1);
	}
}
7.3 找博客共同好友案例
1．需求
以下是博客的好友列表数据，冒号前是一个用户，冒号后是该用户的所有好友（数据中的好友关系是单向的）
求出哪些人两两之间有共同好友，及他俩的共同好友都有谁？
（1）数据输入

2．需求分析
先求出A、B、C、….等是谁的好友
第一次输出结果
A	I,K,C,B,G,F,H,O,D,
B	A,F,J,E,
C	A,E,B,H,F,G,K,
D	G,C,K,A,L,F,E,H,
E	G,M,L,H,A,F,B,D,
F	L,M,D,C,G,A,
G	M,
H	O,
I	O,C,
J	O,
K	B,
L	D,E,
M	E,F,
O	A,H,I,J,F,
第二次输出结果
A-B	E C 
A-C	D F 
A-D	E F 
A-E	D B C 
A-F	O B C D E 
A-G	F E C D 
A-H	E C D O 
A-I	O 
A-J	O B 
A-K	D C 
A-L	F E D 
A-M	E F 
B-C	A 
B-D	A E 
B-E	C 
B-F	E A C 
B-G	C E A 
B-H	A E C 
B-I	A 
B-K	C A 
B-L	E 
B-M	E 
B-O	A 
C-D	A F 
C-E	D 
C-F	D A 
C-G	D F A 
C-H	D A 
C-I	A 
C-K	A D 
C-L	D F 
C-M	F 
C-O	I A 
D-E	L 
D-F	A E 
D-G	E A F 
D-H	A E 
D-I	A 
D-K	A 
D-L	E F 
D-M	F E 
D-O	A 
E-F	D M C B 
E-G	C D 
E-H	C D 
E-J	B 
E-K	C D 
E-L	D 
F-G	D C A E 
F-H	A D O E C 
F-I	O A 
F-J	B O 
F-K	D C A 
F-L	E D 
F-M	E 
F-O	A 
G-H	D C E A 
G-I	A 
G-K	D A C 
G-L	D F E 
G-M	E F 
G-O	A 
H-I	O A 
H-J	O 
H-K	A C D 
H-L	D E 
H-M	E 
H-O	A 
I-J	O 
I-K	A 
I-O	A 
K-L	D 
K-O	A 
L-M	E F
3．代码实现
（1）第一次Mapper类
package com.atguigu.mapreduce.friends;
import java.io.IOException;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class OneShareFriendsMapper extends Mapper<LongWritable, Text, Text, Text>{
	
	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, Text>.Context context)
			throws IOException, InterruptedException {
	
		// 1 获取一行 A:B,C,D,F,E,O
		String line = value.toString();
		
		// 2 切割
		String[] fields = line.split(":");
		
		// 3 获取person和好友
		String person = fields[0];
		String[] friends = fields[1].split(",");
		
		// 4写出去
		for(String friend: friends){
	
			// 输出 <好友，人>
			context.write(new Text(friend), new Text(person));
		}
	}
}
（2）第一次Reducer类
package com.atguigu.mapreduce.friends;
import java.io.IOException;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class OneShareFriendsReducer extends Reducer<Text, Text, Text, Text>{
	
	@Override
	protected void reduce(Text key, Iterable<Text> values, Context context)throws IOException, InterruptedException {
		
		StringBuffer sb = new StringBuffer();
	
		//1 拼接
		for(Text person: values){
			sb.append(person).append(",");
		}
		
		//2 写出
		context.write(key, new Text(sb.toString()));
	}
}
（3）第一次Driver类
package com.atguigu.mapreduce.friends;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class OneShareFriendsDriver {

	public static void main(String[] args) throws Exception {

// 1 获取job对象
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);
		
		// 2 指定jar包运行的路径
		job.setJarByClass(OneShareFriendsDriver.class);
	
		// 3 指定map/reduce使用的类
		job.setMapperClass(OneShareFriendsMapper.class);
		job.setReducerClass(OneShareFriendsReducer.class);
		
		// 4 指定map输出的数据类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		
		// 5 指定最终输出的数据类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		
		// 6 指定job的输入原始所在目录
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		// 7 提交
		boolean result = job.waitForCompletion(true);
		
		System.exit(result?0:1);
	}
}
（4）第二次Mapper类
package com.atguigu.mapreduce.friends;
import java.io.IOException;
import java.util.Arrays;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class TwoShareFriendsMapper extends Mapper<LongWritable, Text, Text, Text>{
	
	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
	
		// A I,K,C,B,G,F,H,O,D,
		// 友 人，人，人
		String line = value.toString();
		String[] friend_persons = line.split("\t");
	
		String friend = friend_persons[0];
		String[] persons = friend_persons[1].split(",");
	
		Arrays.sort(persons);
	
		for (int i = 0; i < persons.length - 1; i++) {
			
			for (int j = i + 1; j < persons.length; j++) {
				// 发出 <人-人，好友> ，这样，相同的“人-人”对的所有好友就会到同1个reduce中去
				context.write(new Text(persons[i] + "-" + persons[j]), new Text(friend));
			}
		}
	}
}
（5）第二次Reducer类
package com.atguigu.mapreduce.friends;
import java.io.IOException;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class TwoShareFriendsReducer extends Reducer<Text, Text, Text, Text>{
	
	@Override
	protected void reduce(Text key, Iterable<Text> values, Context context)	throws IOException, InterruptedException {
		
		StringBuffer sb = new StringBuffer();
	
		for (Text friend : values) {
			sb.append(friend).append(" ");
		}
		
		context.write(key, new Text(sb.toString()));
	}
}
（6）第二次Driver类
package com.atguigu.mapreduce.friends;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class TwoShareFriendsDriver {

	public static void main(String[] args) throws Exception {

// 1 获取job对象
		Configuration configuration = new Configuration();
		Job job = Job.getInstance(configuration);
		
		// 2 指定jar包运行的路径
		job.setJarByClass(TwoShareFriendsDriver.class);
	
		// 3 指定map/reduce使用的类
		job.setMapperClass(TwoShareFriendsMapper.class);
		job.setReducerClass(TwoShareFriendsReducer.class);
		
		// 4 指定map输出的数据类型
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		
		// 5 指定最终输出的数据类型
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		
		// 6 指定job的输入原始所在目录
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		
		// 7 提交
		boolean result = job.waitForCompletion(true);
		System.exit(result?0:1);
	}
}
第8章 常见错误及解决方案
1）导包容易出错。尤其Text和CombineTextInputFormat。
2）Mapper中第一个输入的参数必须是LongWritable或者NullWritable，不可以是IntWritable.  报的错误是类型转换异常。
3）java.lang.Exception: java.io.IOException: Illegal partition for 13926435656 (4)，说明Partition和ReduceTask个数没对上，调整ReduceTask个数。
4）如果分区数不是1，但是reducetask为1，是否执行分区过程。答案是：不执行分区过程。因为在MapTask的源码中，执行分区的前提是先判断ReduceNum个数是否大于1。不大于1肯定不执行。
5）在Windows环境编译的jar包导入到Linux环境中运行，
hadoop jar wc.jar com.atguigu.mapreduce.wordcount.WordCountDriver /user/atguigu/ /user/atguigu/output
报如下错误：
Exception in thread "main" java.lang.UnsupportedClassVersionError: com/atguigu/mapreduce/wordcount/WordCountDriver : Unsupported major.minor version 52.0
原因是Windows环境用的jdk1.7，Linux环境用的jdk1.8。
解决方案：统一jdk版本。
6）缓存pd.txt小文件案例中，报找不到pd.txt文件
原因：大部分为路径书写错误。还有就是要检查pd.txt.txt的问题。还有个别电脑写相对路径找不到pd.txt，可以修改为绝对路径。
7）报类型转换异常。
通常都是在驱动函数中设置Map输出和最终输出时编写错误。
Map输出的key如果没有排序，也会报类型转换异常。
8）集群中运行wc.jar时出现了无法获得输入文件。
原因：WordCount案例的输入文件不能放用HDFS集群的根目录。
9）出现了如下相关异常
Exception in thread "main" java.lang.UnsatisfiedLinkError: org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Ljava/lang/String;I)Z
	at org.apache.hadoop.io.nativeio.NativeIO$Windows.access0(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$Windows.access(NativeIO.java:609)
	at org.apache.hadoop.fs.FileUtil.canRead(FileUtil.java:977)
java.io.IOException: Could not locate executable null\bin\winutils.exe in the Hadoop binaries.
	at org.apache.hadoop.util.Shell.getQualifiedBinPath(Shell.java:356)
	at org.apache.hadoop.util.Shell.getWinUtilsPath(Shell.java:371)
	at org.apache.hadoop.util.Shell.<clinit>(Shell.java:364)
解决方案：拷贝hadoop.dll文件到Windows目录C:\Windows\System32。个别同学电脑还需要修改Hadoop源码。
方案二：创建如下包名，并将NativeIO.java拷贝到该包名下

10）自定义Outputformat时，注意在RecordWirter中的close方法必须关闭流资源。否则输出的文件内容中数据为空。
@Override
public void close(TaskAttemptContext context) throws IOException, InterruptedException {
		if (atguigufos != null) {
			atguigufos.close();
		}
		if (otherfos != null) {
			otherfos.close();
		}
}















## Hadoop扩展



#### Yarn源码分析之参数mapreduce.job.reduce.slowstart.completedmaps介绍



