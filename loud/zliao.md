# 技术分享预备资料

## 一、暴力破解

### 常见模块

登录认证，密码找回

使用登录认证的行业有：金融行业，互联网行业，政务，电商，P2P等

概念：针对应用系统用户登录账号与密码进行的穷举测试，针对账号或密码进行逐一比较，直到找出正确的的账号与密码 

![&#x793A;&#x4F8B;](../.gitbook/assets/image%20%281000%29.png)

### 情况 

1、已知账号，加载密码字典针对密码进行穷举测试   
2、未知账号，加载账号字典，结合密码字典进行穷举测试   
3、未知账号与密码，加载账号字典与密码字典进行穷举测试 

安全建议：   
1、增加验证码，登录失败一次，验证码变换一次   
2、配置登录失败次数限制策略，如在同一用户尝试登录的情况下，5 分钟内连续登录失败超过6次，则禁止此用户在3小时内登录系统。   
3、在条件允许的情况下，增加手机接收短信验证码或邮箱接收邮件验证码，实现双因素认证的防暴力破解机制。

### 相关信息

#### 登陆失败信息

概念：在用户登录系统失败时，系统会在页面显示用户登录的失败信息，假如提交账号在系统中不存在，系统提示“用户名不存在”、“账号不存在”等明确信息；假如提交账号在系统中存在，则系统提示“密码/口令错误”等间接提示信息。攻击者可根据此类登录失败提示信息来判断当前登录账号是否在系统中存在，从而进行有针对性的暴力破解口令测试

安全建议：      
对系统登录失败提示语句表达内容进行统一的模糊描述，从而提高攻击者对登录系统用户名及密码的可猜测难度，如“登录账号或密码错误”、“系统登录失败”等

####  DVWA的防御机制

1、LOW

![&#x51E0;&#x4E4E;&#x6CA1;&#x6709;&#x9632;&#x5FA1;](../.gitbook/assets/image%20%281002%29.png)

2、Medium：加入了SQL字符转义逻辑，避免了SQL注入攻击，但是不能防御暴力破解

![](../.gitbook/assets/image%20%281001%29.png)

3、High：加入了Token机制，每次登录页面都会随机生成Token字串，那么无脑爆破就不可能了，因为Token是完全随机的。但是如果用更复杂的方法，每次先抓取当前页面Token值再随即进行字典式爆破，仍可以实现破解

![](../.gitbook/assets/image%20%281003%29.png)

4、Impossible：加入了账号锁定机制，数次登录失败后，账号会锁定，那么暴力破解就行不通了。可以说这是比较完善的防御机制

![](../.gitbook/assets/image%20%281005%29.png)



#### 验证码暴力破解

概念：主要被用于防止暴力破解、防止DDoS攻击、识别用户身份等，常见的验证码主要有图片验证码、邮件验证码、短信验证码、滑动验证码和语音验证码。以短信验证码为例。短信验证码大部分情况下是由4～6位数字组成，如果没有对验证码的失效时间和尝试失败的次数做限制，攻击者就可以通过尝试这个区间内的所有数字来进行暴力破解攻击

安全建议：

1、设置验证码的失效时间，建议为180秒；  
2、限制单位时间内验证码的失败尝试次数，如5分钟内连续失败5次即锁定该账号15分钟

#### PHP实现验证码

php.ini 中，取消extension=php\_gd2.dll前面的分号；并打开GD库

{% tabs %}
{% tab title="PHP文件" %}
```php
<?php
session_start();
//生成验证码图片
Header("Content-type: image/PNG");
$im = imagecreate(44,18);
$back = ImageColorAllocate($im, 245,245,245);
imagefill($im,0,0,$back); //背景
srand((double)microtime()*1000000);
//生成4位数字
for($i=0;$i<4;$i++){
$font = ImageColorAllocate($im, rand(100,255),rand(0,100),rand(100,255));
$authnum=rand(1,9);
$vcodes.=$authnum;
imagestring($im, 5, 2+$i*10, 1, $authnum, $font);
}
for($i=0;$i<100;$i++) //加入干扰象素
{
$randcolor = ImageColorallocate($im,rand(0,255),rand(0,255),rand(0,255));
imagesetpixel($im, rand()p , rand()0 , $randcolor);
}
ImagePNG($im);
ImageDestroy($im);
$_SESSION['Checknum'] = $vcodes;
?>
```
{% endtab %}

{% tab title="显示验证码" %}
```markup
<input type="text" name="passcode" >
<img src="code.php">
```
{% endtab %}

{% tab title="判断并获取值" %}
```javascript
...
session_start();//启动会话
$code=$_POST["passcode"];
if( $code == $_SESSION["Checknum"])
{
...
}
```
{% endtab %}
{% endtabs %}

目前广泛使用的验证码类型：（触发，嵌入，弹出）

![&#x7C7B;&#x578B;&#x4E0E;&#x4F7F;&#x7528;](../.gitbook/assets/image%20%28999%29.png)

还可使用人脸识别技术进行检测：

如：人脸对比，人脸检测，身份证OCR

案例：京训钉等

暴力攻击 漏洞描述  
暴力破解的基本思想是根据题目的部分条件确定答案的大致范围，并在此范围内对所有可能的情况逐一验证，直到全部情况验证完毕。若某个情况验证符合题目的全部条件，则为本问题的一个解；若全部情况验证后都不符合题目的全部条件，则本题无解。常常存在于网站的登录系统中，通过对已知的管理员用户名，进行对其登录口令的大量尝试。 

检测条件

1、已知Web网站具有登录页面。   
2、登录页面无验证码，无锁定机制。 

检测方法  
1、找到网站登录页面。   
2、利用burp对登录页面进行抓包，将其发送到Intruder,并设置其密码参数，如pwd=为变量，添加payload（字典），进行攻击，攻击过程中查看其返回的字节长度，来判断是否成功。 

修复方案  
防止暴力攻击的一些方法如下： 

1、账户锁定 账户锁定是很有效的方法，因为暴力破解程序在5-6次的探测中猜出密码的可能性很小。但是同时也拒绝了正常用户的使用。如果攻击者的探测是建立在用户名探测成功之后的行为，那么会造成严重的拒绝服务攻击。对于对大量用户名只用一个密码的探测攻击账户锁定无效。如果对已经锁定的账户并不返回任何信息，可能迷惑攻击者。   
2、返回信息 如果不管结果如何都返回成功的信息，破解软件就会停止攻击。但是对人来说很快就会被识破。   
3、页面跳转 产生登录错的的时候就跳到另一个页面要求重新登录。比如126和校内网都是这样做的。局限性在于不能总是跳转页面，一般只在第一次错误的时候跳转，但是第一次之后又可以继续暴力探测了。   
4、适当的延时 检查密码的时候适当的插入一些暂停，可以减缓攻击，但是可能对用户造成一定的影响。   
5、封锁多次登录的IP地址 这种方法也是有缺点的，因为攻击者可以定时更换自己的IP。   
6、验证码 验证码是阻止暴力攻击的好方法，但设计不好的验证码是可以绕过的，而且对于特定目标的手工探测来说验证码是没有作用的。





## 二、信息泄露

### 业务授权访问模块

非授权访问

概念：用户在没有通过认证授权的情况下能够直接访问需要通过认证才能访问到的页面或文本信息。可以尝试在登录某网站前台或后台之后，将相关的页面链接复制到其他浏览器或其他电脑上进行访问，观察是否能访问成功

安全建议：    

未授权访问可以理解为需要安全配置或权限认证的地址、授权页面存在缺陷，导致其他用户可以直接访问，从而引发重要权限可被操作、数据库、网站目录等敏感信息泄露，所以对未授权访问页面做Session认证，并对用户访问的每一个URL做身份鉴别，正确地校验用户ID及Token等

### 运维安全中的注意点：

* webserver
  * 列目录：敏感信息泄露
* 基础服务
  * SNMP：弱口令
  * RSYNC：任意文件读取
  * memcached：未授权访问
  * Redis信息泄露
  * elasticsearch信息泄露
  * Hadoop信息泄露
* 应用层
  * SVN信息泄露
* 账号密码

  * 代码泄露账号密码
    * GitHub中的自动化运维脚本
    * 博客中的代码截图
  * 公司内部账号注册外部业务，但其他公司被脱裤，密码相同导致信息泄露
  * 内部账号所使用的密码与外部相同

### 代码审计中可能出现的问题

1、安全功能中：请求参数没有限定范围导致信息泄露  
2、程序处理中：处理不恰当会造成信息泄露，或出现错误定位的问题

### 相关案例

1. kanboard项目管理软件默认口令

   Kanboard（看板）是一款优秀的项目管理软件，它可以让你能够更有效，更简单地管理项目。该系统存在默认口令，攻击者可利用默认口令进入系统，查看并管理系统中的项目，造成严重信息泄露。

2. [https://blog.csdn.net/songchaofly/article/details/103712917](https://blog.csdn.net/songchaofly/article/details/103712917)
3. [https://www.anquanke.com/post/id/86232](https://www.anquanke.com/post/id/86232)
4. [https://zhuanlan.zhihu.com/p/51176109](https://zhuanlan.zhihu.com/p/51176109)
5. [http://r3start.net/index.php/2019/07/15/546](http://r3start.net/index.php/2019/07/15/546)
6. [https://blog.csdn.net/weixin\_43534549/article/details/91361307?utm\_medium=distribute.pc\_relevant\_t0.none-task-blog-searchFromBaidu-1.control&depth\_1-utm\_source=distribute.pc\_relevant\_t0.none-task-blog-searchFromBaidu-1.control](https://blog.csdn.net/weixin_43534549/article/details/91361307?utm_medium=distribute.pc_relevant_t0.none-task-blog-searchFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-searchFromBaidu-1.control)

敏感数据泄露 分为： 目录浏览， web服务器控制台地址泄露， PHPinfo\(\) 信息泄露， POODLE 信息泄露， SVN 信息泄露， 备份文件泄露， 内网 IP 地址泄露， Cookie 信息泄露， 异常信息泄露， 敏感信息泄露， IIS 短文件名泄露， Robots 文件信息泄露  

