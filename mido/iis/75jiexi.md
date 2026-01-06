# 7.5解析绕过漏洞

漏洞原理： 当安装完成后， php.ini里默认cgi.fix\_pathinfo=1，对其进行访问的时候，在URL路径后添加.php后缀名会当做php文件进行解析，漏洞由此产生

注意：在进行实际的测试的时候，发现漏洞并没有产生，后来发现要设置FastCGI为关闭，该项好像是用来处理数据文件 取消勾选，再次进行测试就能够成功解析

![](<../../.gitbook/assets/image (245).png>)

创建一个txt文件，命名为demo，内容为:&#x20;

```
<?php phpinfo(); ?>
```

进行访问创建的demo文件：

![](<../../.gitbook/assets/image (253).png>)

利用解析漏洞进行访问，在txt 后面加上 /.php进行访问，成功解析

![](<../../.gitbook/assets/image (89).png>)
