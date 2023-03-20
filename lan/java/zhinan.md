# 代码审计指南

1\.	通用代码审计思路


常见的代码审计思路有以下四种：

### 1.1.	根据敏感关键词回溯参数传递过程。&#xD;

根据敏感函数来逆向追踪参数的传递过程，是目前使用得最多的一种方式，因为大多数漏洞是由于函数的使用不当造成的。另外非函数使用不当的漏洞，如SQL注入，也有一些特征，比如Select、Insert等，再结合From和Where等关键字，我们就可以判断这是否是一条SQL语句，通过对字符串的识别分析，就能判断这个SQL语句是否使用了参数化的查询方式，如果没有是否在传递过程中有必要的过滤。

这种方式的优点是只需搜索相应敏感关键字，即可以快速地挖掘想要的漏洞，具有可定向挖掘和高效、高质量的优点。大部分代码审计工具，也是根据这种思路，进行关键字匹配，发现漏洞的。

其缺点为由于没有通读代码，对程序的整体框架了解不够深入，在挖掘漏洞时定位利用点会花费一点时间，另外对逻辑漏洞挖掘覆盖不到。

### 1.2.	查找可控变量，正向追踪变量传递过程。&#xD;

前面提到的根据敏感关键字来回溯传入的参数，是一种逆向追踪的思路，我们也提到了这种方式的优缺点，实际上在需要快速寻找漏洞的情况下用回溯参数的方式是非常有效的。正向追踪变量传递过程，通常是在渗透和代码审计结合着做的时候，或是是复现公开漏洞的时候，在已经找到了漏洞利用点的情况下，来逐步跟进变量的传递过程，确定漏洞代码位置。

### 1.3.	寻找敏感功能点，通读功能点代码。&#xD;

在有了一定的代码审计经验之后，一定会知道哪些功能点通常会有哪些漏洞，在我们想要快速挖掘漏洞的时候就可以这样来做，首先安装好并且运行程序，到处点点，浏览一下，看下程序有哪些功能，这些功能的程序文件分别是怎么样的，是独立的模块还是以插件形式存在，或者是写在一个通用类里面，然后多处调用。在了解这些功能的存在形式后，可以先寻找经常会出问题的功能点，简单黑盒测试一下，如果没有发现很普通、很常见的漏洞，再去读这个功能的代码，这样我们读起来就可以略过一些刚才黑盒测试过的点，提高审计速度。

根据经验，我们来简单介绍几个功能点经常会出现的漏洞，如下所示：

a)	文件上传功能。这里说的文件上传在很多功能点都会出现，比如像文章编辑、资料编辑、头像上传、附件上传，这个功能最常见的漏洞就是任意文件上传了，后端程序没有严格地限制上传文件的格式，导致可以直接上传或者存在绕过的情况，而除了文件上传功能外，还经常发生SQL注入漏洞。因为一般程序员都不会注意到对文件名进行过滤，但是又需要把文件名保存到数据库中，所以就会存在SQL注入漏洞。



b)	文件管理功能。在文件管理功能中，如果程序将文件名或者文件路径直接在参数中传递，则很有可能会存在任意文件操作的漏洞，比如任意文件读取等，利用的方式是在路径中使用../或者..\跳转目录。

除了任意文件操作漏洞以外，还可能会存在XSS漏洞，程序会在页面中输出文件名，而通常会疏忽对文件名进行过滤，导致可以在数据库中存入带有尖括号等特殊符号的文件名，最后显示在页面上的时候就会被执行。

c)	登录认证功能。登录认证功能不是指一个登录过程，而是整个操作过程中的认证，目前的认证方式大多是基于Cookie和Session，不少程序会把当前登录的用户账号等认证信息放到Cookie中，或许是加密方式，是为了保持用户可以长时间登录，不会一退出浏览器或者Session超时就退出账户，进行操作的时候直接从Cookie中读取出当前用户信息，这里就存在一个算法可信的问题，如果这段Cookie信息没有加salt一类的东西，就可以导致任意用户登录漏洞，只要知道用户的部分信息，即可生成认证令牌，甚至有的程序会直接把用户名明文放到Cookie中，操作的时候直接取这个用户名的数据，这也是常说的越权漏洞。



d)	交易支付功能。交易支付过程中，会涉及到金额的计算，这块如果没有做数值为正的校验，或者对支付结果都校验不严格，就可能存在支付漏洞。



### 1.4.	直接通读全文代码&#xD;

如果代码审计的时间充足或者通过关键词回溯未能发现足够的风险点，我们需要了解整个应用的业务逻辑，才能挖掘到更多更有价值的漏洞。

通读全文代码也有一定的技巧，并不是随便找文件逐个读完就可以了，这样你是很难真正读懂这套Web程序的，也很难理解代码的业务逻辑，首先我们要看程序的大体代码结构，用了什么开发框架，持久层框架是mybatis还是Hibernate，等等。如主目录有哪些文件，模块目录有哪些文件，插件目录有哪些文件，除了关注有哪些文件，还要注意文件的大小、创建时间。我们根据这些文件的命名就可以大致知道这个程序实现了哪些功能，核心文件是哪些。utils工具类，controller类，各个功能接口的实现类，配置文件等都属于重点部分。

2\.	数据校验类漏洞


### 2.1.	SQL注入&#xD;

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

iBATIS SQL映射允许在SQL语句中通过#字符指定动态参数，例如：

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

然而，iBATIS也允许使用$符号指示使用某个参数来直接拼接SQL语句，这种做法是有SQL注入漏洞的：（order by 只能用$,用#{}会多个' '导致sql语句失效.此外还有一个like 语句后也需要用${}，这俩语句需要单独对传入参数做过滤）

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

### 2.2.	XSS&#xD;

说明：Cross Site Script（XSS），跨站脚本攻击。

攻击者利用应用程序的动态展示数据功能，在 html 页面里嵌入恶意代码。当用户浏览该页之时，这些嵌入在 html中的恶意代码会被执行，用户浏览器被攻击者控制，从而达到攻击者的特殊目的。

跨站脚本攻击有三种攻击形式

1、反射型跨站脚本攻击

攻击者会通过社会工程学手段，发送一个 URL 连接给用户打开，在用户打开页面的同时，浏览器会执行页面中嵌入的恶意脚本。

2、存储型跨站脚本攻击

攻击者利用 web 应用程序提供的录入或修改数据功能，将数据存储到服务器或用户 cookie 中，当其他用户浏览展示该数据的页面时，浏览器会执行页面中嵌入的恶意脚本。

所有浏览者都会受到攻击。

3、DOM 跨站攻击

由于 html 页面中，定义了一段 JS，根据用户的输入，显示一段 html 代码，攻击者可以在输入时，插入一段恶意脚本，最终展示时，会执行恶意脚本。

DOM 跨站和以上两个跨站攻击的差别是，DOM 跨站是纯页面脚本的输出，只有规范使用 JAVASCRIPT，才可以防御。

恶意攻击者可以利用跨站脚本攻击做到：

1、盗取用户 cookie，伪造用户身份登录。

2、控制用户浏览器。

3、结合浏览器及其插件漏洞，下载病毒木马到浏览者的计算机上执行。

4、衍生 URL 跳转漏洞。

5、让官方网站出现钓鱼页面。

6、蠕虫攻击

错误示例：

直接在 html 页面展示“用户可控数据”，将直接导致跨站脚本威胁。

```markup
<input type="text" value="<%= getParameter("keyword") %>">
<button>搜索</button>
<div>
  您搜索的关键词是：<%= getParameter("keyword") %>
</div>
```

代码中的keyword变量，被直接输出到了页面中，没有做任何安全过滤，一旦让用户可以输入数据，都可能导致用户浏览器把“用户可控数据”当成 JS/VBS 脚本执行，或页面元素被“用户可控数据”插入的页面 HTML 代码控制，从而造成攻击。

正确示例（进行escapeHTML编码）：

```markup
<input type="text" value="<%= escapeHTML(getParameter("keyword")) %>">
<button>搜索</button>
<div>
  您搜索的关键词是：<%= escapeHTML(getParameter("keyword")) %>
</div>
```

修复方案：

HTML/XML 页面输出规范：

1， 在 HTML/XML 中显示“用户可控数据”前，应该进行 html escape 转义。

JAVA 示例：

```java
<div>#escapeHTML($user.name) </div>
<td>#escapeHTML($user.name)</td>
```

所有 HTML 和 XML 中输出的数据，都应该做 html escape 转义。

escapeHTML 函数参考 esapi 实现：

> http://code.google.com/p/owasp-esapi-java/source/browse/trunk/src/main/java/org/owasp/esapi/codecs/HTMLEntityCodec.java>

escapeHTML 需要进行 html 转义应该按照以下列表进行转义

> &	--> \&amp;> \
> <	--> \&lt;&#x20;> \
> \> --> \&gt;> \
> " --> \&quot;&#x20;> \
> ' --> \&#39;>

2，在 javascript 内容中输出的“用户可控数据”，需要做 javascript escape 转义。

html 转义并不能保证在脚本执行区域内数据的安全，也不能保证脚本执行代码的正

常运行。

JAVA 示例：

```java
<script>alert('#escapeJavaScript($user.name)')</script> <script>x='#escapeJavaScript($user.name)'</script>
<div onmouseover="x='#escapeJavaScript($user.name)'"</div> 
```

需要转义的字符包括

> /	-->  \\/> \
> '	--> \\' " --> \\"> \
> \  -->  \\\\>

escapeJavaScript 函数参考 esapi 实现：

> http://code.google.com/p/owasp-esapi-java/source/browse/trunk/src/main/java/org/o> wasp/esapi/codecs/JavaScriptCodec.java>

3，对输出到富文本中的“用户可控数据”，做富文本安全过滤（允许用户输出 HTML 的情况）。

在服务器检查敏感的HTML代码，有基于黑名单和基于白名单的两种过滤方式。因为HTML标签种类繁多，基于黑名单的过滤方法考虑的并不全面。而且对伪协议的考虑也不全面等等。所以推荐使用基于白名单的过滤方法，只保留合法的标签及内容。

4，输出在 url 中的数据，做 url 安全输出。

一些 html 标签的属性，需要，如果接收“用户可控数据”，需要做安全检查。以下属性的值，如果是用户可控数据，需要做安全检查

```
'action', 'background', 'codebase', 'dynsrc', 'href', 'lowsrc', 'src',
```

这些属性的值，一般都是一个 URL，如果整串 URL 都是由“用户可控数据”组成的，则必须满足以下条件：

1）以“http”开头

```java
char[] uc = url.toCharArray();
if(uc[0] != 'h' || uc[1] != 't' || uc[2] != 't' || uc[3] != 'p'){
return "";
 
}
```

2）转义“用户可控数据”中的以下字符

> < --> %3C> \
> \> --> %3E> \
> " --> %22> \
> ' --> %27>

5，针对 DOM 跨站的解决方案，是javascript 的编码问题&#x20;

在使用 .innerHTML、.outerHTML、document.write() 时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用 .textContent、.setAttribute() 等。

如果用 Vue/React 技术栈，并且不使用 v-html/dangerouslySetInnerHTML 功能，就在前端 render 阶段避免 innerHTML、outerHTML 的 XSS 隐患。

DOM 中的内联事件监听器，如 location、onclick、onerror、onload、onmouseover 等，\<a> 标签的 href 属性，JavaScript 的 eval()、setTimeout()、setInterval() 等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些 API，很容易产生安全隐患，请务必避免。

```java
<!-- 内联事件监听器中包含恶意代码 -->
<img onclick="UNTRUSTED" onerror="UNTRUSTED" src="data:image/png,">

<!-- 链接内包含恶意代码 -->
<a href="UNTRUSTED">1</a>

<script>
// setTimeout()/setInterval() 中调用恶意代码
setTimeout("UNTRUSTED")
setInterval("UNTRUSTED")

// location 调用恶意代码
location.href = 'UNTRUSTED'

// eval() 中调用恶意代码
eval("UNTRUSTED")
</script>
```

如果项目中有用到这些的话，一定要避免在字符串中拼接不可信数据。

6，在给用户设置认证 COOKIE 时，加入 HTTPONLY。

7，在 style 内容中输出的“用户可控数据”，需要做 CSS escape 转义。

举例使用：

```java
String safe = ESAPI.encoder().encodeForCSS( request.getParameter("input") );
```

encodeForCSS 实现代码参考：

http://code.google.com/p/owasp-esapi-java/source/browse/trunk/src/main/java/org/o wasp/esapi/codecs/CSSCodec.java

AJAX 输出规范：

1、XML 输出“用户可控数据”时，对数据部分做 HTML 转义。

示例：

```markup
<?xml version="1.0" encoding="UTF-8" ?>
<man>
<name>#xmlEscape($name)</name>
<man>
```

2、json 输出要先对变量内容中的“用户可控数据”单独作 htmlEscape，再对变量内容做一次 javascriptEscape。

```markup
String cityname=”浙江<B>”+StringUtil.htmlEscape(city.name)+”</B>”;
String json =
"citys:{city:['"+
StringUtil.javascript(cityname) +
"']}";
```

3、非 xml 输出（包括 json、其他自定义数据格式），response 包中的 http 头的contentType，必须为 json，并且用户可控数据做 htmlEscape 后才能输出。

```java
response.setContentType("application/json"); 
PrintWriter out = response.getWriter(); out.println(StringUtil.htmlEscape(ajaxReturn));
```

### 2.3.	XML注入&#xD;

说明：使用不可信数据来构造XML会导致XML注入漏洞。一个用户，如果他被允许输入结构化的XML片段，则他可以在XML的数据域中注入XML标签来改写目标XML文档的结构与内容。XML解析器会对注入的标签进行识别和解释。

错误示例（未检查的输入）：

```java
private void createXMLStream(BufferedOutputStream outStream, User user) throws IOException
{
    String xmlString;
    xmlString = "<user><role>operator</role><id>" + user.getUserId()
                + "</id><description>" + user.getDescription() + "</description></user>";
    outStream.write(xmlString.getBytes());
    outStream.flush();
}
```

某个恶意用户可能会使用下面的字符串作为用户ID：

`"joe</id><role>administrator</role><id>joe"`

并使用如下正常的输入作为描述字段：

`"I want to be an administrator"`

最终，整个XML字符串将变成如下形式：

```markup
<user>
    <role>operator</role>
    <id>joe</id>
    <role>administrator</role>
    <id>joe</id>
    <description>I want to be an administrator</description>
</user>
```

由于SAX解析器（org.xml.sax and javax.xml.parsers.SAXParser）在解释XML文档时会将第二个role域的值覆盖前一个role域的值，因此导致此用户角色由操作员提升为了管理员。

错误示例（XML Schema或者 DTD校验 ）:

```java
private void createXMLStream(BufferedOutputStream outStream, User user)
            throws IOException
{
    String xmlString;
    xmlString = "<user><id>" + user.getUserId()
            + "</id><role>operator</role><description>"
            + user.getDescription() + "</description></user>";
    
    StreamSource xmlStream = new StreamSource(new StringReader(xmlString));
    
    // Build a validating SAX parser using the schema
    SchemaFactory sf = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
    StreamSource ss = new StreamSource(new File("schema.xsd"));
    try
    {
        Schema schema = sf.newSchema(ss);
        Validator validator = schema.newValidator();
        validator.validate(xmlStream);
    }
    catch (SAXException x)
    {
        throw new IOException("Invalid userId", x);
    }      
    // the XML is valid, proceed
    outStream.write(xmlString.getBytes());
    outStream.flush();
}
```

如下是 schema.xsd 文件中的schema定义：

```markup
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">
<xs:element name="user">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="id" type="xs:string"/>
      <xs:element name="role" type="xs:string"/>
      <xs:element name="description" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
</xs:schema>
```

某个恶意用户可能会使用下面的字符串作为用户ID：

`"joe</id><role>Administrator</role><!—"`

并使用如下字符串作为描述字段：

`"--><description>I want to be an administrator"`

最终，整个XML字符串将变成如下形式：

```markup
<user>
	<id>joe</id>
	<role>Administrator</role><!--</id> <role>operator</role> <description> -->
	<description>I want to be an administrator</description>
</user>
```

用户ID结尾处的“\<!--”和描述字段开头处的“-->”将会注释掉原本硬编码在XML字符串中的角色信息。虽然用户角色已经被攻击者篡改成管理员类型，但是整个XML字符串仍然可以通过schema的校验。XML schema或者DTD校验仅能确保XML的格式是有效的，但是攻击者可以在不打破原有XML格式的情况下，对XML的内容进行篡改。

正确示例（白名单校验）：

```java
private void createXMLStream(BufferedOutputStream outStream, User user) throws IOException
{
    // Write XML string if userID contains alphanumeric and underscore characters only
    if (!Pattern.matches("[_a-bA-B0-9]+", user.getUserId()))
    {
        // Handle format violation
    }
    if (!Pattern.matches("[_a-bA-B0-9]+", user.getDescription()))
    {
        // Handle format violation
    }
    String xmlString = "<user><id>" + user.getUserId()
            + "</id><role>operator</role><description>"
            + user.getDescription() + "</description></user>";
    outStream.write(xmlString.getBytes());
    outStream.flush();
}
```

这个方法使用白名单的方式对输入进行清理，要求输入的userId字段中只能包含字母、数字或者下划线。

正确示例（使用安全的XML库）：

```java
public static void buidlXML(FileWriter writer, User user) throws IOException
{        
    Document userDoc = DocumentHelper.createDocument();
    Element userElem = userDoc.addElement("user");
    Element idElem = userElem.addElement("id");
    idElem.setText(user.getUserId());
    Element roleElem = userElem.addElement("role");
    roleElem.setText("operator");
    Element descrElem = userElem.addElement("description");
    descrElem.setText(user.getDescription());
    XMLWriter output  = null;
    try
    {
        OutputFormat format = OutputFormat.createPrettyPrint();
        format.setEncoding("UTF-8");
        output = new XMLWriter(writer, format);
        output.write(userDoc); 
        output.flush();
    }
    finally
    {
       try
       {
            output.close();                
       }
       catch (Exception e)
       {
           // handle exception
       }
    }
}
```

这个正确示例使用dom4j来构建XML，dom4j是一个良好定义的、开源的XML工具库。Dom4j将会对文本数据域进行XML编码，从而使得XML的原始结构和格式免受破坏。

这个例子中，如果攻击者输入如下字符串作为用户ID：

`"joe</id><role>Administrator</role><!—"`

以及使用如下字符串作为描述字段：

`"--><description>I want to be an administrator"`

则最终会生成如下格式的XML：

```markup
<user>
  <id>joe&lt;/id&gt;&lt;role&gt;Administrator&lt;/role&gt;&lt;!—</id>
  <role>operator</role>
  <description>--&gt;&lt;description&gt;I want to be an administrator</description>
</user>
```

可以看到，“<”与“>”经过XML编码被分别替换成了“\&lt”与“\&gt”，导致攻击者未能将其角色类型从操作员提升到管理员。

### 2.4.	日志伪造&#xD;

说明：如果在记录的日志中包含未经校验的不可信数据，则可能导致日志注入漏洞。恶意用户会插入伪造的日志数据，从而让系统管理员误以为这些日志数据是由系统记录的。例如，一个用户可能通过输入一个回车符和一个换行符（CRLF）序列来将一条合法日志拆分成两条日志，其中每一条都可能会令人误解。将未经净化的用户输入写入日志还可能会导致向信任边界之外泄露敏感数据，或者导致违反当地法律法规，在日志中写入和存储了某些类型的敏感数据。例如，如果一个用户要把一个未经加密的信用卡号插入到日志文件中，那么系统就会违反了PCI DSS（Payment Card Industry Data Security Standard）标准。可以通过验证和净化发送到日志的任何不可信数据来防止日志注入攻击。

错误示例：

```java
if (loginSuccessful)
{
    logger.severe("User login succeeded for: " + username);
}
else
{
    logger.severe("User login failed for: " + username);
}
```

此错误示例代码中，在接收到非法请求时，会记录用户的用户名。它没有执行任何输入净化，在这种情况下可能遭受生日志注入攻击。当username字段的值是david时，会生成一条标准的日志信息：

> May 15, 2011 2:19:10 PM java.util.logging.LogManager$RootLogger log> \
> SEVERE: User login failed for: david>

但是，如果记录日志时使用的username字段不是david，而是多行字符串时，如下所示：

> david> \
> May 15, 2011 2:25:52 PM java.util.logging.LogManager$RootLogger log> \
> SEVERE: User login succeeded for: administrator>

那么日志中包含了以下可能引起误导的信息：

> May 15, 2011 2:19:10 PM java.util.logging.LogManager$RootLogger log> \
> SEVERE: User login failed for: david> \
> May 15, 2011 2:25:52 PM java.util.logging.LogManager log> \
> SEVERE: User login succeeded for: administrator>

正确示例：

```java
if (!Pattern.matches("[A-Za-z0-9_]+", username))
{
    // Unsanitized username
    logger.severe("User login failed for unauthorized user");
}
else if (loginSuccessful)
{
    logger.severe("User login succeeded for: " + username);
}
else
{
    logger.severe("User login failed for: " + username);
}
```

这个正确示例在登录之前会对用户名输入进行净化，从而防止注入攻击

### 2.5.	命令执行&#xD;

说明：在执行任意系统命令或者外部程序时使用了未经校验的不可信输入，就会导致产生命令和参数注入漏洞。对于命令注入漏洞，命令将会以与Java应用程序相同的特权级别执行，它向攻击者提供了类似系统shell的功能。在Java中，Runtime.exec()经常被用来调用一个新的进程，但是这个调用并不会通过命令行Shell来执行。因此，无法通过链接或者管道的方式来连续执行多个命令。但是，如果在Runtime.exec()中使用命令行shell（例如，cmd.exe 或 /bin/sh）来调用一个程序，则可被命令注入。当通过Runtime.exec()来运行bat或者sh脚本时，命令行shell将自动被调用。例如，Runtime.getRuntime().exec("test.bat & notepad.exe")，由于bat文件默认是由命令行解释器cmd.exe来解释执行的，这里的“&”符号将会被cmd.exe当做一个命令分隔符，从而导致test.bat与notepad.exe都将会被执行。当参数中包含空格，双引号，以-或者/符号开头表示一个参数开关时，可能会导致参数注入漏洞。与命令和参数注入相关的特殊符号的详细信息，请参考附录B

错误示例：

```java
class DirList
{
    public static void main(String[] args)
    {
        if (args.length == 0)
        {
            System.out.println("No arguments");
            System.exit(1);
        }
        try
        {
            Runtime rt = Runtime.getRuntime();
            Process proc = rt.exec("cmd.exe /c dir " + args[0]);
            // ...
        }
        catch (Exception e)
        {
            // Handle errors
        }
    }
}
```

攻击者可以通过以下命令来利用这个漏洞程序：

`java DirList "dummy & echo bad"`

实际将会执行两个命令：

```bash
dir dummy 
echo bad
```

这会试图列举一个不存的文件夹dummy中的文件，然后往控制台输出bad。

正确示例（避免使用 Runtime.exec()）：

```java
class DirList
{
    public static void main(String[] args)
    {
        if (args.length == 0)
        {
            System.out.println("No arguments");
            System.exit(1);
        }
        try
        {
            File dir = new File(args[0]);
            if (!validate(dir)) // the dir need to be validated
            {
                System.out.println("An illegal directory");
            }
            else
            {
                for (String file : dir.list())
                {
                    System.out.println(file);
                }
            }
        }
        catch (Exception e)
        {
            System.out.println("An unexpected exception");
        }
      }
      // Other omitted code…
}
```

如果可以使用标准的API替代运行系统命令来完成任务，则应该使用标准的API。这个正确示例使用File.list()方法来列举目录下的内容，而不是调用Runtime.exec()来运行一个外部进程，从而消除了发生命令注入与参数注入的可能。

正确示例（数据校验）:

```java
// ...
if (!Pattern.matches("[0-9A-Za-z@]+", dir))
{
    // Handle error
}
// ...
```

如果无法避免使用Runtime.exec()，则必须要对输入数据进行检查和净化，防止其中包含命令分隔符，管道，或者重定向操作符（“&&”，“&”，“||”，“|”，“>”，“>>”）等，用于连续执行多个命令或者重定向输入输出，或者是包含参数开关符（“-”，“/”）来发起参数注入攻击。

### 2.6.	CSRF&#xD;

说明：Cross-Site Request Forgery（CSRF），跨站请求伪造攻击。

攻击者在用户浏览网页时，利用页面元素（例如 img 的 src），强迫受害者的浏览器向 Web 应用程序发送一个改变用户信息的请求。

由于发生 CSRF 攻击后，攻击者是强迫用户向服务器发送请求，所以会造成用户信息被迫修改，更严重者引发蠕虫攻击。

CSRF 攻击可以从站外和站内发起。从站内发起 CSRF 攻击，需要利用网站本身的业务，比如“自定义头像”功能，恶意用户指定自己的头像 URL 是一个修改用户信息的链接，当其他已登录用户浏览恶意用户头像时，会自动向这个链接发送修改信息请求。

从站外发送请求，则需要恶意用户在自己的服务器上，放一个自动提交修改个人信息的 htm 页面，并把页面地址发给受害者用户，受害者用户打开时，会发起一个请求。

如果恶意用户能够知道网站管理后台某项功能的 URL，就可以直接攻击管理员，强迫管理员执行恶意用户定义的操作。

错误示例：

```java
HttpServletRequest request, HttpServletResponse response) {
int userid=Integer.valueOf( request.getSession().getAttribute("userid").toString()); String email=request.getParameter("email");
 
String tel=request.getParameter("tel");
String realname=request.getParameter("realname"); Object[] params = new Object[4]; params[0] = email;
params[1] = tel;
params[2] = realname;
params[3] = userid;
final String sql = "update user set email=?,tel=?,realname=? where userid=?"; conn.execUpdate(sql,params);
```

代码中接收用户提交的参数“email,tel,realname”，之后修改了该用户的数据，一旦接收到一个用户发来的请求，就执行修改操作。

提交表单代码：

```markup
<form action="http://localhost/servlet/modify" method="POST">
<input name="email">
<input name="tel">
<input name="realname">
<input name="userid">
<input type="submit">
</form>
```

当用户点提交时，就会触发修改操作。

解决方案：

要防御 CSRF 攻击，必须遵循一下三步：

1、 在用户登陆时，设置一个 CSRF 的随机 TOKEN，同时种植在用户的 cookie 中，当用户浏览器关闭、或用户再次登录、或退出时，清除 token。

2、 在表单中，生成一个隐藏域，它的值就是 COOKIE 中随机 TOKEN。

3、 表单被提交后，就可以在接收用户请求的 web 应用中，判断表单中的 TOKEN 值是否和用户 COOKIE 中的 TOKEN 值一致，如果不一致或没有这个值，就判断为 CSRF 攻击，同时记录攻击日志（日志内容见“Error Handing and Logging” 章节）。由于攻击者无法预测每一个用户登录时生成的那个随机 TOKEN 值，所以无法伪造这个参数。

### 2.7.	代码注入&#xD;

说明：Code injection，代码注入攻击

web 应用代码中，允许接收用户输入一段代码，之后在 web 应用服务器上执行这段代码，并返回给用户。

由于用户可以自定义输入一段代码，在服务器上执行，所以恶意用户可以写一个远程控制木马，直接获取服务器控制权限，所有服务器上的资源都会被恶意用户获取和修改，甚至可以直接控制数据库。

错误代码：

servlet 代码：

```java
public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
response.setContentType("text/html");
PrintWriter out = response.getWriter();
try {
File file = File.createTempFile("JavaRuntime", ".java", new File( System.getProperty("user.dir")));
String filename = file.getName();
String classname = filename.substring(0, filename.length() - 5); 
String[] args = new String[] { "-d",System.getProperty("user.dir"), filename };
PrintWriter outfile = new PrintWriter(new FileOutputStream(file));
 
outfile.write("public class " + classname +	"{public void myfun(String args)" + "{try {" +	request.getParameter("code") +	"} catch (Exception e) {}}}");
outfile.flush();
outfile.close();
(new Main()).compile(args, outfile); URL url = new URL(file:// +	file.getPath().substring(0, file.getPath().lastIndexOf("\\") + 1)); java.net.URLClassLoader myloader = new URLClassLoader(
new URL[] { url }, Thread.currentThread().getContextClassLoader());
Class cls = myloader.loadClass(classname);
cls.getMethod("myfun", new Class[] { String.class }).invoke(cls.newInstance(), new Object[] { "" });
} catch (Exception se) { se.printStackTrace();
}
out.println();
out.flush();
out.close();
}
```

接收用户输入的 code 参数内容，编译为一个 class 文件，之后调用这个文件相关代码。

解决方案：

尽可能不要使用可以直接执行代码的函数。如果必须使用，执行代码的参数，或文件名，禁止和用户输入相关，只能由开发人员定义代码内容，用户只能提交“1、2、3”参数，代表相应代码。必须用户输入时，对用户的输入进行严格过滤。



### 2.8.	 URL跳转攻击&#xD;

说明：URL redirect，URL 跳转攻击。

Web 应用程序接收到用户提交的 URL 参数后，没有对参数做“可信任 URL”的验证，就向用户浏览器返回跳转到该 URL 的指令。

如果 360.cn下的某个 web 应用程序存在这个漏洞，恶意攻击者可以发送给用户一个 360.cn的链接，但是用户打开后，却来到钓鱼网站页面，将会导致用户被钓鱼攻击，账号被盗，或账号相关财产被盗。

错误示例：

这是一段没有验证目的地址，就直接跳转的经典代码：

```java
if(checklogin(request)){
response.sendRedirect(request.getParameter("url"));
}
```

这段代码存在 URL 跳转漏洞，当用户登陆成功后，会跳转到 url 参数所指向的地址。

解决方案：

为了保证用户所点击的 URL，是从 web 应用程序中生成的 URL，所以要做 TOKEN 验证。

1、当用户访问需要生成跳转 URL 的页面时，首先生成随机 token，并放入 cookie。

2、在显示连接的页面上生成 URL，在 URL 参数中加入 token。示例：\
http://www.360.com/member/sigin.htm?done=http://www.360.com\&token=23123213123123

3、 应用程序在跳转前，判断 token 是否和 cookie 中的 token 一致，如果不一致，就判定为 URL 跳转攻击，并记录日志（日志内容见“Error Handing and Logging”章节）。

4、 如果在 javascript 中做页面跳转，需要判断域名白名单后，才能跳转。如果应用只有跳转到360集团网站的需求，可以设置白名单，判断目的地址是否在白名单列表中，如果不在列表中，就判定为 URL 跳转攻击，并记录日志。不允许配置集团以外网站到白名单列表中。这两个方案都可以保证所有在应用中发出的重定向地址，都是可信任的地址。

3\.	异常处理类漏洞


### 3.1.	错误的异常处理&#xD;

说明：编码人员常常会通过一个空的或者无意义的catch块来抑制捕获的已检查异常。每一个catch块都应该确保程序只会在继续有效的情况下才会继续运行下去。因此，catch块必须要么从异常情况进行恢复，要么重新抛出适合当前catch块上下文的另一个异常以允许最邻近的外层try-catch语句块来进行恢复工作。异常会打断应用原本预期的控制流程。例如，try块中位于异常发生点之后的任何表达式和语句都不会被执行。因此，异常必须被妥当处理。许多抑制异常的理由都是不合理的。例如，当对客户端从潜在问题恢复过来不抱期望时，一种好的做法是让异常被广播出来，而不是去捕获和抑制这个异常。

错误示例：

```java
//...
try
{
    //...
}
catch (IOException ioe)
{
    ioe.printStackTrace();
}
//...
```

在这个错误示例中，catch块只是简单地将异常堆栈轨迹打印出来。虽然打印异常的堆栈轨迹对于定位问题是有帮助的，但是最终的程序运行逻辑等同于抑制异常时的情况。注意，即使这个错误示例在发生异常时会打印一个堆栈轨迹，但是程序会继续运行，如同异常从未被抛出过。换句话说，除了try块中位于异常发生点之后的表达式和语句不会被执行之外，发生的异常不会影响程序的其他行为。

正确示例（交互）：

```java
// ...
volatile boolean validFlag = false;
do
{
    try
    {
        // If requested file does not exist, throws FileNotFoundException
        // If requested file exists, sets validFlag to true
        validFlag = true;
    }
    catch (FileNotFoundException e)
    {
        // Ask the user for a different file name
    }
} while (validFlag != true);
// Use the file
```

这个正确示例通过要求用户指定另外一个文件名来处理FileNotFoundException异常。为了遵循禁止在异常中泄露敏感信息，一个用户只允许访问该用户特定的目录。这可以防止往循环外抛出其他IOException异常泄露文件系统的敏感信息。

正确示例（Exception Reporter）：

```java
public interface Reporter
{
    public void report(Throwable t);
}

public class ExceptionReporter
{
    // Exception reporter that prints the exception 
    // to the console (used as default)
    private static final Reporter printException = new Reporter()
    {
        public void report(Throwable t)
        {
            System.err.println(t.toString());
        }
    };
    
    // Stores the default reporter.
    // The default reporter can be changed by the user.
    private static Reporter default = printException;
    
    // Helps change the default reporter back to 
    // PrintException in the future
    public static Reporter getPrintException()
    {
        return printException;
    }
    
    public static Reporter getExceptionReporter()
    {
        return default;
    }
    
    // May throw a SecurityException (which is unchecked)
    public static void setExceptionReporter(Reporter reporter)
    {
        // Custom permission
        ExceptionReporterPermission perm = new ExceptionReporterPermission(
                "exc.reporter");
        SecurityManager sm = System.getSecurityManager();
        if (sm != null)
        {
            // Check whether the caller has appropriate permissions
            sm.checkPermission(perm);
        }
        // Change the default exception reporter
        default = reporter;
    }
}
```

如何恰当的报告异常情况依赖与具体的上下文。例如，对于GUI应用应该以图形界面的方式报告异常，例如弹出一个错误对话框。对于大部分库中的类应该能够客观的决定该如何报告一个异常来保持模块化；它们不能依赖于System.err、特定的logger，或者windowing环境的可用性。因此，对于将会报告异常的库类，应该指定一个此类用来报告异常的API。在这个正确示例中，指定了一个包含report()方法的用来报告异常的接口，以及一个默认的异常reporter类供库类使用。可以定义子类来覆盖这个异常reporter类。

库中的类后续便可以在catch子句中使用这个异常reptorter类：

```java
try
{
    // ...
}
catch (IOException warning)
{
    ExceptionReporter.getExceptionReporter().report(warning);
    // Recover from the exception...
}
```

任何使用这个异常reporter类的客户代码，只要具备所需的权限许可，就能够覆写这个ExceptionReporter，使用一个logger或者提供一个对话框来报告异常。例如，一个使用Swing的GUI客户代码，要求使用对话框来报告异常：

```java
ExceptionReporter.setExceptionReporter(new ExceptionReporter()
{
    public void report(Throwable exception)
    {
        JOptionPane.showMessageDialog(frame,
               exception.toString, exception.getClass().getName(),
                    JOptionPane.ERROR_MESSAGE);
    }
});
```

正确示例（继承Exception Reporter并过滤敏感异常）：

```java
class MyExceptionReporter extends ExceptionReporter
{
    public static void report(Throwable t)
    {
        t = filter(t);
        // Do any necessary user reporting (show dialog box or send to console)
    }
    
    public static Exception filter(Throwable t)
    {
        // Sanitize sensitive data or replace sensitive exceptions with non-sensitive exceptions (whitelist)
        // Return non-sensitive exception
    }
}
```

出于安全原因，有时候必须对用户隐藏异常。在这种情况下，一种可行的方式是继承ExceptionReporter类，并且在重写默认report()方法的基础上，增加一个filter()方法。

### 3.2.	异常中泄露敏感信息&#xD;

说明：敏感数据的范围应该基于应用场景以及产品威胁分析的结果来确定。典型的敏感数据包括口令、银行账号、个人信息、通讯记录、密钥等。如果在传递异常的时候未对其中的敏感信息进行过滤常常会导致信息泄露，而这可能帮助攻击者尝试发起进一步的攻击。攻击者可以通过构造恶意的输入参数来发掘应用的内部结构和机制。不管是异常中的文本消息，还是异常本身的类型都可能泄露敏感信息。例如，对于FileNotFoundException异常，其中的异常消息会透露文件系统的结构信息，而通过异常本身的类型，可以得知所请求的文件不存在。因此，当异常会被传递到信任边界以外时，必须同时对敏感的异常消息和敏感的异常类型进行过滤。附录C列出了一些常见的需要注意的异常类型。

错误示例（异常消息和类型泄露敏感信息）：

```java
public class ExceptionExample
{
    public static void main(String[] args) throws FileNotFoundException
    {
        // Linux stores a user's home directory path in
        // the environment variable $HOME, Windows in %APPDATA%
        // ... Other omitted code
        FileInputStream fis = new FileInputStream(System.getenv("APPDATA")
                + args[0]);
    }  
}
```

此错误示例代码中，当请求的文件不存在时，FileInputStream的构造器会抛出FileNotFoundException异常。这使得攻击者可以不断传入伪造的路径名称来重现出底层文件系统结构。

错误示例（封装后重新抛出敏感异常）：

```java
try
{
    FileInputStream fis = new FileInputStream(System.getenv("APPDATA")
                    + args[0]);
}
catch (FileNotFoundException e)
{
    // Log the exception
    throw new IOException("Unable to retrieve file", e);
}
```

此错误示例中，即使用户无法访问到日志中记录的异常，但是原始的异常仍然会被广播，攻击者可能会利用这点来发现敏感的文件系统结构信息。

错误示例 （异常净化）：

```java
class SecurityIOException extends IOException
{
    /* ... */
};

// ...
try
{
    FileInputStream fis = new FileInputStream(System.getenv("APPDATA") + args[0]);
}
catch (FileNotFoundException e)
{
    // Log the exception
    throw new SecurityIOException();
}
```

与前面几个错误示例相比，此例代码抛出的异常虽然泄露的有用信息较少，但是它仍然会透露出指定的文件不可读。程序对于存在的文件路径输入和不存在的文件路径输入会有不同的反应。因此，攻击者可以根据程序的行为推断出文件系统的敏感信息。未对用户输入做限制，使得系统面临暴力攻击的风险，攻击者可以多次传入所有可能的文件名进行查询来发现有效文件：如果传入一个文件名后，程序返回一个净化后的异常，则暗示该文件不存在，而如果不抛出异常则说明该文件是存在的。

正确示例（安全策略）:

```java
public class ExceptionExample
{
    public static void main(String[] args)
    {
        File file = null;
        try
        {
            file = new File(System.getenv("APPDATA") + args[0]).getCanonicalFile();
            if (!file.getPath().startsWith("c:\\homepath"))
            {
                System.out.println("Invalid file");
                return;
            }
        }
        catch (IOException x)
        {
            System.out.println("Invalid file");
            return;
        }
        try
        {
            FileInputStream fis = new FileInputStream(file);
        }
        catch (FileNotFoundException x)
        {
            System.out.println("Invalid file");
            return;
        }
    }
}
```

在这个正确示例中，规定用户只能打开c:\homepath目录下的文件，用户不可能发现这个目录以外的任何信息。在这个方案中，如果无法打开文件，或者文件不在合法的目录下，则会产生一条简洁的错误消息。任何c:\homepath目录以外的文件信息都被会隐蔽起来。

正确示例（限制输入）：

```java
public class ExceptionExample
{
    public static void main(String[] args)
    {
        FileInputStream fis = null;
        try
        {
            switch (Integer.valueOf(args[0]))
            {
                case 1:
                    fis = new FileInputStream("c:\\homepath\\file1");
                    break;
                case 2:
                    fis = new FileInputStream("c:\\homepath\\file2");
                    break;
                // ...
                default:
                    System.out.println("Invalid option");
                    break;
            }
        }
        catch (Throwable t)
        {
            MyExceptionReporter.report(t); // Sanitize any sensitive data
        }
    }
}
```

这个正确示例限制用户只能打开c:\homepath\file1与c:\homepath\file2。同时，它也会过滤在catch块中捕获的异常中的敏感信息。

4\.	文件操作类漏洞


### 4.1.	文件上传&#xD;

说明：多File upload，任意文件上传攻击。

Web 应用程序在处理用户上传的文件时，没有判断文件的扩展名是否在允许的范围

内，就把文件保存在服务器上，导致恶意用户可以上传任意文件，甚至上传脚本木马到 web 服务器上，直接控制 web 服务器。

错误示例：

```java
PrintWriter pw = new PrintWriter(new BufferedWriter(new FileWriter( request.getRealPath("/")+getFIlename(request))));
ServletInputStream in = request.getInputStream();
int i = in.read();
while (i != -1) {
pw.print((char) i);
i = in.read();
}
pw.close();
```

解决方案：处理用户上传文件，要做以下检查：

1、 检查上传文件扩展名白名单，不属于白名单内，不允许上传。

2、 上传文件的目录必须是 http 请求无法直接访问到的。如果需要访问的，必须上传到其他（和 web 服务器不同的）域名下，并设置该目录为不解析 jsp 等脚本语言的目录。

3、 上传文件要保存的文件名和目录名由系统根据时间生成，不允许用户自定义。

4、 图片上传，要通过处理（缩略图、水印等），无异常后才能保存到服务器。

### 4.2.	路径穿越&#xD;

说明：绝对路径名或者相对路径名中可能会包含文件链接，比如符号（软）链接（symbolic \[soft] links）、硬链接（hard links）、快捷方式（shortcuts）、影子文件（shadows）、别名（aliases）与连接文件（junctions）。这些文件链接在文件验证操作进行之前必须被完全解析。路径名中可能会包含特殊的文件名，这也会让验证变得困难：

* &#x9;“.”指目录本身。
* &#x9;在一个目录内，“..”指该目录的上一级目录。

除了这些特殊的问题之外，还有很多操作系统和文件系统相关的命名约定会使验证变得困难。对文件名标准化可以使得验证文件路径更加容易。对于同一个目录或者文件，可以通过多种路径名来引用它。此外，一条路径的文字描述基本不会给出有关它所引用的目录或文件的任何信息。因此，所有的路径名在验证之前都需要被完全解析或者标准化（canonicalized）。

程序中可能经常会涉及到文件或目录检查，例如，当试图限制用户只能访问某个特定目录中的文件时，或者当基于文件名或者路径名来做安全决策时。攻击者可能会利用目录遍历（directory traversal）或者等价路径（path equivalence）的方式来绕过这些限制。目录遍历漏洞使得攻击者能够让I/O操作跳出一个特定的目录。等价路径漏洞是指攻击者可以通过一个资源的不同但是等价的名称来绕过安全检查。

程序获取一个文件标准路径的时间和打开这个文件的时间之间会有一个固有的竞争窗口。当文件的标准路径正在被验证时，该文件可能已经被修改，且之前获取的标准路径可能已经不再指向原来的有效文件。幸运的是，可以很容易得消除这种条件竞争。我们可以使用文件的标准路径来判断该文件是否是在安全目录之中。如果引用的文件是在一个安全目录之中，则根据定义，攻击者无法篡改该文件，因此便无法引起条件竞争。

错误示例：

```java
public static void main(String[] args)
{
        File f = new File(System.getProperty("user.home")
                + System.getProperty("file.separator") + args[0]);
        String absPath = f.getAbsolutePath();
        if (!isInSecureDir(Paths.get(absPath)))
        {
            // Refer to Rule 3.5 for the details of isInSecureDir()
            throw new IllegalArgumentException();
        }
        if (!validate(absPath))
        {   
            // Validation
            throw new IllegalArgumentException();
        }
        /* … */
}
```

File.getAbsolutePath()返回文件的绝对路径，但是它不会解析文件链接，也不会消除等价错误。

注意，在Windows和Macintosh平台中，File.getAbsolutePath()方法可以解析符号链接、别名和快捷方式。尽管如此，在Sun的Java语言标准中却不能保证这样的行为在所有的平台上都有效，或者在未来的实现中均会这样做。

正确示例（getCanonicalPath()）:

```java
public static void main(String[] args) throws IOException
{
    File f = new File(System.getProperty("user.home")
                + System.getProperty("file.separator") + args[0]);
    String canonicalPath = f.getCanonicalPath();
    if (!isInSecureDir(Paths.get(absPath)))
    {
        // Refer to Rule 3.5 for the details of isInSecureDir()
        throw new IllegalArgumentException();
    }
    if (!validate(absPath))
    {   
        // Validation
        throw new IllegalArgumentException();
    }
    /* ... */
}
```

这个正确示例使用了File.getCanonicalPath()方法，它能在所有的平台上对所有别名、快捷方式以及符号链接进行一致地解析。特殊的文件名，比如“..”会被移除，这样输入在验证之前会被简化成对应的标准形式。当使用标准形式的文件路径来做验证时，攻击者将无法使用../序列来跳出指定目录。

### 4.3.	zip炸弹&#xD;

说明：从java.util.zip.ZipInputStream中解压文件时需要小心谨慎。有两个特别的问题需要避免：一个是提取出的文件标准路径落在解压的目标目录之外，另一个是提取出的文件消耗过多的系统资源。对于前一种情况，攻击者可以从zip文件中往用户可访问的任何目录写入任意的数据。对于后一种情况，当资源使用远远大于输入数据所使用的资源的时，就可能会发生拒绝服务的问题。Zip算法的本性就可能会导致zip炸弹（zip bomb）的出现，由于极高的压缩率，即使在解压小文件时，比如ZIP、GIF，以及gzip编码的HTTP内容，也可能会导致过度的资源消耗。

Zip算法能够产生非常高的压缩比率。例如，一个文件由多行a字符和多行b字符交替出现构成，对于这样的一个文件可以达到200：1以上的压缩比率。使用针对目标压缩算法的输入数据，或者使用更多的输入数据（无目标的），或者使用其他的压缩方法，甚至可以达到更高的压缩比率。

任何被提取条目的目标路径不在程序预期计划的目录之内时（必须先对文件名进行标准化），要么拒绝将其提取出来，要么将其提取到一个安全的位置。Zip中任何被提取条目，若解压之后的文件大小超过一定的限制时，必须拒绝将其解压。具体限制多少，由平台的处理性能所决定。

错误示例：

```java
static final int BUFFER = 512;
// ...
public final void unzip(String fileName) throws java.io.IOException
{
    FileInputStream fis = new FileInputStream(fileName);
    ZipInputStream zis = new ZipInputStream(new BufferedInputStream(fis));
    ZipEntry entry;
    while ((entry = zis.getNextEntry()) != null)
    {
        System.out.println("Extracting: " + entry);
        int count;
        byte data[] = new byte[BUFFER];
        // Write the files to the disk
        FileOutputStream fos = new FileOutputStream(entry.getName());
        BufferedOutputStream dest = new BufferedOutputStream(fos, BUFFER);
        while ((count = zis.read(data, 0, BUFFER)) != -1)
        {
                dest.write(data, 0, count);
        }
        dest.flush();
        dest.close();
        zis.closeEntry();
     }
     zis.close();
}
```

在这个错误示例中，未对解压的文件名做验证，直接将文件名传递给FileOutputStream构造器。它也未检查解压文件的资源消耗情况，它允许程序运行到操作完成或者本地资源被耗尽。

错误示例（getSize()）：

```java
public static final int BUFFER = 512;
public static final int TOOBIG = 0x6400000; // 100MB
// ...
public final void unzip(String filename) throws java.io.IOException
{
    FileInputStream fis = new FileInputStream(filename);
    ZipInputStream zis = new ZipInputStream(new BufferedInputStream(fis));
    ZipEntry entry;
    try
    {
        while ((entry = zis.getNextEntry()) != null)
        {
            System.out.println("Extracting: " + entry);
            int count;
            byte data[] = new byte[BUFFER];
            // Write the files to the disk, but only if the file is not insanely big
            if (entry.getSize() > TOOBIG)
            {
                throw new IllegalStateException(
                        "File to be unzipped is huge.");
            }
            if (entry.getSize() == -1)
            {
                throw new IllegalStateException(
                        "File to be unzipped might be huge.");
            }
            FileOutputStream fos = new FileOutputStream(entry.getName());
            BufferedOutputStream dest = new BufferedOutputStream(fos,
                    BUFFER);
            while ((count = zis.read(data, 0, BUFFER)) != -1)
            {
                dest.write(data, 0, count);
            }
            dest.flush();
            dest.close();
            zis.closeEntry();
        }
    }
    finally
    {
        zis.close();
    }
}
```

这个错误示例调用ZipEntry.getSize()方法在解压提取一个条目之前判断其大小，以试图解决之前的问题。但不幸的是，恶意攻击者可以伪造ZIP文件中用来描述解压条目大小的字段，因此，getSize()方法的返回值是不可靠的，本地资源实际仍可能被过度消耗。

正确示例：

```java
static final int BUFFER = 512;
static final int TOOBIG = 0x6400000; // max size of unzipped data, 100MB
static final int TOOMANY = 1024; // max number of files
// ...
private String sanitzeFileName(String entryName, String intendedDir) throws IOException
{
    File f = new File(intendedDir, entryName);
    String canonicalPath = f.getCanonicalPath();
    
    File iD = new File(intendedDir);
    String canonicalID = iD.getCanonicalPath();
    
    if (canonicalPath.startsWith(canonicalID))
    {
        return canonicalPath;
    }
    else
    {
        throw new IllegalStateException(
                "File is outside extraction target directory.");
    }
}
// ...
public final void unzip(String fileName) throws java.io.IOException
{
    FileInputStream fis = new FileInputStream(fileName);
    ZipInputStream zis = new ZipInputStream(new BufferedInputStream(fis));
    ZipEntry entry;
    int entries = 0;
    int total = 0;
    byte[] data = new byte[BUFFER];
    try
    {
        while ((entry = zis.getNextEntry()) != null)
        {
            System.out.println("Extracting: " + entry);
            int count;
            // Write the files to the disk, but ensure that the entryName is valid,
            // and that the file is not insanely big
            String name = sanitzeFileName(entry.getName(), ".");
            FileOutputStream fos = new FileOutputStream(name);
            BufferedOutputStream dest = new BufferedOutputStream(fos, BUFFER);
            while (total + BUFFER <= TOOBIG && (count = zis.read(data, 0, BUFFER)) != -1)
            {
                dest.write(data, 0, count);
                total += count;
            }
            dest.flush();
            dest.close();
            zis.closeEntry();
            entries++;
            if (entries > TOOMANY)
            {
                throw new IllegalStateException("Too many files to unzip.");
            }
            if (total > TOOBIG)
            {
                throw new IllegalStateException(
                        "File being unzipped is too big.");
            }
        }
    }
    finally
    {
        zis.close();
    }
}
```

在这个正确示例中，代码会在解压每个条目之前对其文件名进行校验。如果某个条目校验不通过，整个解压过程都将会被终止。实际上也可以忽略跳过这个条目，继续后面的解压过程，甚至也可以将这个条目解压到某个安全位置。除了校验文件名，while循环中的代码会检查从zip存档文件中解压出来的每个文件条目的大小。如果一个文件条目太大，此例中是100MB，则会抛出异常。最后，代码会计算从存档文件中解压出来的文件条目总数，如果超过1024个，则会抛出异常。

5\.	访问控制类漏洞


### 5.1.	越权访问页面&#xD;

说明：Vertical Access Control，垂直权限安全攻击，也就是权限提升攻击。

由于 web 应用程序没有做权限控制，或仅仅在菜单上做了权限控制，导致的恶意用户只要猜测其他管理页面的 URL，就可以访问或控制其他角色拥有的数据或页面，达到权限提升目的。

这个威胁可能导致普通用户变成管理员权限。

错误示例：一个仅仅做了菜单控制的代码：

```markup
<tr><td><a href="/user.jsp">管理个人信息</a></td></tr> 
<%if (power.indexOf("administrators")>-1){%> 
<tr><td><a href="/userlist.jsp">管理所有用户</a></td></tr>
<%}%>
```

解决方案：在打开管理页面 URL 时，首先判断当前用户是否拥有该页面的权限，如果没有权限，就判定为“权限提升”攻击，同时记录安全日志。

建议使用成熟的权限框架处理权限问题。

### 5.2.	越权增删改查&#xD;

说明：Horizontal Access Control，访问控制攻击，也就是水平权限安全攻击。

Web 应用程序接收到用户请求，修改某条数据时，没有判断数据的所属人，或判断数据所属人时，从用户提交的 request 参数（用户可控数据）中，获取了数据所属人 id，导致恶意攻击者可以通过变换数据 ID，或变换所属人 id，修改不属于自己的数据。

恶意用户可以删除或修改其他人数据。

错误示例：访问数据层（dao），所有的更新语句操作，都可能产生这个漏洞。

以下代码存在这个漏洞，web 应用在修改用户个人信息时，从从用户提交的 request 参数（用户可控数据）中，获取了 userid，执行修改操作。

修改用户个人信息页面

```markup
<form action="/struts1/edituser.htm" method="post">
<input name="userid" type="hidden" value="<%=userid%>"> <table border="1">
<tr>
<td>username:</td>
<td><%=rs.getString("name")%></td>
</tr>
<tr>
<td>passwd:</td>
<td> <input name="pass" value="<%=rs.getString("pass")%>"></td>
</tr>
<tr>
<td>type:</td>
<td><%=rs.getString("type")%></td>
</tr>
<tr>
<td>realname:</td>
<td><input name="realname" value="<%=rs.getString("realname")%>"></td>
</tr>
<tr>
<td>email:</td>
<td> <input name="email" value="<%=rs.getString("email")%>"></td>
</tr>
<tr>
<td>tel:</td>
<td> <input name="tel" value="<%=rs.getString("tel")%>"></td>
</tr>
</table>
<html:submit/>
</form>
```

表单中，将用户的 useird 作为隐藏字段，提交给处理修改个人信息的应用。下面代码是修改个人信息的应用

```java
int userid=Integer.valueOf( request.getParameter("userid")); 
String email=request.getParameter("email"); 
String tel=request.getParameter("tel");
String realname=request.getParameter("realname");
String pass=request.getParameter("pass");
JdbcConnection conn = null;
try {
conn = new JdbcConnection();
Object[] params = new Object[5];
params[0] = email;
params[1] = tel;
params[2] = realname;
params[3] = pass;
params[4] = userid;
final String sql = "update user set email=?,tel=?,realname=?,pass=? where userid=?"; 
conn.execUpdate(sql,params);
conn.closeConn();
```

这段代码是从 request 的参数列表中，获取 userid，也就是表单提交上来的 userid，之后修改 userid 对应的用户数据。

而表单中的 userid 是可以让用户随意修改的。

正确示例：

从用户的加密认证 cookie 中，获取当前用户的 id，并且需要在执行的 SQL 语句中，加入当前用户 id 作为条件语句。由于是 web 应用控制的加密算法，所以恶意用户无法修改加密信息。

```java
int userid=Integer.valueOf( GetUseridFromCookie(request)); String email=request.getParameter("email"); String tel=request.getParameter("tel");
String realname=request.getParameter("realname");
String pass=request.getParameter("pass");
JdbcConnection conn = null;
try {
conn = new JdbcConnection();
Object[] params = new Object[5];
params[0] = email;
params[1] = tel;
params[2] = realname;
params[3] = pass;
params[4] = userid;
final String sql = "update user set email=?,tel=?,realname=?,pass=? where userid=?";
conn.execUpdate(sql,params);
conn.closeConn();
```

代码中通过 GetUseridFromCookie，从加密的 COOKIE 中获取了当前用户的 id，并加入到 SQL 语句中的 WHERE 条件中。

6\.	序列化和反序列化漏洞


### 6.1.	序列化未加密的敏感数据&#xD;

说明：虽然序列化可以将对象的状态保存为一个字节序列，之后通过反序列化该字节序列又能重新构造出原来的对象，但是它并没有提供一种机制来保证序列化数据的安全性。可访问序列化数据的攻击者可以借此获取敏感信息并确定对象的实现细节。攻击者也可恶意修改其中的数据，试图在其被反序列化之后对系统造成危害。因此，敏感数据序列化之后是潜在对外暴露着的。永远不应该被序列化的敏感信息包括：密钥、数字证书、以及那些在序列化时引用敏感数据的类。此条规则的意义在于防止敏感数据被无意识的序列化导致敏感信息泄露。

错误示例：

```java
public class GPSLocation implements Serializable
{
    private double x; // sensitive field
    private double y; // sensitive field
    private String id;// non-sensitive field
    
    // other content
}
public class Coordinates
{
    public static void main(String[] args)
    {
        FileOutputStream fout = null;
        try
        {
            GPSLocation p = new GPSLocation(5, 2, "northeast");
            fout = new FileOutputStream("location.ser");
            ObjectOutputStream oout = new ObjectOutputStream(fout);
            oout.writeObject(p); 
            oout.close();
        }
        catch (Throwable t)
        {
            // Forward to handler 
        }
        finally
        {
            if (fout != null)
            {
                try
                {
                    fout.close();
                }
                catch (IOException x)
                {
                    // handle error 
                }
            }
        }
    }
}
```

在这段示例代码中，假定坐标信息是敏感的，那么将其序列化到数据流中使之面临敏感信息泄露与被恶意篡改的风险。

正确示例（transient）：

```java
public class GPSLocation implements Serializable
{
    private transient double x; // transient field will not be serialized    
    private transient double y; // transient field will not be serialized 
    private String id;
     
    // other content
}
```

在将某个包含敏感数据的类序列化时，程序必须确保敏感数据不被序列化。这包括阻止包含敏感信息的数据成员被序列化，以及不可序列化或者敏感对象的引用被序列化。该示例将相关字段声明为transient，从而使它们不包括在依照默认的序列化机制应该被序列化的字段列表中。这样既避免了错误的序列化，又防止了敏感数据被意外序列化。

正确示例（serialPersistentFields）:

```java
public class GPSLocation implements Serializable
{
    private double x;
    private double y;
    private String id;
    // sensitive fields x and y are not content in serialPersistentFields
    private static final ObjectStreamField[] serialPersistentFields = {new ObjectStreamField("id", String.class)};
    // other content
}
```

该示例通过定义serialPersistentFields数组字段来确保敏感字段被排除在序列化之外，除了上述方案，也可以通过自定义writeObject()、writeReplace()、writeExternal()这些函数，不将包含敏感信息的字段写到序列化字节流中。

例外情况：

可以序列化已正确加密的敏感数据。

### 6.2.	序列化和反序列化被利用来绕过安全管理&#xD;

说明：序列化和反序列化可能被利用来绕过安全管理器的检查。一个可序列化类的构造器中出于防止不可信代码修改类的内部状态等原因可能需要引入安全管理器的检查。这种安全管理器的检查必须应用到所有能够构建类实例的地方。例如，如果某个类依据安全检查的结果来判定调用者是否能够读取其敏感内部状态，那么这类安全检查必须也在反序列化中应用。这就确保了攻击者无法通过反序列化对象来提取敏感信息。

错误示例：

```java
public final class Hometown implements Serializable
{
    private static final long serialVersionUID = 9078808681344666097L;
    
    // Private internal state 
    private String town;
    
    private static final String UNKNOWN = "UNKNOWN";
    
    void performSecurityManagerCheck() throws SecurityException
    {
        // verify whether current user has rights to access the file
    }
    
    void validateInput(String newCC) throws InvalidInputException
    {
        // ...
    }
    
    public Hometown()
    {
        performSecurityManagerCheck();
        // Initialize town to default value
        town = UNKNOWN;
    }
    
    // Allows callers to retrieve internal state
    String getValue()
    {
        performSecurityManagerCheck();
        return town;
    }
    
    // Allows callers to modify (private) internal state
    public void changeTown(String newTown) throws InvalidInputException
    {
        if (town.equals(newTown))
        {
            // No change
            return;
        }
        else
        {
            performSecurityManagerCheck();
            validateInput(newTown);
            town = newTown;
        }
    }
    
    private void writeObject(ObjectOutputStream out) throws IOException
    {
        out.writeObject(town);
    }
    
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException
    {
        in.defaultReadObject();
        // If the deserialized name does not match
        // the default value normally
        // created at construction time, duplicate the checks
        if (!UNKNOWN.equals(town))
        {
            validateInput(town);
        }
    }
}
```

在该错误示例中，安全管理器检查被应用在构造器中，但在序列化与反序列化涉及的writeObject()和readObject()方法中没有用到。这样会允许非信任代码恶意创建类实例。

正确示例：

```java
public final class Hometown implements Serializable
{
    // ... all methods the same except the following: 
    // writeObject() correctly enforces checks during serialization 
    private void writeObject(ObjectOutputStream out) throws IOException
    {
        performSecurityManagerCheck();
        out.writeObject(town);
    }
    
    // readObject() correctly enforces checks during deserialization 
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException
    {
        in.defaultReadObject();
        // If the deserialized name does not match the default value normally 
        // created at construction time, duplicate the checks 
        if (!UNKNOWN.equals(town))
        {
            performSecurityManagerCheck();
            validateInput(town);
        }
    }
}
```

正确做法是在所有构造器以及可以修改或检索内部数据的方法中都需要应用安全管理器检查。这样一来，攻击者就不能用反序列化来修改对象的实例或者读取字节流来窃取序列化的数据。

7\.	会话管理类漏洞


### 7.1.	缺少Httponly属性&#xD;

说明：Cookie http only，是设置 COOKIE 时，可以设置的一个属性，如果 COOKIE 没有设置这个属性，该 COOKIE 值可以被页面脚本读取。

当攻击者发现一个 XSS 漏洞时，通常会写一段页面脚本，窃取用户的 COOKIE，为了增加攻击者的门槛，防止出现因为 XSS 漏洞导致大面积用户 COOKIE 被盗，所以应该在设置认证 COOKIE 时，增加这个属性。

错误示例：

设置 cookie 的代码

```java
response.setHeader("SET-COOKIE", "user=" + request.getParameter("cookie"));
```

这段代码没有设置 http only 属性

正确示例：

设置 cookie 时，加入属性即可

```java
response.setHeader("SET-COOKIE", "user=" + request.getParameter("cookie") + "; HttpOnly");
```

### 7.2.	缺少Secure属性&#xD;

说明：Cookie Secure，是设置 COOKIE 时，可以设置的一个属性，设置了这个属性后，只有在 https 访问时，浏览器才会发送该 COOKIE。

浏览器默认只要使用 http 请求一个站点，就会发送明文 cookie，如果网络中有监控，可能被截获。

如果 web 应用网站全站是 https 的，可以设置 cookie 加上 Secure 属性，这样浏览器就只会在 https 访问时，发送 cookie。

攻击者即使窃听网络，也无法获取用户明文 cookie。

错误示例：

设置 cookie 的代码

```java
response.setHeader("SET-COOKIE", "user=" + request.getParameter("cookie") + "; HttpOnly");
```

这段代码没有设置 secure属性

正确示例：

设置 cookie 时，加入secure属性即可

```java
response.setHeader("SET-COOKIE", "user=" + request.getParameter("cookie") + "; HttpOnly;Secure");
```

### 7.3.	缺少会话有效期&#xD;

说明：由于 Session 没有在 web 应用中设置强制超时时间，攻击者一旦曾经获取过用户的Session，就可以一直使用。

错误示例：

设置 cookie 的代码

```java
response.setHeader("SET-COOKIE", "user=" + request.getParameter("cookie") + "; HttpOnly;Secure");
```

这段代码没有在服务器中设置强制超时时间。

正确示例：

在设置认证 cookie 中，加入两个时间，一个是“即使一直在活动，也要失效”的时间，一个是“长时间不活动的失效时间”。并在 web 应用中，首先判断两个时间是否已超时，再执行其他操作。

```java
//	判断会员的 cookie 是否过期 if (isLogin) {
String timeStampStr = (String)
map.get(UserAuthenticationContext.TIMESTAMP); long loginTime = 0;
try {
loginTime = Long.parseLong(timeStampStr);
} catch (NumberFormatException e) {
if (logger.isInfoEnabled()) {
logger.info(" loginId: " + usr.getLoginId() + " timestamp has exception " + timeStampStr);
}
}
long now = System.currentTimeMillis() / 1000;
if (now - loginTime > UserAuthenticationContext.COOKIE_VALIDITY) { usr.setAuthenticated(false, true);
if (logger.isInfoEnabled()) {
logger.info("loginId: " + usr.getLoginId() + " loginTime: " + loginTime
+	" nowTime: " + now);
}
}
}
```

8\.	运行环境类漏洞


### 8.1.	生产代码存在调试入口点&#xD;

说明：一种常见的做法就是由于调试或者测试目的在代码中添加特定的后门代码，这些代码并没有打算与应用一起交付或者部署。当这类的调试代码不小心被留在了应用中，这个应用对某些无意的交互就是开放的。这些后门入口点可以导致安全风险，因为在设计和测试的时候并没有考虑到而且处于应用预期的运行情况之外。

被忘记的调试代码最常见的例子比如一个web应用中出现的main()方法。虽然这在产品生产的过程中也是可以接受的，但是在生产环境下，J2EE应用中的类是不应该定义有main()的。

错误示例：

```java
public class Stuff
{   
    // other fields and methods
    public static void main(String args[])
    {
        Stuff stuff = new Stuff();
        // Test stuff
    }
}
```

在这个错误代码示例中，Stuff类使用了一个main()函数来测试其方法。尽管对于调试是很有用的，如果这个函数被留在了生产代码中（例如，一个Web应用），那么攻击者就可能直接调用Stuff.main()来访问Stuff类的测试方法。

正确示例：

正确的代码示例中将main()方法从Stuff类中移除，这样攻击者就不能利用这个入口点了。

9\.	其他类型漏洞


### 9.1.	日志中保存口令、密钥和其他敏感数据&#xD;

说明：在日志中不能输出口令、密钥和其他敏感信息，口令包括明文口令和密文口令。对于敏感信息建议采取以下方法：

* 不在日志中打印敏感信息。
* 若因为特殊原因必须要打印日志，则用固定长度的星号（\*）代替输出的敏感信息。

### 9.2.	使用私有或者弱加密算法&#xD;

说明：禁止使用私有算法或者弱加密算法（比如DES，SHA1等）。应该使用经过验证的、安全的、公开的加密算法。

加密算法分为对称加密算法和非对称加密算法。推荐使用的对称加密算法有：

* AES

推荐使用的非对称算法有：

* RSA

推荐使用的数字签名算法有：

* DSA
* ECDSA

除了以上提到的几种算法之外，还经常使用安全哈希算法（SHA256）等来验证消息的完整性。如果使用哈希算法来存储口令，则必须加入盐值（salt）

对每个推荐的算法，其密钥长度需符合以下最低安全要求：

* AES: 128位
* RSA: 2048位
* DSA: 2048位

### 9.3.	不安全的哈希算法（未加salt）&#xD;

说明：实践中，一个口令可以编码为一个哈希值，且无法从哈希值逆向计算出原始的口令。口令是否相等可以通过比较它们的哈希值是否相等来判断。如果一个口令的哈希值储存在一个数据库中，由于哈希算法的不可逆性，攻击者就应该不可能还原出口令。如果说可以恢复口令，那么唯一的方式就是暴力破解攻击，比如计算所有可能口令的哈希值，或是字典攻击，计算出所有常用的口令的哈希值。如果每个口令都只仅经过简单哈希，相同的口令将得到相同的哈希值。仅保存口令哈希有以下两个缺陷：

* 由于“生日判定”，攻击者可以快速找到一个口令，尤其是当数据库中的口令数量较大的时候。
* 攻击者可以使用事先计算好的哈希列表在几秒钟之内破解口令。

为了解决这些问题，可以在进行哈希运算之前在口令中引入盐值。一个盐值是一个固定长度的随机数。这个盐值对于每个存储入口来说必须是不同的。可以明文方式紧邻哈希后的口令一起保存。在这样的配置下，攻击者必须对每一个口令分别进行暴力破解攻击。这样数据库便能抵御“生日”或者“彩虹表”攻击。

为了减慢哈希的计算速度，推荐进行n次迭代操作。虽然对一个口令进行n次哈希对于攻击者和典型用户来说的确减慢了哈希，但是典型用户并不会有太大的感知，因为哈希的时间相对于他们与系统互动的总时间来说只是非常小的一个比例。另一方面，攻击者破解时几乎100%的时间花在哈希计算上，所以哈希n次以n为因子减慢了攻击者的速度而对于典型用户几乎不可察觉。

建议：

* 盐值至少应该包含8字节而且必须是由安全随机数产生。
* 应使用强哈希函数，推荐使用SHA-256或者更加安全的哈希函数。
* 推荐默认进行50000次哈希，至少对有性能限制（比如说嵌套系统）的产品进行5000次以上哈希。



正确示例（PBKDF2）：

```java
public static byte[] createHash(char[] password)
        throws NoSuchAlgorithmException, InvalidKeySpecException
{
    SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
    byte[] salt = new byte[8];
    random.nextBytes(salt);
    int iterCount = 50000;
    PBEKeySpec spec = new PBEKeySpec(password, salt, iterCount, 256);
    //PBKDF2WithHmacSHA256 is supportted from JDK1.8
    SecretKeyFactory skf = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
    byte[] hashed = skf.generateSecret(spec).getEncoded();
    return hashed;
}
```

口令单向Hash场景下可以使用PBKDF2算法，PBKDF2是一个密钥导出算法，既可用于导出密钥，也可用于口令保存，并且已在RFC 2898标准中定义。它使用最为广泛，能被大多数算法库所支持。注意，SunJCE Provider从JDK1.8版本才开始支持PBKDF2WithHmacSHA256算法。对于JDK1.8之前的版本，可以接受选择PBKDF2WithHmacSHA1算法，或者考虑使用其他可靠JCE Provider提供支持的PBKDF2与SHA256或者更强哈希算法的组合。

### 9.4.	敏感信息硬编码在程序中&#xD;

说明：如果将敏感信息（包括口令和加密密钥）硬编码在程序中，可能会将敏感信息暴露给攻击者。任何能够访问到class文件的人都可以反编译class文件并发现这些敏感信息。因此，不能将信息硬编码在程序中。同时，硬编码敏感信息会增加代码管理和维护的难度。例如，在一个已经部署的程序中修改一个硬编码的口令需要发布一个补丁才能实现。

错误示例：

```java
public class IPaddress
{
    private String ipAddress = "172.16.254.1";
    
    public static void main(String[] args)
    {
        //...
    }
    
}
```

恶意用户可以使用javap -c IPaddress命令来反编译class来发现其中硬编码的服务器IP地址。反编译器的输出信息透露了服务器的明文IP地址：172.16.254.1。

正确示例：

```java
public class IPaddress
{
    public static void main(String[] args) throws IOException
    {
        char[] ipAddress = new char[100];
        BufferedReader br = new BufferedReader(new InputStreamReader(
                new FileInputStream("serveripaddress.txt")));
        
        // Reads the server IP address into the char array,
        // returns the number of bytes read 
        int n = br.read(ipAddress);
        // Validate server IP address
        // Manually clear out the server IP address
        // immediately after use 
        for (int i = n - 1; i >= 0; i--)
        {
            ipAddress[i] = 0;
        }
        br.close();
    }   
}
```

这个正确代码示例从一个安全目录下的外部文件获取服务器IP地址。并在其使用完后立即从内存中将其清除可以防止后续的信息泄露。

### 9.5.	不安全的随机数&#xD;

说明: 伪随机数生成器（PRNG）使用确定性数学算法来产生具有良好统计属性的数字序列。但是这种数字序列并不具有真正的随机特性。伪随机数生成器通常以一个算术种子值为起始。算法使用该种子值生成一个输出以及一个新的种子，这个种子又被用来生成下一个随机值，以此类推。

Java API 提供了伪随机数生成器（PRNG）—— java.util.Random类。这个伪随机数生成器具有可移植性和可重复性。因此，如果两个java.util.Random类的实例创建时使用的是相同的种子值，那么对于所有的Java实现，它们将生成相同的数字序列。在系统重启或应用程序初始化时，Seed值总是被重复使用。在一些其他情况下，seed值来自系统时钟的当前时间。攻击者可以在系统的一些安全脆弱点上监听，并构建相应的查询表预测将要使用的seed值。

因此，java.util.Random类不能用于安全敏感应用或者敏感数据保护。应使用更加安全的随机数生成器，例如java.security.SecureRandom类。

正确示例：

```java
public byte[] genRandBytes(int len)
{
	byte[] bytes = null;
	if (len > 0 && len < 1024)
	{
		bytes = new byte[len];
		SecureRandom random = new SecureRandom();
		random.nextBytes(bytes);
	}
	return bytes;
}
```

### 9.6.	不安全的Socket &#xD;

说明：当在不安全的传输通道中传输敏感数据时，程序必须使用javax.net.ssl.SSLSocket类，而不能是java.net.Socket类。SSLSocket类提供了诸如SSL/TLS等安全协议来保证通道不受监听和恶意篡改的影响。

Socket不提供但是SSLSocket提供的主要保护包括：

* 完整性保护：SSL防止消息被主动窃取者篡改。
* 认证：在大多数模式下，SSL都对对端进行认证。服务器通常都被认证，如果服务器要求，客户端也可以被认证。
* 保密性（隐私保护）：在大多数模式下，SSL对客户端和服务器之间传输的数据进行加密。 这样保护了数据的保密性，被动窃听器不能监听诸如财务或者个人信息之类的敏感信息。

错误示例：

```java
//Exception handling has been omitted for the sake of brevity
class EchoServer
{
    public static void main(String[] args) throws IOException
    {
        //Exception handling has been omitted for the sake of brevity
        //...
        ServerSocket serverSocket = new ServerSocket(9999);
        Socket socket = serverSocket.accept();
        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
        BufferedReader in = new BufferedReader(new InputStreamReader(
                    socket.getInputStream()));
        String inputLine;
        while ((inputLine = in.readLine()) != null)
        {
                System.out.println(inputLine);
                out.println(inputLine);
        }
        // ...
    }
    // ...
}

class EchoClient
{
    public static void main(String[] args) throws UnknownHostException,
            IOException
    {
        // Exception handling has been omitted for the sake of brevity
        // ...
        Socket socket = new Socket(getServerIp(), 9999);
        PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
        BufferedReader in = new BufferedReader(new InputStreamReader(
                    socket.getInputStream()));
        BufferedReader stdIn = new BufferedReader(new InputStreamReader(
                    System.in));
        String userInput;
        while ((userInput = stdIn.readLine()) != null)
        {
            out.println(userInput);
            System.out.println(in.readLine());
        }
        // ...
    }
    // ...
}
```

正确示例：

```java
class EchoServer
{
    public static void main(String[] args) throws IOException
    {
        // Exception handling has been omitted for the sake of brevity
        // ...
        SSLServerSocket SSLServerSocketFactory sslServerSocketFactory = (SSLServerSocketFactory) SSLServerSocketFactory.getDefault();
        sslServerSocket = (SSLServerSocket) sslServerSocketFactory.createServerSocket(9999);
        SSLSocket sslSocket = (SSLSocket) sslServerSocket.accept();
        PrintWriter out = new PrintWriter(sslSocket.getOutputStream(), true);
        BufferedReader in = new BufferedReader(new InputStreamReader(
                    sslSocket.getInputStream()));
        String inputLine;
        while ((inputLine = in.readLine()) != null)
        {
            System.out.println(inputLine);
            out.println(inputLine);
        }
        // ...
    }
    // ...
}
class EchoClient
{
    public static void main(String[] args) throws IOException
    {
        // Exception handling has been omitted for the sake of brevity
        // ...
        SSLSocket SSLSocketFactory sslSocketFactory = (SSLSocketFactory) SSLSocketFactory.getDefault();
        sslSocket = (SSLSocket) sslSocketFactory.createSocket(getServerIp(),
                    9999);
        PrintWriter out = new PrintWriter(sslSocket.getOutputStream(), true);
        BufferedReader in = new BufferedReader(new InputStreamReader(
                    sslSocket.getInputStream()));
        BufferedReader stdIn = new BufferedReader(new InputStreamReader(
                    System.in));
        String userInput;
        while ((userInput = stdIn.readLine()) != null)
        {
            out.println(userInput);
            System.out.println(in.readLine());
        }
        // ...
    }
    // ...
}
```

该正确代码示例应用SSLSocket来使用SSL/TLS安全协议保护传输的报文。使用SSLSocket的程序如果尝试连接不使用SSL的端口，那么这个程序将无限期被阻塞。同理，一个不使用SSLSocket的程序如果要同一个使用SSL的端口建立连接也将会被阻塞。.

例外情况：

因为SSLSocket提供的报文安全传输机制性，将造成巨大的性能开销。在以下情况下，普通的套接字就可以满足需求：

* 套接字上传输的数据不敏感。
* 数据虽然敏感，但是已经过恰当加密
* 套接字的网络路径从来不越出信任边界。这种情况只有在特定的情况下才能发生。例如，套接字的两端都在同一个本例网络，而且整个网络都是可信的情况时。

### 9.7.	不安全的第三方组件&#xD;

说明: 组件（例如：库、框架和其他软件模块）拥有和应用程序相同的权限。如果应用程序中含有已知漏洞的组件被攻击者利用，可能会造成严重的数据丢失或服务器接管。同时，使用含有已知漏洞的组件的应用程序和API可能会破坏应用程序防御、造成各种攻击并产生严重影响。

审计工具：

Dependency-Check是OWASP（Open Web Application Security Project）的一个实用开源程序，用于识别项目依赖项并检查是否存在任何已知的，公开披露的漏洞。目前，已支持Java、.NET、Ruby、Node.js、Python等语言编写的程序，并为C/C++构建系统（autoconf和cmake）提供了有限的支持。而且该工具还是OWASP Top 10的解决方案的一部分。

Dependency-Check支持面广（支持多种语言）、可集成性强，作为一款开源工具，在多年来的发展中已经支持和许多主流的软件进行集成，比如：命令行、Ant、Maven、Gradle、Jenkins、Sonar等；具备使用方便，落地简单等优势。

项目地址：https://owasp.org/www-project-dependency-check/

```
##   windows：
.\dependency-check.bat --project 项目名称 -s lib库的路径 -o 报告保存路径
##   linux：
bash dependency-check.sh --project 项目名称 -s lib库的路径 -o 报告保存路径
```

附录（代码审计常见关键字）


下表中总结代码审计中逆向审计常用关键字：

![](<../../.gitbook/assets/image (979).png>)
