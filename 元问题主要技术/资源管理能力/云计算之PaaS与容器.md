# 云计算之PaaS与容器
将容器技术划到PaaS并不十分准确，实际上，容器技术给IaaS与PaaS搭了一个桥，使两者能够相互打通？？

## 容器
### Docker
包括了伸缩性、可用性
* docker核心能力：打包(将应用及其依赖的整个OS文件系统打包，应用在容器内与本地真正做到一致，开发人员无需处理任何环境迁移问题

## 容器编排
### Kubernetes
包括了伸缩性、可用性
* k8s资源管理能力
  * 资源分类：分为可压缩和不可压缩
  * 资源使用设置和限制：limit和request区别，设置每个容器的资源限制
  * 资源优先级：用以在资源紧张时谁被优先回收，QoS和pod里的等级将资源分了等级
  * 资源使用的读取：通过cgroup
* k8s默认调度器
  * queue阶段：每个资源请求pod将排队到一个统一queue里，调度器循环并发取出pod
  * predicate阶段：从schedule cache里取出所有可用node信息
  * priority阶段：给node打分，最适合pod的node会被打最高分
  * bind阶段：pod被部署到最高分node里启动运行