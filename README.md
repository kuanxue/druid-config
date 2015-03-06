druid连接池配置
===

###一、添加JAR包

###二、配置web.xml
#####1.根据配置中的url-pattern来访问内置监控页面:
```xml
	<!-- druid 监控-->
	<servlet>
		<servlet-name>DruidStatView</servlet-name>
		<servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>DruidStatView</servlet-name>
		<url-pattern>/druid/*</url-pattern>
	</servlet-mapping>
```

#####2.配置采集关联web的数据
```xml
	<!-- WebStatFilter用于采集web-jdbc关联监控的数据。 -->
	<filter>
	  <filter-name>DruidWebStatFilter</filter-name>
	  <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
	  <init-param>
	      <param-name>exclusions</param-name>
	      <param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
	  </init-param>
	</filter>
	<filter-mapping>
	  <filter-name>DruidWebStatFilter</filter-name>
	  <url-pattern>/*</url-pattern>
	</filter-mapping>
```

###三、配置druid连接池，applicationContext.xml修改
```xml
<!-- 配置数据源  org.apache.commons.dbcp.BasicDataSource com.alibaba.druid.pool.DruidDataSource-->
	<bean id="dataSource_admin" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 
	    <!-- 基本属性 url、user、password -->
	    <property name="url" value="${admin.jdbcUrl}" />
	    <property name="username" value="${admin.user}" />
	    <property name="password" value="${admin.password}" />
	    <!-- 配置初始化大小、最小、最大 -->
	    <property name="initialSize" value="1" />
	    <property name="minIdle" value="1" /> 
	    <property name="maxActive" value="20" />
	    <!-- 配置获取连接等待超时的时间 -->
	    <property name="maxWait" value="60000" />
	    <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
	    <property name="timeBetweenEvictionRunsMillis" value="60000" />
	    <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
	    <property name="minEvictableIdleTimeMillis" value="300000" />
	    <property name="validationQuery" value="SELECT 'x'" />
	    <property name="testWhileIdle" value="true" />
	    <property name="testOnBorrow" value="false" />
	    <property name="testOnReturn" value="false" />
	    <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
	    <property name="poolPreparedStatements" value="true" />
	    <property name="maxPoolPreparedStatementPerConnectionSize" value="20" />
	    <!-- 配置监控统计拦截的filters -->
	    <property name="filters" value="stat" /> 
	</bean>
```

###四、配置druid监控，采用AOP方式，主要是service方法
```xml
	<!-- 配置druid监控-->
  <bean id="druid-stat-interceptor" class="com.alibaba.druid.support.spring.stat.DruidStatInterceptor"></bean>

	<bean id="druid-stat-pointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut" scope="prototype">
	    <property name="patterns">
	        <list>
	            <value>com.kuanxue.admin.service.*</value>
	        </list>
	    </property>
	</bean>

	<aop:config>
	    <aop:advisor advice-ref="druid-stat-interceptor" pointcut-ref="druid-stat-pointcut" />
	</aop:config>
```

###五、访问http://XXXX/druid/sql.html 即可查看监控状态