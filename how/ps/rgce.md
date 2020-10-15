# 如何绕过PowerShell访问限制并实现PowerShell代码执行

![](https://image.3001.net/images/20191103/1572780230_5dbeb8c678aeb.jpg!small)

 **如果你是一名专业的渗透测试人员，那你可能真的离不开PowerShell，但是如果目标系统中的某个策略组织我们访问PowerShel.exe，那我们该怎么办呢？没错，这个问题经常会困扰我们，而且网上也有很多的应对方法。**

 在这篇文章中，我将跟大家介绍一种快速且狡猾的绕过技术，这项技术需要利用C\#编译代码来执行我们的PowerShell脚本。

 首先，我们为什么不适用powershell.exe来执行我们的.ps1脚本呢？因为PowerShell脚本跟C\#一样，它们两个对于.NET框架而言，都只是“前端”方面的东西，它们的本质都只是一种编程语言。不过在C\#中，我们需要对程序代码进行编译才可以使用和执行，即编译型语言，这是它和PowerShell这种解释型脚本语言有很大区别。

 鉴于PowerShell.exe只是.NET程序集“system.management.automation”的解释器，因此它应该可以在C程序中与这个对象进行交互并执行.ps1脚本。

 **下面给出的就是实例代码：**

```diff
using System.Collections.ObjectModel; 

    using System.Management.Automation; 

    using System.Management.Automation.Runspaces; 

    using System.IO;

    using System;

    using System.Text;

    namespace PSLess

    {

     class PSLess

     {

       static void Main(string[] args)

       {

         if(args.Length ==0)

             Environment.Exit(1);

         string script=LoadScript(args[0]);

         string s=RunScript(script);

         Console.WriteLine(s);

         Console.ReadKey();

       }

     private static string LoadScript(string filename) 

     { 

       string buffer ="";

       try {

        buffer = File.ReadAllText(filename);

        }

       catch (Exception e) 

       { 

         Console.WriteLine(e.Message);

         Environment.Exit(2);

        }

      return buffer;

     }

     private static string RunScript(string script) 

     { 

        Runspace MyRunspace = RunspaceFactory.CreateRunspace();

        MyRunspace.Open();

        Pipeline MyPipeline = MyRunspace.CreatePipeline(); 

        MyPipeline.Commands.AddScript(script);

        MyPipeline.Commands.Add("Out-String");

        Collection<PSObject> outputs = MyPipeline.Invoke();

        MyRunspace.Close();

       StringBuilder sb = new StringBuilder(); 

       foreach (PSObject pobject in outputs) 

       { 

           sb.AppendLine(pobject.ToString()); 

       }

        return sb.ToString(); 

      }

     }

    }
```

 其中的RunScript\(\)方法会创建一个“runspace”对象，我们可以将其视作是PowerShell运行时的一个独立实例。接下来，我们需要将脚本添加到新创建的pipeline中，并对通信信道进行排序，最后通过Invoke\(\)方法执行我们的脚本命令。

 最终的结果将会被追加到我们的字符串生成器中，然后作为字符串发回给调用函数，以便显示在控制台的输出结果中。这也就是为什么我们要在命令中添加“Out-String”的原因。

 怎么样，整个过程很简单吧？

 接下来，我们需要对代码进行编译并完成代码测试。

```text
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe    /reference:    C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0__31bf3856ad364e35\system.management.automation.dll     /out:c:\setup\powerless.exe c:\scripts\powersless.cs
```

 请记住，具体的执行路径需要取决于目标系统中所安装的框架版本。除此之外，别忘了添加对“system.management.automation.dll”程序集的引用。

 如果一切正常的话，我们就已经完成了代码的编译。接下来，创建一个简单地测试脚本：

```text
test.ps1:    echo "Hello from powershell-less"    echo "PID: $pid"
```

 **然后运行该脚本：**

![](https://image.3001.net/images/20191103/1572780253_5dbeb8dddb4f9.jpg!small)

 没错，我们成功了！我们成功地在不直接使用powershell.exe的情况下调用并执行了我们的脚本代码。

 实际上，这是一个非常简单的脚本，如果加上用户输入等处理机制的话，就会变得比较麻烦和复杂了，但对于大多数需要涉及到PowerShell的渗透活动来说，这应该已经够了吧。当然了，广大研究人员也可以根据自己的需要来修改脚本代码，以实现自己的需求。

