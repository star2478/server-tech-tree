# 服务端软件技术树概览

IT界发展至今，已经诞生了数以亿计的服务端软件技术，大到操作系统，小到仅一行代码的工具。这些技术共同构筑成了一棵庞大的技术树，不夸张地说，这是现代计算机技术自20世纪30年代发展至今，人类点亮的最耀眼的科技树之一。

从问题驱动的角度来看，每项服务端技术的出发点都是为了解决某个或某几个问题，而所有问题本质上都可归结成若干个共性问题，本文称之为`元问题`。没有哪项技术可以完美解决所有元问题，每项技术都各有偏重。本文将介绍每个元问题领域里常见的技术和理论，希望能为后端技术人员提供一个技术视角，并在实际工作中有所帮助，尤其在技术选型阶段。

```
技术树图
说明下各个技术的极限能力
每项技术的能力偏向饼图（以10个技术维度）
比如，举个例子
遇到一个成本问题，但成本并非元问题，而是结果，经过分析是研发效率问题，xxx，最后用xx指标进行效果评估
运维实际主要解决的是资源管理效率问题
p和np问题
```

## 元问题
* [研发效率](https://github.com/star2478/server-tech-tree/blob/master/研发效率.md)
* [资源管理效率](https://github.com/star2478/server-tech-tree/blob/master/资源管理效率.md)
* [可用性](https://github.com/star2478/server-tech-tree/blob/master/可用性.md)
* [数据](https://github.com/star2478/server-tech-tree/blob/master/数据.md)
* [性能](https://github.com/star2478/server-tech-tree/blob/master/性能.md)
* [伸缩性](https://github.com/star2478/server-tech-tree/blob/master/伸缩性.md)
* [扩展性](https://github.com/star2478/server-tech-tree/blob/master/扩展性.md)
* [安全](https://github.com/star2478/server-tech-tree/blob/master/安全.md)
* [质量](https://github.com/star2478/server-tech-tree/blob/master/质量.md)



主要应用已有的模型和方法解决一些实际问题，我们可以理解为数据挖掘；第二，根据应用问题的需要，提出和发展模型、方法和算法以及研究支撑它们的数学原理或理论基础等，我理解这是机器学习学科的核心内容。从这里，我们看到，数据挖掘和机器学习本质上是一样的，其区别是数据挖掘更接地于数据库端，而机器学习则更接近于智能端。

tdingine，IoT大数据平台，super table创新模型


————— 2019-07-22 —————

cooooo 20:31

RAID 可以看作是一种垂直伸缩，在一台计算机集成更多的磁盘实现大数据存储，hdfs则是横向伸缩


————— 2019-07-23 —————

cooooo 09:49

文件系统可用性：hdfs的容错

常用的保证系统可用性的策略有冗余备份、失效转移和降级限流。


————— 2019-07-24 —————

cooooo 12:51

线程模型：从共享内存，逐渐发展到Actor , Go routine

cooooo 20:57

MapReduce关键能力：代码到数据所在地执行(边缘计算)、map输出的同一个key结果会shuffle到同一个reduce进程节点处理(对key做hash%reduce数，就可保证到同一个ID的reduce上，但当同一个key数据大且reduce数不够时，可能会压垮reduce)

cooooo 21:11

hdfs的namenode和datanode


————— 2019-07-25 —————

cooooo 09:03



cooooo 09:04

注，上图，application master可能和MapReduce作业(图中容器)不在同一个datanode

cooooo 09:09

yarn(对标mesos和docker)资源调度框架：
1.独立部署一个resource manager节点集群，代码提交给它，它通过与nodemanager通讯，将某个MapReduce作业对应的application master程序分派到一个datanode的一个容器中执行，application master向resource manager汇报每个datanode的资源使用情况，resource manager通过fair或capacity算法分配map或reduce任务给不同datanode。resource manager；
2.每个datanode进程节点上部署一个nodemanager进程，将节点分成一个个容器，每个容器都拥有自己的cpu和内存等资源。

注1：ApplicationMaster 向资源管理器申请计算资源时可以指定目标节点（数据分片所在节点），而如果系统资源能够满足，就会把mapreduce计算任务分发到指定的服务器上。如果资源不允许，比如目标节点非常繁忙，这时部分mapreduce计算任务可能会分配另外的服务器（数据分片不在本地，即做不到边缘计算，数据需要传到map和reduce程序所在节点）
注2：为何专门起一个application master来给resource manager汇报和为mr作业申请资源，而非利用node manager(这样一个datanode很可能就有datanode、nodemanager、application master三个进程了)?这是因为一是节点管理(nodemanager)和作业监控/资源申请(application master)进行解耦，可以兼容各种计算框架(MapReduce、spark等)，计算框架只要遵循yarn规范写applicationmaster即可实现自己的资源调度策略，二是每个MapReduce作业都对应一个application master，即每个作业可以定制化自己的资源调度策略，也有默认策略(fair或capacity)。可以说，application master是面向某个map和reduce应用的，而node manager是面向某个datanode的

cooooo 09:12

另一方面，你看一下 Hadoop 几个主要产品的架构设计，就会发现它们都有相似性，都是一主多从的架构方案。HDFS，一个 NameNode，多个 DataNode；MapReduce 1，一个 JobTracker，多个 TaskTracker；Yarn，一个 ResourceManager，多个 NodeManager。

事实上，很多大数据产品都是这样的架构方案：Storm，一个 Nimbus，多个 Supervisor；Spark，一个 Master，多个 Slave。

大数据因为要对数据和计算任务进行统一管理，所以和互联网在线应用不同，需要一个全局管理者。而在线应用因为每个用户请求都是独立的，而且为了高性能和便于集群伸缩，会尽量避免有全局管理者。


————— 2019-07-26 —————

cooooo 08:57

spark：RDD 弹性数据集（Resilient Distributed Datasets），Spark 分布式计算的数据分片、任务调度都是以 RDD 为单位展开的，每个 RDD 分片都会分配到一个执行进程去处理。
RDD 上定义的函数分两种，一种是转换（transformation）函数，这种函数的返回值还是 RDD；另一种是执行（action）函数，这种函数不再返回 RDD。

RDD 定义了很多转换操作函数(很像rxjava)，比如有计算map(func)、过滤filter(func)、合并数据集union(otherDataset)、根据 Key 聚合reduceByKey(func, [numPartitions])、连接数据集join(otherDataset, [numPartitions])、分组groupByKey([numPartitions]) 等十几个函数。

cooooo 09:28

相对rm的map和reduce，spark一个作业可以支持更多函数的计算，速度更快。spark也有关键的shuffle过程(一个数据集中的多个数据分片需要进行分区传输，写入到另一个数据集的不同分片中)，shuffle会把上一次计算结果按key进行数据分片，不同分片传到不同的下一个计算节点

cooooo 09:30



cooooo 09:36

spark会将一个作业所有函数转化成dag(有向无环图，见上图)，rdd是一个个数据集，如图上，一个rdd包含了多个小块，每个小块代表一个数据分片，相当于rdd是逻辑上的数据集，里面的一个分片对应一个物理位置

cooooo 09:44

spark会将一个作业所有函数转化成dag(有向无环图，见上图)，rdd是一个个数据集，如图上，一个rdd包含了多个小块，每个小块代表一个数据分片，相当于rdd是逻辑上的数据集，里面的一个分片对应一个物理位置。
有些函数不会shuffle，比如上图map，c和d两个rdd数据量都一样。有些需要shuffle，比如上图groupby，rdd a转成一个数据量完全不同的rdd b，a结果要shuffle出去形成rdd b


————— 2019-07-27 —————

cooooo 16:46

Spark 有三个主要特性：RDD 的编程模型更简单，DAG 切分的多阶段计算过程更快速，使用内存存储中间计算结果更高效。这三个特性使得 Spark 相对 Hadoop MapReduce 可以有更快的执行速度，以及更简单的编程实现。

首先，Spark 在 2012 年左右开始流行，那时内存的容量提升和成本降低已经比 MapReduce 出现的十年前强了一个数量级，Spark 优先使用内存的条件已经成熟；其次，使用大数据进行机器学习的需求越来越强烈，不再是早先年那种数据分析的简单计算需求。而机器学习的算法大多需要很多轮迭代，Spark 的 stage 划分相比 Map 和 Reduce 的简单划分，有更加友好的编程体验和更高效的执行效率。
