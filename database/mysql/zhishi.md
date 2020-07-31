# 知识

## 1、MySQL基础

### 1、特性

1. 使用C和C++编写，并使用了多种编译器进行测试，保证源代码的可移植性。
2. 支持AIX、FreeBSD、HP-UX、Linux、Mac OS、NovellNetware、OpenBSD、OS/2 Wrap、Solaris、Windows等多种操作系统。
3. 为多种编程语言提供了API。这些编程语言包括C、C++、Python、Java、Perl、PHP、Eiffel、Ruby,.NET和Tcl等。
4. 支持多线程，充分利用CPU资源。
5. 优化的SQL查询算法，可有效地提高查询速度。
6. 既能够作为一个单独的应用程序应用在客户端服务器网络环境中，也能够作为一个库嵌入到其他的软件中。
7. 提供多语言支持，常见的编码如中文的GB 2312、BIG5，日文的Shift\_JIS等都可以用作数据表名和数据列名。
8. 提供TCP/IP、ODBC和JDBC等多种数据库连接途径。
9. 提供用于管理、检查、优化数据库操作的管理工具。
10. 支持大型的数据库，可以处理拥有上千万条记录的大型数据库。
11. 支持多种存储引擎。
12. MySQL是开源的，所以用户不需要支付额外的费用。
13. MySQL使用标准的SQL数据语言形式。
14. MySQL对PHP有很好的支持，PHP是目前最流行的Web开发语言之一。
15. MySQL是可以定制的，采用了GPL协议，用户可以修改源码来开发自己的MySQL系统。
16. 在线DDL/更改功能，数据架构支持动态应用程序和开发人员灵活性。
17. 复制全局事务标识，可支持自我修复式集群。
18. 复制无崩溃从机，可提高可用性。
19. 复制多线程从机，可提高性能。

### 2、安装MySQL

开源软件特点：

1. 降低风险
2. 质量更有保障
3. 透明
4. 剪裁
5. 有利的版权和许可价格

安装过程：

1、Windows端，本人使用了phpstudy直接安装  
2、Linux端，可通过使用源码包的方式下载，下载好之后进入命令行设置密码并重启服务即可

```text
SET PASSWORD FOR'root'@'localhost'=PASSWORD('123456'); #在MySQL命令行中设置root账户的密码为123456
```

### 3、可视化管理工具

工具种类及详情如下：

1. MySQL Workbench： MySQL Workbench是一个由MySQL开发的跨平台、可视化数据库工具。它作为DBDesigner4工程的替代应用程序而备受瞩目。MySQL Workbench可以作为Windows、Linux和OS X系统上的原始GUI工具，它有各种不同的版本。
2. Aqua Data Studio： 对于数据库管理人员、软件开发人员以及业务分析师来说，Aqua Data Studio是一个完整的集成开发环境（Intergrated Development Environment，IDE）。它主要具备了以下4个方面的功能：         ⑴数据库查询和管理工具；         ⑵一套数据库、源代码管理以及文件系统的比较工具；         ⑶为Subversion（SVN）和CVS设计了一个完整的集成源代码管理客户端；         ⑷提供了一个数据库建模工具（Modeler），它和最好的独立数据库图表工具一样强大。
3. SQLyogSQLyog： 是一个全面的MySQL数据库管理工具。它的社区版（Community Edition）是具有GPL许可的免费开源软件。这款工具包含了开发人员在使用MySQL时所需的绝大部分功能：查询结果集合、查询分析器、服务器消息、表格数据、表格信息，以及查询历史，它们都以标签的形式显示在界面上，开发人员只要单击鼠标即可。此外，它还可以方便地创建视图和存储过程。
4. MySQL-Front： 这个MySQL数据库的图形GUI是一个“真正的”应用程序，它提供的用户界面比用PHP和HTML建立起来的系统更加精确。因为不会因为重载HTML网页而导致延时，所以它的响应是即时的。如果供应商允许的话，可以让MySQL-Front直接与数据库进行工作。如果不行，也只需要在发布网站上安装一个小的脚本即可。
5. Sequel ProSequel Pro： 是一款管理Mac OS X数据库的应用程序，它可以让用户直接访问本地以及远程服务器上的MySQL数据库，并且支持从流行的文件格式中导入和导出数据，其中包括SQL、CSV和XML等文件。

以Navicat为例，下面是该软件的安装过程：

官方地址：http://www.navicat.com.cn/download/navicat-for-mysql

#### 1、Linux

1、下载文件到根目录后

```text
sudo su    #切换到root账户
tar -zxvf navicat_for_mysql_10.0.11_cn_linux.tar.gz    #解压文文件到/opt/目录下
cd /opt/navicat    #进入软件的安装目录
./start_navicat    #启动Navicat
```

#### 2、Windows

1、双击安装程序，然后点击下一步

![](../../.gitbook/assets/mysql01001.jpg)

2、我同意→下一步

![](../../.gitbook/assets/mysql01002.jpg)

3、选择文件夹，并点击下一步

![](../../.gitbook/assets/mysql01003.jpg)

4、下一步→下一步→下一步→安装→完成，即可

5、双击打开破解程序，并选中安装好的navicat文件

![](../../.gitbook/assets/mysql01004.jpg)

6、等到出现这个界面后，破解成功

![](../../.gitbook/assets/mysql01005.jpg)

7、打开Navicat，并点击连接，可打开连接界面

![](../../.gitbook/assets/mysql01006.jpg)



