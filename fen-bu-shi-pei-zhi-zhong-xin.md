# Spring Cloud的分布式配置中心

       分布式配置中心，为微服务的所有服务提供属性配置属性。将配置属性集中管理。配置中心的属性文件可以存放在服务本地也可以存储在git上。配置服务中心同其他服务相同也可以集群化。

##### 一、创建一个配置中心服务 config-server

1、创建工程config-server,并添加如下依赖

```
dependencies {
	compile('org.springframework.cloud:spring-cloud-config-server')
	compile('org.springframework.cloud:spring-cloud-starter-eureka')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

2、添加配置文件application.yml如下

```
server:
  port: 8888
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/hand-wanggang/CloudConfig # git 仓库地址
          #password:  #git 仓库密码
          #username:  #git 密码
          search-paths: '{profile}' # 查找路径
        bootstrap: true
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

3、在程序入口类上添加注解@EnableConfigServer和@EnableEurekaClients

注意：http请求地址和映射资源关系如下

* /{application}/{profile}\[/{label}\]

* /{application}-{profile}.yml
* /{label}/{application}-{profile}.yml
* /{application}-{profile}.properties
* /{label}/{application}-{profile}.properties

##### 二 、创建config-client项目

1、创建项目config-client,并添加如下依赖

```
dependencies {
	compile('org.springframework.cloud:spring-cloud-starter-config')
	compile('org.springframework.cloud:spring-cloud-starter-eureka')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

2、因为使用的是配置中心进行配置，所以属性文件名称修改为bootstrap.yml

```
server:
  port: 8889
spring:
  application:
    name: config-client
  cloud:
    config:
      label: master  # 指明远程仓库分支
      profile: dev   # 指明spring.profile
      discovery:
        enabled: true
        service-id: config-server # 配置服务中心的注册服务名称
      name: config-client
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

3、在入口程序的类上添加注解@EnableEurekaClient

4、编写代码将从配置中心获取到的属性，打印出来

```
@RestController
@RequestMapping("/")
public class GetConfigPropertiesController {

	@Value("${foo.version}")
	private String version;

	@RequestMapping("/version")
	public String version(){
		return version;
	}
}
```

5、访问地址[http://laptop-hqmj7ho2:8889/version](http://laptop-hqmj7ho2:8889/version)

![](/assets/import-config-client-1.png)

##### 三、配置服务中心的高可用

同之前服务相同，spring clod的中的服务似乎都是可以高可用的。下面我们来配置一个高可用的服务中心。因为上面的config-client的例子中，我们没有明确指定配置中心的地址，而是使用配置服务中心的注册名，这位下面的例子做了铺垫。现在我们只需要对confg-server稍作修改即可。





