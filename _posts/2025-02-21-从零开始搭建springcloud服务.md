---
layout:     post   				    # 使用的布局（不需要改）
title:      从零开始搭建springcloud服务 
subtitle:   springcloud搭建教程
date:       2025-02-21 				  # 时间
author:     Tong 						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - springcloud
typora-copy-images-to: ../assets/images
typora-root-url: ../../blog
---

### 前言

最近在复习springcloud相关组件知识，想到学了忘，忘了学，不断的循环，虽然在这个过程中也会感悟到些新的东西，但终究所花的时间太多，由此想着为什么不从零搭建一整套微服务呢，下次如果需要直接看blog岂不更省事。说干就干



## 选型

微服务众所周知有好多组件，那我们该怎么选呢？这是个好问题。

我准备采用目前最主流最常用的springcloud + springcloud-alibaba组合。

选好了框架，那就到了该用哪个版本了

### 版本如何选择，如何解决兼容问题？

兼容问题那肯定得打开我们的阿里提供的 [版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E?spm=5238cd80.2ef5001f.0.0.3f613b7c6W6ooa) 啦

#### 2022.x分支

![image-20250221122418542](/assets/images/image-20250221122418542.png)



#### 2021.x分支

![image-20250221122451925](/assets/images/image-20250221122451925.png)



#### 2.2.x分支

![image-20250221122531089](/assets/images/image-20250221122531089.png)



#### 组件版本关系

![image-20250221122649412](/assets/images/image-20250221122649412.png)



好了，版本之间的对应关系已经有了，那就让我们开始搭建服务吧







## 建个空项目来管理依赖版本

新建一个空的Maven项目

修改pom文件，后面引入的依赖就不需要选版本号了

````xml
<dependencyManagement>
    <dependencies>
        <!-- 引入spring cloud的依赖管理 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR12</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- 引入springcloud alibaba的依赖管理 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.8.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.3.12.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
````







## 引入Nacos做注册中心

### 启动Nacos服务

根据我们使用的springcloud alibaba的版本，Nacos服务对应版本为v2.1.0，搭建Nacos服务端可以使用官网提供的jar包启动，也可以使用docker来搭建。

我个人的项目环境基本都是通过docker来安装管理的，所以采用docker来安装Nacos服务

安装命令：

````shell
# 我使用的M系列的芯片，要用arm架构的镜像v2.1.0-slim
docker run --name cloud-nacos-slim -e MODE=standalone -p 8848:8848 -p 9848:9848 -p 9849:9849 -d nacos/nacos-server:v2.1.0-slim
````

需要注意的一点是2.x相对1.x版本的Nacos新增了gRPC的通讯方式，因此需要多开放两个端口，如果没开放这两个端口服务是注册不到Nacos服务上的

![image-20250221141435184](/assets/images/image-20250221141435184.png)

服务启动后在浏览器输入：http://localhost:8848/nacos

![image-20250221141813349](/assets/images/image-20250221141813349.png)

Nacos服务搭建好了



### 服务注册发现

微服务的服务注册和发现相信都用过Eureka，要自己本地构建一个Eureka微服务，但是整合了Alibaba的Nacos则不用那么复杂，直接启动Alibaba提供的Nacos服务即可，这样让程序员把全部精力放在业务上，下面是一个简单的架构图：

![image-20250221142901717](/assets/images/image-20250221142901717.png)



### 演示效果

#### 搭建nacos-provider服务

在主项目上新建一个带springboot web的Module

##### 1、引入Nacos服务发现依赖

````xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
````



##### 2、配置YML文件

````yaml
server:
  port: 8070

spring:
  application:
    name: nacos-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
````



##### 3、开启服务注册发现功能

这个大部分Spring Boot功能模块相同，都需要使用`@EnableXxxx`注解来开启某个功能，否则无法引入自动配置。这里需要使用Spring Cloud的原生注解`@EnableDiscoveryClient`来开启服务注册发现的功能，如下：

```java
@EnableDiscoveryClient
@SpringBootApplication
public class NacosProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosProviderApplication.class, args);
    }

}
```



##### 4、写个演示服务

nacos-provider服务作为服务提供方，那自然要提供接口给服务消费者消费的，下面是随便写的接口

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello nacos";
    }

}
```



##### 5、启动项目

按照上面的5个步骤算是完成了最基本的一个服务，现在只需要启动nacos-provider这个服务即可。

启动成功之后在nacos的服务管理->服务列表这里将会发现注册进入的nacos-provider这个服务，如下图：

![image-20250221153638055](/assets/images/image-20250221153638055.png)

OK，在nacos中能够看到服务注册成功了，完成任务..........





#### 搭建nacos-consumer服务

都是要注册到nacos服务的，因此大概步骤是一样的，步骤如下：

##### 1、引入Nacos服务注册

和provider服务一样

````xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
````



##### 2、配置YML文件

````yaml
# 应用服务 WEB 访问端口
server:
  port: 8080

spring:
  application:
    name: nacos-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
````



##### 3、开启服务注册功能

还是一样

````java
@EnableDiscoveryClient
@SpringBootApplication
public class NacosConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerApplication.class, args);
    }

}
````



##### 4、写个演示服务

如何演示呢？nacos-provider提供了一个服务，那么我们就调用它的服务来演示一把。

其实Nacos集成了Ribbon，我们可以通过使用Ribbon的负载均衡来调用服务

- 创建RestTemplate，使用`@LoadBalanced`注解标注开启负载均衡：

  ```java
  @Configuration
  public class ConsumerConfig {
      
      @LoadBalanced
      @Bean
      public RestTemplate restTemplate() {
          return new RestTemplate();
      }
  
  }
  ```

  

- 直接使用注册到nacos的中的服务名作为访问地址调用服务

  ```java
  @RestController
  public class ConsumerController {
  
      @Autowired
      private RestTemplate restTemplate;
  
  
      @GetMapping("/consumer")
      public String consumer() {
          return restTemplate.getForObject("http://nacos-provider/hello", String.class);
      }
  
  }
  ```

  

##### 5、启动项目

启动成功之后将会在nacos中的服务列表中查看到两个服务，分别是nacos-provider、nacos-consumer，如下图：

![image-20250221155136418](/assets/images/image-20250221155136418.png)

此时服务提供者和消费者都已成功注册到Nacos，那么接下来就是测试服务能否调的通的问题了。

直接调用nacos-consumer的接口，输入地址：`http://localhost:8080/consumer`，返回信息如下图则表示相互调用成功：

![image-20250221155327958](/assets/images/image-20250221155327958.png)



### 总结

Nacos的服务注册发现很简单，比Eureka简单多了，无需自己构建个注册中心。







## 使用Nacos来做配置中心

为什么要用配置管理？其实这已经不仅仅是微服务的痛点了，单体服务也存在这样的痛点。试问线上的项目如果想要的修改某个配置，比如添加一个数据源，难道要停服更新？显然是不太现实，那么如何解决呢？

使用Nacos来做配置中心就可以解决这些问题啦



### 演示效果

#### 1、新建nacos-config模块

添加依赖

```xml
<!-- 引入web依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 引入nacos配置依赖 -->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>

<!-- 引入lombok依赖 -->
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
</dependency>
```



#### 2、配置YML文件

在`bootstrap.yml`文件中设置Nacos的配置，注意不要直接在application.yml文件上配置nacos，如果nacos上的配置文件是properties格式的不会报错，如果是yml格式的就会报错，这都是泪啊

```yaml
spring:
  application:
    name: nacos-config
  ## 当前环境，这个和dataId有关-> ${prefix}-${spring.profiles.active}.${file-extension}
  profiles:
    active: test
  cloud:
    nacos:
      config:
        ## nacos的地址，作为配置中心
        server-addr: localhost:8848
        ## 配置内容的数据格式，目前只支持 properties 和 yml 类型，这个和dataId有关-> ${prefix}-${spring.profiles.active}.${file-extension}
        file-extension: yml
```

在application.yml文件中配置

````yaml
server:
  port: 8090
````



#### 3、Nacos服务上新增一个配置

dataId是一个配置的唯一标识，怎么取值呢？格式如下：

````yml
${prefix}-${spring.profiles.active}.${file-extension}
````

- **`prefix`**：前缀，默认是**`spring.application.name`**的值，也可以通过配置项 **`spring.cloud.nacos.config.prefix`**来配置。
- **spring.profiles.active**： 即为当前环境对应的 profile。当 **spring.profiles.active** 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 **${prefix}.${file-extension}**
- **`file-exetension`** 为配置内容的数据格式，可以通过配置项 **`spring.cloud.nacos.config.file-extension`** 来配置。目前只支持 `properties` 和 `yaml` 类型。 **注意：**这个后缀和nacos服务上的dataid后缀要一样，我这个项目配置的就是yml，所以nacos上的文件后缀就是yml。



![image-20250226172129644](/assets/images/image-20250226172129644.png)



#### 4、写过演示服务

写个单独接收配置的组件

```java
@Data
@Component
public class DynamicNacosConfigProperties {

    @Value("${tong.name}")
    private String name;

}
```

服务类

````java
@RestController
public class HelloController {

    @Autowired
    private DynamicNacosConfigProperties dynamicNacosConfigProperties;

    @GetMapping("/hello")
    public String hello() {
        return "hello " + dynamicNacosConfigProperties.getName();
    }

}
````



#### 5、启动服务

![image-20250226172644511](/assets/images/image-20250226172644511.png)



#### 6、配置动态刷新

现在nacos上改配置文件，无法实时刷新，需要重启项目才会刷新，我们想动态刷新要怎么做呢，很简单，加个@RefreshScope注解即可

```java
@RefreshScope
@Data
@Component
public class DynamicNacosConfigProperties {

    @Value("${tong.name}")
    private String name;

}
```



现在就能动态刷新了

![image-20250226172949347](/assets/images/image-20250226172949347.png)





#### 多环境如何隔离？(namespace)

试想一下：正常的业务开发至少有三个环境吧，如下：

- dev：本地开发环境
- test：测试环境
- prod：生产环境

那么每个环境的配置肯定是不同的，那么问题来了，如何将以上三种不同的配置在Nacos能够很明显的区分呢？

很多人可能会问：`DataId`格式中不是有环境的区分吗？这个不是可以满足吗？

`DataId`当然能够区分，但是微服务配置可不止这几个啊？一旦多了你怎么查找呢？多种环境的配置杂糅到一起，你好辨别吗？

当然阿里巴巴的Nacos开发团队显然考虑到了这种问题，官方推荐用命名空间（namespace）来解决环境配置隔离的问题。

> Namespace（命名空间）：解决多环境及多租户数据的隔离问题 在多套不同的环境下，可以根据指定的环境创建不同的Namespace，实现多环境的数据隔离

Nacos中默认提供的命名空间则是`public`，上述我们创建的`config.version`这个配置就属于`public`这个命名空间，如下图：

![image-20250226174746936](/assets/images/image-20250226174746936.png)



当然我们可以根据业务需要创建自己的命名空间，我直接创建了三个，分别是**dev**、**test**、**prod**

![image-20250226175143753](/assets/images/image-20250226175143753.png)

分别给这三个命名空间新增配置文件：

![image-20250226175349830](/assets/images/image-20250226175349830.png)

![image-20250226175414996](/assets/images/image-20250226175414996.png)

![image-20250226175428845](/assets/images/image-20250226175428845.png)



此时只需要在`bootstrap.yml`配置中指定如下配置：

````yaml
spring:
  application:
    name: nacos-config
  ## 当前环境，这个和dataId有关-> ${prefix}-${spring.profiles.active}.${file-extension}
  profiles:
    active: test
  cloud:
    nacos:
      config:
        ## nacos的地址，作为配置中心
        server-addr: localhost:8848
        ## 配置内容的数据格式，目前只支持 properties 和 yml 类型，这个和dataId有关-> ${prefix}-${spring.profiles.active}.${file-extension}
        file-extension: yml
        namespace: test
````



启动项目

![image-20250226175744976](/assets/images/image-20250226175744976.png)



#### 不同业务配置如何隔离？(Group)

通过group也可以对配置进行隔离，目前没有用到，以后用到再补充。





#### Nacos如何共享配置？

一个项目的微服务数量逐渐增多，势必会有相同的配置，那么我们可以将相同的配置抽取出来作为项目中共有的配置，比如集群中的数据源信息..



Nacos中新建共享配置

![image-20250227154103074](/assets/images/image-20250227154103074.png)



修改配置

````yaml
spring:
  application:
    name: nacos-config
  ## 当前环境，这个和dataId有关-> ${prefix}-${spring.profiles.active}.${file-extension}
  profiles:
    active: test
  cloud:
    nacos:
      config:
        ## nacos的地址，作为配置中心
        server-addr: localhost:8848
        ## 配置内容的数据格式，目前只支持 properties 和 yml 类型，这个和dataId有关-> ${prefix}-${spring.profiles.active}.${file-extension}
        file-extension: yml
        namespace: test
        shared-configs:
          - data-id: share-db-config.yml
            group: DEFAULT_GROUP
            refresh: true
          - data-id: share-redis-config.yml
            group: DEFAULT_GROUP
            refresh: true
````



修改程序

````java
@RefreshScope
@Data
@Component
public class DynamicNacosConfigProperties {

    @Value("${tong.name}")
    private String name;

    @Value("${database.username}")
    private String username;

    @Value("${spring.redis.host}")
    private String host;

}

@RestController
public class HelloController {

    @Autowired
    private DynamicNacosConfigProperties dynamicNacosConfigProperties;

    @GetMapping("/hello")
    public String hello() {
        return "hello " + dynamicNacosConfigProperties.getName() + ", " + dynamicNacosConfigProperties.getUsername() + ", " + dynamicNacosConfigProperties.getHost();
    }

}
````



启动服务

![image-20250227154305448](/assets/images/image-20250227154305448.png)





### Nacos是CP还是AP？

先来简单复习下CAP的概念吧，如下：

- 一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
- 可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
- 分区容错性（P）：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

一般分布式系统中，肯定是优先保证P，剩下的就是C和A的取舍。

<img src="/assets/images/image-20250227160759524.png" alt="image-20250227160759524" style="zoom:67%;"/>



当然不同的注册中心遵循的CAP也是不同的，如下：

- Zookeeper：保证CP，放弃可用性；一旦zookeeper集群中master节点宕了，则会重新选举leader，这个过程可能非常漫长，在这过程中服务不可用。
- Eureka：保证AP，放弃一致性；Eureka集群中的各个节点都是平等的，一旦某个节点宕了，其他节点正常服务（一旦客户端发现注册失败，则将会连接集群中其他节点），虽然保证了可用性，但是每个节点的数据可能不是最新的。
- Nacos：同时支持CP和AP，默认是AP，可以切换；AP模式下以临时实例注册，CP模式下服务永久实例注册。











## 通过引入依赖的方式来自动配置Nacos

每新增一个微服务都需要配置Nacos，如果需要变更Nacos的配置，需要修改所有的文件，那有没有办法只需要在一个地方配置就所有的微服务都不用修改呢？答案是肯定的，那下面我们就做个示例吧。

首先要解决这个问题，我们要先了解到，springboot中的配置项实际上都是加载到Properties类中，所以我们的思路就是：直接在代码中将这nacos的配置项写入到Properties中。

















































