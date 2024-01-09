# 目录  
1.服务注册中心  


**附录:**  
1.微服务基本概念介绍  


## 1.服务注册中心
**目录:**  
1.1 服务注册中心基本介绍  


### 1.1 服务注册中心基本介绍  
*提示:服务注册中心的基本概念将不在赘述*  

1.`nacos`基本介绍  
nacos是一个更易于构建**云原生**应用的动态服务发现、服务配置和服务管理平台  
特点:  
* 服务发现和服务健康检测
* 动态配置服务
* 动态DNS服务
* 配置及其元数据管理
  所谓的元数据就是服务的基本信息,比如服务的地址、服务名称等;服务通过与注册中心交换数据从而感知与被感知,从而进行远程调用  

2.注册中心演变过程  
最开始服务之间的调用就类似RPC分布式系统中那样,一个服务使用类似RestTemplate通过HTTP去请求另外一个服务;但这种方式是十分不可靠的,如远程服务器的地址是不可以变更的;  

现在演变为注册中心,注册中心使用MySQL存储了一张<font color="#00FF00">注册表(server-register)</font>,注册表中的内容包含服务名称、服务地址、服务的健康状态  
当服务启动的时候就会去服务注册中心将自已注册到注册表中,接着会从注册表中获取服务列表  
服务自已会在本地维护一个定时任务,每5s向注册中心发送一个心跳,一旦注册中心超过5s没有收到心跳,则注册中心就视作当前的这个服务已经宕机,注册表就会把服务的状态设置为down;如果30s都没有收到服务的心跳,则注册表会把该服务移除  
当服务获取到所有健康的节点之后,如果要进行远程调用则不会再走nginx(如果还走nginx,则nginx还是要维护所有的节点信息,徒增消耗),而是会直接调用目标服务;在调用目标服务的时候就需要使用负载均衡,所以<font color="#00FF00">现在的负载均衡是在服务端(消费者端)来做的</font>  

![示例图](resources/springcloud/3.png)  







# 附录:  
**目录:**  
1.微服务基本概念介绍  


## 1.微服务基本概念介绍
**目录:**  
1.1 微服务基本概念介绍  
1.2 实验环境介绍  
1.3 SpringBoot与Springcloud版本对应表  
1.4 组件说明  
1.5 基本环境搭建  

### 1.1 微服务基本概念介绍
一个微服务需要由哪些内容组成  
服务的注册与发现、服务调用、服务网关、服务配置中心、负载均衡、服务熔断、服务降级、服务消息队列、服务监控、全链路追踪  
*提示:这里的服务监控包含日志收集*  
SpringCloud是一套服务的总和  

**发展历史介绍:**  
1.ORM单体应用  

2.MVC垂直架构  
这种架构说白了就是拆分系统,例如将淘宝、天猫、支付宝都单独拆分为一个独立的单体应用  
*这种架构的问题是,系统与系统之间没有交互性,无法相互调用*  

3.RPC分布式服务  
这种架构就是将所有系统公共的部分抽离出一个单独的系统,供这些系统进行使用;例如淘宝、天猫这两个系统都有用户模块、订单模块、库存模块;于是乎就将这些模块拆分出来  
*在这个阶段很多分布式场景下的问题开始出现,比如分布式缓存、分布式session、分布式锁;并且在这种架构中使用远程调用一般是通过HTTP、Dubbo的方式去进行请求,还算不上真正的远程调用,因为真正的远程调用还要考虑负载均衡、节点的健康状况等因素,在这种架构下的实现是比较复杂的*  

4.SOA  
SOA架构就是为了解决在RPC分布式架构中调用关系复杂,难以维护的问题  
SOA使用治理中心ESB,ESB会管理所有的服务(模块),每个服务都需要将自已注册到治理中心ESB的总线(BUS)中,那么治理中心就会监控所有的服务,当某一个服务出现宕机的情况,那么治理中心就会将该服务排除  
并且ESB还可以进行协议转换,例如使用ESB就可以完成两个异构系统之间的服务调用  
*缺点:服务间会有依赖关系,一旦某个环节出错会影响较大(服务雪崩)*  
什么是服务雪崩,假设下单服务需要调用订单服务、库存服务、短信服务,此时短信服务崩溃那就有可能导致整个服务的不可用,从而使得服务雪崩;这种也称为<font color="#00FF00">扇出</font>  

5.微服务架构  

**一种可用的微服务架构**  
![架构图](resources/springcloud/2.png)  
*提示:最外层的流量肯定还是走NGINX的,NGINX要通过负载均衡的算法把流量转到网关上,当然在这中间可以做分片*  

**名词解释:**  
* 水平扩容:增加节点的数量
* 垂直扩容:增加单台服务器的硬件

### 1.2 实验环境介绍  
经过慎重考虑,决定本次实验环境完全按照教程,使用OpenJdk9+SpringBoot2.0  

### 1.3 SpringBoot与Springcloud版本对应表  
![版本对应表](resources/springcloud/1.png)  
也可以访问以下连接查看最新版本支持情况:[https://start.spring.io/actuator/info](https://start.spring.io/actuator/info)  
还可以访问SpringCloudAlibaba的说明:[https://github.com/alibaba/spring-cloud-alibaba/wiki/版本说明](https://github.com/alibaba/spring-cloud-alibaba/wiki/%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E)  



### 1.4 组件说明  
排名分先后顺序  
* 服务注册中心:`zookpeer`、`nacos`、consul、eureak
* 服务调用:`Dubbo`、`OpenFeign`、Ribbon、LoadBalancer、Feign
* 服务降级:`Sentinel`、Resilience4j、Hystrix
* 服务网关:`Kong`、`gateway`、Zuul
* 服务配置:`nacos`、Config
* 服务总线:`nacos`、Bus
* 链路追踪:`Zipkin`、`Skywalking`
* 分布式事务:`Seata`
* 服务网格:`Istio`

### 1.5 基本环境搭建
1.创建父工程  

2.修改pom文件  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.github.cnsukidayo</groupId>
    <artifactId>springcloud</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--修改为pom打包的时候不会打包为jar包,因为父工程只管理配置-->
    <packaging>pom</packaging>
    <modules>
        <module>service-order</module>
        <module>service-stock</module>
    </modules>

    <properties>
        <maven.compiler.source>9</maven.compiler.source>
        <maven.compiler.target>9</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <spring.cloud.alibaba.version>2.2.5.RELEASE</spring.cloud.alibaba.version>
        <spring.boot.version>2.3.11.RELEASE</spring.boot.version>
        <spring.cloud.version>Hoxton.SR8</spring.cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>


        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <!--type:pom 因为一个项目只能有一个parent,所以这里使用这种方式来变相让SpringCloudAlibaba来管理项目-->
        <dependencies>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--SpringBoot的版本管理-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-parent</artifactId>
                <version>${spring.boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--SpringCloud的版本管理-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

        </dependencies>

    </dependencyManagement>

    <!--<build>-->
    <!--    <plugins>-->
    <!--        <plugin>-->
    <!--            <groupId>org.springframework.boot</groupId>-->
    <!--            <artifactId>spring-boot-maven-plugin</artifactId>-->
    <!--        </plugin>-->
    <!--    </plugins>-->
    <!--</build>-->


</project>
```

3.创建子模块  
分别创建两个子模块订单模块(service-order)和库存模块(service-stock)  
为这两个模块添加如下pom内容:  
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```






