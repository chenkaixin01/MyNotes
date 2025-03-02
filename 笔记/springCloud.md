SpringCloud是一个微服务实现框架，它提供了诸多组件帮助程序员搭建一个微服务系统；其提供的组件主要有以下

> **服务发现注册：** Eureka，Nacos
> 
> **负载均衡：** ribbon，openfeign
> 
> **容错处理：** Hystrix
> 
> **服务网关：** springCloudGatway
> 
> **配置管理：**  Nacos

# 1 服务间请求流程

1. 微服务在启动将自己的服务注册到Nacos注册中心，同时发布http接口供其他系统调用，一般都是基于SpringMVC。

2. 服务消费者基于Feign调用服务提供者对外发布的接口，先对调用的本地接口加上注解@FeignClient，Feign会针对加了该注解的接口生成动态代理，服务消费者针对Feign生成的动态代理去调用方法时，会在底层生成Http协议格式的请求，类似 /stock/deduct? productId=100。

3. Feign最终会调用Ribbon从本地的Nacos注册表的缓存里根据服务名取出服务提供在机器的列表，然后进行负载均衡并选择一台机器出来，对选出来的机器IP和端口拼接之前生成的url请求，生成调用的Http接口地址。

# 2 Nacos

![](https://developer.qcloudimg.com/http-save/yehe-4283147/e025db912caf85d55375b2cb0a968db5.png)

## 2.1 功能点

**服务注册**：Nacos Client会通过发送REST请求的方式向Nacos Server注册自己的服务，提供自身的元数据，比如ip地址、端口等信息。Nacos Server接收到注册请求后，就会把这些元数据信息存储在一个双层的内存Map中。

**服务心跳**：在服务注册后，Nacos Client会维护一个定时心跳来持续通知Nacos Server，说明服务一直处于可用状态，防止被剔除。默认5s发送一次心跳。

**服务健康检查**：Nacos Server会开启一个定时任务用来检查注册服务实例的健康情况，对于超过15s没有收到客户端心跳的实例会将它 的healthy属性置为false(客户端服务发现时不会发现)，如果某个实例超过30秒没有收到心跳，直接剔除该实例(被剔除的实例如果恢复 发送心跳则会重新注册)

**服务发现**：服务消费者（Nacos Client）在调用服务提供者的服务时，会发送一个REST请求给Nacos Server，获取上面注册的服务清 单，并且缓存在Nacos Client本地，同时会在Nacos Client本地开启一个定时任务定时拉取服务端最新的注册表信息更新到本地缓存

**服务同步**：Nacos Server集群之间会互相同步服务实例，用来保证服务信息的一致性。

# 3 openFeign

OpenFeign是Spring官方开发的组件，是对feign的替代；相较于feign，openfeign增加了对SpringMVC的支持，如RequestMapping等；还整合了Nacos等服务注册组件，可以自动利用服务名进行负载均衡

OpenFeign通过动态代理在本地实例化远程接口->封装Request对象并进行编码->发送请求并对获取结果进行解码

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a48747e8d04d4d7fa69692f875d132ce~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

# 4 ribbon

Ribbon是Netflix发布的一个客户端负载均衡工具，它基于HTTP和TCP协议，主要用于微服务架构中进行服务发现和负载均衡。Ribbon的底层实现原理主要包括服务发现、负载均衡和远程调用三个部分。

1. 服务发现：Ribbon通过与服务注册中心（如Eureka）进行交互，获取可用的服务实例列表。服务注册中心维护了各个服务实例的网络地址和端口号等信息，Ribbon定期从注册中心获取最新的服务实例列表，并将其缓存在本地。

2. 负载均衡：Ribbon内置了多种负载均衡策略，如轮询、随机、加权轮询等。当Ribbon客户端发起请求时，它会根据所选的负载均衡策略从可用的服务实例列表中选择一个合适的服务实例进行调用。负载均衡策略的选择可以通过配置文件或注解进行配置，默认是轮询策略。

3. 远程调用：Ribbon底层使用了HTTP或TCP协议进行远程调用。在发送请求之前，Ribbon会根据所选的服务实例构建请求URL，并设置相应的请求头和请求参数。然后，Ribbon使用底层的HTTP或TCP客户端库（如Apache HttpClient、Netty等）发起请求，并等待服务端的响应。一旦收到响应，Ribbon会将响应数据返回给调用方。

# 5 Hystrix

Hystrix 可以让我们在分布式系统中对服务间的调用进行控制，加入一些**调用延迟**或者**依赖故障**的**容错机制**。

Hystrix 通过将依赖服务进行**资源隔离**，进而阻止某个依赖服务出现故障时在整个系统所有的依赖服务调用中进行蔓延；同时 Hystrix 还提供故障时的 fallback 降级机制。

**总而言之，Hystrix 通过这些方法帮助我们提升分布式系统的可用性和稳定性。**

Hystrix的核心功能有三个：**资源隔离**、**熔断**、**降级**

### 5.1.1 Hystrix 的设计原则

- 对依赖服务调用时出现的调用延迟和调用失败进行**控制和容错保护**。
- 在复杂的分布式系统中，阻止某一个依赖服务的故障在整个系统中蔓延。比如某一个服务故障了，导致其它服务也跟着故障。
- 提供 `fail-fast`（快速失败）和快速恢复的支持。
- 提供 fallback 优雅降级的支持。
- 支持近实时的监控、报警以及运维操作。

![](http://image.chn520.cn/2022-04-09/5bf1ebab-a704-4ab0-9fcb-30809d06d9bc)

## 5.2 资源隔离

Hystrix进行资源隔离的基本思想是：**将对某一个依赖服务的所有调用请求，全部隔离在同一份资源池内，不会去用其它的资源**，一共有两种实现方案：*线程池隔离*和*信号量隔离*。

### 5.2.1 线程池隔离

线程池隔离，是Hystrix最基本的资源隔离技术。**Hystrix会将每一次服务调用请求，封装在一个Command对象内，每个Command都是使用线程池内的一个线程去执行**的，这样就能跟调用线程本身隔离开来，避免因为外部依赖的接口调用超时，导致调用线程本身被卡死的情况。

### 5.2.2 信号量隔离

如果采用信号量隔离技术，**每接收一个请求，都是服务自身线程去直接调用依赖服务**，信号量就相当于一道关卡，每个线程通过关卡后，信号量数量减1，当为0时不再允许线程通过，而是直接执行fallback逻辑并返回，说白了仅仅做了一个限流。

## 5.3 降级

Hystrix的fallback降级机制，会在以下情况下触发：

- 断路器处于打开状态，直接走降级流程；
- 线程池/队列/semaphore满了，请求被reject；
- 执行了请求，但是Command的run()或construct()方法抛出异常；

上述三种情况，都属于异常情况，Hystrix会发送异常事件到断路器中去进行统计，如果断路器发现异常事件的占比达到了一定的比例，会直接开启断路。

## 5.4 熔断

Hystrix熔断，其实就是打开了断路器（Circuit Breaker）。Hystrix在运行过程中会向每个`commandKey`对应的断路器报告**成功、失败、超时**和**拒绝**的状态，断路器会统计并维护这些数据，根据这些统计信息来确定断路器的状态：**打开（open）**、**半开（half-open）**、**关闭（close）**。

- 打开状态：后续的请求都会被截断，直接走fallback降级；
- 半开状态：默认每隔5s，尝试半开，放入一部分流量进来，相当于对依赖服务进行一次健康检查，如果服务恢复了，则断路器关闭，随后完全恢复调用；
- 关闭状态：断路器完全关闭，走正常的请求调用流程。

# 6 spring-cloud-gateway

## 6.1 限流算法

- ### **固定窗口算法**

- ### **滑动窗口算法**

- ### **漏桶算法**

- ### **令牌桶算法**

## 6.2 gateway的实现方式

> **RequestRateLimiter:** 基于redis实现的过滤器限流，算法是令牌桶算法
> 
> 集成Hystrix，Sentinel等组件进行限流
