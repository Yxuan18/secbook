# 日志分析

## 1、前言

Windows 日志本质上是一个数据库，其中包括应用程序、安全、系统的操作记录。记录的事件包含9个字段：日期/时间、事件类型、用户、计算机、事件ID、来源、类别、描述、数据等。

系统信息提取脚本：[https://yunpan.360.cn/surl\_yrad892HMjZ](https://yunpan.360.cn/surl_yrad892HMjZ) （提取码：d1b6）

Windows 日志共有五种类型，所有的记录只能属于其中的一种事件类型：

| 选项 | 说明 |
| :--- | :--- |
| 
| 警告（Warning） | 警告事件指不是直接的、主要的，但是会导致将来问题发生的问题。例如，当磁盘空间不足或未找到打印机时，都会记录一个“警告”事件 |
| 错误（Error） | 错误事件指用户应该知道的重要的问题。错误事件通常指功能和数据的丢失。例如, 如果一个服务不能作为系统引导被加载，那么它会产生一个错误事件 |
| 成功审核（Success audit） | 成功的审核安全访问尝试，主要是指安全性日志，这里记录着用户登录/注销、对象访问、特权使用、账户管理、策略更改、详细跟踪、目录服务访问、账户登录等事件，例如所有的成功登录系统都会被记录为“成功审核”事件 |
| 失败审核（Failure audit） | 失败的审核安全登录尝试，例如用户试图访问网络驱动器失败，则该尝试会被作为失败审核事件记录下来 |

## 2、Windows日志位置

### 1、系统日志

记录操作系统组件产生的事件，主要包括驱动程序、系统组件和应用软件的崩溃以及数据

默认位置：

```text
## NT/Win2000/XP/Server 2003
C:\WINDOWS\system32\config\SysEvent.Evt

## Vista/Win7/Win8//Win10/Server 2008/Server 2012
C:\WINDOWS\system32\winevt\Logs\System.evtx
```

### 2、应用程序日志

包含由应用程序或系统程序记录的事件，主要记录程序运行方面的事件

默认位置：

```text
## NT/Win2000/XP/Server 2003
C:\WINDOWS\system32\config\AppEvent.Evt

## Vista/Win7/Win8//Win10/Server 2008/Server 2012
C:\WINDOWS\system32\winevt\Logs\Application.evtx
```

### 3、安全日志

记录系统的安全审计事件，包含各种类型的登录日志、对象访问日志、进程追踪日志、特权使用、帐号管理、策略变更、系统事件。安全日志也是调查取证中最常用到的日志

默认位置：

```text
## NT/Win2000/XP/Server 2003
C:\WINDOWS\system32\config\SecEvent.Evt

## Vista/Win7/Win8//Win10/Server 2008/Server 2012
C:\WINDOWS\system32\winevt\Logs\Security.evtx
```

### 4、powershell日志

事件查看器 -&gt; 应用程序和服务日志 -&gt; Microsoft -&gt; Windows -&gt; PowerShell -&gt; Operationnal

 系统和应用程序日志存储着故障排除信息，对于系统管理员更为有用。 安全日志记录着审计信息，包括用户验证（登录、远程访问等）和特定用户在认证后对系统做了什么，对于寻找攻击链更有帮助

## 3、EventID

对于 Windows 日志分析，我们可以根据自己的需要，根据事件ID迅速找出我们关心的信息。不同的事件ID代表不同的意义，具体信息可以参考微软官方文档

| EVENT ID
| :--- | :--- | :--- | :--- |
| 528 | 4624 | 成功登录 | Security |
| 529 | 4625 | 失败登录 |  Security |
| 680 | 4776 | 成功/失败的账户认证 | Security |
| 624 | 4720 | 创建用户 | Security |
| 636 | 4732 | 添加用户到启用安全性的本地组中  | Security |
| 632 | 4728 | 添加用户到启用安全性的全局组中 | Security |
| 2934 | 7030 | 服务创建错误 | System |
| 2944 | 7040 | IPSEC服务服务的启动类型已从禁用更改为自动启动 | System |
| 2949 | 7045 | 服务创建 | System |

#### 登录类型事件

成功/失败登录事件提供的有用信息之一是用户/进程尝试登录（登录类型），但 Windows 将此信息显示为数字，下面是数字和对应的说明

| 登录ID | 登录类型 | 描述 |
| :--- | :--- | :--- |
| 2 | Interactive | 用户登录到本机 |
| 3 | Network | 用户或计算手机从网络登录到本机，如果网络共享，或使用 net use 访问网络共享，net view 查看网络共享 |
| 4 | Batch | 批处理登录类型，无需用户干预 |
| 5 | Service | 服务控制管理器登录 |
| 7 | Unlock | 用户解锁主机 |
| 8 | NetworkCleartext | 用户从网络登录到此计算机，用户密码用非哈希的形式传递 |
| 9 | NewCredentials | 进程或线程克隆了其当前令牌，但为出站连接指定了新凭据 |
| 10 | Remotelnteractive | 使用终端服务或远程桌面连接登录 |
| 11 | Cachedlnteractive | 用户使用本地存储在计算机上的凭据登录到计算机（域控制器可能无法验证凭据），如主机不能连接域控，以前使用域账户登录过这台主机，再登录就会产生这样日志 |
| 12 | CachedRemotelnteractive | 与 Remotelnteractive 相同，内部用于审计目的 |
| 13 | CachedUnlock | 登录尝试解锁 |

123

## 4、远程登录事件

### 1、RDP

攻击者使用 RDP 远程登录受害者计算机，源主机和目的主机都会生成相应事件。 

重要的事件 ID（Windows日志 → 安全日志），必须要检查。

```text
4624：账户成功登录
4648：使用明文凭证尝试登录
4778：重新连接到一台 Windows 主机的会话
4779：断开到一台 Windows 主机的会话
```

远程连接日志（应用程序和服务日志 → Microsoft → Windows → TerminalServices → RemoteConnectionManager → Operational）的 ID ：

```text
1149：用户认证成功
21：远程桌面服务：会话登录成功
24：远程桌面服务：会话已断开连接
25：远程桌面服务：会话重新连接成功
```

远程连接日志关注 RemoteInteractive（10） 和CachedRemoteInteractive（12）表明使用了 RDP ，因为这些登录类型专用于RDP使用。

### 2、计划任务和AT

计划任务事件（应用程序和服务日志 → Microsoft → Windows → TaskScheduler → Operational），该日志默认是禁用的。

```text
100：任务已开始
102：任务完成
106：已注册任务（关注点）
107：在调度程序上触发任务
110：用户触发的任务
129：创建任务流程（推出）
140：任务已更新
141：任务已删除
200：运行计划任务
325：启动请求排队
```

### 3、案例

PSExec是系统管理员的远程命令执行工具，包含在“Sysinternals Suite”工具中，但它通常也用于内网渗透中的横向移动。 

PsExec的典型行为

* 在具有网络登录（类型3）的远程计算机上将 PsExec 服务执行文件（默认值：PSEXESVC.exe）复制到％SystemRoot％。 
* 如果使用-c选项，则通过 $Admin 共享将文件复制到 ％SystemRoot％ 执行命令。 
* 注册服务（默认值：PSEXESVC），并启动服务以在远程计算机上执行该命令。
* 停止服务（默认值：PSEXESVC），并在执行后删除远程计算机上的服务。

PSExec选项的重要选项：

```text
-r 
# 更改复制的文件名和远程计算机的服务名称（默认值：％SystemRoot％\ PSEXESVC.exe和PSEXESVC）

-s 
#	由SYSTEM帐户执行。

-C 
#	将程序复制到远程计算机
#	被复制到Admin$（％SystemRoot％）

-u 
#	使用特定凭据登录到远程计算机
#	生成登录类型2和登录类型3 的事件
可以从System.evtx中查找事件 ID 7045 发现 PSExec，相关的事件 ID

Security.evtx 
#	4624：帐户已成功登录

Ssystem.evtx 
#	7045：系统中安装了服务
PsExec在执行命令时在远程主机上创建服务，默认服务名称为PSEXESVC，配合检测系统 7045 事件可以确定。
如果使用-r参数更改了默认的服务名称，通过以下特征可以检测 PSExec 的执行：

PSExec服务执行文件（默认值：PSEXESVC.exe）被复制到远程计算机上的“％SystemRoot％”目录中

服务名称与没有“.exe”扩展名的执行名称相同

服务以“用户模式”执行，而不是“内核模式”

“LocalSystem”帐户用于服务帐户

实际帐户用于执行服务执行文件，而不是“SYSTEM”
```

## 5、小知识点

### 1、账户类型

1. 用户账户
2. 计算机账户：此帐户类型表示每个主机。 此帐户类型的名称以字符“$”结尾。 例如，“DESKTOP-SHCTJ7L $”是计算机帐户的名称。
3. 服务账户：每个服务帐户都创建为特定服务的所有者。 例如，IUSR是IIS的所有者，而krbtgt是作为密钥分发中心一部分的服务的所有者

### 2、日志被删除或日志服务被关闭

攻击者入侵系统后，很可能会删除日志，比较粗暴的手法是直接删除所有日志和停止日志服务。如果只是删除某一段时间内的日志，说明攻击者在这个时间段内进行了恶意行为，应该重点排查这段时间内的其他日志以及创建、修改过的文件

### 3、系统存在大量日志

搜索关键事件的ID，先找出危险动作，再排查危险动作的上下文日志
