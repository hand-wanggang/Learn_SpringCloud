# 消息总线 Spring Cloud Bus

Spring Cloud的消息总线是基于消息代理实现了，如rabbitMQ。Spring Cloud将分布式的服务都挂载在消息总线上，从而达到相互联系和控制的目的。

下面我们基于Spring Cloud Bus 来实现服务的配置属性实时刷新。

##### 一、基于config-client

1、向config-client 工程中添加如下依赖

```
dependencies {
    compile('org.springframework.cloud:spring-cloud-starter-config')
    compile('org.springframework.cloud:spring-cloud-starter-eureka')
    compile('org.springframework.cloud:spring-cloud-starter-bus-amqp')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

2、在需要刷新属性的类上标注@RefreshScope注解

3、调用接口[http://localhost:8889/bus/refresh](http://localhost:8889/bus/refresh)   \(POST方式请求\)，如果请求成功即可实时刷新属性。

4、上面的那种方式会刷新所有服务，可以通过destination参数指定刷新的服务。

如http://localhost:8889/bus/refresh?destination=customers:9000



