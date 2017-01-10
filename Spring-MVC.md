## Spring-MVC访问静态资源 ##
	/**/src/main/webapp/WEB-INF/web.xml

	<servlet-mapping>	
		<servlet-name>SpringMVC</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
DispatcherServlet配置为拦截“/”，即拦截所有请求，造成对*.js,*.jpg的访问也被拦截。除了将配置改为拦截“\*.do”以外还有3种解决方法

1. 使用Tomcat的defaultServlet来处理静态文件
		/**/src/main/webapp/WEB-INF/web.xml

		<servlet-mapping>        
			<servlet-name>default</servlet-name>       
			<url-pattern>*.jpg</url-pattern>     
		</servlet-mapping>
		<servlet-mapping>            
			<servlet-name>default</servlet-name>         
			<url-pattern>*.js</url-pattern>    
		</servlet-mapping>    
		<servlet-mapping>             
			<servlet-name>default</servlet-name>            
			<url-pattern>*.css</url-pattern>      
		</servlet-mapping>
	需配置多个，并且要写在DispatcherServlet的前面， 让defaultServlet先拦截，这种处理方式就不会进入Spring了，性能可能是最好的。
		容器默认servlet-name

		Tomcat、Jetty、JBoss、GlassFish ： "default"
		Google App Engine ： "_ah_default"
		Resin ： "resin-file"
		WebLogic ： "FileServlet"
		WebSphere ： "SimpleFileServlet" 
2. 使用spring3.0.4以后提供的mvc:resources 		
		/storm/src/main/resources/spring-mvc.xml

		<!-- 对静态资源文件的访问 -->     
		<mvc:resources mapping="/images/**" location="/images/"/>
	使用<mvc:resources/>元素，把mapping的URI注册到SimpleUrlHandlerMapping的urlMap中（key为mapping的URI pattern值，value为ResourceHttpRequestHandler），这样就巧妙的把对静态资源的访问由HandlerMapping转到ResourceHttpRequestHandler处理，location指定静态资源的位置。可以是web application根目录下，也可以是jar包里面，这样可以把静态资源压缩到jar包中。
	
	注：不要对SimpleUrlHandlerMapping设置defaultHandler，因为对static uri的defaultHandler就是ResourceHttpRequestHandler，否则无法处理static resources request。
3. 使用<mvc:default-servlet-handler/>
		/storm/src/main/resources/spring-mvc.xml
		
		<mvc:default-servlet-handler/>   
	使用<mvc:default-servlet-handler/>，会对进入DispatcherServlet的请求进行筛查，把"/"url注册到SimpleUrlHandlerMapping的urlMap中，如果发现是没有经过映射的请求（对静态资源的访问），将该请求由HandlerMapping转到org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler处理，DefaultServletHttpRequestHandler使用就是各个Servlet容器自己的默认Servlet。

	注：若所使用的 WEB 服务器的默认 Servlet 名称不是 default，则需要通过 default-servlet-name 属性显式指定，同时需要配置<mvc:annotation-driven />，否则requestmapping失效
 
补充说明多个HandlerMapping的执行顺序问题：

DefaultAnnotationHandlerMapping的order属性值是：0
<mvc:resources/>自动注册的SimpleUrlHandlerMapping的order属性值是： 2147483646
<mvc:default-servlet-handler/>自动注册的SimpleUrlHandlerMapping的order属性值是：2147483647

spring会先执行order值比较小的。当访问一个a.jpg图片文件时，先通过DefaultAnnotationHandlerMapping 来找处理器，一定是找不到的，我们没有叫a.jpg的Action。再按order值升序找，由于最后一个SimpleUrlHandlerMapping 是匹配"/"的，所以一定会匹配上，再响应图片。