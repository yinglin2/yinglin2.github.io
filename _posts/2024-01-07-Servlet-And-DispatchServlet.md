---
title: Servlet and DispatcherServlet
date: 2024-01-07 20:15:00 +0800
categories: [Spring]
tags: [sping_mvc, servlet]

---




# Servlet

Anyone have experience with creating Java web application must have encounter Tomcat. Tomcat is basically a web server and a servlet and JSP container. A web server can serve static content of a website, while servlet and JSP are responsible for rendering dynamic content for a website. After users sending requests to a website, the web server receive the request, call a servlet class to deal with the request, and then sending the reponse back. 

Here is the Java code of Servlet. As you can see, servlet is an interface. Usually, we implement this interface and write code to dealing with clients' requests. 

```java
public interface Servlet {
    public void init(ServletConfig config) throws ServletException;
    public ServletConfig getServletConfig();
    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;
    public String getServletInfo();
    public void destroy();
}
```

- public void init(ServletConfig config) throws ServletException;

`ServletConfig` is a servlet configuraion obejct and it contains information in the `web.xml` file. Tomcat parses this xml file,  save the configuration information in `ServletConfig`, and then pass it as an argument in init function. In `init` method, we can get configuration information from `ServletConfig`. 

```xml
<web-app>
    <servlet>
      <servlet-name>HelloWorld</servlet-name>
      <servlet-class>HelloWorldServlet</servlet-class>
      <init-param>
          <param-name>username</param-name>
          <param-value>helloworld</param-value>
      </init-param>
    </servlet>

    <servlet-mapping>
      <servlet-name>HelloWorld</servlet-name>
      <url-pattern>/hello-world</url-pattern>
    </servlet-mapping>
</web-app>
```

- service(ServletRequest req, ServletResponse res) throws ServletException, IOException;

After Tomcat receving a request, it will put client information into `ServletRequest`  object and create a `ServletResponse`  object for us to put reponse information. Tomcat passes these two objects,  `ServletRequest` and `ServletResponse` , as argument to the servlet's service method. The service method is the place to deal with clients' request and generate a reponse. 



# DispatcherServlet

As metioned previously, tomcat is a web server and a servlet and JSP container. In Spring MVC, tomcat only used as a web server. Spring MVC will be responsible for dealing with HTTP requests and responses and more. 

Spring MVC, as many other web frameworks, is designed around the front controller pattern where a central `Servlet`, the `DispatcherServlet`, provides a shared algorithm for request processing, while actual work is performed by configurable delegate components. This model is flexible and supports diverse workflows.

The `DispatcherServlet`, as any `Servlet`, needs to be declared and mapped according to the Servlet specification by using Java configuration or in `web.xml`. In turn, the `DispatcherServlet` uses Spring configuration to discover the delegate components it needs for request mapping, view resolution, exception handling, [and more](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/special-bean-types.html).

When creating a Spring MVC web project, there are two ways to create DispatcherServlet obejct. 

1. Implements `WebApplicationInitializer`

```
public class MyWebApplicationInitializer implements WebApplicationInitializer {

	@Override
	public void onStartup(ServletContext servletContext) {

		// Load Spring web application configuration
		AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
		context.register(AppConfig.class);

		// Create and register the DispatcherServlet
		DispatcherServlet servlet = new DispatcherServlet(context);
		ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
		registration.setLoadOnStartup(1);
		registration.addMapping("/app/*");
	}
}
```



2. implements `AbstractAnnotationConfigDispatcherServletInitializer`

```java
public class SpringMvcInit extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MvcConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}

```

When running the Spring MVC program, it will automatically initializes `WebApplicationInitializer`or `AbstractAnnotationConfigDispatcherServletInitializer` objects and run `onStartup` method.



# AbstractAnnotationConfigDispatcherServletInitializer

![image-20240107130053949](/assets/img/posts/image-20240107130053949.png)

```java
public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer {
	...
	public void onStartup(ServletContext servletContext) throws ServletException {
        super.onStartup(servletContext);
        this.registerDispatcherServlet(servletContext);
    }
    ...
}
    
```

```java
public abstract class AbstractContextLoaderInitializer implements WebApplicationInitializer {
	...
	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		registerContextLoaderListener(servletContext);
	}
	
	protected void registerContextLoaderListener(ServletContext servletContext) {
		WebApplicationContext rootAppContext = createRootApplicationContext();
		if (rootAppContext != null) {
			ContextLoaderListener listener = new ContextLoaderListener(rootAppContext);
			listener.setContextInitializers(getRootApplicationContextInitializers());
			servletContext.addListener(listener);
		}
		else {
			logger.debug("No ContextLoaderListener registered, as " +
					"createRootApplicationContext() did not return an application context");
		}
	}
}
```

```java
public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer {
	...
	public void onStartup(ServletContext servletContext) throws ServletException {
        super.onStartup(servletContext);
        this.registerDispatcherServlet(servletContext);
    }
    
    protected void registerDispatcherServlet(ServletContext servletContext) {
        String servletName = this.getServletName();
        Assert.state(StringUtils.hasLength(servletName), "getServletName() must not return null or empty");
        // create an ioc container
        WebApplicationContext servletAppContext = this.createServletApplicationContext();
        Assert.state(servletAppContext != null, "createServletApplicationContext() must not return null");
        // crate a dispatchservlet object
        FrameworkServlet dispatcherServlet = this.createDispatcherServlet(servletAppContext);
        Assert.state(dispatcherServlet != null, "createDispatcherServlet(WebApplicationContext) must not return null");
        dispatcherServlet.setContextInitializers(this.getServletApplicationContextInitializers());
        
        ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
        if (registration == null) {
            throw new IllegalStateException("Failed to register servlet with name '" + servletName + "'. Check if there is another servlet registered under the same name.");
        } else {
            registration.setLoadOnStartup(1);
            registration.addMapping(this.getServletMappings());
            registration.setAsyncSupported(this.isAsyncSupported());
            Filter[] filters = this.getServletFilters();
            if (!ObjectUtils.isEmpty(filters)) {
                Filter[] var7 = filters;
                int var8 = filters.length;

                for(int var9 = 0; var9 < var8; ++var9) {
                    Filter filter = var7[var9];
                    this.registerServletFilter(servletContext, filter);
                }
            }

            this.customizeRegistration(registration);
        }
    }
    ...
}
    
```

```java
public abstract class AbstractAnnotationConfigDispatcherServletInitializer extends AbstractDispatcherServletInitializer {
	protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        Class<?>[] configClasses = this.getServletConfigClasses();
        if (!ObjectUtils.isEmpty(configClasses)) {
            context.register(configClasses);
        }

        return context;
    }
}
```



**Reference**

https://docs.spring.io/spring-framework/reference/web/webmvc.html
https://www.zhihu.com/question/21416727/answer/690289895

https://zhuanlan.zhihu.com/p/65658315