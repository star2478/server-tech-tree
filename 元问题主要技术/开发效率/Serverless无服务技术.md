# Serverless无服务技术

## serverless
* AWS lambda、knative、OpenWhisk、CSE
* 研发效率 & 资源管理能力
* 适用场景：
  * 应用负载有显著的波峰波谷
  * 基于事件的数据处理。数据中台是否可以用serverless来etl数据呢？
当前IaaS付费粒度还是时间维度的，serverless服务可以实现IaaS层的弹性扩容缩容，按按时间或调用次数计费等等。AWS lambda、[阿里CSE](https://yq.aliyun.com/articles/704496?spm=a2c4e.11155472.0.0.58b46ebe2Y17b8)
* CSE可以在存量系统上实现serverless，无需重构。内部调用流程是A调用B，如果B资源不足（线程池满或触发限流阈值）则要求总控系统快速新建实例B'，然后总控系统将B‘地址发给A，A去调用B’。如果是外部调用发现服务资源不足呢？
 * serverless关注服务启动时间，CSE将服务分为3级L1～L3，L1采用预先启动，L2磁盘快照，L3磁盘镜像冷启动
