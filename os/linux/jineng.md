# 实战技能

Linux常见目录介绍：

> / 根目录  
> /root 用户的家目录  
> /home/username 普通用户的家目录  
> /etc 配置文件目录  
> /bin 命令目录  
> /sbin 管理命令目录  
> /usr/bin /usr/sbin 系统预装其他命令



## 一、操作系统

### 1、万能的帮助命令

#### man帮助

man是manual的缩写 用法： `man ls` 

![](../../.gitbook/assets/image%20%281062%29.png)

man也是一条命令，分为9章，可以使用man命令获得man的帮助 `man 7 man`

![](../../.gitbook/assets/image%20%281063%29.png)

man各章内容：

![](../../.gitbook/assets/image%20%281064%29.png)

`man -a passwd`,可查看文件的属性

#### help帮助

shell自带的命令称为内部命令，其他的是外部命令

内部命令使用help帮助 `help cd`

外部命令使用help帮助 `ls --help`

![](../../.gitbook/assets/image%20%281065%29.png)



#### info帮助

info帮助比help更详细，作为help的补充 `info ls` ，只是都是英文版的结果



### 2、pwd&ls

一切皆文件

* 文件查看
* 文件目录的创建与删除
* 通配符
* 文件操作
* 文本内容查看

显示当前的目录名称：

* `pwd` 显示当前的目录名称

文件查看：

* `ls` 查看当前目录下的文件
* `ls [参数，选项]参数`

  常用参数：

* `-l`  长格式显示文件
* `-a`    显示隐藏文件
* `-r`    逆序显示
* `-t`    按照事件顺序显示
* `-R`    递归显示

`man ls` 可获得命令的详细选项

![](../../.gitbook/assets/image%20%281067%29.png)

![](../../.gitbook/assets/image%20%281066%29.png)

### 3、cd

更改当前的操作目录：

* `cd` 更改当前的操作目录
* `cd /path/to/...` 绝对路径
* `cd ./path/to/...` 相对路径
* `cd ../path/to/...` 相对路径

 因为shell编程有补全功能，所以可以通过Tab键补全命令行

技巧： 

> `cd -` 回到刚才的目录  
> `cd  ..`回到当前目录的上级目录

### 4、目录的创建删除，复制移动

## 二、系统管理

## 三、shell

## 四、文本操作

## 五、服务管理

