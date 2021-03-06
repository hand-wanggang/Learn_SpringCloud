# 服务注册-Eureka

本文代码示例: [https://github.com/hand-wanggang/erueka-demo](https://github.com/hand-wanggang/erueka-demo "文中代码实例")

Spring Clound为开发人员提供了快速构建系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞争、分布锁等。eureka是服务的发现和注册中心，微服务架构的项目的所有模块服务都应在eureka中注册。

在spring cloud官网的介绍中eureka有集群工作模式和单机模式两种。单机模式就是只有一个eureka服务；协同模式就是多个eureka服务相互注册，协同工作，如果服务A在eureka-1中注册发现了，那么eureka-2、eureka-3等erueka服务都会感知到服务A。

##### 一、创建一个eureka-server服务（所有的Demo都是gradle构建）

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

##### 二、创建eureka-client服务

1、创建工程eureka-client

添加如下依赖：

```
'org.springframework.cloud:spring-cloud-starter-eureka:1.3.4.RELEASE'
```

2、在项目主类上添加注解@EnableEurekaClient

3、配置文件application.yml如下

```
server:
  port: 8081
spring:
  application:
    name: eureka-client1

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/   #配置服务注册的地址
```

4、启动服务，去eureka-server页面可以看到服务eureka-client1已经注册

![](/assets/import-2.png)

至此，erueka注册是发现服务的单机模式已经介绍完了。到这里我隐约感觉到eureka像一个经纪人，但是它不姓宋^.^。

##### 三、Eureka的协同工作模式

eureka的协同模式，我们可以再创建几个eureka-server的项目，也可以通过修改配置文件的方式来启动多个服务。下面使用修改配置文件的方式。

1、在项目eureka-server项目下添加两个配置文件application-server1.yml和application-server2.yml

application-server1.yml

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
    service-url:
      defaultZone: http://${eureka.instance.hostname}:8762/eureka/
```

application-server2.yml

```
server:
  port: 8762                     # 配置服务端口
spring:
  application:
    name: eureka-server-2        # 配置服务名称，这是一个服务的唯一标志
eureka:
  instance:
    hostname: localhost
  client:
    service-url:
      defaultZone: http://${eureka.instance.hostname}:8761/eureka/
```

2、项目启动时配置不同的环境变量\(spring.profiles.active=server1或者server2\)，激活不同的配置文件，将启动eureka-server1和eureka-server2和eureka-client。运行成功后访问[http://localhost:8761/和http://localhost:8762/发现，两个注册服务中都发现了服务eureka-client。](http://localhost:8761/和http://localhost:8762/发现，两个注册服务中都发现了服务eureka-client。)

可是，eureka-client明明只向端口为8761的eureka-server注册了，为什么eureka-server2会知道呢？这是因为在协同模式下，各eureka-server之间会交流信息，类似于路由器之间交换路由信息。

![](/assets/import-eureka-1.png)![](/assets/import-eureka-2.png)

##### 四、Spring Cloud文档上的一些细节

1、因为gradle依赖项目没有像maven一样的依赖管理pom，可以通过添加如下插件来实现

```
buildscript {
  dependencies {
    classpath "io.spring.gradle:dependency-management-plugin:0.4.0.RELEASE"
  }
}

apply plugin: "io.spring.dependency-management"

dependencyManagement {
  imports {
    mavenBom 'org.springframework.cloud:spring-cloud-starter-parent:1.0.0.RELEASE'
  }
}
```

2、eureka在协作模式下，各eureka-server之间要确保至少有一条路线是可达的。

3、在eureka上注册的服务，需要定时的发送存活心跳，来维持其在eureka上的身份。

4、默认情况下，注册到eureka的服务都显示其服务名称，如之前的eureka-client1,就是服务名称，如果要显示其ip地址的话，可以在eureka-server的配置文件中设置eureka.instance.preferIpAddress=true（IP优先）。























