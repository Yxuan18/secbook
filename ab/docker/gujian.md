# 固件相关

##  binwalk安装

解路由器固件需要用到binwalk。

本人建议下载源码，自己编译安装，这样可以安装到最新版本，还有一个原因就是 apt-get安装的binwalk会缺少很多依赖。

```bash
apt-get update -y && apt-get install -y build-essential autoconf git  

# https://github.com/devttys0/binwalk/blob/master/INSTALL.md  
git clone https://gitee.com/yixuan1/binwalk.git  
cd binwalk  


# python wget git 
apt-get update
apt-get install -y python2.7  python3 git wget

# python2.7安装
python2.7 setup.py install  

# python2.7手动安装依赖库  
apt-get install -y python-lzma python-crypto libqt4-opengl python-opengl python-qt4 python-qt4-gl python-numpy python-scipy python-pip  
pip install pyqtgraph  

apt-get install -y  python-pip  
pip install capstone  

# Install standard extraction utilities（必选）  
apt-get install -y mtd-utils gzip bzip2 tar arj lhasa p7zip p7zip-full cabextract cramfsprogs cramfsswap squashfs-tools  

# Install sasquatch to extract non-standard SquashFS images（必选）  
apt-get install -y zlib1g-dev liblzma-dev liblzo2-dev  
> git clone https://github.com/devttys0/sasquatch
> git clone https://gitee.com/yixuan1/sasquatch.git  # 二选一
cd sasquatch && ./build.sh

# Install jefferson to extract JFFS2 file systems（可选）  
pip install cstruct  
git clone https://github.com/sviehb/jefferson  
(cd jefferson && sudo python setup.py install)  

# Install ubi_reader to extract UBIFS file systems（可选）  
apt-get install -y liblzo2-dev python-lzo  
git clone https://github.com/jrspruitt/ubi_reader  
(cd ubi_reader && sudo python setup.py install)  

# Install yaffshiv to extract YAFFS file systems（可选）  
git clone https://github.com/devttys0/yaffshiv  
(cd yaffshiv && sudo python setup.py install)  

# Install unstuff (closed source) to extract StuffIt archive files（可选）  
wget -O - http://my.smithmicro.com/downloads/files/stuffit520.611linux-i386.tar.gz | tar -zxv  
cp bin/unstuff /usr/local/bin/
```

## qemu-system安装

```bash
apt-get update -y && apt-get install -y qemu -system-mips && apt-get clean
```

