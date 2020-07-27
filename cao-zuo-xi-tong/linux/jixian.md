# 基线检查

## 1、身份鉴别

### 1、口令复杂度

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
      <td style="text-align:left">&#x5BF9;&#x4E8E;&#x91C7;&#x7528;&#x9759;&#x6001;&#x53E3;&#x4EE4;&#x8BA4;&#x8BC1;&#x6280;&#x672F;&#x7684;&#x8BBE;&#x5907;&#xFF0C;&#x53E3;&#x4EE4;&#x957F;&#x5EA6;&#x81F3;&#x5C11;
        8 &#x4F4D;&#xFF0C;&#x5E76;&#x81F3;&#x5C11;&#x5305;&#x62EC;&#x6570;&#x5B57;&#x3001;
        &#x5C0F;&#x5199;&#x5B57;&#x6BCD;&#x3001;&#x5927;&#x5199;&#x5B57;&#x6BCD;&#x548C;&#x7279;&#x6B8A;&#x7B26;&#x53F7;&#x4E2D;&#x7684;&#x4E09;&#x79CD;&#x3002;&#x589E;&#x5F3A;&#x53E3;&#x4EE4;&#x590D;&#x6742;&#x5EA6;&#x662F;&#x4E3A;&#x4E86;&#x589E;&#x52A0;&#x653B;&#x51FB;
        &#x8005;&#x7834;&#x89E3;&#x5BC6;&#x7801;&#x7684;&#x96BE;&#x5EA6;&#x548C;&#x65F6;&#x95F4;&#x6210;&#x672C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x6267;&#x884C;&#xFF1A;more /etc/pam.d/system-auth &#x68C0;&#x67E5;&#x6587;&#x4EF6;&#x4E2D;&#x662F;&#x5426;&#x5BF9;
          pam_cracklib.so &#x7684;&#x53C2;&#x6570;&#x8FDB;&#x884C;&#x4E86;&#x6B63;&#x786E;&#x8BBE;&#x7F6E;&#xFF1B;
          <br
          />2&#x3001;&#x6267;&#x884C;&#xFF1A;awk -F: &apos;($2 == &quot;&quot;) {
          print $1 }&apos; /etc/shadow&#xFF0C;&#x68C0;&#x67E5;&#x662F;&#x5426;&#x5B58;&#x5728;&#x7A7A;&#x53E3;&#x4EE4;
          &#x5E10;&#x53F7;&#x3002;</p>
        <p></p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C; # cat /etc/pam.d/system-auth&#xFF0C;&#x627E;&#x5230;
        password &#x6A21;&#x5757;&#x63A5;&#x53E3;&#x7684;&#x914D;&#x7F6E;&#x90E8;&#x5206;&#xFF0C;&#x627E;&#x5230;&#x7C7B;
        &#x4F3C;&#x5982;&#x4E0B;&#x7684;&#x914D;&#x7F6E;&#x884C;&#xFF1A;
        <br />password requisite /lib/security/$ISA/pam_cracklib.so minlen =8 &#x53C2;&#x6570;&#x8BF4;&#x660E;&#x5982;&#x4E0B;&#xFF1A;
        <br
        />1&#x3001;retry=N&#xFF0C;&#x786E;&#x5B9A;&#x7528;&#x6237;&#x521B;&#x5EFA;&#x5BC6;&#x7801;&#x65F6;&#x5141;&#x8BB8;&#x91CD;&#x8BD5;&#x7684;&#x6B21;&#x6570;&#xFF1B;
        <br
        />2&#x3001;minlen=N&#xFF0C;&#x786E;&#x5B9A;&#x5BC6;&#x7801;&#x6700;&#x5C0F;&#x957F;&#x5EA6;&#x8981;&#x6C42;&#xFF0C;&#x4E8B;&#x5B9E;&#x4E0A;&#xFF0C;&#x5728;&#x9ED8;&#x8BA4;&#x914D;&#x7F6E;&#x4E0B;&#xFF0C;&#x6B64;&#x53C2;&#x6570;&#x4EE3;
        &#x8868;&#x5BC6;&#x7801;&#x6700;&#x5C0F;&#x957F;&#x5EA6;&#x4E3A; N-1&#xFF1B;
        <br
        />3&#x3001;dcredit=N&#xFF0C;&#x5F53; N &#x5C0F;&#x4E8E; 0 &#x65F6;&#xFF0C;&#x4EE3;&#x8868;&#x65B0;&#x5BC6;&#x7801;&#x4E2D;&#x6570;&#x5B57;&#x5B57;&#x7B26;&#x6570;&#x91CF;&#x4E0D;&#x5F97;&#x5C11;&#x4E8E;&#xFF08;N&#xFF09;&#x4E2A;&#x3002;&#x4F8B;&#x5982;&#xFF0C;dcredit=-2
        &#x4EE3;&#x8868;&#x5BC6;&#x7801;&#x4E2D;&#x8981;&#x81F3;&#x5C11;&#x5305;&#x542B;&#x4E24;&#x4E2A;&#x6570;&#x5B57;&#x5B57;&#x7B26;&#xFF1B;
        <br
        />4&#x3001;ucredit=N&#xFF0C;&#x5F53; N &#x5C0F;&#x4E8E; 0 &#x65F6;&#xFF0C;&#x4EE3;&#x8868;&#x5219;&#x65B0;&#x5BC6;&#x7801;&#x4E2D;&#x5927;&#x5199;&#x5B57;&#x7B26;&#x6570;&#x91CF;&#x4E0D;&#x5F97;&#x5C11;&#x4E8E;&#xFF08;N&#xFF09;&#x4E2A;&#xFF1B;
        <br
        />5&#x3001;lcredit=N&#xFF0C;&#x5F53; N &#x5C0F;&#x4E8E; 0 &#x65F6;&#xFF0C;&#x4EE3;&#x8868;&#x5219;&#x65B0;&#x5BC6;&#x7801;&#x4E2D;&#x5C0F;&#x5199;&#x5B57;&#x7B26;&#x6570;&#x91CF;&#x4E0D;&#x5F97;&#x5C11;&#x4E8E;&#xFF08;-N&#xFF09;
        &#x4E2A;&#xFF1B;
        <br />6&#x3001;ocredit=N&#xFF0C;&#x5F53; N &#x5C0F;&#x4E8E; 0 &#x65F6;&#xFF0C;&#x4EE3;&#x8868;&#x5219;&#x65B0;&#x5BC6;&#x7801;&#x4E2D;&#x7279;&#x6B8A;&#x5B57;&#x7B26;&#x6570;&#x91CF;&#x4E0D;&#x5F97;&#x5C11;&#x4E8E;&#xFF08;
        N&#xFF09;&#x4E2A;&#xFF1B;2.&#x4E0D;&#x5B58;&#x5728;&#x7A7A;&#x53E3;&#x4EE4;&#x5E10;&#x53F7;&#x3002;
        <br
        />shadow &#x6587;&#x4EF6;&#x7B2C;&#x4E8C;&#x4E2A;&#x5B57;&#x6BB5;&#x4E3A;&#x7A7A;&#x5373;&#x5B58;&#x5728;&#x7A7A;&#x53E3;&#x4EE4;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5F71;&#x54CD;&#x6027;&#x5206;&#x6790;</td>
      <td style="text-align:left">&#x52A0;&#x56FA;&#x64CD;&#x4F5C;&#x5BF9;&#x4E1A;&#x52A1;&#x65E0;&#x5F71;&#x54CD;&#xFF0C;&#x52A0;&#x56FA;&#x64CD;&#x4F5C;&#x5BF9;&#x4E1A;&#x52A1;&#x65E0;&#x5F71;&#x54CD;,&#x4F46;&#x662F;&#x64CD;&#x4F5C;&#x5931;&#x8BEF;&#x4F1A;&#x5BFC;&#x81F4;&#x7CFB;&#x7EDF;&#x8BA4;
        &#x8BC1;&#x9519;&#x8BEF;&#xFF0C;&#x65E0;&#x6CD5;&#x6B63;&#x5E38;&#x767B;&#x9646;&#x3002;
        &#x5EFA;&#x8BAE;&#x5728;&#x4FEE;&#x6539;&#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x524D;&#xFF0C;&#x5BF9;&#x539F;&#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x8FDB;&#x884C;&#x5907;&#x4EFD;
        cp xx xx20190503.bak</td>
    </tr>
  </tbody>
</table>

![](../../.gitbook/assets/image%20%2850%29.png)

### 2、口令生存期

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 对于采用静态口令认证技术的设备，帐户口令的生存期不长于 90 天。最短期 限为 6 天。  由于所有密码理论上都是可以被破解的，只是时间长短问题，设置口令生存期 是为了让攻击者破解密码后密码已经更新 |
| 检测步骤 | 执行：more /etc/login.defs，检查 PASS\_MAX\_DAYS、 PASS\_MIN\_DAYS、 PASS\_WARN\_AGE 参数 |
| 符合性依据 | 参考配置操作  用 vi 编辑/etc/login.defs 文件中配置：  PASS\_MAX\_DAYS 90\#新建用户的口令最长使用天数  PASS\_MIN\_DAYS 6\#新建用户的口令最短使用天数  PASS\_WARN\_AGE 7\#新建用户的口令到期提前提醒天数 |

![](../../.gitbook/assets/image%20%2847%29.png)

### 3、默认账户

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 攻击者一般都从系统默认帐户进行尝试破解。应锁定与设备运行、维护等工作无关的账户 |
| 检测步骤 | 查看锁定用户：  \# cat /etc/passwd，查看哪些账户的 shell 域中为 nologin |
| 符合性依据 | 系统默认帐户应被禁止登录或者锁定如下账号：  lp, sync, shutdown, halt, news, uucp, operator, games, gopher 。  参考配置操作：  锁定用户：  修改/etc/passwd 文件，将需要锁定的用户的 shell 域设为 nologin；  通过\#passwd –l username 锁定账户；  只有具备超级用户权限的使用者方可使用\#passwd –l username 锁定用户,用 \#passwd –d username 解锁后原有密码失效，登录需输入新密码 |
| 回退操作 | 账户解锁：  \# usermod -U username 或\# passwd -u username |

![](../../.gitbook/assets/image%20%2857%29.png)

### 4、共享账户

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 系统需按照实际用户分配账户；  避免不同用户间共享账户。 |
| 检测步骤 | 使用命令 cat /etc/passwd 查看当前所有用户的信息，与管理员确认是否有共 享账户情况存在 |
| 符合性依据 | 若存在账户共享的情况，则低于安全要求；  参考配置操作：  cat /etc/passwd 查看当前所有用户的情况；  如需建立用户，参考如下：  \#useradd username \#创建账户  \#passwd username \#设置密码  使用该命令为不同的用户分配不同的账户，设置不同的口令及权限信息等。 |

![](../../.gitbook/assets/image%20%2843%29.png)

### 5、FTP账户

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 如果系统开启 FTP 服务，检查系统是否禁止 ROOT 等系统帐户远程访问 FTP FTP 容易被攻击者攻击获得权限，FTP 与系统帐户不共享，防止攻击者进一步 获取系统帐户攻击操作系统 |
| 检测步骤 | \(一\) 查看 FTP 进程:  1、ps -ef \|grep ftp，检查是否有 ftp 进程;  2、chkconfig --list \|grep ftp，检查 ftp 服务是否开机启动；  若以上两项未开启 FTP，则忽略以下 FTP 帐户配置。  \(二\) 检查 FTP 帐户  1、默认 FTP 就是禁止 root 用户登录的（如果没有禁用在/etc/vsftpd/ftpusers 和/etc/vsftpd/user\_list，把 root 去掉）  2、更改/etc/vsftpd/vsftpd.conf 文件，查看是否有： userlist\_enable=YES 这条 配置，如果没有则加上； 再增加 userlist\_deny=NO（只允许 userlist 文件的用 户登录 FTP，其它默认用户不允许）  3、更改/etc/vsftpd/user\_list 文件 添加 ftp 允许登陆的帐户，并注释掉原所有帐户；  4、重新启动 vsftpd 服务 service vsftpd restart |
| 符合性依据 | 如不使用 FTP，关闭 FTP；  如需使用，专门为 FTP 设置账号，与系统帐户不共享。 |

![](../../.gitbook/assets/image%20%2845%29.png)

### 6、超级用户控制

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 帐号与口令-检查是否存在除 root 之外 UID 为 0 的用户\(超级用户\)  一般默认情况无其他超级帐户（运维主动添加除外），排除攻击者暗藏添加的超级用户长期潜伏的可能 |
| 检测步骤 | 执行：awk -F: '\($3 == 0\) { print $1 }' /etc/passwd |
| 符合性依据 | 返回值包括“root”以外的条目，则低于安全要求； |
| 备注 | 补充操作说明：  UID 为 0 的任何用户都拥有系统的最高特权，保证只有 root 用户的 UID 为 0 修改方法：编辑 /etc/passwd 文件（文件内容结构为 Root : x : 0 : 0 : root : /root : /bin/bash）把此用户的的第三项改为非 0 的数值，第三项表示用户 UID 值 |

![](../../.gitbook/assets/image%20%2840%29.png)

### 7、用户锁定策略

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 对于采用静态口令认证技术的设备，应配置当用户连续认证失败次数超过 5 次，锁定该用户使用的帐号。  用户认证失败锁定是为了防止暴力破解。 |
| 检测步骤 | 查看文件中是否对 pam\_tally2.so 的参数进行了正确设置。  执行：more /etc/pam.d/system-auth |
| 符合性依据 | 连续输错 5 次口令，帐号锁定 5 分钟，  参考配置操作：  使用命令“vi /etc/pam.d/system-auth”修改配置文件 在进行此项安全加固工作前，请先检查 PAM 模块版本，搜索 pam\_tally2 是 否存在，如果 pam\_tally2 存在,请在 system-auth 文件中  配 置 参 数 auth required pam\_tally2.so deny=5unlock\_time=300 even\_deny\_root root\_unlock\_time=300  如果不存在，请在 auth 段最后一行添加上述行。 |

### 

## 2、访问控制

### 1、SSH登录维护

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
      <td style="text-align:left">&#x7CFB;&#x7EDF;&#x5E94;&#x914D;&#x7F6E;&#x4F7F;&#x7528; SSH &#x7B49;&#x52A0;&#x5BC6;&#x534F;&#x8BAE;&#x8FDB;&#x884C;&#x8FDC;&#x7A0B;&#x767B;&#x5F55;&#x7EF4;&#x62A4;&#xFF0C;&#x5E76;&#x5B89;&#x5168;&#x914D;&#x7F6E;
        SSHD &#x7684;&#x8BBE; &#x7F6E;&#x3002;&#x4E0D;&#x80FD;&#x4F7F;&#x7528;
        TELNET &#x8FD9;&#x7C7B;&#x53EF;&#x4EE5;&#x88AB;&#x55C5;&#x63A2;&#x5DE5;&#x5177;&#x6293;&#x5305;&#x83B7;&#x53D6;&#x654F;&#x611F;&#x4FE1;&#x606F;&#x7684;&#x660E;&#x6587;&#x4F20;&#x8F93;&#x534F;
        &#x8BAE;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>&#x67E5;&#x770B; SSH &#x670D;&#x52A1;&#x72B6;&#x6001;&#xFF1A;
          <br />ps &#x2013;elf|grep ssh</p>
        <p>&#x67E5;&#x770B; telnet &#x670D;&#x52A1;&#x72B6;&#x6001;&#xFF1A;
          <br />ps &#x2013;elf|grep telnet</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x6CA1;&#x6709;&#x4F7F;&#x7528; telnet &#x8FDB;&#x884C;&#x8FDC;&#x7A0B;&#x7EF4;&#x62A4;
        <br
        />&#x4F7F;&#x7528; SSH &#x8FDB;&#x884C;&#x8FDC;&#x7A0B;&#x7EF4;&#x62A4;
        <br
        />&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;&#xFF1A;
        <br />SSH &#x914D;&#x7F6E;&#x8981;&#x7B26;&#x5408;&#x5982;&#x4E0B;&#x8981;&#x6C42;
        <br
        />Protocol 2 #&#x4F7F;&#x7528; ssh2 &#x7248;&#x672C;
        <br />X11Forwarding yes #&#x5141;&#x8BB8;&#x7A97;&#x53E3;&#x56FE;&#x5F62;&#x4F20;&#x8F93;&#x4F7F;&#x7528;
        ssh &#x52A0;&#x5BC6;
        <br />IgnoreRhosts yes#&#x5B8C;&#x5168;&#x7981;&#x6B62; SSHD &#x4F7F;&#x7528;.rhosts
        &#x6587;&#x4EF6;
        <br />RhostsAuthentication no #&#x4E0D;&#x8BBE;&#x7F6E;&#x4F7F;&#x7528;&#x57FA;&#x4E8E;
        rhosts &#x7684;&#x5B89;&#x5168;&#x9A8C;&#x8BC1; RhostsRSAAuthentication
        no #&#x4E0D;&#x8BBE;&#x7F6E;&#x4F7F;&#x7528; RSA &#x7B97;&#x6CD5;&#x7684;&#x57FA;&#x4E8E;
        rhosts &#x7684;&#x5B89;&#x5168;&#x9A8C;&#x8BC1;
        <br />HostbasedAuthentication no #&#x4E0D;&#x5141;&#x8BB8;&#x57FA;&#x4E8E;&#x4E3B;&#x673A;&#x767D;&#x540D;&#x5355;&#x65B9;&#x5F0F;&#x8BA4;&#x8BC1;
        PermitRootLogin no #&#x4E0D;&#x5141;&#x8BB8; root &#x767B;&#x5F55;
        <br />PermitEmptyPasswords no #&#x4E0D;&#x5141;&#x8BB8;&#x7A7A;&#x5BC6;&#x7801;
        <br
        />Banner /etc/motd #&#x8BBE;&#x7F6E; ssh &#x767B;&#x5F55;&#x65F6;&#x663E;&#x793A;&#x7684;
        banner</td>
    </tr>
  </tbody>
</table>

![](../../.gitbook/assets/image%20%2849%29.png)

### 2、SSH-ROOT远程登录限制

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 1、限制具备超级管理员权限的用户远程登录。  2、远程执行管理员权限操作，应先以普通权限用户远程登录后，再切换到超 级管理员权限账户后执行相应操作。 |
| 检测步骤 | 使用命令“cat /etc/ssh/sshd\_config”查看配置文件  （1）检查是否允许 root 直接登录            检查“PermitRootLogin”的值是否为 no  （2）检查 SSH 使用的协议版本            检查“Protocol”的值 |
| 影响性分析 | 以 root 进行远程登录的管理员或相关系统都将无法登录。 |
| 回退操作 | 修改/etc/ssh/sshd\_config 文件，将 PermitRootLogin no 改为 PermitRootLogin yes，重启 sshd 服务 |
| 备注 | 远程执行管理员权限操作，应先以普通权限用户远程登录后，再切换到超级 管理员权限账户后执行相应操作 |

![](../../.gitbook/assets/image%20%2854%29.png)

### 3、登录超时设置

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 对于具备字符交互界面的设备，配置定时帐户自动登出。当管理员未对系统操作 10 分钟后，则系统自动注销该账户 |
| 检测步骤 | 使用命令“cat /etc/profile \|grepTMOUT”查看 TMOUT 是否被设置 |
| 符合性依据 | 返回值为空或是时间较长，则低于安全要求。  参考配置操作  通过修改账户中“TMOUT”参数，可以实现此功能。 TMOUT 按秒计算。  编辑 profile 文件（vi /etc/profile），在“HISTFILESIZE=”后面加入这行： TMOUT=600 |
| 备注 | 改变这项设置后，必须先注销用户，再用该用户登录才能激活这个功能 |

![](../../.gitbook/assets/image%20%2841%29.png)

### 4、关键目录权限

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
      <td style="text-align:left">&#x6839;&#x636E;&#x5B89;&#x5168;&#x9700;&#x6C42;&#x5BF9;&#x91CD;&#x8981;&#x76EE;&#x5F55;&#x548C;&#x6587;&#x4EF6;&#x7684;&#x6743;&#x9650;&#x8FDB;&#x884C;&#x8BBE;&#x7F6E;&#x3002;
        <br
        />&#x91CD;&#x70B9;&#x8981;&#x6C42; password &#x914D;&#x7F6E;&#x6587;&#x4EF6;&#x3001;shadow
        &#x6587;&#x4EF6;&#x3001;group &#x6587;&#x4EF6;&#x6743;&#x9650;&#x3002;
        <br
        />&#x5F53;&#x524D;&#x4E3B;&#x6D41;&#x7248;&#x672C;&#x7684; linux &#x7CFB;&#x7EDF;&#x5728;&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x5373;&#x5BF9;&#x91CD;&#x8981;&#x6587;&#x4EF6;&#x505A;&#x4E86;&#x5FC5;&#x8981;&#x7684;&#x6743;&#x9650;&#x8BBE;&#x7F6E;&#xFF0C;
        &#x5728;&#x65E5;&#x5E38;&#x7BA1;&#x7406;&#x548C;&#x64CD;&#x4F5C;&#x8FC7;&#x7A0B;&#x4E2D;&#x5E94;&#x907F;&#x514D;&#x4FEE;&#x6539;&#x6B64;&#x7C7B;&#x6587;&#x4EF6;&#x6743;&#x9650;&#xFF0C;&#x9664;&#x6B64;&#x4EE5;&#x5916;&#xFF0C;&#x5E94;&#x5B9A;&#x671F;&#x5BF9;&#x6743;
        &#x9650;&#x8FDB;&#x884C;&#x68C0;&#x67E5;&#x53CA;&#x590D;&#x6838;&#xFF0C;&#x786E;&#x4FDD;&#x6743;&#x9650;&#x8BBE;&#x7F6E;&#x6B63;&#x786E;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">&#x6267;&#x884C;&#x4EE5;&#x4E0B;&#x547D;&#x4EE4;&#x68C0;&#x67E5;&#x76EE;&#x5F55;&#x548C;&#x6587;&#x4EF6;&#x7684;&#x6743;&#x9650;&#x8BBE;&#x7F6E;&#x60C5;&#x51B5;&#xFF0C;&#x5E76;&#x6309;&#x7167;&#x5982;&#x4E0B;&#x7684;&#x6743;&#x9650;&#x8BBE;&#x7F6E;&#xFF1A;
        ls &#x2013;l /etc 755
        <br />ls &#x2013;l /etc/rc.d/init.d 755
        <br />ls &#x2013;l /tmp 1777
        <br />&#xFF08;&#x53EA;&#x8981;&#x6C42;/tmp &#x6587;&#x4EF6;&#x5939;&#x4E3A;
        1777&#xFF0C;&#x5BF9;/tmp &#x7684;&#x5B50;&#x6587;&#x4EF6;&#x4E0D;&#x8981;&#x6C42;&#x4FEE;
        &#x6539;&#x6743;&#x9650;&#xFF09;
        <br />ls &#x2013;l /etc/passwd644
        <br />ls &#x2013;l /etc/shadow400
        <br />ls &#x2013;l /etc/group 644
        <br />ls &#x2013;l /etc/security755
        <br />ls &#x2013;l /etc/services 644
        <br />ls -l /etc/rc*.d 755</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x82E5;&#x6743;&#x9650;&#x5747;&#x4E3A; 777 &#x6216;&#x8005;&#x8FC7;&#x4F4E;&#xFF0C;&#x5219;&#x4F4E;&#x4E8E;&#x5B89;&#x5168;&#x8981;&#x6C42;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5F71;&#x54CD;&#x6027;&#x5206;&#x6790;</td>
      <td style="text-align:left">&#x52A0;&#x56FA;&#x64CD;&#x4F5C;&#x5BF9;&#x4E1A;&#x52A1;&#x7684;&#x5F71;&#x54CD;&#x9700;&#x5728;&#x6D4B;&#x8BD5;&#x73AF;&#x5883;&#x4E2D;&#x8FDB;&#x884C;&#x6D4B;&#x8BD5;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x56DE;&#x9000;&#x64CD;&#x4F5C;</td>
      <td style="text-align:left">&#x901A;&#x8FC7; chmod &#x547D;&#x4EE4;&#x5BF9;&#x76EE;&#x5F55;&#x7684;&#x6743;&#x9650;&#x8FDB;&#x884C;&#x5B9E;&#x9645;&#x8C03;&#x6574;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">
        <p>&#x5BF9;&#x4E8E;&#x91CD;&#x8981;&#x76EE;&#x5F55;&#xFF0C;&#x5EFA;&#x8BAE;&#x6267;&#x884C;&#x5982;&#x4E0B;&#x7C7B;&#x4F3C;&#x64CD;&#x4F5C;&#xFF1A;
          <br
          />chmod -R 750 /etc/rc.d/init.d/*</p>
        <p>&#x8FD9;&#x6837;&#x53EA;&#x6709; root &#x53EF;&#x4EE5;&#x8BFB;&#x3001;&#x5199;&#x548C;&#x6267;&#x884C;&#x8FD9;&#x4E2A;&#x76EE;&#x5F55;&#x4E0B;&#x7684;&#x811A;&#x672C;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>

![](../../.gitbook/assets/image%20%2846%29.png)

### 5、用户缺省权限

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 控制用户缺省访问权限，当在创建新文件或目录时应除去新文件或目录不应 有的访问允许权限,防止同属于该组的其它用户及别的组的用户修改该用户 的文件或更高限制 |
| 检测步骤 | 执行命令，查看文件下 umask 值设置情况。  more /etc/profile  more /etc/csh.cshrc  more /etc/bashrc |
| 符合性依据 | 若 umask 值小于 027，则低于安全要求  参考配置操作：  针对单独用户设置  可修改用户 home 目录下的.bash\_profile 脚本文件，例如，可增加一条语句： umask 027； 对于权限要求较严格的场合，建议设置为 077。  全局默认设置：  默认通过全局脚本/etc/bashrc 设置所有用户的默认 umask 值，修改脚本即可  实现对用户默认 umask 值的全局性修改，建议将 umask 设置为 022，对于权 限要求较严格的场合，建议设置为 077 |
| 备注 | umask 系统默认设置一般为 022，新创建的文件默认权限 755（777-022=755）；导致其他用户有执行权利。  在 UNIX 类系统中 777 是最高权限，4 是读，2 是写，1 是执行，加起来等于 7，三个 7 分别代表系统用户，本组用户和其他用户的权限。 |



## 3、安全审计

### 1、系统日志审计

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 系统应配置完备日志记录，记录对与系统相关的安全事件，方便发生安全时 间后溯源。 |
| 检测步骤 | 执行命令  1、\# cat /etc/syslog.conf 查看是否有对应配置  2、\# cat /var/log/secure 查看是否记录有需要的设备相关的安全事件  如不存在/etc/syslog.conf，则查看/etc/rsyslog.conf  检查参数 authpriv 的值 |
| 符合性依据 | 若未对所有登录事件都记录，则低于安全要求；  参考配置操作  authpriv. _/var/log/secure  //其中_是通配符，代表任何设备；none 表示不对任何级别的信息进行记录 |
| 影响性分析 | 加固操作增加系统资源开消，请检查系统资源是否充足。 |
| 回退操作 | 修改配置文件 vi /etc/syslog.conf。  配置如下类似语句，注释掉下面：  \#authpriv.\*/var/log/secure |

![](../../.gitbook/assets/image%20%2855%29.png)

### 2、日志远程存储

| 项目 | 详情 |
| :--- | :--- |
| 说明 | 当前系统应配置远程日志功能，将需要重点关注的日志内容传输到日志服务 器进行备份。方便发生安全事件后对事件进行溯源 |
| 检测步骤 | 执行：more /etc/syslog.conf，  如不存在/etc/syslog.conf，查看/etc/rsyslog.conf  查看是否设置了下列项：  \*_.\*_ @10.110.102.23 |
| 符合性依据 | 若未设置远程日志服务器，则低于安全要求。  参考配置操作  修改配置文件 vi /etc/syslog.conf，  加上这一行： \*_.\*_ @10.110.102.23  可以将"\*_.\*_"替换为你实际需要的日志信息。比如：kern.\* _/ mail.\*_ 等等；可以将此处 10.110.102.23 替换为实际的 IP 或域名。  重新启动 syslog 服务，执行下列命令：  services syslogd restart  补充操作说明  注意：\*_.\*_和@之间为一个 Tab 键 |
| 备注 | 建议配置专门的日志服务器，加强日志信息的异地同步备份。 |

![](../../.gitbook/assets/image%20%2848%29.png)

## 4、端口管理

### 1、关闭不必要服务

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
      <td style="text-align:left">&#x6839;&#x636E;&#x6BCF;&#x53F0;&#x673A;&#x5668;&#x7684;&#x4E0D;&#x540C;&#x89D2;&#x8272;&#xFF0C;&#x5173;&#x95ED;&#x4E0D;&#x9700;&#x8981;&#x7684;&#x7CFB;&#x7EDF;&#x670D;&#x52A1;&#xFF0C;&#x6839;&#x636E;&#x670D;&#x52A1;&#x5668;&#x7684;&#x89D2;&#x8272;&#x548C;&#x5E94;&#x7528;
        &#x60C5;&#x51B5;&#x5BF9;&#x542F;&#x52A8;&#x9879;&#x8FDB;&#x884C;&#x4FEE;&#x6539;&#x3002;&#x5BF9;&#x5916;&#x90E8;&#x5173;&#x95ED;&#x4E0D;&#x5FC5;&#x8981;&#x7684;&#x7AEF;&#x53E3;&#x3002;&#x5982;&#x65E0;&#x7279;&#x6B8A;&#x9700;&#x8981;&#xFF0C;&#x5E94;&#x5173;&#x95ED;
        Sendmail&#x3001;Telnet&#x3001;Bind &#x7B49;&#x670D;&#x52A1;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">&#x6267;&#x884C;&#xFF1A;netstat -nap&#xFF0C;&#x67E5;&#x770B;&#x7CFB;&#x7EDF;&#x7AEF;&#x53E3;&#x4F7F;&#x7528;&#x60C5;&#x51B5;
        <br
        />iptables -L -n&#xFF0C;&#x67E5;&#x770B;&#x5BF9;&#x7AEF;&#x53E3;&#x8BBF;&#x95EE;&#x9650;&#x5236;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">
        <p>&#x53C2;&#x8003;&#x914D;&#x7F6E;&#x64CD;&#x4F5C;
          <br />&#x5173;&#x95ED;&#x4E1A;&#x52A1;&#x4E0D;&#x9700;&#x8981;&#x7684;&#x7AEF;&#x53E3;&#xFF0C;&#x4F7F;&#x7528;&#x5982;&#x4E0B;&#x65B9;&#x5F0F;&#x7981;&#x7528;&#x4E0D;&#x5FC5;&#x8981;&#x7684;&#x670D;&#x52A1;&#x3002;
          <br
          /># service &lt;&#x670D;&#x52A1;&#x540D;&gt; stop
          <br /># chkconfig --level 35 off</p>
        <p>&#x4E1A;&#x52A1;&#x9700;&#x8981;&#x4F7F;&#x7528;&#x6216;&#x4E0D;&#x786E;&#x5B9A;&#x7684;&#x7AEF;&#x53E3;&#x670D;&#x52A1;&#x5E94;&#x5728;
          iptables &#x4E2D;&#x8FDB;&#x884C;&#x8BBF;&#x95EE;&#x63A7;&#x5236;&#x3002;
          <br
          />iptables -p INPUTACCEPT
          <br />iptables -p OUTPUT ACCEPT
          <br />iptables -p FORWARD ACCEPT
          <br />&#x9ED1;&#x540D;&#x5355;&#x6A21;&#x5F0F;&#x3002;
          <br />iptables -A INPUT -p tcp --dport31337 -j DROP
          <br />iptables -A FORWARD -p udp --dport135 -j DROP
          <br />&#x53C2;&#x8003;&#x8BF4;&#x660E;
          <br />Linux/Unix &#x7CFB;&#x7EDF;&#x670D;&#x52A1;&#x4E2D;&#xFF0C;&#x90E8;&#x5206;&#x670D;&#x52A1;&#x5B58;&#x5728;&#x8F83;&#x9AD8;&#x5B89;&#x5168;&#x98CE;&#x9669;&#xFF0C;&#x5E94;&#x5F53;&#x7981;&#x7528;&#xFF0C;&#x5305;&#x62EC;&#xFF1A;
          &#x201C;lpd&#x201D;&#xFF0C;&#x6B64;&#x670D;&#x52A1;&#x4E3A;&#x884C;&#x5F0F;&#x6253;&#x5370;&#x673A;&#x540E;&#x53F0;&#x7A0B;&#x5E8F;&#xFF0C;&#x7528;&#x4E8E;&#x5047;&#x8131;&#x673A;&#x6253;&#x5370;&#x5DE5;&#x4F5C;&#x7684;
          UNIX &#x540E;&#x53F0;&#x7A0B; &#x5E8F;&#xFF0C;&#x6B64;&#x670D;&#x52A1;&#x901A;&#x5E38;&#x60C5;&#x51B5;&#x4E0B;&#x4E0D;&#x7528;&#xFF0C;&#x5EFA;&#x8BAE;&#x7981;&#x7528;&#xFF1B;
          <br
          />&#x201C;telnet&#x201D;&#xFF0C;&#x6B64;&#x670D;&#x52A1;&#x91C7;&#x7528;&#x660E;&#x6587;&#x4F20;&#x8F93;&#x6570;&#x636E;&#xFF0C;&#x767B;&#x9646;&#x4FE1;&#x606F;&#x5BB9;&#x6613;&#x88AB;&#x7A83;&#x53D6;&#xFF0C;&#x5EFA;&#x8BAE;&#x7528;
          ssh &#x4EE3;&#x66FF;&#xFF1B;
          <br />&#x201C;routed&#x201D;&#xFF0C;&#x6B64;&#x670D;&#x52A1;&#x4E3A;&#x8DEF;&#x7531;&#x5B88;&#x5019;&#x8FDB;&#x7A0B;&#xFF0C;&#x4F7F;&#x7528;&#x52A8;&#x6001;
          RIP &#x8DEF;&#x7531;&#x9009;&#x62E9;&#x534F;&#x8BAE;&#xFF0C;&#x5EFA;&#x8BAE;&#x7981;&#x7528;&#xFF1B;
          <br
          />&#x201C;sendmail&#x201D;&#xFF0C;&#x6B64;&#x670D;&#x52A1;&#x4E3A;&#x90AE;&#x4EF6;&#x670D;&#x52A1;&#x5B88;&#x62A4;&#x8FDB;&#x7A0B;&#xFF0C;&#x975E;&#x90AE;&#x4EF6;&#x670D;&#x52A1;&#x5668;&#x5E94;&#x5C06;&#x5176;&#x5173;&#x95ED;&#xFF1B;
          <br
          />&#x201C;Bluetooth&#x201D;&#xFF0C;&#x6B64;&#x670D;&#x52A1;&#x4E3A;&#x84DD;&#x7259;&#x670D;&#x52A1;&#xFF0C;&#x5982;&#x679C;&#x4E0D;&#x9700;&#x8981;&#x84DD;&#x7259;&#x670D;&#x52A1;&#x65F6;&#x5E94;&#x5173;&#x95ED;&#xFF1B;
          <br
          />&#x201C;identd&#x201D;&#xFF0C;&#x6B64;&#x670D;&#x52A1;&#x4E3A; AUTH &#x670D;&#x52A1;&#xFF0C;&#x5728;&#x63D0;&#x4F9B;&#x7528;&#x6237;&#x4FE1;&#x606F;&#x65B9;&#x9762;&#x4E0E;
          finger &#x7C7B;&#x4F3C;&#xFF0C;&#x4E00;&#x822C;&#x60C5; &#x51B5;&#x4E0B;&#x8BE5;&#x670D;&#x52A1;&#x4E0D;&#x662F;&#x5FC5;&#x987B;&#x7684;&#xFF0C;&#x5EFA;&#x8BAE;&#x5173;&#x95ED;&#xFF1B;
          <br
          />&#x201C;xfs&#x201D;&#xFF0C;&#x6B64;&#x670D;&#x52A1;&#x4E3A; Linux &#x4E2D;
          X Window &#x7684;&#x5B57;&#x4F53;&#x670D;&#x52A1;&#xFF0C;&#x5173;&#x4E8E;&#x8BE5;&#x670D;&#x52A1;&#x5386;&#x53F2;&#x4E0A;&#x51FA;&#x73B0;&#x8FC7;&#x4FE1;
          &#x606F;&#x6CC4;&#x9732;&#x548C;&#x62D2;&#x7EDD;&#x670D;&#x52A1;&#x7B49;&#x6F0F;&#x6D1E;&#xFF0C;&#x5E94;&#x4EE5;&#x51CF;&#x5C11;&#x7CFB;&#x7EDF;&#x98CE;&#x9669;&#xFF1B;
          <br
          />R &#x670D;&#x52A1;&#xFF08;&#x201C;rlogin&#x201D;&#x3001;&#x201C;rwho&#x201D;&#x3001;&#x201C;rsh&#x201D;&#x3001;&#x201C;rexec&#x201D;&#xFF09;&#xFF0C;R
          &#x670D;&#x52A1;&#x8BBE;&#x8BA1;&#x4E0A;&#x5B58;&#x5728;&#x4E25;&#x91CD;&#x7684;&#x5B89;&#x5168;&#x7F3A;
          &#x9677;&#xFF0C;&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x5C01;&#x95ED;&#x73AF;&#x5883;&#x4E2D;&#x4FE1;&#x4EFB;&#x4E3B;&#x673A;&#x4E4B;&#x95F4;&#x4FBF;&#x6377;&#x8BBF;&#x95EE;&#xFF0C;&#x5176;&#x4ED6;&#x573A;&#x5408;&#x4E0B;&#x5747;&#x5FC5;&#x987B;&#x7981;&#x7528;&#xFF1B;
          <br
          />&#x57FA;&#x4E8E; inetd/xinetd &#x7684;&#x670D;&#x52A1;&#xFF08;daytime&#x3001;chargen&#x3001;echo
          &#x7B49;&#xFF09;&#xFF0C;&#x6B64;&#x7C7B;&#x670D;&#x52A1;&#x5EFA;&#x8BAE;&#x7981;&#x7528;&#x3002;</p>
        <p>&#x5173;&#x95ED;&#x9AD8;&#x5371;&#x7AEF;&#x53E3;&#xFF1A;
          <br />TCP&#xFF1A;31337 &#x5230; 31340&#xFF08;&#x7279;&#x6D1B;&#x4F0A;&#x6728;&#x9A6C;&#xFF09;&#x3001;
          31335&#x3001;27444&#x3001;27665&#x3001;20034&#xFF08;NetBus&#xFF09; &#x3001;
          9704&#x3001;111&#x3001;79&#x3001;137-139&#xFF08;smb&#xFF09;&#x3001;2049(NFS)&#x3001;2745&#x3001;3127&#x3001;6129
          &#x3001;1434&#x3001;593&#x3001; 445 &#x7AEF;&#x53E3;&#x7B49;
          <br />UDP &#xFF1A;135&#x3001;137&#x3001;138&#x3001;139&#x3001;445&#x3001;4444&#x3001;1434&#x3001;593
          &#x7AEF;&#x53E3;&#x7B49; /etc/rc.d/init.d/iptables save
          <br />service iptables restart</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5F71;&#x54CD;&#x6027;&#x5206;&#x6790;</td>
      <td style="text-align:left">&#x5173;&#x95ED;&#x670D;&#x52A1;&#x524D;&#xFF0C;&#x52A1;&#x5FC5;&#x548C;&#x7BA1;&#x7406;&#x5458;&#x786E;&#x5B9A;&#x6B64;&#x670D;&#x52A1;&#x5173;&#x95ED;&#x4E0D;&#x4F1A;&#x5BF9;&#x4F7F;&#x7528;&#x4E2D;&#x7684;&#x5E94;&#x7528;&#x7A0B;&#x5E8F;&#x9020;&#x6210;&#x5F71;&#x54CD;&#x3002;
        &#x6B64;&#x9879;&#x5DE5;&#x4F5C;&#x9700;&#x8981;&#x5728;&#x660E;&#x786E;&#x7CFB;&#x7EDF;&#x7AEF;&#x53E3;&#x5B9E;&#x9645;&#x7528;&#x9014;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x9650;&#x5236;&#x4E0D;&#x5FC5;&#x8981;&#x7684;&#x7AEF;&#x53E3;&#x8BBF;&#x95EE;&#xFF0C;&#x5EFA;
        &#x8BAE;&#x5148;&#x5728;&#x6D4B;&#x8BD5;&#x73AF;&#x5883;&#x4E2D;&#x8FDB;&#x884C;&#x6D4B;&#x8BD5;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x56DE;&#x9000;&#x64CD;&#x4F5C;</td>
      <td style="text-align:left">&#x67E5;&#x770B;&#x6240;&#x6709;&#x5F00;&#x542F;&#x7684;&#x670D;&#x52A1;&#xFF1A;
        <br
        />#ps &#x2013;eaf
        <br />&#x5728; inetd.conf &#x4E2D; &#x5173; &#x95ED; &#x4E0D; &#x7528; &#x7684;
        &#x670D; &#x52A1; &#x9996;&#x5148;&#x590D;&#x5236;/etc/inet/inetd.conf
        &#x3002;
        <br />#cp /etc/inet/inetd.conf /etc/inet/inetd.conf.backup
        <br />&#x7136;&#x540E;&#x7528;vi&#x7F16;&#x8F91;&#x5668;&#x7F16;&#x8F91;inetd.conf
        &#x6587;&#x4EF6;&#xFF0C;&#x628A;&#x5728;&#x76F8;&#x5E94;&#x884C;&#x5F00;&#x5934;&#x6807;&#x8BB0;&quot;#&quot;&#x5B57;&#x7B26;&#x5220;&#x9664;&#xFF0C;
        &#x91CD;&#x542F; inetd &#x8FDB;&#x7A0B;</td>
    </tr>
  </tbody>
</table>

![](../../.gitbook/assets/image%20%2851%29.png)

![](../../.gitbook/assets/image%20%2852%29.png)

## 5、其他安全配置

### 1、系统core dump状态

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
      <td style="text-align:left">&#x7CFB;&#x7EDF;&#x6587;&#x4EF6;-&#x7CFB;&#x7EDF; core dump &#x72B6;&#x6001;
        <br
        />core dump &#x8BB0;&#x5F55;&#x8BB8;&#x591A;&#x7CFB;&#x7EDF;&#x654F;&#x611F;&#x4FE1;&#x606F;&#xFF08;&#x7A0B;&#x5E8F;&#x8FD0;&#x884C;&#x65F6;&#x7684;&#x5185;&#x5B58;&#xFF0C;&#x5BC4;&#x5B58;&#x5668;&#x72B6;&#x6001;&#xFF0C;&#x5806;&#x6808;
        &#x6307;&#x9488;&#xFF0C;&#x5185;&#x5B58;&#x7BA1;&#x7406;&#x4FE1;&#x606F;&#x3001;&#x7CFB;&#x7EDF;&#x9519;&#x8BEF;&#x4FE1;&#x606F;&#xFF09;&#xFF0C;&#x53EA;&#x5728;&#x7CFB;&#x7EDF;&#x8C03;&#x8BD5;&#x9636;&#x6BB5;&#x5BF9;&#x5F00;&#x53D1;&#x4EBA;&#x5458;&#x6709;&#x7528;&#xFF0C;
        &#x6B63;&#x5F0F;&#x8FD0;&#x884C;&#x73AF;&#x5883;&#x5EFA;&#x8BAE;&#x5220;&#x9664;&#xFF0C;&#x5BB9;&#x6613;&#x6CC4;&#x9732;&#x654F;&#x611F;&#x4FE1;&#x606F;&#x88AB;&#x653B;&#x51FB;&#x8005;&#x5229;&#x7528;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p></p>
        <p>&#x6267;&#x884C;&#xFF1A;more /etc/security/limits.conf &#x68C0;&#x67E5;&#x662F;&#x5426;&#x5305;&#x542B;&#x4E0B;&#x5217;&#x9879;&#xFF1A;</p>
        <p>/ * soft core 0</p>
        <p>/ * hard core 0</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x82E5;&#x4E0D;&#x5B58;&#x5728;<em> *soft core 0&#xFF1B;*</em> hard core
        0&#xFF0C;&#x5219;&#x4F4E;&#x4E8E;&#x5B89;&#x5168;&#x8981;&#x6C42;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x56DE;&#x9000;&#x64CD;&#x4F5C;</td>
      <td style="text-align:left">&#x4FEE;&#x6539;&#x914D;&#x7F6E;&#x6587;&#x4EF6;
        <br />vi /etc/security/limits.conf
        <br />&#x914D;&#x7F6E;&#x5982;&#x4E0B;&#x7C7B;&#x4F3C;&#x8BED;&#x53E5;&#xFF0C;&#x6CE8;&#x91CA;&#x6389;&#x4E0B;&#x9762;&#xFF1A;
        <br
        />* soft core 0
        <br />* hard core 0</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">&#x8865;&#x5145;&#x64CD;&#x4F5C;&#x8BF4;&#x660E; core dump &#x4E2D;&#x53EF;&#x80FD;&#x5305;&#x62EC;&#x7CFB;&#x7EDF;&#x4FE1;&#x606F;&#xFF0C;&#x6613;&#x88AB;&#x5165;&#x4FB5;&#x8005;&#x5229;&#x7528;&#xFF0C;&#x5EFA;&#x8BAE;&#x5173;&#x95ED;&#x3002;</td>
    </tr>
  </tbody>
</table>

![](../../.gitbook/assets/image%20%2853%29.png)

### 2、补丁更新

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
      <td style="text-align:left">&#x5E94;&#x6839;&#x636E;&#x9700;&#x8981;&#x53CA;&#x65F6;&#x8FDB;&#x884C;&#x6309;&#x7167;&#x64CD;&#x4F5C;&#x7CFB;&#x7EDF;&#x66F4;&#x65B0;&#x8865;&#x4E01;&#xFF0C;&#x4FEE;&#x590D;&#x7CFB;&#x7EDF;&#x6F0F;&#x6D1E;&#x3002;&#x5BF9;&#x670D;&#x52A1;&#x5668;&#x7CFB;&#x7EDF;
        &#x8865;&#x4E01;&#x66F4;&#x65B0;&#xFF0C;&#x5E94;&#x5148;&#x8FDB;&#x884C;&#x517C;&#x5BB9;&#x6027;&#x6D4B;&#x8BD5;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x68C0;&#x6D4B;&#x6B65;&#x9AA4;</td>
      <td style="text-align:left">
        <p>1&#x3001;&#x67E5;&#x770B;&#x5F53;&#x524D;&#x7CFB;&#x7EDF;&#x5185;&#x6838;&#x7248;&#x672C;&#x3001;&#x8865;&#x4E01;&#x7248;&#x672C;
          <br
          /># uname -a
          <br /># cat /proc/version //&#x67E5;&#x770B;&#x5185;&#x6838;&#x7248;&#x672C;
          <br
          /># lsb_release -a
          <br /># cat /etc/Linux-release//&#x67E5;&#x770B;&#x8865;&#x4E01;&#x7248;&#x672C;</p>
        <p>2&#x3001; &#x68C0;&#x67E5;&#x5B98;&#x7F51;&#x5F53;&#x524D;&#x7CFB;&#x7EDF;&#x7248;&#x672C;&#x662F;&#x5426;&#x53D1;&#x5E03;&#x5B89;&#x5168;&#x66F4;&#x65B0;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7B26;&#x5408;&#x6027;&#x5224;&#x5B9A;&#x4F9D;&#x636E;</td>
      <td style="text-align:left">&#x7248;&#x672C;&#x5E94;&#x4FDD;&#x6301;&#x4E3A;&#x6700;&#x65B0;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5F71;&#x54CD;&#x6027;&#x5206;&#x6790;</td>
      <td style="text-align:left">&#x52A0;&#x56FA;&#x64CD;&#x4F5C;&#x5BF9;&#x4E1A;&#x52A1;&#x7684;&#x5F71;&#x54CD;&#x9700;&#x8981;&#x5728;&#x6D4B;&#x8BD5;&#x73AF;&#x5883;&#x4E2D;&#x8FDB;&#x884C;&#x6D4B;&#x8BD5;&#xFF0C;&#x6D4B;&#x8BD5;&#x7A33;&#x5B9A;&#x540E;&#x624D;&#x5728;&#x7CFB;&#x7EDF;&#x4E0A;
        &#x505A;&#x8865;&#x4E01;&#x5B89;&#x88C5;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x56DE;&#x9000;&#x64CD;&#x4F5C;</td>
      <td style="text-align:left">&#x5378;&#x8F7D;&#x76F8;&#x5173;&#x8865;&#x4E01;</td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5907;&#x6CE8;</td>
      <td style="text-align:left">cat /etc/Linux-release//&#x67E5;&#x770B;&#x8865;&#x4E01;&#x7248;&#x672C;&#x7684;&#x547D;&#x4EE4;&#x8981;&#x6839;&#x636E;&#x7CFB;&#x7EDF;&#x7684;&#x4E0D;&#x901A;&#x8FDB;&#x884C;&#x53D8;&#x5316;&#x5982;&#xFF1A;
        cat /etc/redhat-release&#x3001;cat /etc/centos-release&#x3002;LINUX &#x76F8;&#x5173;&#x7CFB;&#x7EDF;&#x8865;&#x4E01;&#x4E00;&#x822C;
        &#x5747;&#x4F1A;&#x6D89;&#x53CA;&#x5230;&#x7CFB;&#x7EDF;&#x5185;&#x6838;&#x8C03;&#x6574;&#xFF0C;&#x4E00;&#x822C;&#x662F;&#x901A;&#x8FC7;&#x5185;&#x6838;&#x7248;&#x672C;&#x5224;&#x65AD;&#xFF0C;&#x662F;&#x5426;&#x9700;&#x8981;&#x6700;&#x65B0;&#x7684;&#x5185;&#x6838;&#x7248;&#x672C;&#xFF0C;&#x5BF9;&#x4E8E;&#x8FD9;&#x79CD;&#x60C5;&#x51B5;&#xFF0C;&#x4E00;&#x822C;&#x4E0D;&#x5EFA;&#x8BAE;&#x5185;&#x6838;&#x5347;&#x7EA7;</td>
    </tr>
  </tbody>
</table>

![](../../.gitbook/assets/image%20%2844%29.png)

![](../../.gitbook/assets/image%20%2856%29.png)

