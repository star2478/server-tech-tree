# Spring Boot
SpringApplication:run()
由多个start组成
Springboot在Framework上做了什么封装？？
 1：创建独立的spring应用：使用了@SpringBootApplication初始化了spring容器（ioc容器）
 2：嵌入Tomcat, Jetty Undertow 而且不需要部署他们。
 3：提供的“starters” poms来简化Maven配置
 4：尽可能自动配置spring应用。
 5：提供生产指标,健壮检查和外部化配置
 6：绝对没有代码生成和XML配置要求