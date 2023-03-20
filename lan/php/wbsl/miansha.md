# Webshell免杀研究

## 前言

不想当将军的士兵不是好士兵，不想getshell的Hacker不是好Hacker\~\
&#x20;有时候我们在做攻防对抗时经常会碰到可以上传webshell的地方，但是经常会被安全狗、D盾、护卫神、云锁等安全软件查杀，在本篇文章中将会介绍一些常用的木马免杀技巧，主要针对安全狗、护卫神、D盾进行免杀\~

## 查杀软件

### D盾

D盾是一个专门为IIS设计的主动防御的安全性保护软件，它采用以内外防护的方式防止服务器和网站被人入侵，它秉持在正常运行各类网站的情况下，越少的功能，服务器就越安全的理念而设。它具有一句话木马查杀、主动后门拦截、Session保护、CC攻击防御、网页篡改查询、WEB嗅探防御、SQL注入防御、XSS攻击防御、提权防御、恶意文件上传防御、未知0Day防御等特性。\
&#x20;

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128144530-c656dcf8-4199-1.png)

### 安全狗

安全狗是一款安全防护软件，它提供木马查杀、漏洞防御、非法请求拦截等功能，致力于保护网站和服务器的安全。它具有应用防护WEB应用风险、拦截各类SQL注入、XSS攻击防御、0Day攻击防御、特定资源保护、网络木马文件、恶意畸形文件、0Day漏洞、黑链等特性。\
&#x20;

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128144645-f360a74c-4199-1.png)

### 护卫神

护卫神是一款以分离权限为基础，通过一系列独特技术手段，保障网站不被入侵的安全性防护软件。它适用于各类ASP和PHP编写的程序，在目前网站日益猖狂的挂马、入侵情况下，护卫神可以彻底解决用户所面临的众多安全难题，为网络安全保驾护航。 它具有实时木马程序查杀、网站挂马拦截、文件篡改保护、PHP拒绝服务攻击防御、SQL防御、XSS跨站攻击防御、入侵拦截防护、远程桌面安全保护等特性。\
&#x20;

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128144735-10fa1590-419a-1.png)

## 免杀基础

### 免杀需求

因为设计的木马的最终目的在发现目标网站有上传漏洞时可以将木马上传到目标服务器上并且可以远程访问实现远程控制，然而一些网站都会有安全狗、D盾、安骑士、护卫神、云锁等防护软件可以对一些Webshell进行查杀，那么要想使用Webshell进行远控就需要实现免杀，以此来躲避木马查杀工具的检查。

### 查杀技术

目前主流的木马查杀方法有：静态检查、动态检测、日志检查三种方式。\
&#x20;a、静态检查通过匹配特征码、危险函数和木马特征值来查杀木马程序，它的特点是快速方便，对已知的木马程序查找准确率较高，它的缺点是误报率较高，无法查找0Day型的木马程序，而且容易被绕过。\
&#x20;b、动态检测通过木马程序的动态特征来检测，当木马程序被上传到服务器上后，攻击者总会去执行它，当木马程序被执行时所表现出来的特征就是所谓的动态特征。\
&#x20;c、日志检测则主要通过日志分析检测技术来实现，它主要通过分析大量的日志文件并建立请求模型来检测出异常文件。它的优点为当网站上的访问量级达到一致值时，这种检测方法具有比较大参考性价值。它的缺点则是存在一定误报率，对于大量的日志文件，检测工具的处理能力和效率都会变的比较低。

### 免杀技巧

木马程序可以使用多种编程语言来设计，不同的编程语言有不同特性以及提供的系统函数，所以在实现免杀时可以首先考虑灵活运用语言的特性来实现免杀，其次可以根据查杀软件的查杀规则来重构木马程序，躲避木马查杀工具的查杀，同时可以考虑密码学中的加密解密对源木马程序进行加密解密处理，以此来躲避木马查杀工具的检查。木马免杀技术的核心在于“灵活多变”。

## 免杀实战—小马免杀

### 引用免杀

因为D盾、安全狗、护卫神会对关键字eval中的执行变量进行溯源，当追溯到要执行的变量为一个通过POST接收的可疑数据时就会显示可疑木马，为了躲避这种溯源方式，可以通过多次使用&来引用前一个变量，通过一连串的赋值操作最后将要执行的内容与反引号拼接后传入eval实现免杀，具体实现如下所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145159-ae73c334-419a-1.png)

&#x20;之后使用D盾查杀一下看看：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145418-00f513b0-419b-1.jpeg)

&#x20;发现还是被查杀到了，这时候b有给a说了："喂，上面的a，我们换换位置呗\~"，于是a无奈的说："为啥受伤的总是我呢？"

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145526-29a2d2d4-419b-1.jpeg)

之后a对b说："你看，不行吧？换了也没戏"，之后b皱了皱眉说："不急，我们来再拉一下伙伴——反引号，让他帮帮忙"：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145620-49c4de68-419b-1.jpeg)

果然，b小哥说的还是有点门道的哦，从上面的结果中可以看到成功免杀了，之后再来看看安全狗——成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145706-6599c018-419b-1.jpeg)

护卫神——成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145733-7565c1cc-419b-1.jpeg)

至此，D盾、安全狗、护卫神已成功免杀，之后我们来看看可用性：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145801-85e27e00-419b-1.jpeg)

### 可变变量

可变变量是PHP中一种较为独特的变量，它可以动态的改变一个变量的名称，这种特性可以用于木马免杀中。首先可以定义一个变量$do并为其赋值为todo,之后将木马内容赋值给可变变量\$$do,最后在调用eval函数执行时将执行对象定义为$todo即可，具体实现如下所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145831-9803d3cc-419b-1.jpeg)

之后使用D盾查杀一下看看：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145857-a784061e-419b-1.jpeg)

发现不行哦，那么怎么办呢？幸好今天"反引号"大哥来串门，不妨让他来帮个忙：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145925-b7f77bde-419b-1.jpeg)

发现成功免杀，之后我们再使用安全狗查杀一下看看————成功免杀\
&#x20;

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128145951-c808a4bc-419b-1.jpeg)

护卫神————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150017-d7599c82-419b-1.jpeg)

至此，成功免杀安全狗、护卫神、D盾，之后我们试试可用性：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150042-e5f725a2-419b-1.jpeg)

### 二维数组

在免杀时我们可以考虑见要执行的一句话木马程序放到数组中执行达到绕过的目的，例如：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150123-fea76f08-419b-1.jpeg)

之后我们使用D盾进行查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150153-102fb9ec-419c-1.jpeg)

之后我们使用安全狗进行查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150220-204b0c78-419c-1.jpeg)

之后我们使用护卫神进行查杀：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150248-317c61fe-419c-1.jpeg)

oh, My God！护卫神这么强悍的吗？？？好不容易过了D盾、安全狗的查杀检测，走到最后一步却被护卫神给查杀了.....，那么怎么绕呢？既然一维数组不行，那么我就来个二维数组，哼：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150317-42b7fd5c-419c-1.jpeg)

之后再次使用护卫神查杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150343-52076068-419c-1.jpeg)

还是被查杀到了，那么怎么办呢？变量$a给变量$b说："我们不妨换换位置?"，变量$b只好带着"试一试"的心态回复"换就换呗，反正就那样....":

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150417-661b2698-419c-1.jpeg)

&#x20;"竟然成功免杀了？"，变量$b带着痴呆的表情说到。\
&#x20;之后我们再次使用D盾进行查杀————成功免杀：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150448-78eb79f8-419c-1.jpeg)

之后再次使用安全狗查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150513-879422a2-419c-1.jpeg)

至此，成功免杀D盾、安全狗、护卫神，那么来看看可用性：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150534-93ff7028-419c-1.jpeg)

### 数组交集

在做免杀研究是，发现我们可以通过数组的交集来获得我们想要的值，之后将其利用到木马程序的构造当中，例如：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150605-a6797ac8-419c-1.jpeg)

至于设计原理上面的注释中想必已经说得很是详细了，这里就不再一一复述了，下面我们使用D盾进行查杀看看——免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150627-b3d94e8c-419c-1.jpeg)

之后使用安全狗查杀看看————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150657-c5a6ddc8-419c-1.jpeg)

之后使用护卫神查杀看看————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150727-d751beda-419c-1.jpeg)

至此，D盾、安全狗、护卫神成功免杀，下面试试可用性：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150749-e4ca3d80-419c-1.jpeg)

### 回调函数

array\_map() 回调函数会返回用户自定义函数作用后的数组。array\_map() 函数具体使用方法和参数说明如下：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150831-fd723720-419c-1.jpeg)

在这里我们可以先定义一个函数test，其中第一个参数$a用作回调函数名称，第二个参数$b用作回调函数的参数，之后将其传递给array\_map()函数进行执行，之后我们在外部调用test函数，同时传入我们的回调函数名称和回调函数的参数：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150856-0c4c48b2-419d-1.jpeg)

之后使用D盾进行查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150923-1cb7d450-419d-1.jpeg)

之后使用安全狗查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128150948-2b5f3d5e-419d-1.jpeg)

之后使用护卫神查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151017-3cd1744e-419d-1.jpeg)

至此，成功免杀D盾、安全狗、护卫神，之后我们使用菜刀连接试试看是否可以正常使用：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151100-56bb7184-419d-1.jpeg)

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151106-5a432c16-419d-1.jpeg)

## 免杀实战—大马免杀

### 加密&混淆

在免杀处理的众多方法中，加密免杀算是一种常用的技巧，常见的加密方式有rot13、base64加解密，下面我们使用base64来进行免杀研究，首先我们需要一个shell.php的PHP大马：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151202-7b67ddb0-419d-1.jpeg)

之后我们需要使用encode.php对上面的大马程序进行一次base64加密处理，encode.php代码如下：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151235-8efaccb6-419d-1.jpeg)

之后我们在浏览器中访问encode.php即可实现对shell.php大马程序的加密处理：

&#x20;PS:因为木马程序源代码中含有很多敏感的操作，而且有很多容易被查杀到的关键词，所以后续的免杀都是在加密处理的基础上进行的分析与研究\
&#x20;之后我们使用D盾先来一波查杀看看：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151312-a53d4418-419d-1.jpeg)

\
&#x20;\
&#x20;

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151408-c64b5618-419d-1.jpeg)

从上面的查杀结果可以看到这里威胁级别为"5"，而且报"加密后门"的警告，这里应该是D盾检测到了关键字"base64\_decode"，所以我们这里需要做一个简单的混淆处理：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151441-d9f05e34-419d-1.jpeg)

通过以上处理之后发现处理后的木马程序躲避了D盾的查杀，但是被安全狗检测到了：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151508-ea460892-419d-1.jpeg)

经过分析发现被查杀的原因是eval函数执行了一个解密后的内容，为了躲避查杀，这里可以通过将解密后的内容赋值给一个变量，之后通过使用反引号拼接变量然后再让eval去执行的方式躲避查杀，具体实现如下所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151539-fd0430da-419d-1.jpeg)

&#x20;可以发现成功免杀，之后我们再使用D盾进行一次查杀操作，发现依旧成功免杀（毕竟大马程序的复杂度增加了）：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151610-0f7f7666-419e-1.jpeg)

之后我们再使用护卫神进行一次查杀：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151638-1fe41fca-419e-1.jpeg)

至此该木马成功免杀D盾、安全狗、护卫神，同时我们需要检测一下免杀之后的可用性：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151733-40f3910a-419e-1.jpeg)

可以正常使用：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151807-5538083a-419e-1.jpeg)

### Create\_function免杀

在免杀的过程中，发现了一个PHP的内置函数Create\_function，它主要用于创建一个函数，这里可以使用它来进行免杀，但是由于D盾、安全狗有关键词查杀所以这里需要对Create\_function进行一个拆分处理，同时需要加入混淆处理，最后木马程序重构结果如下所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151842-6a216804-419e-1.jpeg)

之后使用D盾进行查杀————成功免杀！

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151909-7a3fb2b8-419e-1.jpeg)

之后使用安全狗查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128151938-8b893c10-419e-1.jpeg)

之后使用护卫神进行查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152006-9c1342e2-419e-1.jpeg)

至此，安全狗、护卫神、D盾成功免杀，之后我们试试可用性：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152031-ab1b94e2-419e-1.jpeg)

可以正常使用\~

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152100-bbd6be24-419e-1.jpeg)

### 可变变量

可变变量是PHP中一种较为独特的变量，它可以动态的改变一个变量的名称，这种特性可以用于木马免杀中。首先可以定义一个变量$do并为其赋值为todo,之后将加密处理过后的木马内容赋值给可变变量\$$do,最后在调用eval函数执行时将执行对象定义为$todo即可，具体实现如下所示：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152135-d0b9de0c-419e-1.jpeg)

之后使用D盾进行查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152205-e2a4692a-419e-1.jpeg)

之后使用安全狗查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152230-f18ffb02-419e-1.jpeg)

之后使用护卫神查杀————成功免杀

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152258-026f32c6-419f-1.jpeg)

至此，成功免杀D盾、安全狗、护卫神，之后我们试试可用性：

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152343-1d1468da-419f-1.jpeg)

![](https://xzfile.aliyuncs.com/media/upload/picture/20200128152320-0faabf32-419f-1.jpeg)

## 总结

免杀与查杀在一次又一次的攻防较量中不断的进步，而我们在使用现有的webshell时也需要留意该webshell的可信程度，有些webshell留有后门，至少笔者分析的N多个大马时发现加密的木马文件几乎都有相关的后门。所以在免杀研究时还是自我设计木马程序为好，一些大马文件的功能不外乎由编程语言的功能函数来实现外加各种加密解密、编码/解码方法等。当然，随着木马查杀库的更新我们也需要研究更多的木马查杀方法与木马查杀机制的缺陷，促进攻防两端的进步。
