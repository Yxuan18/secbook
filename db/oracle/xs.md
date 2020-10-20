# webshell写入

## **1、必备条件**

① 有DBA权限  
② 获得Web目录的绝对路径

## **2、写入方法**

向Oracle写入WebShell的方法可以使用：文件访问包

## **3、使用文件访问包方法写入Webshell**

首先我们需要创建一个ORACLE的目录对象指向某一路径，在真实环境中需要指向Web目录下，在这里我们将其指向/home/oracle这一路径下。

```text
create or replace directory IST0_DIR as '/home/oracle';
```

![](https://image.3001.net/images/20200810/1597038560.png!small)

创建好后，我们需要对其进行一下授权过程，让其能够顺利的写入WebShell代码。

```text
grant read, write on directory IST0_DIR tosystem;
```

然后写入文件，定义变量类型为utl\_file.file\_type，然后将WebShell的代码写入此文件中。

![](https://image.3001.net/images/20200810/1597038590.png!small)

直接访问该文件，即可查看到其中的WebShell代码，如果这个文件是放置在Web目录下的，那么就可以被攻击者成功利用。

