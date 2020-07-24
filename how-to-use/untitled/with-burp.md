# with burp

有小伙伴前几天说在burp上可以直接send to sqlmap，可以直接开始扫描，都不用保存那个.txt文件了

听起来好像蛮不错的，第一次接触burp的插件，打开了插件列表，然后发现，，，需要下载安装Jython

### 1、JYTHON安装

![](../../.gitbook/assets/image%20%289%29.png)

点击了Download Jython 按钮之后，跳转到网站，并且下载

![](../../.gitbook/assets/image%20%285%29.png)

```text
java -jar 安装程序.jar  # 运行安装程序，一直下一步，安装好即可
```

### 2、给电脑安装python2

由于现在的电脑上已经有python3了，如果安装python2的话也可以，想切换版本使用的时候可以：

```text
py -2
py -3
```

使用命令查看版本，效果图如下：

![](../../.gitbook/assets/image%20%288%29.png)

### 3、添加sqlmap.jar文件

 1、文件下载地址：[http://pan.baidu.com/s/1skDVwq5](http://pan.baidu.com/s/1skDVwq5) 密码：ce5f

2、下载完成后，选择添加到burp

![](../../.gitbook/assets/image%20%287%29.png)

3、选择好文件后，点击NEXT即可





