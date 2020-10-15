# 深入分析PowerShell的两面性

## **概述**

在这一系列文章中，我将站在攻击与防御的两个不同视角，重点介绍已知攻击者如何利用不同的策略和技术来实现其恶意目的，并分析我们应该如何检测并阻止这些恶意活动。我将使用MITRE ATT&CK知识库和术语来说明其中涉及的各项技术。如果大家对MITRE ATT&CK技术不是很了解，建议可以先参考我在2018年发表过的文章和视频，题目名为“了解网络犯罪分子的工作方式，实现更加智能的安全”（Smarter Security Starts with Understanding How Cybercriminals Work）。

之所以使用ATT&CK知识库，其原因之一在于，目前安全领域的技术正在不断发展，可以更加准确地评估针对现实世界的特定安全攻击的效果。已经有越来越多的组织和网络专业人员正在使用这样的知识库来更加准确地衡量这种有效性。如果您的组织目前还没有使用这种方法论，我强烈建议大家可以考虑尝试采用这种方法。

在本系列的第一篇文章中，我们将首先分析广受攻击者欢迎的PowerShell。

## **关于PowerShell**

微软的.NET是一种免费的跨平台Web服务策略，可以帮助管理员使用多种语言编辑器和库来快速构建应用程序。PowerShell在2006年11月首次发布，是一种基于.NET的命令行脚本语言，可以帮助管理员和高级用户快速自动化管理操作系统所需的某些日常任务。它还可以用于使用多种语言编辑器和库快速构建跨平台的应用程序。在管理网络中的操作系统时，PowerShell基本上已经演变成一种通用的标准。目前，PowerShell是开源的，并且默认预装在很多版本的Windows上。

尽管如此，我们大部分人可能还了解到，攻击者往往会利用这种易于使用的脚本语言，包括利用它来安装恶意Payload，并且已经取得了相当大的“成功”。我们分析其原因如下：

1、目前，PowerShell已经预装在Windows计算机上。

2、PowerShell能提供一种简单的方式，直接从内存中执行任何Payload，这是一个非常有用的特性，可以允许攻击者传递无文件恶意软件。

3、PowerShell易于进行混淆，以逃避基于签名的防御方式。

4、PowerShell是一个受信任的工具，管理员每天都在使用，并且通常不会对其进行加固。

5、许多使用PowerShell的恶意脚本都可以免费获得，包括PowerShellEmpire、PowerSploit等工具。

由于上述这些优势，多年以来，网络攻击者持续利用PowerShell并取得了一定的成功。幸运的是，较高版本的脚本语言已经具有能更好地保护PowerShell环境的功能。

## **防范PowerShell攻击**

要防御PowerShell攻击，我们可以采取一些防御措施，包括：

1、约束语言模式

禁止直接访问.NET脚本，通过Add-Type cmdlet调用Win32 API以及与COM对象进行交互（注意：请谨慎操作，这样可能会影响一些现有的PowerShell管理任务）。

2、升级到PowerShell v5（该版本增加了许多安全功能，具有更加良好的安全性）

（1）脚本块日志：允许管理员查看脚本试图执行的操作，在该版本之前，未记录此活动。

（2）系统级监控。

（3）反恶意软件集成（在v3中实现）：AMSI（反恶意软件扫描接口）为其他安全厂商提供了一个接口，以便在脚本运行前对其进行检查。这是一个非常好的工具，但有一些攻击方法仍然可以对其进行绕过。

（4）Applocker：根据Applocker策略验证脚本。

3、记录PowerShell活动

通过组策略为各种PowerShell模块启用日志记录。

4、删除PowerShell v2

这是较旧的不安全脚本，在安装v5版本时不会被删除。如果该版本仍存在，攻击者就仍然可以使用这一不安全的版本。

powershell.exe -Version 2.0 -Command {

5、代码签名

我们可以对PowerShell脚本进行代码签名，并且仅运行已经签名的脚本。尽管目前仍然存在一些绕过该方法的方式，但这种方式相对比较有效。

6、使用Just Enough Administration（JEA）限制管理员权限

通过PowerShell远程处理启用基于角色的管理员，这样就可以限制登录用户所执行的操作。

正如大家可能会注意到的，在上面我建议大家采取安全措施的同时，还提到攻击者仍然有方法可以绕过其中的一些防御方式。这些绕过技术包括PowerShell降级攻击、进程注入和PowerShell混淆处理等。尽管如此，上述的这些方式仍然非常有价值，因为并非每一个攻击者都知道如何开展这些绕过攻击，所以这样的机制将会有效防范技术实力较弱的攻击者。我们需要时刻知道，我们可以在无需调用powershell.exe的情况下运行PowerShell脚本。

## **真实恶意样本和检测**

通常，攻击者会利用PowerShell和武器化文档（例如Word或Excel文档）共同执行恶意Payload。通常情况下，文档中将包含一个宏，该宏将会调用PowerShell来执行其恶意Payload。当然，攻击者还有很多其他方式可以利用，但接下来，让我们集中讨论一些使用PowerShell来下载和执行文件的常见攻击技术。

PowerShell允许我们使用Invoke-WebRequest、System.Net.WebClient和Start-BitsTransfer来下载文件。随后，我们可以使用Start-Process、Invoke-item或Invoke-Expression来运行下载的文件。如果我们发现了上述这样的组合，并且了解到管理员并没有利用这些工具来下载并执行文件，那么就证明很可能是恶意的，需要引起我们的关注。

除了下载文件之外，System.Net.Webclient还允许我们将文件的内容直接下载到位于内存的正在运行的进程中，随后运行。这也就是众所周知的“无文件恶意软件”，用于尝试绕过传统的反病毒产品。以下是使用System.Net.Webclient cmdlet的两种方式，以及一些我们在野外看到的恶意脚本的真实示例，这些示例用于下载恶意Payload并执行。

\(New-object System.net.webclient\).Downloadfile\(\) 将文件下载到磁盘

\(New-object System.net.webclient\).DownloadString\(\) 将内容直接移动到内存中

在下图中，我们可以看到一个攻击者生成的PowerShell代码示例，该PowerShell代码从多个动态生成的URL下载Emotet恶意文件。如果大家想了解有关该威胁的更多信息，请参考我们在去年发表的文章《Emotet恶意软件的新变种分析》。

用于下载Emotet恶意文件的PowerShell代码：

![&#x653B;&#x51FB;&#x4E0E;&#x9632;&#x5FA1;&#x7684;&#x53CC;&#x5203;&#x5251;&#xFF1A;&#x6DF1;&#x5165;&#x5206;&#x6790;PowerShell&#x7684;&#x4E24;&#x9762;&#x6027;](https://img.4hou.com/uploads/ueditor/php/upload/image/20200308/1583653116793792.png)

下图中展示的另外一个示例，是利用PowerShell下载一个广为人知的威胁，被称为NetWire RAT，该威胁会记录受害者的所有键盘操作（击键记录器）。如果大家想了解有关该威胁的更多信息，FortiGuard Labs此前曾经在一篇文章中分析过该威胁的一个变种，标题为[《通过网络钓鱼传播：新型NetWire RAT恶意变种分析》](https://www.4hou.com/posts/qAq7)。

通过PowerShell下载NetWire RAT：

![&#x653B;&#x51FB;&#x4E0E;&#x9632;&#x5FA1;&#x7684;&#x53CC;&#x5203;&#x5251;&#xFF1A;&#x6DF1;&#x5165;&#x5206;&#x6790;PowerShell&#x7684;&#x4E24;&#x9762;&#x6027;](https://img.4hou.com/uploads/ueditor/php/upload/image/20200308/1583653123129838.png)

有关近期攻击者利用PowerShell加载Payload的示例，请参考我们在今年早些时候发表的[《Predator the Thief又增加了新的恶意功能》](https://www.4hou.com/posts/L5kj)文章。它利用“恶意Word文档--&gt;宏--&gt;下载混淆后的PowerShell--&gt;Autoit”这样的流程来运行解码后的Autoit脚本，该脚本最终通过进程镂空（Process Hollowing）技术将Payload加载到进程之中。

如果我们要调查攻击者滥用PowerShell的方法，我们可能需要进行更为深入的分析调查，才能确定检测到的进程是否属于恶意。其中的一个方式，我们可以查看其他进程派生出的进程，也就是查看某一特定进程的父进程和祖父进程。这里最为重要的，是我们需要首先掌握合法的父子进程关系，才能判断出异常的父子进程关系。

例如，看到explorer.exe派生出powershell.exe，这是正常的典型行为，但是如果发现cmd.exe是powershell.exe的父进程，则这会非常可疑，因为许多恶意攻击都是通过命令行进程来实现的。当然，这并非一概而论，仍有可能这样的情况是合法的，但我们可以回到派生的cmd.exe进程进行调查。如果被MS Office程序（例如Word或Excel）调用，就很有可能是恶意软件。如果我们的FortiResponder团队在FortiEDR中看到这样的情况，通常会将其判断为网络钓鱼攻击，并且可能表明有用户点击了一些不应点击的内容。

如我们所见，了解进程之间的关联性非常重要。除了MS Office派生出cmd.exe、powershell.exe的情况非常可疑之外，我们还需要关注包括mshta、tasking、wuapp、wscript和script等一些其他的程序。

此外，如果PowerShell尝试下载内容，那么我们可以识别目标IP地址。我们还需要关注每次启动时使用随机脚本名称但扩展名保持不变的情况。在下图中，我们可以发现父子进程关系是非常可疑的，并且如果我们去识别这个IP地址，会发现它实际上是一台恶意C&C服务器。

可疑的父子进程关系：

![&#x653B;&#x51FB;&#x4E0E;&#x9632;&#x5FA1;&#x7684;&#x53CC;&#x5203;&#x5251;&#xFF1A;&#x6DF1;&#x5165;&#x5206;&#x6790;PowerShell&#x7684;&#x4E24;&#x9762;&#x6027;](https://img.4hou.com/uploads/ueditor/php/upload/image/20200308/1583653130124760.png)

还有一些标志选项，可以帮助我们寻找或检测到恶意PowerShell活动。下面是一些值得关注的标志：

1、Exec Bypass：在设置执行策略后，该标志可以允许绕过策略。我们可能会设置一些用于提升安全性的限制，但该标志可能会绕过其中的一些限制。

2、WindowsStyle hidden：该标志将导致PowerShell界面对用户不可见，用于隐藏窗口。

3、Nop或Noprofile：该标志将忽略计算机中预先设置好的配置文件。

4、Enc或Encode：将使用Base64进行编码。

5、Mixed Case：将使用大小写字母混合的方式。

6、lex：运行命令或表达式。

在下图中，展示了使用Base64编码后的PowerShell脚本的示例。

使用Base64编码的PowerShell脚本：

![&#x653B;&#x51FB;&#x4E0E;&#x9632;&#x5FA1;&#x7684;&#x53CC;&#x5203;&#x5251;&#xFF1A;&#x6DF1;&#x5165;&#x5206;&#x6790;PowerShell&#x7684;&#x4E24;&#x9762;&#x6027;](https://img.4hou.com/uploads/ueditor/php/upload/image/20200308/1583653138534301.png)

我上面所列举出来的大多数内容，都是利用PowerShell执行Payload以获取对计算机的初始访问权的攻击者经常会用到的标志。但是，需要关注的是，PowerShell技术还可以用于在网络上横向移动和建立持久性。

在掌握了恶意攻击者使用PowerShell的各种方式之后，我们就需要确保能正确收集日志，并设置正确的安全控制。用户如果使用EDR技术或MDR服务，将有助于防御或检测并响应所有与PowerShell相关的已知威胁，也包括很多未知威胁。同样，用户也可以选择SIEM来实现检测，其中包含许多预先设置的规则，一旦攻击者尝试借助PowerShell执行恶意活动就会触发这样的规则。下图展示了攻击者在尝试利用PowerShell下载内容时，FortiSIEM产生的告警。

攻击者尝试打开PowerShell并下载文件时触发的告警：

![&#x653B;&#x51FB;&#x4E0E;&#x9632;&#x5FA1;&#x7684;&#x53CC;&#x5203;&#x5251;&#xFF1A;&#x6DF1;&#x5165;&#x5206;&#x6790;PowerShell&#x7684;&#x4E24;&#x9762;&#x6027;](https://img.4hou.com/uploads/ueditor/php/upload/image/20200308/1583653147436103.png)

如果用户没有部署这些产品或使用类似的技术，而是希望能自行进行监控，那么就需要确保在终端上记录进程信息（查找事件4688和4104），并查看PowerShell脚本（脚本块日志记录）内容。通过上述方式，管理员可以创建自己的触发器，但由于需要考虑多种不同维度的指标，并且攻击者的特征可能持续会增多，这一过程相对比较困难。下图展示了一个PowerShell脚本示例，该脚本下载并执行MimiKatz以转储凭据。

用于下载并执行MimiKatz的PowerShell脚本：

![&#x653B;&#x51FB;&#x4E0E;&#x9632;&#x5FA1;&#x7684;&#x53CC;&#x5203;&#x5251;&#xFF1A;&#x6DF1;&#x5165;&#x5206;&#x6790;PowerShell&#x7684;&#x4E24;&#x9762;&#x6027;](https://img.4hou.com/uploads/ueditor/php/upload/image/20200308/1583653152101015.png)

## **模拟攻击技术的演练**

随着我们越来越熟悉攻击者滥用PowerShell的方式，我们也需要测试自己的防御措施是否有效，或者选用成熟的安全产品来实现防御。无论选择哪种方式，我们还需要了解可以利用的一些工具。下面列出了其中的一些工具。

### 1、模拟工具

（1）Atomic Red Team：通过执行简单的测试来检测控制措施是否有效，这些测试均采用了与攻击者相同的技术。

（2）APT Simulator：Windows批处理脚本，使用一组工具和输出文件来使系统看上去像是被攻陷的状态。

（3）Caldera（后渗透攻击）：一种自动化的攻击者模拟系统，可以在Windows Enterprise网络中进行后渗透攻击的攻击行为。

### 2、开源防御测试工具

（1）Kali Linux / Metasploit：基于Debian的Linux发行版本，用于高级渗透测试和安全审计。Kali中包含数百种工具，可以用于各种信息安全任务，例如渗透测试、安全研究、计算机取证和逆向工程。

（2）PowerShell Empire（目前已停止维护，但仍然非常好用）：Empire是一个PowerShell后漏洞利用代理，基于加密的安全通信和灵活的体系结构来构建。

（3）PowerSploit：PowerSploit是微软PowerShell模块的集合，可以用于帮助渗透测试人员模拟并评估所有的攻击阶段。

（4）Unicorn Python脚本：这是一个简单的工具，首先进行PowerShell降级攻击，然后将Shellcode直接注入到内存中。还有其他更多工具可以选用，但是其中的一些工具能很好地运行。下面的视频中演示了如何创建一个混淆后的宏，该宏可以用于创建典型的恶意Word文档进行模拟测试。其中包含一个PowerShell脚本，该脚本使用Unicorn脚本（用于PowerShell降级攻击）和Metasploit建立与远程服务器的连接。这样的工具可以帮助我们测试安全防御措施，并确保部署了正确的控制措施并开启了适当的日志记录。

演示视频：https://youtu.be/Z8m1XsFLBtk

## **总结**

如我们所见，PowerShell是一把善与恶之间的双刃剑。当威胁行为者使用这个预装的工具来实现攻击时，由于它们无需安装其他工具即可完成恶意任务，因此也被称为“离地攻击”（Living off the Land）。要检测并保护组织免受这些技术的攻击，其关键在于，我们需要首先了解环境中合法的PowerShell行为特征，然后了解恶意行为的迹象。这样一来，我们就能够检测环境中是否存在恶意技术，从而可以更好地了解安全状况。

需要说明的是，我在本文中仅仅列举了一些需要注意的事项，因此请确保实时了解所有最新的恶意技术，或者确保安全厂商所提供的安全控制措施能够紧跟最新威胁，同时有能力借助EDR、UEBA和SIEM等技术降低风险并识别恶意活动。最后，在我们测试每种PowerShell攻击技术的过程中，我们需要模拟攻击技术，并监控安全控制措施是否有效，评估是否存在任何差距，记录和分析所需要改进的地方，这非常关键。

