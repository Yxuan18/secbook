# 基线检查

## 1、身份鉴别

### 1、Apache账户安全

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 为运行 httpd 服务设置专门设置用户和组 |
| 检测步骤 | 1、根据需要为 Apache 创建专门的用户和组  2、参考配置操作 修改 httpd.conf 配置文件对应行参数。  User apache  Group apache  其中 apache 是专门为 Apache 服务创建的用户和组。 |
| 符合性依据 | 检查/httpd.conf 配置文件。有专门的 Apache 的用户、组即为符合。该账户 不能是默认的 daemon、操作系统管理员用户 |
| 影响性分析 | 加固操作对业务的影响需要在测试环境中进行测试 |
| 备注 | apache 默认是用 daemon |



## 2、日志配置

### 1、审核登录

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x9879;&#x76EE;</th>
      <th style="text-align:left">&#x8BE6;&#x60C5;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x8BF4;&#x660E;</td>
      <td style="text-align:left">&#x8BBE;&#x5907;&#x5E94;&#x914D;&#x7F6E;&#x65E5;&#x5FD7;&#x529F;&#x80FD;&#xFF0C;&#x5BF9;&#x8FD0;&#x884C;&#x9519;&#x8BEF;&#x3001;&#x7528;&#x6237;&#x8BBF;&#x95EE;&#x7B49;&#x8FDB;&#x884C;&#x8BB0;&#x5F55;&#xFF0C;&#x8BB0;&#x5F55;&#x5185;&#x5BB9;&#x5305;&#x62EC;&#x65F6;
        &#x95F4;&#xFF0C;&#x7528;&#x6237;&#x4F7F;&#x7528;&#x7684; IP &#x5730;&#x5740;&#x7B49;&#x5185;&#x5BB9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />&#x7F16;&#x8F91; httpd.conf &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;&#x8BBE;&#x7F6E;&#x65E5;&#x5FD7;&#x8BB0;&#x5F55;&#x6587;&#x4EF6;&#x3001;&#x8BB0;&#x5F55;&#x5185;&#x5BB9;&#x3001;&#x8BB0;&#x5F55;&#x683C;&#x5F0F;&#x3002;
          LogLevel notice
          <br />ErrorLog logs/error_log
          <br />LogFormat &quot;%h %l %u %t \&quot;%r\&quot; %&gt;s %b \&quot;%{Accept}i\&quot;
          \&quot;%{Referer}i\&quot; \&quot;%{UserAgent}i\&quot;&quot; combined</p>
        <p>CustomLog logs/access_log combined
          <br />ErrorLog &#x6307;&#x4EE4;&#x8BBE;&#x7F6E;&#x9519;&#x8BEF;&#x65E5;&#x5FD7;&#x6587;&#x4EF6;&#x540D;&#x548C;&#x4F4D;&#x7F6E;&#x3002;&#x9519;&#x8BEF;&#x65E5;&#x5FD7;&#x662F;&#x6700;&#x91CD;&#x8981;&#x7684;&#x65E5;&#x5FD7;&#x6587;&#x4EF6;&#xFF0C;
          Apache httpd &#x5C06;&#x5728;&#x8FD9;&#x4E2A;&#x6587;&#x4EF6;&#x4E2D;&#x5B58;&#x653E;&#x8BCA;&#x65AD;&#x4FE1;&#x606F;&#x548C;&#x5904;&#x7406;&#x8BF7;&#x6C42;&#x4E2D;&#x51FA;&#x73B0;&#x7684;&#x9519;&#x8BEF;&#x3002;&#x82E5;&#x8981;&#x5C06;&#x9519;&#x8BEF;&#x65E5;&#x5FD7;&#x9001;&#x5230;
          Syslog&#xFF0C;&#x5219;&#x8BBE;&#x7F6E;&#xFF1A;ErrorLog syslog&#x3002;
          <br
          />CustomLog &#x6307;&#x4EE4;&#x8BBE;&#x7F6E;&#x8BBF;&#x95EE;&#x65E5;&#x5FD7;&#x7684;&#x6587;&#x4EF6;&#x540D;&#x548C;&#x4F4D;&#x7F6E;&#x3002;&#x8BBF;&#x95EE;&#x65E5;&#x5FD7;&#x4E2D;&#x4F1A;&#x8BB0;&#x5F55;&#x670D;&#x52A1;&#x5668;&#x6240;
          &#x5904;&#x7406;&#x7684;&#x6240;&#x6709;&#x8BF7;&#x6C42;&#x3002;
          <br />LogFormat &#x8BBE;&#x7F6E;&#x65E5;&#x5FD7;&#x683C;&#x5F0F;&#x3002;LogLevel
          &#x7528;&#x4E8E;&#x8C03;&#x6574;&#x8BB0;&#x5F55;&#x5728;&#x9519;&#x8BEF;&#x65E5;&#x5FD7;&#x4E2D;&#x7684;&#x4FE1;&#x606F;&#x7684;&#x8BE6;
          &#x7EC6;&#x7A0B;&#x5EA6;&#xFF0C;&#x5EFA;&#x8BAE;&#x8BBE;&#x7F6E;&#x4E3A;
          notice</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">1&#x3001;&#x67E5;&#x770B; logs &#x76EE;&#x5F55;&#x4E2D;&#x76F8;&#x5173;&#x65E5;&#x5FD7;&#x6587;&#x4EF6;&#x5185;&#x5BB9;&#xFF0C;&#x8BB0;&#x5F55;&#x5B8C;&#x6574;&#x3002;
        <br
        />2&#x3001;&#x67E5;&#x770B;&#x76F8;&#x5173;&#x65E5;&#x5FD7;&#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x3002;</td>
    </tr>
  </tbody>
</table>



## 3、访问权限

### 1、禁止访问外部文件

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 禁止 Apache 访问 Web 目录之外的任何文件。 |
| 检测步骤 | 1、参考配置操作  编辑 httpd.conf 配置文件，   &lt;Directory&gt; Options None  AllowOverride None  Order deny,allow  Deny from all   &lt;Directory&gt; 2、补充操作说明  设置可访问目录，  Order Allow,Deny  Allow from all |
| 符合性依据 | 1、访问服务器上不属于 Web 目录的一个文件，结果应无法显示（无法访问 Web 目录之外的文件）。  2、检查相应配置文件 |



## 4、防攻击管理

### 1、错误页面处理

| 项目 | 详情 |
| :--- | :--- |
| 说明 | Apache 错误页面重定向 |
| 检测步骤 | 1、参考配置操作  \(1\) 修改 httpd.conf 配置文件：  ErrorDocument 400 /custom400.html  ErrorDocument 401 /custom401.html  ErrorDocument 403 /custom403.html  ErrorDocument 404 /custom404.html  ErrorDocument 405 /custom405.html  ErrorDocument 500 /custom500.html  Customxxx.html 为要设置的错误页面。  \(2\)重新启动 Apache 服务 |
| 符合性依据 | 在 URL 地址栏中输入 http://ip/xxxxxxx~~~（一个不存在的页面）  检测是否指向指定错误页面 |



### 2、目录列表访问限制

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x9879;&#x76EE;</th>
      <th style="text-align:left">&#x8BE6;&#x60C5;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x8BF4;&#x660E;</td>
      <td style="text-align:left">&#x7981;&#x6B62; Apache &#x5217;&#x8868;&#x663E;&#x793A;&#x6587;&#x4EF6;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />(1) &#x4FEE;&#x6539; httpd.conf &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;
          <br
          />Options FollowSymLinks
          <br />AllowOverride None
          <br />Order allow,deny
          <br />Allow from all
          <br />&#x5C06; Options Indexes FollowSymLinks &#x4E2D;&#x7684; Indexes &#x53BB;&#x6389;&#xFF0C;&#x5C31;&#x53EF;&#x4EE5;&#x7981;&#x6B62;
          Apache &#x663E;&#x793A;&#x8BE5;&#x76EE;&#x5F55;&#x7ED3;&#x6784;&#x3002;Indexes
          &#x7684;&#x4F5C;&#x7528;&#x5C31;&#x662F;&#x5F53;&#x8BE5;&#x76EE;&#x5F55;&#x4E0B;&#x6CA1;&#x6709;
          index.html &#x6587;&#x4EF6;&#x65F6;&#xFF0C; &#x5C31;&#x663E;&#x793A;&#x76EE;&#x5F55;&#x7ED3;&#x6784;&#x3002;
          <br
          />(2)&#x8BBE;&#x7F6E; Apache &#x7684;&#x9ED8;&#x8BA4;&#x9875;&#x9762;&#xFF0C;&#x7F16;&#x8F91;%apache%\conf\httpd.conf
          &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;</p>
        <p>&lt;IfModule dir_module&gt;</p>
        <p>DirectoryIndex index.html</p>
        <p>&lt;/IfModule&gt;</p>
        <p>&#x5176;&#x4E2D; index.html &#x5373;&#x4E3A;&#x9ED8;&#x8BA4;&#x9875;&#x9762;&#xFF0C;&#x53EF;&#x6839;&#x636E;&#x60C5;&#x51B5;&#x6539;&#x4E3A;&#x5176;&#x5B83;&#x6587;&#x4EF6;&#x3002;
          <br
          />(3)&#x91CD;&#x65B0;&#x542F;&#x52A8; Apache &#x670D;&#x52A1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">1&#x3001;&#x76F4;&#x63A5;&#x8BBF;&#x95EE; http://ip:8800/xxx&#xFF08;xxx
        &#x4E3A;&#x67D0;&#x4E00;&#x76EE;&#x5F55;&#xFF09; &#xFF0C;&#x5F53; WEB &#x76EE;&#x5F55;&#x4E2D;&#x6CA1;&#x6709;&#x9ED8;&#x8BA4;
        &#x9996;&#x9875;&#x5982; index.html &#x6587;&#x4EF6;&#x65F6;&#xFF0C;&#x4E0D;&#x4F1A;&#x5217;&#x51FA;&#x76EE;&#x5F55;&#x5185;&#x5BB9;&#x3002;
        <br
        />2&#x3001;&#x76F4;&#x63A5;&#x67E5;&#x770B; httpd.conf &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x53C2;&#x6570;&#xFF1A;
        <br />Options +Indexes +FollowSymLinks +ExecCGI
        <br />AllowOverride All
        <br />Order allow,deny
        <br />Allow from all
        <br />Require all granted</td>
    </tr>
  </tbody>
</table>



### 3、拒绝服务防范

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 拒绝服务防范。 |
| 检测步骤 | 1、参考配置操作  \(1\) 编辑 httpd.conf 配置文件，  Timeout 10 KeepAlive On  KeepAliveTimeout 15  MaxKeepAliveRequests 100  AcceptFilter http data  AcceptFilter https data  \(2\)重新启动 Apache 服务 |
| 符合性依据 | 检查配置文件是否设置。 |



### 4、删除无用文件

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 删除缺省安装的无用文件 |
| 检测步骤 | 1、参考配置操作  删除缺省 HTML 文件：  \# rm -rf /usr/local/apache2/htdocs/  _删除缺省的 CGI 脚本：  \# rm –rf /usr/local/apache2/cgi-bin/_  删除 Apache 说明文件：  \# rm –rf /usr/local/apache2/manual  删除源代码文件：  \# rm -rf /path/to/httpd-2.2.4\*  根据安装步骤不同和版本不同，某些目录或文件可能不存在或位置不同 |
| 符合性依据 | 检查对应目录。 |



### 5、隐藏敏感信息

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 隐藏 Apache 的版本号及其它敏感信息。 |
| 检测步骤 | 1、参考配置操作  修改 httpd.conf 配置文件：  ServerSignature Off  ServerTokens Prod |
| 符合性依据 | 1、判定条件  2、检测操作  检查配置文件 |



### 6、请求信息长度

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x9879;&#x76EE;</th>
      <th style="text-align:left">&#x8BE6;&#x60C5;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x8BF4;&#x660E;</td>
      <td style="text-align:left">&#x9650;&#x5236; http &#x8BF7;&#x6C42;&#x7684;&#x4FE1;&#x606F;&#x4E3B;&#x4F53;&#x6700;&#x5927;&#x5B57;&#x8282;&#x957F;&#x5EA6;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />&#x4FEE;&#x6539; httpd.conf &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x5BF9;&#x5E94;&#x884C;&#x53C2;&#x6570;</p>
        <p>&#x4FEE;&#x6539; LimitRequestBody &#x53C2;&#x6570;&#x4E3A; 10240000Byte&#xFF0C;&#x5982;&#x6CA1;&#x6709;
          LimitRequestBody &#x53C2; &#x6570;&#xFF0C;&#x52A0;&#x4E0A;&#x6B64;&#x884C;&#x3002;
          <br
          />LimitRequestBody 10240000</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x68C0;&#x67E5;&#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x5BF9;&#x8BF7;&#x6C42;&#x5185;&#x5BB9;&#x5927;&#x5C0F;&#x8FDB;&#x884C;&#x8BBE;&#x7F6E;&#x3002;</p>
        <p>2&#x3001;&#x4E0A;&#x4F20;&#x6587;&#x4EF6;&#x8D85;&#x8FC7; 10M &#x5C06;&#x62A5;&#x9519;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">&#x5177;&#x4F53;&#x6839;&#x636E;&#x4E1A;&#x52A1;&#x7CFB;&#x7EDF; http
        &#x8BF7;&#x6C42;&#x7684;&#x6700;&#x5927;&#x503C;&#x786E;&#x5B9A;&#x3002;</td>
    </tr>
  </tbody>
</table>



### 7、关闭TRACE

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 防止攻击者利用 trace 对服务器进行信息获取、跨站脚本攻击。 |
| 检测步骤 | 1、参考配置操作  修改配置文件 httpd.conf 对应行参数 TraceEnable 并设置为 Off |
| 符合性依据 | 1、使用 telnet 发送以下命令  输入“telnet x.x.x.x 80” 点击回车 。  输入”CTRL+\]” 点击回车，在点击回车，再点击回车。  输入“TRACE / HTTP/1.0”在点击回车  输入“X-Test:abcde ”在回车两次  观察 HTTP 响应码，如为 405 则说明 trace 请求已关闭，如返回 200 则没有 关闭。  2、直接查看 httpd.conf 配置文件。 |
| 影响性分析 | 加固操作对业务的影响需要在测试环境中进行测试。 |



## 5、补丁管理

### 1、补丁管理

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 补丁或者版本升级应联系业务系统开发商咨询和测试，确认测试无问题后，进行更新 |
| 影响性分析 | 补丁更新加固操作对业务有不可预知的影响，一定需要在测试环境中进行测 试 |





