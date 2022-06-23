# 原始SSM

首先，未整合之前的情况：就是配置文件很多撒，烦得很，我先一一列出来如下：

1. **Spring的配置**

   核心配置文件——**applicationContext.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
   
   <!--组件扫描-->
       <context:component-scan base-package="com.CCooky">
           <!--排除@Controller注解的扫描-->
           <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
       </context:component-scan>
   </beans>
   ```

   整合SpringMVC——**web.xml**。

   ```xml
   <!--    spring监听器（这里整合了SpringMVC）-->
       <context-param>
           <param-name>contextConfigLocation</param-name>
           <param-value>classpath:applicationContext.xml</param-value>
       </context-param>
       <listener>
           <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
       </listener>
   ```

2. **SpringMVC的配置**

   核心配置文件——**spring-mvc.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/mvc
          http://www.springframework.org/schema/mvc/spring-mvc.xsd">
     
   <!--		组件扫描controller-->
       <context:component-scan base-package="com.CCooky.controller"/>
   <!--		配置mvc注解驱动-->
       <mvc:annotation-driven></mvc:annotation-driven>
   <!--    内部资源视图解析器-->
       <bean id="resourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
           <property name="prefix" value="/WEB-INF/pages/"></property>
           <property name="suffix" value=".jsp"></property>
       </bean>
   <!--    开放静态资源的访问权限-->
       <mvc:default-servlet-handler></mvc:default-servlet-handler>
   </beans>
   ```

   前端控制器——**web.xml**

   ```xml
   <!--Springmvc 前端控制器-->
       <servlet>
           <servlet-name>DispatcherServlet</servlet-name>
           <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
           <init-param>
               <param-name>contextConfigLocation</param-name>
               <param-value>classpath:spring-mvc.xml</param-value>
           </init-param>
         	<!--服务器启动同时启动前端控制器-->
           <load-on-startup>1</load-on-startup>
       </servlet>
       <servlet-mapping>
           <servlet-name>DispatcherServlet</servlet-name>
         	<!--这是判断该访问资源是否进入我的前端控制器（这样就是任何访问资源-->
           <url-pattern>/</url-pattern>
       </servlet-mapping>
   <!--    springMVC请求数据乱码过滤器-->
       <filter>
           <filter-name>CharacterEncodingFilter</filter-name>
           <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
           <init-param>
               <param-name>encoding</param-name>
               <param-value>UTF-8</param-value>
           </init-param>
       </filter>
       <filter-mapping>
           <filter-name>CharacterEncodingFilter</filter-name>
           <url-pattern>/*</url-pattern>
       </filter-mapping>
   ```

3. **Mybatis的配置**

   数据库连接信息——**jdbc.properties**

   ```properties
   jdbc.driver=com.mysql.jdbc.Driver
   jdbc.url = jdbc:mysql://localhost:3306/studysql?useSSL=false
   jdbc.username = root
   jdbc.password= 5240zhouquan
   ```

   核心配置文件——**mybatis-config.xml**

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   
   
   <configuration>
   <!--    加载外部数据源配置-->
       <properties resource="jdbc.properties"></properties>
   <!--    给实体类全限定名起别名-->
       <typeAliases>
           <package name="com.CCooky.pojo"/>
       </typeAliases>
   <!--数据库-->
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="${jdbc.driver}"/>
                   <property name="url" value="${jdbc.url}"/>
                   <property name="username" value="${jdbc.username}"/>
                   <property name="password" value="${jdbc.password}"/>
               </dataSource>
           </environment>
       </environments>
   <!--    扫描SQL映射文件-->
       <mappers>
           <package name="com.CCooky.mapper"/>
       </mappers>
   </configuration>
   ```

这样相当于仅仅把SpringMVC整合到了Spring里面，Mybatis还是他的那一套操作，体现在我们的业务层，如下：我们需要通过Mybatis来手动创建SqlSessionFactory工厂对象，最后拿到Mapper对象进行操作。有太多的重复代码了，很烦洛。一个是获取外面的Mapper接口对象，一个是事务的控制。

```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Override
    public boolean save(Account account) {
        /**
         * 原生mybatis的操作
         */
        SqlSessionFactory sqlSessionFactory = SqlSessionFactoryUtil.getSqlSessionFactory();
        SqlSession sqlSession = sqlSessionFactory.openSession();
        AccountMapper mapper = sqlSession.getMapper(AccountMapper.class);
        boolean result = mapper.save(account);
        sqlSession.commit();
        sqlSession.close();
        return result;
    }

    @Override
    public List<Account> selectAll() {
        SqlSessionFactory sqlSessionFactory = SqlSessionFactoryUtil.getSqlSessionFactory();
        SqlSession sqlSession = sqlSessionFactory.openSession();
        AccountMapper mapper = sqlSession.getMapper(AccountMapper.class);
        List<Account> accountList = mapper.selectAll();
        sqlSession.close();
        return accountList;
    }
}
```

```java
// Mybatis工厂对象创建工具
public class SqlSessionFactoryUtil {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static SqlSessionFactory getSqlSessionFactory() {
        return sqlSessionFactory;
    }
}
```

### 依赖

```xml
  <dependencies>
<!--    Dao-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.9</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.49</version>
    </dependency>
      <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>druid</artifactId>
          <version>1.2.6</version>
      </dependency>

<!--    Test-->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>
<!--    Spring & aop(aspectj)-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.3.14</version>
    </dependency>
      <dependency>
          <groupId>org.aspectj</groupId>
          <artifactId>aspectjweaver</artifactId>
          <version>1.9.6</version>
      </dependency>
      <!--spring整合mybatis-->
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-jdbc</artifactId>
          <version>5.0.5.RELEASE</version>
      </dependency>
      <!--spring整合mybatis-->
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-tx</artifactId>
          <version>5.0.5.RELEASE</version>
      </dependency>
      <!--spring整合mybatis,这里提供了一个工厂对象的实现类-->
      <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis-spring</artifactId>
          <version>2.0.7</version>
      </dependency>
<!--      springMVC-->
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>5.3.14</version>
      </dependency>
      <dependency>
          <groupId>javax.servlet</groupId>
          <artifactId>servlet-api</artifactId>
          <version>2.5</version>
          <scope>provided</scope>
      </dependency>
<!--      Log-->
      <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-api</artifactId>
          <version>1.7.32</version>
      </dependency>
      <dependency>
          <groupId>ch.qos.logback</groupId>
          <artifactId>logback-classic</artifactId>
          <version>1.2.9</version>
      </dependency>
      <dependency>
          <groupId>ch.qos.logback</groupId>
          <artifactId>logback-core</artifactId>
          <version>1.2.9</version>
      </dependency>
<!--      Lombok-->
      <dependency>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <version>1.18.20</version>
      </dependency>
<!--      jsp-->
      <dependency>
          <groupId>javax.servlet.jsp</groupId>
          <artifactId>jsp-api</artifactId>
          <version>2.2</version>
          <scope>provided</scope>
      </dependency>
      <dependency>
          <groupId>jstl</groupId>
          <artifactId>jstl</artifactId>
          <version>1.2</version>
      </dependency>
  </dependencies>
```

# 整合SSM

这里整合也就是解决了前面的重复代码问题。

## 整合思路

![](images/11.png)

第一点：我们想直接从容器里面拿到我们的Mapper接口对象实例。省去前面所有的操作，让Spring去给我们创建SqlSessionFactory工厂对象。

第二点：事务控制交给Spring管理。

### **第一步：Mapper接口对象实例**

导入依赖。

```xml
<!--spring整合mybatis-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
<!--spring整合mybatis-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.0.5.RELEASE</version>
</dependency>
<!--spring整合mybatis,这里提供了一个工厂对象的实现类-->
      <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis-spring</artifactId>
          <version>2.0.7</version>
      </dependency>
```

删除原先mybatis-config.xml里面的一些配置信息，全部给Spring配置管理。之前的配置文件，现在仅仅剩下了起别名的配置。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--    给实体类全限定名起别名-->
    <typeAliases>
        <package name="com.CCooky.pojo"/>
    </typeAliases>
    
</configuration>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

<!--组件扫描-->
    <context:component-scan base-package="com.CCooky">
        <!--排除controller注解的扫描-->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

<!--    下面是整合Mybatis的步骤-->
    <!--1. 加载数据库properties文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--2. 配置数据源信息-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <!--3. 配置SqlSessionFactory工厂。这里spring提供了一个工厂对象的实现类-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--加载mybatis的核心配置文件-->
        <property name="configLocation" value="classpath:mybatis-config-spring.xml"/>
    </bean>
    <!--4. 扫描mapper映射文件所在包，并且为我们的mapper接口创建实现类，放到容器里面-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.CCooky.mapper"/>
    </bean>
</beans>
```

现在，我们的Mapper接口对象的实现类就有了，并且放在了容器当中。那么在我们的业务层就可以直接注入该对象，使用。

```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {
		
  	// 整合了Mybatis
    @Autowired
    private AccountMapper accountMapper;

    @Override
    public boolean save(Account account) {
        /**
         * 原生mybatis的操作
         */
//        SqlSessionFactory sqlSessionFactory = SqlSessionFactoryUtil.getSqlSessionFactory();
//        SqlSession sqlSession = sqlSessionFactory.openSession();
//        AccountMapper mapper = sqlSession.getMapper(AccountMapper.class);
//        boolean result = mapper.save(account);
//        sqlSession.commit();
//        sqlSession.close();
//        return result;
      
        /**
         * 整合了mybatis的操作
         */
        boolean result = accountMapper.save(account);
        return result;
    }
}
```

### 第二点：声明式事务控制

这个很简单，就是Spring里面的声明式事务控制。在Spring核心配置文件加上去就好了。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd">

<!--组件扫描-->
    <context:component-scan base-package="com.CCooky">
        <!--排除controller注解的扫描-->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

<!--    下面是整合Mybatis的步骤-->
    <!--1. 加载数据库properties文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>
    <!--2. 配置数据源信息-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    <!--3. 配置SqlSessionFactory工厂。这里spring提供了一个工厂对象的实现类-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--加载mybatis的核心配置文件-->
        <property name="configLocation" value="classpath:mybatis-config-spring.xml"/>
    </bean>
    <!--4. 扫描mapper映射文件所在包，并且为我们的mapper接口创建实现类，放到容器里面-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.CCooky.mapper"/>
    </bean>
  
  
<!--    5. 声明式事务控制-->
    <!--配置平台事务管理器-->
    <bean id="transctionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--通知 事务的增强配置-->
    <tx:advice id="txAdvice" transaction-manager="transctionManager">
        <!--设置事务的属性信息-->
        <tx:attributes>
            <!--name 是被增强的方法名-->
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>
    <!--    配置事务AOP织入-->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.CCooky.service.impl.*.*(..))"/>
    </aop:config>
</beans>
```