# uploadfile-labs

## 一、摘要

项目地址： [https://github.com/c0ny1/upload-labs](https://github.com/c0ny1/upload-labs)

![&#x9776;&#x673A;&#x5305;&#x542B;&#x6F0F;&#x6D1E;&#x7C7B;&#x578B;](../../.gitbook/assets/image%20%28608%29.png)

![&#x5224;&#x65AD;&#x4E0A;&#x4F20;&#x7C7B;&#x578B;](../../.gitbook/assets/image%20%28613%29.png)



## 二、关卡及说明

### pass-01

 说明：该关卡在前端在JS中对上传文件的后缀进行限制

 步骤：

1、点击F12或查看源码，可看到JavaScript字段，其中对可上传文件后缀做了限制

![](../../.gitbook/assets/image%20%28607%29.png)

2、上传步骤如下：

{% tabs %}
{% tab title="方法一" %}
1、使用burp抓包，在返回包中去掉JavaScript字段，然后放包

![](../../.gitbook/assets/image%20%28612%29.png)

2、直接在页面中选择 XX.php 的文件，上传

![](../../.gitbook/assets/image%20%28614%29.png)

3、新标签中打开文件，即可看到上传好的文件

![](../../.gitbook/assets/image%20%28606%29.png)
{% endtab %}

{% tab title="方法二" %}
1、在本地文件夹中，将 .php 后缀的文件改为 .jpg 等后缀，打开 burp 抓包然后点击上传

2、在burp中，将所上传 .jpg 后缀的文件更换回 .php ，点击放包

![](../../.gitbook/assets/image%20%28611%29.png)

3、在新标签中打开文件，可看到上传好的文件

![](../../.gitbook/assets/image%20%28606%29.png)
{% endtab %}
{% endtabs %}

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

![pass01&#x4E3A;&#x4F8B;](../../.gitbook/assets/image%20%28610%29.png)

1、故在上传文件时，只需要将 .php 文件中的`Content-Type`部分更改为允许的MIME即可，如下图

![&#x4FEE;&#x6539;&#x540E;&#xFF0C;&#x653E;&#x5305;](../../.gitbook/assets/image%20%28616%29.png)

2、新标签页中打开文件，如图：

![](../../.gitbook/assets/image%20%28615%29.png)

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

### pass-04

### pass-05

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

