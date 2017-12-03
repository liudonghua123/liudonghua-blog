---
title: Ajax跨域访问的一种实现方式
tags:
  - cros
id: 558
categories:
  - 学习
date: 2014-11-20 14:11:38
---

有时自己的服务器通过ajax请求其他服务器资源时通常会遇到跨域请求（[CROS](http://www.w3.org/TR/cors/)）问题，这有很多解决方式，如使用IE的[XDomainRequest](http://msdn.microsoft.com/en-us/library/ie/dd573303(v=vs.85).aspx)替代XMLHttpRequest（不跨平台），使用JSON+callback(即JSONP，只能支持json格式数据)，还有一些其他的。。。

<!--more-->

并且注意什么是跨域，如liudonghua.com/some_path访问www.liudonghua.com/other_path或liudonghua.com:port_num/yet_another_path都是属于跨域请求，打开浏览器上的JS Console，如果看到如"XMLHttpRequest cannot load http://liudonghua.com:8080/simpleweb/users. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://liudonghua.com' is therefore not allowed access."错误，那就说明你遇到跨域请求问题了！
这里要实现的是一种在自己原始资源服务器上添加"Access-Control-Allow-Origin"等请求头，然后在客户端的ajax请求中设置"options.crossDomain = true;"(这一步是非常必要的，我当时忘了做这一步，结果客户端只发送preflight的options请求，接下来的GET/POST/PUT/DELETE请求没发送)。
可参考我写的一个小[Demo](https://github.com/liudonghua123/simpleweb)
注意以下高亮显示的代码

```java
package com.liudonghua.tutorials.simpleweb.controller;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import net.sf.json.JSONArray;
import net.sf.json.JSONObject;

import com.liudonghua.tutorials.simpleweb.dao.UserDao;
import com.liudonghua.tutorials.simpleweb.dao.UserDaoImpl;
import com.liudonghua.tutorials.simpleweb.model.User;

/**
 * Servlet implementation class UserController
 */
public class UserController extends HttpServlet {
	private static final long serialVersionUID = 1L;

	private Pattern regexpAllPattern = Pattern.compile("/users");
	private Pattern regexpSinglePattern = Pattern.compile("/users/([0-9]+)");
	private Pattern regexpUserStatisticsPattern = Pattern.compile("/users/statistics");

	private UserDao userDao = new UserDaoImpl();

	private boolean isUserStatisticsRequest(HttpServletRequest request) {
		Matcher matcher;
		String pathInfo = getPathInfo(request);
		matcher = regexpUserStatisticsPattern.matcher(pathInfo);
		if (matcher.find()) {
			return true;
		}
		return false;
	}

	private int isSingleUserInfoRequest(HttpServletRequest request) {
		Matcher matcher;
		String pathInfo = getPathInfo(request);
		matcher = regexpSinglePattern.matcher(pathInfo);
		int id = -1;
		if (matcher.find()) {
			id = Integer.parseInt(matcher.group(1));
		} else {
			matcher = regexpAllPattern.matcher(pathInfo);
			if (matcher.find()) {
				id = 0;
			}
		}

		return id;
	}

	private String getPathInfo(HttpServletRequest request) {
		//		String pathInfo = request.getPathInfo();
				String requestURI = request.getRequestURI();
				String contextPath = request.getContextPath();
				String pathInfo = requestURI.substring(requestURI.lastIndexOf(contextPath) + contextPath.length());
		return pathInfo;
	}

	public static String getBody(HttpServletRequest request) throws IOException {

		String body = null;
		StringBuilder stringBuilder = new StringBuilder();
		BufferedReader bufferedReader = null;

		try {
			InputStream inputStream = request.getInputStream();
			if (inputStream != null) {
				bufferedReader = new BufferedReader(new InputStreamReader(
						inputStream));
				char[] charBuffer = new char[128];
				int bytesRead = -1;
				while ((bytesRead = bufferedReader.read(charBuffer)) > 0) {
					stringBuilder.append(charBuffer, 0, bytesRead);
				}
			} else {
				stringBuilder.append("");
			}
		} catch (IOException ex) {
			throw ex;
		} finally {
			if (bufferedReader != null) {
				try {
					bufferedReader.close();
				} catch (IOException ex) {
					throw ex;
				}
			}
		}

		body = stringBuilder.toString();
		return body;
	}

	/**
	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse
	 *      response)
	 */
	protected void doGet(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		// set HTTP ContentType header of "application/json"
		response.setContentType("application/json");
		response.addHeader("Access-Control-Allow-Origin", "*");
		PrintWriter out = response.getWriter();
		boolean isUserStatisticsRequest = isUserStatisticsRequest(request);
		int id = isSingleUserInfoRequest(request);
		// request user statistics info
		if (isUserStatisticsRequest) {
			Map<String,String> statisticsInfo = userDao.getUserStatisticsInfo();
			JSONObject jsonObject = JSONObject.fromObject(statisticsInfo);
			out.print(jsonObject.toString());
		}
		// request all user info
		else if (id == 0) {
			List<User> users = userDao.getAllUsers();
			JSONArray jsonObject = JSONArray.fromObject(users);
			out.print(jsonObject.toString());
		}
		// request a single user info
		else if (id != -1) {
			User user = userDao.getUserById(id);
			JSONObject jsonObject = JSONObject.fromObject(user);
			out.print(jsonObject.toString());
		}
	}

	/**
	 * @see HttpServlet#doPost(HttpServletRequest request, HttpServletResponse
	 *      response)
	 */
	protected void doPost(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("application/json");
		response.addHeader("Access-Control-Allow-Origin", "*");
		PrintWriter out = response.getWriter();
		String json = getBody(request);
		JSONObject jsonObject = JSONObject.fromObject(json);
		User user = (User) JSONObject.toBean(jsonObject, User.class);
		int id = userDao.addUser(user);
		User insertedUser = userDao.getUserById(id);
		JSONObject userJson = JSONObject.fromObject(insertedUser);
		out.print(userJson.toString());
	}

	/**
	 * @see HttpServlet#doPut(HttpServletRequest, HttpServletResponse)
	 */
	protected void doPut(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("application/json");
		response.addHeader("Access-Control-Allow-Origin", "*");
		PrintWriter out = response.getWriter();
		String json = getBody(request);
		JSONObject jsonObject = JSONObject.fromObject(json);
		User user = (User) JSONObject.toBean(jsonObject, User.class);
		userDao.updateUser(user);
		User insertedUser = userDao.getUserById(user.getId());
		JSONObject userJson = JSONObject.fromObject(insertedUser);
		out.print(userJson.toString());
	}

	/**
	 * @see HttpServlet#doDelete(HttpServletRequest, HttpServletResponse)
	 */
	protected void doDelete(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("application/json");
		response.addHeader("Access-Control-Allow-Origin", "*");
		PrintWriter out = response.getWriter();
		int id = isSingleUserInfoRequest(request);
		if (id > 0) {
			userDao.deleteUser(id);
			out.print("{\"msg\":\"successful\"}");
		}
	}

	@Override
	protected void doOptions(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		// http://stackoverflow.com/questions/1099787/jquery-ajax-post-sending-options-as-request-method-in-firefox
		response.addHeader("Access-Control-Allow-Origin", "*");
		response.addHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS");
		response.addHeader("Access-Control-Max-Age", "86400");
		response.addHeader("Access-Control-Allow-Headers", "User-Agent,Origin,Cache-Control,Content-Type,x-zd,Date,Server,withCredentials");
		super.doOptions(request, response);
	}

}
```

前端ajax请求中添加如下

[js highlight="3"]
$.ajaxPrefilter( function( options, originalOptions, jqXHR ) {
  // ...
  options.crossDomain = true;
});
[/js]

参考资料
1. [no-access-control-allow-origin-header-is-present-on-the-requested-resource](http://stackoverflow.com/questions/20035101/no-access-control-allow-origin-header-is-present-on-the-requested-resource)
2. [no-access-control-allow-origin-header-is-present-on-the-requested-resource-or](http://stackoverflow.com/questions/20433655/no-access-control-allow-origin-header-is-present-on-the-requested-resource-or)
3. [jquery-ajax-post-sending-options-as-request-method-in-firefox](http://stackoverflow.com/questions/1099787/jquery-ajax-post-sending-options-as-request-method-in-firefox)
4. [jquery-cors-content-type-options](http://stackoverflow.com/questions/12320467/jquery-cors-content-type-options)
5\. [jquery-i-get-options-request-instead-of-get](http://stackoverflow.com/questions/1743845/jquery-i-get-options-request-instead-of-get)
6. [disable-preflight-option-request-when-sending-a-cross-domain-request-with-custom](http://stackoverflow.com/questions/16303591/disable-preflight-option-request-when-sending-a-cross-domain-request-with-custom)