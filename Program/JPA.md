# 目录  
1.JPA基本概念介绍  
2.Spring Data JPA  
3.底层原理  
4.SpringBoot整合Spring Data JPA  




## 1. JPA基本概念介绍
**目录:**  
1.1 JPA基本概念介绍  
1.2 Hibernate上手  
1.3 切换JPA实现  
1.4 JPA对象的四种状态  

### 1.1 JPA基本概念介绍
1.ORM  
ORM全称Object Relation Mapping;用于解决JDBC访问数据库太麻烦的问题,Mybatis、Hibernate就是ORM框架.  
java针对ORM提出提出了`JPA`,JPA本质上是一种ORM规范,并不是ORM框架,也就是JPA为了实现ORM这一功能制定了规范,其中Hibernate就是JPA的实现,所以Hibernate拥有ORM功能.  
application、JPA、JPA实现、JDBC和数据库的关系图:  
![关系图](resources/JPA/1.png)  

2.JPA Query和SQL Query的对比  
* JPA Query:面向JavaBean;代表作hibernate  
* SQL Query:面向数据库;代表作mybatis


|                     mybatis                      |                       hibernate                        |
|:------------------------------------------------:|:------------------------------------------------------:|
|                小巧、半自动、直接                |                   方便、全自动、复杂                   |
| 国内更流行,处理复杂查询;在比较复杂的系统进行使用 | 优势在简单查询,不太适合太复杂的查询,但符合`微服务`趋势 |

**注意:**  
实际上mybatis并不能算是ORM框架,<font color="#00FF00">严格来说ORM框架的定义是使用实体类来操作数据库</font>,所以mybatis只是一个半自动的ORM框架  


3.JPA&JDBC  
JPA是Sun在JDK1.5时提出的一种ORM规范,是一种对JDBC的`升级`  
* 减少对sql语句的依赖,减少基本的开发成本(但也提高了高级的开发成本)
* 提高数据库的移植性
  为了提高移植性,<font color="#00FF00">JPA推荐NOSQL</font>也就是不写SQL语句来对数据库进行查询  
  *注意:这里NOSQL的意思不是非关系型数据库的意思,而是不写SQL的意思;因为SQL是和数据库强绑定的,移植性不好*
* 进一步明确java对象和数据库类的映射关系

*提示:JPA之于ORM(持久层框架,如MyBatis、Hibernate等,用于管理应用层Object与数据库Data之间的映射)正如JDBC之于数据库驱动(好好理解这句话)*  

<font color="#00FF00">所以JPA和JDBC是两种用于操作数据库的不同规范</font>,只不过JPA是基于JDBC的;需要依赖JDBC  

JPA规范提供了如下特性:  
* ORM映射元数据  
  JPA支持XML和注解两种元数据形式,元数据描述对象和表之间的关系,框架据此将实体对象持久化到数据库表中  
  如:`@Entity`、`@Table`、`@Id`、`@Column`等注解
* JPA提供API可以直接通过操作实体来操作数据库,摆脱JDBC、SQL  
* 通过面向对象而非面向数据库的查询语言查询数据(HQL),避免程序的SQL语句紧耦合  
  如:`from Student s where s.name = ?` 注意这里的Student不是表,而是对象(实体类)  
  这种面向对象的HQL是低耦合的  

4.Spring Data JPA  
Spring Data JPA是Spring提供的一套简化JPA开发的框架,Spring Data JPA`通过约定好的方法命名规则来编写dao接口`,从而在不写接口实现的情况下实现对数据库的访问和操作.同时Spring Data JPA还提供了很多除了CRUD之外的功能,如分页、排序、复杂查询等  
![关系图](resources/JPA/2.png)  
*相当于Spring Data JPA再在之上包装了一层*  

此外,Spring Data JPA致力于为数据访问(DAO)提供熟悉且一致的编程模板;对于每种持久性存储,dao通常需要为不同存储库提供不同CRUD(增删改查)持久化操作.Spring Data为这些持久性存储以及特定实现提供了通用接口(`CrudRespository、PagingAndSortingRespository`)和模板(`jdbcTemplate、redisTemplate、RestTemplate、MongoTemplate`)  
这也是为什么使用Spring Data JPA的原因之一,<font color="#FF00FF">因为Spring Data JPA提供了对各种类型的数据库的存储服务的支持.</font>  
*注意:这里说的dao是笼统的一种叫法,指的就是数据访问层*

虽然说spring data jpa默认是基于hibernate实现的,但如果要想切换底层的实现框架也是完全可以的(可以切换为openJPA),即<font color="#FF00FF">spring data jpa支持随时切换底层的实现框架</font>  

Spring Data主要模板(Spring Data支持的持久层技术非常多):  
* Spring Data common:用于支持每个Spring Data模块的核心公共模块
* Spring Data JDBC:对JDBC的Spring Data存储库支持
* Spring Data JPA:对JPA的Spring Data存储库支持
* Spring Data MongoDB:对MongoDB的基于Spring对象文档的存储库支持
* Spring Data Redis:封装Jedis技术,对redis实现访问操作
* Spring Data Elasticsearch:对Elasticsearch实现访问操作
* Spring Data REST:将Spring Data存储库导出为超媒体驱动的RESTful资源

![spring data jpa](resources/JPA/4.png)  



5.对象/数据存储映射  

`JPA:`  
```java
@Entity
@Table(name="TUSR") // 一个类就是一个表
public class User {
  @Id
  private String id;
	@Column(name="fn") // 一个属性就是一个列
	private String name;
	private Date lastLogin;
	@OneToMany // 表的关系
	private List<Role roles;
}
```

`MongoDB`:  
```java
@Document(collection="usr")
public class User {
	@Id
	private String id;
	@Field("fn")
	private String name;
	private DAte lastLogin;
	private List<Role> roles;
}
```

`Neo4j`:  
```java
@NodeEntity
public class User {
	@GraphId
	Long id;
	private String name;
	private Date lastLogin;
	@RelatedTo(type="has", direction=Direction.OUTGOING)
	private List<Role> roles;
}
```

### 1.2 Hibernate上手
1.创建项目  
首先创建springdata根项目,在根项目下创建jpa-hibernate作为hibernate的上手项目  
创建后的模块示意图如下:  
![模块示意图](resources/JPA/9.png)  

2.修改pom  
```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.3.32.Final</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>
</dependencies>
```

3.创建实体类  
```java
@Entity
@Table(name = "tb_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long customerId;

    @Column(name = "customer_name")
    private String customerName;

    @Column(name = "customer_address")
    private String customerAddress;

    public Long getCustomerId() {
        return customerId;
    }

    public void setCustomerId(Long customerId) {
        this.customerId = customerId;
    }

    public String getCustomerName() {
        return customerName;
    }

    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }

    public String getCustomerAddress() {
        return customerAddress;
    }

    public void setCustomerAddress(String customerAddress) {
        this.customerAddress = customerAddress;
    }
}
```

*提示:*  
在JPA中讲究代码先行,现在数据库中是没有Customer这个实体类的,但是hibernate会帮助我们自动创建对应的表  
不过数据库是需要自已创建的,这里我们创建spring_data_jpa数据库  

3.1 Entity命名策略  
* 显示命名:即通过`@Table`的name属性指定对应的数据库表名称,`@Column`的name属性指定实体字段对应数据库字段的名称  
* 隐式命名(默认):交给框架来进行隐式命名

如果我们没有使用@Table或@Column指定了表或字段的名称,则由SpringImplicitNamingStrategy为我们隐式处理,表名隐式处理为类名,列名隐式处理为字段名.如果指定了表名列名,SpringImplicitNamingStrategy不起作用  
将上面处理过的逻辑名称解析成物理名称.无论在实体中是否显示指定表名列名,SpringPhysicalNamingStrategy都会被调用

3.2 JPA对象属性与数据库字段的映射  
|                             Java Type                              |             Database Type             |
|:------------------------------------------------------------------:|:-------------------------------------:|
|                        String(char,char[])                         |   varchar(char,varchar2,clob,text)    |
| Number(BigDecimal,BigInteger,Integer,Double,Long,Float,Short,Byte) | numeric(number,int,long,float,double) |
|                  int,long,float,double,short,byte                  | numeric(number,int,long,float,double) |
|                               byte[]                               |        varbinary(binary,blob)         |
|                          boolean(Boolean)                          |   BOOLEAN(bit,smallint,int,number)    |
|                           java.util.Date                           |       timestamp(Date,DateTime)        |
|                           java.sql.Date                            |       Date(timestamp,datetime)        |
|                           java.sql.Time                            |       Time(timestamp,datetime)        |
|                         java.sql.Timestamp                         |       TIMESTAMP(datetime,Date)        |
|                         java.util.Calendar                         |       TIMESTAMP(datetime,Date)        |
|                           java.lang.Enum                           |         NUMERIC(varchar,char)         |
|                       java.util.Serializable                       |        varbinary(binary,blob)         |


4.写配置文件  
在resource目录下创建hibernate.cfg.xml作为hibernate的配置文件  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="connection.username">root</property>
        <property name="connection.password">root</property>
        <property name="connection.driver">com.mysql.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql://192.168.149.131:7901/jdd_sport_info?useSSL=FALSE</property>


        <!-- 选择数据库方言 -->
        <property name="dialect">org.hibernate.dialect.MySQL8Dialect</property>
        <!-- 是否在控制台打印sql语句 -->
        <property name="show_sql">true</property>
        <!-- 是否格式化sql语句，也就是是否使sql语句按标准规范打印出来 -->
        <property name="format_sql">true</property>
        <!-- 表的生成策略;
        none:不自动生成;
        update:没有的表会创建,已经存在的表会检查更新;
        create:每次都会将表重新生成一遍 -->
        <property name="hbm2ddl.auto">update</property>
        <!-- 指定具体的pojo类路径,指定了路径后相当于被hibernate接管 -->
        <mapping class="com.cnsukidayo.hibernate.Customer"/>
    </session-factory>
</hibernate-configuration>
```

**关于SQL方言**  
方言说白了就是不同的数据库产品有不同的SQL语法规则,所以需要选择方言;这里选择的是MySQL8  
找到Dialect类,按下`Ctrl+H`查看类继承关系  
![方言](resources/JPA/5.png)  
比如这里使用的MySQL就选择MySQL对应的方言,还可以根据不同的引擎选择不同的方言,选择方言实际上就是选择数据库  

5.创建测试类  
在测试目录下创建测试类来测试hibernate  
```java
public class HibernateTest {
    // session工厂,类似mybatis中的sqlSession;是数据库的一次会话
    private SessionFactory sessionFactory;

    @Before
    public void init() {
        StandardServiceRegistry registry = new StandardServiceRegistryBuilder().configure("/hibernate.cfg.xml").build();
        // 根据服务注册类创建一个元数据资源集,同时构建元数据并生成应用
        sessionFactory = new MetadataSources(registry).buildMetadata().buildSessionFactory();
    }

    @Test
    public void testC() {
        // 通过Session进行持久化操作
        Session session = sessionFactory.openSession();
        Transaction transaction = session.beginTransaction();
        Customer customer = new Customer();
        customer.setCustomerName("蔡徐坤");
        session.save(customer);
        transaction.commit();
        session.close();
    }

}
```

运行上述testC方法后成功在数据库中看到插入的数据  
![结果](resources/JPA/6.png)  

6.更多的示例  
```java
@Test
public void testR() {
    // 通过Session进行持久化操作
    Session session = sessionFactory.openSession();
    // 通过实体类进行查询
    Customer customer = session.find(Customer.class, 1L);
    System.out.println(customer);
    session.close();
}

@Test
public void testR_lazy() {
    // 通过Session进行持久化操作
    Session session = sessionFactory.openSession();
    // 懒加载查询,通过控制台的日志可以发现,当调用load方法时并不会立即去数据库中查询数据,而是当真正使用customer对象时才会查询;所以这里会先打印===========这段内容,然后才去数据库中查询数据
    Customer customer = session.load(Customer.class, 1L);
    System.out.println("=======================");
    System.out.println(customer);
    session.close();
}

@Test
public void testU() {
    Session session = sessionFactory.openSession();
    Customer customer = new Customer();
    //customer.setCustomerId(1L);
    customer.setCustomerName("鸡哥");
    // 如果设置了id则会更新,如果没有设置id则会插入数据
    session.saveOrUpdate(customer);
    session.close();
}

@Test
public void testJPQL() {
    Session session = sessionFactory.openSession();
    // JPQL可以省略 select * 步骤;而且这里from的不是一张表,而是一个对象
    // 这里customerId使用的不是数据库的字段,而是对象的字段,并且后面的:id是占位符的意思,在setParameter中指定查询的具体参数即可
    String jpql = "from Customer where customerId=:id";
    List<Customer> resultList = session.createQuery(jpql, Customer.class)
            .setParameter("id",1L)
            .getResultList();
    System.out.println(resultList);
    session.close();
}

```

### 1.3 切换JPA实现
*提示:*hibernate只是一种JPA的实现,本节展示如何切换JPA的实现  
在resource目录下创建META-INF目录,接着在该目录下创建persistence.xml配置文件  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <!--
    持久化单元,在这里可以定义多个持久化单元;以后就可以通过这里的name来切换多个不同的JPA的实现
    name:持久化名称
    transaction-type:事务管理方式
        JTA:分布式事务管理
        RESOURCE_LOCAL:本地事务管理
    -->
    <persistence-unit name="hibernateJPA" transaction-type="RESOURCE_LOCAL">
        <!--JPA的实现-->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <!--需要进行ORM的POJO类-->
        <class>com.cnsukidayo.hibernate.Customer</class>
        <!--可选配置,配置JPA实现方的配置-->
        <properties>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="root"/>
            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mysql://192.168.149.131:7901/spring_data_jpa?useSSL=FALSE"/>
            <!--配置hibernate的配置信息-->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL8Dialect"/>
        </properties>

    </persistence-unit>
</persistence>
```

1.创建测试类  
*提示:既然是使用JPA的方式,所以肯定就不是使用hibernate的SessionFactory来操作数据库了,而是要使用JPA规范的API来进行实现了,就有点像JDBC了;我们使用的JDBC都是抽象API它背后的实现是数据库厂商进行实现的*  

```java
public class JPATest {

    private EntityManagerFactory entityManagerFactory;

    @Before
    public void init() {
        // args0:指定持久化单元的名称,和persistence.xml配置文件中的name对应
        entityManagerFactory = Persistence.createEntityManagerFactory("hibernateJPA");
    }

    @Test
    public void testC() {
        EntityManager entityManager = entityManagerFactory.createEntityManager();
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();

        Customer customer = new Customer();
        customer.setCustomerName("鸡哥");
        entityManager.persist(customer);

        transaction.commit();
    }

}
```
EntityManager说白了就是JPA规范提供的API,就像Connect一样;<font color="#00FF00">只不过真正的实现是hibernate而已</font>.  

运行代码,查看数据库发现数据已经被插入  
![数据](resources/JPA/7.png)  

*提示:如果需要另外的实现,只需要在persistence.xml配置文件中添加一个新的持久化单元,并且在调用createEntityManagerFactory方法的时候指定对应的name即可*  

2.更多的示例  
```java
@Test
public void testR() {
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();
    // 懒加载查询
    Customer customer = entityManager.getReference(Customer.class, 1L);

    transaction.commit();
}

```
<font color="#00FF00">提示:JPA没有单独的更新语句,如果传入的对象有id则会先查询,查询有结果就更新否则就插入,这是JPA的特性</font>  

3.另外,JPA也是支持原生SQL的;如果有业务场景必须使用原生SQL完成,JPA也是支持的  

4.无法删除游离状态的实例  
```java
@Test
public void testD() {
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();
    /*
    无法删除处于游离状态的实例
    Customer customer = new Customer();
    customer.setCustomerId(1L);
    */
    // 只有从数据库中查出的实例才可以被删除
    Customer customer = entityManager.find(Customer.class, 1L);
    entityManager.remove(customer);

    transaction.commit();
}
```

### 1.4 JPA对象的四种状态
1.JPA对象(Entity)的四种状态  
Entity的生命周期由EntityManager管理,其生命周期在persistence context内  

* 临时状态(New/Transient):刚创建出来,由手动New出来的对象,没有与entityManager发生关系,没有被持久化,不处于entityManager中的对象(不被entityManager管理的对象)
* 持久状态(Managed/Persistent):与entityManager发生关系,已经被持久化,可以把持久化状态当作`实实在在的数据库记录`(此状态的属性值修改,将在提交时,同步数据库)  
  <font color="#FF00FF">在持久状态下的任何修改,都将在事务提交之后同步到数据库</font>  
* 删除状态(Removed):执行remove方法,事务提交之前
* 游离状态(Detached):游离状态就是提交到数据库后,事务commit后实体的状态.因为事务已经提交了,此时实体的属性的任何改变,`都不会同步到数据库`

<font color="#00FF00">对象状态转换图如下:</font>  
![状态](resources/JPA/3.png)

Entity生命周期的四个基本操(CRUD)  
```java
EntityManager em = factory.createEntityManager();

EntityTransaction tx = em.getTransaction();
tx.begin();
// 临时状态
Customer customer = new Customer();
// 游离状态
customer.setCustId(6L);
// 持久状态
customer = em.find(Customer.class, 5L);
// 删除状态
em.remove(customer);

tx.commit();
```

持久状态:  
```java
EntityManager em = factory.createEntityManager();

EntityTransaction tx = em.getTransaction();
tx.begin();
// 持久状态
customer = em.find(Customer.class, 5L);
customer.setCustomerName("鸡哥");

tx.commit();
```
最终的结果是,虽然没有调用merge方法(merge方法是JPA的update效果);但当事务提交之后也会修改数据库中对应的实体的数据  

2.JPA中的持久性上下文  
* 持久化上下文的生命周期与系统事务一致
* 持久化上下文提供自动脏检查
* 持久化上下文是一级缓存

**持久化上下文提供自动脏检查:**  
在事务提交的时候,JPA会执行一个脏检查机制,会检查持久化上下文中的对象状态和数据库中的状态是否一致,如果不一致,就会根据持久化上下文中的状态去更新数据库中的状态.**但是这个动作只有在数据库事务提交的时候在会做,如果事务回滚了,不会做这个动作**

可以调用JpaRepository提供的flush或saveAndFlush方法立刻同步状态到数据库,而不是等到事务提交的时候在同步.需要注意的是,这里的立刻同步到数据库是指将修改/删除操作所执行的SQL语句先执行,此时事务并没有提交,只有在事务提交后,这个更新/删除才会起作用  

3.hibernate中的缓存  
hibernate中缓存的概念类似于mybatis中缓存的概念  
hibernate中有一级缓存和二级缓存  
* 一级缓存:在同一个EntityManager中查询同一条记录时只会查询一次
* 二级缓存:不同的EntityManager查询同一条记录也只会查询一次

一级缓存实验:  
```java
@Test
public void testCache() {
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    Customer customer0 = entityManager.find(Customer.class, 1L);
    Customer customer1 = entityManager.find(Customer.class, 1L);

    transaction.commit();
}
```
通过查看日志发现,这里只会查询一次  



## 2.Spring Data JPA
**目录:**  
2.1 Spring Data JPA基本环境搭建  
2.2 Spring Data Repository  
2.3 自定义操作  
2.4 JPA注解(表/属性)  
2.5 JPA注解(关联)  
2.6 乐观锁  
2.7 审计  

### 2.1 Spring Data JPA基本环境搭建
1.创建Spring Data JPA的项目  
在springdata根模块下创建spring-data-jpa模块  

2.修改pom  
修改父项目的pom文件,添加一个类似parent的依赖来统一之后jpa需要使用的各种依赖的版本  
```xml
<!-- 统一管理springdata子项目的版本 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-bom</artifactId>
            <version>2021.1.0</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>
```

来到刚才创建的spring-data-jpa模块的pom文件,修改引入spring-data-jpa依赖  
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
    </dependency>
    <!--由于spring-data-jpa只是基于JPA框架的封装,所以这里需要为spring-data-jpa指定当前使用的底层jpa实现框架是什么-->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.5.0.Final</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>

    <!--连接池-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.8</version>
    </dependency>
    <!--spring-test-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring‐test</artifactId>
        <version>5.3.10</version>
        <scope>test</scope>
    </dependency>

</dependencies>
```

3.编写实体类  
在项目下创建com.cnsukidayo.jpa.pojo包,在该包下创建Customer和上面的定义一样  
详情见1.2 Hibernate上手=>3.创建实体类  

4.编写dao层的repository接口  
在项目下创建com.cnsukidayo.jpa.repository包,再在该包下创建CustomerRepository接口,定义如下  
```java
public interface CustomerRepository extends CrudRepository<Customer, Long> {
}
```

5.编写config配置类  
在项目下创建com.cnsukidayo.jpa.config包,在该包下创建SpringDataJPAConfig配置类,其中的内容如下:  
```java
@Configuration
// 设置repository接口的路径
@EnableJpaRepositories(basePackages = "com.cnsukidayo.jpa.repository")
@EnableTransactionManagement
public class SpringDataJPAConfig {

    @Bean
    public DataSource dataSource() {
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUsername("root");
        druidDataSource.setPassword("root");
        druidDataSource.setDriverClassName("com.mysql.jdbc.Driver");
        druidDataSource.setUrl("jdbc:mysql://192.168.149.131:7901/spring_data_jpa?useSSL=FALSE");
        return druidDataSource;
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {

        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        vendorAdapter.setGenerateDdl(true);

        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setJpaVendorAdapter(vendorAdapter);
        // 注意这里要添加实体类的包路径
        factory.setPackagesToScan("com.cnsukidayo.jpa.pojo");
        factory.setDataSource(dataSource());
        return factory;
    }

    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {

        JpaTransactionManager txManager = new JpaTransactionManager();
        txManager.setEntityManagerFactory(entityManagerFactory);
        return txManager;
    }

}
```

6.编写测试类  
```java
@ContextConfiguration(classes = SpringDataJPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringDataJPATest {

    @Autowired
    private CustomerRepository customerRepository;

    @Test
    public void testR(){
        Optional<Customer> customer = customerRepository.findById(1L);
        System.out.println(customer.get());
    }
}
```
最终的结果是成功查询到数据  

### 2.2 Spring Data Repository
*解释:Spring官方提供了很多Repository,通过这些Repository来实现对数据库不同的操作*  

1.Repository的继承关系  
在JPA中`Repository`接口作为一个标识,它的子接口扩展了一些功能  
* Repository:仅仅是一个标识,表示任何继承它的类就一个仓库接口  
  * CrudRepository:实现了CRUD相关方法
    * PagingAndSortingRepository:实现了分组排序相关方法
      * JpaRepository:实现了JPA规范相关方法
* QueryByExampleExecutor:详情见2.3 自定义操作=>2.3.3 Query By Example
* JpaSpecificationExecutor:详情见2.3 自定义操作=>2.3.4 Specification
* QuerydslPredicateExecutor:详情见2.3 自定义操作=>2.3.5 QueryDSL

2.CrudRepository  
*基本的CRUD接口*
* `save(S) return S`
  用于插入和修改,有主键就是修改;没有主键就是插入;如果是插入该方法返回的对象会携带插入后的自增ID
* `saveAll(Iterable<S>) return Iterable<S>`
  插入一个集合
* `findById(ID) return Optional<T>`
  根据ID查询一个数据
* `existsById(ID) return boolean`  
  通过主键查询是否存在返回布尔值
* `findAllById(Iterable<ID>) return Iterable<T>`
  通过ID集合查询出所有的数据
* `count() return long`
  获取当前实体的数量
* `deleteById(ID) return void`
  根据ID删除实体
* `delete(T) return void`
  根据一个实体删除实体
* `deleteAllById(Iterable<? extends ID>) return void`
  根据ID批量删除实体
* `deleteAll(Iterable<? extends T>) return void`
  根据实体批量删除所有的实体
* `deleteAll() return void`
  删除所有实体

3.PagingAndSortingRepository
*实现了分页和排序的接口,该接口继承自CrudRepository*  
* `findAll(Sort) return Iterable<T>`
  查询所有的数据,通过Sort进行排序
* `findAll(Pageable) return Page<T>`

4.QueryByExampleExecutor  
详情见2.3 自定义操作=>2.3.3 Query By Example

**通过一个示例来演示该接口的用法**  
3.1 修改CustomerRepository接口如下  
```java
public interface CustomerRepository extends PagingAndSortingRepository<Customer, Long> {
}
```

3.2 编写测试类
```java
@ContextConfiguration(classes = SpringDataJPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringDataJPAPagingAndSortTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Test
    public void testPaging(){
        System.out.println(customerRepository.findAll(PageRequest.of(0, 2)).getContent());
    }
    @Test
    public void testSort(){
      // 采用属性硬编码的方式,有个问题是属性名可能发生变化
        Sort.by("customer_id")
                .descending()
                .and(Sort.by("customer_address"));
    }

    @Test
    public void testTypeSafe() {
        // 通过类型安全的方式来构建Sort
        Sort.TypedSort<Customer> sortType = Sort.sort(Customer.class);
        Sort sort = sortType.by(Customer::getCustomerId)
                .ascending()
                .and(
                        sortType.by(Customer::getCustomerAddress)
                                .descending()
                );
    }
}
```

3.3 PageRequest  
`static of(int,int,Direction,String...) return PageRequest`  
在JPA中可以通过PageRequest的of静态方法来创建一个分页查询对象  
agrs0:代表当前是第几页(默认从第0页开始计算)  
args1:每页的大小  
args2:排序的顺序;这是一个枚举类,可以自已查看该类的内容就懂了  
args3:哪些字段参与排序  
例如:`PageRequest.of(1, 50, Sort.Direction.DESC, "id");`  
查找第一页,每页大小为50,按照id降序返回结果  

3.4 Sort  
当然Sort对象也是可以调用它的静态方法来构造的,用的时候只需要把sort传入到args2处即可;单独构造Sort可以使查询更加灵活  
<font color="#00FF00">一般而言都是结合PageRequest和Sort进行使用的,单独调用findAll(Sort)方法查询所有数据比较少用</font>  
<font color="#FF00FF">另外推荐使用类型安全的方式构建Sort避免以后数据库字段发生改变</font>  

### 2.3 自定义操作
**介绍:**  
2.2节介绍的repository还是有很多局限性的,如果需要更加灵活的SQL查询就需要自定义擦做
**目录:**  
2.3.1 JPQL  
2.3.2 规定方法名称  
2.3.3 Query By Example  
2.3.4 Specification  
2.3.5 QueryDSL  

#### 2.3.1 JPQL
```java
public interface CustomerRepository extends PagingAndSortingRepository<Customer, Long> {
    /**
     * 在JPA中如果要查询表中所有字段则可以省略select * 直接从from开始写
     * 这里from的后面是实体类,而不是表
     * where后面的条件customerName也是实体类的属性
     * 并且该方法的参数是通过顺序来指定的,即?1占位符表示方法的第一个参数
     */
    @Query("from Customer where customerName = ?1")
    Customer findCustomerByCustomerNameWithOrder(String customerName);

    /**
     * 这里如果想要使用参数名来指代占位符,不能直接使用形参的名称
     * 而必须指定@Param注解,在该注解中指定该占位符的名称与形参的对应情况
     * 并且必须要加冒号:
     */
    @Query("from Customer where customerName =: customerName")
    Customer findCustomerByCustomerNameWithName(@Param("customerName") String customerName);

    /**
     * 更新操作
     * 在spring-data-jpa中,增删改操作必须使用事务;
     * @Transactional 注解可以标注在service层,也可以标注在dao层;
     * 此外还必须标注@Modifying注解告知spring-data-jpa这是增删改的操作
     */
    @Modifying
    @Transactional
    @Query("update Customer c set c.customerName =: customerName where c.customerId =: customerId")
    int updateCustomer(@Param("customerName") String customerName, @Param("customerId") Long customerId);

    // 删除操作
    @Modifying
    @Transactional
    @Query("delete from Customer c where c.customerId = ?1")
    int deleteCustomer(Long id);

    /**
     * JPQL其实是不支持新增的
     * 但可以通过hibernate实现伪新增的方式
     * 如果你把以下代码复制进IDEA你会发现语法提示也没有了,而且insert爆红;
     * 说明JPQL它是不支持插入的,但这里如果执行该方法是会插入成功的
     * 并且这种该方法只能在hibernate中才能成功
     */
    @Modifying
    @Transactional
    @Query("insert into Customer(customerName) select c.customer from Customer c where c.customerId = ?1")
    int insertCustomerBySelect(Long id);
}
```

*JPQL也是支持使用原生SQL的*  
```java
public interface CustomerRepository extends PagingAndSortingRepository<Customer, Long> {
    @Query(value = "select * from tb_customer where customer_name =: customerName",nativeQuery = true)
    Customer findCustomerByCustomerNameWithNative(@Param("customerName") String customerName);
}
```
需要指定`nativeQuery = true`参数  

#### 2.3.2 规定方法名称
1.Spring Data JPA的查询约定  
![约定表](resources/JPA/8.png)

2.方法名的命名规范  
spring-data-jpa中方法命名主要由两部分组成:  
* 主题关键字:用于决定当前方法的作用(通常用于方法的前缀)
  * 以`find...By、read...By、get...By、query...By、search...By、stream...By`关键字开头的方法都是用于查询数据的
  * `exists...By` 是否存在
  * `count...By` 计数
  * `delete...By、remove...By` 删除
  * `...First[number(default=1)]...、...Top[number(default=1)]...`  
    First和Top关键字可以放在find(以及其它关键字)与by之间的任意位置,其中`number`指代前多少条数据被本次方法所影响
    例如:  
    `findFirst8ByName()、deleteTop3ByTime()`
  * `...Distinct...` 去重,Distinct关键字可以放在find(以及其它关键字)与by之间的任意位置
* 谓词关键字:决定查询条件(这里只举部分例子)
  * `And` and条件
  * `Or` or条件
  * `Like` 模糊查询,<font color="#00FF00">在JPA中模糊查询是需要自已拼接模糊查询%%符号的</font>

**<font color="#FF00FF">通过约定的名称进行CRUD也是需要在增删改方法上面标注@Modifying注解的</font>**

#### 2.3.3 Query By Example
**不足:** 通过2.3.1节JPQL和2.3.2节的规定方法名称是不能实现动态查询条件的;
动态查询条件的意思就是,例如做搜索查询的时候,有时需要根据名称查询,有时需要根据时间查询,只通过上述的两种方式是没办法实现这种动态查询的功能的,它不像mybatis一样如果是null就没有这个条件  

通过2.3.3、2.3.4、2.3.5这三节讲述的内容都是可以实现动态查询的,但是它们的使用场景都各不相同  

1.Query By Example特点介绍  
* 它只支持查询
* 不支持嵌套或分组的查询
  例如查询name = '张三' or name = '李四' 这种嵌套查询就不支持,要查只能查单个
* 只支持字符串查询 `start/contains/ends/regex`匹配和其他属性类型的精确匹配
  即开头/包含/结尾/正则 这四种方式的匹配

2.创建CustomerQBERepository  
```java
public interface CustomerQBERepository extends
        PagingAndSortingRepository<Customer, Long>,
        QueryByExampleExecutor<Customer> {
    
}
```

*多提一嘴:这里的QueryByExampleExecutor包括后面的JpaSpecificationExecutor、QuerydslPredicateExecutor这些执行器都是需要搭配PagingAndSortingRepository或者说是CrudRepository使用的,如果这里的接口只继承QueryByExampleExecutor这一个接口肯定是不行的,具体原理涉及到spring-data-jpa底层执行时的一些逻辑*  

3.编写测试类  
```java
@ContextConfiguration(classes = SpringDataJPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class QBETest {
    @Autowired
    private CustomerQBERepository customerQBERepository;

    @Test
    public void testR() {
        // 通过这种方式就可以做到类似mybatis一样的效果
        // 只查询前端传过来的字段,对于对象为null的字段不进行查询
        // 再次说明该方法只能用于查询一个对象,不能嵌套查询
        Customer customer = new Customer();
        customer.setCustomerName("吴亦凡");
        customer.setCustomerAddress("朝阳看守所");
        Example<Customer> example = Example.of(customer);
        List<Customer> result = (List<Customer>) customerQBERepository.findAll(example);
        System.out.println(result);
    }

    /**
     * 通过匹配器来进行条件的限制
     * 对应上述讲的start/contains/ends/regex这四个条件
     */
    @Test
    public void testMatch() {
        Customer customer = new Customer();
        customer.setCustomerName("吴亦凡");
        customer.setCustomerAddress("BEIJING");
        // 通过该匹配器表示在查询的时候忽略customerName这个条件
        ExampleMatcher exampleMatcher = ExampleMatcher.matching()
                .withIgnorePaths("customerName")    // 忽略customerName条件
                .withIgnoreCase("customerAddress") // 忽略customerAddress属性的大小写
                .withStringMatcher(ExampleMatcher.StringMatcher.CONTAINING)
                .withMatcher("customerName", ExampleMatcher.GenericPropertyMatcher::endsWith); //对单个属性进行匹配,第二个参数是lambda表达式
        // 把exampleMatcher匹配器传入到Example中
        Example<Customer> example = Example.of(customer, exampleMatcher);
        List<Customer> result = (List<Customer>) customerQBERepository.findAll(example);
    }
}
```

*提示:ExampleMatcher提供了很多静态方法来对条件进行限制;再次强调它只适用于字符串查询或其它类型的精确匹配*  


4.QueryByExampleExecutor  
* `findOne(Example<S>) return Optional<S>`
* `findAll(Example<S>) return Iterable<S>`
* `findAll(Example<S>,Sort) return Iterable<S>`
* `findAll(Example<S>,Pageable) return Page<S>`
* `count(Example<S> example) return long`
* `exists(Example<S>) return boolean`

*注意:其实这些方法和CrudRepository中的方法没什么两样(从结果层面来说)*

#### 2.3.4 Specification
1.创建CustomerSpecificationRepository  
```java
public interface CustomerSpecificationRepository extends
        PagingAndSortingRepository<Customer, Long>,
        JpaSpecificationExecutor<Customer> {
}
```

2.编写测试类  
```java
@ContextConfiguration(classes = SpringDataJPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class SpecificationTest {
    @Autowired
    private CustomerSpecificationRepository customerSpecificationRepository;

    @Test
    public void testR() {
        List<Customer> result = customerSpecificationRepository.findAll((root, query, criteriaBuilder) -> {
            /*
            root:用户获取需要查询的列;得到的返回值的泛型是该属性对应的泛型(需要手动设置,默认是Object)
            criteriaBuilder:构造查询条件
            query:组合 orderBy,where
             */
            Path<Long> customerId = root.get("customerId");
            Path<String> customerName = root.get("customerName");
            Path<String> customerAddress = root.get("customerAddress");
            // 通过criteriaBuilder来设置查询条件;args0:为那个字段设置条件;args1:值
            criteriaBuilder.equal(customerAddress, "BEIJING");
            // 返回最后一个生成的predicate即可
            Predicate predicate = criteriaBuilder.greaterThan(customerId, 0L);
            return predicate;
        });
        System.out.println(result);
    }

    @Test
    public void testSearch() {
        // 模拟一个真实的查询效果,假设这里的Customer是前端传过来的
        Customer customer = new Customer();
        customer.setCustomerName("蔡徐坤");

        List<Customer> result = customerSpecificationRepository.findAll((root, query, criteriaBuilder) -> {
            Path<Long> customerId = root.get("customerId");
            Path<String> customerName = root.get("customerName");
            Path<String> customerAddress = root.get("customerAddress");
            // 需要一个List来保存所有的动态条件
            List<Predicate> list = new ArrayList<>();
            if (customer.getCustomerId() != null) {
                list.add(criteriaBuilder.equal(customerId, customer.getCustomerId()));
            }
            if (StringUtils.hasText(customer.getCustomerName())) {
                list.add(criteriaBuilder.equal(customerName, customer.getCustomerName()));
            }
            if (StringUtils.hasText(customer.getCustomerAddress())) {
                list.add(criteriaBuilder.equal(customerAddress, customer.getCustomerAddress()));
            }
            Predicate predicate = criteriaBuilder.and(list.toArray(new Predicate[list.size()]));
            return predicate;
        });
        System.out.println(result);
    }

    @Test
    public void testSearchAndSort() {
        // 模拟一个真实的查询效果,假设这里的Customer是前端传过来的
        Customer customer = new Customer();
        customer.setCustomerName("蔡徐坤");

        List<Customer> result = customerSpecificationRepository.findAll((root, query, criteriaBuilder) -> {
            Path<Long> customerId = root.get("customerId");
            Path<String> customerName = root.get("customerName");
            Path<String> customerAddress = root.get("customerAddress");
            // 需要一个List来保存所有的动态条件
            List<Predicate> list = new ArrayList<>();
            if (customer.getCustomerId() != null) {
                list.add(criteriaBuilder.equal(customerId, customer.getCustomerId()));
            }
            if (StringUtils.hasText(customer.getCustomerName())) {
                list.add(criteriaBuilder.equal(customerName, customer.getCustomerName()));
            }
            if (StringUtils.hasText(customer.getCustomerAddress())) {
                list.add(criteriaBuilder.equal(customerAddress, customer.getCustomerAddress()));
            }
            Predicate[] where = list.toArray(new Predicate[list.size()]);
            // 如果需要按某些字段排序
            // 需要使用query对象的方法来进行实现,最终调用getRestriction方法返回Predicate对象
            Order desc = criteriaBuilder.desc(customerId);
            return query.where(where).orderBy(desc).getRestriction();
        });
        System.out.println(result);
    }

}
```

<font color="#00FF00">这里的第三个测试方法是比较重要的,可以重点看一下</font>  

3.Specifications详解  
<font color="#00FF00">Specifications是一个用于构建动态查询条件的抽象类 </font> 
该接口有如下方法:  
```java
Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder);
```
通过<font color="#00FF00">实现</font>该抽象方法来构建动态查询条件,查询条件最终会生成一个查询条件对象(该方法最终需要返回一个Predicate对象);该方法有三个回调参数:  
* Root:用户获取需要查询的列  
  如:`root.get("id")`
* CriteriaQuery:用于自定义查询方式  
  如:`query.distinct(true).select(...).where(...).groupBy(...).having(...).getRestriction()`最后通过getRestriction()方法,可以获得一个`Predicate`对象
  *注意:虽然这里有groupBy和having方法,但实际上调用该方法也不会生效,因为它的底层会将你设置的值改掉,即Specification不支持分组、聚合函数;像这样的函数还有multiselect等方法,需要自已鉴别*
* CriteriaBuilder:构造查询条件(用于构建`Predicate`对象)  
  通过`criteriaBuilder.and(...)`、`criteriaBuilder.equal(...)`、`criteriaBuilder.greaterThan()`等方法来构建一个Predicate对象  

*注意:这三个参数都是JPA规范提供的抽象类,而不是spring提供的;实际上你可以通过EntityManager来得到这三个参数构造自已的查询参数,摆脱SpringDataJPA对CriteriaQuery的限制(但要求也变高了)*

通用的查询模板:  
*提示:这里查询的方式和第2步的方式实际上是大同小异*  
```java
Specification<InfoCommentENT> specification = (root, criteriaQuery, cb) -> {
  // 通过Predicate对象来构建SQL语句
  Predicate predicate = cb.conjunction();
  /**
   * predicate.getExpressions()代表获取到当前动态SQL中的所有构成SQL的片段,该方法返回的是一个List集合,接着调用add方法表明添加筛选条件
   * cb.equal()代表其中一个筛选条件是SQL中的=等于条件
   * root.get("id")和上面的解释一致代表当前等于条件的查询字段是id,后面的1代表匹配值为1的记录
   * 最终将predicate对象返回即可
   */
  predicate.getExpressions().add(cb.equal(root.get("id"), 1));
}
```

4.缺陷  
* 不支持分组、聚合函数

#### 2.3.5 QueryDSL
1.介绍  
QueryDSL是基于ORM框架或SQL平台上的一个`通用查询框架`.借助QueryDSL可以在任何支持的ORM框架或SQL平台上`以通用API方式构建查询`  
JPA是QueryDSL的主要集成技术,是JPQL和Criteria查询的代替方法.目前QueryDSL支持的平台包括**JPA,JDO,SQL,Mongodb**等等  

2.加入QueryDSL相关的依赖  
```xml
<querydsl.version>4.4.0</querydsl.version>
<apt.version>1.1.3</apt.version>

<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>${querydsl.version}</version>
</dependency>
<!--添加maven插件-->
<!--这个插件是为了让程序自动生成query type(查询实体,命名方式为:"Q"+对应实体名)-->
<build>
    <plugins>
        <plugin>
            <groupId>com.mysema.maven</groupId>
            <artifactId>apt‐maven‐plugin</artifactId>
            <version>${apt.version}</version>
            <dependencies>
                <dependency>
                    <groupId>com.querydsl</groupId>
                    <artifactId>querydsl‐apt</artifactId>
                    <version>${querydsl.version}</version>
                </dependency>
            </dependencies>
            <executions>
                <execution>
                    <phase>generate‐sources</phase>
                    <goals>
                        <goal>process</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>target/generated‐sources/queries</outputDirectory>
                        <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                        <logOnlyOnError>true</logOnlyOnError>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

*提示:springboot项目引入插件可能报错,所以直接引入apt包就可以了*

3.编写测试类  
*提示:这里没跑起来,依赖导入不进来;暂时跳过,而且也不是很常用*

3.QuerydslPredicateExecutor  
* `findOne(Predicate) return Optional<T>`
* `findAll(Predicate) return Iterable<T>`
* `findAll(Predicate,Sort) return Iterable<T>`
* `findAll(Predicate,OrderSpecifier<?>...) return Iterable<T>`
* `findAll(OrderSpecifier<?>...) return Iterable<T>`
* `findAll(Predicate,Pageable) return Page<T>`
* `count(Predicate) return long`
* `exists(Predicate) return boolean`


### 2.4 JPA注解(表/属性)  
`@Entity`  
用于添加在实体类上,定义该JAVA类成为被JPA管理的实体,将映射到指定的数据库表.如定义一个实体类Category,它将映射到数据库中的category表中  

`@Table`  
指定数据库的表名,与`@Entity`一同使用;添加在实体类上
* name:指定表的名称;如果不写默认表名为类名
* catalog,schema:用于设置目录和此表所在的schema(模式),一般不需要填写
* uniqueConstraints:唯一性约束;只有创建表的时候有用
* indexes:索引;只有创建表的时候需要,默认不需要

`@Id`  
定义属性为数据库表中的主键列,一个实体里面必须有一个;添加在属性上

`GeneratedValue`  
主键生成策略;添加在被@ID注解修饰的属性上  
* strategy:id的生成策略
  * GenerationType.TABLE:通过表产生主键,框架由表模拟序列产生主键,使用该策略可以使应用更易于数据库移植
  * GenerationType.SEQUENCE:通过序列产生主键;通过`@SequenceGenerator`注解指定序列名;MySql不支持这种方式
  * GenerationType.IDENTITY:数据库id自增长,多用于MySql
  * GenerationType.AUTO:JPA自动选择合适的策略,默认选项
* generator:通过Sequence生成id,常见Orcale数据库id生成,需要配合`@SequenceGenerator`使用

`@Column`  
定义该属性对应数据库中的列名  
* name:数据库中的列名,如果不写默认和实体属性名一致
* unique:是否唯一,默认为false
* nullable:是否允许为空,默认为false
* insertable:执行insert操作的时候是否包含此字段,默认为true
* updatable:执行updatable操作时是否包含次字段,默认为true
* columnDefinition:表示该字段在数据库中的实际类型
* length:数据库字段的长度,默认为255
* table:
* precision:
* scale:

*提示:这里面有些字段是用于生成数据库表时使用的*  

`IdClass`  
联合主键;通过外部类来实现联合主键  
```java
@Data
@Entity
@IdClass(value = UnionKey.class)
public class IdClassDemo{

  @Id
  private String idOne;

  @Id
  private String idTwo;

  private String context;
}
```
假设有上述这样的实体类,它由idOne和idTwo这两个键构成联合主键,除了在这两个属性上都标注@Id注解之外,还需要在类上标注一个@IdClass注解,并且该注解内的value属性指定了一个外部类;看看这个外部类的定义结构  

```java
@Getter
@Setter
@NoArgsConstructor
@EqualsAndHashCode
public class UnionKey implements Serializeable {
  private String idOne;

  private String idTwo;
}
```
UnionKey类必须实现Serializeable接口,必须有默认的无参构造方法,必须重写equals和hashcode方法  

**使用方法:**  
在Repository接口的编写时还需要注意,Repository接口要把主键改为外部主键类  
```java
public interface IdClassRepository extends JpaRepository<IdClassDemo,UnionKey> {

}
```

`@Basic`  
表示属性是到数据库表字段的映射,如果实体的字段上没有任何注解,默认即为@Basic
* fetch:抓取方式,默认FetchType.EAGER(立即加载),FetchType.LAZY(延迟加载,主要应用在大字段上面)
* optional:设置这个字段是否可以为null,默认是true

`@Transient`  
表示该属性并非是一个到数据库表的字段的映射,<font color="#00FF00">与@Basic作用相反</font>.JPA映射数据库的时候忽略它  

`@Temporal`  
用来设置Date类型的属性映射到对应精度的字段;标注在属性上  
* TemporalType.DATE 映射为数据库date
* TemporalType.TIME 映射为数据库time
* TemporalType.TIMESTAMP 映射为datetime

`@Enumerated`  
映射枚举类  
* value
  * EnumType.ORDINAL 枚举字段的下标
  * EnumType.STRING 枚举字段的Name

`@Lob`  
将属性字段映射成数据库支持的大对象类型,支持以下两种数据库类型的字段  
* Clob(Character Large Objects)类型:长字符串类型,java.sql.Clob、Character[]、char[]和String将被映射成Clob类型
* Blob(Binary Large Objects)类型:字节类型,java.sql.Blob、Byte[]、byte[]和实现了Serializable接口的类型将被映射为Blob类型

*注意:由于Clob、Blob占用内存空间较大,一般配合@Basic(fetch=FetchType.LAZY)将其设置为延迟加载*

`@NamedQuery`  

`@NamedNativeQuery`  

`@MappedSuperClass`  
可以把一些公共的属性提取到添加该注解的类里,如id,creteTime,updateTime等.该类不会生成表,该类中相应的字段都会生成在子类中.该类没有被@Entity注释,不是一个实体  

`@Inheritance`  
控制实体之间的继承关系 

### 2.5 JPA注解(关联)
*提示:spring-data-jpa本身并没有提供关联表支持,这里的关联完全是由hibernate实现JPA规范从而得到支持的*  
**目录:**  
2.5.1 基本环境搭建  
2.5.2 一对一关系  
2.5.3 一对多关系  
2.5.4 多对一关系  
2.5.5 多对多关系  

#### 2.5.1 基本环境搭建
1.spring-data-jpa关联特性  
* 在增加/删除/查询/修改的时候可以同时对关联的表进行关联操作

2.新建模块spring-data-jpa-relation  
新建一个spring-data-jpa-relation模块作为本次的测试,修改pom文件的内容如下  
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
    </dependency>
    <!--由于spring-data-jpa只是基于JPA框架的封装,所以这里需要为spring-data-jpa指定当前使用的底层jpa实现框架是什么-->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.5.0.Final</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
    </dependency>
    <!--连接池-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.8</version>
    </dependency>
    <!--spring-test-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>5.3.11</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.22</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

3.编写config配置文件  
参考2.1 Spring Data JPA基本环境搭建=>5.编写config配置文件

#### 2.5.2 一对一关系
1.注解介绍  
`@OneToOne`  
标注在实体类的属性上,并且该属性也是一个实体(可以看下面的例子)
关联分为两种:
* **单向关联:** 如果只有一方引用另一方则称为单向关联  
* **双向关联:** 如果双方互相引用则称为双向关联  
- - -
* targetEntity:关联的目标实体类,默认字段或属性的类型
* cascade:级联操作策略,默认情况下没有级联操作(配置枚举类)
  * ALL:所有操作关联修改
  * PERSIST:新增操作关联修改
  * MERGE:修改操作关联修改
  * REMOVE:删除操作关联修改
  * REFRESH:刷新操作关联修改
  * DETACH:
* fetch:数据获取方式,默认EAGER,立即查询
  * EAGER:立即查询
  * LAZY:懒加载
* optional:是否允许关联对象对null,默认为true
* mappedBy:拥有关系的字段,此元素仅在关联的反(非拥有)端指定,关联关系被谁维护
  * mappedBy不能与@JoinColumn、@JoinTable同时使用
  * mappedBy的值是另一方实体属性的名称
* orphanRemoval:是否级联删除,和CascadeType.REMOVE效果一样,这两个配置只要配置其中一个就会生效;默认false;通常在修改的时候会用到

`@JoinColumn`  
用于指定连接实体关联或元素集合的列  
* name:外键列的名称
* table:外键所在的表名
* referencedColumnName:这个外键列引用的列的名称
* unique:是否唯一,默认false
* nullable:能否为null,默认为true
* insertable:是否跟随一起新增,默认为true
* updatable:是否跟随一起修改,默认为true
* columnDefinition:DDL SQL片段
* foreignKey:于在表生成时指定或控制外键约束的生成.如果未指定此元素,则将应用持久性提供程序的默认外键策略.一般默认即可

*提示:@OneToOne配合@JoinColumn一起使用,可以单项关联也可以双向关联,是具体情况而定.双向一对一决定哪一方来管理外键,通常使用常用的一方来管理*

2.创建pojo实体类  
创建Customer和Account类作为本次一对一关系的实验  
```java
@Data
@Entity
@Table(name = "tb_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long customerId;

    @Column(name = "customer_name")
    private String customerName;

    @Column(name = "customer_address")
    private String customerAddress;

    /**
     * 单向关联,一对一<br>
     * 通过@JoinColumn设置外键ID,即当前Customer表会有一个account_id字段外键关联Account表的id字段
     */
    @OneToOne
    @JoinColumn(name = "account_id")
    private Account account;

}
```

```java
@Data
@Entity
@Table(name = "tb_account")
public class Account {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "account_id")
    private Long id;

    @Column(name = "account_user_name")
    private String userName;

    @Column(name = "account_password")
    private String password;
}
```

*这里只有Customer关联Account,所以这里是**单向关联***  

3.编写repository类  
```java
public interface CustomerRepository extends PagingAndSortingRepository<Customer, Long> {
}
```

4.修改Customer表配置关联操作  
如果这里直接测试运行添加Customer对象则会报错,提示没有添加关联操作;修改Customer如下  
```java
@Data
@Entity
@Table(name = "tb_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long customerId;

    @Column(name = "customer_name")
    private String customerName;

    @Column(name = "customer_address")
    private String customerAddress;

    // 主要修改这里,所有操作都会关联修改
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "account_id")
    private Account account;

}

```

5.编写测试类  
```java
@ContextConfiguration(classes = SpringDataJPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class OneToOneTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Test
    public void testC() {
        // 初始化数据
        Account account = new Account();
        account.setUserName("caixukun");
        Customer customer = new Customer();
        customer.setCustomerName("蔡徐坤");
        customer.setAccount(account);

        customerRepository.save(customer);
    }

    @Test
    public void testR() {
        // 输出Optional[Customer(customerId=1, customerName=蔡徐坤, customerAddress=null, account=Account(id=1, userName=caixukun, password=null))]
        System.out.println(customerRepository.findById(1L));
    }

}
```

查看结果,可以看到两条数据都插入到数据库中了  
![查看结果](resources/JPA/10.png)  

6.测试`fetch = FetchType.LAZY`属性  

修改Customer表中Account属性如下  
```java
@OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
@JoinColumn(name = "account_id")
private Account account;
```

**编写测试方法:**  
```java
@Test
@Transactional
public void testLazy() {
    Optional<Customer> customer = customerRepository.findById(1L);
    System.out.println("=============");
    System.out.println(customer.get()); // 调用toString方法时会调用打印account对象,此时才会去查询account对象
}
```
该方法的执行顺序是先查询Customer对象,然后打印=============;最后查询Account对象,而不是一开始就查询Customer+Account对象  
所以懒加载就是在关联关系中,当你用到了关联对象时才去查询而不是立即去查询,从而提高性能  
*提示:可以看到这里配置了@Transactional,这是因为当repository调用完方法之后session就会关闭,一旦session关闭之后就不能进行查询,所以这里需要使用事务让session直到事务关闭了之后再关闭session*  

7.双向关联一对一  
**不足:**  
现在Customer和Account是一对一的关系,之前已经尝试过通过级联删除在删除Customer对象的同时删除Account对象,假设现在需要在删除Account对象的同时删除Customer对象就需要使用双向关联.  
假设现在也在Account对象中关联一个@OneToOne以及@JoinColumn注解指定外键;则由于MySQL外键的特性,当要删除account表中的一条数据,由于该数据被customer表关联了所以必须先删除customer表中的数据再删除account表中的数据,假设现在account表和customer表互相关联,则最终的结果就是<font color="#00FF00">这两张表的数据全都删不掉</font>.
```java
@Data
@Entity
@Table(name = "tb_account")
public class Account {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "account_id")
    private Long id;

    @Column(name = "account_user_name")
    private String userName;

    @Column(name = "account_password")
    private String password;

    @OneToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```
但实际上在spring-data-jpa是可以实现双向删除的,具体的原理就是spring-data-jpa在删除某条数据之前先将**关联记录的外键**设置为一个不存在的外键,然后在把当前记录删除之后再删除关联的记录

另外一种方式是某一方放弃外键关联,使用@OneToOne注解的mappedBy属性来取消关联;修改Customer对象的内容如下  
```java
@Data
@Entity
@Table(name = "tb_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long customerId;

    @Column(name = "customer_name")
    private String customerName;

    @Column(name = "customer_address")
    private String customerAddress;

    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY, mappedBy = "customer")
    // @JoinColumn(name = "account_id")
    private Account account;
}
```

*注意:mappedBy指定为目标对象中关联当前对象的属性名称,例如这里就是customer,另外这里就没必要使用@JoinColumn注解了*    

#### 2.5.3 一对多关系
1.注解详解  
`@OneToMany`  
指定一个具有一对多的多值关联
*提示:如果使用泛型定义集合以指定元素类型,则不需要指定关联的目标实体类型;否则,必须指定目标实体类*  
*提示:如果关系是双向的,则必须使用mappedBy元素指定关系的所有者实体的关系字段或属性*

* targetEntity:关联的目标实体类
  * 只有使用java泛型定义集合时,才是可选的;否则必须指定
  * 使用泛型定义时,默认为集合的参数化类型
* cascade:级联操作策略;默认情况下没有级联操作
* fetch:数据获取方式,默认LAZY,延迟加载`(与一对一、多对一不同)`
  * EAGER:立即查询
  * LAZY:懒加载
* mappedBy:拥有关系的字段.除非关系是单向的,否则是必需的
* orphanRemoval: 是否级联删除,和CascadeType.REMOVE效果一样,只要配置其中一种就会级联删除.默认false,指定集合中的一个元素中集合中移除,是否从数据库中删除

*提示:@OneToMany单独使用建立单项一对多关系时,如果不配合@JoinColumn使用,会额外产生一张表来维护关联关系.配合@JoinColumn使用时,外键会生成在目标表中*  
*提示:这个注解修饰的集合元素必须是被@Entity修饰的JPA类.如果集合的元素是基本类型,则需要使用内嵌表示@ElementCollection*

`@OrderBy`  
指定关联查询时的排序,一般和`@OneToMany`一起使用

2.新增pojo类  
```java
@Entity
@Table(name = "tb_message")
@Data
public class Message {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "message_id")
    private Long id;

    @Column(name = "message_info")
    private String info;

}
```

3.修改Customer类的定义  
*提示:现在的需求是一个Customer对应多个Message对象,那么就需要在**一**的这一方即Customer这一方添加关联关系*  
```java
@Data
@Entity
@Table(name = "tb_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long customerId;

    @Column(name = "customer_name")
    private String customerName;

    @Column(name = "customer_address")
    private String customerAddress;

    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY, mappedBy = "customer")
    @JoinColumn(name = "account_id")
    private Account account;

    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "customer_id")
    private List<Message> messages;

}
```

*注意:这里和一对一就体现出区别了,一对一声明的@JoinColumn注解中指定的属性account_id是声明在当前tb_customer这张表下的,而在一对多中声明的属性customer_id是添加到tb_message这张表下面的.这是因为在一对多关系中,外键很显然是放在**多**的这一端的*  

4.编写测试类  
```java
@ContextConfiguration(classes = SpringDataJPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class OneToManyTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Test
    public void testC() {
        List<Message> messages = new ArrayList<>();
        Message message0 = new Message();
        message0.setInfo("鸡你");
        Message message1 = new Message();
        message1.setInfo("太美");
        messages.add(message0);
        messages.add(message1);
        Customer customer = new Customer();
        customer.setCustomerName("丁真");
        customer.setMessages(messages);
        customerRepository.save(customer);
    }

}
```
**执行结果如下:**  
![一对多](resources/JPA/11.png)  

5.更多示例  
```java
@Test
@Transactional(readOnly = true)
public void testR() {
    /*
    该方法最终执行的效果是先打印======
    在执行SQL,这是因为在一对多的关系中默认就是使用懒加载
    所以这也是为什么该方法需要使用@Transactional注解,这和之前2.5.2 一对一关系
    =>6.测试`fetch = FetchType.LAZY` 时的原因是一样的,都是因为
    customerRepository执行完后就关闭session导致的
    */
    // 可以通过fetch修改为立即加载
    Optional<Customer> result = customerRepository.findById(1L);
    System.out.println("=========");
    System.out.println(result);
}
```

#### 2.5.4 多对一关系
1.注解详情  
`ManyToOne`  
指定与另一个具有多对一的实体类的单值关联  
* targetEntity:关联的目标实体类,默认字段或属性的类型
* cascade:级联操作策略,默认情况下没有级联操作
* fetch:数据获取方式,是懒加载还是立即加载;默认为立即加载
* optional:是否允许为空,默认为true

2.修改Message对象的定义  
```java
@Data
@Entity
@Table(name = "tb_message")
public class Message {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "message_id")
    private Long id;

    @Column(name = "message_info")
    private String info;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;

}
```
这里使用了多对一注解,<font color="#FF00FF">所以如何判断是一对多还是多对一取决于那一方去维护</font>,而且这里message和customer之间的关系是确定的,即一个customer对应多个message;如果在customer中维护就是一对多,如果在message中维护就是多对一.  

3.创建MessageRepository  
```java
public interface MessageRepository extends PagingAndSortingRepository<Message, Long> {
    /**
     * 通过约定方法来实现关联查询,需要通过关联属性进行匹配
     * 但是只能通过id进行匹配
     */
    List<Message> findByCustomer(Customer customer);
}
```

4.创建测试类  
```java
@ContextConfiguration(classes = SpringDataJPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class ManyToOneTest {

    private MessageRepository messageRepository;

    @Test
    public void testC() {
        Customer customer = new Customer();
        customer.setCustomerName("马嘉祺");
        List<Message> list = new ArrayList<>();
        Message message0 = new Message();
        message0.setInfo("小");
        message0.setCustomer(customer);
        Message message1 = new Message();
        message1.setInfo("气球");
        message1.setCustomer(customer);
        list.add(message0);
        list.add(message1);
        customer.setMessages(list);
        messageRepository.saveAll(list);
    }

    @Test
    public void testR() {
        // 这里设置setCustomerName方法是没用的,因为findByCustomer只会通过customerId来进行查询
        Customer customer = new Customer();
        customer.setCustomerId(1L);
        customer.setCustomerName("xxxx");
        List<Message> messages = messageRepository.findByCustomer(customer);
        // 这里可能会报错留个心眼,循环调用toString
        System.out.println(messages);
    }
}
```
<font color="#00FF00">在真实的业务场景中通过多端插入多条数据是更合理的</font>

#### 2.5.5 多对多关系
1.注解详解  
`ManyToMany`  
指定具有多对多多重性的多值关联  
* targetEntity:关联的目标实体类,只有使用java泛型定义集合时,才是可选的.否则必须指定,使用泛型定义时,默认为集合的参数化类型
* cascade:级联操作策略
* fetch:延迟加载设置;默认是懒加载
* mappedBy:拥有关系的字段;除非关系是单向的,否则是必须的

`@JoinTable`  
指定关联的中间表配置,如果没有@JoinTable注解,则使用注解元素默认值  
<font color="#00FF00">在多对多关系中必然要使用中间表,所以@JoinColumn注解不起作用,需要替换为@JoinTable注解进行使用</font>  
* name:连接表名称.(默认:两个关联的主实体表的连接名称,用下划线分隔,如:table_b_table_b)
* catalog:表的目录;默认为默认目录.一般默认即可.
* schema:表的schema.默认为用户的默认schema.一般默认即可.
* joinColumns:连接表的外键列,它引用拥有关联的实体的主表.
* inverseJoinColumns:连接表的外键列,它引用不拥有关联的实体的主表.(即相反的一面).
* foreignKey::用于在表生成时为joinColumns元素对应的列指定或控制外键约束的生成.* 一般默认即可.
* inverseForeignKey:于指定或控制在表生成时对应于inverseJoinColumns元素的列的* 外键约束的生成.一般默认即可.
* uniqueConstraints:要放在表上的唯一约束.只有在表生成时才使用这些方法.默认没有* 附加约束.一般默认即可.
* indexes:表的索引.只有在表生成时才使用这些方法.一般默认即可.

*提示:@ManyToMany一般和@JoinTable一起使用*
<font color="#00FF00">多对多也分为单向多对多和双向多对对</font>  

2.需求介绍  
现在模拟多对多的关联关系,由customer、role、customer_role这三张表构成,分别是用户、角色、用户角色表  

3.编写pojo类  
```java
@Data
@Entity
@Table(name = "tb_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long customerId;

    @Column(name = "customer_name")
    private String customerName;

    @Column(name = "customer_address")
    private String customerAddress;

    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY, mappedBy = "customer")
    @JoinColumn(name = "account_id")
    private Account account;

    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "customer_id")
    private List<Message> messages;

    // 这里是单向多对多
    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(
            name = "tb_customer_role",
            joinColumns = {@JoinColumn(name = "c_id")},
            inverseJoinColumns = {@JoinColumn(name = "r_id")}
    )
    private List<Role> roles;

}
```

```java
@Data
@Entity
@Table(name = "tb_role")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "role_name")
    private String rName;

}
```

4.编写测试类  
```java
@ContextConfiguration(classes = SpringDataJPAConfig.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class ManyToManyTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Test
    public void testC() {
        List<Role> roles = new ArrayList<>();
        Role role0 = new Role();
        role0.setRName("超级管理员");
        Role role1 = new Role();
        role1.setRName("商品管理员");
        Customer customer = new Customer();
        roles.add(role0);
        roles.add(role1);
        customer.setRoles(roles);

        customerRepository.save(customer);
    }

}

```

![运行结果](resources/JPA/12.png)  

5.更多示例  
```java
@Test
// 同样多对多是懒加载
@Transactional(readOnly = true)
public void testR() {
    // 该方法输出结果为:Optional[Customer(customerId=10, customerName=null, customerAddress=null, account=null, messages=[], roles=[Role(id=1, rName=超级管理员), Role(id=2, rName=商品管理员)])]
    System.out.println(customerRepository.findById(10L));
}

@Test
@Transactional
// 需要添加@Commit注解,这是spring单元测试中的注解,正常springboot项目是不需要使用该注解的
@Commit
public void testD() {
    Optional<Customer> customer = customerRepository.findById(10L);
    customerRepository.delete(customer.get());
}

```

6.双向多对多  
修改
```java
@Data
@Entity
@Table(name = "tb_role")
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "role_name")
    private String rName;
    /*
    由于已经在Customer实体对象中指定了@JoinTable注解了
    所以这里就没必要再添加了,双向多对多只需要一方指定就可以了
    */
    @ManyToMany(cascade = CascadeType.ALL)
    private List<Role> roles;
}
```

### 2.6 乐观锁
1.介绍  
乐观锁同样是hibernate中提供的支持  
有机会可以学习一下hibernate  
乐观锁的概念可以学习一下  

2.上手体验  
想要使用乐观锁在hibernate中十分方便  
在customer类中加入以下字段即可  
```java
private @Version Long version;
```

### 2.7 审计
1.什么是审计  
在阿里巴巴开发规范手册中提到过,每张表都需要加四个字段:创建人、创建时间、修改时间、修改人  
这样可以方便追溯以及一些记录  
所以每次创建更新字段时就需要频繁修改这些值,spring-data-jpa为这一功能提供了支持

2.修改pojo类  
```java
@Data
@Entity
@Table(name = "tb_customer")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long customerId;

    @Column(name = "customer_name")
    private String customerName;

    @Column(name = "customer_address")
    private String customerAddress;

    /**
     * 单向关联,一对一<br>
     * 通过@JoinColumn设置外键ID,即当前Customer表会有一个account_id字段外键关联Account表的id字段
     */
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY, mappedBy = "customer")
    @JoinColumn(name = "account_id")
    private Account account;

    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "customer_id")
    private List<Message> messages;


    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(
            name = "tb_customer_role",
            joinColumns = {@JoinColumn(name = "c_id")},
            inverseJoinColumns = {@JoinColumn(name = "r_id")}
    )
    private List<Role> roles;

    @CreatedBy
    private String createBy;

    @LastModifiedBy
    private String modifiedBy;

    @Temporal(TemporalType.TIMESTAMP)
    @CreatedDate
    protected Date dateCreate = new Date();

    @Temporal(TemporalType.TIMESTAMP)
    @LastModifiedDate
    protected Date dateModified = new Date();
}
```

3.编写AuditorAwareBean  
在SpringDataJPAConfig类中往容器中添加Bean  
```java
@Bean
public AuditorAware<String> auditorAware() {
    /*
    该方法返回的泛型要和实体类中@CreatedBy指定的泛型匹配
    相当于被@CreatedBy注解标注的实体类属性,在插入前会调用该方法获取插入的值
    调用的时候只会调用泛型匹配的方法
    */
    return () -> Optional.of("zhangsan");
}
```

*注意泛型需要匹配*  

4.在Customer类上添加`@EntityListeners(AuditingEntityListener.class)`注解  
这是固定写法  
```java
@Data
@Entity
@Table(name = "tb_customer")
@EntityListeners(AuditingEntityListener.class)
public class Customer {
}
```

5.添加`@EnableJpaAuditing`注解  
在SpringDataJPAConfig类上添加`@EnableJpaAuditing`注解
```java
@Configuration
@EnableJpaRepositories(basePackages = "com.cnsukidayo.jpa.repository")
@EnableTransactionManagement
@EnableJpaAuditing
public class SpringDataJPAConfig {
}
```

6.添加依赖  
这里需要添加spring-aspects依赖否则会报错
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.3.10</version>
</dependency>
```

7.运行测试代码  
```java
@Test
public void testC() {
    List<Message> messages = new ArrayList<>();
    Message message0 = new Message();
    message0.setInfo("鸡你");
    Message message1 = new Message();
    message1.setInfo("太美");
    messages.add(message0);
    messages.add(message1);
    Customer customer = new Customer();
    customer.setCustomerName("丁真");
    customer.setMessages(messages);
    customerRepository.save(customer);
}
```
最终的运行结果如下  
![最终运行结果](resources/JPA/13.png)  


## 3.底层原理
**目录:**  
3.1 Repository底层原理  
3.2 手写JPA实现  
3.3 Spring整合JPA的原理  

### 3.1 Repository底层原理  
1.动态代理  
Repository的实现原理实际上就是JDK的动态代理;像是PagingAndSortingRepository、QueryByExampleExecutor、JpaSpecificationExecutor这些接口都有一个默认的实现`SimpleJpaRepository`,也就是说普通的CRUD方法实际上调用的是SimpleJpaRepository实现类的实现方法  

2.手写一个JPA实现  
首先有CustomerRepository抽象类定义如下,这是使用层面使用的类  
```java
public interface CustomerRepository extends PagingAndSortingRepository<Customer, Long> {
}
```

接着有我们自已的实现类,内容如下  
```java
public class MyJPARepository implements PagingAndSortingRepository {
    private EntityManager entityManager;
    private Class aClass;
    public MyJPARepository(EntityManager entityManager, Class aClass) {
        this.entityManager = entityManager;
        this.aClass = aClass;
    }

    @Override
    public Optional findById(Object id) {
        return Optional.ofNullable(entityManager.find(aClass, id));
    }

}
```

**解释:**  
我们的类实现了PagingAndSortingRepository接口,并且为接口的findById提供了一个默认的实现,于是乎之后我们就可以使用JDK的动态代理,根据被代理类调用的方法名称来调用代理类的真实方法  

动态代理类MyJPAProxy:  
```java
public class MyJPAProxy implements InvocationHandler {

    private EntityManager entityManager;
    private Class aClass;

    public MyJPAProxy(EntityManager entityManager, Class aClass) {
        this.entityManager = entityManager;
        this.aClass = aClass;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        MyJPARepository myJPAProxy = new MyJPARepository(entityManager, aClass);
        Method proxyMethod = myJPAProxy.getClass().getMethod(method.getName(), method.getParameterTypes());
        return proxyMethod.invoke(myJPAProxy, args);
    }
}
```

测试类:  
```java
public class JPAProxyTest {

    @Test
    public void testP() throws ClassNotFoundException {
        // 创建Spring容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(SpringDataJPAConfig.class);
        // 从容器中获取获取LocalContainerEntityManagerFactoryBean然后得到EntityManager
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = annotationConfigApplicationContext.getBean(LocalContainerEntityManagerFactoryBean.class);
        EntityManager entityManager = entityManagerFactoryBean.createNativeEntityManager(null);
        // 通过反射获取当前接口泛型中指定的泛型类型
        ParameterizedType parameterizedType = (ParameterizedType) CustomerRepository.class.getGenericInterfaces()[0];
        Type type = parameterizedType.getActualTypeArguments()[0];
        Class<?> aClass = Class.forName(type.getTypeName());
        CustomerRepository customerRepository = (CustomerRepository) Proxy.newProxyInstance(
                CustomerRepository.class.getClassLoader(),
                new Class[]{CustomerRepository.class},
                new MyJPAProxy(entityManager, aClass)
        );
        Optional<Customer> customer = customerRepository.findById(10L);
        System.out.println("输出结果:" + customer);
    }

}
```

输出结果:Optional[Customer(customerId=10, customerName=null, customerAddress=null, account=null, messages=[], roles=[Role(id=1, rName=超级管理员), Role(id=2, rName=商品管理员)], createBy=null, modifiedBy=null, dateCreate=null, dateModified=null)]

3.看源码  
*提示:看源码讲究先看主线、整体;之后再追究细节;读取源码要把自已想象成写这段代码的程序员,如果是你你会怎么做?*  

3.1 当调用repository接口的方法时最先会来到`JdkDynamicAopProxy`这个类是AOP的统一处理类,这里会执行该类的`invoke`方法  
至于为什么JdkDynamicAopProxy会代理repository接口,之后再说  

3.2 获取目标类  
之后会执行到`target = targetSource.getTarget();`这段代码,这段代码会获取到当前实现repository接口的目标实现类,也就是`SimpleJpaRepository`类  
![目标类](resources/JPA/14.png)  

3.3 SimpleJpaRepository实现  
![实现](resources/JPA/15.png)  
最终它会调用`SimpleJpaRepository`类的实现方法,而SimpleJpaRepository类里的这些实现方法都是通过`EntityManager`来实现的  
所以结论就是JPA的底层实现是通过<font color="#FF00FF">JDK动态代理+EntityManager进行实现的</font>

### 3.2 手写JPA实现
1.问题  
* Spring是如何创建Repository这个Bean的
* Spring怎么将动态代理创建为Bean的

2.Spring是如何创建Repository这个Bean的  
2.1 `@EnableJpaRepositories(basePackages = "com.cnsukidayo.jpa.repository")`指定了该注解  
2.2 Spring IOC容器启动的时候IOC容器加载,根据"com.cnsukidayo.jpa.repository"去创建Bean

3.如何注册Bean接口  

3.1 BeanDefinition接口,实际上通过xml方式配置Spring,xml中的每一个&lt;bean&gt;标签都会映射为一个BeanDefinition实例,可以通过查看`AbstractBeanDefinition`来获取更多信息,AbstractBeanDefinition类中就定义了很多属性来对应&lt;bean&gt;标签可以指定的属性,例如懒加载、作用域等属性

3.2 通过AbstractBeanDefinition(BeanDefinition)就可以读取到&lt;bean&gt;中定义的beanClass对象,接着通过beanClass对象的newInstance()方法,通过反射的方式来创建Bean对象  

3.3 那么BeanDefinition又是如何创建的,BeanDefinition实际上是通过BeanDefinitionReader来创建的,因为不同的配置方式需要不同的BeanDefinitionReader实现来读取,比如读取xml的实现是`XmlBeanDefinitionReader`,而读取注解的实现是`AnnotatedGenericBeanDefinition`  

所以在`AnnotationConfigApplicationContext`的构造方法中它就创建了一个`AnnotatedBeanDefinitionReader`  
![AnnotatedBeanDefinitionReader](resources/JPA/16.png)  

3.4 读取到配置之后,下一步就需要进行解析;也就是把读取到的xml中的&lt;bean&gt;或者&lt;component-scan&gt;注解指定的Bean解析为BeanDefinition  
亦或是把@Bean、@Import、@ComponentScan注解指定的的Bean解析为BeanDefinition  
<font color="#00FF00">把BeanDefinitionReader读取到的配置进行解析实际上是通过ConfigurationClassParser实现的</font>  
ConfigurationClassParser解析器会解析@Bean、@Import、@ComponentScan注解,对于@ComponentScan注解,它又会让ComponentScanParser解析类去进行解析

![解析](resources/JPA/17.png)

ComponentScanParser在解析之前肯定需要先将@ComponentScan注解指定的Bean扫描(或者读取到),所以又要通过ClassPathBeanDefinitionScanner进行扫描  

接着它会扫描被@Component注解修饰的类,并判断该类是不是抽象类或者接口,如果是则不会创建该Bean(因为无法实例化);否则就会把类解析为BeanDefinition(也就是调用下图中的registerBeanDefinition()方法将BeanDefinition进行注册)  
<font color="#00FF00">所以结论就是ComponentScanParser不会解析被扫描到的接口</font>  
![获取候选组件](resources/JPA/18.png)
  
3.5 整体流程图  
![整体流程图](resources/JPA/19.png)  

4.自定义扫描器  

4.1 findCandidateComponents
在上述3.4步最后一张图中被红色方框圈出的代码,它调用了findCandidateComponents方法,由该方法来获取候选组件,进入该方法它会筛选掉当前是抽象类或者接口的类  
![抽象类](resources/JPA/20.png)

4.2 重写ClassPathBeanDefinitionScanner
通过重写ClassPathBeanDefinitionScanner中的isCandidateComponent方法将Repository扫描到容器中(重写扫描时的逻辑,原有的逻辑是会跳过接口或抽象类)  
还是在spring-data-relation模块下创建com.cnsukidayo.jpa.scanner包,然后在该包下创建一个JPAClassPathBeanDefinitionScanner
```java
public class JPAClassPathBeanDefinitionScanner extends ClassPathBeanDefinitionScanner {
    public JPAClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry) {
        super(registry);
    }

    @Override
    protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
        // 实际上这里严格来说要判断当前接口是不是扩展了Repository接口
        AnnotationMetadata metadata = beanDefinition.getMetadata();
        return metadata.isInterface();
    }
}
```

4.3 将自定义扫描器JPAClassPathBeanDefinitionScanner动态注册到Spring中,还是在com.cnsukidayo.jpa.scanner包下创建JPABeanDefinitionRegistryPostProcessor类,通过该类来动态注册扫描器,不影响原有的扫描逻辑  
BeanDefinitionRegistryPostProcessor:是后置处理器,所以它的生命周期是比较靠后的  
```java
@Component
public class JPABeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        // 动态注册扫描器
        JPAClassPathBeanDefinitionScanner jpaClassPathBeanDefinitionScanner = new JPAClassPathBeanDefinitionScanner(registry);
        // 必须实现repository接口
        jpaClassPathBeanDefinitionScanner.addIncludeFilter(new AssignableTypeFilter(Repository.class));
        // 扫描路径
        jpaClassPathBeanDefinitionScanner.scan("com.cnsukidayo.jpa.repository");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

    }
}
```

4.4 运行测试  
首先注释掉SpringDataJPAConfig类上面的@EnableJpaRepositories注解,先不让Spring的自动配置生效  
接着在SpringDataJPAConfig上添加注解`@ComponentScan(basePackages = "com.cnsukidayo.jpa.scanner")`把自定义扫描器添加到Spring容器中  
```java
    @Test
    public void testS() throws ClassNotFoundException {
        // 创建Spring容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(SpringDataJPAConfig.class);
        CustomerRepository customerRepository = annotationConfigApplicationContext.getBean(CustomerRepository.class);
        Optional<Customer> customer = customerRepository.findById(1L);

    }
```
运行结果:
`nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.cnsukidayo.jpa.repository.CustomerRepository]: Specified class is an interface
`
**问题:**  
现在的问题是,虽然已经把接口扫描到容器中,但现在还无法实例化;因为接口无法实例化,所以根据之前的思路使用动态代理,那又如何将动态代理创建为Bean?  

*小结一下流程图:*  
![流程图](resource/JPA/21.png)

4.5 通过FactoryBean将动态代理创建为Bean  
```java
@Component
public class JPAFactoryBean implements FactoryBean {

    @Autowired
    private LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBean;

    private Class<?> repositoryInterface;

    public JPAFactoryBean(Class<?> repositoryInterface) {
        this.repositoryInterface = repositoryInterface;
    }

    /**
     * 随意控制实例化过程
     */
    @Override
    public Object getObject() throws Exception {
        // 这段代码就是之前动态代理的代码
        EntityManager entityManager = localContainerEntityManagerFactoryBean.createNativeEntityManager(null);
        ParameterizedType parameterizedType = (ParameterizedType) CustomerRepository.class.getGenericInterfaces()[0];
        Type type = parameterizedType.getActualTypeArguments()[0];
        Class<?> aClass = Class.forName(type.getTypeName());
        return Proxy.newProxyInstance(
                CustomerRepository.class.getClassLoader(),
                new Class[]{CustomerRepository.class},
                new MyJPAProxy(entityManager, aClass)
        );
    }

    /**
     * 该方法的返回值,也就是Spring在实例化一个Bean之前,会先判断该Bean
     * 的类型与该方法返回的类型是否匹配,如果匹配的话就调用getObject()方法
     */
    @Override
    public Class<?> getObjectType() {
        return repositoryInterface;
    }
}
```

修改JPAClassPathBeanDefinitionScanner的代理,能够让动态代理创建Bean;并且传入repositoryInterface参数  
```java
public class JPAClassPathBeanDefinitionScanner extends ClassPathBeanDefinitionScanner {
    public JPAClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry) {
        super(registry);
    }

    @Override
    protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
        AnnotationMetadata metadata = beanDefinition.getMetadata();
        return metadata.isInterface();
    }

    @Override
    protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
        /*
        这一步是由JPABeanDefinitionRegistryPostProcessor指示的
        beanDefinitionHolders就是所有扫描到的repository
        */
        Set<BeanDefinitionHolder> beanDefinitionHolders = super.doScan(basePackages);
        for (BeanDefinitionHolder beanDefinitionHolder : beanDefinitionHolders) {
            ScannedGenericBeanDefinition beanDefinition = (ScannedGenericBeanDefinition) beanDefinitionHolder.getBeanDefinition();
            // 得到当前repository的真正类型
            String beanClassName = beanDefinition.getBeanClassName();
            // 因为JPAFactoryBean不可能自动注入要动态代理的对象的类型,所以通过构造函数的方式传入
            beanDefinition.getConstructorArgumentValues().addGenericArgumentValue(beanClassName);
            // 偷天换日,将实例化repository的工厂改掉 mybatis也是这么做的
            beanDefinition.setBeanClass(JPAFactoryBean.class);
        }
        return beanDefinitionHolders;
    }
}
```

最终结果:  
![最终结果](resource/JPA/22.png)  
<font color="#00FF00">成功查询到数据,并且此时任意在repository包下添加任意类型的Repository接口都是可以的</font>  

### 3.3 Spring整合JPA的原理
1.进入@EnableJpaRepositories注解  
看到该注解被`@Import(JpaRepositoriesRegistrar.class)`注解修饰,之前在讲SpringBoot自动配置原理的时候提到,每个自动配置注解都有一个与之对应的@Import注解用于导入`@Configuration`  

2.JpaRepositoriesRegistrar  
该Bean又继承自RepositoryBeanDefinitionRegistrarSupport  

3.RepositoryBeanDefinitionRegistrarSupport  
RepositoryBeanDefinitionRegistrarSupport类实现了`ImportBeanDefinitionRegistrar`接口,那么它的效果就类似上面讲述的`BeanDefinitionRegistryPostProcessor`接口的作用  
用于`@Import`注解的动态注册器,那么此时就拥有了动态注册BeanDefinition的能力  

4.RepositoryComponentProvider  
自定义了RepositoryComponentProvider扫描器,效果就和上述JPAClassPathBeanDefinitionScanner差不多;不过该扫描器是直接重写了`findCandidateComponents()`方法,效果和`doScan()`方法差不太多  
并且该实现类也重写了`isCandidateComponent()`方法  

5.将扫描到的候选BeanDefinition重写创建BeanDefinition  

6.设置新的BeanDefinition为JpaRepositoryFactoryBean  

7.JpaRepositoryFactoryBean是一个FactoryBean  

8.JpaRepositoryFactoryBean会创建动态代理Bean  

总结一句话就是
**<font color="#FF00FF">SpringDataJPA的原理是:JDK动态代理+EntityManager+重写BeanDefinition</font>**



## 4.SpringBoot整合Spring Data JPA  
**目录:**  
4.1 SpringBoot整合Spring Data JPA  
4.2 配置大全  
4.3 SpringBoot自动配置原理(JPA)  

### 4.1 SpringBoot整合Spring Data JPA
1.新建模块springboot-jpa  

2.修改pom文件  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.0</version>
        <relativePath/>
    </parent>

    <artifactId>springboot-jpa</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>


    <dependencies>
        <!--JPA 场景启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
            <scope>compile</scope>
        </dependency>

    </dependencies>

</project>
```
3.创建启动类  
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
4.复制上述的pojo类和repository类  
这里以Customer为例进行复制即可  

5.编写yml配置文件  
```yml
spring:
  jpa:
    hibernate:
      ddl-auto: update # 表生成策略
    show-sql: true # 是否在控制台显示SQL(日志是否记录SQL)
  datasource:
    url: jdbc:mysql://192.168.149.131:7901/spring_data_jpa?useSSL=FALSE
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

6.创建controller  
```java
@RestController
public class CustomerController {

    private CustomerRepository customerRepository;

    public CustomerController(CustomerRepository customerRepository) {
        this.customerRepository = customerRepository;
    }

    @GetMapping("get")
    public List<Customer> get() {
        return (List<Customer>) customerRepository.findAll();
    }
}
```

7.测试运行  
![运行效果](resource/JPA/23.png)  

### 4.2 配置大全  
```yml
spring:
  jpa:
    hibernate:
      ddl-auto: update # 表生成策略
      naming:
        implicit-strategy:
        #物理命命名策略 一共五种策略
        # ImplicitNamingStrategyJpaCompliantImpl:默认的命名策略,兼容JPA 2.0的规范
        # ImplicitNamingStrategyLegacyHbmImpl 兼容Hibernate老版本中的命名规范
        # ImplicitNamingStrategyLegacyJpaImpl 兼容JPA 1.0规范中的命名规范
        # ImplicitNamingStrategyComponentPathImpl 大部分与ImplicitNamingStrategyJpaCompliantImpl
        # 但是对于@Embedded等注解标志的,组件处理是通过使用attributePath完成的
        # ImplicitNamingStrategyJpaCompliantImpl 默认,上面的四种策略均继承自它
        physical-strategy:  #隐式的命名策略,当使用@Column和@Table显示指定后改配置无效
        # PhysicalNamingStrategyStandardImpl 直接映射不做过多处理
        # SpringPhysicalNamingStrategy 表名,字段为小写,当有大写字母的时候会添加下划线分隔符号
      use-new-id-generator-mappings:  # 是否为Auto、table、sequence使用hibernate较新的ID生成策略

    show-sql: true # 是否在控制台显示SQL(日志是否记录SQL)
    generate-ddl: true #表的生成策略(如果是true则会生成,否则不会生成),和ddl-auto这两个二选一使用即可
    mapping-resources: # 设置JPA的配置文件persistence.xml文件的路径的
    open-in-view:
    # 将OpenEntityManagerInViewInterceptor绑定到一个线程,默认是true;
    # 使用true之后当一个线程进入之后会将session绑定到该线程,则该线程在执行完毕之前都可以拥有session对象
    properties: #设置JPA框架的各种配置,在这里可以设置hibernate它自已的各种配置
      hibernate:
        format_sql: true # 例如格式化SQL
  datasource:
    url: jdbc:mysql://192.168.149.131:7901/spring_data_jpa?useSSL=FALSE
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

### 4.3 SpringBoot自动配置原理(JPA)
1.通过@EnableAutoConfiguration  
*提示:每一个自动配置类都对应一个properties配置类,从yml中读取配置文件*  
spring-data-jpa有以下自动配置类  
`JpaBaseConfiguration` => JpaProperties
`JpaRepositoriesAutoConfiguration`

JpaRepositoriesAutoConfiguration:因为这里我们并没有使用@EnableJpaRepositories注解,所以这个注解的功能实际上是通过该配置类来进行实现的,在这个配置类中通过JpaRepositoriesRegistrar注册了`@EnableJpaRepositories`这个注解  


