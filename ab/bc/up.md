# uploadfile-labs

## 一、摘要

项目地址： [https://github.com/c0ny1/upload-labs](https://github.com/c0ny1/upload-labs)

![&#x9776;&#x673A;&#x5305;&#x542B;&#x6F0F;&#x6D1E;&#x7C7B;&#x578B;](../../.gitbook/assets/image%20%28608%29.png)

![&#x5224;&#x65AD;&#x4E0A;&#x4F20;&#x7C7B;&#x578B;](../../.gitbook/assets/image%20%28611%29.png)



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

![](../../.gitbook/assets/image%20%28610%29.png)

2、直接在页面中选择 XX.php 的文件，上传

![](../../.gitbook/assets/image%20%28612%29.png)

3、新标签中打开文件，即可看到上传好的文件

![](../../.gitbook/assets/image%20%28606%29.png)
{% endtab %}

{% tab title="方法二" %}
1、在本地文件夹中，将 .php 后缀的文件改为 .jpg 等后缀，打开 burp 抓包然后点击上传

2、在burp中，将所上传 .jpg 后缀的文件更换回 .php ，点击放包

![](../../.gitbook/assets/image%20%28609%29.png)

3、在新标签中打开文件，可看到上传好的文件

![](../../.gitbook/assets/image%20%28606%29.png)
{% endtab %}
{% endtabs %}



### pass-02



### pass-03

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

