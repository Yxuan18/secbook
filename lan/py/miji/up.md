# 上半部分

## 一、使用python模块

###  1、介绍

概念：是python程序的最高级别组件，是模块化的

作用：

1. 能够作为整体程序的一部分插入其他模块，在创建紧密结合的应用程序时提供更好的代码分离
2. 会产生单独的命名空间

变量阴影：  
涉及在不同命名空间中重复的变量名，可能导致解释器不能正常使用变量

### 2、使用和导入模块以及命名空间

 命名空间：

概念：模块或组件的控制域。  
作用：隔离统一程序中的对象；隔离不同模块之间的对象

 关于全局变量：

1. 为什么可用： 可以被任何函数调用并相互作用
2. 为什么不被看做最佳实践： 修改全局变量后没有考虑命名空间，而导致程序后面部分造成中断



 函数中调用变量过程：

1. 在函数中查找变量声明
2. 向上移动堆栈并寻找全局定义变量
3. 查看内置库
4. 若上述三步仍未找到，报错

![&#x5B9E;&#x4F8B;](../../../.gitbook/assets/image%20%28527%29.png)

### 3、实现python虚拟环境

### 4、python安装包选项

### 5、利用需求文件并解决冲突

### 6、使用本地不定和约束文件

### 7、使用包

### 8、出差能见wheel和bundle

### 9、源代码与字节码比较

### 10、如何创建和引用模块包

### 11、操作系统专用二进制文件

### 12、如何上传程序到PyPL

### 13、项目打包

### 14、上传到PyPl



## 二、使用python解释器

### 1、介绍2、3、4、5、6、7、8、9、10、11、12、13、

![](../../../.gitbook/assets/image%20%281030%29.png)

## 三、使用装饰器

### 1、介绍

### 2、回顾函数

### 3、装饰器简介

### 4、使用函数装饰器

### 5、使用类装饰器

### 6、使用装饰器模块

## 四、使用collections

### 1、介绍

### 2、回顾容器

### 3、实现namedtuple

### 4、实现双端队列

### 5、实现ChainMap

### 6、实现计数器

### 7、实现OrderedDict

### 8、实现defaultdict

### 9、实现userdict

### 10、实现userlist

### 11、实现userstring

### 12、优化python collections

### 13、窥探collections-extended模块

## 五、生成器，协同程序和并行处理

### 1、介绍

### 2、python中的迭代是如何工作的

### 3、使用itetools模块

### 4、使用生成器的函数

### 5、使用协同程序模拟多线程

### 6、何时使用并行处理

### 7、Fork进程

### 8、如何实现多线程

### 9、如何实现多进程
