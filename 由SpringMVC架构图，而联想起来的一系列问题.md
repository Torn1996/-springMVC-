### 一、SpringMVC架构图

> #### SpringMVC架构图
>
> - ​	DispatcherServlet：前端控制器
>
>   用户请求到达前端控制器，它就相当于MVC模式中的C，DispatcherServlet是整个流程控制的中心，由它调用其他的组件处理用户的请求，，DispatcherServlet的存在减低了其他组件的耦合性。
>
> - HandlerMapping：处理器映射器
>
>   HandlerMapping负责根据用户请求URL找到Handler即处理器，SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式、接口实现方式、注解方式等。
>
> - Handler：处理器
>
>   Handler是继承DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的的用户请求进行处理。由于Handler涉及到了具体的用户业务，所以一般情况下需要程序员根据业务开发Handler。
>
> - HandlerAdapter：处理器适配器
>
>   通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
>
> - View Resolver：视图解析器
>
>   View Resolver视图解析器将处理结果解析成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。
>
> - View：视图
>
>   SpringMVC框架提供了很多的View视图类型的支持：包括：jstlView、freemarkerView、PdfView等。我们最常用的视图就是jsp。

![](https://github.com/Torn1996/-springMVC-/blob/master/images/Tomcat%E5%AE%B9%E5%99%A8%E6%A8%A1%E5%9E%8B.png)

### 二、请求是怎么到达DispatcherServlet

> #### Tomcat容器模型
>
> ​	介绍servlet首先就先要把servlet容器介绍清楚，现在市面上比较流行的servlet容器有Tomcat、Jetty，在这里还是以我们比较熟悉的Tomcat为例。Tomcat容器模型还是比较复杂的，在Tomcat的容器等级中，Context容器是直接管理Servlet在容器中的包装类Wrapper,所以Context容器如何运行将直接影响Servlet的工作方式。

![](https://github.com/Torn1996/-springMVC-/blob/master/images/Tomcat%E5%AE%B9%E5%99%A8%E6%A8%A1%E5%9E%8B.png)

​	从上图可以看出Tomcat的容器分成4个等级，真正管理Servlet的容器是Context容器，一个Context容器对应一个Web工程，在Tomcat的配置文件中可以很容易的看出这一点，如下：

```xml
<Context docBase="ssmCrud" path="/ssmCrud" reloadable="true" source="org.eclipse.jst.jee.server:ssmCrud"/></Host>
path 和 docBase 分别代表这个应用在 Tomcat 中的访问路径和这个应用实际的物理路径
```

