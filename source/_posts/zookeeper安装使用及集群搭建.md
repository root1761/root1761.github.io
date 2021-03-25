---
title: zookeeper安装使用及集群搭建
date: 2020-07-08 22:55:48
tags:
    - zookeeper
    - linux
---

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。
## 1.zookeeper安装
```bash
wget https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz
tar -zxvf apache-zookeeper-3.6.1-bin.tar.gz
```
**1.解压完成后修改配置文件，创建并修改存储数据的目录**
```bash
vi zoo.cfg
dataDir=/home/zookeeper/zkdata
```
<div style="margin:auto;width:70%" >{% asset_img zookeeper.png%}</div>

**2.配置完成后，启动zookeeper**
```bash
./bin/zkServer.sh start ./conf/zoo.cfg
```
**3.启动成功后，通过客户端连接zookeeper**
```bash
./bin/zkCli.sh -server 10.0.0.1:2181
```
zookeeper常用命令：
{% blockquote %}
 ls /path  : 查看特定节点下面的子节点
 create /path data:  创建一个节点。并给节点绑定数据
	注意：  默认是持久性节点
create -s /parent/childPath :在父节点下面创建持久性顺序子节点
create -e  /path data  :创建临时性节点
create -e -s /path  data:  创建临时顺序节点		
get /path  :获得节点上绑定的数据信息
set/ path data: 修改节点上绑定的数据。
delete /path  ：删除节点
ls /path true:在查看的时候。给节点设置监听： 监听本节点数据的修改和删除。同时监听子节点的添加和删除
get /path true:  监听本节点数据的修改和删除。同时监听子节点的添加和删除
{% endblockquote %}

## 2. zookeeper分布式集群搭建
 
 生产环境下最少3个节点

 | 节点        |   IP       |       data               |
| :----------: | :--------: | :----------------------: |
| zookeeper1   |  10.0.0.1  | /home/zookeeper/zkdata   |
| zookeeper2   |  10.0.0.2  | /home/zookeeper/zkdata   | 
| zookeeper3   |  10.0.0.3  | /home/zookeeper/zkdata   |

**1.创建存储数据的文件夹**
```bash
mkdir zkdata
touch ./zkdata/myid
```
myid的内容是服务器表示 1|2|3|

**2.修改zookeeper的cfg文件**
```bash
tickTime=2000
dataDir=/home/zookeeper/zkdata
clientPort=2181
initLimit=5
syncLimit=2
server.1=10.0.0.1:2888:3888
server.2=10.0.0.2:2888:3888
server.3=10.0.0.3:2888:3888
```
server.X :x为服务器的唯一标识。
10.0.0.1：服务器所在的ip地址
2888：数据同步使用的端口号
3888：选举使用的端口号

**3.分别启动zookeeper**
```bash
./bin/zkServer.sh start ./conf/zoo.cfg
```
查看zookeeper的状态：
```bash
./bin/zkServer.sh status ./conf/zoo.cfg
```
**4.任意连接一个zookeeper操作**

## 3.zookeeper开机自启动脚本

进入到/etc/init.d/目录下，创建zookeeper文件，复制以下内容：
```bash 
#!/bin/bash
#chkconfig:2345 20 90
#description:zookeeper
#processname:zookeeper
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/jre/
case $1 in
          start) su root /home/zookeeper/bin/zkServer.sh start;;
          stop) su root /home/zookeeper/bin/zkServer.sh stop;;
          status) su root /home/zookeeper/bin/zkServer.sh status;;
          restart) su root /home/zookeeper/bin/zkServer.sh restart;;
          *)  echo "require start|stop|status|restart"  ;;
esac
```
权限管理：
```bash
chmod a+x zookeeper
chkconfig --add zookeeper
```