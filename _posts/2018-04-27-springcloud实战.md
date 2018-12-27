---
layout:     post
title:      spring-cloud实战
subtitle:   spring-cloud全家桶学习
date:       2018-12-27
author:     CDz
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - spring-cloud
    - spring
---

应为最近spring-cloud比较火,所以系统的了解一下spring-cloud系统框架,因为学习时,cloud2.0还没有正式发布,所以有很多的坑在其中,解决他们的过程中,也是了解整体框架思路的好方法.

# springcloud_sell

eureka:服务注册中心

# 客户端负载均衡:
## ribbon:
### 内部机制
- 服务发现
- 服务选择规则
- 服务监听(检查所有服务,做到高效剔除)
### 内部组件
- ServerList
- IRule
规则的使用,通过源码可以发现 若不设置使用的是RoundRobinRule(轮询),也可以自定义规则或者切换其他规则
通过yml参数设置

- ServerListFilter
步骤:

1.通过ServerList获取所有的服务列表

2.通过ServerListFilter过滤部分地址

3.通过IRule定义的规则命中一台地址

### 使用到的组件:
- RestTemplate:spring提供的restFul请求接口(用到LoadBalancer(ribbon)来实现客户端的 软负载均衡)
- Feign(本质http客户端)
    - 声明式REST客户端，伪RPC
    - 采用基于`接口加注解`的形式
- Zuul

# spring cloud config
    配置中心：使用的目的是为了更容易管理配置文件，微服务架构中项目分散，配置往往比较难于管理，如果还是传统项目一样将配置文件写在项目中，会有很大的管理成本。
    
主要作用：
   - 同一所有项目，管理配置文件。
   - 做到修改配置文件不需要重启项目
## server端
新建config项目加入依赖：
- `spring-cloud-starter-netflix-eureka-client`同样也是一个客户端
- `spring-cloud-config-server`config server依赖
- `spring-cloud-starter-bus-amqp`消息总线（用于修改文件项目实时更新）

### 添加配置
```
// eureka 客户端的相关配置 就不写在这里了 ...
  cloud:
    config:
      server:
        git:
          uri: https://gitlab.com/your-Repository-url //配置文件的git仓库地址
          username: @username //登陆用户名
          password: @password //密码
          basedir: E:\Workspaces\springcloud_sell\config\basedir //本地快照存储目录
          //（这里configserver 会在链接上git后从上下载一份一样的在本地，当git链接不上时使用本地配置文件）
 ```
 ### 注意
 启动后就成为一个config客户端了，通过 `http://localhost:8082/order-dev.yml`直接访问配置文件
 如果直接访问``是会报404 看启动日子可知
 ```
 {[/{name}-{profiles}.properties],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.properties(java.lang.String,java.lang.String,boolean) throws java.io.IOException
 "{[/{name}-{profiles}.yml || /{name}-{profiles}.yaml],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.yaml(java.lang.String,java.lang.String,boolean) throws java.lang.Exception
 "{[/{name}/{profiles:.*[^-].*}],methods=[GET]}" onto public org.springframework.cloud.config.environment.Environment org.springframework.cloud.config.server.environment.EnvironmentController.defaultLabel(java.lang.String,java.lang.String)
 "{[/{label}/{name}-{profiles}.properties],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.labelledProperties(java.lang.String,java.lang.String,java.lang.String,boolean) throws java.io.IOException
 "{[/{label}/{name}-{profiles}.json],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.labelledJsonProperties(java.lang.String,java.lang.String,java.lang.String,boolean) throws java.lang.Exception
 "{[/{name}/{profiles}/{label:.*}],methods=[GET]}" onto public org.springframework.cloud.config.environment.Environment org.springframework.cloud.config.server.environment.EnvironmentController.labelled(java.lang.String,java.lang.String,java.lang.String)
 "{[/{name}-{profiles}.json],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.jsonProperties(java.lang.String,java.lang.String,boolean) throws java.lang.Exception
 "{[/{label}/{name}-{profiles}.yml
 
 //截取部分 还有很多形式
 ```
 访问格式必须要是其指定的格式：
 
 `{name}-{profiles}.yml`中name：配置文件的名字，profiles：环境。
 
 `{label}/{name}-{profiles}.yml`,label:指定git的分支
 
 **值得注意的是**:在用浏览器访问时，修改后缀名比如我们在git上是yml格式，访问后缀该为properties，会将yml自动转换为properties格式。
 
 ## config客户端
 引入依赖：`spring-cloud-config-client`,不需要在启动类上加入注解。
 
 ### 使用
 修改yml配置文件
```
//这里顺序是，1，服务先注册到 eureka上，
//2，然后在去config服务
//3，这里发现没有设置访问那个文件 其根据application.name来查找配置文件
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
spring:
  cloud:
    config:
      discovery:
        enabled: true
        service-id: CONFIG
      profile: dev
  application:
    name: order
```

###　注意
这里有很多人发现eureka client设置去掉一样也可以加载，但是我也是非常的纳闷，他是怎么知道config项目的，因为config项目跟此项目是没有关联的，如果没有注册到eureka，是不可能找到的。

后来发现每个eureka客户端启动时，如果没有配置注册中心地址，那么就会使用默认`http://localhost:8761/eureka/`.

## 消息总线（spring cloud bus）
上面做到了，配置的集中化管理，但是其中如果想要实现，配置的实时更新还是不行的。

# 服务网关
为什么要使用服务网关：如果有多个服务，那么客户端如何去调用这些服务？一个一个服务去打交道，这是一个非常不现实的。

需要一个角色来充当request的同一入口，zuul在spring cloud 中就充当这一角色。

## 服务网管的要素
- 稳定性,高可用
- 性能,并发性
- 安全性
- 扩展性

## 常用的网关方案
- Nginx+Lua
- Kong(商业软件)
- Tyk(开源go语言发开)
- spring cloud zuul（天生适合微服务，主要Java微服务使用spring cloud 作为网关的不二选择，但是性能上没有nginx强大）
    - 一般解决方案 Nginx+zuul，使用Nginx作为第一层的负载均衡到各个zuul网关服务上，通过zuul的发挥自身优势

### spring cloud zuul
zuul特点 
- **路由+过滤器=zuul**
- 核心是一系列的过滤器

#### zuul的四种过滤器API
- 前置（pre）
- 路由（route）
- 后置（post）
- 错误（error）

其中每一类的过滤器是不会通信的，通过RequestContext（请求上下文）做到数据的传递。

yml设置：
```yaml
zuul:
  routes:
    myProduct:
      serviceId: product
      path: /myProduct/**
#     cookie的传递 将敏感头设置为null
      sensitiveHeaders:
#    简单配置 自定义访问url
#    product: /myProduct/**
#设置不允许一些接口暴露外部，只服务间调用，参数是set集合
#此为 正则方式匹配
  ignored-patterns: 
    - **/product
```
这里发现一个坑，其网关默认请求接口过期时间是1s，只要一个接口超过1秒，就会直接返回错误。


而其底层使用的是，ribbon来实现的HTTP restFul请求。所以问题在于ribbon当一个接口时间超过1s就会报错。

#### zuul过滤器编写
需要继承`ZuulFilter`,实现其中方法
```java
@Component
public class TokenFilter extends ZuulFilter {
    @Override
    public String filterType() {
        //常量在 FilterConstants 下寻找
        return POST_TYPE;
    }

    /**
    * 过滤器执行循序
    * 值越小 优先级越高
    * 
    * 在FilterConstants有定义顺序常量.
    */
    @Override
    public int filterOrder() {
        //常量在 FilterConstants 下寻找
        return SERVLET_DETECTION_FILTER_ORDER - 1;
    }

    /**
    *
    *是否使用常量
    */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
    *   具体
    *
    */   
    @Override
    public Object run() throws ZuulException {


        RequestContext currentContext = RequestContext.getCurrentContext();

        HttpServletRequest request = currentContext.getRequest();

        String token = request.getParameter("token");
        if (StringUtils.isEmpty(token)){
            currentContext.setSendZuulResponse(false);
            currentContext.setResponseStatusCode(HttpStatus.UNAUTHORIZED.value());
        }
        return null;
    }
}
```

在过滤器中可以实现,鉴权与限流的操作.

# 熔断器(spring cloud hystrix)
- 服务降级
    - 优先核心服务,非核心服务不可用或弱可用
    - 通过 HystrixCommand注解指定
    - fallbackMethod(回退函数)中具体实现降级逻辑
- 服务熔断
- 依赖隔离
    - hystrix会为每一个 @HystrixCommand 注解创建一个线程池，此服务会与其他服务实现线程隔离，就算此服务延迟过高情况下，不会影响到其他服务的调用。依赖隔离与服务降级是一体化实现的。
- 监控(hystrix dashboard)

## 使用
加入依赖
```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
``` 
在启动类上加入注解`@EnableCircuitBreaker`

其实也可以直接使用 `@SpringCloudApplication`相当于
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootApplication //springboot启动类注解
@EnableDiscoveryClient//eureka客户端注解
@EnableCircuitBreaker//hystrix启用注解
public @interface SpringCloudApplication {
}
```

### 服务熔断

在调用接口方法上加入`@HystrixCommand`其参数很多,如果想实现降级加入参数`fallbackMethod = "fallback"`
```java
    @RestController
    public class HystixController {
    
        /**
         * 使用 RestTemplate 实现调用服务接口
         * @return
         */
    
        @GetMapping("/getProductInfoList")
        @HystrixCommand(fallbackMethod = "fallback")
        public String getProductInfoList(){
            RestTemplate restTemplate = new RestTemplate();
            return restTemplate.getForObject("http://localhost:8080/product", String.class);
        }
    
        private String fallback(){
            return "太拥挤了,请稍后再试.....";
        }
    }
```

从开始实验中发现其实是因为,restTemplate调用请求时,抛出了异常才使得其进入`fallback`降级方法.

故测试,是否方法中,直接抛出异常是否会返回`太拥挤了,请稍后再试.....`,事实是可以的,**说明其实hytsrix不只可以降级服务.同样也可以降级自己的服务**

若想要将一个类中所有的服务,统一处理降级逻辑在类上
全部参数解释:
```java
        public @interface HystrixCommand {

            // HystrixCommand 命令所属的组的名称：默认注解方法类的名称
            String groupKey() default "";

            // HystrixCommand 命令的key值，默认值为注解方法的名称
            String commandKey() default "";

            // 线程池名称，默认定义为groupKey
            String threadPoolKey() default "";
            // 定义回退方法的名称, 此方法必须和hystrix的执行方法在相同类中
            String fallbackMethod() default "";
            // 配置hystrix命令的参数
            HystrixProperty[] commandProperties() default {};
            // 配置hystrix依赖的线程池的参数
             HystrixProperty[] threadPoolProperties() default {};

            // 如果hystrix方法抛出的异常包括RUNTIME_EXCEPTION，则会被封装HystrixRuntimeException异常。我们也可以通过此方法定义哪些需要忽略的异常
            Class<? extends Throwable>[] ignoreExceptions() default {};

            // 定义执行hystrix observable的命令的模式，类型详细见ObservableExecutionMode
            ObservableExecutionMode observableExecutionMode() default ObservableExecutionMode.EAGER;

            // 如果hystrix方法抛出的异常包括RUNTIME_EXCEPTION，则会被封装HystrixRuntimeException异常。此方法定义需要抛出的异常
            HystrixException[] raiseHystrixExceptions() default {};

            // 定义回调方法：但是defaultFallback不能传入参数，返回参数和hystrix的命令兼容
            String defaultFallback() default "";
        }
```

在之前`zuul`中遇到过的超时时间问题,在这里hystrix中同样也会有,情景是当调用接口超时时间太久同样会将其降级处理.

设置超时时间方法,使用上述参数`commandProperties`,`commandProperties`的参数是一个list集合,集合中`@HystrixProperty`,参数所有的定义在
`com.netflix.hystrix.HystrixCommandProperties`

- HystrixCommandProperties涉及到超时的参数
    - default_executionTimeoutInMilliseconds  -> execution.isolation.thread.timeoutInMilliseconds

- HystrixCommandProperties涉及熔断配置
    - circuitBreakerRequestVolumeThreshold -> circuitBreaker.enabled
        - 滚动窗口中,断路器最小请求数
    - circuitBreakerSleepWindowInMilliseconds -> circuitBreaker.requestVolumeThreshold
        - 时间窗口,当触发熔断后,会有一个记时窗口,这里的值就是计时的时间,熔断后fallback会变成主逻辑,当休眠窗口到达设置的时间后,熔断到半开(half-open)状态,此时允许请求服务,如果请求成功,熔断结束.如果失败继续休眠,且重新及时.
    - circuitBreakerEnabled -> circuitBreaker.sleepWindowInMilliseconds
        - 是否开启熔断设置
    - circuitBreakerErrorThresholdPercentage -> circuitBreaker.errorThresholdPercentage
        - 断路器打开条件,是百分数.如设置为60,circuitBreakerRequestVolumeThreshold设置为10,那么就是当在滚动窗口中发生10次调用,10次中有7次发生异常,70%>60%就开启熔断

事例：

注解配置：
```java
//设置超时时间
    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")//设置接口过期时间
//            @HystrixProperty(name = "",value = "")
    })
```

yml文件配置：
```yaml
hystrix:
  command:
  #注意这里是用的default，意思是全局配置。
  #在注解参数中发现有commandKey参数，此默认为方法名，也可以自定义，故可将default换成commandKey值，就可以细化控制具体服务
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 2000
```

```java
//设置断路器参数
    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"),
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),//休眠时间窗口单位 毫秒
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60")//断路器打开的错误百分比条件
    })
```

#### feign与hystrix整合
第一步：

需要在yml中配置：注意这里是没有提示的
```
feingn:
  hystrix:
    enabled: true
```

第二步：


然后在client的`@FeignClient`注解上添加参数`@FeignClient(name = "product",fallback = ProductClient.ProductClientFallback.class)`

且在`ProductClientFallback`类上加入`@Component`使其加载到Spring中。
```

@FeignClient(name = "product",fallback = ProductClient.ProductClientFallback.class)
public interface ProductClient {

    @PostMapping("/product/productIds")
    List<ProductInfo> findByProductIdList(@RequestBody List<String> productIds);

    @PostMapping("/product/decreaseStock")
    void decreaseStock(@RequestBody List<OrderFrom.OrderItem> orderItems);

    @Component
    static class ProductClientFallback implements ProductClient{

        @Override
        public List<ProductInfo> findByProductIdList(List<String> productIds) {
            return null;
        }

        @Override
        public void decreaseStock(List<OrderFrom.OrderItem> orderItems) {

        }
    }
}
```

第三步：

在调用服务方的启动注解上加入扫描`@ComponentScan(basePackages = "com.cdz")`扩大范围，使其可以扫描到ProductClientFallback这个类。
```
@EnableFeignClients(basePackages = "com.cdz.productclient")//使用feign组件调用其它客户端接口
@ComponentScan(basePackages = "com.cdz")
@EntityScan(value = "com.cdz.common.domain")//springdata jpa bean的扫描路
//@SpringBootApplication
//@EnableDiscoveryClient//申明是eureka的客户端
//@EnableCircuitBreaker
@SpringCloudApplication
public class OrderServiceApplication {

  public static void main(String[] args) {
    SpringApplication.run(OrderServiceApplication.class, args);
  }
}

```

###  hystrix-dashboard 断路器可视化
加入依赖：
```
      <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
<!--
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-netflix-archaius</artifactId>
        </dependency>-->
```

在启动类上加入注解：`@EnableHystrixDashboard`

在页面上输入项目 `host:port/hystrix`。进入后并不能找到，原因是Springcloud2.0的一个坑，其访问的是` http://localhost:8081/hystrix.stream?delay=2000`但是Springcloud2.0改为了`application/hystrix.stream`,所以要加入yml配置。

```
management:
  endpoints:
    web:
      exposure:
      # 展示所有可访问url
        include: "*"
      # 所有url以/开始 （去掉application）
      base-path: /
```
这样就可以进入看到loading等待了。当访问时页面上会展示出服务的熔断情况。

# 链路追踪 Spring Cloud Sleuth
引入依赖：
```
  <!--联络追踪依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
```

```
product:
INFO [product,f7bdceab5c12c9c1,51ee183bda33f82d,false] 

order:
INFO [order,f7bdceab5c12c9c1,51ee183bda33f82d,false] false表示不发送给外部展示
INFO [order,f7bdceab5c12c9c1,51ee183bda33f82d,false]
INFO [order,f7bdceab5c12c9c1,51ee183bda33f82d,false]
INFO [order,f7bdceab5c12c9c1,51ee183bda33f82d,false]
INFO [order,f7bdceab5c12c9c1,51ee183bda33f82d,false]
DEBUG [order,f7bdceab5c12c9c1,f7bdceab5c12c9c1,false]
```

## 可视化联络追踪zipkin
引入依赖：
```
<!--联络追踪依赖-->
        <!--<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-sleuth-zipkin</artifactId>
        </dependency>-->

        <!--包含 sleuth zipkin-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
```

安装zipkin客户端：
```
docker run -d -p 9411:9411 openzipkin/zipkin
```

设置配置：
```
spring:
  zipkin:
  # 发送到的zipkin的地址
    base-url: http://localhost:9411/
  sleuth:
    sampler:
    #抽样的百分比（默认为0.1）在线上时一般不会讲所有的链路都发送到zipkin这样对于流量效率是极大的损害
    #开发调试过程中设置为1 ，可以方便调试。
      probability: 1
```

