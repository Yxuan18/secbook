# GDB

gdb是一个由GNU开源组织发布的、UNIX/Linux操作系统下的、基于命令行的、功能强大的程序调试工具。

###  1、安装

在大多数Linux发行版中，gdb都是默认安装的，如果没有，那么在Ubuntu下可以通过apt-get进行安装，安装命令为： 

```text
sudo apt-get install gdb 
```

如果需要调试其他架构的elf程序，则可以安装gdb-multiarch，安装命令为： 

```text
sudo apt-get install gdb-multiarch 
```

此外，gdb也有很多插件，如peda、gef、pwndbg等，这里的插件提供了一些额外的命令，便于对程序进行逆向分析。这些插件都可以在Github上找到，根据其安装说明进行安装即可。下图为gdb安装了peda插件之后运行的界面，可以通过pedahelp命令查看新增的命令。

![](https://staticcdn1-5.umiwi.com/pcebook/online/img/202006/2415930024139100290345813196357108138602.jpg)

### 2、常用命令及功能

| 命令 | 功能 |
| :---: | :---: |
| gdb program | 启动gdb开始调试program |
| quit | 退出调试器 |
| run arguments | 运行被调试程序（指定参数arguments\) |
| attach processID | 把调试器附加到PID为processID的进程上 |
| break \*address | 下断点 |
| info breakpoints | 查看所有断点 |
| delete number | 删除某个断点 |
| stepi | 执行一条机器指令，单步进入子函数 |
| nexti | 执行一条机器指令，不会进入子函数 |
| continue | 继续执行 |
| x/countFormatSize addr | 以指定格式Format打印地址address处的指定大小size、指定数量count的对象Size:b \(字节\)、h \(半字\)、w \(字\)、g \( 8字节）Format： o \(八进制\)、d \(十进制）、x \(十六进制\)、u \(无符号十进制\)、t \(二进制\)、）、f \(浮 点数\)、a \(地址\)、i \(指令\)、c \(字符\)、s \(字符串） |
| info register | 查看寄存器 |
| backtrace | 打印函数调用栈回溯 |
| set $reg=value | 修改寄存器 |
| set _\(type_\) \(address\)=value | 修改内存地址addr为value |
| shell command | 执行shell命令command |
| set follow-fork-mode parent\|child | 当发生fork时，指示调试器跟踪父进程还是子进程 |
| handler SIGALRM ignore | 忽视信号SIGALRM，调试器接收到的SIGALRM信号不会发送给被调试程序 |
| target remote ip:port | 连接远程调试 |

end

