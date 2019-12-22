# Spring Boot
`Spring Boot`是Spring家族2014年推出的一款开发框架，也是市场份额最大的Spring框架产品。Spring Boot主要基于Spring Framework/Spring MVC，大部分核心都依赖了Spring Framework/Spring MVC库。

Spring Boot官网是[https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot)，源码托管在[https://github.com/spring-projects/spring-boot](https://github.com/spring-projects/spring-boot)，目前已发展到2.x版本。

## 核心设计

[Spring Boot官网](https://spring.io/projects/spring-boot)中feature一栏，罗列了Spring Boot的核心设计，大部分是Spring Boot对Spring Framework/Spring MVC的封装和改进。

* 可创建独立的Spring应用：这个特性没什么可说的，是最低标准，Spring Framework都可以做到
* 内嵌web容器：支持内嵌Tomcat（默认）、Jetty、Undertow，具体讲就是可以将应用打包成一个jar包直接运行，不需要独立安装web容器部署war包再运行
* 提供starter简化依赖：原来开发者使用Spring Framework开发一个功能，要在maven或gradle中加很多依赖，而Spring Boot将常用的依赖集合整合成一个个独立依赖，一般以spring-boot-starter-{name}命名，这样能极大简化依赖配置和开发者工作量，比如spring-boot-starter-web整合了Spring web依赖库和web容器依赖库，开发者只要依赖一个spring-boot-starter-web就能开发web应用。Spring Boot为不同常用功能都提供了对应的starter依赖库，比如web、jdbc、log等。另外，如果一个应用依赖多个starter库，但这些库不同版本可能无法兼容，那开发者可以依赖一个spring-boot-starter-parent，只需为这个parent库指定版本，它可以将其他starter库自动配置至相互兼容的版本，这样开发者就无需自己指定其他starter库版本了
* 尽可能自动配置Spring和第三方库
* 提供诸多辅助功能，包括度量、健康监测、外部配置，这些对工业级生产环境应用至关重要
* 无XML配置：XML配置出了名的难维护已经被开发者吐槽多年，Spring借Spring Boot之手，终于可以做到零XML配置。熟悉Spring MVC的开发者都知道，使用Spring MVC要配置几个XML，比如web.xml、springmvc.xml等，但实际上现在Spring MVC已经支持无XML启动，通过继承WebApplicationInitializer，完成DispatcherServlet的指定和初始化等一些原本在XML配置里指定的行为，例子详见[官网](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html)。无论如何，Spring Boot较之前辈们倡导`约定大于配置`原则，不光可以无XML配置，甚至可以不需要任何配置，一切配置项都有默认的约定值，但在实际应用中，我们一般都需要为少量配置项指定值，比如端口号、数据库信息等，这些配置可以放在application.properties里。虽然XML格式表现力更好，但properties的kv格式更简单更友好

## 启动原理
下面代码片段是使用Spring Boot的典型入口程序，通过这段代码来看Spring Boot启动过程。

```Java
@SpringBootApplication
public class HelloWorld {
	public static void main(String[] args) {
    	SpringApplication.run(HelloWorld.class, args);
	}
}
```

run()依赖Tomcat包，Tomcat实现了Servlet规范，run启动时会初始化一个Tomcat对象（设置端口号、监听web servlet请求）。