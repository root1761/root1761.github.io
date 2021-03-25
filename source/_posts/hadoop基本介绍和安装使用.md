---
title: hadoop基本介绍和安装使用
date: 2020-07-26 09:01:11
tags:
  - linux
  - hadoop
  - java
---

## 一.hadoop基本介绍

Hadoop由HDFS、MapReduce、HBase、Hive和ZooKeeper等成员组成，其中最重要的是底层用于存储集群中所有存储节点文件的文件系HDFS（Hadoop Distributed File System）和上层用来执行MapReduce程序的MapReduce引擎
<div style="margin:auto;width:90%" >{% asset_img hadoop_1.png%}</div>
{% blockquote %}
1、Pig是一个基于Hadoop的大规模数据分析平台，Pig为复杂的海量数据并行计算提供了一个简易的操作和编程接口
2、Chukwa是基于Hadoop的集群监控系统，由yahoo贡献
hive是基于Hadoop的一个工具，提供完整的sql查询功能，可以将sql语句转换为MapReduce任务进行运行
3、ZooKeeper：高效的，可扩展的协调系统,存储和协调关键共享状态
4、HBase是一个开源的，基于列存储模型的分布式数据库
5、HDFS是一个分布式文件系统。有着高容错性的特点，并且设计用来部署在低廉的硬件上，适合那些有着超大数据集的应用程序
6、MapReduce是一种编程模型，用于大规模数据集（大于1TB）的并行运算
{% endblockquote %}

下图是一个典型的Hadoop试验集群的部署结构：

<div style="margin:auto;width:90%">{% asset_img hadoop_2.png%}</div>

Hadoop组件之间的依赖关系：

<div style="margin:auto;width:90%">{% asset_img hadoop_3.png%}</div>

## 二.Hadoop的核心架构

<div style="margin:auto;width:90%">{% asset_img hadoop_4.png%}</div>

**1.HDFS**
---
HDFS是一个高度容错性的分布式文件系统，可以被广泛的部署于廉价的PC上。它以流式访问模式访问应用程序的数据，这大大提高了整个系统的数据吞吐量，因而非常适合用于具有超大数据集的应用程序中。 
HDFS的架构如图所示。HDFS架构采用主从架构（master/slave）。一个典型的HDFS集群包含一个NameNode节点和多个DataNode节点。NameNode节点负责整个HDFS文件系统中的文件的元数据的保管和管理，集群中通常只有一台机器上运行NameNode实例，DataNode节点保存文件中的数据，集群中的机器分别运行一个DataNode实例。在HDFS中，NameNode节点被称为名称节点，DataNode节点被称为数据节点。DataNode节点通过心跳机制与NameNode节点进行定时的通信。 
<div style="margin:auto;width:90%">{% asset_img hadoop_5.png%}</div>
{% blockquote %}

**1、NameNode**
**可以看作是分布式文件系统中的管理者，存储文件系统的meta-data，主要负责管理文件系统的命名空间，集群配置信息，存储块的复制。** 
 &emsp;fsimage - 它是在NameNode启动时对整个文件系统的快照
 &emsp;edit logs - 它是在NameNode启动后，对文件系统的改动序列<br/>
**2、DataNode** 
**是文件存储的基本单元。它存储文件块在本地文件系统中，保存了文件块的meta-data，同时周期性的发送所有存在的文件块的报告给NameNode。**<br/>
**3、Secondary NameNode**
**合并NameNode的edit logs到fsimage文件中。**<br/>
**4、Client**
**就是需要获取分布式文件系统文件的应用程序。**
{% endblockquote %}

### HDFS读写操作：
文件写入：
   &emsp;&emsp; 1.Client向NameNode发起文件写入的请求
   &emsp;&emsp; 2.NameNode根据文件大小和文件块配置情况，返回给Client它所管理部分DataNode的信息。
   &emsp;&emsp; 3.Client将文件划分为多个文件块，根据DataNode的地址信息，按顺序写入到每一个DataNode块中
<div style="margin:auto;width:90%">{% asset_img hadoop_6.png%}</div>
文件读取：<br/>
&emsp;&emsp;1.Client向NameNode发起文件读取的请求<br/> 
&emsp;&emsp;2. NameNode返回文件存储的DataNode的信息。<br/> 
&emsp;&emsp;3. Client读取文件信息。
<div style="margin:auto;width:90%">{% asset_img hadoop_7.png%}</div>

**2.MapReducer**
---
MapReduce是一种编程模型，用于大规模数据集的并行运算。Map（映射）和Reduce（化简），采用分而治之思想，先把任务分发到集群多个节点上，并行计算，然后再把计算结果合并，从而得到最终计算结果。多节点计算，所涉及的任务调度、负载均衡、容错处理等，都由MapReduce框架完成。
下图是MapReducer的处理过程：
<div style="margin:auto;width:90%">{% asset_img hadoop_8.png%}</div>

用户提交任务给JobTracer，JobTracer把对应的用户程序中的Map操作和Reduce操作映射至TaskTracer节点中；输入模块负责把输入数据分成小数据块，然后把它们传给Map节点；Map节点得到每一个key/value对，处理后产生一个或多个key/value对，然后写入文件；Reduce节点获取临时文件中的数据，对带有相同key的数据进行迭代计算，然后把终结果写入文件。
如果这样解释还是太抽象，可以通过下面一个具体的处理过程来理解：（WordCount实例） 

<div style="margin:auto;width:90%">{% asset_img hadoop_9.png%}</div>

Hadoop的核心是MapReduce，而MapReduce的核心又在于map和reduce函数。它们是交给用户实现的，这两个函数定义了任务本身。

map函数：接受一个键值对（key-value pair）（例如上图中的Splitting结果），产生一组中间键值对（例如上图中Mapping后的结果）。Map/Reduce框架会将map函数产生的中间键值对里键相同的值传递给一个reduce函数。 
reduce函数：接受一个键，以及相关的一组值（例如上图中Shuffling后的结果），将这组值进行合并产生一组规模更小的值（通常只有一个或零个值）（例如上图中Reduce后的结果）

但是，Map/Reduce并不是万能的，适用于Map/Reduce计算有先提条件： 
（1）待处理的数据集可以分解成许多小的数据集； 
（2）而且每一个小数据集都可以完全并行地进行处理；

**3.Yarn**
---
Apache Hadoop YARN 是开源 Hadoop 分布式处理框架中的资源管理和作业调度技术。作为 Apache Hadoop 的核心组件之一，YARN 负责将系统资源分配给在 Hadoop 集群中运行的各种应用程序，并调度要在不同集群节点上执行的任务。
YARN 的基本思想是将资源管理和作业调度/监视的功能分解为单独的 daemon(守护进程)，其拥有一个全局 ResourceManager(RM) 和每个应用程序的 ApplicationMaster(AM)。应用程序可以是单个作业，也可以是作业的 DAG。
<div style="margin:auto;width:90%">{% asset_img hadoop_10.webp%}</div>

yarn的基本组成：
ResourceManager是Master上一个独立运行的进程，负责集群统一的资源管理、调度、分配等等；
NodeManager是Slave上一个独立运行的进程，负责上报节点的状态；
ApplicationMaster相当于这个Application的监护人和管理者，负责监控、管理这个Application的所有Attempt在cluster中各个节点上的具体运行，同时负责向Yarn ResourceManager申请资源、返还资源等；
Container是yarn中分配资源的一个单位，包涵内存、CPU等等资源，YARN以Container为单位分配资源；

## 三.hadoop安装和使用

**1.hadoop准备工作，ssh免密登录**

检查不通过密码ssh到本地主机
```bash
ssh localhost
```
如果不能进行如下操作
```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat〜/ .ssh / id_rsa.pub >>〜/ .ssh / authorized_keys
chmod 0600〜/ .ssh / authorized_keys
```
安装java的环境变量JDK

**2.安装hadoop**
```bash
wget https://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.10.0/hadoop-2.10.0.tar.gz
tar -zxvf hadoop-2.10.0.tar.gz
```

**3.hadoop伪分布式搭建**

1.hdfs:

(1)编辑文件etc/hadoop-env.sh设置java的环境变量
```bash
export JAVA_HOME=JDK安装路径
```
(2)编辑文件/hadoop/core-site.xml
```bash
 <property>
        <name>fs.defaultFS</name>
        <value>hdfs://主机名:9000</value>
    </property>
	<property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/hadoop-2.10.0/tmp-${user.name}</value>
  </property>
```
(3)编辑文件/hadoop/hdfs-site.xml
```bash
 <property>
        <name>dfs.replication</name>
        <value>1</value>
 </property>
```
(5)格式化namenode
```bash
./bin/hdfs namenode -format
```
(6)启动datanode和namenode
```bash
./sbin / start-dfs.sh
```
完成后可以通过以下命令关闭：
```bash
  ./sbin / stop-dfs.sh
```

浏览Web界面的NameNode；默认情况下，它在以下位置可用：

http://ip:50070

hds的基本命令：

{% blockquote %}

./bin/hadoop fs -touchz /hdfs/aa.t&emsp;&emsp;   //在hdfs中创建一个大小为0的文件
./bin/hadoop fs -touchz /usr/a.txt /hdfs/dest.t &emsp;&emsp;  //将本地文件内容追加到hdfs中的特定文件中
./bin/hadoop fs -cat /hdfs/f1.t /hdfs/f2.t&emsp;&emsp;//将f1.t和f2.t文件内容合并现实查看
./bin/hadoop fs -chmod -R a+x /hdfs/f3/*   &emsp;&emsp; //递归更改特定目录下文件的权限。权限表示和linux shell一样
./bin/hadoop fs -count /hdfs/f1.t &emsp;&emsp;   //查看path文件数量相关信息
./bin/hadoop fs -cp /hdfs/f1.t /hdfs/f1   &emsp;&emsp;//在hdfs系统中进行文件copy
./bin/hadoop fs -du /hdfs/f1.t &emsp;&emsp;  //计算文件占用磁盘大小
./bin/hadoop fs /hdfs/* /usr/ &emsp;&emsp;   //将hdfs系统中的文件下载到本地系统
./bin/hadoop fs -ls /hdfs   &emsp;&emsp;  //现实特定目录下的所有文件
./bin/hadoop fs -mkdir /hdfs/f2 &emsp;&emsp;  //在hdfs中创建文件夹
./bin/hadoop fs /hdfs/* /hdfs/f2 &emsp;&emsp;  //在hdfs中完成文件的移动
./bin/hadoop fs -put /usr/aa.t /hdfs/  &emsp;&emsp; //将本地文件上传到hdfs系统中
./bin/hadoop fs -rm /hdfs/f1.t  &emsp;&emsp;  //删除特定文件
./bin/hadoop fs -tail /hdfs/f1.t  &emsp;&emsp;  //查看文件最后的1k数据
./bin/hadoop fs -text /hdfs/f1.zip &emsp;&emsp;   //以二进制的形式查看文件内容
./bin/hadoop fs -usage text   &emsp;&emsp; //查看text指令的使用方式

{% endblockquote %}

2.MapReducer

注意：MapReducer是在HDFS的基础上搭建的

(1)编辑etc / hadoop / mapred-site.xml文件
```bash
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>

```
(2)编辑etc/hadoop/yarn-site.xml文件
```bash
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
```
(3)启动nodeManager和ResourceManager
```bash
  ./sbin/start-yarn.sh
```
(4)完成后可以通过以下方式关闭
```bash
./sbin/stop-yarn.sh
```
(5)启动jar测试mapReducer
```bash
./bin/hadoop jar ./share/hadoop/mapreducehadoop-mapreduce-examples-2.10.0 wordcount /dir1/a.txt /result
```
/dir1/a.txt：读取文件目录
/result：输出结果目录
浏览Web界面以查找ResourceManager；默认情况下，它在以下位置可用：
http://ip:8088
{% blockquote 访问—— https://github.com/root1761/bigdata/tree/master/hadoop hadoop%}
hadoop相关的代码案例及具体使用：
{% endblockquote %}

## 四.hadoop集群搭建
| 节点     |   IP       |  namenode |  datanode  |  journalnode(替代secondary namenode) |yarn  |
| :-----:  | :--------: | :-------: | :--------: | :----------: | :-------: |
| hadoop01 |  10.0.0.1  |     √     |     √      |      √       |     √     |
| hadoop02 |  10.0.0.2  |     √     |     √      |      √       |           |
| hadoop03 |  10.0.0.3  |           |     √      |      √       |           |

**1.hadoop多台机器间免密登陆**
(1)添加主机名和ip地址映射
```bash
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.0.1 hadoop01
10.0.0.2 hadoop02
10.0.0.3 hadoop03
```
(2)产生免密码登录的公私钥对
```bash
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
```
(3)将公钥拷贝到统一的机器上
```bash
scp /root/.ssh/id_dsa.pub root@hadoop01:/root/.ssh/id_1_dsa.pub
scp /root/.ssh/id_dsa.pub root@hadoop01:/root/.ssh/id_2_dsa.pub
```
(4)将公钥添加信任列表中
```bash
cat /root/.ssh/id_1_rsa.pub >> /root/.ssh/authorized_keys
cat /root/.ssh/id_2_rsa.pub >> /root/.ssh/authorized_keys
```
(5)将信任列表进行同步
```bash
scp /root/.ssh/authorized_keys root@hadoop02:/root/.ssh/
scp /root/.ssh/authorized_keys root@hadoop03:/root/.ssh/
```
(6)赋予权限
ssh目录的权限必须是700
ssh/authorized_keys文件权限必须是600
```bash
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
```
**2.hadoop集群搭建准备工作**

1.准备三台服务器
2.准备机架配置文件

rack.sh
```bash
while [ $# -gt 0 ] ; do
  nodeArg=$1
  exec</usr/hadoop-2.10.0/etc/hadoop/topology.data 
  result="" 
  while read line ; do
    ar=( $line ) 
    if [ "${ar[0]}" = "$nodeArg" ] ; then
      result="${ar[1]}"
    fi
  done 
  shift 
  if [ -z "$result" ] ; then
    echo -n "/defaulta/rack_"
  else
    echo -n "$result "
  fi
done 
```
topology.data
```bash
10.0.0.1     /dc1_rack1
10.0.0.2     /dc1_rack2
10.0.0.3     /dc1_rack3
```
3.zookeeper集群

**3.hadoop集群搭建**

(1)配置core-site.xml
```bash
<property>
<name>fs.defaultFS</name>
<value>hdfs://mycluster</value> //mycluster是对namenode做命名服务
</property>
<property>
<name>fs.trash.interval</name> //开启回收站
<value>10</value>
</property>
<property>
<name>net.topology.script.file.name</name> //机架配置
<value>/usr/hadoop-2.10.0/etc/hadoop/rack.sh</value>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/usr/hadoop-2.10.0/tmp-${user.name}</value>
</property> 
```
(2)配置hdfs-site.xml
```bash
<property>
<name>dfs.replication</name>
<value>3</value>
</property> 
<property>
<name>dfs.nameservices</name>
<value>mycluster</value>
</property>
<property>
<name>dfs.ha.namenodes.mycluster</name>
<value>nn1,nn2</value>
</property>
<property>
<name>dfs.namenode.rpc-address.mycluster.nn1</name>
<value>hadoop01:8020</value>
</property>
<property>
<name>dfs.namenode.rpc-address.mycluster.nn2</name>
<value>hadoop02:8020</value>
</property>
<property>
<name>dfs.namenode.http-address.mycluster.nn1</name>
<value>hadoop01:50070</value>
</property>
<property>
<name>dfs.namenode.http-address.mycluster.nn2</name>
<value>hadoop02:50070</value>
</property>
<property>
<name>dfs.namenode.shared.edits.dir</name>
<value>qjournal://hadoop01:8485;hadoop02:8485;hadoop03:8485/mycluster</value>
</property>
<property>
<name>dfs.client.failover.proxy.provider.mycluster</name> <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence</value>
</property>
<property>
<name>dfs.ha.fencing.ssh.private-key-files</name>
<value>/root/.ssh/id_dsa</value>
</property> 
<property>
<name>dfs.ha.automatic-failover.enabled</name>
<value>true</value>
</property>
<property> 
<name>ha.zookeeper.quorum</name>
<value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value> --zookeeper
</property>
```
(3)配置datanode信息，修改/etc/hadoop/slaves
```bash
hadoop01
hadoop02
hadoop03
```
(4)配置yarn，修改mapred-site.xml
```bash
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
```
修改yarn-site.xml
```bash
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.resourcemanager.hostname</name>
<value>hadoop01</value>
</property>
```
(5)将以上文件进行同步：
```bash
scp /usr/hadoop-2.10.0/etc/hadoop/* root@hadoop02:/usr/hadoop-2.10.0/etc/hadoop/
scp /usr/hadoop-2.10.0/etc/hadoop/* root@hadoop03:/usr/hadoop-2.10.0/etc/hadoop/
```
**4.hadoop集群启动**

(1)启动zookeeper集群

(2)启动各个节点的journalnode
```bash
/usr/hadoop-2.10.0/sbin/hadoop-daemon.sh start journalnode
```
(3)格式化hadoop01上的namnode
```bash
/usr/hadoop-2.10.0/bin/hdfs namenode -format
```
(4)启动hadoop01上的namenode
```bash
/usr/hadoop-2.10.0/sbin/hadoop-daemon.sh start namenode
```
(5)在hadoop02上引导格式化namnode
```bash
/usr/hadoop-2.10.0/bin/hdfs namenode -bootstrapStandby
```
(6)启动hadoop2上的namnode
```bash
/usr/hadoop-2.10.0/sbin/hadoop-daemon.sh start namenode
```
(7)注册namenode到zookeeper 只需要在任意一台namenode上执行
```bash
./hadoop-2.10.0/bin/hdfs zkfc -formatZK
```
(8)分别在两个namenode上启动zkfc监视器
```bash
./hadoop-2.10.0/sbin/hadoop-daemon.sh start zkfc
```
(9)分别在各个节点上启动datanode
```bash
./hadoop-2.10.0/sbin/hadoop-daemon.sh start datanode
```
(10)在hadoop01上启动yarn
```bash
./hadoop-2.10.0/sbin/start-yarn.sh
```
(11)集群创建完成后以后可以通过快捷方式启动和关闭
注意：namenode和ResourceManager必须在同一节点上才能使用start-all.sh
```bash
./hadoop-2.10.0/sbin/start-all.sh
./hadoop-2.10.0/sbin/stop-all.sh
```