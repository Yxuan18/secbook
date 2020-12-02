# 应用审计指南

## 一． 概述 

### 1.1 前言 

提高安全测试的效率，配合业务方自查。在开发以及 QA 阶段 就消除常见的安全问题，提高安全审计的效率。 本文档有些检测项会涉及到 smali 相关知识，如果对 smali 语法不熟悉的话，可直接使用我们推出的静态审计工具一键 操作进行审计。 

### 1.2 适用范围和对象 

本文档用于 Android 客户端的日常审计参考，提升安全审计 的效率。 

### 1.3 工具集 

二进制查看工具：winhex、UltraEdit、010Editor。   
SO 文件查看工具：readelf.exe 、IDA APK   
反编译工具：apktool.jar DEX   
反编译工具：baksmali.jar/smali.jar AndroidManifest.xml   
反编译工具：AxmlPrinter.jar Apk   
查壳工具：isProtected.py/ PKiD.exe 

## 

![](../../../.gitbook/assets/image%20%281016%29.png)

2.获取反编译输出目录下所有 smali 文件,从文件内容中查 找“Landroid/content /Context;-&gt;openOrCreateDatabase\(Ljava/lang/String;I Landroid/database/sqlite/SQLiteDatabase$CursorFactory;\)Landroid/database/sqlite/SQLiteDatabase;”字符 串，找到以后根据 smali 语法去分析方法第二个参数值为 1/2/3，说明存在风险。 示例：

![](../../../.gitbook/assets/image%20%281042%29.png)

漏洞描述：APP 在使用 openOrCreateDatabase 创建数据库 时，将数据库设置了全局的可读权限，攻击者恶意读取数 据库内容，获取敏感信息。 漏洞类型：APP 自身逻辑问题 漏洞评级：中危 漏洞危害：攻击者会篡改、伪造内容，也可能会进行诈骗 等行为，造成用户财产损失。 修复建议：使用 sqlcipher 加密库对数据库做加密处理 2. SharedPreference 全局读写漏洞 测试方式： 1. 使用 Apktool 反编译 apk 文件，命令行如下： java –jar apktool.jar d xxx.apk –o out/ 反编译成功结果如下图：

![](../../../.gitbook/assets/image%20%281016%29.png)



![](../../../.gitbook/assets/image%20%281016%29.png)

![](../../../.gitbook/assets/image%20%281016%29.png)

![](../../../.gitbook/assets/image%20%281016%29.png)

![](../../../.gitbook/assets/image%20%281016%29.png)

![](../../../.gitbook/assets/image%20%281016%29.png)

![](../../../.gitbook/assets/image%20%281016%29.png)

![](../../../.gitbook/assets/image%20%281016%29.png)

