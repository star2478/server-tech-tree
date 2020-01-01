# Spring Cloud

`Spring Cloud`于2015年推出，是Spring真正能完成微服务应用开发的框架产品，其内核是Spring Boot。Spring Cloud框架由很多子项目组成，这些子项目的贡献者除了Spring还有其他机构。子项目的概念是从官网直译过来的，可能不易理解，实际上可以等同于组件或依赖库，开发一个微服务应用，可能要依赖多个不同子项目/组件/依赖库。

Spring Cloud官网是[https://spring.io/projects/spring-cloud](https://spring.io/projects/spring-cloud)，中文网是[https://www.springcloud.cc](https://www.springcloud.cc)，源码托管在[https://github.com/spring-projects/spring-cloud](https://github.com/spring-projects/spring-cloud)，目前发展到Hoxton版本，对应的Spring Boot版本是2.2。

> Spring Cloud版本号以伦敦地铁命名，而非惯用的数字

绝大部分的Spring Cloud模块，本质上还是一个Spring Boot/Spring Framework模块，不过一个微服务应用除了一般的应用模块外，还包括了一些特殊职能的模块，比如注册中心、网关等，这些模块可能依赖了非Spring机构的子项目/组件/依赖库，所以这些模块很可能不是Spring风格的。下面代码片段是Spring Cloud的典型入口程序，可以看到与Spring Boot唯一区别是增加了一个`@EnableDiscoveryClient`注解（该注解的作用是将本模块注册到注册中心），这样看来，Spring Cloud启动也没什么好说的。对于Spring Cloud，本文侧重于介绍其名下一些关键的子项目，比如注册中心、网关等，而这些子项目的主业大多不在开发效率元问题上，而是在可用性等其他元问题上。

@SpringBootApplication
@EnableDiscoveryClient
```Java

@SpringBootApplication
@EnableDiscoveryClient
public class HelloWorld {
	public static void main(String[] args) {
    	SpringApplication.run(HelloWorld.class, args);
	}
}
```

## 核心设计

要打造一个微服务应用，除了开发一个个模块外，还要考虑模块间通讯、路由、可用性等等，这正是Spring Cloud的核心设计所在。[Spring Cloud官网](https://spring.io/projects/spring-cloud)中feature一栏，罗列了Spring Cloud的核心设计，每条设计的背后都有一个或多个子项目提供的解决方案，这些子项目在本文其他元问题上进行分析介绍。

* 分布式/版本化配置：对应的主要子项目包括Spring Cloud Config
* 服务注册与发现：对应的主要子项目包括Spring Cloud Netflix Eureka、Spring Cloud Zookeeper
* 路由：对应的主要子项目包括Spring Cloud Gateway、Spring Cloud Netflix Zuul
* 服务间通讯：对应的主要子项目包括Spring Cloud OpenFeign
* 负载均衡：对应的主要子项目包括Spring Cloud OpenFeign、Spring Cloud Netflix Ribbon
* 熔断器：对应的主要子项目包括Spring Cloud Netflix Hystrix
* 全局锁：对应的主要子项目包括Spring Cloud Cluster，借助Zookeeper、Redis实现
* 领导选举和集群状态：对应的主要子项目包括Spring Cloud Cluster，借助Zookeeper、Hazelcast、Etcd实现
* 分布式消息传递：对应的主要子项目包括Spring Cloud Bus