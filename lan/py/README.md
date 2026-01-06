# python

## 一、参考资料

一些电子书参考资料，链接如下：

1. &#x20;[**《A Byte of Python（简明 Python 教程）》**](https://wizardforcel.gitbooks.io/a-byte-of-python/content/)\
   Python 初学者的极佳教材
2. &#x20;[**《Python Cookbook》**](https://python3-cookbook.readthedocs.io/zh_CN/latest/)\
   有很多高级技巧，想了解 Python 底层的工作原理就看这本
3. &#x20;[**《利用 Python 进行数据分析》**](https://seancheney.gitbook.io/python-for-data-analysis-2nd/)\
   学习 Python 基础库最好的书
4. &#x20;[**《Scikit-Learn与TensorFlow机器学习实用指南》**](https://hand2st.apachecn.org/#/README)\
   机器学习书中理论结合实战最好的书
5. &#x20;[**《problem-solving-with-algorithms-and-data-structure-using-python》**](https://facert.gitbooks.io/python-data-structure-cn/)\
   Python 数据结构与算法相关的书很少，这本堪称最好
6. [最全的在线手册](https://docs.pythontab.com)\
   各种教程手册汇集到一起，哪个不会点哪个
7. 《[自学是门手艺](http://lixiaolai.com/#/the-craft-of-selfteaching/)》——李笑来

## 二、相关问题

### 1、在Linux上没有PIP模块应该怎么办&#x20;

终极解决方案：[get-pip.py](https://bootstrap.pypa.io/get-pip.py)&#x20;

```bash
## 下载安装脚本
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```

在没有PIP库的时候，直接在本地环境上以对应python版本运行该脚本文件，即可获得PIP&#x20;

详细用法为：&#x20;

```bash
python3 get-pip.py 
python2 get-pip.py 
```

以KALI为例，运行成功后，会出现两个pip，分别为pip2与pip3，对应的就是python2的pip与python3的pip

![](<../../.gitbook/assets/image (687).png>)

### 2、pip install 的时候报错，想要更换源怎么操作&#x20;

Windows方法如下：&#x20;

在自己用户的文件夹下创建名为pip的文件夹，并在文件夹中创建pip.ini文件

此处使用清华的源，内容为：

```aspnet
[global] 
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

&#x20;之后，点击保存即可&#x20;

![](<../../.gitbook/assets/image (643).png>)

最后效果为： C:\Users\XXX\pip\pip.ini&#x20;

Linux方法如下：&#x20;

在自己用户目录下创建隐藏文件夹.pip，并在文件夹中创建pip.conf文件，内容为：

```yaml
[global] 
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install] 
trusted-host = https://pypi.tuna.tsinghua.edu.cn
```

之后，保存并退出即可&#x20;

![](<../../.gitbook/assets/image (282).png>)

最后文件路径为：\~/.pip/pip.conf&#x20;

### 3、python2与3共存在一个系统上&#x20;

Linux中默认版本是python2，python3也有，此处以Windows为例：

1、在配置环境变量的时候，若是想使用python2，就可以直接将python2的变量排在python3之前，具体如下（反之同理）：

![](<../../.gitbook/assets/image (351).png>)

此时尝试输入python，即可看到所对应的版本：

![](<../../.gitbook/assets/image (849).png>)

2、如果也想使用python3，此时就可以通过`py -3`的方式来切换所使用的python版本

![](<../../.gitbook/assets/image (437).png>)

### 4、pip更新第三方库时报错

1、错误内容如下：

![错误图片](<../../.gitbook/assets/image (81).png>)

```bash
ERROR: After October 2020 you may experience errors when installing or updating packages. This is because pip will change the way that it resolves dependency conflicts.
We recommend you use --use-feature=2020-resolver to test your packages with the new resolver before it becomes the default.
jedi 0.17.2 requires parso<0.8.0,>=0.7.0, but you'll have parso 0.8.0 which is incompatible.
```

2、解决如下：在更新第三方库或者安装的时候，在命令中添加：  `--use-feature=2020-resolver`

```
py -3 -m pip install --upgrade colorama idna lxml parso requests setuptools urllib3 --use-feature=2020-resolver
```

### 5、批量更新pip第三方库

&#x20;建议来自网络，涉及脚本代码如下：（建议根据电脑有的python版本进行适当更改call函数中的内容）

```python
import pip
from pip._internal.utils.misc import get_installed_distributions
from subprocess import call
import time
 
for dist in get_installed_distributions():
    print(dist.project_name)
 
for dist in get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```

![运行截图](<../../.gitbook/assets/image (215).png>)

### 6、ubuntu更新python3版本

&#x20;背景：因为阿里云主机上原来的python版本是3.6，但还是想使用最新的python版本（虽然自己知道差距并不是太大），鼓捣了好长的时间，终于可以了

&#x20;步骤如下：

1、更新依赖包

```bash
$ apt update
$ apt install software-properties-common
```

![](<../../.gitbook/assets/image (633).png>)

2、添加PPA源

```
$ add-apt-repository ppa:deadsnakes/ppa
>> Press [ENTER] to continue or Ctrl-c to cancel adding it.(当到这里时，点击回车键即可)
```

![](<../../.gitbook/assets/image (638).png>)

3、安装python3.9并配置为系统默认版本

```bash
apt install python3.9    #安装python3.9

which python3.9
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1
which python3.6
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 2
update-alternatives --config python3
```

![](<../../.gitbook/assets/image (487).png>)



