# Spring-MVC

### 快速使用spring-mvc



1. Idea 创建 spring-mvc项目
2. 编写Action实现Controller接口，其中我们只要实现handleRequest方法即可，方法需要传入request和response对象，并且返回一个ModelAndView(<font color="green">视图路径和封装的数据</font>)
3. 在web.xml中注册核心控制器
```
    <!-- 注册springmvc框架核心控制器 -->
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!--到类目录下寻找我们的配置文件-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/applicationContext.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <!--映射的路径为.action-->
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>
```
4. 在applicationContext中创建Spring-mvc控制器
```
<bean class = "Action.HelloAction" name = "hello.action">
```
#### 