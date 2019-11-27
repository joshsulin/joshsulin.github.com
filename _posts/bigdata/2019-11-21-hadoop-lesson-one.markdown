---
layout: post
title:  "Hadoop第一课(Hadoop核心及环境搭建)"
date:   2019-11-21 20:19:20 +0800
categories: SpringBoot
tags: ['Hadoop']
---

### 一、Hadoop是什么

Hadoop是一个开源的大数据框架, Hadoop是一个分布式计算的解决方案. Hadoop=`HDFS(分布式文件系统)`+`MapReduce(分布式计算)`

#### 1、Hadoop核心

A、HDFS分布式文件系统: 存储是大数据技术的基础. 如果连存储都搞不定的话, 你还谈什么大数据.

B、MapReduce编程模型: 分布式计算是大数据应用的解决方案.

举个例子: 假如我有个100M的文件, 如果我想通过过滤找出含有Hadoop字符串的行, 应该怎么办呢?

方案1: Linux grep命令

方案2: 编写Java/Python程序

以上方案都可以搞定, 100M并不会让你感觉到绝望, 假如100T呢, 或者是100P呢, 这个时候你能怎么办呢? 就算你的程序能跑出结果, 你又需要多少时间呢? 

`其实Hadoop这两个核心它就解决了这个问题, HDFS解决了数据存储的问题, MapReduce解决了分布式计算的问题, 强强联手, 不说100P了, 再来100P它也能搞定. `

#### 2、HDFS总结

HDFS为什么能称之为Hadoop的核心呢? 因为它有很多特性来支持大数据的存储, 因为存储是大数据的基础, 我们知道单机挂个1T或者2T的硬盘都不是大问题, 那如果是几百个T呢, 1PB呢 单机还能扛得住吗? HDFS实际上是为大量数据能横跨成百上千台机器, 但是你用起来就像使用本地文件一样简单. 比如说你要获取某一个文件的数据, 但是这个数据存储到很多台机器上, 你作为用户你都不用考虑它到底在哪台机器上, HDFS自动就为你搞定了.

`A、普通的成百上千台机器.`

`B、按TB甚至PB为单位的大量数据.`

`C、简单便捷的文件获取.`

那到底数据是怎么存储在HDFS上呢? 我们就要简单的了解一下HDFS了.

#### 3、HDFS概念.

`3.1、数据块`

我们都知道存储在HDFS上的数据都是按块存储的, 数据块是什么呢? 在HDFS上, 数据块就是一个抽象的概念, 数据块是抽象块而非整个文件作为存储单元. 块默认大小为64MB, 一般设置为128M, 备份设置为3个。

比如我要存储一个文件10M, 由于数据是按块存储的, 所以这个文件会独占一个数据块.

`3.2、NameNode`

我们提到HDFS是分布式的文件系统, 既然是分布式的, 就是主从模式, 那么谁是主、谁是从呢, NameNode就是主, DataNode就是从。所以说HDFS是由1个NameNode和多个DataNode组成的. 

NameNode做哪些工作呢?

A、管理文件系统的命名空间, 存放文件元数据.

B、维护着文件系统的所有文件和目录, 文件与数据块的映射.

C、记录每个文件中各个块所在数据节点的信息. 这些信息在DataNode启动的时候, 会发送给NameNode. 这里要思考一个问题了, 如果NameNode挂掉了怎么办? 大家可以思考一下, 这也是分布式系统中常常存在的单点问题.

`3.3、DataNode`

DataNode是文件系统中的工作节点, 它就是负责存储的, 它负责存储并检索数据块, 并且将它存储的列表发送给NameNode进行更新。

#### 4、HDFS的优缺点

通过对HDFS的学习, 我们可以总结一下HDFS的优缺点.

`A、优点`

适合大文件的存储, 支持TB、PB级的数据存储, 并有副本策略. 

可以构建在廉价的机器上, 并有一定的容错和恢复机制.

支持流式数据访问, 一次写入, 多次读取最高效.

`B、缺点`

不适合小文件的存储.

不适合并发写入, 不支持文件随机修改. 

不支持随机读等低延时的访问方式.

### 讨论问题1: 数据块大小设置成多大合适呢?

数据块一般我们将其设置为128M, 因为如果数据块设置成大小的话, 一般的文件也会被分割成多个数据块, 那么在访问的时候就会查找多个数据块的地址, 这样的效率肯定是不高的, 而且如果数据块设置成太小的话, 对NameNode的内存消耗会比较严重, 我们都知道NameNode存储了整个集群的数据信息, 相对于那么多的DataNode, NameNode的内存压力是比较大的, 那么反过来说, 数据块设置的过大的话, 对于并行的支持就不会太好了, 可能会涉及到系统的其它的问题, 系统重启的时候, 会重新加载数据, 数据块越大, 数据恢复的时候就越长。

### 讨论问题2: 如果NameNode挂掉了怎么办?

NameNode挂掉了, 确定是一件比较可怕的事情, 因为你没有办法将位于多个DataNode的数据块组建成一个文件, 之前Hadoop容错的机制我们就不多说了, 我们说一下Hadoop 2.X的容错机制, 现在的Hadoop2.X可以配置成HA, 高可用的集群。集群里面有2个NameNode的节点, 一台处于Active状态为主节点, 其它处于standby状态的为备用节点, 两者数据时刻保持一致, 当主节点出现问题的时候, 备用节点就可以自动切换. 用户基于感知不到. 这样就避免了NameNode的单点问题.

### 配置伪分布式环境

#### 配置文件

`Hadoop版本: 2.7.3`

Hadoop配置文件: etc/hadoop

需要更改的配置文件: `hadoop-env.sh`、`hdfs-site.xml`、`core-site.xml` 这三个文件是与HDFS相关的, Hadoop有两个核心, HDFS与mapreduce。MapReduce需要更改yarn-site.xml、mapred-site.xml.template, 但是这两个我们现在不去更改它, 我们只去配置刚才提到的与HDFS相关的这3个配置文件. 

`1、更改hadoop-env.sh.`

```Java
export JAVA_HOME=${JAVA_HOME}
```

`2、更改hdfs-site.xml.`

打开文件默认为:
```Java
<configuration>

</configuration>
```

配置成伪分布式集群, 需要做如下配置:

```Java
<configuration>
	<property>
			<name>dfs.replication</name>
			<value>1</value>
	</property>
	<property>
			<name>dfs.namenode.name.dir</name>
			<value>file:/Users/sulin/project/bigdata/hadoop_data/dfs/name</value>
	</property>
	<property>
			<name>dfs.datanode.data.dir</name>
			<value>file:/Users/sulin/project/bigdata/hadoop_data/dfs/data</value>
	</property>
</configuration>
```

dfs.replication=1 表示存储副本的数量, 由于我们是伪分布式模式, datanode只有一个节点, 所以这里就设置为1.

dfs.namenode.name.dir namenode相关信息存储路径

dfs.datanode.data.dir datanode相关信息存储路径

再来回顾一下namenode与datanode各自的作用.

`3、更改core-site.xml.`

打开文件默认为:
```Java
<configuration>

</configuration>
```

配置成伪分布式集群, 需要做如下配置:
```Java
<configuration>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>file:/Users/sulin/project/bigdata/hadoop_data</value>
	</property>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://0.0.0.0:9000</value>
	</property>
</configuration>
```

配置了两个属性, tmp.dir、defaultFS. defaultFS里面就是默认的IP和端口.

相关的配置就己经配置完了, 更多的配置就在Hadoop官网上. 

#### 启动Hadoop集群

`1、首先创建数据存储目录.`

```Java
./bin/hdfs namenode -format
```

看到 'INFO util.ExitUtil: Exiting with status 0' 这个信息, 就表示namenode format成功了.

`2、启动集群.`

```Java
./sbin/start-dfs.sh
```

### 思考, 为什么配置Hadoop环境时, 需要配置SSH免密登录?
