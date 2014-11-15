---
layout: post
title: 开发工程师应该知道的数据库使用(未完成)
---

在应用中，数据库的资源及其珍贵。尤其数据量级和访问量级较大时，慢查询会拖慢整个应用。在某些情况下，比如电商秒杀活动，数据库的QPS会达到很高的峰值，如果代码未经过优化，随意使用数据库连接，就会导致数据库阻塞，使得应用的响应时间变长，从而带来不可估量的损失。

因此，作为工程师，在开发过程中对于每一个数据库相关的操作都要非常慎重。

#### 预编译(PreparedStatement)



代码的可读性和可维护性

性能

安全性
Statement中使用字符串拼接，很可能被SQL注入攻击；而PreparedStatement先预编译，再填充参数，因此不存在SQL注入。
例如，通过用户名和密码从表中获取用户信息：

	strSQL = "SELECT * FROM users WHERE (name = '" + userName + "') and (pw = '"+ passWord +"');"

如果传入`passWord = '"1' OR '1'='1'"`由于`'1'='1'`永远为真，上述语句可以获得表中任意用户的信息。
使用预编译改写为：

	java.sql.PreparedStatement prep = connection.prepareStatement(
               	 				"SELECT * FROM users WHERE name = ? AND pw = ?");
	prep.setString(1, username);
	prep.setString(2, password);
	prep.executeQuery();



#### 尽量减少数据库连接次数

在一次查询中，大部分时间花在建立连接这一步上，因此在代码中尽量减少连接的次数，选择批量查询，而不是在循环中进行单次查询。


#### 使用连接池



#### 为表添加必要的索引

执行一条SQL时，MySQL内部会进行优化，选择它认为最优的索引。而事实上，在某些情况下（比如同时使用where和order by），MySQL选择的索引并不是最优的，这时通过`ignore index`强制忽略某个索引或者`force index`强制选择某个索引。


#### 使用缓存缓解落地查询的压力

当查询的大部分结果集中在少量数据上时，可以考虑使用缓存，减少直接查询数据库的次数。

#### 从业务角度减少不必要的查询

分库分表，主流程与辅流程分离，使用异步消息的方法更新非核心字段。


[参考文献]

1. [阿里双十一数据库技术](http://www.hellodb.net/2014/02/taobao_1111_database.html)