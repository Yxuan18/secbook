# 信息泄露漏洞\_java

## 什么是信息泄露

是指网站无意间向用户泄露敏感信息。根据上下文，网站可能会将各种信息泄漏给潜在的攻击者，包括：

* 有关其他用户的数据，例如用户名或财务信息
* 敏感的商业或商业数据
* 有关网站及其基础架构的技术细节

泄露敏感的用户或业务数据的危险相当明显，但泄露技术信息有时可能同样严重。尽管某些信息用途有限，但它可能是暴露其他攻击面的起点，其中可能包含其他有趣的漏洞。

有时，敏感信息可能会不慎泄露给仅以正常方式浏览网站的用户。但是，更常见的是，攻击者需要通过以意外或恶意的方式与网站进行交互来引发信息泄露。然后，他们将仔细研究网站的响应，以尝试找出有趣的行为。

## 基本示例

信息披露的一些基本示例如下：

* 通过robots.txt文件或目录列表显示 隐藏目录的名称、结构和内容
* 通过临时备份提供对源代码文件的访问
* 在错误消息中明确提及数据库表或列名称
* 不必要地暴露高度敏感的信息，例如信用卡详细信息
* 在源代码中硬编码 API 密钥、IP 地址、数据库凭据等
* 通过应用程序行为的细微差异暗示资源、用户名等的存在或不存在

![](../../.gitbook/assets/image%20%281087%29.png)

由于Coremail邮件系统的mailsms模块的参数大小写敏感存在缺陷，使得攻击者利用该漏洞，在未授权的情况下，通过远程访问URL地址获知Coremail服务器的系统配置文件，造成数据库连接参数等系统敏感配置信息泄露

## 测试方法

### 模糊测试：

如果您发现有趣的参数，您可以尝试提交意外的数据类型和特制的模糊字符串，看看这有什么影响。密切关注; 尽管响应有时会明确披露有趣的信息，但它们也可以更巧妙地暗示应用程序的行为

可以使用 Burp Intruder 等工具自动执行此过程的大部分工作。•

可以使用 Burp Scanner。这提供了在您浏览时审核项目的实时扫描功能，或者您可以安排自动扫描以代表您抓取和审核目标站点。这两种方法都会自动为您标记许多信息泄露漏洞。例如，Burp Scanner 会在响应中发现敏感信息（例如私钥、电子邮件地址和信用卡号）时提醒您。它还将识别任何备份文件、目录列表等

## 漏洞是如何产生的

信息披露漏洞可能以无数不同的方式出现，但大致可以分为以下几类：

* 未能从公共内容中删除内部内容。例如，标记中的开发人员评论有时对生产环境中的用户可见。
*  网站和相关技术的不安全配置。例如，未能禁用调试和诊断功能有时可以为攻击者提供有用的工具来帮助他们获取敏感信息。
* 默认配置也可能使网站容易受到攻击，例如，显示过于冗长的错误消息。 
* 有缺陷的应用程序设计和行为。例如，如果网站在出现不同的错误状态时返回不同的响应，这也可能允许攻击者枚举敏感数据，例如有效的用户凭据。

## 漏洞常见来源

信息披露可能发生在网站内的各种环境中。以下是您可以查看敏感信息是否暴露的一些常见位置示例。

* 网络爬虫文件：例如 /robots.txt，/sitemap.xml
* 目录列表
* 开发者评论
* 错误消息
* 调试数据
* 用户帐户页面
* 备份文件
* 不安全的配置
* 版本控制历史：例如 /.git  /.svn 等

### 错误信息与代码逻辑

当黑客在登录某个页面时，在用户名位置输入一个单引号，在密码位置输入一个“g”之后，就会出现如下的错误信息。

```java
An Error Has Occurred. Error Message: 
System.Data.OleDb.OleDbException: Syntax error (missing operator) in query expression 'username = ''' and password = 'g''. at 
System.Data.OleDb.OleDbCommand.ExecuteCommandTextErrorHandling ( Int32 hr) at System.Data.OleDb.OleDbCommand.ExecuteCommandTextForSingleResult ( tagDBPARAMS dbParams, Object& executeResult) at 

```

从这个错误信息中，我们可以看到，网页最终执行了一个 SQL 语句，这个 SQL 语句的部分内容为username = ''' and password = 'g'。

第一，错误信息反馈的是 Syntax error，即语法错误。在密码位置输入单个字母“g”肯定不会引起错误，所以，这个 SQL 语句是因为多了一个单引号导致的报错。而如果使用了 PreparedStatement 等方法，是不会产生这个错误的。因此，后台的 SQL 查询应该是直接采用的字符串拼接，且没有过滤单引号。

第二，错误信息中显示了部分的 WHERE 条件是username = '' and password = ''。这又是一个登录的逻辑，所以，只要用户名和密码正确，这个 SQL 语句会返回黑客需要的用户信息。因此，后台的 SQL 语句应该是形如 select from where 的格式。根据这些信息，黑客很容易就可以发起 SQL 注入攻击了。

### 如何避免前端显示？

PHP:

```text
error_reporting  =  E_ALL;		向PHP报告发生的每个错误   
display_errors = Off;		不显示满足上条指令所定义规则的所有错误报告   
log_errors = On;			决定日志语句记录的位置   
log_errors_max_len = 1024;		设置每个日志项的最大长度   
error_log = /var/log/php_error.log;	指定产生的错误报告写入的日志文件位置
```

JAVA Spring: 配置ExceptionHandler等

## 如何使用 Spring 为 REST API 实现异常处理

Spring 3.2 之前，在 Spring MVC 应用程序中处理异常的两种主要方法是HandlerExceptionResolver或@ExceptionHandler注释。

从 3.2 开始，可使用@ControllerAdvice注释

Spring 5 引入了 ResponseStatusException 类

### 方案一：控制器级@ExceptionHandler

适用于@Controller级别。我们将定义一个方法来处理异常并使用@ExceptionHandler 对其进行注释：

```java
public class FooController{
    //...
    @ExceptionHandler({ CustomException1.class, CustomException2.class })
    public void handleException() {
        //
    }
}
```

这种方法有一个很大的缺点：@ExceptionHandler注解的方法才会被激活针对特定的控制器，而不是全局为整个应用程序。当然，将它添加到每个控制器使其不太适合通用异常处理机制。

我们可以通过让所有控制器扩展一个基本控制器类来解决这个限制。

### 方案二：HandlerExceptionResolver

定义一个HandlerExceptionResolver。这将解决应用程序抛出的任何异常。它还允许我们在 REST API 中实现统一的**异常处理机制**。

### 方案三：@ControllerAdvice

Spring 3.2 支持带有 @ControllerAdvice注释的全局@ExceptionHandler。

这启用了一种脱离旧 MVC 模型并利用ResponseEntity以及@ExceptionHandler的类型安全性和灵活性的机制：

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler 
  extends ResponseEntityExceptionHandler {
    @ExceptionHandler(value 
      = { IllegalArgumentException.class, IllegalStateException.class })
    protected ResponseEntity<Object> handleConflict(
      RuntimeException ex, WebRequest request) {
        String bodyOfResponse = "This should be application specific";
        return handleExceptionInternal(ex, bodyOfResponse, 
          new HttpHeaders(), HttpStatus.CONFLICT, request);
    }
}
```

该@ControllerAdvice注释使我们能够巩固我们的源码，@ExceptionHandler是全球性的错误处理组件。

实现的机制非常简单，但也非常灵活：

* 它使我们可以完全控制响应的主体以及状态代码。
* 它提供了多个异常到同一方法的映射，以便一起处理。
* 它很好地利用了较新的 RESTful ResposeEntity响应。

要记住的一件事是：  
 将用@ExceptionHandler声明的异常与用作方法参数的异常进行匹配。

### 解决方案4：ResponseStatusException （Spring 5及以上）

Spring 5 引入了ResponseStatusException类。

我们可以创建它的一个实例，提供HttpStatus和可选的原因和原因。

```java
@GetMapping(value = "/{id}")
public Foo findById(@PathVariable("id") Long id, HttpServletResponse response) {
    try {
        Foo resourceById = RestPreconditions.checkFound(service.findOne(id));

        eventPublisher.publishEvent(new SingleResourceRetrievedEvent(this, response));
        return resourceById;
     }
    catch (MyResourceNotFoundException exc) {
         throw new ResponseStatusException(
           HttpStatus.NOT_FOUND, "Foo Not Found", exc);
    }
}
```

使用ResponseStatusException 有什么好处？•非常适合原型设计：我们可以非常快速地实现基本解决方案。

* 一种类型，多种状态代码：一种异常类型可以导致多种不同的响应。与@ExceptionHandler相比，这减少了紧密耦合。
* 我们不必创建那么多的自定义异常类。
* 我们可以更好地控制异常处理，因为可以通过编程方式创建异常。

那么权衡呢？

* 没有统一的异常处理方式：与提供全局方法的@ControllerAdvice 相比，强制执行一些应用程序范围的约定更加困难。
* 代码重复：我们可能会发现自己在多个控制器中复制代码。

我们还应该注意到，可以在一个应用程序中组合不同的方法。

例如，我们可以全局实现@ControllerAdvice，但也可以 在本地实现 ResponseStatusException。

但是，我们需要小心：如果可以通过多种方式处理同一个异常，我们可能会注意到一些令人惊讶的行为。一种可能的约定是始终以一种方式处理一种特定类型的异常。

## 处理Spring Security中拒绝访问

当经过身份验证的用户尝试访问他没有足够权限访问的资源时，会发生访问被拒绝。

详情有：

* MVC - 自定义错误页面
* 自定义AccessDeniedHandler

### MVC - 自定义错误页面

XML 配置：

```markup
<http>
    <intercept-url pattern="/admin/*" access="hasAnyRole('ROLE_ADMIN')"/>   
    ... 
    <access-denied-handler error-page="/my-error-page" />
</http>

```

Java配置：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/admin/*").hasAnyRole("ROLE_ADMIN")
        ...
        .and()
        .exceptionHandling().accessDeniedPage("/my-error-page");
}
```

当用户在没有足够权限的情况下尝试访问资源时，他们将被重定向到“/my-error-page”。

### 自定义AccessDeniedHandler

如何编写自定义AccessDeniedHandler：

```java
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle
      (HttpServletRequest request, HttpServletResponse response, AccessDeniedException ex) 
      throws IOException, ServletException {
        response.sendRedirect("/my-error-page");
    }
}
```

使用XML 配置：

```markup
<http>
    <intercept-url pattern="/admin/*" access="hasAnyRole('ROLE_ADMIN')"/> 
    ...
    <access-denied-handler ref="customAccessDeniedHandler" />
</http>

```

## Spring Boot 支持

Spring Boot 提供了一个 ErrorController实现来以合理的方式处理错误。

简而言之，它为浏览器提供后备错误页面（又名白标错误页面），并为 RESTful 非 HTML 请求提供 JSON 响应：

```javascript
{
    "timestamp": "2019-01-17T16:12:45.977+0000",
    "status": 500,
    "error": "Internal Server Error",
    "message": "Error processing the request!",
    "path": "/my-endpoint-with-exceptions"
}
```

像往常一样，Spring Boot 允许使用属性配置这些功能：

* server.error.whitelabel.enabled：可用于禁用 Whitelabel 错误页面并依赖 servlet 容器提供 HTML 错误消息
* server.error.include-stacktrace：具有始终 值；在 HTML 和 JSON 默认响应中包含堆栈跟踪
* server.error.include-message： 从2.3版本开始，Spring Boot在响应中隐藏了 message字段，避免泄露敏感信息；我们可以使用带有always 值的属性 来启用它

除了这些属性之外，我们还可以为 /错误提供我们自己的视图解析器映射， 覆盖白标签页面。

我们还可以通过在上下文中包含一个ErrorAttributes bean 来自定义要在响应中显示的属性 。我们可以扩展Spring Boot 提供的 DefaultErrorAttributes类来使事情变得更容易

```java
@Component
public class MyCustomErrorAttributes extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(
      WebRequest webRequest, ErrorAttributeOptions options) {
        Map<String, Object> errorAttributes = 
          super.getErrorAttributes(webRequest, options);
        errorAttributes.put("locale", webRequest.getLocale()
            .toString());
        errorAttributes.remove("error");

        //...
        return errorAttributes;
    }
}
```

如果我们想进一步定义（或覆盖）应用程序将如何处理特定内容类型的错误，我们可以注册一个 ErrorController  bean。

同样，我们可以利用Spring Boot 提供的默认 BasicErrorController 来帮助我们。

例如，假设我们想要自定义我们的应用程序如何处理在 XML 端点中触发的错误。我们所要做的就是使用@RequestMapping定义一个公共方法 ，并声明它产生application/xml媒体类型：

```java
@Component
public class MyErrorController extends BasicErrorController {

    public MyErrorController(
      ErrorAttributes errorAttributes, ServerProperties serverProperties) {
        super(errorAttributes, serverProperties.getError());
    }

    @RequestMapping(produces = MediaType.APPLICATION_XML_VALUE)
    public ResponseEntity<Map<String, Object>> xmlError(HttpServletRequest request) {
        
    // ...

    }
}
```

这里我们仍然依赖于在项目中定义的server.error.\*引导属性，这些属性绑定到ServerProperties bean。

## 除了错误信息，还有什么地方会泄露代码逻辑

除了错误信息之外，间接的信息泄露方式还有两种：返回信息泄露和注释信息泄露。

### 注释信息

所有的前端代码基本都不需要编译就可以展示在浏览器中，所以黑客很容易就可以看到前端代码中的注释信息。但是，如果这些注释信息中出现服务器 IP、数据库地址和认证密码这样的关键信息。一旦这些关键信息被泄露，将会造成十分严重的后果。

系统中存在大量JSP页面当中注释采用HTML注释，而不是JSP注释的情况。 此情况下，攻击者在浏览器浏览页面的时候，右键查看页面源代码，可以看到HTML注释内的信息，很多时候甚至是代码块，导致系统信息泄露。

Jsp html 注释区别：

1. HTML页面注释-----这里面的注释会被编译（加载页面时，会进行语法判断，取需要的变量默认值
2. jsp页面注释------这里之后的注释都不会被编译 &lt;%--JSP中的注释，这里面的内容在查看页面源代码时，看不到这里面注释书写的内容 --%&gt; 所以涉及业务的建议使用&lt;%-- --%&gt;注释，文字描述性的使用注释。

### 返回信息

#### 场景一：（SSRF）

服务端在请求一个图片地址的时候，会根据地址的“存活”情况和返回数据的类型，分别返回三种结果：“图片不存在”“格式错误”以及图片正常显示。而黑客正是通过服务端返回信息的逻辑，利用一个请求图片的 SSRF，摸清整个后端服务的“存活情况”。

#### 场景二：

当在登录应用的时候，应用的返回逻辑可能是这样的：如果输入的用户名和密码正确，则登录成功；如果应用没有这个用户，则返回“用户名不存在”；如果输入的用户名和密码不匹配，则返回“密码错误”。

尽管这样清晰的登录提示对于用户体验来说，确实是一个较优解，但这个逻辑同样也暴露了过多的信息给黑客。黑客只需要不断地发起登录请求，就能够知道应用中存在的用户名，然后通过遍历常见的弱密码进行尝试，很容易就能够猜对密码。这样一来，猜对密码的成功率就比尝试同时猜测用户名和密码要高很多。

## 有哪些常见的直接泄露方式

常见的方式有：

1. 版本管理工具中的隐藏文件
2. 上传代码到 GitHub等开源平台

### 隐藏文件

SVN 会在项目目录中创建一个.svn 文件夹，里面保存了应用每一个版本的源文件信息，这也是 SVN 实现代码回滚的数据基础。如果 SVN 可以通过.svn 中的数据提取应用任意版本的代码，那黑客也可以。只要你没有在上线代码的时候删除其中的.svn 目录，那就代表黑客可以通过.svn 中的 URL 访问里面的所有文件。接下来，只需要通过执行简单的脚本，黑客就可以回溯出一个完整版本的代码了。

#### .DS\_Store

.DS\_Store\(英文全称 Desktop Services Store\)是一种由苹果公司的Mac OS X操作系统所创造的隐藏文件，目的在于存储目录的自定义属性，例如文件的图标位置或者是背景色的选择。相当于 Windows 下的 desktop.ini

互联网上有.DS\_Store 文件泄漏利用脚本，它解析.DS\_Store文件并递归地下载文件到本地。

#### .git

GitHack是一个.git泄露利用脚本，通过泄露的.git文件夹下的文件，重建还原工程源代码。

工具相关原理：

* 解析.git/index文件，找到工程中所有的： \( 文件名，文件sha1 \)
* 去.git/objects/ 文件夹下下载对应的文件
* zlib解压文件，按原始的目录结构写入源代码

对于这种因为目录中额外内容（.svn/.git）导致的源码泄露，我们一方面需要对线上代码进行人工的代码审查，确保无关的文件和文件夹被正确地清除；另一方面，我们也可以在 HTTP 服务中对部分敏感的路径进行限制。比如，在 Apache httpd 中配置下面的内容，来禁止黑客对.svn 和.git 目录的访问

```markup
<DirectoryMatch \.(svn|git)>
  Order allow,deny
  Deny from all
</DirectoryMatch>

```

### 开源代码托管平台泄露

使用 GitHub 上传代码通常属于个人行为，所以，我们很难从技术层面上进行预防。

对这种情况，公司应该从加强员工安全意识的培训、强化公司管理制度入手，避免员工私自上传代码。除此之外，公司还可以对 GitHub 发起巡检（比较知名的工具有Hawkeye），通过定期检索公司代码的关键字（比如常用的包名、域名等）来进行检测。通过这些方式匹配到的结果，很可能就是员工私自公开的代码。确认之后，我们就可以联系上传的人员进行删除。

## 那么间接泄露呢？

列举几个： 

1. Apache样例文件泄露  apache 的一些样例文件没有删除，可能存在cookie、session伪造，进行后台登录操作
2. 字段加\[\] 造成信息泄露  数据包的所有参数都可能存在，把参数变为数组即可报错出网站绝对路径。 
3. 修改请求方法  将get方法改为put 程序就报错，泄漏网站绝对路径。 
4. tomcat  tomcat 后缀改成大写，会显示源码，只要把jsp文件后缀名改为大写就可以 
5. war文件信息泄露  war文件信息泄露是指部署在war文件由于配置不当，导致其整个报文件以及其他重要的配置文件信息泄露，例如可以直接浏览目录，获取其下面的配置文件：WEB-INF/jdbc.properties,jdbc.properties为数据库链接配置文件。包含数据库链接的账户和密码等重要信息。

## 那么，还有吗？

旁路信息泄漏：  
 登录时，对于无效用户和有效用户的登录请求，如果服务端处理耗时不一样也会泄漏信息。

padding oracle攻击：  
 只要服务端返回的信息可以区分解密成功与否，就可以在没有密钥的情况下经过有限次尝试枚举出解密后的信息

## 漏洞有什么影响？

信息泄露漏洞可以产生直接和间接影响，具体取决于网站的目的以及攻击者能够获取的信息。在某些情况下，仅披露敏感信息的行为就会对受影响的各方产生重大影响。例如，一家网上商店泄露其客户的信用卡详细信息可能会产生严重的后果。•

另一方面，泄露技术信息，例如目录结构或正在使用的第三方框架，可能几乎没有直接影响。但是，如果落入坏人之手，这可能是构建任意数量的其他漏洞利用所需的关键信息。这种情况下的严重性取决于攻击者能够使用这些信息做什么。

## 如何评估信息泄露漏洞的严重性

尽管最终影响可能非常严重，但只有在特定情况下，信息披露本身才是一个严重的问题。在测试期间，技术信息的披露通常只有在您能够证明攻击者如何利用它做一些有害的事情时才有意义。

例如，如果该版本已完全修补，则网站正在使用特定框架版本的知识的用途是有限的。但是，当网站使用包含已知漏洞的旧版本时，此信息变得很重要。在这种情况下，执行破坏性攻击可能就像应用公开记录的漏洞一样简单。

当您发现潜在的敏感信息被泄露时，运用常识很重要。在您测试的许多网站上，很可能可以通过多种方式发现次要的技术细节。因此，您的主要关注点应该是泄露信息的影响和可利用性，而不仅仅是信息披露作为一个独立问题的存在。一个明显的例外是泄露的信息非常敏感，以至于它本身就值得关注。

## 如何防范

1. 由于信息泄露的发生方式多种多样，因此完全防止信息泄露非常棘手。但是，您可以遵循一些通用的最佳实践来最大程度地降低此类漏洞潜入您自己网站的风险。 
2. 确保参与制作网站的每个人都完全了解哪些信息被视为敏感信息。有时，看似无害的信息对攻击者来说可能比人们意识到的要有用得多。突出显示这些危险有助于确保您的组织在一般情况下更安全地处理敏感信息。 
3. 作为 QA 或构建过程的一部分，审计任何潜在信息泄露的代码。自动化一些相关任务应该相对容易，例如剥离开发人员评论。 
4. 尽可能使用通用错误消息。不要向攻击者提供有关应用程序行为的不必要的线索。 
5. 仔细检查在生产环境中是否禁用了任何调试或诊断功能。 
6. 确保您完全了解您实施的任何第三方技术的配置设置和安全影响。花点时间调查并禁用您实际上不需要的任何功能和设置。

