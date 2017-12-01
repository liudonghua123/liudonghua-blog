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

在Eclipse中创建Web工程时如果选择的的Dynamic web module version &gt;= 3.0 则通过向导创建Servlet、Filter、Listener就会默认使用注解方式，这里的Dynamic web module version其实就对应着Servlet Spec version<!--more-->
[![create_dynamic_web_project](http://202.203.209.55:8080/wp-content/uploads/2014/10/create_dynamic_web_project.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/create_dynamic_web_project.png)

Tomcat 7可以支持Servlet 3.0，Tomcat 8可以支持到Servlet 3.1
[![tomcat_version](http://202.203.209.55:8080/wp-content/uploads/2014/10/tomcat_version.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/tomcat_version.png)

但使用注解就带来了一个问题，没有方法指定对应相同的url-patterns或servlet的Filter加载顺序，因为正常情况下Filter的加载顺序是有Filter在web.xml中定义的顺序确定的，如下的官方文档是没有说明清楚的，到底是filter还是filter-mapping定义顺序
> [The order in which filters are invoked depends on the order in which they are configured in the web.xml file. The first filter in web.xml is the first one invoked during the request, and the last filter in web.xml is the first one invoked during the response. Note the reverse order during the response.](http://docs.oracle.com/cd/B14099_19/web.1012/b14017/filters.htm)
不过后来在这里找到了更加详细的说明
> [The filters are invoked in the order in which filter mappings appear in the filter mapping list of a WAR.](http://docs.oracle.com/javaee/6/tutorial/doc/bnagb.html)
知道了定义顺序再来看下图就很清楚Filter拦截request、response的过程了（左为无Filter，右为有Filter）
[![servlet_filter](http://202.203.209.55:8080/wp-content/uploads/2014/10/servlet_filter.gif)](http://202.203.209.55:8080/wp-content/uploads/2014/10/servlet_filter.gif)

定义Filter的注解WebFilter是没办法定义顺序的，其源码如下
[su_spoiler title="WebFilter.java" style="fancy"]

[java]
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface WebFilter {

/**
 * @return description of the Filter, if present
 */
 String description() default &quot;&quot;;

/**
 * @return display name of the Filter, if present
 */
 String displayName() default &quot;&quot;;

/**
 * @return array of initialization params for this Filter
 */
 WebInitParam[] initParams() default {};

/**
 * @return name of the Filter, if present
 */
 String filterName() default &quot;&quot;;

/**
 * @return small icon for this Filter, if present
 */
 String smallIcon() default &quot;&quot;;

/**
 * @return the large icon for this Filter, if present
 */
 String largeIcon() default &quot;&quot;;

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
[/java]

[/su_spoiler]

只能定义filterName及urlPatterns(与value相同)，例如

[java padlinenumbers="true" gutter="false"]
@WebFilter(filterName=&quot;SampleFilter1&quot;, urlPatterns=&quot;/SampleServlet&quot;)
public class SampleFilter1 implements Filter
[/java]

通过查阅网上资料，却大多数都是类似于这里的解决方式
[su_spoiler title="how-to-define-servlet-filter-order-of-execution-using-annotations" style="fancy"]

[how-to-define-servlet-filter-order-of-execution-using-annotations](http://stackoverflow.com/questions/6560969/how-to-define-servlet-filter-order-of-execution-using-annotations)

[text gutter="false"]
@WebFilter(filterName=&quot;filter1&quot;, urlPatterns=&quot;/url1/*&quot;)
public class Filter1 implements Filter {}

@WebFilter(filterName=&quot;filter2&quot;, urlPatterns=&quot;/url2/*&quot;)
public class Filter2 implements Filter {}

&lt;filter-mapping&gt;
    &lt;filter-name&gt;filter1&lt;/filter-name&gt;
    &lt;url-pattern /&gt;
&lt;/filter-mapping&gt;
&lt;filter-mapping&gt;
    &lt;filter-name&gt;filter2&lt;/filter-name&gt;
    &lt;url-pattern /&gt;
&lt;/filter-mapping&gt;
[/text]

[/su_spoiler]

但我实验了还是不行，如下是我的测试代码

[su_spoiler title="SampleServlet.java" style="fancy"]

[java highlight="16"]
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
@WebServlet(&quot;/SampleServlet&quot;)
public class SampleServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;

    /**
     * @see HttpServlet#HttpServlet()
     */
    public SampleServlet() {
        super();
        System.out.println(&quot;SampleServlet() invoke&quot;);
        // TODO Auto-generated constructor stub
    }

	/**
	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
        System.out.println(&quot;doGet() invoke&quot;);
      //Get the session object
        HttpSession session = request.getSession(true);
        // Invalidate Session
        session.invalidate(); 

        // set content type and other response header fields first
        response.setContentType(&quot;text/html&quot;);

        // then write the data of the response
        PrintWriter out = response.getWriter();

        out.println(&quot;&lt;HEAD&gt;&lt;TITLE&gt; &quot; + &quot;Sample Servlet&quot;
          + &quot;&lt;/TITLE&gt;&lt;/HEAD&gt;&lt;BODY&gt;&quot;);
        out.println(&quot;&lt;h1&gt;Sample Servlet Demo&lt;/h1&gt;&quot;);
        out.close();
	}

	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
        System.out.println(&quot;doPost() invoke&quot;);
	}

}

[/java]

[/su_spoiler]

[su_spoiler title="SampleFilter1.java" style="fancy"]

[java highlight="16"]
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
@WebFilter(filterName=&quot;SampleFilter1&quot;, urlPatterns=&quot;/SampleServlet&quot;)
public class SampleFilter1 implements Filter {

    /**
     * Default constructor.
     */
    public SampleFilter1() {
        // TODO Auto-generated constructor stub
        System.out.println(&quot;SampleFilter1() invoke&quot;);
    }

	/**
	 * @see Filter#destroy()
	 */
	public void destroy() {
		// TODO Auto-generated method stub
        System.out.println(&quot;SampleFilter1.destroy() invoke&quot;);
	}

	/**
	 * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
	 */
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		// TODO Auto-generated method stub
		// place your code here
        System.out.println(&quot;SampleFilter1.doFilter() before chain.doFilter invoke&quot;);

		// pass the request along the filter chain
		chain.doFilter(request, response);
        System.out.println(&quot;SampleFilter1.doFilter() after chain.doFilter invoke&quot;);
	}

	/**
	 * @see Filter#init(FilterConfig)
	 */
	public void init(FilterConfig fConfig) throws ServletException {
		// TODO Auto-generated method stub
        System.out.println(&quot;SampleFilter1.init() invoke&quot;);
	}

}
[/java]

[/su_spoiler]

[su_spoiler title="SampleFilter2.java" style="fancy"]

[java highlight="16"]
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
@WebFilter(filterName=&quot;SampleFilter2&quot;, urlPatterns=&quot;/SampleServlet&quot;)
public class SampleFilter2 implements Filter {

    /**
     * Default constructor.
     */
    public SampleFilter2() {
        // TODO Auto-generated constructor stub
        System.out.println(&quot;SampleFilter2() invoke&quot;);
    }

	/**
	 * @see Filter#destroy()
	 */
	public void destroy() {
		// TODO Auto-generated method stub
        System.out.println(&quot;SampleFilter2.destroy() invoke&quot;);
	}

	/**
	 * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
	 */
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		// TODO Auto-generated method stub
		// place your code here
        System.out.println(&quot;SampleFilter2.doFilter() before chain.doFilter invoke&quot;);

		// pass the request along the filter chain
		chain.doFilter(request, response);
        System.out.println(&quot;SampleFilter2.doFilter() after chain.doFilter invoke&quot;);
	}

	/**
	 * @see Filter#init(FilterConfig)
	 */
	public void init(FilterConfig fConfig) throws ServletException {
		// TODO Auto-generated method stub
        System.out.println(&quot;SampleFilter2.init() invoke&quot;);
	}

}
[/java]

[/su_spoiler]

[su_spoiler title="OtherFilter.java" style="fancy"]

[java highlight="16"]
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
@WebFilter(filterName=&quot;OtherFilter&quot;, urlPatterns=&quot;/SampleServlet&quot;)
public class OtherFilter implements Filter {

    /**
     * Default constructor.
     */
    public OtherFilter() {
        // TODO Auto-generated constructor stub
    	System.out.println(&quot;OtherFilter() invoke&quot;);
    }

	/**
	 * @see Filter#destroy()
	 */
	public void destroy() {
		// TODO Auto-generated method stub
    	System.out.println(&quot;OtherFilter.destroy() invoke&quot;);
	}

	/**
	 * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
	 */
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		// TODO Auto-generated method stub
		// place your code here

    	System.out.println(&quot;OtherFilter.doFilter() before chain.doFilter invoke&quot;);
		// pass the request along the filter chain
		chain.doFilter(request, response);
    	System.out.println(&quot;OtherFilter.doFilter() after chain.doFilter invoke&quot;);
	}

	/**
	 * @see Filter#init(FilterConfig)
	 */
	public void init(FilterConfig fConfig) throws ServletException {
		// TODO Auto-generated method stub
    	System.out.println(&quot;OtherFilter.init() invoke&quot;);
	}

}
[/java]

[/su_spoiler]

[su_spoiler title="SampleServletContextListener.java" style="fancy"]

[java]
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
        System.out.println(&quot;SampleServletContextListener() invoke&quot;);
    }

	/**
     * @see ServletContextListener#contextDestroyed(ServletContextEvent)
     */
    public void contextDestroyed(ServletContextEvent sce)  {
         // TODO Auto-generated method stub
        System.out.println(&quot;SampleServletContextListener.contextDestroyed() invoke&quot;);
    }

	/**
     * @see ServletContextListener#contextInitialized(ServletContextEvent)
     */
    public void contextInitialized(ServletContextEvent sce)  {
         // TODO Auto-generated method stub
        System.out.println(&quot;SampleServletContextListener.contextInitialized() invoke&quot;);
    }

}
[/java]

[/su_spoiler]

[su_spoiler title="web.xml" style="fancy"]

[java]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;
&lt;web-app xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot; xmlns=&quot;http://java.sun.com/xml/ns/javaee&quot; xsi:schemaLocation=&quot;http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd&quot; id=&quot;WebApp_ID&quot; version=&quot;3.0&quot;&gt;
  &lt;display-name&gt;web&lt;/display-name&gt;
  &lt;welcome-file-list&gt;
    &lt;welcome-file&gt;index.html&lt;/welcome-file&gt;
    &lt;welcome-file&gt;index.htm&lt;/welcome-file&gt;
    &lt;welcome-file&gt;index.jsp&lt;/welcome-file&gt;
    &lt;welcome-file&gt;default.html&lt;/welcome-file&gt;
    &lt;welcome-file&gt;default.htm&lt;/welcome-file&gt;
    &lt;welcome-file&gt;default.jsp&lt;/welcome-file&gt;
  &lt;/welcome-file-list&gt;

  &lt;filter-mapping&gt;
    &lt;filter-name&gt;SampleFilter2&lt;/filter-name&gt;
    &lt;url-pattern /&gt;
  &lt;/filter-mapping&gt;
  &lt;filter-mapping&gt;
    &lt;filter-name&gt;OtherFilter&lt;/filter-name&gt;
    &lt;url-pattern /&gt;
  &lt;/filter-mapping&gt;
  &lt;filter-mapping&gt;
    &lt;filter-name&gt;SampleFilter1&lt;/filter-name&gt;
    &lt;url-pattern /&gt;
  &lt;/filter-mapping&gt;
&lt;/web-app&gt;
[/java]

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

[java]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;
&lt;web-app xmlns:xsi=&quot;http://www.w3.org/2001/XMLSchema-instance&quot; xmlns=&quot;http://java.sun.com/xml/ns/javaee&quot; xsi:schemaLocation=&quot;http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd&quot; id=&quot;WebApp_ID&quot; version=&quot;3.0&quot;&gt;
  &lt;display-name&gt;web&lt;/display-name&gt;
  &lt;welcome-file-list&gt;
    &lt;welcome-file&gt;index.html&lt;/welcome-file&gt;
    &lt;welcome-file&gt;index.htm&lt;/welcome-file&gt;
    &lt;welcome-file&gt;index.jsp&lt;/welcome-file&gt;
    &lt;welcome-file&gt;default.html&lt;/welcome-file&gt;
    &lt;welcome-file&gt;default.htm&lt;/welcome-file&gt;
    &lt;welcome-file&gt;default.jsp&lt;/welcome-file&gt;
  &lt;/welcome-file-list&gt;

&lt;filter&gt;
    &lt;filter-name&gt;SampleFilter1&lt;/filter-name&gt;
    &lt;filter-class&gt;com.liudonghua.tutorial.javaweb.annotation.SampleFilter1&lt;/filter-class&gt;
  &lt;/filter&gt;
  &lt;filter&gt;
    &lt;filter-name&gt;SampleFilter2&lt;/filter-name&gt;
    &lt;filter-class&gt;com.liudonghua.tutorial.javaweb.annotation.SampleFilter2&lt;/filter-class&gt;
  &lt;/filter&gt;
  &lt;filter&gt;
    &lt;filter-name&gt;OtherFilter&lt;/filter-name&gt;
    &lt;filter-class&gt;com.liudonghua.tutorial.javaweb.annotation.OtherFilter&lt;/filter-class&gt;
  &lt;/filter&gt;
    &lt;filter-mapping&gt;
    &lt;filter-name&gt;SampleFilter2&lt;/filter-name&gt;
    &lt;url-pattern&gt;/SampleServlet&lt;/url-pattern&gt;
  &lt;/filter-mapping&gt;
  &lt;filter-mapping&gt;
    &lt;filter-name&gt;OtherFilter&lt;/filter-name&gt;
    &lt;url-pattern&gt;/SampleServlet&lt;/url-pattern&gt;
  &lt;/filter-mapping&gt;
  &lt;filter-mapping&gt;
    &lt;filter-name&gt;SampleFilter1&lt;/filter-name&gt;
    &lt;url-pattern&gt;/SampleServlet&lt;/url-pattern&gt;
  &lt;/filter-mapping&gt;
&lt;/web-app&gt;
[/java]

[/su_spoiler]

后面再深入看看这块源码了解一下背后的“真相”