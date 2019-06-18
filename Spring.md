# Spring ioc操作

1.把对象的创建交给spring
2.（1）ioc的配置文件方式（2）ioc的注解方式

### ioc底层原理
1.xml配置文件
2.dom4j解决xml
3.工厂设计模式

### ioc入门
第一步 导入jar包
第二步 创建类，类中创建方法
第三步 创建 spring配置文件，配置创建类(建议放到src下面 一般起名applicationContext.xml)
第四步 写代码测试对象创建   

### Bean实例化的方式
1.使用类的无参数构造创建
2.使用静态工厂创建
3.使用实例工厂创建

### Bean标签常用属性
1.id属性：随便的名字
2.class属性：创建对象所在类的全路径
3.name属性：和id的功能一样，区别在于name可以加特殊符号
4.scope属性

### Spring注入属性
1.set方法注入(\<property name=" " value=" ">)
2.有参数构造注入（\<constructor-arg name=" " value=" ">）如果注入的为对象， \<property name=" " ref=" ">

### Spring整合web项目原理
1.加载spring核心配置文件
2.实现思想：把加载配置文件和创建对象过程在服务器启动时候完成
3.实现原理：
（1）ServletContext对象
（2）监听器
### 基于注解开发的Spring
#### 创建对象
(1)加入context约束，开启扫描注解
\<context:component-scan base-package=""></context:component-scan>
(2)注解:在对象类中使用注解(@Component(values="")表面bean id和class 使用@scope表示作用域)
(3)使用应用上下文创建对象
还可以使用@Controller @Service @Repository这三个注解创建对象

### Bean的生命周期
1.<font color=#0099ff face="黑体">实例化Bean</font>:在ApplicationContext容器中，当容器结束后，便实例化所有bean。实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。
2.<font color=#0099ff face="黑体">设置对象属性(依赖注入)</font>:Spring根据BeanDefinition中的信息进行依赖注入并且通过BeanWrapper提供的设置属性的接口完成依赖注入。
3.<font color=#0099ff face="黑体">注入Aware接口</font>：Spring会检测该对象是否实现xxxAware接口，并将相关的xxxAware实例注入给bean。
4.<font color=#0099ff face="黑体">BeanPostProcessor</font>：bean对象已经被正确构造，可以通过postProcessBeforeInitialzation( Object bean, String beanName )在Initialzation执行前执行，称为前置处理。还可以通过postProcessAfterInitialzation( Object bean, String beanName )在Initialzation执行后执行，称为后置处理。
5.<font color=#0099ff face="黑体">InitializingBean与init-method</font>：这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。
6.<font color=#0099ff face="黑体">DisposableBean和destroy-method</font>:在bean销毁前执行指定的逻辑。

### Bean的管理方式比较（基于xml和基于注解）

|          | 基于xml配置                   | 基于注解配置         |
| -------- | ----------------------------- | -------------------- |
| Bean定义 | \<bean id="..." class="...."> | @Component           |
| Bean名称 | 通过id或name指定              | @Component("person") |
| Bean注入 | \<property> | @Autowrized:按类型注入<br>@Qualifier:按名称注入
| 适用场景 | bean来自第三方 | 由自己开发 |
| 优点 | 结构清晰 | 开发方便 |

# AOP
1.AOP的概念:面向切面的编程，扩展功能不修改源代码实现
2.AOP采取横向抽取机制，取代了传统纵向继承体系重复性代码
3.AOP底层使用动态代理实现
（1）第一种情况:有接口的情况，使用动态代理创建接口实现类代理对象
（2）第二种情况：没有接口的情况，使用动态代理创建类的子类代理对象

### Spring的AOP开发:
#### 基于xml的开发

1. 引入相应的jar包
2. 引入Spring的配置文件
3. 编写目标类
```
public interface UserDao{
    public void say();
}
public class UserDaoImpl() implements UserDao{
    public void say(){
        System.out.println("hello world")
    }
}
```
4. 目标类的配置
```
<bean id="userDao" class="UserDaoImpl"></bean>
```
5. 整合Junit单元测试
6. 通知类型
7. 切入点表达式
8. 编写一个切面类
```
public class Before{
    public void before(){
        System.out.println("这是前置增强")
    }
}
```
9. 配置完成增强
```
<bean id ="before" class="Before"></bean>
<aop:config>
<aop:poincut expression="execution(*UserDao.say)" id="pointcut">
<aop:aspect ref="before">
<aop:before method="before" poincut-ref="pointcut">
</aop aspect>
</aop:config>
```
  10. 其他的增强的配置:
  11. 基于注解模式的AOP开发
  12. 引入相关的jar包
  13. 引入Spring的配置文件
  14. 编写目标类
 ```
 public class UserDao{
     public void save(){
         System.out.println("保存一些东西....")
     }
 }

 ```
 4. 配置目标类
 ```
 <bean id="userDao" class="UserDao"></bean>
 ```
 5. 开启aop注解的自动代理
 ```
 <aop:aspectj-autoproxy/>
 ```
 6. 使用注解进行开发
 7. 编写切面
 ```
 @Aspect
 public class MyAspectAnno{
     @Before("MyAspectAnno.pointcut1()")
     public void before(){
         System.out.println("这是前置增强....")
     }

     @Pointcut("execution(* UserDao.save(..))")
     private void pointcut(){

     }
 }
 ```
 8. 配置切面
 ```
 <bean id="myAspectAnno" class="MyAspectAnno"></bean>
 ```
 9. 其他通知的注解
 ```
 package com.mojianxiao.aopanno;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class UserAdd {
	@Before(value="execution(* com.mojianxiao.aopanno.User.*(..))")
	public void before() {
		System.out.println("........这是前置增强！");
	}
	@Around(value="execution(* com.mojianxiao.aopanno.User.*(..))")
	public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
		System.out.println(".....before");
		proceedingJoinPoint.proceed();
		System.out.println(".....after");	
	}
}

 ```

### 事务
#### 事务的特性

 1. 原子性：强调事务的不可分割
 2. 一致性：事务的执行的前后数据的完整性保持一致
 3. 隔离性：一个事务执行的过程中，不应该受到其他事务的干扰
 4. 事务一旦结束，数据将持久到数据库

  #### 事务的隔离级别
 1. 读未提交
 2. 读已提交
 3. 可重复读
 4. 串行化

### Spring JdbcTemplate
1.首先配置db.properties
```
jdbc.user=root
jdbc.password=123456
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.jdbcUrl=jdbc\:mysql\:///test
```
2.配置spring配置文件applicationContext
```
<context:property-placeholder location="classpath:db.properties"/>
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="user" value="${jdbc.user}"></property>
    <property name="password" value="${jdbc.password}"></property>
    <property name="driverClass" value="${jdbc.driverClass}"></property>
    <property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>
</beans>
```
3.插入一条数据
```
//启动IoC容器
ApplicationContext ctx=new ClassPathXmlApplicationContext("applicationContext.xml");
//获取IoC容器中JdbcTemplate实例
JdbcTemplate jdbcTemplate=(JdbcTemplate) ctx.getBean("jdbcTemplate");
String sql="insert into user (name,deptid) values (?,?)";
int count= jdbcTemplate.update(sql, new Object[]{"caoyc",3});
System.out.println(count);
```
4.插入多条数据
```
List<Object[]>batchArgs = new ArrayList<Object[]>();
batchArgs.add(new Object[] {"gg",22});
batchArgs.add(new Object[] {"mm",11});
jdbcTemplate.batchUpdate(sql, batchArgs);
```
5.从数据中读取数据到实体对象
（1）先创建相应的实体类
```
package com.mojianxiao.jdbcTemplate;

public class User {
	private int id;
	private String name;
	private int deptid;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getDeptid() {
		return deptid;
	}
	public void setDeptid(int deptid) {
		this.deptid = deptid;
	}
	@Override
	public String toString() {
		return "User [id=" + id + ", name=" + name + ", deptid=" + deptid + "]";
	}
}

```
(2)编写查询的SQL语句
```
String sql = "select id,name,deptid from user where id=?";
```
(3)使用RowMapper存储对象
```
RowMapper<User> rowMapper = new BeanPropertyRowMapper<User>(User.class);
User user = jdbcTemplate.queryForObject(sql, rowMapper,1);
//获取多个对象
JdbcTemplate jdbcTemplate = (JdbcTemplate)ctx.getBean("jdbcTemplate");
List<User>users = jdbcTemplate.query(sql, rowMapper);		
``
```