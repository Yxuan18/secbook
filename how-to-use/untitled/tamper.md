# tamper

1、tamper的作用：  
使用SQLMap提供的tamper脚本，可在一定程度上避开应用程序的敏感字符过滤、绕过WAF规则的阻挡，继而进行渗透攻击。

2、WAF是啥？

* WAF：Web应用程序防火墙，Web Application Firewall
* IPS：入侵防御系统， Intrusion Prevention System
* IDS：入侵检测系统，Intrusion Detection System

3、tamper用法

--tamper=TAMPER 利用给定的脚本进行篡改注入数据。其用法可举例说明：

sqlmap.py -u "[http://.../?uname=admin&pwd=pass123](http://.../?uname=admin&pwd=pass123)" --level=5 --risk=3 -p "uname" --tamper=xxx.py

表示对指定的url地址，以所设置的level等级、risk等级，并采用选定的tamper篡改脚本对参数“uname”进行检测

sqlmap中所有的tamper：

![&#x56FE;&#x7247;&#x662F;V1.4.6.9&#x7684;sqlmap&#x91CC;&#x7684;tamper](../../.gitbook/assets/image%20%282%29.png)



