# 命令与选项

### 1、docker run

```bash
--name    给运行的镜像命名：
docker run --name myjingxiang centos:7    #给正在运行的镜像命名为myjingxiang

--env    传入环境变量：
    --env MYSQL_ROOT_PASSWORD=1234    # 设置MySQL的root账户密码为1234

-d    设置后台运行镜像

-p    设置端口映射：
    -p :80    #映射docker内部的80镜像到主机的任意端口
    -p 8888:80    #映射docker内部的80端口到主机的8888端口

--link    连接两个镜像：
    --link db:mysql    #其中，db与mysql都是同一个镜像，其中一个是别名
    

```

