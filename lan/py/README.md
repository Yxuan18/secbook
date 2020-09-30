# python

## 一、参考资料

一些电子书参考资料，链接如下：

1.  [**《A Byte of Python（简明 Python 教程）》**](https://wizardforcel.gitbooks.io/a-byte-of-python/content/) ****Python 初学者的极佳教材
2.  [**《Python Cookbook》**](https://python3-cookbook.readthedocs.io/zh_CN/latest/) ****有很多高级技巧，想了解 Python 底层的工作原理就看这本
3.  [**《利用 Python 进行数据分析》**](https://seancheney.gitbook.io/python-for-data-analysis-2nd/) ****学习 Python 基础库最好的书
4.  [**《Scikit-Learn与TensorFlow机器学习实用指南》**](https://hand2st.apachecn.org/#/README) ****机器学习书中理论结合实战最好的书
5.  [**《problem-solving-with-algorithms-and-data-structure-using-python》**](https://facert.gitbooks.io/python-data-structure-cn/) ****Python 数据结构与算法相关的书很少，这本堪称最好
6. [最全的在线手册](https://docs.pythontab.com) 各种教程手册汇集到一起，哪个不会点哪个

## 二、相关问题

### 1、在Linux上没有PIP模块应该怎么办 

终极解决方案：[get-pip.py](https://bootstrap.pypa.io/get-pip.py) 

在没有PIP库的时候，直接在本地环境上以对应python版本运行该脚本文件，即可获得PIP 

详细用法为： 

```text
python3 get-pip.py 
python2 get-pip.py 
```

以KALI为例，运行成功后，会出现两个pip，分别为pip2与pip3，对应的就是python2的pip与python3的pip

![](../../.gitbook/assets/image%20%28522%29.png)

### 2、pip install 的时候报错，想要更换源怎么操作 

Windows方法如下： 

在自己用户的文件夹下创建名为pip的文件夹，并在文件夹中创建pip.ini文件

此处使用清华的源，内容为：

```text
[global] 
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

 之后，点击保存即可 

![](../../.gitbook/assets/image%20%28524%29.png)

最后效果为： C:\Users\XXX\pip\pip.ini 

Linux方法如下： 

在自己用户目录下创建隐藏文件夹.pip，并在文件夹中创建pip.conf文件，内容为：

```text
[global] 
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install] 
trusted-host = https://pypi.tuna.tsinghua.edu.cn
```

之后，保存并退出即可 

![](../../.gitbook/assets/image%20%28518%29.png)

最后文件路径为：~/.pip/pip.conf 

### 3、python2与3共存在一个系统上 

Linux中默认版本是python2，python3也有，此处以Windows为例：

1、在配置环境变量的时候，若是想使用python2，就可以直接将python2的变量排在python3之前，具体如下（反之同理）：

![](../../.gitbook/assets/image%20%28525%29.png)

此时尝试输入python，即可看到所对应的版本：

![](../../.gitbook/assets/image%20%28523%29.png)

2、如果也想使用python3，此时就可以通过`py -3`的方式来切换所使用的python版本

![](../../.gitbook/assets/image%20%28521%29.png)


