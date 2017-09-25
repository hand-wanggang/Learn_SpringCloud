# Hystrix断路器

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

5、将服务service-provide 停止，并调用接口http://localhost:8099/request，将会出现如下情况。此时由于我们停止的所有的服务提供者service-provide。请求时就会出现异常，此时hystrix替我们调用了fallbackMethod 的方法。返回内容如下。

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

                                 -- THREAD 线程间隔离，各线程之间执行不互相干扰

                                 --SEMAPHONE： 信号量隔离策略，在同一个线程，并发量受信号量控制

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



















