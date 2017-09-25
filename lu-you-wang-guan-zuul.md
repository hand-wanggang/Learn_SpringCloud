# Spring Cloud的路由网关Zuul

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

* 将所有请求地址为http://localhost:8092/server-cus/\*的请求转发到服务名为service-customer的实例上
* 将所有请求地址为http://localhost:8092/server-pro/\*的请求都转发到名为service-provide的实例上

4、测试结果

请求地址http://localhost:8092/server-pro/info

![](/assets/import-zuul-2.png)

请求地址http://localhost:8092/server-cus/request

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

本示例中，将所有访问地址匹配"/\\w+-\\w+/admin"，都直接返回 “must admin can request!'

















