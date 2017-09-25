# 服务的消费者（rest+roibbon/feign）

部分内容来自于：[http://blog.csdn.net/forezp/article/details/69788938](http://blog.csdn.net/forezp/article/details/69788938)

本章代码示例：[https://github.com/hand-wanggang/erueka-demo/tree/ribbon-demo](https://github.com/hand-wanggang/erueka-demo/tree/ribbon-demo)

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

4、提供接口用于请求，接口内使用restTemplate转发请求，并将结果发回给我们。

```
@RestController
@RequestMapping("/")
public class RibbonRequestController {

    @Value("${requestUrl}")
    private String requestUrl;

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("/request")
    public String reques(){
        return restTemplate.getForObject(requestUrl,String.class);
    }
}
```

5、依次启动eureka-server、service-provide\(两个实例\)、service-customer。

查看eureka-server页面如下：service-ribbon注册了两个实例![](/assets/import-ribbon-1.png)

6、多次请求地址[http://localhost:8092/request](http://localhost:8092/request) 会出现如下内容交替出现的情况。这是因为我们实用restTemplate请求时，使用了ribbon对客户端请求做了负载均衡处理，请求是由两台不同的服务器处理的。

![](/assets/import-ribbon-2.png)

![](/assets/import-ribbon-3.png)

##### 四、使用feign来调用服务

因为feign已经封装的非常易用，所以我们在之前的模块service-customer添加一些代码和依赖即可。

1、在service-customer的build.gradle文件中添加依赖 org.springframework.cloud:spring-cloud-starter-feign:1.3.4.RELEASE，如下所示：

```
dependencies {
	compile(['org.springframework.boot:spring-boot-starter-web'],
			['org.springframework.cloud:spring-cloud-starter-eureka:1.3.4.RELEASE'],
			['org.springframework.cloud:spring-cloud-starter-ribbon:1.3.4.RELEASE'],
			['org.springframework.cloud:spring-cloud-starter-feign:1.3.4.RELEASE']
	)
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

2、在项目的入口主类上添加注解@EnableFeignClients，开启feign调用。

3、编写接口，并用@FeignClient标注，表示这是一个feign调用。如果需要处理调用出现异常或超时的情况，可以实现该接口，并在方法中返回特定的返回值。

```java
@FeignClient(name = "service-provide",fallback = RequestServerInfoClientImpl.class)
public interface RequestServerInfoClient {

	@RequestMapping("/info")
	public String info();
}

```

4、在程序中使用feignClient请求服务，其中的serverInfoClient，就是注入的一个feignClient的实例。

```java
	@RequestMapping("/feign")
	public String feign(){
		return serverInfoClient.info();
	}
```

5、多调用接口，发现feign默认实现了负载均衡

![](/assets/import-feign-1.png)

![](/assets/import-feign-2.png)

6、比较三和四中，两种调用服务的方法：相比之下feign更加易用，对错误的处理也更加便捷并且feign默认实现了断路器。当然我们也可以通过Hystrix组件来实现和feign相同的功能，只是操作上要麻烦一些了。





