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

### SQL注入

说明：SQL注入是指原始SQL查询被动态更改成一个与程序预期完全不同的查询。执行这样一个更改后的查询可能导致信息泄露或者数据被篡改。防止SQL注入的方式主要可以分为两类：

* 使用参数化查询
* 对不可信数据进行校验

参数化查询是一种简单有效的防止SQL注入的查询方式，应该被优先考虑使用。另外，参数化查询还能改进数据库访问的性能，例如，SQL Server与Oracle数据库会为其缓存一个查询计划，以便在多次重复执行相同的查询语句时重复使用。

错误示例（Java代码动态构建SQL）：

```java
Statement stmt = null;
ResultSet rs = null;
try
{
    String userName = ctx.getAuthenticatedUserName(); //this is a constant
    String sqlString = "SELECT * FROM t_item WHERE owner='" + userName + "' AND itemName='" + request.getParameter("itemName") + "'";
    stmt = connection.createStatement();
    rs = stmt.executeQuery(sqlString);
    // ... result set handling
}
catch (SQLException se)
{
    // ... logging and error handling
}
```

这里将查询字符串常量与用户输入进行拼接来动态构建SQL查询命令。仅当itemName不包含单引号时，这条查询语句的行为才会是正确的。如果一个攻击者以用户名wiley发起一个请求，并使用以下条目名称参数进行查询：

`name' OR 'a' = 'a`

那么这个查询将变成：

`SELECT * FROM t_item WHERE owner = 'wiley' AND itemname = 'name' OR 'a'='a';`

此处，额外的OR 'a'='a'条件导致整个WHERE子句的值总为真。那么，这个查询便等价于如下非常简单的查询：

`SELECT * FROM t_item`

这个简化的查询使得攻击者能够绕过原有的条件限制：这个查询会返回items表中所有储存的条目，而不管它们的所有者是谁，而原本应该只返回属于当前已认证用户的条目。

正确示例（使用PreparedStatement进行参数化查询）:

```java
PreparedStatement stmt = null
ResultSet rs = null
try
{
    String userName = ctx.getAuthenticatedUserName(); //this is a constant
    String itemName = request.getParameter("itemName");
    // ...Ensure that the length of userName and itemName is legitimate
    // ...
    String sqlString = "SELECT * FROM t_item WHERE owner=? AND itemName=?";
    stmt = connection.prepareStatement(sqlString);
    stmt.setString(1, userName);
    stmt.setString(2, itemName);
    rs = stmt.executeQuery();
    // ... result set handling
}
catch (SQLException se)
{
    // ... logging and error handling
}
```

如果使用参数化查询，则在SQL语句中使用占位符表示需在运行时确定的参数值。参数化查询使得SQL查询的语义逻辑被预先定义，而实际的查询参数值则等到程序运行时再确定。参数化查询使得数据库能够区分SQL语句中语义逻辑和数据参数，以确保用户输入无法改变预期的SQL查询语义逻辑。在Java中，可以使用java.sql.PreparedStatement来对数据库发起参数化查询。在这个正确示例中，如果一个攻击者将itemName输入为name' OR 'a' = 'a，这个参数化查询将免受攻击，而是会查找一个itemName匹配name' OR 'a' = 'a这个字符串的条目。

错误示例（在存储过程中动态构建SQL）:

Java代码：

```java
CallableStatement = null
ResultSet results = null;
try
{
    String userName = ctx.getAuthenticatedUserName(); //this is a constant
    String itemName = request.getParameter("itemName");
    cs = connection.prepareCall("{call sp_queryItem(?,?)}");
    cs.setString(1, userName);
    cs.setString(2, itemName);
    results = cs.executeQuery();
    // ... result set handling
}
catch (SQLException se)
{
    // ... logging and error handling
}
```

SQL Server存储过程：

```java
CREATE PROCEDURE sp_queryItem
    @userName varchar(50),
    @itemName varchar(50) 
AS 
BEGIN 
    DECLARE @sql nvarchar(500); 
    SET @sql = 'SELECT * FROM t_item 
                WHERE owner = ''' + @userName + '''
                AND itemName = ''' + @itemName + '''';
    EXEC(@sql); 
END 
GO
```

在存储过程中，通过拼接参数值来构建查询字符串，和在应用程序代码中拼接参数一样，同样是有SQL注入风险的。

正确示例（在存储过程中进行参数化查询）:

Java 代码：

```java
CallableStatement = null
ResultSet results = null;
try
{
    String userName = ctx.getAuthenticatedUserName(); //this is a constant
    String itemName = request.getParameter("itemName");
    // ... Ensure that the length of userName and itemName is legitimate
    // ... 
    cs = connection.prepareCall("{call sp_queryItem(?,?)}");
    cs.setString(1, userName);
    cs.setString(2, itemName);
    results = cs.executeQuery();
    // ... result set handling
}
catch (SQLException se)
{
    // ... logging and error handling
}
```

SQL Server存储过程：

```java
CREATE PROCEDURE sp_queryItem
    @userName varchar(50), 
    @itemName varchar(50) 
AS 
BEGIN 
    SELECT * FROM t_item  
    WHERE userName = @userName
    AND itemName = @itemName; 
END 
GO
```

这个存储过程使用参数化查询，而未包含不安全的动态SQL构建。数据库编译此存储过程时，会生成一个SELECT查询的执行计划，只允许原始的SQL语义被执行。任何参数值，即使是被注入的SQL语句也不会被执行，因为它们不是执行计划的一部分。

错误示例（Hibernate: 动态构建SQL/HQL）：

原生SQL查询：

```java
String userName = ctx.getAuthenticatedUserName(); //this is a constant
String itemName = request.getParameter("itemName");
Query sqlQuery = session.createSQLQuery("select * from t_item where owner = '" + userName + "' and itemName = '" + itemName + "'");
List<Item> rs = (List<Item>) sqlQuery.list();
```

HQL查询：

```java
String userName = ctx.getAuthenticatedUserName(); //this is a constant
String itemName = request.getParameter("itemName");
Query hqlQuery = session.createQuery("from Item as item where item.owner = '" + userName + "' and item.itemName = '" + itemName + "'");
List<Item> hrs = (List<Item>) hqlQuery.list();
```

即使是使用Hibernate，如果在动态构建SQL/HQL查询时包含了不可信输入，同样也会面临SQL/HQL注入的问题。

正确示例（Hibernate: 参数化查询）：

HQL中基于位置的参数化查询：

```java
String userName = ctx.getAuthenticatedUserName(); //this is a constant
String itemName = request.getParameter("itemName");
Query hqlQuery = session.createQuery("from Item as item where item.owner = ? and item.itemName = ?");
hqlQuery.setString(1, userName);
hqlQuery.setString(2, itemName);
List<Item> rs = (List<Item>) hqlQuery.list();
```

HQL中基于名称的参数化查询：

```java
String userName = ctx.getAuthenticatedUserName(); //this is a constant
String itemName = request.getParameter("itemName");
Query hqlQuery = session.createQuery("from Item as item where item.owner = :owner and item.itemName = :itemName");
hqlQuery.setString("owner", userName);
hqlQuery.setString("itemName", itemName);
List<Item> rs = (List<Item>) hqlQuery.list();
```

原生参数化查询：

```java
String userName = ctx.getAuthenticatedUserName(); //this is a constant
String itemName = request.getParameter("itemName");
Query sqlQuery = session.createSQLQuery("select * from t_item where owner = ? and itemName = ?");
sqlQuery.setString(0, owner);
sqlQuery.setString(1, itemName);
List<Item> rs = (List<Item>) sqlQuery.list();
```

Hibernate支持SQL/HQL参数化查询。为了防止SQL注入以及改善性能，以上这些示例使用了参数化绑定的方式来设置查询参数。

正反示例（iBATIS SQL映射）:

iBATIS SQL映射允许在SQL语句中通过\#字符指定动态参数，例如：

```java
<select id="getItems" parameterClass="MyClass" resultClass="Item">
	SELECT * FROM t_item WHERE owner = #userName# AND itemName = #itemName#
</select>
#符号括起来的userName和itemName两个参数指示iBATIS在创建参数化查询时将它们替换成占位符：
String sqlString = "SELECT * FROM t_item WHERE owner=? AND itemName=?";
PreparedStatement stmt = connection.prepareStatement(sqlString);
stmt.setString(1, myClassObj.getUserName());
stmt.setString(2, myClassObj.getItemName());
ResultSet rs = stmt.executeQuery();
// ... convert results set to Item objects
```

然而，iBATIS也允许使用$符号指示使用某个参数来直接拼接SQL语句，这种做法是有SQL注入漏洞的：（order by 只能用$,用\#{}会多个' '导致sql语句失效.此外还有一个like 语句后也需要用${}，这俩语句需要单独对传入参数做过滤）

```java
<select id="getItems" parameterClass="MyClass" resultClass="items">
	SELECT * FROM t_item WHERE owner = #userName# AND itemName = '$itemName$'
</select>
```

iBATIS将会为以上SQL映射执行类似下面的代码：

```java
String sqlString = "SELECT * FROM t_item WHERE owner=? AND itemName='" + myClassObj.getItemName() + "'";
PreparedStatement stmt = connection.prepareStatement(sqlString);
stmt.setString(1, myClassObj.getUserName());
ResultSet rs = stmt.executeQuery();
// ... convert results set to Item objects
```

在这里，攻击者可以利用itemName参数发起SQL注入攻击。

正确示例（对不可信输入做校验）:

```java
public List<Book> queryBooks(List<Expression> queryCondition)
{
    /* ... */
    try
    {
        StringBuilder sb = new StringBuilder("select * from t_book where ");
        Codec oe = new OracleCodec();
        if (queryCondition != null && !queryCondition.isEmpty())
        {
            for (Expression e : queryCondition)
            {
                String exprString = e.getColumn() + e.getOperator() + e.getValue();
                String safeExpr = ESAPI.encoder().encodeForSQL(oe, exprString);
                sb.append(safeExpr).append(" and ");
            }
            sb.append("1=1");
            Statement stat = connection.createStatement();
            ResultSet rs = stat.executeQuery(sb.toString());
            //other omitted code     
        }
    }
    /* ... */
}
```

虽然参数化查询是防止SQL注入最便捷有效的一种方式。但是对于一条SQL语句，不是其任何部分在执行前都能够被占位符所替代，参数化查询无法应用于所有场景（有关SQL语句执行前可被占位符替代的元素，请参考你所使用数据库的帮助文档）。当使用执行前不可被占位符替代的不可信数据来动态构建SQL语句时，如同上面的示例代码，必须要对不可信数据进行校验，此例代码中是做转义。每种DBMS都有其特定的转义机制，通过这种机制来告诉数据库此输入应该被当做数据，而不应该是代码逻辑。因此，只要输入数据被适当转义，就不会发生SQL注入的问题。对于一些常用数据库中需要注意的特殊字符，请参考附录A。此示例代码使用了ESAPI来做转义。ESAPI （OWASP企业安全应用程序接口）是一个免费、开源的、网页应用程序安全控件库，程序员使用它能够更简单方便得写出低风险的程序。ESAPI为大部分常用的数据库提供了安全的转义实现，推荐大家使用。

###  webgoat



