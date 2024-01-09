# 目录  
1.Kong环境搭建  
2.Kong的基本使用  

**附录:**  
A.基本知识介绍  


## 1. Kong环境搭建  
**目录:**  
1.1 Kong介绍  
1.2 Kong环境搭建(基于docker)  
1.3 Konga环境搭建  

### 1.1 Kong介绍  
1.Kong介绍  
Kong是一款基于OperResty(Nginx+Lua模块)编写的高可用、易扩展的,由Mashape公司开源的APl Gateway项目.Kong是基于NGINX和Apache Cassandra(`NOSQL`)或PostgreSQL(`关系型数据库`)构建的,能提供易于使用的`RESTful API`来操作和配置API管理系统,所以它可以水平扩展多个Kong服务器,通过前置的负载均衡配置把请求均匀地分发到各个server,来应对大批量的网络请求.  

kong调用示意图:  
![kong](resources/kong/1.png)  

使用网关的好处:  
![使用网关的好处](resources/kong/2.png)  
这是kong官方的一张图,它的好处就是正常网关提供的好处,例如:统一的访问入口、统一的权限校验;把真正独立的业务下沉到各个具体的模块

**Kong主要有三个组件:**  
* Kong Server:基于nginx的服务器,用来接受API请求  
* Apache Cassandra/PostgreSQL:用来存储操作数据
* Kong dashboard:官方推荐UI管理工具;当然Kong本身支持restful风格管理kong(类似于ElasticSearch)

Kong采用插件机制进行功能定制,插件集(可以是O或N个)在APl请求响应循环的生命周期中被执行.插件使用Lua编写,目前已有几个基础功能:HTTP基本认证、密钥认证、CORS(跨域请求)、TCP、UDP、文件日志、API请求限流、请求转发以及Nginx监控.

2.Kong网关的特性  
* 可扩展性:通过简单地添加更多的服务器,可以轻松地进行横向扩展,这意味着您的平台可以在一个较低负载的情况下处理任何请求
* 模块化:可以通过添加新的插件进行扩展,这些插件可以通过RESTful Admin API轻松配置
* 在任何基础架构上运行:Kong网关可以在任何地方都能运行.您可以在云或内部网络环境中部署Kong,包括单个或多个数据中心设置,以及public,private或invite-only APls.

3.Kong架构  
![Kong架构](resources/kong/3.png)  

* Kong核心基于OpenResty构建,实现了请求/响应的Lua处理化
* Kong插件拦截请求/响应
* Kong Restful管理API提供了API/API消费者/插件的管理
* 数据中心用于存储Kong集群节点信息、APl、消费者、插件等信息,目前提供了PostgresQL和Cassandra支持,如果需要高可用建议使用Cassandra
* Kong集群中的节点通过gossip协议自动发现其他节点,当通过一个Kong节点的管理APl进行一些变更时也会通知其他节点.每个Kong节点的配置信息是会缓存的,如插件,那么当在某一个Kong节点修改了插件配置时,需要通知其他节点配置的变更

### 1.2 Kong环境搭建(基于docker)
[官网安装教程](https://konghq.com/install)  
Kong支持docker、K8S等方式的安装,可以安装在Linux、Ubuntu、Amazon Linux等操作系统上.  

1.安装docker  
详情见docker笔记  
docker下安装kong有两种方式,一种是没有数据库依赖的DB-less模式,另一种是<font color="#00FF00">With a Database模式</font>,下面将选中DataBase模式,因为这种模式功能更齐全.

2.构建Kong的容器网络  
创建一个Docker自定义网络,以允许容器相互发现和通信;下面创建的`kong-net`就是创建的Docker网络名称  
`docker network create kong-net`  

3.搭建数据库环境  
Kong使用Cassandra(NOSQL)或者PostgreSQL(关系型),两者选一即可;<font color="#00FF00">推荐使用PostgreSQL</font>(因为和MySQL非常相似).  
*注意:自定义网络为--network=kong-net*  

`Cassandra`安装如下:
```shell
docker run -d --name kong-database \
--network=kong-net \
-p 9024:9024 \
cassandra:3
```

`PostgreSQL`安装如下:  
如果需要将PostgreSQL的数据挂载到宿主机,直接使用-v命令是不好用的,推荐使用`docker volume create`命令来创建一个挂载  
例如:`docker volume create kong-volume`  
于是乎PostgreSQL就可以通过`-v kong-volume:/var/lib/postgresql/data`进行挂载了  
*提示:镜像启动成功后可以直接使用Navicat进行连接*  

```shell
docker run -d --name kong-database \
--network=kong-net \
-p 5432:5432 \
-v kong-volume:/var/lib/postgresql/data \
-e "POSTGRES_USER=kong" \
-e "POSTGRES_DB=kong" \
-e "POSTGRES_PASSWORD=kong"  \
postgres:9.6
```

**解释:**  
* POSTGRES_USER:指定数据库用户为kong
* POSTGRES_PASSWORD:指定数据库用户kong的密码为kong
* POSTGRES_DB:指定数据库为kong
* -p:PostgreSQL默认端口为5432

*提示:启动成功之后可以使用Navicat来访问该数据库*  
![Navicat](resources/kong/4.png)  

4.初始化数据库&kong  
*注意:这一步使用的是kong来初始化数据库,因为数据是存储于kong里的,PostgreSQL只是一个支撑数据库*

```shell
docker run --rm \
--network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_PASSWORD=kong" \
-e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
kong:latest kong migrations bootstrap
```

**解释:**  
* docker run --rm:使用这种方式运行docker(初始化数据库);该命令执行后会退出容器但保留内部的数据卷(volume);`kong:latest`会拉取docker镜像,在第5步会启动该镜像即kong  
* KONG_CASSANDRA_CONTACT_POINTS:指定数据库  
* KONG_PG_PASSWORD:指定数据库密码
* KONG_PG_HOST:指定host
* KONG_DATABASE:指定kong使用的数据库类型

5.启动kong容器  
完成初始化或者迁移数据库后,我们就可以启动一个连接到数据库容器的Kong容器,<font color="#00FF00">请务必保证你的数据库容器启动状态</font>,同时检查所有的环境参数`-e`是否是你定义的环境  
```shell
docker run -d --name kong \
--network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_PASSWORD=kong" \
-e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
-e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
-e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
-e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
-p 8000:8000 \
-p 8443:8443 \
-p 8001:8001 \
-p 8444:8444 \
kong:latest
```

**解释:**  
* network:kong-net指定网络  
* KONG_DATABASE:指定kong的数据库类型(固定)
* KONG_PG_HOST:指定kong使用的数据库名称,这个名称应该是第3步创建PostgreSQL时指定的名称
* KONG_PG_PASSWORD:指定数据库的密码
* KONG_ADMIN_LISTEN:kong-admin的监听端口,因为kong可以通过restful风格进行管理,通过暴露这个端口就可以对kong进行管理,这里暴露的时候指定<font color="#00FF00">0.0.0.0:8001</font>可以被远程访问

6.访问kong,看是否成功启动  
通过访问虚拟机的ip+8001端口看是否可以正常获取到kong的信息  

### 1.3 Konga环境搭建
**介绍:**  
kong本身是支持restful风格的方式进行管理的,但是这种方式太麻烦;kong自已也提供了一套图形化的Web界面来管理,但它是收费的;于是乎出现了开源的管理UI即[Konga](https://github.com/pantsel/konga)  

1.Konga也是需要使用数据库的,于是乎这里同样使用`PostgreSQL`数据库来支撑konga  

2.定义挂载卷`konga-postgresql`  
执行:`docker volume create konga-postgresql`  
执行:`docker volume ls` 可以查看目前所有的卷  
3.创建konga的数据库容器(这里是新开了一个数据库实例,貌似不能共用一个数据库?)  
```shell
docker run -d --name konga-database  \
--network=kong-net  \
-p 5433:5432 \
-v  konga-postgresql:/var/lib/postgresql/data  \
-e "POSTGRES_USER=konga"  \
-e "POSTGRES_DB=konga" \
-e "POSTGRES_PASSWORD=konga"  \
postgres:9.6
```

4.初始化konga数据库  
```shell
docker run --rm  --network=kong-net  \
pantsel/konga:latest -c prepare -a postgres -u postgres://konga:konga@konga-database:5432/konga
```

**解释:**  
* --rm:同样这里会下载konga的镜像,然后将其移除
* -c:执行的命令,这里执行的事prepare
* -a:adapter的简写,这里可以是postgres/MySQL  
  也就是说当前konga需要使用什么类型的数据库,这说明konga支持MySQL和postgres这两种数据库
* -u:指定登陆数据库的用户(类似JDBC的连接方式)  
  `konga:konga@konga-database:5432/konga`  
  <font color="#00FF00">用户名:密码@数据库地址:端口/模式(数据库)</font>

**konga的其它参数:**  
![konga](resources/kong/6.png)  

5.执行成功之后使用Navicat连接数据库  
![Navicat](resources/kong/5.png)  

6.启动konga  
```shell
docker run -d -p 1337:1337  \
--network kong-net  \
-e "DB_ADAPTER=postgres"  \
-e "DB_URI=postgres://konga:konga@konga-database:5432/konga"  \
-e "NODE_ENV=production"  \
-e "DB_PASSWORD=konga" \
--name konga \
pantsel/konga
```

**解释:**  
* DB_URI:指定数据库的连接地址(类似JDBC)  

7.访问konga  
浏览器访问:`LinuxIP:1337`  
第一次进入konga需要注册管理员  
![注册管理员](resources/kong/7.png)  
* 用户名:admin (随便创建的)
* 密码:12345678

8.设置kong的地址  
konga是管理kong的图形界面,自然要配置当前konga管理哪个kong;所以点击左侧的connections然后点击+new connection按钮来创建连接;也可以直接在主页dashboard去创建一个连接.  
这里的配置大概如下:  
![设置konga](resources/kong/8.png)  



## 2.Kong的基本使用 
**目录:**  
2.1 动态负载均衡实现  
2.2 基于Basic Auth&JWT身份认证  
2.3 kong的限流配置  
2.4 黑白名单的配置  


### 2.1 动态负载均衡实现  
kong相较于Nginx实现负载均衡的一大优势便是动态负载均衡,nginx更新负载均衡的规则后需要重启(执行`nginx -s reload`命令),而kong可以动态地更改负载均衡的规则立即生效.  

1.nginx下负载均衡的实现  
```shell
upstream tulingmall-product-upstream {
	server 192.168.65.190:8866 weight=100;
	server 192.168.65.190:8867 weight=100;
}

server {
	listen	80;
	location /pms/ {
		proxy_pass http://tulingmall-product-upstream;
	}
}
```

**解释:**  
* `upstream` 指定负载均衡的节点有哪些,server指定节点的ip和端口,weight指定节点的权重.
* `location` 指定路由(在尚上优选项目中配置过)

2.kong组件说明  

| Kong 组件           | 说明                                                                                                                                                                                                      |
| :------------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| service(服务)       | service就是对应一个服务(服务集群),它可以直接指向一个具体的微服务(ip+端口),也可以指向一个<font color="#00FF00">upstream</font>实现负载均衡,由<font color="#00FF00">upstream</font>来转发到具体的微服务节点 |
| route(路由)         | route对应路由,它负责将某次请求映射到一个service中;当然service可以自已选择它是映射到upstream还是target中                                                                                                   |
| upstream(上游)      | upstream 对应**一组**API节点,实现负载均衡                                                                                                                                                                 |
| target(目标)        | target对应**一个**API节点(最终具体的某个微服务节点)                                                                                                                                                       |
| consumer(消费者)    | 1                                                                                                                                                                                                         |
| plugin(插件)        | 1                                                                                                                                                                                                         |
| certificate(证书)   | 1                                                                                                                                                                                                         |
| SNI(服务器名称指示) | 1                                                                                                                                                                                                         |

*关于Kong组件的详细介绍见:<font color="#00FF00">附录=>A.基本知识介绍=>3.Kong的基础对象</font>*

**示意图如下:**  
![请求架构图](resources/kong/9.png)  

3.创建一个upstream  
这里指定一个名称就可以了  
![创建upstream](resources/kong/10.png)  

4.配置upstream中的target节点  
<font color="#FF00FF">一个target就相当于一个微服务节点</font>  
第三步创建完upsteam之后,点击新创建的upstream的detail按钮,点击target->add target按钮即可  
![添加target](resources/kong/11.png)  

在弹出的界面中配置target信息,配置目标微服务的地址以及权重来实现负载均衡  
![配置target](resources/kong/12.png) 

**成功后的效果如下:**  
![配置target成功](resources/kong/13.png)  

5.restful风格的好处
以上配置完全都可以使用restful风格的方式来配置,但或许你会感觉到麻烦?  
实际上这种图形界面对批量配置并不友好,实际上运维人员并不可能一个一个地去点击配置  
**<font color="#FF00FF">如果需要批量化的配置,运维编写一个脚本,直接批量发送restful风格的命令即可,而不是使用图形化界面</font>**

6.配置service  
*提示:之前说过,service可以对应一个upstream也可以对应一个具体的微服务(target);这里需要让service对应一个upstream,自然要先把upstream配置好再配置service*  
![配置service](resources/kong/14.png)  
这里面有三个主要的参数:  
* name:这是当前service的名称
* host:这是我们指定当前service与哪个upstream进行绑定,这里指定的就是第3步创建的upstream
* path:<font color="#00FF00">这个路径被用来去请求upstream</font>  
  待会第7步会配置service对应的route;route是用于做路由匹配的;假设我们请求一个地址为  
  `http://localhost:8000/api/order/getOrderInfo`  
  这是一个订单服务的请求,在编写route规则的时候需要配置path(route的path)为/api/order;这样该路由(route)才会映射到当前service,但映射成功之后kong默认会把匹配的路径删除,即请求service时如果service步做配置,service真正收到并转发给upstream的请求是  
  `http://localhost:8080/getOrderInfo`  
  而target真正接受的请求实际上就是
  `http://localhost:8000/api/order/getOrderInfo`  
  所以现在直接请求会404,为了解决这个问题,在配置service的path加上/api/order  
  <font color="#FF00FF">访问网关kong时,统一入口是8000;8001端口是kong的管理地址</font>

7.配置route  
![配置route](resources/kong/16.png)  
这里配置了路由的名称以及它对应需要的paths  

### 2.2 基于Basic Auth&JWT身份认证
**介绍:**  
kong本身提供了非常多的插件,这些插件可以辅助kong完成很多事情  
![kong的插件](resources/kong/17.png)  
总体来说插件的类型分为:**<font color="#FFC800">Authentication(身份认证)、Security(安全)、Traffic Control(流量控制)、Serverless(无服务器计算)、Analytics & Monitoring(分析与监控)、Transformations(变换器)、Logging(日志)、Other(其它)</font>**  
当然kong也支持编写自定义的插件(按照kong插件编写规范进行编写,使用lua脚本语言)  
kong中插件的粒度分为四种:<font color="#00FF00">全局 > service > 路由 & consumer</font>  
可以进入到不同的层级来配置对应的插件  
*提示:这里可能会存在误区;虽然route(路由)是匹配转发到service的,并且route也是配置在service下面的,但多个route可以映射到一个service;所以不同的请求匹配到不同的route,每个route可以配置自已的插件,固然它的粒度就小;而最终都落入到同一个service中,则service的粒度固然就大*  

**优势:**  
在传统的设计架构中,业务的API接口无论是私有的、公开的还是三方的,每个业务所需的身份认证、限速、日志、监控、缓存等功能,都需要开发人员自主实现(<font color="#00FF00">不同的语言体系</font>)并耦合到自已的应用程序当中,这会导致后续维护困难.  
<font color="#00FF00">Kong采用插件的方式统一收纳管理这些业务通用的功能</font>,使它们之间边界清晰、彼此隔离、互不影响,从而减少运维和管理成本.  

1.添加route纬度的插件(Basic Auth)  
点击侧边栏route=>选择2.1节配置的route=>local-product-route=>plugins=>+ Add plugin=>AUTHENTICATION=>basic auth  
大致效果如下:  
![配置Basic Auth](resources/kong/18.png)  
*注意:这里会有一个consumer*  

2.配置consumer  
**介绍:**  
什么是consumer,可以理解为能够使用kong中service的用户  
在刚才配置的插件下,可以看到有consumer这一栏显示的是All consumers  
![consumer](resources/kong/19.png)  
表明该认证插件允许所有consumer通过  
点击侧边栏consumer=>+ create consumer=>username指定为fox  

效果如下:  
![配置consumer](resources/kong/20.png)

可以在credentials中配置当前消费者的认证信息  
![消费者](resources/kong/21.png)  

3.Postman进行请求  
请求的时候进行如下设置即可  
![postman](resources/kong/22.png)  

4.JWT的配置稍微麻烦一点,但大同小异  

### 2.3 kong的限流配置
**介绍:**  
Kong提供了<font color="#00FF00">Rate Limiting</font>插件,实现对请求的**限流**功能,避免过大的请求量过大.  
Rate Limiting支持秒/分/小时/日/月/年多种**时间维度**的限流,并且可以组合使用;如限制每秒最多100次请求,每分钟最多1000次请求  
Rate Limiting采用的限流算法是<font color="#00FF00">计数器</font>的方式,所以无法提供类似令牌桶算法的平滑限流能力

Rate Limiting支持三种**基础维度**的限流:  
* `consumer`:每个消费者允许每秒请求的次数
* `credential`:
* `ip`(default):每个IP允许每秒请求的次数  

Rate Limiting支持三种**计数方式**进行存储:  
- `local`:存储在Nginx本地,实现单实例限流
- `cluster`(default):存储在Cassandra或PostgreSQL数据库,实现集群限流
- `redis`:存储在Redis数据库,实现集群限流

1.根据自已需要的粒度添加插件=>TRAFFIC CONTROL=>rate limiting  
![限流插件](resources/kong/23.png)  

### 2.4 黑白名单的配置
**解释:**  
白名单就是运行哪些ip来访问当前服务,黑名单就是不允许哪些ip来访问服务  

1.配置黑白名单插件  
Add plugin=>SECURITY=>ip restriction  
![配置黑白名单](resources/kong/24.png)   


## 附录:  
A.基本知识介绍  

### A.基本知识介绍  
**目录:**  
1.名词介绍  
2.Kong的基础配置  
3.Kong的基础对象  



#### 1.名词介绍  

1.网关  
网关是用于保护内部服务的安全屏障,可以用于实现:<font color="#00FF00">路由、版本、缓存、认证、授权、身份验证、限流、熔断、灰度、过滤、转换、计费、审计、脱敏、日志和监控</font>等功能
| 网关服务 |          |          |          | 网关管理 |
| :------: | :------: | :------: | :------: | :------: |
| 负载均衡 | 动态路由 | 认证授权 | 请求转换 | 管理平台 |
| 响应转换 | 流量切片 | 灰度发布 | 流量镜像 | 服务注册 |
| 限频控制 | 流量控制 | 数据缓存 | 数据校验 | 日志收集 |
| 服务计费 | 服务熔断 | 黑白名单 | 跨域设置 | 监控警报 |
| 健康检测 | 服务降级 | 服务发现 | 人机识别 | 链路分析 |
| 服务容灾 | 协议转换 | 安全防护 |   其它   | 性能分析 |


2.微服务&分布式系统  
将传统的系统拆分为多个微服务,这些微服务可以单独部署上线;不同的微服务可以使用不同的开发语言,并通过restful的方式通过网关向外提供服务接口;这样形成的系统称之为<font color="#00FF00">分布式系统.</font>  

3.nginx  
NGINX采用多进程模型,即单个主进程对应过个工作进程,主进程负责处理外部信号、读取配置文件以及初始化工作进程,工作进程则采用 **<font color="#FF00FF">异步非阻塞的epoll函数</font>** 来处理网络请求.  

4.OpenResty  
OpenResty是一个基于nginx与luaJIT的高性能web平台,其内部继承了大量精良的lua库、第三方模块.  
总之使用OpenResty使你不再需要用C、C++编写复杂的扩展模块,使用lua就能够快速构建出高性能的网关系统.  

`//todo 这里少画了配置文件`

```mermaid
flowchart LR

A[Hard] -->|Text| B(Round)
B --> C{Decision}
C -->|One| D[Result 1]
C -->|Two| E[Result 2]
```

5.Kong  
基于nginx和OpenResty,是一个具有<font color="#00FF00">分布式、高性能、可伸缩、可扩展,提供动态负载均衡、散列负载均衡、动态配置、服务注册、服务发现、服务熔断、健康检测、故障恢复、授权认证、速率限制、缓存处理、指标监控、日志收集、插件扩展、亚毫秒级延迟等特性和功能的微服务抽象层</font>.  

6.Kong的插件  
详情见:2.2 基于Basic Auth&JWT身份认证的介绍  

7.服务网格  
基础设施层,用于处理服务之间的通信,<font color="#00FF00">负责向结构复杂的云原生应用程序传递可靠的网络请求</font>.在实践中,服务网格通常被实现为一组和应用程序部署在一起的轻量级网络代理,对于应用程序和使用者来说,它是透明的、无感知的.  
服务网格不仅仅提供了传递通讯的功能,它还提供:路由、熔断、负载均衡、服务发现、重试等等其它功能.  
如果一个系统采用多种语言开发,<font color="#00FF00">则每种语言都要实现一遍远程调用的库并与业务系统耦合在一起</font>.另一种方式是通过nginx、OpenResty、Kong进行集中式的集群部署,<font color="#00FF00">然后所有请求都通过此代理被统一管理和实现</font>,这种方式称为<font color="#FF00FF">集中代理模式</font>.

<font color="#FF00FF">边车代理模式:</font>  
边车代理模式说白了就是解决<font color="#00FF00">富容器(docker)</font>的问题  
详情可见:[一文带你搞懂 Kubernetes 容器边车模式](https://zhuanlan.zhihu.com/p/645463183)  
说白了就是在一个pod中,各个容器之间共享同一个`Network Namespace`,并且可以共享同一个`Volume`(挂载)  
在这种模式下这些代理功能既不是耦合在应用程序中,也不是集中式的集群部署,而是作为独立进程被部署在应用程序的旁边,即每一个服务器上或<font color="#00FF00">pod</font>中.  
这样一个服务器上的多个应用程序就能共享这个代理,然后通过<font color="#00FF00">控制平面和数据平面</font>进行统一管理.  
![Kong](resources/kong/25.png)  

* 控制平面:节点以控制平面的角色运行,会将最新配置信息更新给数据平面的节点  
* 数据平面:节点以数据平面的角色运行,会从控制平面的节点接受最新的配置信息并提供代理服务.  


#### 2.Kong的基础配置
1.Kong使用的端口及说明  
* 8000:网关的统一入口(HTTTP),用于接收用户的HTTP请求;并将其转发给上游服务器
* 8443:监听传入的<font color="#00FF00">HTTPS</font>请求流量,并将其转发给上游服务器
* 8001:管理kong的adminAPI流量(HTTP)
* 8444:管理kong的adminAPI流量(<font color="#00FF00">HTTPS</font>)

2.Kong的配置信息  
由于使用了docker,所以全权交由docker来进行配置,详情见=>1.2 Kong环境搭建


#### 3.Kong的基础对象
在2.1动态负载均衡实现已经简单介绍过Kong的基础对象了,主要有8大类:  
* 路由(route)
* 服务(service)
* 上游(upstream)
* 目标(target)
* 消费者(consumer)
* 插件(plugin)
* 证书(certificate)
* SNI(Server Name Indication服务器名称指示)

1.路由:  
路由定义了客户端请求与服务之间的匹配规则,它支持精确匹配、模糊匹配以及正则匹配,是请求的入口点.每个路由都与**一个特定**的服务相关联,**一个服务下可以有一个或多个与之相关联的路由,每个路由匹配到的请求都将被代理到与该路由关联的服务.**<font color="#00FF00">即路由与服务之间是多对一的关系.</font>  
所以路由是一种细粒度的匹配入口点  

```mermaid
graph LR;
RouteA<-->Server
RouteB<-->Server
RouteC<-->Server
```

一个路由至少需要一个**匹配规则和协议**.针对不同的协议,必须设置以下属性中的至少一个  
* 对于HTTP协议,至少设置methods、hosts、headers或paths
* 对于HTTPS协议,至少设置methods、hosts、headers、paths敦或snis
* 对于TCP协议,至少设置sources或destinations
* 对于TLS协议,至少设置sources、destinations或snis
* 对于GRPC协议,至少设置hosts、headers顿或paths
* 对于GRPCS协议,至少设置hosts、headers、paths或snis

|          属性名称          |                                                                                                                                      说明                                                                                                                                      |
| :------------------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|            name            |                                                                                                                             表示路由名称,可选属性                                                                                                                              |
|         protocols          |                                                                                       允许的协议列表,将其设置为https时,http请求将通过升级到https请求得到答复,其默认值为["http","https"]                                                                                        |
|          methods           |                                                                                                        路由匹配的http方法列表,如GET、POST、PUT、DELETE等,这是半可选属性                                                                                                        |
|           hosts            |                                                                                                      路由匹配的域名列表.不仅支持精确匹配域名,还支持泛域名,这是半可选属性                                                                                                       |
|           paths            |                                                                                                      路由匹配的路径列表,即支持精确前缀匹配,也支持正则匹配,这是半可选属性                                                                                                       |
|          headers           |                                                                                             路由匹配的请求头列表,需要注意的是,请求头中的域名需要使用hosts属性来指定,这是半可选属性                                                                                             |
| https_redirect_status_code |                   当路由的所有属性(协议除外)都匹配,但请求的协议是http而不是https时,该状态码将响应为指定值,可将http协议强行切换成https.如果该字段的值被设置为301、302、307或308,则location响应报头由kong注入.其接受值为426、301、302、307或者308,默认值为426                    |
|       regex_priority       | 优先级值,当多个路由同时使用正则表达式且都匹配时,regex_priority大的路由具有更高的优先级.当两条请求的路径都匹配且相同,同时regex_priority也相同时,将使用最先创建的(create_at)路由.注意:对于普通的非正则表达式来说,路由的优先级是不同的(较长的路径会优先匹配).其默认值为),可选属性 |
|         strip_path         |                                                                               当请求匹配到路由中的某一条路径时,将先从请求URL中删除次路径前缀,再将请求转发至上游服务.其默认值为true,这是可选属性                                                                                |
|       preserve_host        |                                                                         当请求匹配到路由中的某一条域名时,将在请求头中使用此域名.如果将该值设置为false,则请求头中的域名将是上游服务的域名,这是可选属性                                                                          |
|            snis            |                                                                                                              使用流路由时,与此路由相匹配的SNI列表,这是半可选属性                                                                                                               |
|          sources           |                                                                    使用流路由时,与此路由相匹配的传入连接的IP源地址列表.其中,每个条目都是一个对象,其字段为ip(可选项,使用CIDR范围表示法)和port,这是半可选属性                                                                    |
|        destinations        |                                                                   使用流路由时,与此路由相匹配的请求连接的IP目的地址列表.其中,每个条目都是一个对象,其字段为ip(可选项,使用CIDR范围表示法)和port,这是半可选属性                                                                   |
|            tags            |                                                                                                             与路由相关的一组可选字符串,用于分组和过滤,这是可选属性                                                                                                             |
|          service           |                                                                                                           路由所关联的服务,这是理由代理最终要访问的对象,这是可选属性                                                                                                           |
