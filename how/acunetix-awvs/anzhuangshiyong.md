# 安装到使用

### 1、Windows安装过程

1、解压安装包，并双击Activation.exe

2、点“next”

![](<../../.gitbook/assets/image (824).png>)

3、勾选同意条款，点“next”

![](<../../.gitbook/assets/image (368).png>)

4、填写邮箱、密码和确认密码，点“next”

![](<../../.gitbook/assets/image (1072).png>)

5、打钩，点“next”

![](<../../.gitbook/assets/image (671).png>)

![](file:///C:/Users/73604/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg)6、创建快捷方式，继续点“next”

![](<../../.gitbook/assets/image (700).png>)

7.点“install”

![](<../../.gitbook/assets/image (698).png>)

8、安装

![](<../../.gitbook/assets/image (719).png>)

9、完成安装finish

![](<../../.gitbook/assets/image (832).png>)

10、激活，复制补丁包至安装的目录内

![](<../../.gitbook/assets/image (1).png>)

11、双击Patch.exe

![](<../../.gitbook/assets/image (829).png>)

12、以管理员的身份打开并应用该补丁包

![](<../../.gitbook/assets/image (180).png>)

![](<../../.gitbook/assets/image (68).png>)

13、文件补丁成功后，弹出以下界面

![](<../../.gitbook/assets/image (803).png>)

14、直接点next

![](<../../.gitbook/assets/image (787).png>)

15、点击finish

![](<../../.gitbook/assets/image (1024).png>)

16、    打开后没提示注册，破解成功

![](<../../.gitbook/assets/image (408).png>)

### 2、Linux安装过程

1、安装awvs需要的扩展依赖包

```
sudo apt-get install libxdamage1 libgtk-3-0 libasound2 libnss3 libxss1 -y
```

![](<../../.gitbook/assets/image (286).png>)

2、解压

```
unzip AWVS_12_For_linux.zip
```

![](<../../.gitbook/assets/image (213).png>)

3、给权限

![](<../../.gitbook/assets/image (935).png>)

4、运行安装文件

```
sudo ./acunnerix_trial.sh
```

![](<../../.gitbook/assets/image (376).png>)

enter-->yes-->127.0.0.1-->邮箱、登录密码

![](<../../.gitbook/assets/image (762).png>)

![](<../../.gitbook/assets/image (766).png>)

安装过程

![](<../../.gitbook/assets/image (694).png>)

5、安装完成后，直接打补丁，把破解补丁复制到指定目录下，并设置好权限，直接运行即可

```
cp -a patch_awvs /home/acunetix/.acunetix_trial/v_190325161/scanner/
Chmod 777/home/acunetix/.acunetix_trial/v_190325161/scanner/patch_awvs
/home/acunetix/.acunetix_trial/v_190325161/scanner/patch_awvs
```

![](<../../.gitbook/assets/image (1007).png>)

6、替换license文件

```
chattr +i /home/acunetix/.acunetix_trial/data/license/license_info.json
rm -rf /home/acunetix/.acunetix_trial/data/license/wa_tata.dat
touch /home/acunetix/.acunetix_trial/data/license/wa_tata.dat
chattr +i /home/acunetix/.acunetix_trial/data/license/wa_data.dat
```

![](<../../.gitbook/assets/image (2).png>)

7、破解成功后重启服务

```
sudo systemctl restart acunetix_trial.service
```

8、访问：https://注册时输入的网址:13443/#/login/

![](<../../.gitbook/assets/image (900).png>)

&#x20;  再输入注册时的邮箱和密码

![](file:///C:/Users/73604/AppData/Local/Temp/msohtmlclip1/01/clip_image052.jpg)9、打开后没提示注册，破解成功

![](<../../.gitbook/assets/image (90).png>)

### 3、使用教程

1、创建扫描目标，点击Address，在输入待测网址，\[http://www.testfire.net（一个漏洞网站）]点击Add Target



![](<../../.gitbook/assets/image (1073).png>)

2、选中待测网址，点击Scan

![](<../../.gitbook/assets/image (1034).png>)

![](file:///C:/Users/73604/AppData/Local/Temp/msohtmlclip1/01/clip_image058.jpg)3、扫描结果 ![](file:///C:/Users/73604/AppData/Local/Temp/msohtmlclip1/01/clip_image060.jpg)

![](<../../.gitbook/assets/image (332).png>)
