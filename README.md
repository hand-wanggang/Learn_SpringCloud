# Spring Cloud学习笔记

以下内容是最近学习SpringCloud的笔记。以下部分内容来自于[http://blog.csdn.net/forezp/article/details/70148833](http://blog.csdn.net/forezp/article/details/70148833)这篇博客，非常适合入门。本人也从SpringCloud的官方文档中摘取了一部分内容作为补充并做了一些代码实例作为参考。

# Chapter1:服务注册-Eureka

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

# Chapter2:服务的消费者（rest+roibbon/feign）

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

# Chapter3:Hystrix断路器

在微服务的架构中，服务与服务之间可以互相调用。如果在服务间调用时，如果单个服务出现故障就会出现线程阻塞的情况，如果此时大量的请求涌入，会导致web容器的线程资源耗尽，服务不可用的情况。并且这种故障在服务间是会传播的。

##### 一、在Ribbon中使用断路器hystrix

1、因为此处使用涉及到Ribbon所以继续使用第二章中的模块service-customer，在模块中添加依赖：spring-cloud-starter-hystrix

2、service-customer模块的入口类上添加注解@EnableHystrix，表示开启熔断器功能。

3、在使用了ribbon的方法上添加注解

```
       @HystrixCommand(fallbackMethod = "hasError")
```

```
    @RequestMapping("/request")
    @HystrixCommand(fallbackMethod = "hasError")
    public String reques(){
        return restTemplate.getForObject(requestUrl,String.class);
    }

    public String hasError(){
        return "sorry server error!";
    }
```

当熔断器触发的时候，就会调用hasError方法返回"sorry server error!"。

4、编写完成后，我们启动服务，erueka-server和service-provide、server-customer。

确保访问[http://localhost:8099/request可以轻轻成功。](http://localhost:8099/request可以轻轻成功。)

5、将服务service-provide 停止，并调用接口[http://localhost:8099/request，将会出现如下情况。此时由于我们停止的所有的服务提供者service-provide。请求时就会出现异常，此时hystrix替我们调用了fallbackMethod](http://localhost:8099/request，将会出现如下情况。此时由于我们停止的所有的服务提供者service-provide。请求时就会出现异常，此时hystrix替我们调用了fallbackMethod) 的方法。返回内容如下。

![](/assets/import-hystrix-1.png)

##### 二、@HystrixCommand的commandProperties

```
    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "500")
        })
```

如上示例是设置执行的超时时间为500ms。

###### 1、execution前缀的属性设置

* execution.isolation.strategy 设置执行的隔离策略：

  ```
                             -- THREAD 线程间隔离，各线程之间执行不互相干扰

                             --SEMAPHONE： 信号量隔离策略，在同一个线程，并发量受信号量控制
  ```

* execution.isolation.thread.timeoutInMilliseconds 设置执行的超时时间，若执行超过该时间，将会对该服务进行降级处理。

* execution.timeout.enabled 设置是否开启超时时间设置，默认为true

* execution.isolation.thread.interruptOnTimeout 设置当hystrix执行超时时，是否将请求中断。默认为true
* execution.isolation.semaphore.maxConcurrentRequests 当设置隔离策略为信号量时，设置的并发信号量最大数量，当并发量大于该数值，请求将会被直接拒绝。

###### 2、前缀为fallback的配置

* fallback.enabled 用于设置降级服务是否启用

###### 3、前缀为**circuitBreaker** 的配置

* circuitBreaker.enabled 用于设置当请求命令失败时是使用断路器来跟踪其执行情况和熔断请求，默认为true
* circuitBreaker.requestVolumeThreshold 在归档的请求时间内断路器请求的最大次数，如果执行次数到达，并且执行时间还未超时，则短路其开始工作，熔断请求。

###### 4、requestContext配置

* requestCache.enabled 该属性用来配置是否开启请求缓存。
* requestLog.enabledg 用来配置和设置HystrixCommand的执行和事件是否打印日志到HystrixRequestLog，默认值为true

###### 5、threadPool配置

coreSize: 该属性用来设置执行命令线程池的核心线程数量，命令执行的最大并发量 默认10

maxQueueSize:线程池的最大队列大小，当设置为-1时，线程池将使用SynchronousQueue实现队列，否则使用LinkedBlockingQueue实现队列。默认值-1

具体的参数设置参考：[http://www.cnblogs.com/li3807/p/7501427.html](http://www.cnblogs.com/li3807/p/7501427.html)

[https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica\#configuration](https://github.com/Netflix/Hystrix/tree/master/hystrix-contrib/hystrix-javanica#configuration)

# Chapter4:Spring Cloud的路由网关Zuul

Demo:[https://github.com/hand-wanggang/erueka-demo/tree/zuul-demo/service-zuul](https://github.com/hand-wanggang/erueka-demo/tree/zuul-demo/service-zuul)

zuul的功能主要是起到路由转发的作用。在zuul我们可以通过地址匹配的方式将,访问特定地址的请求转发的特定的服务，这种转发不是明确的指向一个服务实例，他是支持负载均衡的。同样我们也可以通过zuul来对用户进行权限校验等。

##### 一、创建工程实例

1、创建模块service-zuul并像器build.gradle文件中添加如下依赖：

```
dependencies {
    compile('org.springframework.cloud:spring-cloud-starter-zuul')
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.springframework.cloud:spring-cloud-starter-eureka')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

2、在模块程序入口的主类上添加@EnableZuulProxy 和@EnableEurekaClient，开启zuul功能和注册到eureka-server

3、添加配置文件application.yml文件如下

```
server:
  port: 8092
spring:
  application:
    name: zuul
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
zuul:
  routes:
    ser-pro:
      path: /server-pro/**          # 如果匹配地址成功则转发到 service-provide 服务
      serviceId: service-provide
    ser-cus:
      path: /server-cus/**          # 如果匹配地址成功则转发到 server-cus 服务
      serviceId: service-customer
```

该配置文件的主要作用：

* 将所有请求地址为[http://localhost:8092/server-cus/\*的请求转发到服务名为service-customer的实例上](http://localhost:8092/server-cus/*的请求转发到服务名为service-customer的实例上)
* 将所有请求地址为[http://localhost:8092/server-pro/\*的请求都转发到名为service-provide的实例上](http://localhost:8092/server-pro/*的请求都转发到名为service-provide的实例上)

4、测试结果

请求地址[http://localhost:8092/server-pro/info](http://localhost:8092/server-pro/info)

![](/assets/import-zuul-2.png)

请求地址[http://localhost:8092/server-cus/request](http://localhost:8092/server-cus/request)

![](/assets/import-zuul-1.png)

##### 二、测试zuul的测试功能

1、在原来代码的基础上添加MyFilter继承自ZuulFilter

```
@Component
public class MyFilter extends ZuulFilter {

    private static final Logger LOG = LoggerFactory.getLogger(MyFilter.class);

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return Boolean.TRUE;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        HttpServletResponse response = ctx.getResponse();
        String requestUrl =  request.getRequestURI();
        String reg ="/\\w+-\\w+/admin";
        if(requestUrl.matches(reg)){
            try {
                PrintWriter writer = response.getWriter();
                writer.print("must admin can request!");
                writer.flush();
                writer.close();
            }catch (Exception ex){
                ex.printStackTrace();
            }
        }
        return null;
    }
}
```

filterType 用于标注过滤器的拦截时机：

* pre：路由之前

* routing：路由之时

* post： 路由之后
* error：发送错误调用
* filterOrder：过滤的顺序
* shouldFilter：这里可以写逻辑判断，是否要过滤，本文true,永远过滤。
* run：过滤器的具体逻辑。可用很复杂，包括查sql，nosql去判断该请求到底有没有权限访问。

filterOrder 用于设置该过滤器在过滤器链中的执行顺序

shouldFilter 用于设置是否需要执行过滤方法run\(\),如果为false则不会执行run（）方法。

run 实际的过滤逻辑

本示例中，将所有访问地址匹配"/\w+-\w+/admin"，都直接返回 “must admin can request!'

