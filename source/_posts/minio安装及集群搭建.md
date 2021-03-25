---
title: minio安装及集群搭建
date: 2020-05-28 20:25:30
comments: true
tags:
    - java
    - minio
    - linux
---
Minio 是一个基于Go语言的对象存储服务。它实现了大部分亚马逊S3云存储服务接口，可以看做是是S3的开源版本，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。

|              项目                   |        参数       |
| :---------------------------------: | :---------------: |
|    最大驱动器数量                   |       16          |
|    最小驱动器数量                   |        4          |
|    读仲裁                           |       n/2         |
|    写仲裁                           |       n/2+1       |
|    最大桶数	                      |       无限制      |
|    每桶最大对象数                   |       无限制      |
|    最大对象大小                     |        5TB        |
|    最小对象大小                     |         0         |
|    每次 PUT 操作的最大对象大小	  |       5GB         |
|    每次上传的最大 Part 数量	      |       10000       |
|     Part 大小	                      | 5MB到5GB. 最后一个part可以从0B到5GB|
| 每次list parts请求可返回的part最大数量 |   1000         |
| 每次list objects请求可返回的object最大数量 | 1000       |
| 每次list multipart uploads请求可返回的multipart uploads最大数量	| 1000    |		





## 一.minio基本原理

**1.minio基本特点**
{% blockquote %}
**高性能**：作为高性能对象存储，在标准硬件条件下它能达到55GB/s的读、35GG/s的写速率 
**可扩容**：不同MinIO集群可以组成联邦，并形成一个全局的命名空间，并跨越多个数据中心
**云原生**：容器化、基于K8S的编排、多租户支持
**Amazon S3兼容**：Minio使用Amazon S3 v2 / v4 API。可以使用Minio SDK，Minio Client，AWS SDK和AWS CLI访问Minio服务器。
**可对接后端存储**: 除了Minio自己的文件系统，还支持DAS、 JBODs、NAS、Google云存储和Azure Blob存储。
**SDK支持**: 基于Minio轻量的特点，它得到类似Java、Python或Go等语言的sdk支持
**Lambda计算**: Minio服务器通过其兼容AWS SNS / SQS的事件通知服务触发Lambda功能。支持的目标是消息队列，如Kafka，NATS，AMQP，MQTT，Webhooks以及Elasticsearch，Redis，Postgres和MySQL等数据库。
**有操作页面**
**功能简单**: 这一设计原则让MinIO不容易出错、更快启动
**支持纠删码**：MinIO使用纠删码、Checksum来防止硬件错误和静默数据污染。在最高冗余度配置下，即使丢失1/2的磁盘也能恢复数据
{% endblockquote %}

**2.minio的存储机制**

Minio使用纠删码erasure code和校验和checksum来保护数据免受硬件故障和无声数据损坏。 即便丢失一半数量（N/2）的硬盘，仍然可以恢复数据。

**3.minio的纠删码**

纠删码是一种恢复丢失和损坏数据的数学算法，目前，纠删码技术在分布式存储系统中的应用主要有三类，阵列纠删码（Array Code: RAID5、RAID6 等）、RS(Reed-Solomon)里德-所罗门类纠删码和 LDPC(LowDensity Parity Check Code)低密度奇偶校验纠删码。Erasure Code 是一种编码技术，它可以将 n 份原始数据，增加 m 份数据，并能通过 n+m 份中的任意 n 份数据，还原为原始数据。即如果有任意小于等于 m 份的数据失效，仍然能通过剩下的数据还原出来

**4.minio的擦除代码**

MinIO 使用每个对象的内联擦除编码来保护数据，这种编码是用汇编代码编写的，可以提供尽可能高的性能。MinIO 使用 Reed-Solomon 代码将对象条带化为 n/2 数据和 n/2 奇偶校验块——尽管这些可以配置为任何所需的冗余级别。这意味着在 12 个驱动器设置中，一个对象被分割为 6 个数据和 6 个奇偶校验块。即使您丢失了 5 个(n/2) -1 个驱动器，无论是奇偶校验还是数据，您仍然可以从剩余驱动器可靠地重构数据。MinIO 的实现确保即使多个设备丢失或不可用，也可以读取对象或写入新对象。最后，MinIO 的擦除代码在对象级别，可以一次治愈一个对象。

## 二.minio安装

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
./minio server /data
```
后台运行
```bash
nohup /usr/local/bin/minio server  /home/minio/data > /home/minio/data/minio.log 2>&1 &
```
minio默认端口9000，直接访问ip:9000,如图：
<div style="width:70%;margin:auto">{% asset_img minio1.png %}</div>

## 三.minio使用
引入minio客户端jar包
```java
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>7.0.2</version>
</dependency>
```
{% blockquote 访问—— https://github.com/root1761/bigdata/tree/main/minio  minio%}
minio相关的代码及具体使用：
{% endblockquote %}
## 四. minio分布式集群搭建
生产环境下最少4个节点

| 节点     |   IP       |       data         |
| :-----:  | :--------: | :----------------: |
| minio1   |  10.0.0.1  | /home/minio/data   |
| minio2   |  10.0.0.2  | /home/minio/data   | 
| minio3   |  10.0.0.3  | /home/minio/data   |
| minio4   |  10.0.0.4  | /home/minio/data   |


1.初始化ACCESS KEY 和SECRET KEY

```bash
export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=admin1234
```
2.搭建minio
```bash
./minio server --address :9000 http://10.0.0.1/home/minio/data \
                               http://10.0.0.2/home/minio/data \
                               http://10.0.0.3/home/minio/data \
                               http://10.0.0.4/home/minio/data \
```
注意：minio搭建机器时间差距不能过大，使用以下指令查看和修改当前系统时间:
```bash 
date
date -s '2020-06-11 21:02:00'
```
## 五.脚本启动minio
**1.创建run.sh 文件**
```bash
touch run.sh
```
**2.编辑run.sh 文件**
```bash
#!/bin/bash
export MINIO_CACHE="on"
export MINIO_CACHE_DRIVES="http://10.0.0.1/home/minio/data,http://10.0.0.2/home/minio/data,http://10.0.0.3/home/minio/data,http://10.0.0.4/home/minio/data"
export MINIO_CACHE_QUOTA=80
export MINIO_CACHE_AFTER=3
export MINIO_CACHE_WATERMARK_LOW=70
export MINIO_CACHE_WATERMARK_HIGH=90
export MINIO_ACCESS_KEY=admin
export MINIO_SECRET_KEY=admin1234

chmod a+x /hom/minio/data

./minio server --address :9000 \
http://10.0.0.1/home/minio/data \
http://10.0.0.2/home/minio/data \
http://10.0.0.3/home/minio/data \
http://10.0.0.4/home/minio/data \
```
**3.minio.service** 
WorkingDirectory:二进制文件
ExecStart:集群启动脚本
```bash
cat > /usr/lib/systemd/system/minio.service <<EOF
[Unit]
Description=Minio service
Documentation=https://docs.minio.io/

[Service]
WorkingDirectory=/home/minio/
ExecStart=/home/minio/run.sh

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
**4.权限管理**
```bash
chmod +x data && chmod +x run.sh
chmod +x /usr/lib/systemd/system/minio.service
```
**5.启动minio**
```bash
systemctl daemon-reload
systemctl start minio
```

开机自启动

```bash
systemctl enable minio.service 
```
## 六.nginx动态代理

修改nginx的conf文件
```bash
upstream http_minio {
    server 10.0.0.1:9000 weight=25 max_fails=2 fail_timeout=30s;
    server 10.0.0.2:9000 weight=25 max_fails=2 fail_timeout=30s;
    server 10.0.0.3:9000 weight=25 max_fails=2 fail_timeout=30s;
    server 10.0.0.4:9000 weight=25 max_fails=2 fail_timeout=30s;

}

server{
    listen       9001;
    server_name  10.0.0.1;

    ignore_invalid_headers off;
    client_max_body_size 0;
    proxy_buffering off;

    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-Host  $host:$server_port;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto  $http_x_forwarded_proto;
        proxy_set_header   Host $http_host;

        proxy_connect_timeout 300;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_ignore_client_abort on;

        proxy_pass http://http_minio;
    }
}
```
进入sbin目录下重启nginx即可
```bash
./nginx -s reload
```

访问10.0.0.1:9001