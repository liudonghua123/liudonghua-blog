---
title: The last packet successfully received from the server was ...
tags:
  - autoReconnect
  - mysql
  - wait_timeout
id: 574
categories:
  - 学习
date: 2014-11-29 12:27:25
---

我之前部署的一个简单的web服务项目由于长时间没有访问，今天访问时数据没有显示出来，猜想可能是数据库访问模块出现了问题，查看日志，发现错误信息"**The last packet successfully received from the server was** 140,287,058 milliseconds ago.  The last packet sent successfully to the server was 140,287,058 milliseconds ago. **is longer than the server configured value of 'wait_timeout'**. You should consider either expiring and/or testing connection validity before use in your application, **increasing the server configured values** for client timeouts, or using the Connector/J **connection property 'autoReconnect=true'** to avoid this problem."，这个错误信息之前使用数据库连接池时也遇到过，即如果服务器里的数据库连接长时间不用，超过mysql配置的"wait_timeout"（默认为28800s=8h），就会报这个错，从错误信息里就可以看到解决办法，延长"wait_timeout"，或配置"autoReconnect"，但这些方式都不是最好的方式——延长"wait_timeout"会浪费一些资源，配置"autoReconnect"（可以在jdbc_url中设置，如jdbc:mysql://localhost:3306/databasename?useUnicode=true&amp;characterEncoding=utf-8_&amp;_**autoReconnect=true**）只能在下次生效。最好的方式还是在捕获的SQLException中判断是否connection失效，如果是重新建立连接，然后再进行刚才的操作。

<!--more-->

以下是wait_timeout, autoReconnect的官方解释
[![mysql_sysvar_wait_timeout](/resources/2014/11/mysql_sysvar_wait_timeout.png)](/resources/2014/11/mysql_sysvar_wait_timeout.png)
> [<span class="strong">**autoReconnect**</span>](http://dev.mysql.com/doc/connector-j/en/connector-j-reference-configuration-properties.html)
> 
> 
> Should the driver try to re-establish stale and/or dead connections? If enabled the driver will throw an exception for a queries issued on a stale or dead connection, which belong to the current transaction, but will attempt reconnect before the next query issued on the connection in a new transaction. The use of this feature is not recommended, because it has side effects related to session state and data consistency when applications don't handle SQLExceptions properly, and is only designed to be used when you are unable to configure your application to handle SQLExceptions resulting from dead and stale connections properly. Alternatively, as a last option, investigate setting the MySQL server variable "wait_timeout" to a high value, rather than the default of 8 hours.
以下是我修改的方式

```java
......
public int addUser(User user) {
	int id = -1;
	try {
		// http://stackoverflow.com/questions/4246646/mysql-java-get-id-of-the-last-inserted-value-jdbc
		PreparedStatement preparedStatement = connection
				.prepareStatement("insert into users(username,age,salary) values (?, ?, ? )", Statement.RETURN_GENERATED_KEYS);
		preparedStatement.setString(1, user.getUsername());
		preparedStatement.setInt(2, user.getAge());
		preparedStatement.setDouble(3, user.getSalary());
		preparedStatement.executeUpdate();
		ResultSet rs = preparedStatement.getGeneratedKeys();
		if (rs.next()){
		    id=rs.getInt(1);
		}
	} catch (SQLException e) {
		e.printStackTrace();
		// try re-connect is needed
		if(tryReconnectConnectionIfNeeded()) {
			return addUser(user);
		}
	}
	return id;
}
......
private boolean tryReconnectConnectionIfNeeded() {
	try {
		// given 5 seconds timeout to reconnect
		if(!connection.isValid(5)) {
			connection = DatabaseUtil.reconnect();
			return true;
		}
	} catch (SQLException e) {
		e.printStackTrace();
	}
	return false;
}
......
```

 