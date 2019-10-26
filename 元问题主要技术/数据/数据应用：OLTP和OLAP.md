# 数据应用：OLTP和OLAP

## 数据仓库
### Hive

## BI
* cboard开源
* tableau商业

## 日志数据
* ELK（Elasticsearch、Logstash、Kibana）。log的设计思路和目标？？

## 配置数据
* 代表：Archaius、Spring Cloud Config
* 关键技术

  
## OLAP例子：[淘宝数据魔方技术架构解析（2011）](https://link.juejin.im/?target=http%3A%2F%2Fhistory.programmer.com.cn%2F7578%2F)
  * 数据魔方已下线，转为市场行情专业版
![1.png](https://upload-images.jianshu.io/upload_images/2119886-dd4f7b69c459b428.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  * 其业务属于OLAP，针对数据非实时写入场景，但读支持实时和非实时
  * 计算层：融合了批处理（云梯）和流计算（银河）。主要是离线批处理，但仍然有些数据要求时效性，例如针对搜索词的统计数据，引入了流计算，存入存储层NoSQL中
  * 存储层：融合关系型数据库（MyFOX）和NoSQL（Prom）。数据中间层glider相当于存储层对外的网关，对外屏蔽MyFOX和Prom的差异，提供统一的基于HTTP协议restful格式的数据查询接口
    * MyFOX是一个分布式MySQL集群的查询代理层，使得分区对客户端完全透明。进行了冷热数据分离，热数据使用15k转/m SAS接口磁盘（应该是SSD）存储，冷数据使用7.5k转/m SATA接口磁盘存储，前者成本是后者的三倍
    * Porm弥补多属性场景的查询，基于HBase。主要用在以各种商品属性作为筛选条件查询交易数据，所以用商品属性+属性值作为row-key，用交易列表和交易详情相关字段作为column-family
    * 存储层使用了多级缓存（分别位于glider、MyFOX、Prom不同层）以提高查询效率，并使用缓存所有数据（包括空值）减少缓存穿透
