# modsecurity

中文官网：[http://www.modsecurity.cn/](http://www.modsecurity.cn/)

一条示例规则如下：

```text
## 凡诺企业网站管理系统PHP版v3.0存在SQL注入漏洞

SecRule REQUEST_FILENAME "@rx (?i:(/admin/cms_channel\.php))" "chain,rev:'1',ver:'MYWAF_CRS/1.0.7',t:none,t:urlDecodeUni,msg:'SQL injection vulnerability',tag:'MYWAF_CRS/WEB_ATTACK/SQLI',logdata:'Matched Data: %{TX.0} found within %{MATCHED_VAR_NAME}: %{MATCHED_VAR}'"
SecRule ARGS:del "@rx (?i:(select|union|and|update|insert|\(|\-\-|#|\/\*\*))" "capture,setvar:'tx.msg=%{rule.msg}',setvar:tx.anomaly_score=+%{tx.critical_anomaly_score}"
```

中间的每个字段都各代表什么意思，请让我慢慢给你拆解

## 一、通用字段

| 字段 | 含义 |
| :--- | :--- |
| SecRule | 创建一个使用所选运算符分析指定变量的规则 |
| @rx | 通过提供的正则表达式，对指定的变量进行匹配检测。rx是默认运算符，所有未明确指定运算符的规则都将默认使用@rx作为运算符 |
| chain | 使用紧随其后的规则与当前规则进行链接，形成规则链。链式规则允许更复杂的处理逻辑。 |
| rev/ver | 指定规则的修订版本。它与id动作结合使用可以提供规则已更改的指示。/  指定规则集版本。 |
| msg | 将自定义信息分配给规则或规则链。 该消息将与每次警报一起记录到日志中。 |
| tag | 为规则或链指定标记（类别）。 |
| logdata | 将指定的数据片段记录为警报消息的一部分。 |
| MATCHED\_VAR\_NAME | 此变量保存与之匹配的变量的全名。 |
| MATCHED\_VAR | 此变量保存最近匹配的变量的值。它类似于TX:0，但它由所有运算符自动支持，无需指定捕获操作。 |
| captue | 与正则表达式运算符（@rx）一起使用时，捕获操作将创建正则表达式捕获的副本，并将它们放入事务变量集合中。 |

下图是我理解中的一条较长规则的示意图：

![&#x6211;&#x6240;&#x7406;&#x89E3;&#x7684;&#x89C4;&#x5219;&#x7ED3;&#x6784;](../../.gitbook/assets/image%20%28512%29.png)



## 二、其他

值得拎出来另说的有：

### 1、TX

该变量代表临时事务集合，用于存储数据片段，创建事务异常分值等。放入此集合的变量仅在事务完成之前可用。

| 数值 | 含义 |
| :--- | :--- |
| 0 | 将@rx或@pm运算符与捕获操作一起使用时的匹配值 |
| 1-9 | 使用@rx运算符捕获parens和捕获动作时捕获的子表达式值 |
| MSC\_.\* | ModSecurity处理标志 |

### 2、setvar

创建，删除或更新变量。变量名称不区分大小写。

| 作用 | 用法 |
| :--- | :--- |
| 创建变量并同时对其进行初始化 | setvar:TX.score= |
| 删除变量 | setvar:!TX.score |
| 增加变量值 | setvar:TX.score=+ |
| 减少变量值 | setvar:TX.score=- |

### 3、解码规则（t:）

此操作用于指定转换管道，用于在匹配之前转换规则中使用的每个变量的值。

| 解码 | 含义 |
| :--- | :--- |
| none | 不是实际的转换函数，而是指向ModSecurity的指令，用于删除与当前规则关联的所有转换函数。 |
| urlDecodeUni | 解码URL编码的输入字符串。 |
| htmlEntityDecode | 将编码为HTML实体的字符进行解码。 |
| lowercase | 使用当前C语言环境将所有字符转换为小写。 |
| compressWhiteSpace | 将任何空白字符（0x20, \f, \t, \n, \r, \v, 0xa0）转换为空格（ASCII 0x20），将多个连续空格字符压缩为一个。 |
| base64Decode | 解码Base64编码的字符串 |





## 三、web请求过程

### 1、request

例如发送请求包为：

```text
POST /abc/def/index.php?a=1&b=2&c=3 HTTP/1.1

d=index&e=p
```

| 字段 | 含义 | 匹配部分 |
| :--- | :--- | :--- |
| REQUEST\_BASENAME | 该变量仅包含REQUEST\_FILENAME的文件名部分（例如，index.php）。 | index.php |
| REQUEST\_FILENAME | 此变量包含不带查询字符串部分的相对请求URL（例如，/index.php）。 | /abc/def/index.php |
|  |  |  |

### 2、response

| 字段 | 含义 | 匹配部分 |
| :--- | :--- | :--- |
| REQUEST\_BASENAME | 该变量仅包含REQUEST\_FILENAME的文件名部分（例如，index.php）。 | index.php |

## 四、其他

