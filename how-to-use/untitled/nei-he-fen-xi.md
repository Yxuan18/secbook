# 内核分析

## sqlmap 内核分析 I: 基础流程

在本系列文章中，我们主要针对 sqlmap 的最核心的方方面面进行分析，本文主要针对基础流程进行介绍与描述，本文由非常细致的 sqlmap 源码解读，希望有需要的读者可以从中受益。

### 0x00 准备工作

想要阅读 sqlmap 源码我相信大家的选择肯定更多的是从 github 下直接 clone 代码到本地，直接使用本地编辑器或者 IDE 打开直接来分析。所以基本操作也就是

```text
git clone https://github.com/sqlmapproject/sqlmap
cd sqlmap
```

进入 sqlmap 的 repos 下，直接打开编辑器吧！

当然很多读者是 Python3 用户，其实也没有必要费很大力气在本机上安装 Python2 然后再进行操作。笔者使用的环境是

* Mac OS X
* Pyenv
* VSCode

推荐使用 Pyenv（+virtualenv） 构建 Python 环境运行 sqlmap。

### 0x01 初始化与底层建筑

笔者当然可以直接指出所有的重要逻辑在什么位置，但是这样并不好。这样做的后果就是大家发出奇怪的疑问：

> 它里面 conf 和 kb 又是啥？这两个全局变量里面到底存了啥？

逐步熟悉整个项目的构建和项目中贯穿全局的两个奇怪的全局变量，对于加速理解 sqlmap 的核心逻辑起了很大的作用。在笔者的工作和实践中，确实是很有感触。

所以我们还是从头看起吧！

![img](https://p4.ssl.qhimg.com/t018001bdb5e9d83cb2.jpg)

我们在上图中，可以找到很明显的程序命令行入口，我们暂且只分析命令行入口所以，我把无关的东西全部打了马赛克，所以接下来我们看到 `main` 函数直接来了解

![img](https://p3.ssl.qhimg.com/t014a5639084b0fa84e.jpg)

我相信大家看到了上图应该就知道我们主要应该看 `try` 中的内容。实际上 `except` 中指的是 sqlmap 中各种各样异常处理，包含让程序退出而释放的异常/用户异常以及各种预期或非预期异常，在 `finally` 中，大致进行了数据库（HashDB）的检查/恢复/释放以及 `dumper` 的收尾操作和多线程的资源回收操作。具体的不重要的代码我们就不继续介绍了，接下来直接来了解比较重要的部分吧。

![img](https://p3.ssl.qhimg.com/t019c4f045cdcf3d5ed.jpg)

在实际在工作部分中，我们发现了 1-4 函数对环境和基础配置进行了一同操作，然后在 5 步骤的时候进行步骤初始化，然后开始启动 sqlmap。实际上这些操作并不是一无是处，接下来有详有略介绍这些步骤究竟发生了什么。

1. 在 DirtyPatches 中，首先设定了 httplib 的最大行长度（`httplib._MAXLINE`），接下来导入第三方的 windows 下的 ip地址转换函数模块（`win_inet_pton`），然后对编码进行了一些替换，把 `cp65001` 替换为 `utf8` 避免出现一些交互上的错误，这些操作对于 sqlmap 的实际功能影响并不是特别大，属于保证起用户体验和系统设置的正常选项，不需要进行过多关心。
2. 在环境检查中，做了如下操作：检查模块路径，检查 Python 版本，导入全局变量。我们可能并不需要关心太多这一步，只需要记得在这一步我们导入了几个关键的全局变量：`("cmdLineOptions", "conf", "kb")`，需要提醒大家的是，直接去 `lib.core.data` 中寻找这几个变量并不是明智的选择，因为他们并不是在这里初始化的（说白了就是找到了定义也没有用，只需要知道有他们几个就够啦）。
3. 初始化各种资源文件路径。
4. 打印 Banner。
5. 这一部分可以说是非常关键了，虽然表面上仍然是属于初始化的阶段，但是实际上，如果不知晓这一步，面对后面的直接对全局变量 `kb` 和 `conf` 的操作将会变的非常奇怪和陌生。在这步中，我们进行了配置文件初始化，知识库（KnowledgeBase初始化）以及用户操作的 `Merge` 和初始化。我们在之后的分析中如果遇到了针对 `kb` 和 `conf` 的操作，可以直接在这个函数对应的 `lib.core.option` 模块中寻找对应的初始化变量的定义。当然，这一步涉及到的一些 `kb/conf` 的 fields 也可能来源于 `lib.parse.cmdline` 中，可以直接通过 `ctrl+F` 搜索到

![img](https://p0.ssl.qhimg.com/t01e93760276299cfdb.jpg)

1. 中主要包含所有初始变量的初始值，这些初始值在 `init()` 的设定主要是引用各种各样的函数来完成基础设置，我们没有必要依次对其进行分支，只需要用到的时候知道回来寻找就可以了。
2. 冒烟测试，测试程序本身是否可以跑得通。
3. 功能测试，测试 sqlmap 功能是否完整。

进入上一段代码的条件是 `if not conf.updateAll`，这个是来源于 `lib.parse.cmdline` 中定义的更新选项，如果这个选项打开，sqlmap 会自动更新并且不会执行后续测试步骤和实际工作的步骤。

![img](https://p5.ssl.qhimg.com/t0157fdefaf1100b316.jpg)

在实际的启动代码中，笔者在上图中标注了两处，我们在使用命令行的时候，更多的是直接调用 start\(\) 函数，所以我们直接跟入其中寻找之后需要研究的部分。

### 0x02 测试前的目标准备

当我们找到 start\(\) 函数的时候，映入眼帘的实际上是一个很平坦的流程，我们简化一下，以下图代码为例：

![img](https://p0.ssl.qhimg.com/t013a0fd36bbf76ba5c.jpg)

我们仍然看到了 `conf` 中一些很奇怪的选项，针对这些选项我们在 0x01 节中强调过，可以在某一些地方找到这些选项的线索，我们以 `conf.direct` 为例，可以在 `lib.parse.cmdline` 中明确找到这个选项的说明：

![img](https://p1.ssl.qhimg.com/t013e21ed52df49ac35.jpg)

根据说明，这是直连数据库的选项，所以我们可能暂时并不需要关心他，我们暂时只关注 sqlmap 是如何检测漏洞的，而不关心他是怎么样调用数据库相关操作的。

接下来稍有一些想法的读者当然知道，我们直接进行第四部分针对这个目标循环的分析是最简单有效的办法了！

好的，接下来我们就打开最核心的检测方法：

![img](https://p0.ssl.qhimg.com/t01598be66a050886f2.jpg)

进入循环体之后，首先进行检查网络是否通断的选项，这个选项很容易理解我们就不多叙述了；确保网络正常之后，开始设置 `conf.url,conf.method,conf.data,conf.cookie` 和 headers 等字段，并且在 `parseTargetUrl()` 中进行各种合理性检查；之后会根据 HTTP 的 Method 提取需要检查的参数；随后如果当前启动时参数接受了多个目标的话，会在第4步中做一些初始化的工作。

在完成上述操作之后，执行 `setupTargetEnv()` 这个函数也是一个非常重要的函数，其包含如下操作：

```text
def setupTargetEnv():
    _createTargetDirs()
    _setRequestParams() 
    _setHashDB()
    _resumeHashDBValues()
    _setResultsFile()
    _setAuthCred()
```

其中除了 `setRequestParams()` 都是关于本身存储（缓存）扫描上下文和结果文件的。当然我们最关注的点肯定是 `setRequestParams()` 这个点。在深入了解这一个步骤之后，我们发现其中主要涉及到如下操作：

![img](https://p2.ssl.qhimg.com/t0106a5f71c992237ef.jpg)

![img](https://p5.ssl.qhimg.com/t01141e534618cf6480.jpg)

所以我们回归之前的 `start()` 方法中的 foreach targets 的循环体中，在 `setupTargetEnv()` 之后，我们现在已经知道了关于这个目标的所有的可以尝试注入测试的点都已经设置好了，并且都存在了 `conf.paramDict` 这个字典中了。

至此，在正式开始检测之前，我们已经知道，`conf.url, conf.method, conf.headers ...` 之类的包含基础的测试的目标的信息，在 `conf.paramDict` 中包含具体的不同位置的需要测试的参数的字典，可以方便随时渲染 Payload。关于其具体的行为，其实大可不必太过关心，因为我们其实并不需要具体的处理细节，这些细节应该是在我们遇到问题，或者遇到唔清楚的地方再跳出来在这些步骤中寻找，并且进行研究。

### 0x03 万事俱备

可以说在读者了解上面两节讲述的内容的时候，我们就可以正式探查真正的 SQL 注入检测时候 sqlmap 都坐上了什么。其实简单来说，需要经过下面步骤：

![img](https://p3.ssl.qhimg.com/t01b5e58e7099401eff.jpg)

笔者通过对 `controller.py` 中的 `start()` 函数进行分析，得出了上面的流程图。在整个检测过程中，我们暂且不涉及细节；整个流程都是针对检查一个目标所要经历的步骤。

**checkWaf**

在 `checkWaf()` 中，文档写明：`Reference: http://seclists.org/nmap-dev/2011/q2/att-1005/http-waf-detect.nse`，我们可以在这里发现他的原理出处，有兴趣的读者可以自行研究。在实际实现的过程中代码如下：

![img](https://p5.ssl.qhimg.com/t01e5e329ff7a650b90.jpg)

笔者在关键部分已经把标注和箭头写明，方便大家理解。我们发现 `payload` 这个变量是通过随机一个数字 + space + 一个特制 Payload（涉及到很多的关于敏感关键词，可以很容易触发 WAF 拦截）。

随即，sqlmap 会把 payload 插入该插入的位置：对于 GET 类的请求，sqlmap 会在之前的 query 语句后面加入一个新的参数，这个参数名通过 `randomStr()` 生成，参数的值就是经过处理的 Payload。如果有读者不理解，我们在这里可以举一个例子：

如果我们针对

```text
http://this.is.a.victim.com/article.php?id=1
```

这样的 URL 进行 Waf 的检查，sqlmap 会发起一个

```text
http://this.is.a.victim.com/article.php?id=1&mbjwe=2472%20AND%201%3D1%20UNION%20ALL%20SELECT%201%2CNULL%2C%27%3Cscript%3Ealert%28%22XSS%22%29%3C/script%3E%27%2Ctable_name%20FROM%20information_schema.tables%20WHERE%202%3E1--/%
2A%2A/%3B%20EXEC%20xp_cmdshell%28%27cat%20../../../etc/passwd%27%29%23
```

的新的请求，这个请求会有很大概率触发 Waf 的反应，然后 sqlmap 通过判断返回页面和之前页面的 Page Ratio 来判断是否触发了 WAF。

**我们似乎遇到一些问题**

有心的读者可能发现，我们在上小节出现了一个神奇陌生的词 Page Ratio, 这个词其实在整个 sqlmap 中是非常重要的存在，我们之后会在后续的文章中详细介绍这部分理论。

### 0x04 然后呢

其实我们当然可以继续讲解每一个函数都做了什么，但是限于篇幅问题，我们可能要先暂停一下了；与此同时，我们本文的内容“基础流程”实际上已经介绍完了，并且引出了我们需要在下一篇文章介绍的概念之一“Page Ratio”。

所以接下来我们可能要结束本文了，但是我更希望的是，每一个读者都能够尝试自己分析，自己去吃透 sqlmap 的细节。

## sqlmap 内核分析 II: 核心原理-页面相似度算法实践

### 0x00 PageRatio 是什么？

要说 PageRatio 是什么，我们可能需要先介绍另一个模块 `difflib`。这个模块是在 sqlmap 中用来计算页面的相似度的基础模块，实际处理的时候，sqlmap 并不仅仅是直接计算页面的相似度，而是通过首先对页面进行一些预处理，预处理之后，根据预设的阈值来计算请求页面和模版页面的相似度。

对于 `difflib` 模块其实本身并没有什么非常特殊的，详细参见[官方手册](https://link.zhihu.com/?target=https%3A//docs.python.org/2/library/difflib.html)，实际在使用的过程中，sqlmap 主要使用其 `SequenceMatcher` 这个类。以下是关于这个类的简单介绍：

> This is a flexible class for comparing pairs of sequences of any type, so long as the sequence elements are hashable. The basic algorithm predates, and is a little fancier than, an algorithm published in the late 1980’s by Ratcliff and Obershelp under the hyperbolic name “gestalt pattern matching.” The idea is to find the longest contiguous matching subsequence that contains no “junk” elements \(the Ratcliff and Obershelp algorithm doesn’t address junk\). The same idea is then applied recursively to the pieces of the sequences to the left and to the right of the matching subsequence. This does not yield minimal edit sequences, but does tend to yield matches that “look right” to people.

简单来说这个类使用了 Ratcliff 和 Obershelp 提供的算法，匹配最长相同的字符串，设定无关字符（junk）。在实际使用中，他们应用最多的方法应该就是 `ratio()`。

![img](https://p5.ssl.qhimg.com/dm/1024_90_/t01e2a0acd93ceb40b9.jpg)

根据文档中的描述，这个方法返回两段文本的相似度，相似度的算法如下：我们假设两段文本分别为 `text1` 与 `text2`，他们相同的部分长度总共为 `M`，这两段文本长度之和为 `T`，那么这两段文本的相似度定义为 `2.0 * M / T`，这个相似度的值在 0 到 1.0 之间。

**PageRatio 的小例子**

![img](https://p2.ssl.qhimg.com/t01d1126ad273ec29e3.jpg)

我们通过上面的介绍，知道了对于 `abcdefg` 和 `abce123` 我们计算的结果应该是 `2.0 * 4 / 14`所以计算结果应该是：

![img](https://p3.ssl.qhimg.com/t0181ed8cf9e630822a.jpg)

到现在我们理解了 PageRatio 是什么样的一种算法，我们就可以开始观察 sqlmap 是如何使用这一个值的了～

### 0x01 RATIO in checkWaf

在上节的内容中，我们对于 sqlmap 的源码了解到 `checkWaf` 的部分，结合刚才讲的 PageRatio 的例子，我们直接可以看懂这部分代码：

![img](https://p3.ssl.qhimg.com/dm/1024_160_/t01ad19d3201908ae17.jpg)

现在设定 `IDS_WAF_CHECK_RATIO = 0.5` 表明，只要打了检测 IDS/WAF 的 Payload 的页面结果与模版页面结果文本页面经过一定处理，最后比较出相似度相差 0.5 就可以认为触发了 IDS/WAF。

与 `checkWaf` 相关的其实还有 `identityWaf`, 但是这个方法太简单了我们并不想仔细分析，有兴趣的读者可以自行了解一下，本文选择直接跳过这一个步骤。

### 0x02 checkStability

这个函数其实是在检查原始页面是存在动态内容，并做一些处理。何为动态内容？在 sqlmap 中表示以同样的方式访问量次同一个页面，访问前后页面内容并不是完全相同，他们相差的内容属于动态内容。当然，sqlmap 的处理方式也并不是随意的比较两个页面就没有然后了，在比较完之后，如果存在动态页面，还会做一部分的处理，或者提出扩展设置（`--string/--regex`），以方便后续使用。

![img](https://p3.ssl.qhimg.com/dm/1024_675_/t01ab4e4d5b1ac875b9.jpg)

我们发现，实际的 sqlmap 源码确实是按照我们介绍的内容处理的，如果页面内容是动态的话，则会提示用户处理字符串或者增加正则表达式来验证页面。

> 默认情况下sqlmap通过判断返回页面的不同来判断真假，但有时候这会产生误差，因为有的页面在每次刷新的时候都会返回不同的代码，比如页面当中包含一个动态的广告或者其他内容，这会导致sqlmap的误判。此时用户可以提供一个字符串或者一段正则匹配，在原始页面与真条件下的页面都存在的字符串，而错误页面中不存在（使用–string参数添加字符串，–regexp添加正则），同时用户可以提供一段字符串在原始页面与真条件下的页面都不存在的字符串，而错误页面中存在的字符串（–not-string添加）。用户也可以提供真与假条件返回的HTTP状态码不一样来注入，例如，响应200的时候为真，响应401的时候为假，可以添加参数–code=200。

**checkDynamicContent\(firstPage, secondPage\)**

我们发现，如果说我们并没指定 `string / regex` 那么很多情况，我们仍然也可以正确得出结果；根据 sqlmap 源码，它实际上背后还是有一些处理方法的，而这些方法就在 `checkDynamicContent(firstPage, secondPage)` 中：

![img](https://p1.ssl.qhimg.com/dm/1024_603_/t01c3417e69abd25ece.jpg)

我们在这个函数中发现如果 `firstPage 和 secondPage` 的相似度小于 0.98 （这个相似度的概念就是前一节介绍的 PageRatio 的概念），则会重试，并且尝试 `findDynamicContent(firstPage, secondPage)` 然后细化页面究竟是 `too dynamic` 还是 `heavily dynamic`。

如果页面是 `too dynamic` 则提示启用 `--text-only` 选项：

> 有些时候用户知道真条件下的返回页面与假条件下返回页面是不同位置在哪里可以使用–text-only（HTTP响应体中不同）–titles（HTML的title标签中不同）。

如果页面仅仅是显示 `heavy dynamic` 的话，sqlmap 会不断重试直到区分出到底是 `too dynamic`还是普通的可以接受的动态页面（相似度大于 0.98）。

对于 `too dynamic` 与可以接受的动态页面（相似度高于 0.98），其实最根本的区别就是在于 PageRatio, 如果多次尝试（超过 conf.retries） 设置的尝试次数，仍然出现了相似度低于 0.98 则会认为这个页面 `too dynamic`。

**findDynamicContent\(firstPage, secondPage\)**

这个函数位于 `common.py` 中，这个函数作为通用函数，我们并不需要非常严格的去审他的源码，为了节省大家的时候，笔者在这里可以描述这个函数做了一件什么样的事情，并举例说明。

这个函数按函数名来解释其实是，寻找动态的页面内容。

实际在工作中，如果寻找到动态内容，则会将动态内容的前后内容（前：`prefix`，后：`suffix`，长度均在 `DYNAMICITY_BOUNDARY_LENGTH` 中设定，默认为 20）作为一个 tuple，存入 `kb.dynamicMarkings`，在每一次页面比较之前，会默认移除这些动态内容。

```text
kb.dynamicMarkings.append((prefix if prefix else None, suffix if suffix else None))
```

例如，在实际使用中，我们按照官方给定的一个例子：

```text
    """
    This function checks if the provided pages have dynamic content. If they
    are dynamic, proper markings will be made
                          
    >>> findDynamicContent("Lorem ipsum dolor sit amet, congue tation referrentur ei sed. Ne nec legimus habemus recusabo, natum reque et per. Facer tritani reprehendunt eos id, modus constituam est te. Usu sumo indoctum ad, pri paulo molestiae complectitur no.", 
                           "Lorem ipsum dolor sit amet, congue tation referrentur ei sed. Ne nec legimus habemus recusabo, natum reque et per. <script src='ads.js'></script>Facer tritani reprehendunt eos id, modus constituam est te. Usu sumo indoctum ad, pri paulo molestiae complectitur no.")
    >>> kb.dynamicMarkings
    [('natum reque et per. ', 'Facer tritani repreh')]
    """
```

根据观察，两段文本差别在 `script` 标签，标记的动态内容应该是 `script` 标签，所以动态内容的前 20 字节的文本座位 `prefix` 后 20 字节的文本作为 `suffix`，分别为：

* prefix: `'natum reque et per. '`
* suffix: `'Facer tritani repreh'`

### 0x03 中场休息与阶段性总结

我们虽然之分析了两个大函数，但是整个判断页面相应内容的核心原理应该是已经非常清晰了；可能有些读者反馈我们的进度略慢，但是其实这好比一个打基础的过程，我们基础越扎实对 sqlmap 越熟悉，分析后面的部分就越快。

为了更好的继续，我们需要回顾一下之前的流程图

![img](https://p2.ssl.qhimg.com/t017f96952321bca736.jpg)

好的，接下来我们的目标就是图中描述的部分“过滤重复以及不需要检查的参数，然后检查参数是为动态参数”，在下一篇文章中，我们将会详细介绍 sqlmap 其他的核心函数，诸如启发式检测，和 sql 注入检测核心函数。

### 0x04 参数预处理以及动态参数检查

**参数预处理**

参数预处理包含如下步骤：

参数排序

```text
# Order of testing list (first to last)
orderList = (PLACE.CUSTOM_POST, PLACE.CUSTOM_HEADER, PLACE.URI, PLACE.POST, PLACE.GET)
​
for place in orderList[::-1]:
if place in parameters:
parameters.remove(place)
parameters.insert(0, place)
参数分级检查for place in parameters:
# Test User-Agent and Referer headers only if
# --level >= 3
skip = (place == PLACE.USER_AGENT and conf.level < 3)
skip |= (place == PLACE.REFERER and conf.level < 3)
​
# Test Host header only if
# --level >= 5
skip |= (place == PLACE.HOST and conf.level < 5)
​
# Test Cookie header only if --level >= 2
skip |= (place == PLACE.COOKIE and conf.level < 2)
​
skip |= (place == PLACE.USER_AGENT and intersect(USER_AGENT_ALIASES, conf.skip, True) not in ([], None))
skip |= (place == PLACE.REFERER and intersect(REFERER_ALIASES, conf.skip, True) not in ([], None))
skip |= (place == PLACE.COOKIE and intersect(PLACE.COOKIE, conf.skip, True) not in ([], None))
skip |= (place == PLACE.HOST and intersect(PLACE.HOST, conf.skip, True) not in ([], None))
​
skip &= not (place == PLACE.USER_AGENT and intersect(USER_AGENT_ALIASES, conf.testParameter, True))
skip &= not (place == PLACE.REFERER and intersect(REFERER_ALIASES, conf.testParameter, True))
skip &= not (place == PLACE.HOST and intersect(HOST_ALIASES, conf.testParameter, True))
skip &= not (place == PLACE.COOKIE and intersect((PLACE.COOKIE,), conf.testParameter, True))
​
if skip:
continue
​
if kb.testOnlyCustom and place not in (PLACE.URI, PLACE.CUSTOM_POST, PLACE.CUSTOM_HEADER):
continue
​
if place not in conf.paramDict:
continue
​
paramDict = conf.paramDict[place]
​
paramType = conf.method if conf.method not in (None, HTTPMETHOD.GET, HTTPMETHOD.POST) else place
```

参数过滤

![img](https://p4.ssl.qhimg.com/dm/1024_500_/t017da87db63d7c8ed0.jpg)

**checkDynParam\(place, parameter, value\)**

我们进入 `checkDynParam` 函数发现，整个函数其实看起来非常简单，但是实际上我们发现 `agent.queryPage` 这个函数现在又返回了一个好像是 Bool 值的返回值作为 `dynResult` 这令我们非常困惑，我们上一次见这个函数返回的是 `(page, headers, code)` 。

![img](https://p0.ssl.qhimg.com/dm/1024_609_/t01a1342a5f461cba86.jpg)

我们发现实际上的页面比较逻辑也并不是在 `checkDynParam` ，所以表面上，我们这一节的内容是在 `checkDynParam` 这个函数，但是实际上我们仍然需要跟进到 `agent.queryPage`。

那么，等什么呢？继续吧！

**agent.queryPage 与 comparison**

跟进 `agent.queryPage` 我相信一定是痛苦的，这其实算是 sqlmap 的核心基础函数之一，里面包含了接近三四百行的请求前预处理，包含 `tamper` 的处理逻辑以及随机化参数和 CSRF 参数的处理检测逻辑。同时如果涉及到了 `timeBasedCompare` 还包含着时间盲注的处理逻辑；除此之外，一般情况下 `agent.queryPage` 中还存在着针对页面比较的核心调用，页面对比对应函数为 `comparison`。为了简化大家的负担，笔者只截取最后返回值的部分 `agent.queryPage` 。

![img](https://p1.ssl.qhimg.com/dm/1024_268_/t01b8a3421162fca370.jpg)

在标注中，我们发现了我们之前的疑问，为什么 `agent.queryPage` 时而返回页面内容，时而返回页面与模版页面的比较结果。其实在于如果 `content/response` 被设置为 `True` 的时候，则会返回页面具体内容，headers，以及响应码；如果 `timeBasedCompare` 被设定的时候，返回是否发生了延迟；默认情况返回与模版页面的比较结果。

我们发现这一个 `comparison` 函数很奇怪，他没有输入两个页面的内容，而是仅仅输入当前页面的相关信息，但是为什么笔者要明确说是与“模版页面”的比较结果呢？我们马上就跟入 `comparison` 一探究竟。

![img](https://p1.ssl.qhimg.com/dm/1024_427_/t0155017907c959a1e7.jpg)

进去之后根据图中的调用关系，我们主要需要观察一下 `_comparison` 这个函数的行为。当打开这个函数的时候，我们发现也是一段接近一百行的函数，仍然是一个需要硬着头皮看下去的一段代码。

![img](https://p0.ssl.qhimg.com/dm/1024_541_/t01b3582899fe1b4ffb.jpg)

根据图中的使用红色方框框住的代码，我们很容易就能发现，这其实是在禁用 `PageRatio` 的页面相似度算法，而是因为用户设定了 `--string/--not-string/--regex/--code` 从而可以明确从其他方面区分出页面为什么不同。当然，我们的重点并不是他，而是计算 `ratio` 并且使用 `ratio`得出页面相似的具体逻辑。

![img](https://p1.ssl.qhimg.com/t0105b9475b9c4f27d0.jpg)

我相信令大家困惑的可能是这两段关于 `nullConnection` 的代码，在前面的部分中，我们没有详细说明 `nullConnection` 究竟意味着什么：

```text
  Optimization:
    These options can be used to optimize the performance of sqlmap
​
    -o                  Turn on all optimization switches
    --predict-output    Predict common queries output
    --keep-alive        Use persistent HTTP(s) connections
    --null-connection   Retrieve page length without actual HTTP response body
    --threads=THREADS   Max number of concurrent HTTP(s) requests (default 1)
```

根据官方手册的描述，`nullConnection` 是一种不用获取页面内容就可以知道页面大小的方法，这种方法在布尔盲注中有非常好的效果，可以很好的节省带宽。具体的原理详见[这一片古老的文章](https://link.zhihu.com/?target=http%3A//www.wisec.it/sectou.php%3Fid%3D472f952d79293)。

明白这一点，上面的代码就变得异常好懂了，如果没有启用 `--null-connection` 优化，两次比较的页面分别为 `page` 与 `kb.pageTemplate`。其实 `kb.pageTemplate` 也并不陌生，其实就是第一次正式访问页面的时候，存下的那个页面的内容。

```text
conf.originalPage = kb.pageTemplate = page
```

如果启用 `--null-connection`，计算 ratio 就只是很简单的通过页面的长度来计算，计算公式为

```text
ratio = 1. * pageLength / len(kv.pageTemplate)
​
if ratio > 1.:
    ratio = 1. / ratio
```

接下来我们再顺着他的逻辑往下走：

![img](https://p5.ssl.qhimg.com/dm/1024_500_/t0136e012785679de70.jpg)

根据上面对源码的标注，我们很容易理解这个 `ratio` 是怎么算出来的，同样我们也很清楚，其实并不只是简单无脑的使用 `ratio` 就可以起到很好的效果，配合各种各样的选项或者预处理：比如移除页面的动态内容，只比较 `title`，只比较文本，不比较 `html` 标签。

![img](https://p2.ssl.qhimg.com/dm/1024_425_/t01ff27d62d7e291fb5.jpg)

上面源码为最终使用 `ratio` 对页面的相似度作出判断的逻辑，其中

```text
UPPER_RATIO_BOUND = 0.98
LOWER_RATIO_BOUND = 0.02
DIFF_TOLERANCE = 0.05
```

### 0x05 结束语

阅读完本文，我相信读者对 sqlmap 中处理各种问题的细节都会有自己的理解，当然这是最好的。

在下一篇文章，笔者将会带大家进入更深层的 sqlmap 的逻辑，敬请期待。

## sqlmap 内核分析 III: 核心逻辑

### 0x00 前言

上一篇文章，我们介绍了页面相似度算法以及 sqlmap 对于页面相似的判定规则，同样也跟入了 sqlmap 的一些预处理核心函数。在接下来的部分中，我们会直接开始 sqlmap 的核心检测逻辑的分析，主要涉及到以下方面：

* heuristicCheckSqlInjection 启发式 SQL 注入检测（包括简单的 XSS FI 判断）
* checkSqlInjection SQL 注入检测

### 0x01 heuristicCheckSqlInjection

这个函数位于 controller.py 的 start\(\) 函数中，同时我们在整体逻辑中也明确指明了这一个步骤：

![img](https://p2.ssl.qhimg.com/t01d9f55ae956c95573.jpg)

这标红的两个步骤其实就是本篇文章主要需要分析的两个部分，涉及到 sqlmap 检测 sql 注入漏洞的核心逻辑。其中 heuristicCheckSqlInjection 是我们本节需要分析的问题。这个函数的执行位置如下：

![img](https://p0.ssl.qhimg.com/dm/1024_554_/t01c58d055a1e8cb9bd.jpg)

再上图代码中，2标号为其位置。

**启发式 sql 注入检测整体逻辑**

通过分析其源代码，笔者先行整理出他的逻辑：

![img](https://p3.ssl.qhimg.com/dm/1024_907_/t01653a50212801ccee.jpg)

根据我们整理出的启发式检测流程图，我们做如下补充说明。

1. 进行启发式 sql 注入检测的前提条件是没有开启 nullConnection 并且页面并不是 heavilyDynamic。关于这两个属性，我们在第二篇文章中都有相关介绍，对于 nullConnection 指的是一种不需要知道他的具体内容就可以知道整个内容大小的请求方法；heavilyDynamic 指的是，在不改变任何参数的情况下，请求两次页面，两次页面的相似度低于 0.98。
2. 在实际的代码中，决定注入的结果报告的，主要在于两个标识位，分别为：casting 与 result。笔者在下方做代码批注和说明：

![img](https://p2.ssl.qhimg.com/dm/1024_322_/t011a853a8ea1f21b2f.jpg)

1. casting 这个标识位主要取决于两种情况：第一种在第一个请求就发现存在了特定的类型检查的迹象；第二种是在请求小数情况的时候，发现小数被强行转换为整数。通常对于这种问题，在不考虑 tamper 的情况下，一般很难检测出或者绕过。
2. result 这个标识位取决于：如果检测出 DBMS 错误，则会设置这个标识位为 True；如果出现了数据库执行数值运算，也置为 True。

**XSS 与 FI**

实际上在启发式 sql 注入检测完毕之后，会执行其他的检测：

![img](https://p2.ssl.qhimg.com/dm/1024_542_/t019bb8b8bcbee8be9c.jpg)

1. 检测 XSS 的方法其实就是检查 “&lt;‘\”&gt;”，是否出现在了结果中。作为扩展，我们可以在此检查是否随机字符串还在页面中，从而判断是否存在 XSS 的迹象。
2. 检测 FI（文件包含），就是检测结果中是否包含了 include/require 等报错信息，这些信息是通过特定正则表达式来匹配检测的。

### 0x02 checkSqlInjection

这个函数可以说是 sqlmap 中最核心的函数了。在这个函数中，处理了 Payload 的各种细节和测试用例的各种细节。

大致执行步骤分为如下几个大部分：

1. 根据已知参数类型筛选 boundary
2. 启发式检测数据库类型 heuristicCheckDbms
3. payload 预处理（UNION）
4. 过滤与排除不合适的测试用例
5. 对筛选出的边界进行遍历与 payload 整合
6. payload 渲染
7. 针对四种类型的注入分别进行 response 的响应和处理
8. 得出结果，返回结果

下图是笔者折叠无关代码之后剩余的最核心的循环和条件分支，我们发现他关于 injectable 的设置完全是通过 if method == PAYLOAD.METHOD.\[COMPARISON/GREP/TIME/UNION\] 这几个条件分支去处理的，同时这些条件显然是 sqlmap 针对不同的注入类型的 Payload 进行自己的结果处理逻辑饿和判断逻辑。

![img](https://p0.ssl.qhimg.com/dm/1024_757_/t01d00f7b6bb3c35a95.jpg)

**数据库类型检测 heuristicCheckDbms**

我们在本大节刚开始的时候，就已经说明了第二步是确定数据库的类型，那么数据库类型来源于用户设定或者自动检测，当截止第二步之前还没有办法确定数据库类型的时候，就会自动启动 heuristicCheckDbms 这个函数，利用一些简单的测试来确定数据库类型。

![img](https://p1.ssl.qhimg.com/dm/1024_543_/t01a2ee6b72107acc8b.jpg)

其实这个步骤非常简单，核心原理是利用简单的布尔盲注构造一个 \(SELECT “\[RANDSTR\]” \[FROM\_DUMMY\_TABLE.get\(dbms\)\] \)=”\[RANDSTR1\]” 和 \(SELECT ‘\[RANDSTR\]’ \[FROM\_DUMMY\_TABLE.get\(dbms\)\] \)='\[RANDSTR1\]’ 这两个 Payload 的请求判断。其中

```text
FROM_DUMMY_TABLE = {
    DBMS.ORACLE: " FROM DUAL",
    DBMS.ACCESS: " FROM MSysAccessObjects",
    DBMS.FIREBIRD: " FROM RDB$DATABASE",
    DBMS.MAXDB: " FROM VERSIONS",
    DBMS.DB2: " FROM SYSIBM.SYSDUMMY1",
    DBMS.HSQLDB: " FROM INFORMATION_SCHEMA.SYSTEM_USERS",
    DBMS.INFORMIX: " FROM SYSMASTER:SYSDUAL"
}
```

例如，检查是否是 ORACLE 的时候，就会生成

```text
(SELECT 'abc' FROM DUAL)='abc' 
(SELECT 'abc' FROM DUAL)='abcd'
```

这样的两个 Payload，如果确实存在正负关系（具体内容参见后续章节的布尔盲注检测），则表明数据库就是 ORACLE。

当然数据库类型检测并不是必须的，因为 sqlmap 实际工作中，如果没有指定 DBMS 则会按照当前测试 Payload 的对应的数据库类型去设置。

实际上在各种 Payload 的执行过程中，会包含着一些数据库的推断信息\(&lt;details&gt;\)，如果 Payload 成功执行，这些信息可以被顺利推断则数据库类型就可以推断出来。

**测试数据模型与 Payload 介绍**

在实际的代码中，checkSqlInjection 是一个接近七百行的函数。当然其行为也并不是仅仅通过我们上面列出的步骤就可以完全概括的，其中涉及到了很多关于 Payload 定义中字段的操作。显然，直到现在我们都并不是特别了解一个 Payload 中存在着什么样的定义，当然也不会懂得这些操作对于这些字段到底有什么具体的意义。所以我们没有办法在不了解真正 Payload 的时候开始之后的步骤。

因此在本节中，我们会详细介绍关于具体测试 Payload 的数据模型，并且基于这些模型和源码分析 sqlmap 实际的行为，和 sql 注入原理的细节知识。 ·

**&lt;test&gt; 通用模型**

关于通用模型其实在 sqlmap 中有非常详细的说明，位置在 xml/payloads/boolean\_blind.xml中，我们把他们分隔开分别来讲解具体字段对应的代码的行为。

首先我们必须明白一个具体的 testcase 对应一个具体的 xml 元素是什么样子：

```text
<test>
    <title></title>
    <stype></stype>
    <level></level>
    <risk></risk>
    <clause></clause>
    <where></where>
    <vector></vector>
    <request>
        <payload></payload>
        <comment></comment>
        <char></char>
        <columns></columns>
    </request>
    <response>
        <comparison></comparison>
        <grep></grep>
        <time></time>
        <union></union>
    </response>
    <details>
        <dbms></dbms>
        <dbms_version></dbms_version>
        <os></os>
    </details>
</test>
```

关于上面的一个 &lt;test&gt; 标签内的元素都是实际上包含的不只是一个 Payload 还包含

```text
Sub-tag: <title>
    Title of the test. 测试的名称，这些名称就是我们实际在测试的时候输出的日志中的内容
```

[![img](https://p2.ssl.qhimg.com/t0153a7ac5d62b28ece.jpg)](https://p2.ssl.qhimg.com/t0153a7ac5d62b28ece.jpg)

上图表示一个 &lt;test&gt; 中的 title 会被输出作为调试信息。

除非必要的子标签，笔者将会直接把标注写在下面的代码块中，

```text
Sub-tag: <stype>
    SQL injection family type. 表示注入的类型。
​
    Valid values:
        1: Boolean-based blind SQL injection
        2: Error-based queries SQL injection
        3: Inline queries SQL injection
        4: Stacked queries SQL injection
        5: Time-based blind SQL injection
        6: UNION query SQL injection
​
Sub-tag: <level>
    From which level check for this test. 测试的级别
​
    Valid values:
        1: Always (<100 requests)
        2: Try a bit harder (100-200 requests)
        3: Good number of requests (200-500 requests)
        4: Extensive test (500-1000 requests)
        5: You have plenty of time (>1000 requests)
​
Sub-tag: <risk>
    Likelihood of a payload to damage the data integrity.这个选项表明对目标数据库的损坏程度，risk 最高三级，最高等级代表对数据库可能会有危险的•操作，比如修改一些数据，插入一些数据甚至删除一些数据。
​
    Valid values:
        1: Low risk
        2: Medium risk
        3: High risk
​
Sub-tag: <clause>
    In which clause the payload can work. 这个字段表明 <test> 对应的测试 Payload 适用于哪种类型的 SQL 语句。一般来说，很多语句并不一定非要特定 WHERE 位置的。
​
    NOTE: for instance, there are some payload that do not have to be
    tested as soon as it has been identified whether or not the
    injection is within a WHERE clause condition.
​
    Valid values:
        0: Always
        1: WHERE / HAVING
        2: GROUP BY
        3: ORDER BY
        4: LIMIT
        5: OFFSET
        6: TOP
        7: Table name
        8: Column name
        9: Pre-WHERE (non-query)
​
    A comma separated list of these values is also possible.
```

在上面几个子标签中，我们经常见的就是 level/risk 一般来说，默认的 sqlmap 配置跑不出漏洞的时候，我们通常会启动更高级别 \(level=5/risk=3\) 的配置项来启动更多的 payload。

接下来我们再分析下面的标签

```text
Sub-tag: <where>
    Where to add our '<prefix> <payload><comment> <suffix>' string.
​
    Valid values:
        1: Append the string to the parameter original value
        2: Replace the parameter original value with a negative random
            integer value and append our string
        3: Replace the parameter original value with our string
​
Sub-tag: <vector>
    The payload that will be used to exploit the injection point.
    这个标签只是大致说明 Payload 长什么样子，其实实际请求的 Payload 或者变形之前的 Payload 可能并不是这个 Payload，以 request 子标签中的 payload 为准。
​
Sub-tag: <request> 
    What to inject for this test.
    关于发起请求的设置与配置。在这些配置中，有一些是特有的，但是有一些是必须的，例如 payload 是肯定存在的，但是 comment 是不一定有的，char 和 columns 是只有 UNION 才存在
​
    Sub-tag: <payload>
        The payload to test for. 实际测试使用的 Payload
​
    Sub-tag: <comment>
        Comment to append to the payload, before the suffix.
​
    Sub-tag: <char> 只有 UNION 注入存在的字段
        Character to use to bruteforce number of columns in UNION
        query SQL injection tests.
​
    Sub-tag: <columns> 只有 UNION 注入存在的字段
        Range of columns to test for in UNION query SQL injection
        tests.
​
Sub-tag: <response>
    How to identify if the injected payload succeeded.
    由于 payload 的目的不一定是相同的，所以，实际上处理请求的方法也并不是相同的，具体的处理方法步骤，在我们后续的章节中有详细的分析。
​
    Sub-tag: <comparison> 
        针对布尔盲注的特有字段，表示对比和 request 中请求的结果。
        Perform a request with this string as the payload and compare
        the response with the <payload> response. Apply the comparison
        algorithm.
​
        NOTE: useful to test for boolean-based blind SQL injections.
​
    Sub-tag: <grep> 
        针对报错型注入的特有字段，使用正则表达式去匹配结果。
        Regular expression to grep for in the response body.
​
        NOTE: useful to test for error-based SQL injection.
​
    Sub-tag: <time>
        针对时间盲注
        Time in seconds to wait before the response is returned.
​
        NOTE: useful to test for time-based blind and stacked queries
        SQL injections.
​
    Sub-tag: <union>
        处理 UNION •注入的办法。
        Calls unionTest() function.
​
        NOTE: useful to test for UNION query (inband) SQL injection.
​
Sub-tag: <details>
    Which details can be infered if the payload succeed.
    如果 response 标签中的检测结果成功了，可以推断出什么结论？
​
    Sub-tags: <dbms>
        What is the database management system (e.g. MySQL).
​
    Sub-tags: <dbms_version>
        What is the database management system version (e.g. 5.0.51).
​
    Sub-tags: <os>
        What is the database management system underlying operating
        system.
```

在初步了解了基本的 Payload 测试数据模型之后，我们接下来进行详细的检测逻辑的细节分析，因为篇幅的原因，我们暂且只针对布尔盲注和时间盲注进行分析，

**真正的 Payload**

我们在前面的介绍中发现了几个疑似 Payload 的字段，但是遗憾的是，上面的每一个 Payload 都不是真正的 Payload。实际 sqlmap 在处理的过程中，只要是从 \*.xml 中加载的 Payload，都是需要经过一些随机化和预处理，这些预处理涉及到的概念如下：

1. Boundary：需要为原始 Payload 的前后添加“边界”。边界是一个神奇的东西，主要取决于当前“拼接”的 SQL 语句的上下文，常见上下文：注入位置是一个“整形”；注入位置需要单引号/双引号\(‘/”\)闭合边界；注入位置在一个括号语句中。
2. –tamper：Tamper 是 sqlmap 中最重要的概念之一，也是 Bypass 各种防火墙的有力的武器。在 sqlmap 中，Tamper 的处理位于我们上一篇文章中的 agent.queryPage\(\) 中，具体位于其对 Payload 的处理。
3. “Render”：当然这一个步骤在 sqlmap 中没有明显的概念进行对应，其主要是针对 Payload 中随机化的标签进行渲染和替换，例如：\[INFERENCE\] 这个标签通常被替换成一个等式，这个等式用于判断结果的正负`Positive/Negative` \[RANDSTR\] 会被替换成随机字符串 \[RANDNUM\] 与 \[RANDNUMn\] •会被替换成不同的数字 \[SLEEPTIME\] 在时间盲注中会被替换为 SLEEP 的时间

所以，实际上从 \*.xml 中加载出来的 Payload 需要经过上面的处理才能真的算是处理完成。这个 Payload 才会在 agent.queryPage 的日志中输出出来，也就是我们 sqlmap -v3 选项看到的最终 Payload。

在上面的介绍中，我们又提到了一个陌生的概念，Boundary，并且做了相对简单的介绍，具体的 Boundary，我们在 {sqlmap\_dir}/xml/boundaries.xml 中可以找到：

```text
<boundary>
    <level></level> 
    <clause></clause>
    <where></where>
    <ptype></ptype>
    <prefix></prefix>
    <suffix></suffix>
</boundary>
```

在具体的定义中，我们发现没见过的子标签如下：

```text
    Sub-tag: <ptype>
        What is the parameter value type. 参数•类型（参数边界上下文类型）
​
        Valid values:
            1: Unescaped numeric
            2: Single quoted string
            3: LIKE single quoted string
            4: Double quoted string
            5: LIKE double quoted string
​
    Sub-tag: <prefix>
        A string to prepend to the payload.
​
    Sub-tag: <suffix>
        A string to append to the payload.
```

其实到现在 sqlmap 中 Payload 的结构我们就非常清楚了

```text
<prefix> <payload><comment> <suffix>
```

其中 &lt;prefix&gt; &lt;suffix&gt; 来源于 boundaries.xml 中，而 &lt;payload&gt; &lt;comment&gt; 来源于本身 xml/payloads/\*.xml 中的 &lt;test&gt; 中。在本节中都有非常详细的描述了

**针对布尔盲注的检测**

在接下来的小节中，我们将会针对几种注入进行详细分析，我们的分析依据主要是 sqlmap 设定的 Payload 的数据模型和其本身的代码。本节先针对布尔盲注进行一些详细分析。

在分析之前，我们先看一个详细的 Payload:

```text
<test>
    <title>PostgreSQL OR boolean-based blind - WHERE or HAVING clause (CAST)</title>
    <stype>1</stype>
    <level>3</level>
    <risk>3</risk>
    <clause>1</clause>
    <where>2</where>
    <vector>OR (SELECT (CASE WHEN ([INFERENCE]) THEN NULL ELSE CAST('[RANDSTR]' AS NUMERIC) END)) IS NULL</vector>
    <request>
        <payload>OR (SELECT (CASE WHEN ([RANDNUM]=[RANDNUM]) THEN NULL ELSE CAST('[RANDSTR]' AS NUMERIC) END)) IS NULL</payload>
    </request>
    <response>
        <comparison>OR (SELECT (CASE WHEN ([RANDNUM]=[RANDNUM1]) THEN NULL ELSE CAST('[RANDSTR]' AS NUMERIC) END)) IS NULL</comparison>
    </response>
    <details>
        <dbms>PostgreSQL</dbms>
    </details>
</test>
```

根据上一节介绍的子标签的特性，我们可以大致观察这个 &lt;test&gt; 会至少发送两个 Payload：第一个为 request 标签中的 payload 第二个为 response 标签中的 comparison 中的 Payload。

当然我们很容易想到，针对布尔盲注的检测实际上只需要检测 request.payload 和 response.comparison 这两个请求，只要这两个请求页面不相同，就可以判定是存在问题的。可是事实真的如此吗？结果当然并没有这么简单。

我们首先定义 request.payload 中的的请求为正请求 Positive，对应 response.comparison中的请求为负请求 Negative，在 sqlmap 中原处理如下：

![img](https://p4.ssl.qhimg.com/dm/1024_528_/t0127e4740c970d7ae6.jpg)

在代码批注中我们进行详细的解释，为了让大家看得更清楚，我们把代码转变为流程图：

![img](https://p2.ssl.qhimg.com/t01ecdd2a80b47b6fcf.jpg)

其中最容易被遗忘的可能并不是正负请求的对比，而是正请求与模版页面的对比，负请求与错误请求的对比和错误请求与模版页面的对比，因为广泛存在一种情况是类似文件包含模式的情况，不同的合理输入的结果有大概率不相同，且每一次输入的结果如果报错都会跳转到某一个默认页面（存在默认参数），这种情况仅仅用正负请求来区分页面不同是完全不够用的，还需要各种情形与模版页面的比较来确定。

**针对 GREP 型（报错注入）**

针对报错注入其实非常好识别，在报错注入检测的过程中，我们会发现他的 response 子标签中，包含着是 grep 子标签：

```text
<test>
    <title>MySQL &gt;= 5.7.8 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (JSON_KEYS)</title>
    <stype>2</stype>
    <level>5</level>
    <risk>1</risk>
    <clause>1,2,3,9</clause>
    <where>1</where>
    <vector>AND JSON_KEYS((SELECT CONVERT((SELECT CONCAT('[DELIMITER_START]',([QUERY]),'[DELIMITER_STOP]')) USING utf8)))</vector>
    <request>
        <payload>AND JSON_KEYS((SELECT CONVERT((SELECT CONCAT('[DELIMITER_START]',(SELECT (ELT([RANDNUM]=[RANDNUM],1))),'[DELIMITER_STOP]')) USING utf8)))</payload>
    </request>
    <response>
        <grep>[DELIMITER_START](?P&lt;result&gt;.*?)[DELIMITER_STOP]</grep>
    </response>
    <details>
        <dbms>MySQL</dbms>
        <dbms_version>&gt;= 5.7.8</dbms_version>
    </details>
</test>
```

我们发现子标签 grep 中是正则表达式，可以直接从整个请求中通过 grep 中的正则提取出对应的内容，如果成功提取出了对应内容，则说明该参数可以进行注入。

在具体代码中，其实非常直观可以看到：

![img](https://p4.ssl.qhimg.com/dm/1024_408_/t014b674a169e0eca56.jpg)

再 sqlmap 的实现中其实并不是仅仅检查页面内容就足够的，除了页面内容之外，检查如下项：

1. HTTP 的错误页面
2. Headers 中的内容
3. 重定向信息

**针对 TIME 型（时间盲注，HeavilyQuery）**

当然时间盲注我们可以很容易猜到应该怎么处理：如果发出了请求导致延迟了 X 秒，并且响应延迟的时间是我们预期的时间，那么就可以判定这个参数是一个时间注入点。

但是仅仅是这样就可以了嘛？当然我们需要了解的是 sqlmap 如何设置这个 X 作为时间点（请看下面这个函数，位于 agent.queryPage 中）：

![img](https://p5.ssl.qhimg.com/dm/1024_483_/t01885c1b538a24c402.jpg)

我们发现，它里面有一个数学概念：[标准差](http://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/%E6%A8%99%E6%BA%96%E5%B7%AE)

> 简单来说，标准差是一组数值自平均值分散开来的程度的一种测量观念。一个较大的标准差，代表大部分的数值和其平均值之间差异较大；一个较小的标准差，代表这些数值较接近平均值。例如，两组数的集合{0, 5, 9, 14}和{5, 6, 8, 9}其平均值都是7，但第二个集合具有较小的标准差。述“相差k个标准差”，即在 X̄ ± kS 的样本（Sample）范围内考量。标准差可以当作不确定性的一种测量。例如在物理科学中，做重复性测量时，测量数值集合的标准差代表这些测量的精确度。当要决定测量值是否符合预测值，测量值的标准差占有决定性重要角色：如果测量平均值与预测值相差太远（同时与标准差数值做比较），则认为测量值与预测值互相矛盾。这很容易理解，因为如果测量值都落在一定数值范围之外，可以合理推论预测值是否正确。

根据注释和批注中的解释，我们发现我们需要设定一个最小 SLEEPTIME 应该至少大于 样本内平均响应时间 + 7 \* 样本标准差，这样就可以保证过滤掉 99.99% 的无延迟请求。

当然除了这一点，我们还发现

```text
delta = threadData.lastQueryDuration - conf.timeSec
if Backend.getIdentifiedDbms() in (DBMS.MYSQL,):  # MySQL's SLEEP(X) lasts 0.05 seconds shorter on average
    delta += 0.05
return delta >= 0
```

这一段代码作为 mysql 的 Patch 存在 \# MySQL’s SLEEP\(X\) lasts 0.05 seconds shorter on average。

如果我们要自己实现时间盲注的检测的话，这一点也是必须注意和实现的。

**针对 UNION 型（UNION Query）**

UNION 注入可以说是 sqlmap 中最复杂的了，同时也是最经典的注入情形。

其实关于 UNION 注入的检测，和我们一开始学习 SQL 注入的方法是一样的，猜解列数，猜解输出点在列中位置。实际在 sqlmap 中也是按照这个来进行漏洞检测的，具体的测试方法位于：

![img](https://p0.ssl.qhimg.com/t0155eeae376aca7796.jpg)

跟入 unionTest\(\) 中我们发现如下操作

```text
def unionTest(comment, place, parameter, value, prefix, suffix):
    """
    This method tests if the target URL is affected by an union
    SQL injection vulnerability. The test is done up to 3*50 times
    """
​
    if conf.direct:
        return
​
    kb.technique = PAYLOAD.TECHNIQUE.UNION
    validPayload, vector = _unionTestByCharBruteforce(comment, place, parameter, value, prefix, suffix)
​
    if validPayload:
        validPayload = agent.removePayloadDelimiters(validPayload)
​
    return validPayload, vector
```

最核心的逻辑位于 \_unionTestByCharBruteforce 中，继续跟入，我们发现其检测的大致逻辑如下：

![img](https://p1.ssl.qhimg.com/dm/1024_460_/t019b7506447831c5b1.jpg)

别急，我们一步一步来分析！

**猜列数**

我相信做过渗透测试的读者基本对这个词都非常非常熟悉，如果有疑问或者不清楚的请自行百度，笔者再次不再赘述关于 SQL 注入基本流程的部分。

为什么要把一件这么简单的事情单独拿出来说呢？当然这预示着 sqlmap 并不是非常简单的在处理这一件事情，因为作为一个渗透测试人员，当然可以很容易靠肉眼分辨出很多事情，但是这些事情在计算机看来却并不是那么容易可以判断的：

1. 使用 ORDER BY 查询，直接通过与模版页面的比较来获取列数。
2. 当 ORDER BY 失效的时候，使用多次 UNION SELECT 不同列数，获取多个 Ratio，通过区分 Ratio 来区分哪一个是正确的列数。

实际在使用的过程中，ORDER BY 的核心逻辑如下，关于其中页面比较技术我们就不赘述了，不过值得一提的是 sqlmap 在猜列数的时候，使用的是二分法（笔者看了一下，二分法这部分这似乎是七年前的代码）。

![img](https://p1.ssl.qhimg.com/dm/1024_479_/t010c103329c006f4ed.jpg)

除此之外呢，如果 ORDER BY 失效，将会计算至少五个（从 lowerCount 到 upperCount）Payload 为 UNION SELECT \(NULL,\) \* \[COUNT\]，的请求，这些请求的对应 RATIO（与模版页面相似度）会汇总存储在 ratios 中，同时 items 中存储 列数 和 ratio 形成的 tuple，经过一系列的算法，尽可能寻找出“与众不同（正确猜到列数）”的页面。具体的算法与批注如下：

![img](https://p2.ssl.qhimg.com/dm/1024_504_/t01fc628d1e193a4af6.jpg)

我们发现，上面代码表达的核心思想就是 利用与模版页面比较的内容相似度寻找最最不同的那一个请求。

**定位输出点**

假如一切顺利，我们通过上面的步骤成功找到了列数，接下来就应该寻找输出点，当然输出点的寻找也是需要额外讨论的。其实基本逻辑很容易对不对？我们只需要将 UNION SELECT NULL, NULL, NULL, NULL, … 中的各种 NULL 依次替换，然后在结果中寻找被我们插入的随机的字符串，就可以很容易定位到输入出点的位置。实际上这一部分的确认逻辑是位于下图中的函数的 \_unionConfirm

![img](https://p5.ssl.qhimg.com/t010dd1c781b3fcac3b.jpg)

其中主要的逻辑是一个叫 \_unionPosition 的函数，在这个函数中，负责定位输出点的位置，使用的基本方法就是我们在开头提到方法，受限于篇幅，我们就不再展开叙述了。

#### 0x03 结束语

其实按笔者原计划，本系列文章并没有结束，因为还有关于 sqlmap 中其他技术没有介绍：“数据持久化”，“action\(\) – Exploit 技术”，“常见漏洞利用分析（udf，反弹 shell 等）”。但是由于内容是在太过庞杂，笔者计划暂且搁置一下，实际上现有的文章已经足够把 sqlmap 的 SQL 注入检测最核心的也是最有意义的自动化逻辑说清楚了，我想读读者读完之后肯定会有自己的收获。  


