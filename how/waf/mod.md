# modsecurity

中文官网：[http://www.modsecurity.cn/](http://www.modsecurity.cn/)

一条示例规则如下：

```
## 凡诺企业网站管理系统PHP版v3.0存在SQL注入漏洞

SecRule REQUEST_FILENAME "@rx (?i:(/admin/cms_channel\.php))" "chain,rev:'1',ver:'MYWAF_CRS/1.0.7',t:none,t:urlDecodeUni,msg:'SQL injection vulnerability',tag:'MYWAF_CRS/WEB_ATTACK/SQLI',logdata:'Matched Data: %{TX.0} found within %{MATCHED_VAR_NAME}: %{MATCHED_VAR}'"
SecRule ARGS:del "@rx (?i:(select|union|and|update|insert|\(|\-\-|#|\/\*\*))" "capture,setvar:'tx.msg=%{rule.msg}',setvar:tx.anomaly_score=+%{tx.critical_anomaly_score}"
```

中间的每个字段都各代表什么意思，请让我慢慢给你拆解

## 一、通用字段

| 字段                 | 含义                                                           |
| ------------------ | ------------------------------------------------------------ |
| SecRule            | 创建一个使用所选运算符分析指定变量的规则                                         |
| @rx                | 通过提供的正则表达式，对指定的变量进行匹配检测。rx是默认运算符，所有未明确指定运算符的规则都将默认使用@rx作为运算符 |
| chain              | 使用紧随其后的规则与当前规则进行链接，形成规则链。链式规则允许更复杂的处理逻辑。                     |
| rev/ver            | 指定规则的修订版本。它与id动作结合使用可以提供规则已更改的指示。/  指定规则集版本。                 |
| msg                | 将自定义信息分配给规则或规则链。 该消息将与每次警报一起记录到日志中。                          |
| tag                | 为规则或链指定标记（类别）。                                               |
| logdata            | 将指定的数据片段记录为警报消息的一部分。                                         |
| MATCHED\_VAR\_NAME | 此变量保存与之匹配的变量的全名。                                             |
| MATCHED\_VAR       | 此变量保存最近匹配的变量的值。它类似于TX:0，但它由所有运算符自动支持，无需指定捕获操作。               |
| captue             | 与正则表达式运算符（@rx）一起使用时，捕获操作将创建正则表达式捕获的副本，并将它们放入事务变量集合中。         |

下图是我理解中的一条较长规则的示意图：

![我所理解的规则结构](<../../.gitbook/assets/image (43).png>)



## 二、其他

值得拎出来另说的有：

### 1、TX

该变量代表临时事务集合，用于存储数据片段，创建事务异常分值等。放入此集合的变量仅在事务完成之前可用。

| 数值       | 含义                             |
| -------- | ------------------------------ |
| 0        | 将@rx或@pm运算符与捕获操作一起使用时的匹配值      |
| 1-9      | 使用@rx运算符捕获parens和捕获动作时捕获的子表达式值 |
| MSC\_.\* | ModSecurity处理标志                |

### 2、setvar

创建，删除或更新变量。变量名称不区分大小写。

| 作用             | 用法                |
| -------------- | ----------------- |
| 创建变量并同时对其进行初始化 | setvar:TX.score=  |
| 删除变量           | setvar:!TX.score  |
| 增加变量值          | setvar:TX.score=+ |
| 减少变量值          | setvar:TX.score=- |

### 3、解码规则（t:）

此操作用于指定转换管道，用于在匹配之前转换规则中使用的每个变量的值。

| 解码                 | 含义                                                                       |
| ------------------ | ------------------------------------------------------------------------ |
| none               | 不是实际的转换函数，而是指向ModSecurity的指令，用于删除与当前规则关联的所有转换函数。                         |
| urlDecodeUni       | 解码URL编码的输入字符串。                                                           |
| htmlEntityDecode   | 将编码为HTML实体的字符进行解码。                                                       |
| lowercase          | 使用当前C语言环境将所有字符转换为小写。                                                     |
| compressWhiteSpace | 将任何空白字符（0x20, \f, \t, \n, \r, \v, 0xa0）转换为空格（ASCII 0x20），将多个连续空格字符压缩为一个。 |
| base64Decode       | 解码Base64编码的字符串                                                           |





## 三、web请求过程

### 1、request

例如发送请求包为：

```
POST /abc/def/index.php?a=1&b=2&c=3 HTTP/1.1

d=index&e=p
```

| 字段                | 含义                                           | 匹配部分               |
| ----------------- | -------------------------------------------- | ------------------ |
| REQUEST\_BASENAME | 该变量仅包含REQUEST\_FILENAME的文件名部分（例如，index.php）。 | index.php          |
| REQUEST\_FILENAME | 此变量包含不带查询字符串部分的相对请求URL（例如，/index.php）。       | /abc/def/index.php |
|                   |                                              |                    |

### 2、response

| 字段 | 含义 | 匹配部分 |
| -- | -- | ---- |
|    |    |      |

## 四、其他

### 如何快速写出一条规则：

写比较简单的规则，就像做填空题一样，在合适的地方将空用合适的语句填起来就可以了。下面的例子中，我会将关键部位指示出来，方便更多的人参考

```
## 凡诺企业网站管理系统PHP版v3.0存在SQL注入漏洞

SecRule 要圈定的REQUEST方法及其范围 "@rx (?i:(对应范围的数值))" "chain,rev:'1',ver:'MYWAF_CRS/1.0.7',t:none,t:urlDecodeUni,msg:'漏洞的英文名',tag:'MYWAF_CRS/WEB_ATTACK/漏洞类型英文简写',logdata:'Matched Data: %{TX.0} found within %{MATCHED_VAR_NAME}: %{MATCHED_VAR}'"
SecRule 要看的参数 "@rx (?i:(参数对应值))" "capture,setvar:'tx.msg=%{rule.msg}',setvar:tx.anomaly_score=+%{tx.critical_anomaly_score}"
```

&#x20;文档结束，后期继续优化

### 相关漏洞拦截

#### XSS

```

"@rx (?i)(?:[\s\S](?:(?:allowscriptaccess|data:text\/html|formaction)[\s\S]|x(?:(?:link:href|mlns)[\s\S]|html[\s\S] )|(?:@import|base64|href)[\s\S] )|<(?:(?:s(?:cript|tyle)|isindex|object|form)[^>]*?>[\s\S]*?|(?:applet|meta)[^>]*?>[\s\S]*? )|(?:=|U\s*?R\s*?L\s*?\()\s*?[^>]*?\s*?S\s*?C\s*?R\s*?I\s*?P\s*?T\s*?:|[\s\v\"'`;\/0-9\=]+on\w+\s*?=)"

DOM：
"@rx ([\^\[\]\!\;\:\@\'\$\"~\`\>\<]{4,}(document\.(title|write|cookie)|window\.|location\.|innerHTML|confirm|javascript|prompt))"
```

#### SQLi

```
"@rx (?i:(\'|\"|,|;).*\b(select|union|and|or|update|insert)\b.*(\(|\-\-|#|\/\*\*))"

宽字节：
"@rx (ß|運|錦|誠|縗)(\'|\"|\\\\)"
"@rx (ß|锦|運|錦|誠|縗)(\'|\"|\\\\)"
"@pm %e5%5c%27 %df%27 %df%5c %e9%8c%a6 %e5%27 %E9%94%A6%27 %b5%27"

注入绕过：
"@rx (?i:(\bcase\b.*?\bwhen\b.*?\bthen\b|[\"'`´’‘#)][\x01-\x20\x25\xa0]*--|\bselect\b.*?\bfrom\b\s*\S|('\x20\|\||\x2F\*\*\x2Fand\x2F\*\*\x2F))|(\bx?or\b|\|\|)[\x01-\x20\x25\xa0\+\@`\(\'\"]+[^<>=\s]+.{0,50}(([<>=]|!=))|(\band\b|\&\&)[\x01-\x20\x25\xa0\+\@`\(\'\"]+[^<>=\s]+.{0,50}(([<>=]|!=))|(\b(insert|replace)\b[\x01-\x20\x25\xa0]+\binto\b\s*\S)|(\bupdate\b.*?\bset\b.*?=)|(union\b.*?\bselect\b)|(\/\*\!\d+(select\b|union\b|insert\b|update\b)\*\/)|(\bwaitfor\b.*?\bdelay\b)|(\band\b.*?\bsleep\b)|(\+char\()|(\b(and|union|or|xor|select|update|insert|delete)\b.*?\()|\b(if|rlike)\b.+?\belse\b|\bif\s*?\(.+?,.+?,.+?\)|(union\b.*?\bselect\b)|(\/\*\!\d+(select\b|union\b|insert\b|update\b)\*\/))"
"@rx (?i:(\bcase\b.*?\bwhen\b.*?\bthen\b|[\"'`´’‘#)][\x01-\x20\x25\xa0]*--|\bselect\b.*?\bfrom\b\s*\S|('\x20\|\||\x2F\*\*\x2Fand\x2F\*\*\x2F))|(\bx?or\b|\|\|)[\x01-\x20\x25\xa0\+\@`\(\'\"]+[^<>=\s]+.{0,50}(([<>=]|!=))|(\band\b|\&\&)[\x01-\x20\x25\xa0\+\@`\(\'\"]+[^<>=\s]+.{0,50}(([<>=]|!=))|(\b(insert|replace)\b[\x01-\x20\x25\xa0]+\binto\b\s*\S)|(\bupdate\b.*?\bset\b.*?=)|(union\b.*?\bselect\b)|(\/\*\!\d+(select\b|union\b|insert\b|update\b)\*\/)|(\bwaitfor\b.*?\bdelay\b)|(\band\b.*?\bsleep\b)|(\+char\()|\b0x[0-9a-fA-Z]{10,10}|(\b(and|union|or|xor|select|update|insert|delete)\b.*?\()|\b(if|rlike)\b.+?\belse\b|\bif\s*?\(.+?,.+?,.+?\)|(union\b.*?\bselect\b)|(\/\*\!\d+(select\b|union\b|insert\b|update\b)\*\/))"

万能密码：
"@rx (?i:(\'|\").*?\b(or|xor|=|and|\|\-\-)\b)"
"(?i:(\'|\").*?\b(or|xor)\b|(\bx?or\b|\|\|)[\x01-\x20\x25\xa0\+\@`\(\'\"]+[^<>=\s]+.*(([<>=]|!=)))|(\band\b|\&\&)[\x01-\x20\x25\xa0\+\@`\(\'\"]+[^<>=\s]+.*(([<>=]|!=))"
```

#### XXE

```
"@rx (?i:<\!(?:entity|doctype)\b.*?(system|public)\b)"

JSON绕过：
"@rx (?i:([\s\"'`;\/\=]+on\w+\s*=(.*?))(fromcharcode|alert|eval|confirm|expression|prompt)(.*?)(\(|\`)|(<script[^>]*>[\s\S]*?<\/script[^>]*>|<script[^>]*>[\s\S]*?<\/script[[\s\S]]*[\s\S]|<script[^>]*>[\s\S]*?<\/script[\s]*[\s]|<script[^>]*>[\s\S]*?<\/script|<script[^>]*>[\s\S]*?)|(\b(fromcharcode|alert|confirm|expression|prompt)\b|[\'|\"|^|\s|\.|;|,|<|>|=|:](eval))(.*?)(\(|`))"
```

#### SSRF

```
"@rx ((file|dict|gopher|phar)\:\/\/.*)|(http:\/\/[0-9]{10})|([0-9]{2,4}\.[0-9]{1,4}\.[0-9]{1,4}\.[0-9]{1,4}\:[0-9]{2,4})"
```

#### 命令执行

```
json绕过
"@rx (?i:\b(?:(?:n(?:et(?:\b\W+?\blocalgroup|\b\W+?\buser|\b\W+?\badd|\.exe)|(?:map|c)\.exe)|t(?:racer(?:oute|t)\W+\w|elnet\.exe)|(?:w(?:guest|sh)|rcmd)\.exe)\b|c(?:md(?:(?:\.exe|32)?\b\W*?\/c)\b|hmod.{0,40}?\+.{0,3}x))|[`;|&\)]\W*?\b(?:(?:c(?:h(?:grp|mod|own|sh))|p(?:asswd|ython|erl|ing|s)|n(?:asm|map|c)|f(?:inger|tp)|kill|(?:xte)?rm|lsof|telnet|uname|echo)\b(?!(=|:))|id\b(?!(=|:))|g(?:\+\+|cc\b)))|(?i:(\b(echo|cat|\/bin\/bash|wget|curl)\b.*?(\||>)))|(?i:[&+|;]\(?\b(ipconfig|systeminfo|shutdown|taskkill|whoami|ifconfig|netstat|reboot|net user|bash|poweroff|shutdown|etc)\b\)?(?!=))|eval\W{1,5}base64_decode|execute\W{1,5}execute|(\@eval\W{1,5}base64_decode|\@eval\W{1,5}str_rot13|\@eval\(chr\()"

普通：
"(?i:\b(?:(?:n(?:et(?:\b\W+?\blocalgroup|\b\W+?\buser|\b\W+?\badd|\.exe)|(?:map|c)\.exe)|t(?:racer(?:oute|t)\W+\w|elnet\.exe)|(?:w(?:guest|sh)|rcmd)\.exe)\b|c(?:md(?:(?:\.exe|32)?\b\W*?\/c)\b|hmod.{0,40}?\+.{0,3}x))|[`;|&\)]\W*?\b(?:(?:c(?:h(?:grp|mod|own|sh))|\bp(?:asswd|ython|erl|ing|s)\b|n(?:asm|map)|f(?:inger|tp)|kill|(?:xte)?rm|lsof|telnet|uname|echo)\b|\bg(?:\+\+|cc\b)))|(?i:(\b(echo|cat|\/bin\/bash|wget|curl)\b.*?(\||>)))|(?i:[&+|;]\(?\b(ipconfig|systeminfo|shutdown|taskkill|whoami|ifconfig|netstat|reboot|net user|bash|poweroff|shutdown|etc)\b\)?)|eval\W{1,5}base64_decode|execute\W{1,5}execute|(\@eval\W{1,5}base64_decode|\@eval\W{1,5}str_rot13|\@eval\(chr\()"

```

#### 路径遍历

```
JSON绕过：
"@rx \.{1,};{0,}[/\\\\]+\.{1,};{0,}[/\\\\]+"

特殊字符绕过：
"(?i:(%c0%2e|%c0%5c|%c0%2f)|(0x2e0x2e0x5c)|(0x2e0x2e)|(\.\.0x5c|\.\.2f|%uff0e%u2216|%uF025|%uEFC8|%u2216|%u2215|%uff0e|%c0%af|%25c0%25af|%25c0%25ae|%c1%9c|%25c1%259c|%%32%65|%c0%af))"
```

#### 搜索引擎爬虫防护

```
"(?i:(\baltavista\b|\bask jeeves\b|\battentio\b|\bchtml generic\b|\bcrawler\b|\bfastmobilecrawl\b|\bfirefly\b|\bfroogle\b|\bgigabot\b|\bheritrix\b|\bhttrack\b|\bia_archiver\b|\birlbot\b|\biescholar\b|\binfoseek\b|\bjumpbot\b|\blinkcheck\b|\blycos\b|\bmediapartners\b|\bmediobot\b|\bmotionbot\b|\bmshots\b|\bopenbot\b|\bpss-webkit-request\b|\bpythumbnail\b|\bscooter\b|\bsnapbot\b|\btaptubot\b|\btechnoratisnoop\b|\bteoma\b|\btwiceler\b|\byahooseeker\b|\byahooysmcm\b|\byammybot\b|\bahrefsbot\b|\bpingdom.com_bot\b|\bkraken\b|\byandexbot\b|\btwitterbot\b|\btweetmemebot\b|\bopenhosebot\b|\bqueryseekerspider\b|\blinkdexbot\b|\bgrokkit-crawler\b|\blivelapbot\b|\bgermcrawler\b|\bdomaintunocrawler\b|\bgrapeshotcrawler\b|\bcloudflare-alwaysonline\b|\bNikto\b|\bParos proxy\b|\bWebScarab\b|\bWebInspect\b|\bWhisker/libwhisker\b|\bBurpsuite\b|\bWatchfire AppScan\b|\bWAScan\b|\bOWASP Zed ZAP\b|\bNessus\b|\bnmap\b|\bdirbuster\b|\bwwwscan\b|\bcansina\b|\bmetis\b|\bbilbo\b|\bn-stealth\b|\bblack widow\b|\bbrutus\b|\bcgichk\b|\bwebtrends security analyzer\b|\bjaascois\b|\bpmafind\b|\b.nasl\b|\bnsauditor\b|\bparos\b|\bwebinspect\b|\bblackwidow\b|\bnikto\b|\bsqlmap\b|\bRsas\b|\bAppScan\b|\bacunetix_wvs_security_test\b|\bnetsparker\b|\bYodaoBot\b|\bTwiceler\b|\bCollapsarWEB qihoobot\b|\bNaverBot\b|\bYanga WorldSearch Bot v\b|\bdiscobot\b|\bia_archiver\b))"
```

