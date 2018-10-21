---
layout: post
title: spring读网笔记1:jdcbTemplate,c3p0,dbcp
tags: 码农点滴
categories: 读网笔记
---

今天重新开始点读书笔记之类的，因为没有看实体的书本，跟着教学视频学的，暂时定位为读网笔记吧

1.jdbpTemplate只是一个jdbc的工具，它不是ORM框架，不支持级联的属性，比如类A的实例对象中，包含一个成员变量，该变量是一个类B的实例对象，
用jdbcTemplate.queryObject方法，对类A对应的数据表进行select的时候，是没法通过一些指定类B对象的字段，来达到映射出类B的实例对象的目地的。
可以使用Hibernate或者Mybaits这种ORM框架达到级联映射的目的。

2.数据库连接池是什么：
由于建立数据库连接是一个非常耗时耗资源的行为，所以通过连接池预先同数据库建立一些连接，放在内存中，
应用程序需要建立数据库连接时直接到连接池中申请一个就行，用完后再放回去。   

3.c3p0和dbcp是开源的”数据库连接池“，我自己之前一直用的是dbcp但是都没留意过。
现在常用的开源数据连接池主要有c3p0、dbcp和proxool三种，其中： 
hibernate开发组推荐使用c3p0; 
spring开发组推荐使用dbcp(dbcp连接池有weblogic连接池同样的问题，就是强行关闭连接或数据库重启后，无法reconnect，告诉连接被重置，这个设置可以解决); 
hibernate in action推荐使用c3p0和proxool;

```
dbcp所需jar：commons-dbcp.jar、commons-pool.jar
c3p0所需jar：c3p0-0.9.2.1.jar mchange-commons-java-0.2.3.4.jar
```

c3p0是一个开源的JDBC连接池，它实现了数据源和JNDI绑定，支持JDBC3规范和JDBC2的标准扩展。目前使用它的开源项目有Hibernate，Spring等。   
DBCP(DataBase connection pool),数据库连接池。是 apache 上的一个 java 连接池项目，也是 tomcat 使用的连接池组件。
单独使用dbcp需要3个包：common-dbcp.jar,common-pool.jar,common-collections.jar
