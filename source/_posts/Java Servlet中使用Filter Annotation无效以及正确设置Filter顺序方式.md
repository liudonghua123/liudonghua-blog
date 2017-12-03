---
title: Java Servlet中使用Filter Annotation无效以及正确设置Filter顺序方式
tags:
  - filter
  - java
  - listener
  - servlet
id: 212
categories:
  - 学习
date: 2014-10-17 20:41:16
---

在Eclipse中创建Web工程时如果选择的的Dynamic web module version >= 3.0 则通过向导创建Servlet、Filter、Listener就会默认使用注解方式，这里的Dynamic web module version其实就对应着Servlet Spec version

<!--more-->

[![create_dynamic_web_project](/resources/2014/10/create_dynamic_web_project.png)](/resources/2014/10/create_dynamic_web_project.png)

Tomcat 7可以支持Servlet 3.0，Tomcat 8可以支持到Servlet 3.1
[![tomcat_version](/resources/2014/10/tomcat_version.png)](/resources/2014/10/tomcat_version.png)

但使用注解就带来了一个问题，没有方法指定对应相同的url-patterns或servlet的Filter加载顺序，因为正常情况下Filter的加载顺序是有Filter在web.xml中定义的顺序确定的，如下的官方文档是没有说明清楚的，到底是filter还是filter-mapping定义顺序
> [The order in which filters are invoked depends on the order in which they are configured in the web.xml file. The first filter in web.xml is the first one invoked during the request, and the last filter in web.xml is the first one invoked during the response. Note the reverse order during the response.](http://docs.oracle.com/cd/B14099_19/web.1012/b14017/filters.htm)
不过后来在这里找到了更加详细的说明
> [The filters are invoked in the order in which filter mappings appear in the filter mapping list of a WAR.](http://docs.oracle.com/javaee/6/tutorial/doc/bnagb.html)
知道了定义顺序再来看下图就很清楚Filter拦截request、response的过程了（左为无Filter，右为有Filter）
[![servlet_filter](/resources/2014/10/servlet_filter.gif)](/resources/2014/10/servlet_filter.gif)

定义Filter的注解WebFilter是没办法定义顺序的，其源码如下
[su_spoiler title="WebFilter.java" style="fancy"]

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface WebFilter {

/**
 * @return description of the Filter, if present
 */
 String description() default "";

/**
 * @return display name of the Filter, if present
 */
 String displayName() default "";

/**
 * @return array of initialization params for this Filter
 */
 WebInitParam[] initParams() default {};

/**
 * @return name of the Filter, if present
 */
 String filterName() default "";

/**
 * @return small icon for this Filter, if present
 */
 String smallIcon() default "";

/**
 * @return the large icon for this Filter, if present
 */
 String largeIcon() default "";

/**
 * @return array of Servlet names to which this Filter applies
 */
 String[] servletNames() default {};

/**
 * A convenience method, to allow extremely simple annotation of a class.
 *
 * @return array of URL patterns
 * @see #urlPatterns()
 */
 String[] value() default {};

/**
 * @return array of URL patterns to which this Filter applies
 */
 String[] urlPatterns() default {};

/**
 * @return array of DispatcherTypes to which this filter applies
 */
 DispatcherType[] dispatcherTypes() default {DispatcherType.REQUEST};

/**
 * @return asynchronous operation supported by this Filter
 */
 boolean asyncSupported() default false;
}
```

[/su_spoiler]

只能定义filterName及urlPatterns(与value相同)，例如

```java
@WebFilter(filterName="SampleFilter1", urlPatterns="/SampleServlet")
public class SampleFilter1 implements Filter
```

通过查阅网上资料，却大多数都是类似于这里的解决方式
[su_spoiler title="how-to-define-servlet-filter-order-of-execution-using-annotations" style="fancy"]

[how-to-define-servlet-filter-order-of-execution-using-annotations](http://stackoverflow.com/questions/6560969/how-to-define-servlet-filter-order-of-execution-using-annotations)

[text gutter="false"]
@WebFilter(filterName="filter1", urlPatterns="/url1/*")
public class Filter1 implements Filter {}

@WebFilter(filterName="filter2", urlPatterns="/url2/*")
public class Filter2 implements Filter {}

<filter-mapping>
    <filter-name>filter1</filter-name>
    <url-pattern />
</filter-mapping>
<filter-mapping>
    <filter-name>filter2</filter-name>
    <url-pattern />
</filter-mapping>
[/text]

[/su_spoiler]

但我实验了还是不行，如下是我的测试代码

[su_spoiler title="SampleServlet.java" style="fancy"]

```java
package com.liudonghua.tutorial.javaweb.annotation;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

/**
 * Servlet implementation class SampleServlet
 */
@WebServlet("/SampleServlet")
public class SampleServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

    /**
     * @see HttpServlet#HttpServlet()
     */
    public SampleServlet() {
        super();
        System.out.println("SampleServlet() invoke");
        // TODO Auto-generated constructor stub
    }

	/**
	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
        System.out.println("doGet() invoke");
      //Get the session object
        HttpSession session = request.getSession(true);
        // Invalidate Session
        session.invalidate(); 

        // set content type and other response header fields first
        response.setContentType("text/html");

        // then write the data of the response
        PrintWriter out = response.getWriter();

        out.println("<HEAD><TITLE> " + "Sample Servlet"
          + "</TITLE></HEAD><BODY>");
        out.println("<h1>Sample Servlet Demo</h1>");
        out.close();
	}

	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
        System.out.println("doPost() invoke");
	}

}

```

[/su_spoiler]

[su_spoiler title="SampleFilter1.java" style="fancy"]

```java
package com.liudonghua.tutorial.javaweb.annotation;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;

/**
 * Servlet Filter implementation class SampleFilter1
 */
@WebFilter(filterName="SampleFilter1", urlPatterns="/SampleServlet")
public class SampleFilter1 implements Filter {

    /**
     * Default constructor.
     */
    public SampleFilter1() {
        // TODO Auto-generated constructor stub
        System.out.println("SampleFilter1() invoke");
    }

	/**
	 * @see Filter#destroy()
	 */
	public void destroy() {
		// TODO Auto-generated method stub
        System.out.println("SampleFilter1.destroy() invoke");
	}

	/**
	 * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
	 */
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		// TODO Auto-generated method stub
		// place your code here
        System.out.println("SampleFilter1.doFilter() before chain.doFilter invoke");

		// pass the request along the filter chain
		chain.doFilter(request, response);
        System.out.println("SampleFilter1.doFilter() after chain.doFilter invoke");
	}

	/**
	 * @see Filter#init(FilterConfig)
	 */
	public void init(FilterConfig fConfig) throws ServletException {
		// TODO Auto-generated method stub
        System.out.println("SampleFilter1.init() invoke");
	}

}
```

[/su_spoiler]

[su_spoiler title="SampleFilter2.java" style="fancy"]

```java
package com.liudonghua.tutorial.javaweb.annotation;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;

/**
 * Servlet Filter implementation class SampleFilter2
 */
@WebFilter(filterName="SampleFilter2", urlPatterns="/SampleServlet")
public class SampleFilter2 implements Filter {

    /**
     * Default constructor.
     */
    public SampleFilter2() {
        // TODO Auto-generated constructor stub
        System.out.println("SampleFilter2() invoke");
    }

	/**
	 * @see Filter#destroy()
	 */
	public void destroy() {
		// TODO Auto-generated method stub
        System.out.println("SampleFilter2.destroy() invoke");
	}

	/**
	 * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
	 */
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		// TODO Auto-generated method stub
		// place your code here
        System.out.println("SampleFilter2.doFilter() before chain.doFilter invoke");

		// pass the request along the filter chain
		chain.doFilter(request, response);
        System.out.println("SampleFilter2.doFilter() after chain.doFilter invoke");
	}

	/**
	 * @see Filter#init(FilterConfig)
	 */
	public void init(FilterConfig fConfig) throws ServletException {
		// TODO Auto-generated method stub
        System.out.println("SampleFilter2.init() invoke");
	}

}
```

[/su_spoiler]

[su_spoiler title="OtherFilter.java" style="fancy"]

```java
package com.liudonghua.tutorial.javaweb.annotation;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;

/**
 * Servlet Filter implementation class OtherFilter
 */
@WebFilter(filterName="OtherFilter", urlPatterns="/SampleServlet")
public class OtherFilter implements Filter {

    /**
     * Default constructor.
     */
    public OtherFilter() {
        // TODO Auto-generated constructor stub
    	System.out.println("OtherFilter() invoke");
    }

	/**
	 * @see Filter#destroy()
	 */
	public void destroy() {
		// TODO Auto-generated method stub
    	System.out.println("OtherFilter.destroy() invoke");
	}

	/**
	 * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
	 */
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		// TODO Auto-generated method stub
		// place your code here

    	System.out.println("OtherFilter.doFilter() before chain.doFilter invoke");
		// pass the request along the filter chain
		chain.doFilter(request, response);
    	System.out.println("OtherFilter.doFilter() after chain.doFilter invoke");
	}

	/**
	 * @see Filter#init(FilterConfig)
	 */
	public void init(FilterConfig fConfig) throws ServletException {
		// TODO Auto-generated method stub
    	System.out.println("OtherFilter.init() invoke");
	}

}
```

[/su_spoiler]

[su_spoiler title="SampleServletContextListener.java" style="fancy"]

```java
package com.liudonghua.tutorial.javaweb.annotation;

import javax.servlet.ServletContextEvent;
import javax.servlet.ServletContextListener;
import javax.servlet.annotation.WebListener;

/**
 * Application Lifecycle Listener implementation class SampleServletContextListener
 *
 */
@WebListener
public class SampleServletContextListener implements ServletContextListener {

    /**
     * Default constructor.
     */
    public SampleServletContextListener() {
        // TODO Auto-generated constructor stub
        System.out.println("SampleServletContextListener() invoke");
    }

	/**
     * @see ServletContextListener#contextDestroyed(ServletContextEvent)
     */
    public void contextDestroyed(ServletContextEvent sce)  {
         // TODO Auto-generated method stub
        System.out.println("SampleServletContextListener.contextDestroyed() invoke");
    }

	/**
     * @see ServletContextListener#contextInitialized(ServletContextEvent)
     */
    public void contextInitialized(ServletContextEvent sce)  {
         // TODO Auto-generated method stub
        System.out.println("SampleServletContextListener.contextInitialized() invoke");
    }

}
```

[/su_spoiler]

[su_spoiler title="web.xml" style="fancy"]

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>web</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>

  <filter-mapping>
    <filter-name>SampleFilter2</filter-name>
    <url-pattern />
  </filter-mapping>
  <filter-mapping>
    <filter-name>OtherFilter</filter-name>
    <url-pattern />
  </filter-mapping>
  <filter-mapping>
    <filter-name>SampleFilter1</filter-name>
    <url-pattern />
  </filter-mapping>
</web-app>
```

[/su_spoiler]

但不管是使用tomcat7还是tomcat8都如如下执行结果

[text gutter="false"]
SampleServletContextListener() invoke
SampleServletContextListener.contextInitialized() invoke
SampleFilter1() invoke
SampleFilter1.init() invoke
OtherFilter() invoke
OtherFilter.init() invoke
SampleFilter2() invoke
SampleFilter2.init() invoke
SampleServlet() invoke
OtherFilter.doFilter() before chain.doFilter invoke
SampleFilter1.doFilter() before chain.doFilter invoke
SampleFilter2.doFilter() before chain.doFilter invoke
doGet() invoke
SampleFilter2.doFilter() after chain.doFilter invoke
SampleFilter1.doFilter() after chain.doFilter invoke
OtherFilter.doFilter() after chain.doFilter invoke
[/text]

理论上的执行结果应该是

[text gutter="false"]
SampleServletContextListener() invoke
SampleServletContextListener.contextInitialized() invoke
SampleFilter1() invoke
SampleFilter1.init() invoke
OtherFilter() invoke
OtherFilter.init() invoke
SampleFilter2() invoke
SampleFilter2.init() invoke
SampleServlet() invoke
SampleFilter2.doFilter() before chain.doFilter invoke
OtherFilter.doFilter() before chain.doFilter invoke
SampleFilter1.doFilter() before chain.doFilter invoke
doGet() invoke
SampleFilter1.doFilter() after chain.doFilter invoke
OtherFilter.doFilter() after chain.doFilter invoke
SampleFilter2.doFilter() after chain.doFilter invoke
[/text]

最后试了很多中方式，但都不行，只能回归原始途径，使用如下的web.xml方式才能得到以上预期结果（Java代码里的@WebFilter可以不注释）
[su_spoiler title="web.xml" style="fancy"]

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>web</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>

<filter>
    <filter-name>SampleFilter1</filter-name>
    <filter-class>com.liudonghua.tutorial.javaweb.annotation.SampleFilter1</filter-class>
  </filter>
  <filter>
    <filter-name>SampleFilter2</filter-name>
    <filter-class>com.liudonghua.tutorial.javaweb.annotation.SampleFilter2</filter-class>
  </filter>
  <filter>
    <filter-name>OtherFilter</filter-name>
    <filter-class>com.liudonghua.tutorial.javaweb.annotation.OtherFilter</filter-class>
  </filter>
    <filter-mapping>
    <filter-name>SampleFilter2</filter-name>
    <url-pattern>/SampleServlet</url-pattern>
  </filter-mapping>
  <filter-mapping>
    <filter-name>OtherFilter</filter-name>
    <url-pattern>/SampleServlet</url-pattern>
  </filter-mapping>
  <filter-mapping>
    <filter-name>SampleFilter1</filter-name>
    <url-pattern>/SampleServlet</url-pattern>
  </filter-mapping>
</web-app>
```

[/su_spoiler]

后面再深入看看这块源码了解一下背后的“真相”