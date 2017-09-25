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

确保访问http://localhost:8099/request可以轻轻成功。

5、将服务service-provide 停止，并调用接口http://localhost:8099/request，将会出现如下情况。



