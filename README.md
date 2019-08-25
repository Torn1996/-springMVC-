### 一、SpringMVC架构图

![](/images/springMVC架构图.png)

​																		图 1-1 SpringMVC架构图

> #### SpringMVC架构图
>
> - ​	DispatcherServlet：前端控制器
>
> 用户请求到达前端控制器，它就相当于MVC模式中的C，DispatcherServlet是整个流程控制的中心，由它调用其他的组件处理用户的请求，，DispatcherServlet的存在减低了其他组件的耦合性。
>
> - HandlerMapping：处理器映射器
>
> HandlerMapping负责根据用户请求URL找到Handler即处理器，SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式、接口实现方式、注解方式等。
>
> - Handler：处理器
>
> Handler是继承DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的的用户请求进行处理。由于Handler涉及到了具体的用户业务，所以一般情况下需要程序员根据业务开发Handler。
>
> - HandlerAdapter：处理器适配器
>
> 通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
>
> - View Resolver：视图解析器
>
> View Resolver视图解析器将处理结果解析成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。
>
> - View：视图
>
> SpringMVC框架提供了很多的View视图类型的支持：包括：jstlView、freemarkerView、PdfView等。我们最常用的视图就是jsp。

[](https://blog.csdn.net/qq_42780864/article/details/81461306)

### 二、请求是怎么到达DispatcherServlet

> #### Tomcat容器模型
>
> ​	介绍servlet首先就先要把servlet容器介绍清楚，现在市面上比较流行的servlet容器有Tomcat、Jetty，在这里还是以我们比较熟悉的Tomcat为例。Tomcat容器模型还是比较复杂的，在Tomcat的容器等级中，Context容器是直接管理Servlet在容器中的包装类Wrapper,所以Context容器如何运行将直接影响Servlet的工作方式。

![](/images/Tomcat容器模型.png)

​																		图 2-1 Tomcat模型图

​	从上图可以看出Tomcat的容器分成4个等级，真正管理Servlet的容器是Context容器，一个Context容器对应一个Web工程，在Tomcat的配置文件中可以很容易的看出这一点，如下：

```xml
<Context docBase="ssmCrud" path="/ssmCrud" reloadable="true" source="org.eclipse.jst.jee.server:ssmCrud"/></Host>
path 和 docBase 分别代表这个应用在 Tomcat 中的访问路径和这个应用实际的物理路径
```

[](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/)

> #### Tomcat接收到用户的请求
>
> ​	首先Tomcat是一个基于组件的的服务器，它的构成组件都是可以配置的，可以在Tomcat目录下conf/server.xml中进行配置。其配置文件结构如下：

```xml
<Server>顶层类元素：一个配置文件中只能有一个<Server>元素，可包含多个Service。
	<Service>顶层类元素：本身不是容器，可包含一个Engine，多个Connector。
    	<Connector/>连接器类元素：代表通信接口。
    	<Engine>容器类元素：为特定的Service组件处理所有客户请求，可包含多个Host。engine为顶层		Container
      	 <Host>容器类元素：为特定的虚拟主机处理所有客户请求，可包含多个Context。
          	<Context>容器类元素：为特定的Web应用处理所有客户请求。代表一个应用，包含多个Wrapper（封装了servlet）
         	 </Context>
         </Host>
    	</Engine>
	</Service>
</Server>
```

​	Tomcat组件中的Connector负责监听某个端口上的HTTP请求，构造Request、Response最终传递给Container中的servlet处理这个请求，收到响应返回，如下图所示：

![](/images/http和Tomcat交互.png)

​																图 2-2 Http和Tomcat交互

> #### web.xml中注册DispatcherServlet

​	

```xml
<!-- 注册DispatcheServlet -->
  <servlet>
  	<servlet-name>dispatcherServlet</servlet-name>
  	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  	<!-- tomcat启动时自动创建servlet，数字越小优先级越高(>0) -->
  	<load-on-startup>1</load-on-startup>
    <init-param>
  		<!-- 如果dispatcherServlet的配置文件放在WEB-INF目录下，但不想名字的形式为*-servlet.xml,
  		则可以通过namespace属性值进行指定 -->
  		<param-name>namespace</param-name>
  		<param-value>dispatcherServlet</param-value>
  	</init-param>
  </servlet>
  
  <servlet-mapping>
  	<servlet-name>dispatcherServlet</servlet-name>
  	<url-pattern>*.jsp</url-pattern>
  </servlet-mapping>
</web-app>
```

> #### 请求达到DispatcherServlet

```
	假设来自用户的请求为：http://localhost:8080/test/index.jsp,请求将会被转发到本机端口8080：
		1.被在那里侦听的Coyote HTTP/1.1 Connector获得；
		2.然后Connector把请求到交给它所在的Service的Engine来处理，并等待Engine的回应；
		3.Engine获得localhost:8080/test/index,jsp,匹配它所有的虚拟主机Host；
		4.Engine匹配到localhost的Host（即使匹配不到也会把请求交给该Host处理，因为该Host被定义为Engine的默认主机）；
		5.localhost Host获得请求/test/index.jsp,匹配它所拥有的所有Context；
		6.Host匹配到路径为/test的Context（如果匹配不到的话，就将该请求交给路径名称为" "的Context去处理）；
		7.path为“/test”的Context获得/index.jsp的请求,在它的mapping table寻找相对应的servlet；
		8.context匹配到URL pattern为“*.jsp”的servlet，对应于JSPServlet类，构造HTTPServletRequest对象和HTTPServletResponse对象，作为参数调用JSPServlet的doGet和doPost方法；
		9.Context将执行完后的HTTPServletResponse返回给Engine；
		10.Engine将HTTPServletResponse返回给Connector；
		11.Connector将HTTPServletResponse返回给浏览器。
		
```

![](/images/Tomcat和MVC.png)

### 三、Connector怎么解决高并发问题

> #### Connector种类
>
> ​	Tomcat源码中与connector相关的类位于org.apache.coyote包中，Connector分为以下几类：
>
> - Http Connector，基于HTTP协议，负责建立HTTP连接。它又分为BIO Http Connector与NIO Http Connector两种，，后者提供非阻塞IO与长连接Comet支持。默认情况下Tomcat使用的就是这个Connector。
> - AJP Connector，基于AJP协议，AJP是专门设计用来为Tomcat和HTTP服务器之间通信专门定制的协议，能提供较高的的通信速率和效率。如与Tomcat服务器集成时，采用采用这个协议。
> - APR Http Connector，用C实现，通过JNI调用的。主要提升对静态资源（如HTML、CSS、JS等）的访问性能。现在这个库已经独立出来可以用在任何项目中。

​	具体地，Tomcat7中实现了以下几种Connector：

```
org.apache.coyote.http11.Http11Protocol : 支持HTTP/1.1 协议的连接器。
org.apache.coyote.http11.Http11NioProtocol : 支持HTTP/1.1 协议+New IO的连接器。
org.apache.coyote.http11.Http11AprProtocol : 使用APR（Apache portable runtime)技术的连接器,利用Native代码与本地服务器（如linux）来提高性能。
（以上三种Connector实现都是直接处理来自客户端Http请求，加上NIO或者APR）
org.apache.coyote.ajp.AjpProtocol：使用AJP协议的连接器，实现与web server（如Apache httpd）之间的通信
org.apache.coyote.ajp.AjpNioProtocol：SJP协议+ New IO
org.apache.coyote.ajp.AjpAprProtocol：AJP + APR
（这三种实现方法则是与web server打交道，同样加上NIO和APR）
当然，我们可以通过实现ProtocolHandler接口来定义自己的Connector。
```

> #### Connector配置

​	对Connector的配置位于conf/server.xml文件中，内嵌在Service元素中，可以有多个Connector元素。

```xml
BIO HTTP/1.1 Connector配置
<Connector  port=”8080”  protocol=”HTTP/1.1”  maxThreads=”150”  conn ectionTimeout=”20000”   redirectPort=”8443” />
其它一些重要属性如下：
acceptCount : 接受连接request的最大连接数目，默认值是10
address : 绑定IP地址，如果不绑定，默认将绑定任何IP地址
allowTrace : 如果是true,将允许TRACE HTTP方法
compressibleMimeTypes : 各个mimeType, 以逗号分隔，如text/html,text/xml
compression : 如果带宽有限的话，可以用GZIP压缩
connectionTimeout : 超时时间，默认为60000ms (60s)
maxKeepAliveRequest : 默认值是100
maxThreads : 处理请求的Connector的线程数目，默认值为200
```

```xml
NIO HTTP/1.1 Connector配置
<Connector port=”8080” protocol=”org.apache.coyote.http11.Http11NioProtocol” maxThreads=”150” connectionTimeout=”20000” redirectPort=”8443”/>
```

```xml
AJP Connector配置：
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

> #### BIO Http Connector与NIO Http Connector区别

