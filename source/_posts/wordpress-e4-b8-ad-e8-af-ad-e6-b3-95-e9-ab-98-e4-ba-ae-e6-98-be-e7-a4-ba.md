---
title: wordpress中语法高亮显示
tags:
  - wordpress
id: 31
categories:
  - 学习
date: 2014-10-08 16:41:53
---

在wordpress中写的文章中粘贴代码最好还是语法高亮实现一下，实现方式是先安装[插件](https://wordpress.org/plugins/tags/code-highlighter)，然后写文章的时候特别处理一下代码部分即可，例如我安装的插件是“SyntaxHighlighter Evolved”，可以使用以下两种方式处理代码部分，注意如果使用第一种[shortcode](http://en.support.wordpress.com/code/posting-source-code/)方式需要在文本模式而非可视化模式下编辑。<!--more-->

为了更好的显示代码最好粘贴代码之前格式化一下，如代码缩进等等，一些常用的online code format工具有[codebeautify](http://codebeautify.org/)、[jsbeautifier](http://jsbeautifier.org/) 等。

插入的代码如下图所示

[![syntax-highlighter_code_example](http://202.203.209.55:8080/wp-content/uploads/2014/10/syntax-highlighter_code_example.png)](http://202.203.209.55:8080/wp-content/uploads/2014/10/syntax-highlighter_code_example.png)

显示的效果如下所示

&nbsp;

1\. 使用shortcode方式高亮显示代码

[code language="css"]
#button {
	font-weight: bold;
	border: 2px solid #fff;
}
[/code]

2\. 使用简洁的[language][/language]方式高亮显示代码，显示效果与上述方式相同

[java]

public class HibernateListener implements ServletContextListener {

	private Configuration config;
	private SessionFactory factory;

	public void contextInitialized(ServletContextEvent event) {
		try {
			URL url = HibernateListener.class.getResource(path);
			config = new Configuration().configure(url);
			factory = config.buildSessionFactory();

			//save the Hibernate session factory into serlvet context
			event.getServletContext().setAttribute(KEY_NAME, factory);
		} catch (Exception e) {
			System.out.println(e.getMessage());
		}
	}
}
[/java]