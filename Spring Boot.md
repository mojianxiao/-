# Spring Boot 

#### spring boot配置读写分离

建立多个datasource放到TargetDataSource中，重写determineCurrentLookupKey方法来决定使用哪个key。

#### 配置

1. 数据源配置(application.yml)

   ```properties
   spring:
     datasource:
       master:
         jdbc-url: jdbc:mysql://192.168.102.31:3306/test
         username: root
         password: 123456
         driver-class-name: com.mysql.jdbc.Driver
       slave1:
         jdbc-url: jdbc:mysql://192.168.102.56:3306/test
         username: pig   # 只读账户
         password: 123456
         driver-class-name: com.mysql.jdbc.Driver
       slave2:
         jdbc-url: jdbc:mysql://192.168.102.36:3306/test
         username: pig   # 只读账户
         password: 123456
         driver-class-name: com.mysql.jdbc.Driver
   ```

   

2. 多数据源配置

   ```java
   package com.cjs.example.config;
   
   import com.cjs.example.bean.MyRoutingDataSource;
   import com.cjs.example.enums.DBTypeEnum;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.boot.context.properties.ConfigurationProperties;
   import org.springframework.boot.jdbc.DataSourceBuilder;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   import javax.sql.DataSource;
   import java.util.HashMap;
   import java.util.Map;
   
   /**
    * 关于数据源配置，参考SpringBoot官方文档第79章《Data Access》
    * 79. Data Access
    * 79.1 Configure a Custom DataSource
    * 79.2 Configure Two DataSources
    */
   
   @Configuration
   public class DataSourceConfig {
   
       @Bean
       @ConfigurationProperties("spring.datasource.master")
       public DataSource masterDataSource() {
           return DataSourceBuilder.create().build();
       }
   
       @Bean
       @ConfigurationProperties("spring.datasource.slave1")
       public DataSource slave1DataSource() {
           return DataSourceBuilder.create().build();
       }
   
       @Bean
       @ConfigurationProperties("spring.datasource.slave2")
       public DataSource slave2DataSource() {
           return DataSourceBuilder.create().build();
       }
   
       @Bean
       public DataSource myRoutingDataSource(@Qualifier("masterDataSource") DataSource masterDataSource,
                                             @Qualifier("slave1DataSource") DataSource slave1DataSource,
                                             @Qualifier("slave2DataSource") DataSource slave2DataSource) {
           Map<Object, Object> targetDataSources = new HashMap<>();
           targetDataSources.put(DBTypeEnum.MASTER, masterDataSource);
           targetDataSources.put(DBTypeEnum.SLAVE1, slave1DataSource);
           targetDataSources.put(DBTypeEnum.SLAVE2, slave2DataSource);
           MyRoutingDataSource myRoutingDataSource = new MyRoutingDataSource();
           myRoutingDataSource.setDefaultTargetDataSource(masterDataSource);
           myRoutingDataSource.setTargetDataSources(targetDataSources);
           return myRoutingDataSource;
       }
   
   }
   ```

3. mybatis配置

   ```java
   package com.cjs.example.config;
   
   import org.apache.ibatis.session.SqlSessionFactory;
   import org.mybatis.spring.SqlSessionFactoryBean;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
   import org.springframework.jdbc.datasource.DataSourceTransactionManager;
   import org.springframework.transaction.PlatformTransactionManager;
   import org.springframework.transaction.annotation.EnableTransactionManagement;
   
   import javax.annotation.Resource;
   import javax.sql.DataSource;
   
   @EnableTransactionManagement
   @Configuration
   public class MyBatisConfig {
   
       @Resource(name = "myRoutingDataSource")
       private DataSource myRoutingDataSource;
   
       @Bean
       public SqlSessionFactory sqlSessionFactory() throws Exception {
           SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
           sqlSessionFactoryBean.setDataSource(myRoutingDataSource);
           sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
           return sqlSessionFactoryBean.getObject();
       }
   
       @Bean
       public PlatformTransactionManager platformTransactionManager() {
           return new DataSourceTransactionManager(myRoutingDataSource);
       }
   }
   ```

   

4. 设置路由获取key

   ```java
   package com.cjs.example.enums;
   
   public enum DBTypeEnum {
   
       MASTER, SLAVE1, SLAVE2;
   
   }
   ```

   

5. ThreadLocal将数据源设置到每个线程上下文中

   ```java
   package com.cjs.example.bean;
   
   import com.cjs.example.enums.DBTypeEnum;
   
   import java.util.concurrent.atomic.AtomicInteger;
   
   public class DBContextHolder {
   
       private static final ThreadLocal<DBTypeEnum> contextHolder = new ThreadLocal<>();
   
       private static final AtomicInteger counter = new AtomicInteger(-1);
   
       public static void set(DBTypeEnum dbType) {
           contextHolder.set(dbType);
       }
   
       public static DBTypeEnum get() {
           return contextHolder.get();
       }
   
       public static void master() {
           set(DBTypeEnum.MASTER);
           System.out.println("切换到master");
       }
   
       public static void slave() {
           //  轮询
           int index = counter.getAndIncrement() % 2;
           if (counter.get() > 9999) {
               counter.set(-1);
           }
           if (index == 0) {
               set(DBTypeEnum.SLAVE1);
               System.out.println("切换到slave1");
           }else {
               set(DBTypeEnum.SLAVE2);
               System.out.println("切换到slave2");
           }
       }
   
   }
   ```

   

6. 获取路由key

   ```java
   package com.cjs.example.bean;
   
   import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
   import org.springframework.lang.Nullable;
   
   public class MyRoutingDataSource extends AbstractRoutingDataSource {
       @Nullable
       @Override
       protected Object determineCurrentLookupKey() {
           return DBContextHolder.get();
       }
   
   }
   ```

   

7. 设置路由key

   ```java
   package com.cjs.example.aop;
   
   import com.cjs.example.bean.DBContextHolder;
   import org.apache.commons.lang3.StringUtils;
   import org.aspectj.lang.JoinPoint;
   import org.aspectj.lang.annotation.Aspect;
   import org.aspectj.lang.annotation.Before;
   import org.aspectj.lang.annotation.Pointcut;
   import org.springframework.stereotype.Component;
   
   @Aspect
   @Component
   public class DataSourceAop {
   
       @Pointcut("!@annotation(com.cjs.example.annotation.Master) " +
               "&& (execution(* com.cjs.example.service..*.select*(..)) " +
               "|| execution(* com.cjs.example.service..*.get*(..)))")
       public void readPointcut() {
   
       }
   
       @Pointcut("@annotation(com.cjs.example.annotation.Master) " +
               "|| execution(* com.cjs.example.service..*.insert*(..)) " +
               "|| execution(* com.cjs.example.service..*.add*(..)) " +
               "|| execution(* com.cjs.example.service..*.update*(..)) " +
               "|| execution(* com.cjs.example.service..*.edit*(..)) " +
               "|| execution(* com.cjs.example.service..*.delete*(..)) " +
               "|| execution(* com.cjs.example.service..*.remove*(..))")
       public void writePointcut() {
   
       }
   
       @Before("readPointcut()")
       public void read() {
           DBContextHolder.slave();
       }
   
       @Before("writePointcut()")
       public void write() {
           DBContextHolder.master();
       }
   
   
       /**
        * 另一种写法：if...else...  判断哪些需要读从数据库，其余的走主数据库
        */
   //    @Before("execution(* com.cjs.example.service.impl.*.*(..))")
   //    public void before(JoinPoint jp) {
   //        String methodName = jp.getSignature().getName();
   //
   //        if (StringUtils.startsWithAny(methodName, "get", "select", "find")) {
   //            DBContextHolder.slave();
   //        }else {
   //            DBContextHolder.master();
   //        }
   //    }
   }
   ```

   