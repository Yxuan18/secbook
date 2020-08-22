# powershell

## 1、背景介绍

VBScript的缺陷：  
微软没有全心全意地对其提供支持，微软需要确保各种对象都可以通过VBScript访问、调用，而如果开发人员因为时间的原因或者是忘记这块知识，那么你就只能卡在那儿了

powershell优势：  
如果需要自动化一个重复性的任务或者完成在GUI中不支持的工作，那么你可以使用该Shell来达成所愿

适用PowerShell的人群： 

* 主要使用命令行以及采用第三方开发工具的管理员
* 能将命令行和工具集成为一个更复杂的工具（之后那些缺乏经验的成员可以立即使用该工具完成相关工作）的管理员
*  开发可重复使用的工具或者程序的管理员或者开发人员。

查看powershell版本：

![](../../.gitbook/assets/image%20%28492%29.png)

win7下载powershell页面：  
https://www.microsoft.com/en-us/download/confirmation.aspx?id=54616

PowerShell包含两个应用程序组件：基于文本的标准控制台（PowerShell.exe）和集成了命令行环境的图形化界面（ISE；PowerShell\_ISE.exe）

## 2、初识powershell

在计算机找到powershell的两种方式：

1. 附件→powershell
2. 开始菜单：搜索powershell

![&#x5BFB;&#x627E;powershell](../../.gitbook/assets/image%20%28496%29.png)

powershell的两种界面：

![&#x5DE6;&#xFF1A;powershell&#xFF0C;&#x53F3;&#xFF1A;ISE](../../.gitbook/assets/image%20%28503%29.png)

ISE优缺点如下：

| 优点 | 缺点 |
| :--- | :--- |
| ISE界面友好且支持双字节字符集 | ISE要求Windows Presentation Foundation（WPF），意味着不能在没有安装GUI的服务器上运行ISE |
| 在后续章节可以看到ISE能在你创建PowerShell命令和脚本时提供更多的帮助  | 启动和运行需要较长时间，但是这通常只是几秒的差异 |
| ISE使用标准的复制、粘贴按键 | 在PowerShell 5.0之前版本的ISE不支持转录 |

![&#x7684;3&#x4E2A;&#x4E3B;&#x8981;&#x533A;&#x57DF;&#x53CA;&#x63A7;&#x5236;&#x5B83;&#x4EEC;&#x7684;&#x5DE5;&#x5177;&#x680F;](../../.gitbook/assets/image%20%28490%29.png)

关于命令行补全：

输入“Get-S ”，然后按几下Tab键，再按Shift+Tab组合键。PowerShell会循环地显示以Get-S开头的所有命令。然后不停按Shift+Tab组合键，直到出现你期望的命令为止。 

![](../../.gitbook/assets/image%20%28493%29.png)

输入“Dir ”，按空格键，然后输入“C: ”，再按Tab键，PowerShell会从当前文件夹开始循环遍历所有可用的文件和文件夹名。 

![](../../.gitbook/assets/image%20%28500%29.png)

输入“Set-Execu ”，按Tab键，然后输入一个空格和横杠（- ），再开始按Tab键，可以看到PowerShell循环显示当前命令的所有可用参数。另外，也可以输入参数名的一部分，例如-E ，然后按Tab键，开始循环匹配的参数名。按Esc键可以清空命令行。 

![](../../.gitbook/assets/image%20%28494%29.png)

再次输入“Set-Execu ”，按Tab键，再按空格键，然后输入“-E ”，再次按Tab键，然后按一次空格键，再按Tab键。PowerShell会循环显示关于这些参数的合法值。这个功能仅对那些已经预设了可用值（称为枚举）的参数有效。按Esc键同样可以清空命令行。

![](../../.gitbook/assets/image%20%28497%29.png)

常见误区：

* 在控制台应用程序中的水平滚动条——从多年的教学经验中我们得知，正如前面提到过的，配置控制台的窗口，使其不出现水平滚动条非常重要。 
* 32位VS64位——建议你使用64位的Windows并使用64位的PowerShell应用程序（没有出现“x86”字样的应用程序）。虽然对于某些人来说，购买64位的计算机和64位的Windows可能是件大事，但是如果你希望PowerShell高效运行，那么这些投资还是必需的。虽然在本书中我们尽可能覆盖32位环境，但是这些内容在64位的生产环境上将带来很大的差异。 
* 确保PowerShell应用程序的窗体标题显示“管理员”——如果你发现打开的窗体上没有“管理员”字样，关闭窗体并右键单击PowerShell图标，选择“以管理员身份运行”。在生产环境中，不一定总是要这样。本书后面将演示如何使用特定的凭据运行命令。但是通常情况下，为了避免运行时出现一些问题，最好确保以管理员身份运行PowerShell

查看当前版本：

```text
$PSVersionTable
```

![](../../.gitbook/assets/image%20%28492%29.png)

## 3、使用帮助系统

帮助系统的作用：

1. 可以帮助你找到这个命令，而无需使用Google或者Bing
2. 如果你在运行一个命令的时候返回错误信息，帮助系统就可以告诉你如何正确运行命令，因此不再出现错误。 
3. 如果你想将多个命令组合在一起来执行一项复杂的任务，帮助系统就可以帮你找到哪些命令是可以和其他命令结合使用。你不需要在Google或者Bing搜索示例，只需要学习如何使用他们，以便你可以创建出自己的示例和解决方案

```text
help GET-SERVICE
```

![&#x67E5;&#x770B;&#x4E00;&#x4E2A;&#x547D;&#x4EE4;&#x7684;&#x5E2E;&#x52A9;](../../.gitbook/assets/image%20%28498%29.png)

更新帮助文档：

```text
update-help    #更新PowerShell的帮助文档,建议每个月更新一次
```

离线更新方法如下：

先找到一台可以上网的机器，并使用Save-Help 命令把帮助文档下载一份到本地。然后把它放到一个文件服务器或者其他你可以访问的网络中。接着通过在Update-Help 加上-SourcePath 参数指向刚刚下载的那份帮助文档的地址

![](../../.gitbook/assets/image%20%28502%29.png)

使用帮助系统查找命令：

```text
help *命令*
```

![](../../.gitbook/assets/image%20%28489%29.png)

也可以通过命令补全的方式查找命令，这种方式比较快捷，如图：

![](../../.gitbook/assets/image%20%28488%29.png)

345

## 4、运行命令

## 5、使用提供程序

## 6、管道：连接命令

## 7、扩展命令

## 8、对象：数据的另一个名称

## 9、深入理解管道

## 10、11、12、13、14、15、



