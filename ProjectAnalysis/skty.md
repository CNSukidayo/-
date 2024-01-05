### 项目结构分析  
1.项目一共分为8个模块
* jdd_sport_info_admin(controller模块)
* jdd_sport_info_api
* jdd_sport_info_mqc(消息队列模块)
* jdd_sport_info_rpc_api(rpc相关)
* jdd_sport_info_sdk
* jdd_sport_info_service(service模块)  
  * admin:
  * api:存放远程调用的相关实现
  * `common:通用模块`  
  * domain:领域模型,存放数据库的实体映射类(entity类)
  * dubbo:包含consumer和provider,分别是当前模块需要调用别的微服务模块接口以及当前模块对外暴露的接口
  * helper:一些工具类
  * repository:dao
  * service:各种service定义以及领域模型
* jdd_sport_info_task(xxl-job模块)
* jdd_sport_info_wsc(websocket模块)

2.jdd_sport_info_service=>common模块详解  
* aop:
  aop,这里是一种解耦的实现方式,通过spring来完成;切入点为  
  ```java
  @Pointcut("execution(* com.jdddata..*.api.controller..*.*(..))")
  public void pointcut() {
  }

  @Pointcut("execution(* com.jdddata..*.apic..*.controller..*.*(..)")
  public void apicPointcut() {
  }
  ```
  所以就是所有api都会走这些通知  
* bean:没什么用
* component:
* config:配置类
* constant:常量类
* context:上下文类和常量
* enums:枚举类
* event:spring事件
* facade:
* Hessian:
* ip:处理IP相关
* limiter:限制相关;通过aop实现细粒度的控制
* listener:事件监听器(监听spring事件)
* manager:
* rabbitmq:
* redis:redis的工具类
* util:工具类

