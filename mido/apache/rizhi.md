# 日志审计

### &#xD;1、Apache访问日志格式详解  &#x20;

访问日志文件access\_log记录了所有对Web服务器的访问活动，下面是访问日志access\_log中的一条标准记录

192.168.0.1 - - \[01/Apr/2020:10:37:19 +0800] "GET / HTTP/1.1" 200 45

日志字段所代表的内容如下：

1. 远程主机IP：表明访问网站的是谁&#x20;
2. 空白(E-mail)：为了避免用户的邮箱被垃圾邮件骚扰，第二项就用“-”取代了
3. 空白(登录名)：用于记录浏览者进行身份验证时提供的名字。
4. 请求时间：用方括号包围，而且采用“公用日志格式”或者“标准英文格式”。 时间信息最后的“+0800”表示服务器所处时区位于UTC之后的8小时。
5. 方法+资源+协议：服务器收到的是一个什么样的请求。该项信息的典型格式是“METHOD RESOURCE PROTOCOL”，即“方法 资源 协议”。  METHOD: GET、POST、HEAD、…… RESOURCE: /、index.html、/default/index.php、……（请求的文件）  PROTOCOL: HTTP+版本号
6. 状态代码：请求是否成功，或者遇到了什么样的错误。大多数时候，这项值是200，它表示服务器已经成功地响应浏览器的请求，一切正常。
7. 发送字节数：表示发送给客户端的总字节数。它告诉我们传输是否被打断（该数值是否和文件的大小相同）。把日志记录中的这些值加起来就可以得知服务器在一天、一周或者一月内发送了多少数据。

### 2、apapche日志分析技巧——Linux+shell

| 名称                           | 对应命令                                                                                             |
| ---------------------------- | ------------------------------------------------------------------------------------------------ |
| 当天访问次数最多的IP命令                | cut -d- -f 1 log\_file\|uniq -c \| sort -rn \| head -20                                          |
| 查看当天有多少个IP访问                 | awk '{print $1}' log\_file\|sort\|uniq\|wc -l                                                    |
| 查看某一个页面被访问的次数                | grep "/index.php" log\_file \| wc -l                                                             |
| 查看每一个IP访问了多少个页面              | awk '{++S\[$1]} END {for (a in S) print a,S\[a]}' log\_file                                      |
| 将每个IP访问的页面数进行从小到大排序          | awk '{++S\[$1]} END {for (a in S) print S\[a],a}' log\_file \| sort -n                           |
| 查看IP访问了哪些页面                  | grep ^192.168.1.1 log\_file\| awk '{print $1,$7}'                                                |
| 去掉搜索引擎统计当天的页面                | awk '{print $12,$1}' log\_file \| grep ^\\"Mozilla \| awk '{print $2}' \|sort \| uniq \| wc -l   |
| 查看2018年6月21日14时这一个小时内有多少IP访问 | awk '{print $4,$1}' log\_file \| grep 21/Jun/2018:14 \| awk '{print $2}'\| sort \| uniq \| wc -l |

![当天访问次数最多的IP](<../../.gitbook/assets/image (135).png>)

![查看当天有多少个IP访问](<../../.gitbook/assets/image (132).png>)

![查看某一个页面被访问的次数](<../../.gitbook/assets/image (109).png>)

![查看每一个IP访问了多少个页面](<../../.gitbook/assets/image (94).png>)

![将每个IP访问的页面数进行从小到大排序](<../../.gitbook/assets/image (93).png>)

![查看 IP访问了哪些页面](<../../.gitbook/assets/image (127).png>)
