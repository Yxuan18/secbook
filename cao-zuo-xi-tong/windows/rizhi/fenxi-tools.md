# 分析工具

## 1、LogParser

LogParser是微软自家做的工具，可以用来分析IIS日志、Windows系统日志等一些类型的日志，这里主要研究它对Windows系统日志的分析，它使用类似SQL的语法对Windows日志进行查询。最后可以将查询、筛选过的日志结果导出为CSV, XML, DATAGRID等一些格式

### 1、下载

LogParser 

这是本节的主角，下载安装后建议将路径加入PATH变量，方便在CMD中调用。 

下载：

 [https://www.microsoft.com/en-us/download/details.aspx?displaylang=en&id=24659](https://www.microsoft.com/en-us/download/details.aspx?displaylang=en&id=24659) LogParserStudio 这是一款界面化的辅助分析工具，可以在练习时提高语句测试的效率，解压后就可以使用（需要.NET框架），该工具依赖LogParser。 

下载： 

[https://gallery.technet.microsoft.com/Log-Parser-Studio-cd458765](https://gallery.technet.microsoft.com/Log-Parser-Studio-cd458765) 

使用方法简单介绍：

![](../../../.gitbook/assets/image%20%28411%29.png)

### 2、理解字段

先收集所有字段名，并理解每个字段的含义 

```text
logparser -i:evt -o:csv "select * into D:\1.csv from D:\Security.evtx where eventid=4624" 
```

下表为提取的字段名与示例：两个表为1个表，上下顺序为从左往右，包括下面的MESSAGE部分。

| EventLog | RecordNumber | TimeGenerated | TimeWritten | EventID | EventType | EventTypeName | EventCategory |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| D:\Security.evtx | 2 | 2014/2/15 13:09:00 | 2014/2/15 13:09:00 | 4624 | 8 | Success Audit event | 12544 |

| EventCategoryName | SourceName | Strings | ComputerName | SID |
| :--- | :--- | :--- | :--- | :--- |
| The name for category 12544 in Source "Microsoft-Windows-Security-Auditing" cannot be found. The local computer may not have the necessary registry information or message DLL files to display messages from a remote computer | Microsoft-Windows-Security-Auditing | S-1-0-0\|-\|-\|0x0\|S-1-5-18\|SYSTEM\|NT AUTHORITY\|0x3e7\|0\|-\|-\|-\|{00000000-0000-0000-0000-000000000000}\|-\|-\|0\|0x4\|\|-\|- | 37L425-07 | NULL |

MESSAGE：

```text
已成功登录帐户。 
主题: 
安全 ID: S-1-0-0 
帐户名: 
- 帐户域: 
- 登录 ID: 0x0 
登录类型: 0 
新登录: 
安全 ID: S-1-5-18 
帐户名: SYSTEM 
帐户域: NT AUTHORITY 
登录 ID: 0x3e7 
登录 GUID: {00000000-0000-0000-0000-000000000000} 
进程信息: 
进程 ID: 0x4 
进程名: 
网络信息: 
工作站名: - 
源网络地址: - 
源端口: - 
详细身份验证信息: 
登录进程: - 
身份验证数据包: - 
传递服务: - 
数据包名(仅限 NTLM): - 
密钥长度: 0 
在创建登录会话后在被访问的计算机上生成此事件。 
“主题”字段指明本地系统上请求登录的帐户。
这通常是一个服务(例如 Server 服务)或本地进程(例如 Winlogon.exe 或 Services.exe)。 
“登录类型”字段指明发生的登录种类。
最常见的类型是 2 (交互式)和 3 (网络)。 
“新登录”字段会指明新登录是为哪个帐户创建的，即登录的帐户。 
“网络”字段指明远程登录请求来自哪里。
“工作站名”并非总是可用，而且在某些情况下可能会留为空白。 
“身份验证信息”字段提供关于此特定登录请求的详细信息。 
-“登录 GUID”是可以用于将此事件与一个 KDC 事件关联起来的唯一标识符。 
-“传递服务”指明哪些直接服务参与了此登录请求。 
- “数据包名”指明在 NTLM 协议之间使用了哪些子协议。 
-“密钥长度”指明生成的会话密钥的长度。如果没有请求会话密钥则此字段为 0。
```

对以上每个字段进行理解

| 字段名 | 解释 |
| :--- | :--- |
| EventLog | Log文件名 |
| RecordNumber | 在日志文件中的序号 |
| TimeGenerated | 记录时间 |
| TimeWritten | 记录时间 |
| EventID | 事件ID |
| EventType | 事件类型ID |
| EventTypeName | 事件类型名称 |
| EventCategory | 事件分类ID |
| EventCategoryName | 事件分类名称 |
| SourceName | 日志名称如：Microsoft-Windows-Security-Auditing |
| Strings | 包含“详细信息”中的字串，使用\|分割每个字段 |
| ComputerName | 本机计算机名 |
| SID | NULL |
| Message | 包含“常规”中的字符，其中会有中文字符 |
| Data | NULL |

![](../../../.gitbook/assets/image%20%28398%29.png)

![](../../../.gitbook/assets/image%20%28375%29.png)

### 3、使用

下面不写logparser参数，重点放在查询语句上。 筛选条件的内容注意区分大小写，比如：

```text
where computername = 'WIN-AOFVGI63GUG.dclab.com' 
# 其中 WIN-AOFVGI63GUG.dclab.com 必须大小写书写正确。 
```

下面是围绕Security日志进行的测试，多数没有注明的情况下，都使用eventid 4624进行测试，请悉知

#### 1、按照事件ID筛选

和SQL查询一样，使用WHERE语法来限制 

```text
# where eventid=事件id 
select * from D:\Security-2008.evtx where eventid=4624 
# eventid=4624只显示成功登录的事件
```

![](../../../.gitbook/assets/image%20%28373%29.png)

#### 2、只查询指定字段

只查询strings字段

```text
 select strings from D:\Security-2008.evtx where eventid=4624
```

![](../../../.gitbook/assets/image%20%28396%29.png)

多字段查询也是用逗号隔开

```text
select eventid,strings from D:\Security-2008.evtx where eventid=4624
```

![](../../../.gitbook/assets/image%20%28422%29.png)

#### 3、限制查询数量

TOP 10 只显示前10条日志，这个TOP语法和Access语法一样

```text
 select top 10 strings from D:\Security-2008.evtx where eventid=4624
```

![](../../../.gitbook/assets/image%20%28386%29.png)

#### 4、切割strings

Strings里包含“详细信息”中的字串，使用\|分隔，包含一些重要的数据。 使用extract\_token函数可以切割字符串 \# EXTRACT\_TOKEN \( 字符串, 索引 , '分隔字符' \)

 \# 这是事件ID为4624的情况下

“详细信息”参考： 

```text
SubjectUserSid S-1-0-0 
SubjectUserName - 
SubjectDomainName - 
SubjectLogonId 0x0 
TargetUserSid S-1-5-7 
TargetUserName ANONYMOUS LOGON 
TargetDomainName NT AUTHORITY 
TargetLogonId 0x14cc11d 
LogonType 3 
LogonProcessName NtLmSsp 
AuthenticationPackageName NTLM 
WorkstationName 7LAB-PC 
LogonGuid {00000000-0000-0000-0000-000000000000} 
TransmittedServices - 
LmPackageName NTLM V1 
KeyLength 128 ProcessId 0x0 
ProcessName - 
IpAddress 172.16.175.102 
IpPort 49229
```

例子：截取用户名 

```text
select Extract_token(Strings,5,'|') from D:\Security-2008.evtx where eventid=4624 
```

索引5为 用户名，8为登录类型，11为来源主机名，18为来源主机IP，19为来源主机端口

![](../../../.gitbook/assets/image%20%28414%29.png)

 \# 这是事件ID为4625的情况下

“详细信息”参考： 

```text
SubjectUserSid S-1-0-0 
SubjectUserName - 
SubjectDomainName - 
SubjectLogonId 0x0 
TargetUserSid S-1-0-0 
TargetUserName administrator 
TargetDomainName localhost 
Status 0xc000006d 
FailureReason %%2313 
SubStatus 0xc000006a 
LogonType 3 
LogonProcessName NtLmSsp 
AuthenticationPackageName NTLM
WorkstationName HP-PC 
TransmittedServices - 
LmPackageName - 
KeyLength 0 
ProcessId 0x0 
ProcessName - 
IpAddress 192.16.0.94 
IpPort 57817
```

例子：截取用户名  

```text
select Extract_token(Strings,5,'|') from D:\Security-2008.evtx where eventid=4625
```

索引5为 用户名，8为登录失败原因（%%2313=未知用户名或密码错误），10为登录类型，13为来源主机名，19为来源主机IP，20为来源主机端口

![](../../../.gitbook/assets/image%20%28372%29.png)

以下是从网络上搜集的常见失败原因：

| 错误码 | 失败原因 |
| :--- | :--- |
| %%2305 | The specified user account has expired. \(532\) |
| %%2309 | The specified account's password has expired. \(535\) |
| %%2310 | Account currently disabled. \(531\) |
| %%2311 | Account logon time restriction violation. \(530\) |
| %%2312 | User not allowed to logon at this computer. \(533\) |
| %%2313 | Unknown user name or bad password. \(529\) |

#### 5、导出文件

INTO语法导出必须使用LogParser，LPS需要用菜单选择导出。

```text
 logparser -i:evt -o:csv "select Extract_token(Strings,11,'|') into d:\11.csv from D:\Security-2008.evtx where eventid=4624"
```

![](../../../.gitbook/assets/image%20%28376%29.png)

#### 6、只输出有主机名的记录

```text
select Extract_token(Strings,11,'|'),* from D:\Security-2008.evtx where eventid=4624 and Extract_token(Strings,11,'|') not in (NULL;'';'-')
```

![](../../../.gitbook/assets/image%20%28395%29.png)

#### 7、只显示登录类型3的数据

```text
select * from D:\Security-2008.evtx where Extract_token(Strings,8,'|')='3'
```

![](../../../.gitbook/assets/image%20%28383%29.png)

#### 8、只显示指定主机名的数据

主机名保持大写 

```text
select * from D:\Security-2008.evtx where eventid=4624 and Extract_token(Strings,11,'|')='NEW-666'
```

![](../../../.gitbook/assets/image%20%28401%29.png)

#### 9、将日志整理成新的表

把strings中包含的需要的字段进行分割，使用as指定字段别名。

```text
# EventID 4624

select EventLog,TimeGenerated,EventID, 
Extract_token(Strings,8,'|') as LogonType, 
Extract_token(Strings,5,'|') as UserName, 
Extract_token(Strings,11,'|') as HostName, 
Extract_token(Strings,18,'|') as SourceIP, 
Extract_token(Strings,19,'|') as SourcePort 
from D:\Security-2008.evtx where eventid=4624 and 
Extract_token(Strings,11,'|') not in (NULL;'';'-') and 
Extract_token(Strings,5,'|') <> 'ANONYMOUS LOGON'
```

![](../../../.gitbook/assets/image%20%28428%29.png)

```text
# EventID 4625

select EventLog,TimeGenerated,EventID, 
Extract_token(Strings,10,'|') as LogonType, 
Extract_token(Strings,5,'|') as UserName, 
Extract_token(Strings,13,'|') as HostName, 
Extract_token(Strings,19,'|') as SourceIP, 
Extract_token(Strings,20,'|') as SourcePort, 
Extract_token(Strings,8,'|') as FailReason 
from D:\Security.evtx where eventid=4625
```

![](../../../.gitbook/assets/image%20%28378%29.png)

#### 10、根据IP，主机名，用户名等进行排序

使用Order by 语法进行排序 

```text
select EventLog,TimeGenerated,EventID, 
Extract_token(Strings,8,'|') as LogonType, 
Extract_token(Strings,5,'|') as UserName, 
Extract_token(Strings,11,'|') as HostName, 
Extract_token(Strings,18,'|') as SourceIP, 
Extract_token(Strings,19,'|') as SourcePort 
from D:\Security-2008.evtx where eventid=4624 and 
Extract_token(Strings,11,'|') not in (NULL;'';'-') and 
Extract_token(Strings,5,'|') <> 'ANONYMOUS LOGON' 
Order by 
SourceIP DESC
```

![](../../../.gitbook/assets/image%20%28377%29.png)

#### 11、筛选登录失败最多的IP，主机名

筛选出EventID4625结果中每个IP出现的次数 

```text
select Extract_token(Strings,19,'|') as SourceIP, 
Count(SourceIP) as CountIP 
from D:\Security-2008.evtx where eventid=4625 
group by SourceIP 
order by 
CountIP DESC
```

![](../../../.gitbook/assets/image%20%28370%29.png)

筛选出EventID4625结果中每个主机名出现的次数 

```text
select Extract_token(Strings,13,'|') as HostName, 
Count(HostName) as CountHost 
from D:\Security-2008.evtx where eventid=4625 
group by HostName 
order by CountHost DESC
```

![](../../../.gitbook/assets/image%20%28382%29.png)

这样分析不是很方便，推荐整理好字段，直接导出完整的csv文件，放在Excel中分析，整理，或转给其它工具进行分析。 

```text
logparser -i:evt -o:csv 
"select 
EventLog,TimeGenerated,EventID,Extract_token(Strings,10,'|') as 
LogonType,Extract_token(Strings,5,'|') as 
UserName,Extract_token(Strings,13,'|') as 
HostName,Extract_token(Strings,19,'|') as 
SourceIP,Extract_token(Strings,20,'|') as 
SourcePort,Extract_token(Strings,8,'|') as 
FailReason into D:\all.csv 
from D:\Security.evtx where eventid=4625"
```

![](../../../.gitbook/assets/image%20%28429%29.png)

![](../../../.gitbook/assets/image%20%28409%29.png)

![](../../../.gitbook/assets/image%20%28389%29.png)

#### 12、计算文件HASH

说个不太相关的，logparser也能计算文件hash。 

```text
logparser -i:FS "select path,hashmd5_file(path) from d:\22.csv"
```

![](../../../.gitbook/assets/image%20%28397%29.png)

## 2、wevtutil

### 1、构造查询语句的方法

有人说LogParser查询命令太复杂，个人觉得使用XPath不太顺手，反而更喜欢LogParser。 查询语句的构造可以参考这个：

```text
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
    <System>
        <Provider Name="Microsoft-Windows-Security-Auditing" Guid="{54849625-5478-4994-A5BA-3E3B0328C30D}" />
        <EventID>4624</EventID>
        <Version>0</Version>
        <Level>0</Level>
        <Task>12544</Task>
        <Opcode>0</Opcode>
        <Keywords>0x8020000000000000</Keywords>
        <TimeCreated SystemTime="2020-03-10T09:07:15.828987300Z" />
        <EventRecordID>11438</EventRecordID>
        <Correlation />
        <Execution ProcessID="488" ThreadID="596" />
        <Channel>Security</Channel>
        <Computer>WIN-AOFVGI63GUG.dclab.com</Computer>
        <Security />
    </System>
    <EventData>
        <Data Name="SubjectUserSid">S-1-0-0</Data>
        <Data Name="SubjectUserName">-</Data>
        <Data Name="SubjectDomainName">-</Data>
        <Data Name="SubjectLogonId">0x0</Data>
        <Data Name="TargetUserSid">S-1-5-21-2474629087-4018274074-3672862124-1106</Data>
        <Data Name="TargetUserName">laowang</Data>
        <Data Name="TargetDomainName">DCLAB</Data>
        <Data Name="TargetLogonId">0xa72d89</Data>
        <Data Name="LogonType">3</Data>
        <Data Name="LogonProcessName">NtLmSsp </Data>
        <Data Name="AuthenticationPackageName">NTLM</Data>
        <Data Name="WorkstationName"></Data>
        <Data Name="LogonGuid">{00000000-0000-0000-0000-000000000000}</Data>
        <Data Name="TransmittedServices">-</Data>
        <Data Name="LmPackageName">NTLM V2</Data>
        <Data Name="KeyLength">128</Data>
        <Data Name="ProcessId">0x0</Data>
        <Data Name="ProcessName">-</Data>
        <Data Name="IpAddress">172.16.175.99</Data>
        <Data Name="IpPort">45640</Data>
    </EventData>
</Event>
```

比如要指定登录的用户名（TargetUserName），可以这样构造语句

```text
wevtutil qe D:\Security-2008.evtx /lf /q:"*[System[(EventID=4624)] and EventData[(Data[@Name='TargetUserName']='laowang')]]" /c:1
```

同时指定用户名与来源IP地址

```text
wevtutil qe D:\Security-2008.evtx /lf /q:"*[System[(EventID=4624)] and EventData[(Data[@Name='TargetUserName']='laowang') and (Data[@Name='IpAddress']='172.16.175.99')]]" /c:1
```

### 2、用脚本导出CSV

下面使用PS脚本配合wevtutil进行日志导出，修改查询语句必须要更改代码。本想脚本修改为参数传递查询语句，无奈查询语句过于复杂导致参数传递出错。（修改位置有注释）

```text
# 脚本 -f 指定evtx文件位置，会自动导出到同名csv文件，默认寻找当前目录下的Security.evtx进行处理。
Param (
    [string]$f = $pwd.Path+"\Security.evtx"
)

$time=Get-Date -Format h:mm:ss
$evtx=(Get-Item $f).fullname
$outfile=(Get-Item $evtx).BaseName+".csv"

$logsize=[int]((Get-Item $evtx).length/1MB)

write-host [+] $time Load $evtx "("Size: $logsize MB")" ... -ForegroundColor Green
# 这里修改WEVTUtil查询语句
$q = "*[System[Provider[@Name='Microsoft-Windows-Security-Auditing']  and (EventID=4624 or EventID=4625)] and EventData[Data[@Name='LogonType']='3']]"
[xml]$xmldoc=WEVTUtil qe  $evtx /q:$q /e:root /f:Xml  /lf

$xmlEvent=$xmldoc.root.Event

function OneEventToDict {
    Param (
        $event
    )
    $ret = @{
        "SystemTime" = $event.System.TimeCreated.SystemTime | Convert-DateTimeFormat -OutputFormat 'yyyy"/"MM"/"dd HH:mm:ss';
        "EventID" = $event.System.EventID
    }
    $data=$event.EventData.Data
    for ($i=0; $i -lt $data.Count; $i++){
        $ret.Add($data[$i].name, $data[$i].'#text')
    }
    return $ret
}

filter Convert-DateTimeFormat
{
  Param($OutputFormat='yyyy-MM-dd HH:mm:ss fff')
  try {
    ([DateTime]$_).ToString($OutputFormat)
  } catch {}
}

$time=Get-Date -Format h:mm:ss
write-host [+] $time Extract XML ... -ForegroundColor Green
[System.Collections.ArrayList]$results = New-Object System.Collections.ArrayList($null)
for ($i=0; $i -lt $xmlEvent.Count; $i++){
    $event = $xmlEvent[$i]
    $datas = OneEventToDict $event

    $results.Add((New-Object PSObject -Property $datas))|out-null
}

$time=Get-Date -Format h:mm:ss
write-host [+] $time Dump into CSV: $outfile ... -ForegroundColor Green
# 这里设置要导出的字段、顺序
$results | Select-Object 
SystemTime,WorkstationName,IpAddress,TargetDomainName,TargetUserName,EventID,LogonType | Export-Csv $outfile -NoTypeInformation -UseCulture  -Encoding Default -Force
```

使用脚本导出csv

```text
powershell -exec bypass ./getLog.ps1 -f 'd:\Security-2008.evtx'
```

![](../../../.gitbook/assets/image%20%28404%29.png)

然后就可以使用excel分析了

![](../../../.gitbook/assets/image%20%28391%29.png)

## 3、LoogonTracer

LogonTracer是一个登录日志可视化工具，可用于梳理攻击路径本会有用。对本文在编写过程中曾搭建过它，测试了该工具三个版本都无法正常使用。主要呈现的问题是上传日志后，没有分析结果、报错。在上面浪费了几个小时，不建议尝试使用该工具。

这是测试过的三个版本，一个原版，Docker原版，还有一个汉化版。

https://github.com/JPCERTCC/LogonTracer   
https://github.com/TheKingOfDuck/logonTracer

