---
title: Hadoop是一个海量数据存储和计算的平台
tags: Hadoop
categories: 大数据
---
# Hadoop是一个海量数据存储和计算的平台
#### 分布式存储
![Alt text](/img/HadoopDistributedStorageArchitecture.png.png)
#### 分布式计算
移动计算：把计算程序移动到数据端进行计算
![Alt text](/img/HadoopDistributedComputing.png)
## 三大核心组件
### HDFS
HDFS负责海量数据的分布式存储 <br>
支持主从结构，主节点支持多个NameNode,从节点支持多个DataNode <br>
NamdNode负责接收用户请求，维护目录系统的目录结构
DateNode主要负责存储数据
### MapReduce
MapReduce是一个编程模型，主要负责海量数据计算，由Map和Reduce两个阶段组成 <br>
Map阶段是一个独立的程序，会在很多个节点同时执行，每个节点处理一部分数据 <br>
Reduce阶段是一个单独的聚合程序
### Yarn
Yarn负责集群资源的管理和调度，支持主从架构，主节点最多可以有两个，从节点可以有多个 <br>
主节点（ResourceManager）进程主要负责集群资源的分配和管理 <br>
从节点（NodeManager）主要负责单节点资源管理
![Alt text](/img/HadoopBigdataEcosystem.png)
## Hadoop发行版介绍
官方版本：Apache Hadoop 开源，集群安装维护比较麻烦 <br>
第三方发行版本：Cloudera hadoop(CDH) 商业版，使用Cloudera Manager安装维护比较方便 <br>
第三方发行版本：HortenWorks (HDP) 开源版，使用Ambari安装维护比较方便  <br>
