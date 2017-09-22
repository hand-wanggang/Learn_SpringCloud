# 服务的消费者（rest+roibbon）

部分内容来自于：[http://blog.csdn.net/forezp/article/details/69788938](http://blog.csdn.net/forezp/article/details/69788938)

##### 一、官网文档对Ribbon的介绍

根据spring cloud文档的介绍Ribbon是客户端的负载均衡器。并且提供了大量的http和tcp的行为控制。spring cloud中的FeignClient集成了ribbon。ribbon的核心思想是命名客户端。

##### 二、创建工程service-provide（服务的提供者）

因为需要eureka和ribbon的结合才可以实现负载均衡，所以保留之前建立的工程eureka-server用于注册和发现服务。

1、创建工程service-provide作为服务的提供者

添加依赖：`org.springframework.cloud:spring-cloud-starter-eureka:1.3.4.RELEASE`

2、在程序主类上添加`@EnableEurekaClient`

3、添加配置文件，application-dev1.yml和application-dev2.yml两个文件。（因为要启动两个实例）

application-dev1.yml：

```
server:
  port: 8090
spring:
  application:
    name: service-provide
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

application-dev2.yml：

```
server:
  port: 8091
spring:
  application:
    name: service-provide
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

4、因为是服务提供者，我们写一个接口，提供服务。用于输出当前处理请求的服务器名称和端口

```
@RestController
@RequestMapping("/")
public class ServerInfoController {
    @Value("${server.port}")
    private String port;
    @Value("${spring.application.name}")
    private String serverName;

    @RequestMapping("/info")
    public String info(){
        return "Server Name:"+serverName+"; port: "+port;
    }
}
```

5、注意启动工程时需要提供环境变量

##### 三、创建工程service-customer\(服务的消费者\)

1、创建工程service-customer，并添加如下依赖

```
['org.springframework.cloud:spring-cloud-starter-eureka:1.3.4.RELEASE']
['org.springframework.cloud:spring-cloud-starter-ribbon:1.3.4.RELEASE']
```

2、添加配置文件application.yml

```
requestUrl: http://SERVICE-PROVIDE/info # riboon请求地址，注意此处域名使用的是服务名service-provide
server:
  port: 8092
spring:
  application:
    name: service-customer
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

3、实例化bean, RestTemplate,并添加@LoadBalanced注解，此注解表明使用负载均衡。

在程序主类上添加注解@EnableDiscoveryClient，表示允许该服务发现eureka上注册的服务（或者说是可以拉取服务列表）。

4、依次启动eureka-server、service-provide\(两个实例\)、service-customer。

查看eureka-server页面如下：service-ribbon注册了两个实例![](/assets/import-ribbon-1.png)













