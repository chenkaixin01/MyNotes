# IOC

Ioc—Inversion of Control，即“控制反转”，不是什么技术，而是一种设计思想。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

有了IoC容器后，**把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是 松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活**。

## DI是什么

DI—Dependency Injection，即依赖注入：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

## IOC的配置方式

1. xml配置：太过繁琐，已放弃

2. java配置：类的创建交给我们配置的JavcConfig类来完成，Spring只负责维护和管理，采用纯Java创建方式。其本质上就是把在XML上的配置声明转移到Java配置类中

3. 注解配置：通过在类上加注解的方式，来声明一个类交给Spring管理，Spring会自动扫描带有@Component，@Controller，@RestController，@Service，@Repository这四个注解的类，然后帮我们创建并管理，前提是需要先配置Spring的注解扫描器。

## 依赖注入方式

构造器注入与注解注入（@Autowired，@Resource）

### 为什么推荐构造器注入

官方：这个构造器注入的方式**能够保证注入的组件不可变，并且确保需要的依赖不为空**。无法解决循环依赖问题

### @Autowired，@Resource有什么区别

1、@Autowired是Spring自带的，@Resource是JSR250规范实现的，@Inject是JSR330规范实现的

2、@Autowired、@Inject用法基本一样，不同的是@Inject没有required属性

3、@Autowired、@Inject是默认按照类型匹配的，@Resource是按照名称匹配的

4、@Autowired如果需要按照名称匹配需要和@Qualifier一起使用，@Inject和@Named一起使用，@Resource则通过name进行指定

# bean生命周期

 ![](https://img-blog.csdnimg.cn/20200224111702108.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NjM0MTgx,size_16,color_FFFFFF,t_70)

 Spring会扫描指定包下面的Java类，然后根据Java类构建beanDefinition对象，然后再根据beanDefinition来创建Spring的bean.

## getBean

- 解析bean的真正name，如果bean是工厂类，name前缀会加&，需要去掉
- 无参单例先从缓存中尝试获取
- 如果bean实例还在创建中，则直接抛出异常
- 如果bean definition 存在于父的bean工厂中，委派给父Bean工厂获取
- 标记这个beanName的实例正在创建
- 确保它的依赖也被初始化
- 真正创建
  - 单例时
  - 原型时
  - 根据bean的scope创建

## 循环依赖

构造器注入形成的循环依赖： 也就是beanB需要在beanA的构造函数中完成初始化，beanA也需要在beanB的构造函数中完成初始化，这种情况的结果就是两个bean都不能完成初始化，循环依赖难以解决。

Spring解决循环依赖主要是依赖三级缓存，但是的**在调用构造方法之前还未将其放入三级缓存之中**，因此后续的依赖调用构造方法的时候并不能从三级缓存中获取到依赖的Bean，因此不能解决

注解方式调用的是无参构造器，允许先将对象放入缓存中等待引用

## 生命周期

1. 实例化前：spring从上下文或者注解扫描中获取待初始化的bean

2. 实例化阶段：spring通过java反射创建Bean的实例对象

3. 依赖注入：spring检查bean是否有依赖的bean，并对这些依赖注入

4. 初始化阶段：当bean依赖注入完成后，Spring调用bean的初始方法

5. 使用阶段：

6. 销毁阶段：当spring容器关闭时，会调用所有Bean的销毁方法释放资源

# AOP

## Spring AOP和AspectJ是什么关系

![](https://img-blog.csdnimg.cn/img_convert/1715d0bbdca7aa277cfda80eef95336a.png)

1. AspectJ是更强的AOP框架，是实际意义的**AOP标准**；
2. SpringAOP只是使用了AspectJ风格的注解

## 动态代理

动态代理就是，在程序运行期，创建目标对象的代理对象，并对目标对象中的方法进行功能性增强的一种技术。

### CGlib与jdk动态代理区别

- JDK代理只能对实现接口的类生成代理；CGLib是针对类实现代理，对指定的类生成一个子类，并覆盖其中的方法，这种通过继承类的实现方式，不能代理final修饰的类。
- JDK代理使用的是反射机制实现aop的动态代理，CGLib代理使用字节码处理框架ASM，通过修改字节码生成子类。所以jdk动态代理的方式创建代理对象效率较高，执行效率较低，CGLib创建效率较低，执行效率高。
- JDK动态代理机制是委托机制，具体说动态实现接口类，在动态生成的实现类里面委托hanlder去调用原始实现类方法，CGLib则使用的继承机制，具体说被代理类和代理类是继承关系，所以代理类是可以赋值给被代理类的，如果被代理类有接口，那么代理类也可以赋值给接口。

# 自动装配

**通过注解或者一些简单的配置就能在 Spring Boot 的帮助下实现某块功能。**

SpringBoot 定义了一套接口规范，这套规范规定：SpringBoot 在启动时会扫描外部引用 jar 包中的`META-INF/spring.factories`文件，将文件中配置的类型信息加载到 Spring 容器

## 如何实现自动装配

`@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@Configuration`：允许在上下文中注册额外的 bean 或导入其他配置类
- `@ComponentScan`：扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。如下图所示，容器中将排除`TypeExcludeFilter`和`AutoConfigurationExcludeFilter`。

### @EnableAutoConfiguration:实现自动装配的核心注解

AutoConfigurationImportSelector的importselect方法会获取所有需要自动装配的配置类，扫描spring.factories；获取后再根据condition条件进行筛选，符合条件的才进行加载

# Spring事务

#### [声明式事务管理](https://javaguide.cn/system-design/framework/spring/spring-transaction.html#%E5%A3%B0%E6%98%8E%E5%BC%8F%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86)

推荐使用（代码侵入性最小），实际是通过 AOP 实现（基于`@Transactional` 的全注解方式使用最多）。

### [Spring 事务管理接口介绍](https://javaguide.cn/system-design/framework/spring/spring-transaction.html#spring-%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86%E6%8E%A5%E5%8F%A3%E4%BB%8B%E7%BB%8D)

Spring 框架中，事务管理相关最重要的 3 个接口如下：

- **`PlatformTransactionManager`**：（平台）事务管理器，Spring 事务策略的核心。
- **`TransactionDefinition`**：事务定义信息(事务隔离级别、传播行为、超时、只读、回滚规则)。
- **`TransactionStatus`**：事务运行状态。

### 事务传播行为

**事务传播行为是为了解决业务层方法之间互相调用的事务问题**。

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。

- REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。required

- SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。supports

- MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）

- REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。

- NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。

- NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

- NESTED：nested，如果当前存在事务，就在嵌套事务内执行；如果当前没有事务，就创建一个新事物

## 事务失效场景

1. 自身调用

2. 非public方法，@Transactional；编译器会报异常

3. 设置了不支持事务（NOT_SUPPORTED）

4. 异常未抛出

5. 异常不匹配
