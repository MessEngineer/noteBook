# Tomcat 是一个servlet容器?

怎么理解，回顾下传统的一个web应用，定义一个符合规范的servlet（**这个符合规范的servlet，就是一个可以处理请求返回响应的web组件，我们通常会有一个或多个专门用于接收请求的servlet，接收完之后分配给其他业务逻辑处理，类似于Spring MVC和Spring Boot中的DispatcherServlet）**，web.xml中加上这么一个配置，就可以部署应用路由一个url请求，再把这个web工程打成一个war包丢进Tomcat的webapps目录下，启动Tomcat的Bootstrap类，就可以接受这个url请求了

```java
public class ServletExample extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws 		 			ServletException, IOException {
        resp.getWriter().print("Welcome To com.servlet.ServletExample");
    }
}
```

```xml
	<servlet>
        <servlet-name>ServletExample</servlet-name>
        <servlet-class>com.servlet.ServletExample</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>ServletExample</servlet-name>
        <url-pattern>/example</url-pattern>
    </servlet-mapping>
	
```

类似于Spring是Bean的容器，管理了所有bean的生命周期，那么Tomcat作为servlet的容器，同样的除了装载servlet，也同时管理了servlet的生命周期，Spring要去加载解析所有的bean，同样Tomcat也需要，两个容器源码解读时可以进行横向对比。

## Tomcat中Servlet的容器层级结构

```java
Engine
	|--- Host
		|--- Context
			|--- Wrapper
```



- **Wrapper**

  servlet的包装类，存在的意义？一个Wrapper对应一个Servlet，这么来想的话，确实不需要Wrapper，但是我们还要考虑一些其他的情况：

  - 比如Filter，一个Filter是可以对应一个Servlet的。
  - 比如ServletPool，通常的Servlet是所有请求线程公用的，但是在Servlet中支持每一个请求线程单独使用独立的Servlet实例（是不是又有点类似Spring的原型Bean和单例Bean了）

  所以在Wrapper中，不仅仅只包括一个Servlet，还包括过滤器和Servlet池，所以**Wrapper是第一层Servlet容器**。

- **Context**

  Tomcat的应用上下文，管理一个或多个servlet，**第二层servlet容器**

- **Host**

  一个Tomcat中可以部署多个应用，每个Host可以用来配置对应web应用域名，**第三层servlet容器**

- **Engine**

  Host管理Context，Context管理Wrapper，Wrapper管理Servlet，而Engine就是用来管理Host的。**所以Engine是第四层容器。**

### 对应到Tomcat server.xml中的配置，更直观

```xml
<Engine name="Catalina" defaultHost="localhost">

      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
        <context>
          <servlet>
            <servlet-name>ServletExample</servlet-name>
            <servlet-class>com.servlet.ServletExample</servlet-class>
          </servlet>

          <servlet-mapping>
            <servlet-name>ServletExample</servlet-name>
            <url-pattern>/example</url-pattern>
          </servlet-mapping>
        </context>
    </Host>
</Engine>
```

