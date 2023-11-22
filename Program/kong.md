# 目录  
1.Kong环境搭建  
2.Kong的基本使用  




## 1. Kong环境搭建  
**目录:**  
1.1 Kong介绍  
1.2 Kong环境搭建(基于docker)  
1.3 Konga环境搭建  

### 1.1 Kong介绍  
1.Kong介绍  
Kong是一款基于OperResty(Nginx+Lua模块)编写的高可用、易扩展的,由Mashape公司开源的APl Gateway项目.Kong是基于NGINX和Apache Cassandra(`NOSQL`)或PostgreSQL(`关系型数据库`)构建的,能提供易于使用的`RESTful API`来操作和配置API管理系统,所以它可以水平扩展多个Kong服务器,通过前置的负载均衡配置把请求均匀地分发到各个server,来应对大批量的网络请求.  

kong调用示意图:  
![kong](resource/kong/1.png)  

使用网关的好处:  
![使用网关的好处](resource/kong/2.png)  
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
![Kong架构](resource/kong/3.png)  

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
![Navicat](resource/kong/4.png)  

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
![konga](resource/kong/6.png)  

5.执行成功之后使用Navicat连接数据库  
![Navicat](resource/kong/5.png)  

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
![注册管理员](resource/kong/7.png)  
* 用户名:admin (随便创建的)
* 密码:12345678

8.设置kong的地址  
konga是管理kong的图形界面,自然要配置当前konga管理哪个kong;所以点击左侧的connections然后点击+new connection按钮来创建连接;也可以直接在主页dashboard去创建一个连接.  
这里的配置大概如下:  
![设置konga](resource/kong/8.png)  



## 2.Kong的基本使用 
**目录:**  
2.1 动态负载均衡实现  



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

| Kong 组件 | 说明                                                                                                                                                                                                      |
| :-------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| service&emsp;&emsp;   | service就是对应一个服务(服务集群),它可以直接指向一个具体的微服务(ip+端口),也可以指向一个<font color="#00FF00">upstream</font>实现负载均衡,由<font color="#00FF00">upstream</font>来转发到具体的微服务节点 |
| route     | route对应路由,它负责将某次请求映射到一个service中;当然service可以自已选择它是映射到upstream还是target中                                                                                                   |
| upstream  | upstream 对应**一组**API节点,实现负载均衡                                                                                                                                                                 |
| target    | target对应**一个**API节点(最终具体的某个微服务节点)                                                                                                                                                                                 |

**示意图如下:**  
![请求架构图](resource/kong/9.png)  

3.创建一个upstream  
这里指定一个名称就可以了  
![创建upstream](resource/kong/10.png)  

4.配置upstream中的target节点  
<font color="#FF00FF">一个target就相当于一个微服务节点</font>  
第三步创建完upsteam之后,点击新创建的upstream的detail按钮,点击target->add target按钮即可  
![添加target](resource/kong/11.png)  

在弹出的界面中配置target信息,配置目标微服务的地址以及权重来实现负载均衡  
![配置target](resource/kong/12.png) 

**成功后的效果如下:**  
![配置target成功](resource/kong/13.png)  









