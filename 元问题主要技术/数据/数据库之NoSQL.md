# 数据库之NoSQL
* [看图轻松理解数据结构与算法系列(NoSQL存储-LSM树): https://mv.mbd.baidu.com/sm7cwm7?f=wf&u=c5e4660b3a7db8a8]
 * lsm树也是大部分nosql的核心结构，可以避免btree在rmdb上的不足，因为btree无法避免逻辑上相近的数据在物理上很远，导致经常放在一起查的数据比较耗时

## nosql分类
    * 键值型：redis、levelDB
    * 文档型：MongoDB、couchDB
    * 搜索引擎：Elasticsearch、splunk、solr
    * 列式：HBase、Cassandra、ClickHouse（由俄罗斯Yandex 公司开源的面向 OLAP 的分布式列式数据库，能够使用 SQL 查询生成实时数据报告。有几倍于 GreenPlum 等引擎的性能优势。在大数据领域没有走 Hadoop 生态，而是采用 Local attached storage 作为存储，这样整个 IO 可能就没有 Hadoop 那一套的局限。）
    * 图形数据库：Neo4j、ArangoDB、Titan

## 列式存储vs行式存储
* 国内：阿里Doris、小米Pegasus、头条abase(貌似未开源)
  * Doris不关注底层存储(基于Berkeley DB)，主要关注分布式方面的改进，关键技术点包括：分区路由(数据一致性)、失效转移(可用性)、集群伸缩(伸缩性)

## Redis
* 数据包控制在1000字节以下，不开启pipeline吞吐量约为6w/s，使用pipeline约为10w/s。1000字节是redis性能的分水岭

## HBase
![HBase](https://github.com/star2478/server-tech-tree/blob/master/img/hbase.png)

* hbase仍然是主从模式，master记录元信息并注册到zk，client从zk获取数据key所在hregion然后去访问。hbase是根据数据的key来实现数据分片的，一个hregion就是一个分片，存储了某个范围key的数据
* 一个hfile是一个hdfs文件，自动做好分片备份，无需hbase自己做
* HBase其它特性
 * HBASE伸缩性：扩容容易，缩容难?hbase单个hregion存入数据量太大会自动拆成多个并自动迁到其他节点，如果数据变少了呢，hbase有考虑数据缩容情况吗，如果节点数缩容是不是也意味着数据要做缩容？实际上一般数据也不会变少，所以一般不会考虑数据的缩容，一般只考虑应用的缩容
 * HBASE扩展性：nosql都一样，可以扩展无数个字段
 * HBASE性能：通过 LSM 树，使数据可以通过连续写磁盘的方式保存数据，提高数据写入性能。lsm树原理见上面链接，lsm不是一种具体数据结构，而是可以使用不同数据结构组成的设计思想，比如c0 tree可以使用avl树来实现，其他级别tree使用b 树来实现，c0 tree在内存，其他级别tree在磁盘。先写log(顺序写磁盘很快)再写c0 tree，这样一来断电后虽然c0 tree会消失，但可以根据log恢复。