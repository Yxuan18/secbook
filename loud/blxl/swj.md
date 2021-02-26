# sqli-with-java

一段较为典型的代码：

```java
http://xxx.com/a.jsp?id=1

HttpServletRequest request,HttpServletResponse ressponse){
JdbcConnection conn = new JdbcConnection();
final String sql = "select * from product where pname like '%"
                +  resuest.getParameter("id") + "%'"
conn.execqueryResultSet(sql);
}
```

上述代码中，百分号没有闭合，导致了注入漏洞的存在

此时的查询语句为：

```java
final String sql = "select * from product where panme l ike '%1' or '%'='%'"
```

 不同状态下可能存在注入的情况：

```java
# 1. 伪静态
http://1223.com/122.html

String id = url.Substring(url.length-9,4);
final String sql = "select *from product where di = " + id
conn.execqueryResultSet(sql);

# 2. 登录框
POST /login.action

username=admin&password=admin

String sql = "select * from usertable where name ='" +username+"' and password = '"   + password + "'"

# 3. header头参数引起的注入
username,cookie,X-Forward-For等

String referer = request.getHeader("referer");
String sql = "update TABLE set referer = '" + referer + "'";

```

###  注入之JDBC

JDBC： 连接数据库最常用的方法

```java
# 1. 直接从request中获取参数然后拼接
String sql = "select * from product where pname like '%" + request.getParameter("name") + "%'";

# 2. 参数由方法传递而来
String sql = "select * from product where pname like '%" + name + "%'";
```

###  注入之Mybatis

```markup
<select>
 select * from table where name like '%$value$%'
<select>

select * from news where id in (${id})

select * from news where title = '新年' order by ${time} asc;
```

 要防止sql注入，要进行预处理，将要传入的参数先用问号占位，后期再用参数值补进去，补进去后数据库只认为传入的值时字符串，而不会解析类似and ,or 之类的逻辑运算，因此可以保证语句的逻辑完整性

而Mybatis直接使用$拼接  
此时需要针对传入的值重新写代码进行过滤，过滤后才能使用

### 注入之Hibernate（HQL）

```java
String hql = "SELECT warehousename FROM Tswaerhouseinformation  wh WHEWE wh.teuint.unitguid= '" +GUID+ "'";
Query query = session.createQuert(sql);   
```

 语句执行后，若其中的参数没有进行过滤，那么这个地方就会存在注入、防御方式最好的就是进行预处理

###  编写安全的SQL语句

```java
Srting hql = "select e frin Tfqeuipmentfaultdetails e where e.equipmentnumber = ?";
Query query = session,createQuery(hql);
query.setString(0, repairNumber.trim());

```





