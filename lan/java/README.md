# Java

## 一、常见问题

### 1、JDK14没有JRE

1、使用管理员权限打开CMD\
2、进入JDK目录，并输入如下命令：

```bash
bin\jlink.exe --module-path jmods --add-modules java.desktop --output jre
```

3、执行成功后，JDK目录下就会出现JRE文件夹

![](<../../.gitbook/assets/image (208).png>)

### 2、JAVA多版本切换

在使用环境的过程中，难免会遇到好多的问题，其中最显著的就是：有的工具不支持高版本JAVA，而有时候为了用一些好用的工具，只能委屈自己去使用旧版本的JAVA，那么在这，就讲一讲如何做到让多个版本的JAVA能够共存在一台电脑上

#### **此处以**[**JDK1.8**](https://www.java.com/zh-CN/download/)**与**[**JDK14**](https://www.oracle.com/java/technologies/javase-jdk14-downloads.html)**为例**

已有JAVA环境：

1、在安装完jdk1.8以后，先不用去管环境变量，查看系统目录中是否存在javapath的文件夹，如果有，直接选择删除即可。

```bash
C:\Program Files (x86)\Common Files\Oracle\Java\javapath
C:\ProgramData\Oracle\Java\javapath
```

原因如下：\
因为在安装JDK1.8之后，会自动将java.exe、javaw.exe、javaws.exe三个可执行文件复制到系统目录。由于这个目录在WINDOWS环境变量中的优先级高于path设置的环境变量优先级

我的解决方法：直接删除掉了javapath这个目录。（虽然教程上说只需要删掉文件夹中的三个可执行文件即可）

2、从注册表（运行regedit）中删除jdk的默认启动的版本。

```bash
HKEY_LOCAL_MACHINE\SOFTWARE\JavaSoft中：

Java Development Kit的CurrentVersion项的值
Java Runtime Environment的CurrentVersion项的值
```

![](<../../.gitbook/assets/image (283).png>)

3、添加环境变量

此处设置如下环境变量：

```
JAVA_HOME      %JAVA_HOME8%    # 选择java8版本
JAVA_HOME8     C:\java\java8    #选择JDK所在位置
JAVA_HOME14    C:\java\jdk14  
  
path           %JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;
```

&#x20;个人理解：这个过程就像套娃一样，如果手动切换要使用的java版本的话，直接在环境变量中将`JAVA_HOME`的值中的数字更改一下即可，如果要切换为java14版本的，则将环境变量中更改为：

```
JAVA_HOME      %JAVA_HOME14%
```

![版本切换前后效果图](<../../.gitbook/assets/image (731).png>)

#### 半自动版本切换

背景：因为已经设置好了JDK14为默认版本，那么如果要运行JDK8版本才能运行的软件的时候，会发生什么呢？

示例：冰蝎

![你会发现，并没有什么反应](<../../.gitbook/assets/image (551).png>)

那么这种时候，尝试先进入JDK8的目录下，然后再使用目录下的 java.exe 去执行呢？

![恭喜你，运行成功](<../../.gitbook/assets/image (664).png>)

思路就是这样的，那么我们就可以编写一个用来运行软件的  .bat  文件，内容如下：

```
cd C:\java\java81\bin
java.exe -jar C:\SEC\Behinder_v3.0_Beta_7\Behinder.jar
```

&#x20;经证实，该方法可行
