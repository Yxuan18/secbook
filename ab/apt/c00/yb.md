# 样本分析

## 前言

很早之前就对APT组织的攻击方式感兴趣，因为他们的传播方式、隐藏方式、攻击方式都让人眼界大开，也可以拓展分析思路的。所以就找了一个海莲花的样本来尝试分析，收获满满。

## 执行流程

利用微软白文件加载恶意wwlib.dll文件，释放出隐藏的doc文件，并在内存中释放出3段shellcode，最后获取恶意程序的链接并下载

![](<../../../.gitbook/assets/image (814).png>)

## 详细分析

### **样本概括**

EXE文件是带有微软签名的合法 word 程序，攻击者通过该程序加载恶意的 wwlib.dll 文件来释放 shellcode 进行攻击。

![](<../../../.gitbook/assets/image (872).png>)

![](<../../../.gitbook/assets/image (796).png>)

dll文件是隐藏的恶意文件

![](<../../../.gitbook/assets/image (436).png>)

![](<../../../.gitbook/assets/image (356).png>)

### **wwlib.dll**

exe文件调用dll，所以可以在 Loadlibrary 函数处下断点，来等待加载。

![](<../../../.gitbook/assets/image (24).png>)

![](<../../../.gitbook/assets/image (745).png>)

继续运行之后会释放doc文件并运行

![](<../../../.gitbook/assets/image (374).png>)

![](<../../../.gitbook/assets/image (302).png>)

申请空间来存放第一次的shellcode，通过xor异或解密之后再存放

![](<../../../.gitbook/assets/image (836).png>)

第一段 shellcode 是在这里之后转到的：

![](<../../../.gitbook/assets/image (431).png>)

![](<../../../.gitbook/assets/image (322).png>)

### **第一段 shellcode**

从一个巨大的字符串数组中循环取出字符串后，经过解密算法来算出需要的函数字符串名称，如LoadlibraryA、VirtualAllocEx等

![](<../../../.gitbook/assets/image (738).png>)

![](<../../../.gitbook/assets/image (308).png>)

![](<../../../.gitbook/assets/image (321).png>)

解密 VirtualAllocEx 之后，申请空间来存放新的 shellcode：

![](<../../../.gitbook/assets/image (799).png>)

### **第二段 shellcode**

解密shellcode之后，会调用第二段shellcode

![](<../../../.gitbook/assets/image (878).png>)

使用极其耗费时间的大循环来解密字符串，防沙箱等检测工具。

![](<../../../.gitbook/assets/image (1014).png>)

解密出 calloc 之后，申请空间：

![](<../../../.gitbook/assets/image (400).png>)

复制数据到申请的空间中：

![](<../../../.gitbook/assets/image (310).png>)

获取网关信息：

![](<../../../.gitbook/assets/image (4).png>)

第三段 shellcode 使用动态提取的方式来获取 CryptAcquireContextA、CryptCreateHash、CryptHashData、CryptDeriveKey，并通过这些函数来保证CryptDecrypt函数的正常运行。

![](<../../../.gitbook/assets/image (856).png>)

创建空间，存放第三段 shellcode：

![](<../../../.gitbook/assets/image (984).png>)

通过 CryptDecrypt 解密函数，每次解密出shellcode的16位数据：

![](<../../../.gitbook/assets/image (705).png>)

![](<../../../.gitbook/assets/image (879).png>)

将解密出的数据复制到申请的空间中：

### **第三段 shellcode**

以创建进程的方式运行第三段 shellcode：

![](<../../../.gitbook/assets/image (901).png>)

在第3段 shellcode 中，会将要连接的url复制到特定内存中，连接url：

![](<../../../.gitbook/assets/image (434).png>)

![](<../../../.gitbook/assets/image (260).png>)

由于C2早已掉线，无法连接，这次的分析也就只能到这里了，估计下载的是一个EXE，想来应该是个具有窃密，回传信息的恶意程序。

![](<../../../.gitbook/assets/image (395).png>)

## 总结

对于海莲花这种靠后梯队的APT组织来说，他所投递的样本相对高梯队的APT组织投递的样本，分析难度还是较低，至少有迹可循。

样本IOC： SHA256 - c0ea37db94aa0d747ece7f46afcf90e43fb22c06731f291f0b2ba189d4326e33.rar
