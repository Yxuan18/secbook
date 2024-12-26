---
description: 使用burp插件captcha-killer识别图片验证码
---

# captcha-killer

### 0x01 开发背景 <a href="#id-0x01-kai-fa-bei-jing" id="id-0x01-kai-fa-bei-jing"></a>

说起对存在验证码的登录表单进行爆破，大部分人都会想到`PKav HTTP Fuzzer`，这款工具在前些年确实给我们带来了不少便利。反观burp一直没有一个高度自定义通杀大部分图片验证码的识别方案，于是抽了点闲暇的时间开发了[captcha-kille](https://github.com/c0ny1/captcha-killer)，希望burp也能用上各种好用的识别码技术。其设计理念是`只专注做好对各种验证码识别技术接口的调用！`说具体点就是burp通过同一个插件，就可以适配各种验证码识别接口，无需重复编写调用代码。今天不谈编码层面如何设计，感兴趣的可以去github看源码。此处只通过使用步骤来说明设计的细节。

### 0x02 Step1:将获取验证码的数据包发送到插件 <a href="#id-0x02step1-jiang-huo-qu-yan-zheng-ma-de-shu-ju-bao-fa-song-dao-cha-jian" id="id-0x02step1-jiang-huo-qu-yan-zheng-ma-de-shu-ju-bao-fa-song-dao-cha-jian"></a>

使用burp抓取获取验证码数据包，然后右键`captcha-killer` -> `send to captcha panel`发送数据包到插件的验证码请求面板。

将请求验证码数据包发送到插件

![将请求验证码数据包发送到插件](https://gv7.me/articles/2019/burp-captcha-killer-usage/step1-1.png)

然后到切换到插件面板，点击获取即可拿到要识别的验证码图片内容。

请求获取验证码

![请求获取验证码](https://gv7.me/articles/2019/burp-captcha-killer-usage/step1-2.png)

**注意：获取验证码的cookie一定要和intruder发送的cookie相同！**

### 0x03 Step2:配置识别接口的地址和请求包 <a href="#id-0x03step2-pei-zhi-shi-bie-jie-kou-de-di-zhi-he-qing-qiu-bao" id="id-0x03step2-pei-zhi-shi-bie-jie-kou-de-di-zhi-he-qing-qiu-bao"></a>

拿到验证码之后，就要设置接口来进行识别了。我们可以使用网上寻找免费的接口，用burp抓包，然后右键发送到插件的接口请求面板。

将接口调用请求发送到插件

![将接口调用请求发送到插件](https://gv7.me/articles/2019/burp-captcha-killer-usage/step2-1.png)

然后我们把图片内容的位置用标签来代替。比如该例子使用的接口是post提交image参数，参数的值为图片二进制数据的base64编码后的url编码。那么`Request template`(请求模版)面板应该填写如下：

接口请求模版设置

![接口请求模版设置](https://gv7.me/articles/2019/burp-captcha-killer-usage/step2-2.png)

| ID | 标签                          | 描述                |
| -- | --------------------------- | ----------------- |
| 1  | `<@IMG_RAW></@IMG_RAW>`     | 代表验证码图片原二进制内容     |
| 2  | `<@URLENCODE></@URLENCODE>` | 对标签内的内容进行url编码    |
| 3  | `<@BASE64></@BASE64>`       | 对标签内的内容进行base64编码 |

最后点击“识别”即可获取到接口返回的数据包，同时在`request raw`可以看到调用接口最终发送的请求包。

模版被渲染为最终的请求

![模版被渲染为最终的请求](https://gv7.me/articles/2019/burp-captcha-killer-usage/step2-3.png)

### 0x03 Step3:设置用于匹配识别结果的规则 <a href="#id-0x03step3-she-zhi-yong-yu-pi-pei-shi-bie-jie-guo-de-gui-ze" id="id-0x03step3-she-zhi-yong-yu-pi-pei-shi-bie-jie-guo-de-gui-ze"></a>

通过上一步我们获取到了识别接口的返回结果，但是插件并不知道返回结果中，哪里是真正的识别结果。插件提供了4中方式进行匹配，可以根据具体情况选择合适的。

| ID | 规则类型                               | 描述                                                                                           |
| -- | ---------------------------------- | -------------------------------------------------------------------------------------------- |
| 1  | Repose data                        | 这种规则用于匹配接口返回包内容直接是识别结果                                                                       |
| 2  | Regular expression                 | 正则表达式,适合比较复杂的匹配。比如接口返回包`{"coede":1,"result":"abcd"}`说明abcd是识别结果，我们可以编写规则为`result":"(.*?)"\}` |
| 3  | Define the start and end positions | 定义开始和结束位置,使用上面的例子，可以编写规则`{"start":21,"end":25}`                                              |
| 4  | Defines the start and end strings  | 定义开始和结束字符，使用上面的例子，可以编写规则为`{"start":"result\":\","end":"\"\}"}`                               |

通过分析我们知道，接口返回的json数据中，字段`words`的值为识别结果。我们这里使用`Regular expression`(正则表达式)来匹配，然后选择`yzep`右键`标记为识别结果`，系统会自动生成正则表达式规则`" (.*?)"\}\]`。

设置匹配方式和自动生成规则

![设置匹配方式和自动生成规则](https://gv7.me/articles/2019/burp-captcha-killer-usage/step3-1.png)

注意：若右键标记自动生成的规则匹配不精确，可以人工进行微调。比如该例子中可以微调规则为`"words"\: "(.*?)"\}`将更加准确！

到达这步建议将配置好常用接口的url，数据包已经匹配规则保存为模版，方便下次直接通过右键`模板库`中快速设置。同时插件也有默认的模版供大家使用与修改。

保存设置好的配置，方便下次快速配置

![保存设置好的配置，方便下次快速配置](https://gv7.me/articles/2019/burp-captcha-killer-usage/step3-2.png)

### 0x04 Step4:在Intruder模块调用 <a href="#id-0x04step4-zai-intruder-mo-kuai-tiao-yong" id="id-0x04step4-zai-intruder-mo-kuai-tiao-yong"></a>

配置好各项后，可以点击`锁定`对当前配置进行锁定，防止被修改导致爆破失败！接着安装以下步骤进行配置

设置Intruder的爆破模式和payload位置

![设置Intruder的爆破模式和payload位置](https://gv7.me/articles/2019/burp-captcha-killer-usage/step4-1.png)

验证码payload选择有插件来生成

![验证码payload选择有插件来生成](https://gv7.me/articles/2019/burp-captcha-killer-usage/step4-2.png)

进行爆破，可以通过对比识别结果看出识别率

![进行爆破，可以通过对比识别结果看出识别率](https://gv7.me/articles/2019/burp-captcha-killer-usage/step4-3.png)
