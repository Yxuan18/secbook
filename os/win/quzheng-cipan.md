# 磁盘取证实验

## 1、环境准备

目标系统：Win7SP1x86

1、在D盘创建 image.jpg、video.mov、text.txt、document.docx、image\_d.jpg、video\_d.mov、text\_d.txt、document\_d.docx。

2、删除image\_d.jpg、video\_d.mov、text\_d.txt、document\_d.docx，之后清空回收站 。

![](../../.gitbook/assets/image%20%28347%29.png)

## 2、创建磁盘镜像

在进行磁盘取证时，为了尽量减少目标主机文件系统的变动，我们可以使用离线方式进行磁盘取证，将目标主机的磁盘创建镜像，放在移动磁盘中存储。

### 1、在Kali下创建磁盘镜像

这里使用的是虚拟机，因此我可以直接加载ISO文件进入Live环境，如果在物理机上操作，可以先烧录Kali启动U盘、光盘。我在测试过程中内存容量512MB似乎直接进入不了Kali桌面，1GB内存也很容易就死机，若恰巧遇到配置很低的物理机，可要小心使用Kali了。

#### 1、启动到Live模式下

1、首先启动进入取证模式

![](../../.gitbook/assets/image%20%28287%29.png)

2、接入移动硬盘，fdisk -l 确定移动硬盘的设备名为/dev/sdb1

![](../../.gitbook/assets/image%20%28330%29.png)

3、挂载移动硬盘 

```text
cd /mnt 
mkdir udisk  
mount /dev/sdb1 /mnt/udisk
```

![](../../.gitbook/assets/image%20%28318%29.png)

#### 2、使用Guymager

![](../../.gitbook/assets/image%20%28290%29.png)

1、在目标硬盘上右键 Acquire image，设置相关信息、保存路径、文件名，开始获取磁盘镜像。  下面的hash校验我勾掉了，是为了让速度更快一些。

![](../../.gitbook/assets/image%20%28310%29.png)

2、Start开始后，需要一段时间，有磁盘容量、速度与电脑性能决定。

![](../../.gitbook/assets/image%20%28324%29.png)

3、镜像制作完成。

![](../../.gitbook/assets/image%20%28306%29.png)

![](../../.gitbook/assets/image%20%28365%29.png)

全磁盘镜像文件大小共4.7GB

![](../../.gitbook/assets/image%20%28355%29.png)

磁盘实际使用大小是这样的

![](../../.gitbook/assets/image%20%28278%29.png)

#### 3、使用dd

```text
fdisk -l    #判断目标磁盘编号 
#if=指定需要制作映像设备，-of=指定保存的位置

dd if=/dev/sda of=/mnt/udisk/Forensic/dd/sda
```

![](../../.gitbook/assets/image%20%28366%29.png)

dd速度非常慢，且在备份过程中没有任何进度提示，直接放弃换用增强版dd——dc3dd

#### 4、使用dc3dd

dc3dd和dd参数使用是一样的，它们一样是完整备份，对备份盘容量需求比较大，这里只备份sda3（D盘），可以看到备份了约6GB大小

![](../../.gitbook/assets/image%20%28367%29.png)

最终D盘分区镜像大小5.81GB

![](../../.gitbook/assets/image%20%28279%29.png)

### 2、在Windows下创建磁盘镜像 

在Windows下也最好使用Live系统如WindowsPE启动盘进行取证，但是由于这里没有现成的包含取证工具的启动盘，因此直接在系统里操作。取证工具、创建的磁盘镜像文件，都放在虚拟机的共享磁盘上，尽可能避免改变目标文件系统。

#### 1、使用X-Ways Forensics 

这个工具就是Winhex的取证加强版，因此界面几乎都一样。 

1、工具栏选择Create Disk Image

![](../../.gitbook/assets/image%20%28336%29.png)

2、直接给整个磁盘创建镜像，创建分区镜像可以选择上边的

![](../../.gitbook/assets/image%20%28329%29.png)

3、选择好存储路径后点OK开始

![](../../.gitbook/assets/image%20%28277%29.png)

4、开始创建镜像，镜像备份的速度比dd真是快的太多了

![](../../.gitbook/assets/image%20%28292%29.png)

全盘备份5.8GB，要比guymager备份的文件容量多1GB，这个结果可能是受到了在线备份镜像的影响

![](../../.gitbook/assets/image%20%28351%29.png)

#### 2、使用AccessData FTK Imager 

由于我找到的这个版本不支持32位系统，因此只能使用它在另外一台x64虚拟机做一个创建镜像的演示。（D盘环境存在相同的文件读写删除操作）

![](../../.gitbook/assets/image%20%28315%29.png)

1、同样在工具栏选择Create Disk Image

![](../../.gitbook/assets/image%20%28357%29.png)

2、选择整个磁盘或分区，这里准备备份一个分区D盘

![](../../.gitbook/assets/image%20%28328%29.png)

![](../../.gitbook/assets/image%20%28294%29.png)

3、选择备份类型，这里不建议用Raw，那样就跟dd一样创建一个和磁盘大小一样的镜像，无视实际使用空间大小

![](../../.gitbook/assets/image%20%28275%29.png)

4、按需填写证据信息

![](../../.gitbook/assets/image%20%28284%29.png)

5、选择存储位置，之后开始创建镜像

![](../../.gitbook/assets/image%20%28289%29.png)

![](../../.gitbook/assets/image%20%28314%29.png)

D盘镜像大小21.6MB（如果使用RAW格式，将会是10GB）

![](../../.gitbook/assets/image%20%28291%29.png)

## 3、分析磁盘镜像 

Linux下分析镜像的工具没有发现什么比较好的，因此放弃Linux，只在Windows下进行。

### 1、X-Ways Forensics分析证据

相较于FTK，X-Ways拥有更完善的案件、证据管理模式，可以保存案件后续再接着分析。 

1、创建案件

![](../../.gitbook/assets/image%20%28298%29.png)

2、导入证据：可以导入各类证据，这里选择镜像

![](../../.gitbook/assets/image%20%28282%29.png)

3、导入前面创建的4个镜像（包含两个全磁盘镜像、1个x86虚拟机的D盘、1个x64虚拟机的D盘）

![](../../.gitbook/assets/image%20%28281%29.png)

4、查看D盘里的文件，可以正常显示图片，但是这里没有看到被删除的文件（被删除的文件显示为半透明）

![](../../.gitbook/assets/image%20%28326%29.png)

5、寻找被删除的文件：由于删除时，是先del进入回收站，然后清空的，因此被删除的文件会在回收站的路径中。

![](../../.gitbook/assets/image%20%28301%29.png)

6、可以将镜像中有需要的文件恢复出来进一步分析

![](../../.gitbook/assets/image%20%28296%29.png)

### 2、AccessData FTK Imager分析证据 

1、添加证据

![](../../.gitbook/assets/image%20%28346%29.png)

2、选择镜像文件，之后选择位置即可

![](../../.gitbook/assets/image%20%28308%29.png)

3、将4个镜像全部载入，这里不支持重命名

![](../../.gitbook/assets/image%20%28362%29.png)

4、分析文件：FTK同样可以直接预览txt、jpg

![](../../.gitbook/assets/image%20%28327%29.png)

5、寻找被删除的文件：同样到回收站目录下寻找被清空的文件，这里是使用红叉表示其被删除了

![](../../.gitbook/assets/image%20%28320%29.png)

6、导出文件：FTK同样可以导出镜像内的文件

![](../../.gitbook/assets/image%20%28331%29.png)

#### 磁盘镜像挂载

1、FTK有个特殊功能，可以把磁盘镜像映射为一个虚拟磁盘

![](../../.gitbook/assets/image%20%28288%29.png)

2、这样就多了一个和win7x86主机里一样的分区

![](../../.gitbook/assets/image%20%28316%29.png)

3、不用的时候unmount即可

![](../../.gitbook/assets/image%20%28305%29.png)

## 四、快速提取镜像内的文件 

```text
## 使用foremost提取磁盘映像里的文件

foremost -t all -i sda3    
# -i指定镜像文件，-t指定文件类型，all是所有支持的类型，具体支持的类型查看man
```

![](../../.gitbook/assets/image%20%28333%29.png)

经过测试，jpg、mov、txt、docx四种类型的文件，只能提取到docx和jpg两种格式的文件

![](../../.gitbook/assets/image%20%28312%29.png)



