# SHELL编程

## 1、shell编程的作用

1. 自动化系统初始化（update，软件安装，时区统一设置，安全策略统一设置）
2. 自动化批量软件部署程序（LAMP，LNMP，TOMCAT，LVS，BGINX等）
3. 管理应用程序（KVM，集群管理扩容，MySQL的管理程序等）
4. 日志分析处理程序（PV量，UV，200返回的数量，非200返回的情况等），其中常用的有 top 100，grep,awk等
5. 编写自动化的备份/恢复程序（MySQL完全备份/增量备份）
6. 自动化管理程序（批量远程修改密码，软件升级，配置更新）
7. 自动化信息采集及监控程序（收集系统/应用的状态信息，CPU，MEM，DISK，NET，APACHE，TCP Status，MYSQL）
8. 自动化扩容（增加云主机→业务上线）例如：zabbix监控CPU80%+python API增加云主机+shell脚本上线服务
9. 配合ZABBIX信息采集 
10. 打印各种图形，通过键盘操控的小游戏等
11. 相关算法的实现以及排序的实现 
12. 理论上讲，shell可以做任何事，一切取决于需求

注意：

shell语法难度可能只有java等的二十分之一，其中所用到的工具的各种指令只有各软件的管理命令

## 2、C JAVA PYTHON shell执行方式对比

一个简单的shell程序：

```text
# ping01.sh

#!/usr/bin/bash
ping -c1 qq.com &>/dev/null && echo "its up" || echo "its down"
```

脚本中：

1. \`\#!/usr/bin/bash\`代表了指定解析程序的脚本，声明解释器。**不同的程序要使用不同的解释器去处理，且解释器只能在第一行**
2. &&gt;/dev/null：将命令的输出信息输出到NULL中，其中的`&`为混合输出，输出的是正确输出与错误输出
3. `;`表示的是排序 
4. `&&`会表示逻辑判断，and，只有前一个命令的`$?==0`，后面的命令才会执行 
5. `||`表示或者的意思，如果不为真，那么`||`后面的命令执行

![&#x811A;&#x672C;&#x53CA;&#x547D;&#x4EE4;&#x8FD0;&#x884C;&#x7ED3;&#x679C;](../../.gitbook/assets/image%20%28466%29.png)

.sh脚本执行方式：

```text
bash xx.sh
sh xx.sh
```

各种程序的解释器：

```text
bash:
#!/usr/bin/bash
#!/usr/bin/sh

python:
#!/usr/bin/python

C:    


java:
#!/usr/bin/java

Perl:
#!/usr/bin/Perl

expect:
#!/usr/bin/expect
```

其中，C语言的程序执行速度是最快的。

原因：

1. C语言进行编译后，会形成二进制机器码，可以被CPU直接执行
2. 程序是由逻辑与数据组成的
3. shell程序是解释型执行，shell不需要事先编译的，执行时只是解释执行，使用bash解释，执行时才考虑其中的逻辑关系
4. C语言程序在执行时已经是编译好的，不需要考虑逻辑关系
5. JAVA编译后为字节码（中间码），只能被java的虚拟机JDK执行（TOMCAT，HADOOP）
6. python执行方式：
   1. 编译执行：编译为python的字节码，由python虚拟机执行
   2. 解释执行：`/usr/bin/python`  ,python最多使用的，是解释执行

编译性语言的缺点：灵活性较低  
解释性语言可以直接修改源代码，然后直接执行，不需要编译

每个命令执行都有一个返回值，返回值是一个特殊的变量，变量是问号变量。`%?`表示引用它的值

| 变量名 | 含义 | 作用 |
| :--- | :--- | :--- |
| $? | 问号变量 | 返回上一条命令的执行结果，0为正常，127为错误 |

![](../../.gitbook/assets/image%20%28471%29.png)

## 3、bash中调用python expect

```text
# ping01.sh

#!/usr/bin/bash
ping -c1 qq.com &>/dev/null && echo "its up" || echo "its down"

/usr/bin/python <<-EOF
print("hello python")
EOF

echo "hello bash"
```

![&#x8FD0;&#x884C;&#x7ED3;&#x679C;](../../.gitbook/assets/image%20%28478%29.png)

其中：

1.  `/usr/bin/bash <<-EOF XXX EOF` （EXPECT）：将中间XXX的代码传递给python执行，即将代码重定向给python执行。同时，也可以在其中继续重定向其他代码
2. 其中，后面的EOF需要顶头，且EOF只是人们比较习惯的方式
3. 可在EXPECT中传递BASH代码执行，只有BASH中才能识别EOF的结构

![EOF&#x793A;&#x4F8B;](../../.gitbook/assets/image%20%28472%29.png)

示例程序如下：

```text
#!/usr/bin/bash
cd /home
ls
```

![&#x7A0B;&#x5E8F;&#x7684;&#x6267;&#x884C;&#x65B9;&#x5F0F;](../../.gitbook/assets/image%20%28474%29.png)

程序执行结果不同的原因：

1. 上面的方式是子shell页面执行
2. 下面的是在当前的shell中执行
3. 大部分的程序都是在子shell中执行的
4. 若要定义变量的东西在当前shell中生效，可用当前shell执行

示例：重定义`/root/.bashrc`文件

![&#x88AB;&#x4FEE;&#x6539;&#x7684;.bashrc&#x6587;&#x4EF6;](../../.gitbook/assets/image%20%28475%29.png)

```text
## 执行顺序：

vim /root/.bashrc
i
alias yang='ls -l'
:wq
. /root/.bashrc
yang

## 事后记得改回来
```

![&#x6267;&#x884C;&#x7ED3;&#x679C;](../../.gitbook/assets/image%20%28486%29.png)

## 4、shell特性

### 1、shell是什么

命令解释器，能在shell下运行的命令就是shell命令

### 2、Linux支持的shell

![](../../.gitbook/assets/image%20%28479%29.png)

查看对shell有影响的文件：

```text
login shell    #需要登录的shell
nologin shell    #不登录的shell
```

su命令的区别：

```text
su - XXX    #login shell,对应文件如下：

/etc/profile 
/etc/bashrc    
~/.bash_profile
~/.bashrc

su XXX    #nologin shell,对应文件如下：

~/.bash_logout
~/.bash_history

## 与shell有关的文件

#系统级
/etc/profile    #登录时执行
/etc/bashrc    

# 用户级
~/.bash_profile
~/.bashrc
~/.bash_logout    #在离开shell的时候执行
~/.bash_history    
```

### 3、GNU/bash shell 特点

#### 1、命令和文件自动补齐：作用，可提高效率

#### 2、命令历史记忆功能

1、上下键

![!number](../../.gitbook/assets/image%20%28485%29.png)

![!string&#xFF1A;&#x663E;&#x793A;&#x4F7F;&#x7528;&#x8FC7;&#x5E26;l&#x7684;&#x547D;&#x4EE4;](../../.gitbook/assets/image%20%28483%29.png)

![!$&#xFF1A;&#x663E;&#x793A;&#x4E0A;&#x6761;&#x547D;&#x4EE4;&#x7684;&#x6700;&#x540E;&#x53C2;&#x6570;](../../.gitbook/assets/image%20%28480%29.png)

![!!&#xFF1A;&#x4E0A;&#x4E00;&#x6761;&#x547D;&#x4EE4;](../../.gitbook/assets/image%20%28482%29.png)

#### 3、别名功能

```text
alias
unalias cp
~username/.bashrc
\cp -rf /etc/hosts
```

#### 4、快捷键

```text
ctrl+R    #调出之前的命令
ctrl+D    #退出logon
ctrl+A    #将光标移动到命令最左面
ctrl+E    #将光标移动到命令最右面
ctrl+K    #删除光标后面的命令
ctrl+U    #删除光标前面的命令
ctrl+S    #将屏幕锁住
ctrl+Q    #取消锁屏
ctrl+Y    #粘贴消除掉的字符
```

![ctrl+R](../../.gitbook/assets/image%20%28481%29.png)

#### 5、前后台控制作业

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7B26;&#x53F7;</th>
      <th style="text-align:left">&#x7528;&#x6CD5;</th>
      <th style="text-align:left">&#x8BE6;&#x60C5;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&amp;</td>
      <td style="text-align:left">sleep 10 &amp;</td>
      <td style="text-align:left">&#x4F7F;&#x5F97;&#x8FDB;&#x7A0B;&#x5728;&#x540E;&#x53F0;&#x8FD0;&#x884C;&#xFF0C;&#x5373;&#x4F7F;&#x9000;&#x51FA;&#x7EC8;&#x7AEF;&#xFF0C;&#x4F9D;&#x7136;&#x8FD0;&#x884C;</td>
    </tr>
    <tr>
      <td style="text-align:left">nohup</td>
      <td style="text-align:left">nohup sleep 10 &amp;</td>
      <td style="text-align:left">&#x4F7F;&#x5F97;&#x8FDB;&#x7A0B;&#x5728;&#x540E;&#x53F0;&#x8FD0;&#x884C;&#xFF0C;&#x5373;&#x4F7F;&#x9000;&#x51FA;&#x7EC8;&#x7AEF;&#xFF0C;&#x4F9D;&#x7136;&#x8FD0;&#x884C;</td>
    </tr>
    <tr>
      <td style="text-align:left">screen</td>
      <td style="text-align:left">screen -s</td>
      <td style="text-align:left">&#x547D;&#x540D;&#x4F1A;&#x8BDD;</td>
    </tr>
    <tr>
      <td style="text-align:left">Ctrl+C</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">&#x6740;&#x6389;&#x524D;&#x53F0;&#x8FDB;&#x7A0B;</td>
    </tr>
    <tr>
      <td style="text-align:left">Ctrl+Z</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">&#x6740;&#x6389;&#x540E;&#x53F0;&#x8FDB;&#x7A0B;</td>
    </tr>
    <tr>
      <td style="text-align:left">jobs</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">&#x67E5;&#x770B;&#x540E;&#x53F0;&#x8FDB;&#x7A0B;</td>
    </tr>
    <tr>
      <td style="text-align:left">bg</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">&#x8BA9;&#x8FDB;&#x7A0B;&#x5728;&#x540E;&#x53F0;&#x5DE5;&#x4F5C;</td>
    </tr>
    <tr>
      <td style="text-align:left">fg</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">&#x8BA9;&#x8FDB;&#x7A0B;&#x5728;&#x524D;&#x53F0;&#x5DE5;&#x4F5C;&#xFF0C;&#x53EF;&#x4E0E;
        Ctrl+Z &#x4E00;&#x8D77;&#x7528;</td>
    </tr>
    <tr>
      <td style="text-align:left">kill %3
        <br />kill 3</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p>&#xFF08;&#x5F53;&#x524D;&#x4F1A;&#x8BDD;&#x4E2D;&#x4F5C;&#x4E1A;&#x53F7;&#x4E3A;3&#xFF09;</p>
        <p>&#xFF08;&#x7ED9;PID&#x4E3A;3&#x7684;&#x8FDB;&#x7A0B;&#x53D1;&#x4FE1;&#x53F7;&#xFF09;</p>
      </td>
    </tr>
  </tbody>
</table>



#### 6、输入输出重定向

```text
0,1,2 > >> 2> 2>> 2>&1 &>

/*
0：标准输入，描述符的名称，对应的是键盘
1：键盘
2：屏幕
>：输出重定向
>>：输出重定向，与上区别在于两者覆盖度的问题
2>：错误输出
2>>：追加
2>&1：描述符2的内容重定向到描述符1
&>：混合输出
*/

cat < /etc/hosts

/*
将文件/etc/hosts作为参数定向到了cat中

简单的copy命令：
cat </etc/hosts >/etc/hosts1
# 将hosts内容复制到hosts1中

*/

cat <<EOF

/*
将手动输入的内容传给命令去执行
*/

cat >file1 <<EOF

/*
将输入传递到文件中,适合传递多行到文件
*/
```

![cat &amp;lt; /etc/hosts](../../.gitbook/assets/image%20%28487%29.png)

![EOF](../../.gitbook/assets/image%20%28473%29.png)

#### 7、管道

管道的作用：将一个命令的输出当做下一段的输入

```text
ip addr | grep 'inet' | grep eth0

## tee管道：可将输出覆盖到文件中，-a可以增加
ip addr | grep 'inet' |tee test | grep eth0  覆盖
ip addr | grep 'inet' |tee -a test | grep eth0 追加
```

![tee&#x7BA1;&#x9053;&#x793A;&#x4F8B;](../../.gitbook/assets/image%20%28476%29.png)

```text
## '/$':分区
df | grep '/$'
df | tee df.txt | grep '/$'

grub-md5-crypt
grub-md5-crypt >> /etc/grub.conf
grub-md5-crypt |tee -a /etc/grub.conf
```

#### 8、命令排序

;  不具备逻辑

![&#x56DE;&#x5230;&#x5BB6;&#x76EE;&#x5F55;&#x5E76;&#x5F39;&#x51FA;&#x5149;&#x9A71;](../../.gitbook/assets/image%20%28477%29.png)



#### 9、10、11、12、13、14、15、16、17、18、

### 4、5、6、7、8、9、10、

## 5、



## 6、



## 7、



## 8、



## 9、



## 10、



## 11、



## 12、



## 13、



## 14、



## 15、



## 16、



## 17、



## 18、



## 19、



## 20、



## 21、



## 22、



## 23、



## 24、



## 25、



## 26、



## 27、



## 28、



## 29、



## 30、





