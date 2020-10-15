# PoC：滥用PowerShell Core

滥用PowerShell的威胁还在不断增长，攻击者在不断寻找新的方式滥用PowerShell来传播银行木马、后门、勒索软件和加密货币挖矿恶意软件，以及最近的无文件恶意软件和恶意WMI记录。

事实上，PowerShell的灵活性和强大的功能使其成为网络犯罪分子的潜在工具。比如，PowerShell可以被滥用来在被入侵的网络中“内网漫游”\(lateral movement\)和保持驻留。

2018年1月，微软发布PowerShell Core，PowerShell Core是为异类环境和混合云而构建的跨平台\(Windows、macOS 和 Linux\)且开源的新版 PowerShell。因为PowerShell Core可能会和PowerShell一样被滥用用于攻击其他操作系统平台。

研究人员列举了一些攻击者可能滥用PowerShell Core的策略。这些POC可以帮助更好的理解、检测和预防潜在的威胁。研究人员利用PowerShell Core开发的PoC可以在Windows、Linux和mac OS上运行。研究人员使用的攻击者技术大多是来自于之前基于PowerShell的功能。

但是要想成功滥用PowerShell Core，还需要注意一些事项。比如，PowerShell Core并非在所有平台都是默认安装的，也就是说恶意软件创建者不能轻易的入侵没有安装PowerShell Core的机器。对unix和类Unix系统，文件系统的差异需要黑客添加一或两行代码才能执行恶意软件。比如，下载的ELF文件并不是可执行文件，要执行的话就需要对其属性进行修改。PowerShell Core的功能还在开发中，因为其应对滥用的安全机制也就比较弱。

PowerShell Core可以安装在Windows 7之后的Windows操作系统，Linux \(Kali, Fedora 27/28, Ubuntu等\)，macOS 10.12+ \(Sierra及之后的操作系统\)，Windows IoT和Nano服务器。

## **滥用PowerShell Core PoC**

### **Windows: DownloadFile场景**

感染链是从一封垃圾邮件中的恶意附件开始的。在开启了文件的宏内容后，一个命令行和PowerShell Core脚本就会运行来下载和执行payload。

![PoC&#xFF1A;&#x6EE5;&#x7528;PowerShell Core](https://img.4hou.com/wp-content/uploads/2018/12/5836b2f2230e0a8d7eb0.jpg)

图1. 在Windows操作系统中滥用PowerShell Core DownloadFile的感染链

![PoC&#xFF1A;&#x6EE5;&#x7528;PowerShell Core](https://img.4hou.com/wp-content/uploads/2018/12/166c4902585bc2fa508a.png)

图2 用DownloadFile来提取payload的样本PowerShell Core脚本

### **Linux and mac OS: DownloadString场景**

因为Linux和macOS terminal是一样的，因此滥用PowerShell Core的威胁在这两个操作系统是上都可以运行。假设恶意软件是通过垃圾邮件或漏洞利用到达的，可以创建一个PowerShell脚本来完成final payload的下载和执行。通过terminal执行的初始脚本会下载另一个可以提取final payload的PowerShell Core脚本，并修改其属性为executable。这就是为什么通过terminal或PowerShell Core脚本下载的文件在Linux和macOS系统中默认是不可执行的。

![PoC&#xFF1A;&#x6EE5;&#x7528;PowerShell Core](https://img.4hou.com/wp-content/uploads/2018/12/838f70c228c595b47bc2.jpg)

图3. 在Linux和macOS中滥用PowerShell Core DownloadString的感染链

![PoC&#xFF1A;&#x6EE5;&#x7528;PowerShell Core](https://img.4hou.com/wp-content/uploads/2018/12/779f1a4c278f3632d990.png)

图4. 使用DownloadString来提取payload的样本PowerShell Core脚本

![PoC&#xFF1A;&#x6EE5;&#x7528;PowerShell Core](https://img.4hou.com/wp-content/uploads/2018/12/4ec1aecb3d2759d91ff8.png)

图 5 下载的用来提取和执行final payload的PowerShell脚本

### **Windows: 滥用MultiPoolMiner和PowerShell Core**

PowerShell Core可以与其他合法或灰色工具一起被滥用。对加密货币挖矿相关的活动，研究人员测试了开源工具MultiPoolMiner，该工具需要PowerShell Core才能正常运行。

研究人员在复现过程中，首先检查MultiPoolMiner包的start.bat文件，其中包括默认的命令和参数。进一步分析发现它会通过PowerShell Core（pwsh）来执行命令。如果系统中没有安装，就下载和安装PowerShell Core，然后继续执行。

![PoC&#xFF1A;&#x6EE5;&#x7528;PowerShell Core](https://img.4hou.com/wp-content/uploads/2018/12/11a35c9538c1a6a000a4.png)

图6. MultiPoolMiner的Start.bat会下载和执行PowerShell Core

在执行了batch文件后，开始检查系统的说明并下载所有支持的挖矿机应用和benchmark。这是为系统安装最优的加密货币挖矿机。下图是benchmark和加密货币挖矿进程执行的过程。

![PoC&#xFF1A;&#x6EE5;&#x7528;PowerShell Core](https://img.4hou.com/wp-content/uploads/2018/12/d1b6011bc22bbd45268e.png)

图7. MultiPoolMiner的benchmark和加密货币挖矿进程

## **最佳实践**

考虑到PowerShell Core被滥用带来的影响，研究人员建议用户采用最佳实践来减少滥用PowerShell Core带来的威胁：

**·** 应用行为监控这样的安全机制，这可以帮助检测、监控和预防系统中执行的恶意活动。好的行为监控系统不仅可以拦截恶意软件相关的行为，还可以监控和预防不寻常的行为，比如文档中的恶意宏唤醒PowerShell Core。

**·** 深度防御。沙箱可以提供多一层的安全防护。应用防火墙、IDPS系统都可以帮助阻止C2通信和数据泄露这样的行为。

**·** 应用最新的补丁来预防漏洞被利用。

**·** 最小权限原则。限制PowerShell和PowerShell Core的使用，移除或禁用非必须的和过时的插件和组件，以减小系统的攻击面。

