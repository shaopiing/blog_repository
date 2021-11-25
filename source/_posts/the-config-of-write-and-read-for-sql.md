---
title: "知识百科系列 (三) —— Java Web 项目中数据库的读写分离的配置"
date: 2017-09-20 23:49:13
category: 程序人生
tags: [Program,SQL,WikiKnowledge]
---
> 随着项目的使用，数据表中的数据也在不断增加，这时候为了更好的增加用户体验，除了优化 SQL，还可以考虑引入只读库作为从库来分担压力
<!--more-->
**1、注入的方式**

最简单的，可以用注入的方式来配置只读库。

首先，可以在项目的数据库配置文件中配置只读库的数据源：

```properties
env.appCode.readonly.jdbc.url=jdbc:sqlserver://xxx.xxx.xxx.xxx:xxxx;DatabaseName=oms;sendStringParametersAsUnicode=false;ApplicationIntent=ReadOnly
env.appCode.readonly.jdbc.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver
env.appCode.readonly.jdbc.username=test
env.appCode.readonly.jdbc.password=******
```

然后在spring配置文件中增加对应的只读数据源的配置：

```xml
<bean id="dataSourceReadOnly" class="org.apache.tomcat.jdbc.pool.DataSource" destroy-method="close">

    <property name="driverClassName" value="${${env}.boms.readonly.jdbc.driver}" />

    <property name="url" value="${${env}.boms.readonly.jdbc.url}" />

    <property name="username" value="${${env}.boms.readonly.jdbc.username}" />

    <property name="password" value="${${env}.boms.readonly.jdbc.password}" />

</bean>

<bean id="queryOnlyDao" class="com.xxx.web.dao.QueryDao"

          autowire="byName" p:dataSource-ref="dataSourceReadOnly"/>
```

接着在 service 中注入便可以使用了：

```java
@Service
public class TestService{

    @Autowired
    private QueryDao queryOnlyDao;
    
    public List&lt;Entity> queryForDetails(Map&lt;String, Object> params){
        ...
        
        return queryOnlyDao.queryForList(sqlId("queryForDetails"), params);
    }
    
    ...

}
```

可用用如下 SQL 语句验证下只读库连接是否正常：

```sql
SELECT @@SERVERNAME
```


**2、注解的方式**

还有一种方式是，借助并扩展 Spring 中的 AbstractRoutingDataSource，充当 DataSource 的路由中介，根据某种 key 的值来动态切换到真正的 Datasource 上。

`AbstractRoutingDataSource` 继承自 `AbstractDataSource` ，而 `AbstractDataSource` 又是 `javax.sql.DataSource` 的子类，所以只需看它实现的 `getConnection()` 即可：

![](http://p8bc1hri5.bkt.clouddn.com/the-config-of-write-and-read-for-sql-1.png)

获取连接的方法，看下源码中的 `determineTargetDataSource()` 方法：

![](http://p8bc1hri5.bkt.clouddn.com/the-config-of-write-and-read-for-sql-2.png)

该方式是通过一个抽象方法 `determineCurrentLookupKey()` 的返回值作为所用数据源的 key 值，
通过这个 key 值，可以从一个对象 `Map<Object, DataSource> resolvedDataSources` 中获取对应的数据源实例。

所以可以通过`重写该方法`来达到动态切换数据源的目的，同样也可以用来实现读写库切换：

```java
public class RoutingDatasource extends AbstractRoutingDataSource {

    private static final Log logger = LogFactory.getLog(RoutingDatasource.class);
    
    @Override
    protected Object determineCurrentLookupKey() {
        String determineDatabase = RoutingDatasourceHolder.getCurrentDatabase();
        logger.debug("determineDatabase--"+determineDatabase);
        return determineDatabase;
    }

}
```

数据源 key 值的配置类 `RoutingDatasourceHolder` 可以如下设置：

```java
public class RoutingDatasourceHolder {

    /**
     * 只读数据库
     */
    public static final String READ_DB = "dataSource_R";
    
    /**
     * 读写数据库
     */
    public static final String READ_WRITE_DB = "dataSource_RW";
    
    private static ThreadLocal&lt;String> dbThread = new ThreadLocal<>();
    
    public static String getCurrentDatabase(){
        return dbThread.get();
    }
    
    public static void setCurrentDatabase(String dbName){
        dbThread.set(dbName);
    }
    
    public static void clear(){
        dbThread.remove();
    }

}
```

为了方便配置，可以用注解的方式来设置数据源：

```java
@Target({ ElementType.METHOD, ElementType.TYPE })

@Retention(RetentionPolicy.RUNTIME)

@Documented

public @interface DBReadOnly {

}
```

然后通过继承 Spring 提供的拦截适配器

`org.springframework.web.servlet.handler.HandlerInterceptorAdapter` 

更改使用注解地方的数据源：

```java
public class ReadOnlyInterceptor extends HandlerInterceptorAdapter {
           

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if(handler instanceof HandlerMethod ){
            HandlerMethod hm = (HandlerMethod) handler;
            Method m = hm.getMethod();
            Annotation anno = m.getAnnotation(DBReadOnly.class);
            if (anno != null) {
                //开启只读数据库
              RoutingDatasourceHolder.setCurrentDatabase(RoutingDatasourceHolder.READ_DB);
            }
        }
        
        return true;
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        //关闭设置的只读数据库
        RoutingDatasourceHolder.clear();
    }

}
```

然后在使用只读库的方法上面增加 `DBReadOnly` 注解即可，和之前注入 `queryOnlyDao` 的方式比，更加方便使用而且侵入性大大降低。