# 服务注册-Eureka

Spring Clound为开发人员提供了快速构建系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞争、分布锁等。eureka是服务的发现和注册中心，微服务架构的项目的所有模块服务都应在eureka中注册。

在spring cloud官网的介绍中eureka有集群工作模式和单机模式两种。单机模式就是只有一个eureka服务；协同模式就是多个eureka服务相互注册，协同工作，如果服务A在eureka-1中注册发现了，那么eureka-2、eureka-3等erueka服务都会感知到服务A。

##### 一、创建一个eureka-server服务和一个eureka-client服务（所有的Demo都是gradle构建）

1、创建eureka-server工程

添加如下依赖：

```
'org.springframework.cloud:spring-cloud-starter-eureka-server:1.3.4.RELEASE'
```

2、在项目主类，@SpringBootApplication标注的类上添加注解@EnableEurekaServer注解，表示这是一个注册发现服务

3、配置文件application.yml文件添加如下配置

```
server:
  port: 8761                     # 配置服务端口
spring:
  application:
    name: eureka-server-1        # 配置服务名称，这是一个服务的唯一标志
eureka:
  instance:
    hostname: localhost
  client:
    fetch-registry: false         # 默认为true，如果是单机模式配置为false
    register-with-eureka: false   # 默认为true 如果为单机模式配置为false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

4、启动项目看到如下页面，表示已经成功![](/assets/import-1.png)

