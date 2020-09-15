# 基线检查

## 1、身份鉴别

### 1、共享账户，删除无关账户

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
      <td style="text-align:left">&#x5E94;&#x6309;&#x7167;&#x7528;&#x6237;&#x5206;&#x914D;&#x5E10;&#x53F7;&#x3002;&#x907F;&#x514D;&#x4E0D;&#x540C;&#x7528;&#x6237;&#x95F4;&#x5171;&#x4EAB;&#x5E10;&#x53F7;&#xFF0C;&#x53D1;&#x751F;&#x8D8A;&#x6743;&#x3002;
        &#x5E94;&#x5220;&#x9664;&#x6216;&#x9501;&#x5B9A;&#x4E0E;&#x8BBE;&#x5907;&#x8FD0;&#x884C;&#x3001;&#x7EF4;&#x62A4;&#x7B49;&#x5DE5;&#x4F5C;&#x65E0;&#x5173;&#x7684;&#x5E10;&#x53F7;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />&#x7F16;&#x8F91;/conf/tomcat-users.xml &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;&#x4FEE;&#x6539;&#x6216;&#x5220;&#x9664;&#x5E10;&#x53F7;&#x3002;</p>
        <p>&lt;user username=&#x201D;tomcat&#x201D; password=&#x201D; Tomcat!234&#x201D;
          roles=&#x201D;admin&quot;&gt;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">1&#x3001;&#x5404;&#x5E10;&#x53F7;&#x5747;&#x53EF;&#x4EE5;&#x6B63;&#x5E38;&#x767B;&#x5F55;
        Tomcat Web &#x670D;&#x52A1;&#x5668;&#x3002;
        <br />2&#x3001;&#x88AB;&#x5220;&#x9664;&#x7684;&#x65E0;&#x5173;&#x7684;&#x5E10;&#x53F7;&#x65E0;&#x6CD5;&#x6B63;&#x5E38;&#x767B;&#x9646;&#x3002;</td>
    </tr>
  </tbody>
</table>



### 2、用户权限管理

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
      <td style="text-align:left">&#x6839;&#x636E;&#x7528;&#x6237;&#x7684;&#x4E1A;&#x52A1;&#x9700;&#x8981;&#xFF0C;&#x5BF9;&#x4E2A;&#x8D26;&#x6237;&#x914D;&#x7F6E;&#x5176;&#x6240;&#x9700;&#x7684;&#x6700;&#x5C0F;&#x6743;&#x9650;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />&#x7F16;&#x8F91;/conf/tomcat-users.xml &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;&#x4FEE;&#x6539;&#x7528;&#x6237;&#x89D2;&#x8272;&#x6743;&#x9650;
          <br
          />&#x5BF9; tomcat &#x8D26;&#x53F7;&#x6388;&#x6743;&#x8FDC;&#x7A0B;&#x7BA1;&#x7406;&#x6743;&#x9650;&#xFF1A;</p>
        <p>&lt;user username=&#x201D;tomcat&#x201D; password=&#x201D; Tomcat!23&#x201D;roles=&#x201D;admin,manager&quot;&gt;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x4F7F;&#x7528; tomcat &#x5E10;&#x53F7;&#x767B;&#x9646;&#x8FDC;&#x7A0B;&#x7BA1;&#x7406;&#x9875;&#x9762;&#xFF0C;&#x4E14;&#x767B;&#x9646;&#x6210;&#x529F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5F71;&#x54CD;&#x6027;&#x5206;&#x6790;</td>
      <td style="text-align:left">&#x52A0;&#x56FA;&#x64CD;&#x4F5C;&#x5BF9;&#x4E1A;&#x52A1;&#x7CFB;&#x7EDF;&#x7684;&#x5F71;&#x54CD;&#x9700;&#x8981;&#x5728;&#x6D4B;&#x8BD5;&#x73AF;&#x5883;&#x4E2D;&#x8FDB;&#x884C;&#x6D4B;&#x8BD5;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">Tomcat &#x7528;&#x6237;&#x89D2;&#x8272;&#x5206;&#x4E3A;&#xFF1A;role1&#xFF0C;tomcat&#xFF0C;admin&#xFF0C;manager
        &#x56DB;&#x79CD;&#x3002;
        <br />role1&#xFF1A;&#x5177;&#x6709;&#x8BFB;&#x6743;&#x9650;&#xFF1B;
        <br />tomcat&#xFF1A;&#x5177;&#x6709;&#x8BFB;&#x548C;&#x8FD0;&#x884C;&#x6743;&#x9650;&#xFF1B;
        <br
        />admin&#xFF1A;&#x5177;&#x6709;&#x8BFB;&#x3001;&#x8FD0;&#x884C;&#x548C;&#x5199;&#x6743;&#x9650;&#xFF1B;
        <br
        />manager&#xFF1A;&#x5177;&#x6709;&#x8FDC;&#x7A0B;&#x7BA1;&#x7406;&#x6743;&#x9650;&#x3002;
        <br
        />Tomcat 6.0.18 &#x7248;&#x672C;&#x53EA;&#x6709; admin &#x548C; manager
        &#x4E24;&#x79CD;&#x7528;&#x6237;&#x89D2;&#x8272;&#xFF0C;&#x4E14; admin
        &#x7528;&#x6237;&#x5177; &#x6709; manager &#x7BA1;&#x7406;&#x6743;&#x9650;&#x3002;</td>
    </tr>
  </tbody>
</table>



### 3、口令复杂度

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
      <td style="text-align:left">&#x5BF9;&#x4E8E;&#x91C7;&#x7528;&#x9759;&#x6001;&#x53E3;&#x4EE4;&#x8BA4;&#x8BC1;&#x6280;&#x672F;&#x7684;&#x8BBE;&#x5907;&#xFF0C;&#x53E3;&#x4EE4;&#x957F;&#x5EA6;&#x81F3;&#x5C11;&#x4E3A;
        8 &#x4F4D;&#xFF0C;&#x5E76;&#x5305;&#x542B;&#x6570;&#x5B57;&#x3001;&#x5C0F;
        &#x5199;&#x5B57;&#x6BCD;&#x3001;&#x5927;&#x5199;&#x5B57;&#x6BCD;&#x548C;&#x7279;&#x6B8A;&#x7B26;&#x53F7;&#x56DB;&#x7C7B;&#x4E2D;&#x81F3;&#x5C11;&#x4E09;&#x7C7B;&#x3002;&#x4E14;
        5 &#x6B21;&#x4EE5;&#x5185;&#x4E0D;&#x5F97;&#x8BBE;&#x7F6E;&#x76F8;&#x540C;&#x7684;
        &#x53E3;&#x4EE4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />&#x7F16;&#x8F91;/conf/tomcat-users.xml &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;&#x5BF9;&#x53E3;&#x4EE4;&#x590D;&#x6742;&#x5EA6;&#x8FDB;&#x884C;&#x68C0;&#x67E5;&#x3002;</p>
        <p>&lt;user username=&#x201D;tomcat&#x201D; password=&#x201D; Tomcat!234&#x201D;
          roles=&#x201D;admin&quot;&gt;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x4EBA;&#x5DE5;&#x68C0;&#x67E5;&#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x4E2D;&#x5E10;&#x53F7;&#x53E3;&#x4EE4;&#x662F;&#x5426;&#x7B26;&#x5408;&#x53E3;&#x4EE4;&#x590D;&#x6742;&#x5EA6;&#x8981;&#x6C42;&#x3002;</td>
    </tr>
  </tbody>
</table>



### 4、程序运行身份检查

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 确认是否以 tomcat 身份运行、启动 tomcat 服务 |
| 检测步骤 | 1、参考配置操作  LINUX: ps -aux \| grep tomcat Windows: 通过“任务管理器”查看当前运行 tomcat 的用户身份。 |
| 符合性依据 | 若是管理员身份运行则不符合安全基线 |
| 影响性分析 | 加固操作对业务系统的影响需要在测试环境中进行测试。 |



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
      <td style="text-align:left">&#x8BBE;&#x5907;&#x5E94;&#x914D;&#x7F6E;&#x65E5;&#x5FD7;&#x529F;&#x80FD;&#xFF0C;&#x5BF9;&#x7528;&#x6237;&#x767B;&#x5F55;&#x8FDB;&#x884C;&#x8BB0;&#x5F55;&#x3002;&#x8BB0;&#x5F55;&#x5185;&#x5BB9;&#x5305;&#x62EC;&#x7528;&#x6237;&#x767B;&#x5F55;&#x5E10;&#x53F7;&#xFF0C;
        &#x767B;&#x5F55;&#x662F;&#x5426;&#x6210;&#x529F;&#xFF0C;&#x767B;&#x5F55;&#x65F6;&#x95F4;&#xFF0C;&#x4EE5;&#x53CA;&#x8FDC;&#x7A0B;&#x767B;&#x5F55;&#x65F6;&#x7528;&#x6237;&#x4F7F;&#x7528;&#x7684;
        IP &#x5730;&#x5740;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />&#x7F16;&#x8F91;/conf/server.xml &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;&#x786E;&#x8BA4;&#x4EE5;&#x4E0B;&#x5185;&#x5BB9;&#x542F;&#x7528;&#x672A;&#x88AB;&#x6CE8;&#x91CA;&#x6389;&#xFF1A;</p>
        <p>&lt;Valve className=&quot;org.apache.catalina.valves.AccessLogValve&quot;
          directory=&quot;logs&quot; prefix=&quot;localhost_access_log.&quot; suffix=&quot;.txt&quot;
          pattern=&quot;%h %a %l %u %t &quot;%r&quot; %s %b&quot;/&gt;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">1&#x3001;&#x67E5;&#x770B; logs &#x76EE;&#x5F55;&#x4E2D;&#x76F8;&#x5173;&#x65E5;&#x5FD7;&#x6587;&#x4EF6;&#x5185;&#x5BB9;&#xFF0C;&#x8BB0;&#x5F55;&#x5B8C;&#x6574;&#x3002;
        <br
        />2&#x3001;&#x67E5;&#x770B;&#x76F8;&#x5173;&#x65E5;&#x5FD7;&#x8BB0;&#x5F55;&#x6587;&#x4EF6;&#xFF0C;&#x68C0;&#x67E5;&#x6709;&#x65E0;&#x8BB0;&#x5F55;&#x4E8B;&#x4EF6;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">Directory:&#x65E5;&#x5FD7;&#x6587;&#x4EF6;&#x653E;&#x7F6E;&#x7684;&#x76EE;&#x5F55;&#xFF0C;&#x5728;
        tomcat &#x4E0B;&#x9762;&#x6709;&#x4E2A; logs &#x6587;&#x4EF6;&#x5939;&#xFF0C;&#x90A3;&#x91CC;&#x9762;&#x662F;
        &#x4E13;&#x95E8;&#x653E;&#x7F6E;&#x65E5;&#x5FD7;&#x6587;&#x4EF6;&#x7684;&#xFF0C;&#x4E5F;&#x53EF;&#x4EE5;&#x4FEE;&#x6539;&#x4E3A;&#x5176;&#x4ED6;&#x8DEF;&#x5F84;&#x3002;
        <br
        />Prefix:&#x8FD9;&#x4E2A;&#x662F;&#x65E5;&#x5FD7;&#x6587;&#x4EF6;&#x7684;&#x540D;&#x79F0;&#x524D;&#x7F00;&#xFF0C;&#x65E5;&#x5FD7;&#x540D;&#x79F0;&#x4E3A;
        localhost_access_log.200810-22.txt&#xFF0C;&#x524D;&#x9762;&#x7684;&#x524D;&#x7F00;&#x5C31;&#x662F;&#x8FD9;&#x4E2A;
        localhost_access_log&#x3002;
        <br />Suffix:&#x6587;&#x4EF6;&#x540E;&#x7F00;&#x540D;&#x3002;
        <br />Pattern:common &#x65B9;&#x5F0F;&#x65F6;&#xFF0C;&#x5C06;&#x8BB0;&#x5F55;&#x8BBF;&#x95EE;&#x6E90;
        IP&#x3001;&#x672C;&#x5730;&#x670D;&#x52A1;&#x5668; IP&#x3001;&#x8BB0;&#x5F55;&#x65E5;&#x5FD7;&#x670D;&#x52A1;&#x5668;
        IP&#x3001;&#x8BBF;&#x95EE;&#x65B9;&#x5F0F;&#x3001;&#x53D1;&#x9001;&#x5B57;&#x8282;&#x6570;&#x3001;&#x672C;&#x5730;&#x63A5;&#x6536;&#x7AEF;&#x53E3;&#x3001;&#x8BBF;&#x95EE;
        URL &#x5730;&#x5740;&#x7B49;&#x76F8;&#x5173;&#x4FE1;&#x606F;&#x5728;&#x65E5;
        &#x5FD7;&#x6587;&#x4EF6;&#x4E2D;&#x3002;
        <br />resolveHosts:&#x503C;&#x4E3A; true &#x65F6;&#xFF0C;tomcat &#x4F1A;&#x5C06;&#x8FD9;&#x4E2A;&#x670D;&#x52A1;&#x5668;
        IP &#x5730;&#x5740;&#x901A;&#x8FC7; DNS &#x8F6C;&#x6362;&#x4E3A; &#x4E3B;&#x673A;&#x540D;&#xFF0C;&#x5982;&#x679C;&#x662F;
        false&#xFF0C;&#x5C31;&#x76F4;&#x63A5;&#x5199;&#x670D;&#x52A1;&#x5668; IP
        &#x5730;&#x5740;</td>
    </tr>
  </tbody>
</table>



## 3、通信协议

### 1、通信加密

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
      <td style="text-align:left">&#x5BF9;&#x4E8E;&#x901A;&#x8FC7; HTTP &#x534F;&#x8BAE;&#x8FDB;&#x884C;&#x8FDC;&#x7A0B;&#x7EF4;&#x62A4;&#x7684;&#x8BBE;&#x5907;&#xFF0C;&#x8BBE;&#x5907;&#x5E94;&#x652F;&#x6301;&#x4F7F;&#x7528;
        HTTPS &#x7B49;&#x52A0;&#x5BC6;&#x534F; &#x8BAE;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />1&#x3001;&#x4F7F;&#x7528; JDK &#x81EA;&#x5E26;&#x7684; keytool &#x5DE5;&#x5177;&#x751F;&#x6210;&#x4E00;&#x4E2A;&#x8BC1;&#x4E66;
          <br
          />%JAVA_HOME%\bin\keytool -genkey -alias cat -keyalg RSA -keystore 1024
          validity 365 -keystore /home/keystore/certificate.keystore
          <br />-genkeypair &#x8868;&#x793A;&#x751F;&#x6210;&#x5BC6;&#x94A5;
          <br />-keyalg &#x6307;&#x5B9A;&#x5BC6;&#x94A5;&#x7B97;&#x6CD5;,&#x8FD9;&#x91CC;&#x6307;&#x5B9A;&#x4E3A;
          RSA &#x7B97;&#x6CD5;&#x3002;
          <br />-keysize &#x6307;&#x5B9A;&#x5BC6;&#x94A5;&#x957F;&#x5EA6;,&#x9ED8;&#x8BA4;
          1024 &#x4F4D;,&#x8FD9;&#x91CC;&#x6307;&#x5B9A;&#x4E3A; 2048 &#x4F4D;&#x3002;
          <br
          />-sigalg &#x6307;&#x5B9A;&#x6570;&#x5B57;&#x7B7E;&#x540D;&#x7B97;&#x6CD5;,&#x8FD9;&#x91CC;&#x6307;&#x5B9A;&#x4E3A;
          SHA1withRSA &#x7B97;&#x6CD5;&#x3002;
          <br />-validity &#x6307;&#x5B9A;&#x8BC1;&#x4E66;&#x6709;&#x6548;&#x671F;,&#x8FD9;&#x91CC;&#x6307;&#x5B9A;&#x4E3A;
          36000 &#x5929;&#x3002;</p>
        <p>-alias &#x6307;&#x5B9A;&#x522B;&#x540D; -keystore &#x6307;&#x5B9A;&#x5BC6;&#x94A5;&#x5E93;&#x5B58;&#x50A8;&#x4F4D;&#x7F6E;&#xFF0C;&#x8FD9;&#x91CC;&#x662F;/home/keystore/app.keystore
          <br
          />2&#x3001;&#x7F16;&#x8F91; /conf/server.xml &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;&#x66F4;&#x6539;&#x4E3A;&#x4F7F;&#x7528;
          https &#x65B9;&#x5F0F;&#xFF0C;&#x786E;&#x8BA4;&#x4EE5;&#x4E0B; SSL &#x914D;&#x7F6E;&#x5185;&#x5BB9;&#x672A;&#x88AB;&#x6CE8;&#x91CA;&#x6807;&#x8BB0;&#x3002;
          <br
          />CConnector port=&quot;8443&quot; protocol=&quot;org.apache.coyote.http11.Http11Protocol&quot;
          <br
          />maxThreads=&quot;150&quot; SSLEnabled=&quot;true&quot; scheme=&quot;https&quot;
          secure=&quot;true&quot; keystoreFile=&quot;/home/keystore/certificate.keystore&quot;
          keystorePass=&quot;1QAZ2WSX&quot; clientAuth=&quot;false&quot; sslProtocol=&quot;TLS
          <br
          />&#x5176;&#x4E2D; keystorePass &#x7684;&#x503C;&#x4E3A;&#x751F;&#x6210;
          keystore &#x65F6;&#x8F93;&#x5165;&#x7684;&#x53E3;&#x4EE4;
          <br />3&#x3001;&#x91CD;&#x65B0;&#x542F;&#x52A8; tomcat &#x670D;&#x52A1; &#x7B26;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x4F7F;&#x7528; https &#x65B9;&#x5F0F;&#x767B;&#x9646; tomcat &#x670D;&#x52A1;&#x5668;&#x9875;&#x9762;&#xFF0C;&#x5E76;&#x80FD;&#x6210;&#x529F;&#x767B;&#x9646;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5F71;&#x54CD;&#x6027;&#x5206;&#x6790;</td>
      <td style="text-align:left">&#x52A0;&#x56FA;&#x64CD;&#x4F5C;&#x5BF9;&#x4E1A;&#x52A1;&#x7CFB;&#x7EDF;&#x7684;&#x5F71;&#x54CD;&#x9700;&#x8981;&#x5728;&#x6D4B;&#x8BD5;&#x73AF;&#x5883;&#x4E2D;&#x8FDB;&#x884C;&#x6D4B;&#x8BD5;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">&#x6839;&#x636E;&#x5E94;&#x7528;&#x573A;&#x666F;&#x7684;&#x4E0D;&#x540C;&#xFF0C;&#x9009;&#x62E9;&#x662F;&#x5426;&#x5F00;&#x542F;&#x3002;
        <br
        />1&#x3001;&#x5BF9;&#x4E8E;&#x8DE8;&#x7F51;&#x7EDC;&#x8FDC;&#x7A0B;&#x7BA1;&#x7406;&#x7EF4;&#x62A4;&#x65B9;&#x5F0F;&#xFF0C;&#x9700;&#x8981;&#x8FDB;&#x884C;&#x6B64;&#x9879;&#x914D;&#x7F6E;&#x3002;
        <br
        />2&#x3001;&#x5BF9;&#x4E8E;&#x672C;&#x5730;&#x7EF4;&#x62A4;&#xFF0C;&#x5219;&#x65E0;&#x9700;&#x914D;&#x7F6E;&#x3002;</td>
    </tr>
  </tbody>
</table>



## 4、安全管理

### 1、错误页面处理

| 项目 | 详情 |
| :--- | :--- |
| 说明 | Tomcat 错误页面重定向 |
| 检测步骤 | 1、参考配置操作 编辑/conf/web.xml 配置文件。  在最后&lt;/web-app&gt;一行之前加入以下内容： &lt;error-page&gt; &lt;error-code&gt;404&lt;/error-code&gt;  &lt;location&gt;/404.html &lt;/location&gt; &lt;/error-page&gt;  |
| 符合性依据 | 1、在 URL 地址栏中输入 http://ip/xxxxxxx~~~（一个不存在的页面） 检测是否指向指定错误页面。  2、查看 web.xml 配置文件内容 |
| 备注 | 实现将 404 未找到的 jsp 网页导向至 404.html 页面（webapps/ROOT/目录 下），也可以用类似方法添加其多的错误代码导向页面，如 403,500 等。 |



### 2、删除默认文件

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 删除缺省安装的自带 web 文件 |
| 检测步骤 | 1、参考配置操作  删除以下例子程序和无用文件：  tomcat/webapps/examples  tomcat/webapps/docs  tomcat/webapps/ROOT  如和开发人员确认 host-manager、manager 文件不使用，可以在修改部分 文件后删除以下文件（可选）：  tomcat/webapps/host-manager  tomcat/webapps/manager  根据 Tomcat 安装步骤不同和版本不同，某些目录或文件可能不存在或位置 不同 |
| 符合性依据 | 检查对应目录。 |
| 影响性分析 | 加固操作对业务系统的影响需要在测试环境中进行测试。 |



### 3、登录超时

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
      <td style="text-align:left">&#x5BF9;&#x4E8E;&#x5177;&#x5907;&#x5B57;&#x7B26;&#x4EA4;&#x4E92;&#x754C;&#x9762;&#x7684;&#x8BBE;&#x5907;&#xFF0C;&#x5E94;&#x652F;&#x6301;&#x975E;&#x6D3B;&#x52A8;&#x8D26;&#x6237;&#x5B9A;&#x65F6;&#x81EA;&#x52A8;&#x767B;&#x51FA;&#x3002;&#x767B;&#x51FA;&#x540E;
        &#x7528;&#x6237;&#x9700;&#x518D;&#x6B21;&#x767B;&#x5F55;&#x624D;&#x80FD;&#x8FDB;&#x5165;&#x7CFB;&#x7EDF;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />&#x7F16;&#x8F91;/conf/server.xml &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#xFF0C;&#x4FEE;&#x6539;
          connectionTimeout &#x53C2;&#x6570;&#x4E3A; 60 &#x79D2;</p>
        <p>&lt;Connector port=&quot;8080&quot; maxHttpHeaderSize=&quot;8192&quot;maxThreads=&quot;150&quot;
          minSpareThreads=&quot;25&quot; maxSpareThreads=&quot;75&quot;&#x3001; enableLookups=&quot;false&quot;
          redirectPort=&quot;8443&quot; acceptCount=&quot;100&quot; connectionTimeout=&quot;60000&quot;
          disableUploadTimeout=&quot;true&quot;/&gt;
          <br />&#x6216;&#x662F;
          <br />&lt;Connector port=&quot;8080&quot; protocol=&quot;HTTP/1.1&quot; connectionTimeout=&quot;60000&quot;
          redirectPort=&quot;8443&quot;/&gt;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x975E;&#x6D3B;&#x52A8;&#x8D26;&#x6237; 30 &#x79D2;&#x540E;&#x4F1A;&#x81EA;&#x52A8;&#x767B;&#x51FA;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5F71;&#x54CD;&#x6027;&#x5206;&#x6790;</td>
      <td style="text-align:left">&#x52A0;&#x56FA;&#x64CD;&#x4F5C;&#x5BF9;&#x4E1A;&#x52A1;&#x7CFB;&#x7EDF;&#x7684;&#x5F71;&#x54CD;&#x9700;&#x8981;&#x5728;&#x6D4B;&#x8BD5;&#x73AF;&#x5883;&#x4E2D;&#x8FDB;&#x884C;&#x6D4B;&#x8BD5;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">connectionTimeout &#x53C2;&#x6570;&#x9700;&#x8981;&#x8003;&#x8651;&#x5230;&#x7F51;&#x7EDC;&#x7A33;&#x5B9A;&#x6027;&#x7684;&#x540C;&#x65F6;&#x4E5F;&#x9700;&#x8981;&#x8003;&#x8651;&#x4E1A;&#x52A1;&#x6027;
        &#x80FD;&#x3002;&#x5BF9;&#x4E8E;&#x9762;&#x5411;&#x4E92;&#x8054;&#x7F51;&#x7684;&#x5E94;&#x7528;&#xFF0C;&#x6B64;&#x503C;&#x5E94;&#x6D4B;&#x8BD5;&#x540E;&#x8BBE;&#x7F6E;&#xFF0C;&#x6682;&#x63A8;&#x8350;&#x4E3A;
        20000</td>
    </tr>
  </tbody>
</table>



### 4、虚拟目录访问控制

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 禁止 tomcat 虚拟目录显示文件,增加系统安全性。 |
| 检测步骤 | 1、参考配置操作  编辑/conf/web.xml 配置文件，把 listings 参数修改成 false。   &lt;init-param&gt;  &lt;param-name&gt;listings&lt;/param-name&gt; &lt;param-value&gt; false&lt;/param-value&gt;  &lt;/init-param&gt;  重新启动 tomcat 服务 |
| 符合性依据 | 直接访问 http://ip:8080/XXX。其中 XXX 为\webapps\下存在的应用文档，确 认无公开资源即符合安全基线 |



## 5、补丁管理

### 1、补丁管理

| 项目 | 详情 |
| :--- | :--- |
| 检测步骤 | 补丁或者版本升级应联系业务系统开发商咨询和测试，确认测试无问题后， 进行更新 |
| 影响性分析 | 补丁更新加固操作对业务有不可预知的影响，一定需要在测试环境中进行测 试 |





