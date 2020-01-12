# Spring Boot

<nav>
<a href="#简介">一、简介</a><br/>
<a href="#核心设计">二、核心设计</a><br/>
<a href="#启动原理">三、启动原理</a><br/>
<a href="#结语">四、结语</a><br/>
</nav>
<br/>

## 简介

`Spring Boot`是Spring家族2014年推出的一款开发框架，也是市场份额最大的Spring框架产品。Spring Boot主要基于Spring Framework/Spring MVC，大部分核心都依赖了Spring Framework/Spring MVC库。

Spring Boot官网是[https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot)，源码托管在[https://github.com/spring-projects/spring-boot](https://github.com/spring-projects/spring-boot)，目前已发展到2.x版本。

## 核心设计

[Spring Boot官网](https://spring.io/projects/spring-boot)中feature一栏，罗列了Spring Boot的核心设计，大部分是对Spring Framework/Spring MVC的封装和改进，通过简化配置和依赖，提供`开箱即用`的开发体验。

* 可创建独立的Spring应用：Spring Boot应用本质上就是Spring应用，最终就是一堆Spring Bean托管在IoC容器中，这个特性没什么可说的，是最低标准
* 内嵌web容器：支持内嵌Tomcat（默认）、Jetty、Undertow，具体讲就是可以将应用打包成一个jar包直接运行，不需要独立安装web容器部署war包再运行
* 提供starter简化依赖：原来开发者使用Spring Framework开发一个功能，要在maven或gradle中加很多依赖，而Spring Boot将常用的依赖集合整合成一个个独立依赖，一般以spring-boot-starter-{name}命名，这样能极大简化依赖配置和开发者工作量，比如spring-boot-starter-web整合了Spring web依赖库和web容器依赖库，开发者只要依赖一个spring-boot-starter-web就能开发web应用。Spring Boot为不同常用功能都提供了对应的starter依赖库，比如web、jdbc、log等。另外，如果一个应用依赖多个starter库，但这些库不同版本可能无法兼容，那开发者可以依赖一个spring-boot-starter-parent，只需为这个parent库指定版本，它可以将其他starter库自动配置至相互兼容的版本，这样开发者就无需自己指定其他starter库版本了
* 尽可能自动配置Spring和第三方库：利用starter和各类技术（部分详见下面的启动原理），推测容器应需bean并完成自动配置
* 提供诸多辅助功能，包括度量、健康监测、外部配置，这些对工业级生产环境应用至关重要
* 无XML配置：XML配置出了名的难维护已经被开发者吐槽多年，Spring借Spring Boot之手，终于可以做到零XML配置。熟悉Spring MVC的开发者都知道，使用Spring MVC要配置几个XML，比如web.xml、springmvc.xml等，但实际上现在Spring MVC已经支持无XML启动，通过继承WebApplicationInitializer，完成DispatcherServlet的指定和初始化等一些原本在XML配置里指定的行为，例子详见[Spring MVC官网](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html)。无论如何，Spring Boot较之前辈们倡导`约定大于配置`原则，不光可以无XML配置，甚至可以不需要任何配置，一切配置项都有默认的约定值，但在实际应用中，我们一般都需要为少量配置项指定值，比如端口号、数据库信息等，这些配置可以放在application.properties里。虽然XML格式表现力更好，但properties的kv格式更简单更友好

## 启动原理
Spring应用的运行期阶段，主要就是运用IoC容器里各种bean，而在启动阶段却有不少学问，所以本文重点讨论Spring Boot的启动原理。下面代码片段是使用Spring Boot的典型入口程序，通过这段代码来看Spring Boot启动过程，一共两步，分别是SpringApplication实例化和run()执行。

```Java
// @SpringBootApplication是一个复合注解，由多个注解组成，主要有@Configuration、@EnableAutoConfiguration、@ComponentScan
@SpringBootApplication
public class HelloWorld {
	public static void main(String[] args) {
    	SpringApplication.run(HelloWorld.class, args);
	}
}
```

SpringApplication实例化原理如下：

* 推测应用类型：在classpath寻找特征类，决定本次应用的类型，是Web应用(SERVLET)、响应式Web应用(REACTIVE)、还是默认非Web应用(default)
* 加载初始化器和监听器：借助SpringFactoriesLoader，查找、实例化(Class.forName)、排序所有初始化器ApplicationContextInitializer和监听器ApplicationListener
> 初始化器和监听器实际都是Spring Framework相关接口，这里用在了Spring Boot启动阶段。为了在Spring Boot启动时完成自动加载，初始化器和监听器的所有实现类都要配置在classpath下的`META-INF/spring.factories`里，不仅如此，配置在该文件里的所有类，都可以在Spring Boot启动时完成自动加载和执行，这是因为在Spring Boot启动时，会在各个环节调用`SpringFactoriesLoader`去读取该文件加载和执行相关类，同时会将读取结果缓存到cache里，避免重复读取文件。开发者也可以自己实现相关接口并配置到spring.factories里，在Spring Boot启动时实现定制化的操作，所以严格意义上来说，能配置到spring.factories里的类都可以看作Spring Boot的扩展点
* 推断主类：找到main()所在类，并赋值到属性mainApplicationClass

run()的工作原理如下：

* 启动时间监控：初始化StopWatch，可用于监控启动阶段每个任务的总耗时
* 启动事件监听：事件监听器SpringApplicationRunListener主要用于run()后面各个子环节中产生各类事件，将事件广播给对应的监听器处理。监听器在SpringApplication实例化时已完成加载和实例化
> SpringApplicationRunListener也配置在META-INF/spring.factories中，也是Spring Boot的一个扩展点
* 设置环境变量，包括application.properties等配置
* 创建IoC容器：根据SpringApplication实例化时的应用类型推测，决定创建什么类型的ApplicationContext容器。ApplicationContext容器的介绍详见[Spring Framework](SpringFramework.md)
* 创建错误分析器，用于处理启动过程中发生的异常
* 执行初始化器：执行所有初始化器的initialize()。初始化器在SpringApplication实例化时已完成加载和实例化
* 初始化IoC容器：实际会执行到Spring Framework中的refresh()，完成IoC容器初始化和bean初始化，详见[Spring Framework](SpringFramework.md)。除此之外，此环节还完成了以下工作：
	* 完成内嵌web容器的端口设置和初始化工作，默认是Tomcat，还可以支持Jetty和Undertow
	* 充分利用`@SpringBootApplication`注解，将许多关键Spring bean自动注册到IoC容器。@SpringBootApplication是Spring Boot最重要的注解，是一个复合注解，主要集成了`@Configuration`、`@ComponentScan`、`@EnableAutoConfiguration`三个运行期注解，前两个是Spring Framework的注解，后一个是Spring Boot的注解。@Configuration将所修饰的类变成配置类，配置类里可以指定任意类为Spring bean并注册到IoC容器。@ComponentScan扫描指定路径并找出所有Spring bean，实例化注册到IoC容器中。@EnableAutoConfiguration借助SpringFactoriesLoader，从spring.factories中找到所有key为org.springframework.boot.autoconfigure.EnableAutoConfiguration的类，通过反射实例化成bean注册到IoC容器中
* 执行CommandLineRunner：这是Spring Boot一个扩展点，如果开发者想在启动快结束时做点什么，可以通过实现CommandLineRunner接口来实现。开发者可以实现多个CommandLineRunner，并能指定相应的Order来进行排序执行
* 关闭事件监听
* 关闭事件监控

## 结语

Spring Boot比Spring Framework/Spring MVC进一步提高了开发效率，而且非常适合微服务开发。但Spring Boot只能开发单个微服务，而现实中应用大多由多个微服务组成，服务间需要大量的相互关联，Spring Boot对此无能为力。于是，[Spring Cloud](SpringCloud.md)应运而生。
