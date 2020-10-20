# upload-labs

## 一、摘要

项目地址： [https://github.com/c0ny1/upload-labs](https://github.com/c0ny1/upload-labs)

![&#x9776;&#x673A;&#x5305;&#x542B;&#x6F0F;&#x6D1E;&#x7C7B;&#x578B;](../../.gitbook/assets/image%20%28608%29.png)

![&#x5224;&#x65AD;&#x4E0A;&#x4F20;&#x7C7B;&#x578B;](../../.gitbook/assets/image%20%28614%29.png)



## 二、关卡及说明

### pass-01

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

### pass-02

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

### pass-03

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

### pass-04

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

### pass-05

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

其中，禁止了 .htaccess ，只是在大小写方面没有太多限制，于是，可上传 .Php 结尾的文件。上传成功后，访问：



### pass-06

### pass-07

### pass-08

### pass-09

### pass-10

### pass-11

### pass-12

### pass-13

### pass-14

### pass-15

### pass-16

### pass-17

### pass-18

### pass-19

### pass-20

## 三、文件上传对应防御手段

1. 黑白名单；
2. 对上传的文件重命名，不易被猜测；
3. 对上传的内容进行读取检查；
4. 不要暴露上传文件的位置；
5. 禁用上传文件的执行权限；

### 详细版本：

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



