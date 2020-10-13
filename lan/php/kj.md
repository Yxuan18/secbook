# 相关框架简介及漏洞

已知PHP框架有：ThinkPHP与Larvel，简介如下：

## THINKPHP 

ThinkPHP\(FCS\)是一个轻量级的中型框架，是从Java的Struts结构移植过来的中文PHP开发框架。它使用面向对象的开发结构和MVC模式，并且模拟实现了Struts的标签库，各方面都比较人性化，熟悉J2EE的开发人员相对比较容易上手，适合php框架初学者。 ThinkPHP的宗旨是简化开发、提高效率、易于扩展，其在对数据库的支持方面已经包括MySQL、MSSQL、Sqlite、PgSQL、 Oracle，以及PDO的支持。ThinkPHP有着丰富的文档和示例，框架的兼容性较强，但是其功能有限，因此更适合用于中小项目的开发

### 优点

1. 易于上手，有丰富的中文文档；
2. 框架的兼容性较强，PHP4和PHP5完全兼容、完全支持UTF8等。
3. 适合用于中小项目的开发

### 缺点

1. 对Ajax的支持不是很好；
2. 目录结构混乱，需要花时间整理；
3. 上手容易，但是深入学习较难。



## LARAVEL

Laravel 是优雅的 PHP Web 开发框架。具有高效、简洁、富于表达力等优点。采用 MVC 设计，是崇尚开发效率的全栈框架。是最受关注的 PHP 框架

### 优点

Laravel 的设计思想是很先进的，非常适合应用各种开发模式TDD, DDD 和BDD，作为一个框 架，它准备好了一切，composer 是个php 的未来，没有composer，PHP 肯定要走向没落。 laravel 最大的特点和优秀之处就是集合了php 比较新的特性，以及各种各样的设计模式， Ioc 容器，依赖注入等。

### 缺点

基于组件式的框架，所以比较臃肿

## THINKPHP

### Thinkphp 3.x 漏洞 

1. Thinkphp 3.1.3 sql注入漏洞 
2. Thinkphp 3.2.3 update注入漏洞 
3. Thinkphp 3.2.3 select&find&delete 注入漏洞 
4. Thinkphp 3.2.3 缓存漏洞 
5. Thinkphp 3.x order by 注入漏洞 

### Thinkphp 5.x 命令执行漏洞 

1. Thinkphp 5.x 命令执行漏洞说明 
   1. Thinkphp 5.0.1 
   2. Thinkphp 5.0.2 
   3. Thinkphp 5.0.3 
   4. Thinkphp 5.0.4 
   5. Thinkphp 5.0.5 
   6. Thinkphp 5.0.6 
   7. Thinkphp 5.0.7 
   8. Thinkphp 5.0.8 
   9. Thinkphp 5.0.9 
   10. Thinkphp 5.0.10 
   11. Thinkphp 5.0.11 
   12. Thinkphp 5.0.12 
   13. Thinkphp 5.0.13 
   14. Thinkphp 5.0.14 
   15. Thinkphp 5.0.15 
   16. Thinkphp 5.0.16 
   17. Thinkphp 5.0.17 
   18. Thinkphp 5.0.18 
   19. Thinkphp 5.0.19 
   20. Thinkphp 5.0.20 
   21. Thinkphp 5.0.21 
   22. Thinkphp 5.0.22 
   23. Thinkphp 5.0.23 
   24. Thinkphp 5.1.18 
   25. Thinkphp 5.1.29 

### Thinkphp 5.x 漏洞 

1. 5.0.0 &lt;= Thinkphp &lt;=5.0.18 文件包含漏洞 
2. Thinkphp 5.0.5 缓存漏洞 
3. Thinkphp = 5.0.10 sql注入漏洞 
4. 5.0.13 &lt;= Thinkphp &lt;= 5.0.15 sql注入漏洞 
5. 5.0.0 &lt;= Thinkphp &lt;= 5.0.21 sql注入漏洞 
6. Thinkphp 5.0.24 mysql账号密码泄露 
7. 5.1.0 &lt;= ThinkPHP &lt;= 5.1.10 文件包含漏洞 
8. 5.1.0 &lt;= Thinkphp &lt;= 5.1.5 sql注入漏洞 
9. 5.1.6 &lt;= Thinkphp &lt;= 5.1.7（非最新的 5.1.8 版本也可利用）sql注入漏洞 
10. 5.1.16 &lt;= Thinkphp &lt;= 5.1.22 sql注入漏洞 （CVE-2018-16385）
11. Thinkphp &lt; 5.1.23 sql注入漏洞 
12. 5.1.3&lt;=ThinkPHP5&lt;=5.1.25 sql注入漏洞 
13. Thinkphp5 全版本 sql注入漏洞 

### Thinkphp 6.x 漏洞 

1. Thinkphp &lt; 6.0.2 session id未作过滤导致getshell 
2. Thinkphp 6.0 任意文件写入pop链 
3. Thinkphp 6.1 任意文件创建&删除漏洞 

### Thinkphp 反序列化漏洞 

1. Thinkphp 5.0.24 反序列化漏洞 
2. Thinkphp 5.1.1 反序列化pop链构造 
3. Thinkphp 5.1.37 反序列化漏洞 
4. Thinkphp 5.2.-dev 反序列化漏洞 
5. Thinkphp 6.0.-dev 反序列化漏洞

## LARAVEL

1、（CVE-2018-15133）Laravel 反序列化远程命令执行漏洞

2、（CVE-2019-9081）Laravel 5.7 反序列化rce

