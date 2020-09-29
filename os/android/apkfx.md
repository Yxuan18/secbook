# APK终端安全分析法

## APK及其基本结构

### APK组成

* 被编译的代码文件（.dex文件）
* 资源文件（resources）
* assets
* 证书（certificates）
* 清单文件（manifest file）

### 各文件分类及说明

| 文件 | 说明 |
| :--- | :--- |
| classes.dex | DEX是Dalvik EXecutable的简称，classes.dex是将程序中的类和逻辑代码编译成Dalvik虚拟机可以理解的dex文件格式，由Dalvik虚拟机加载并运行，通常为APK的主要代码文件 |
| resources.arsc |  这个文件包含了一些预编译的资源文件，如二进制的XML文档等 |
| META-INF目录 | 在编译生成一个APK包时会对所有要打包的文件做一个SHA-1的校验计算，并把计算结果放在META-INF目录下。 在Android平台上安装APK包时，安装器会对包里的文件进行校验，如果校验结果与META-INF目录下的内容不一致，系统就不会安装这个APK |
| res目录 | res目录主要包含了一些未被编译入resources.arsc的资源文件 |
| assets目录 | assets 目录是一个含有应用程序资源的目录，但是在打包 APK 时不会对 assets目录下的文件做任何处理，并需要使用AssetManager来访问 |
| AndroidManifest.xml | 每个应用的根目录中都必须包含一个 AndroidManifest.xml 文件，这个清单文件为Android系统提供有关应用的基本信息，例如，应用的名字、版本、所需权限、注册的服务、引用的库文件等，系统必须获得这些信息才能运行应用代码。 该文件在打包时会被编译成二进制XML格式，可以使用工具反编译回XML文本格式。 |

### 关于lib目录

lib 目录包含了指定处理器的已编译代码，是特定于处理器的软件层，可分为如下目录

| 目录 | 说明 |
| :--- | :--- |
| Armeabi | 包含所有基于ARM处理器的编译后代码 |
| armeabi-v7a | 包含所有基于ARMv7及更高版本的处理器的编译后代码 |
| arm64-v8a | 包含所有基于ARMv8、ARM64及更高版本的处理器的编译后代码 |
| x86 | 包含所有基于x86处理器的编译后代码 |
| x86\_64 | 包含所有基于x86\_64处理器的编译后代码 |
| Mips | 包含所有基于MIPS处理器的编译后代码 |

## 反编译

### 反编译Dalvik字节码文件

步骤：

1. 解压APK文件并找到classes.dex
2. apktool 反编译APK文件，命令如下： java -jar apktool.jar d APKfile -o outdir
3. smail目录中的文件即为反编译出的代码
4. dex2jar反编译classes.dex文件

### 反编译共享库.so文件

可通过IDA翻译出伪代码分析程序的逻辑



## 逻辑分析

## 重新打包

## 动态调试

## 相关工具

## 保护措施

