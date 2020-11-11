# JSFind

随着SPA网页的兴起，js文件中会包含大量前端路由和后端api的链接信息，获取到这些链接能提供更多的有效信息。

burp自带的JS Link Finder效果很不错，不过找到的链接不能直接复制使用，需要再做处理，使用起来很不方便，根据[js-link-finder](https://github.com/portswigger/js-link-finder)使用burp-clj脚本重新实现，并根据[LinkFinder](https://github.com/GerbenJavado/LinkFinder)提供的正则表达式，新增了REST API的支持，现在可以直接通过[jslink](https://github.com/ntestoc3/burp-scripts/blob/master/jslink.clj)对js中找到的链接进行处理，然后作为burp Intruder的payload使用。

## 2 安装方法 <a id="h2-2"></a>

![burp-clj-install.gif](https://image.3001.net/images/20201014/1602682930_5f870032295fe2bab4f0d.gif)

图1  插件安装流程

### 2.1 安装burp-clj插件 <a id="h3-1"></a>

下载[burp-clj](https://github.com/ntestoc3/burp-clj/releases)最新版本的burp-clj.jar,然后在burp Extender中加载jar插件。

![](https://image.3001.net/images/20201014/1602682889_5f870009c345a1dbce472.png!small)

图2  burp Extender加载burp-clj.jar插件

### 2.2 添加脚本源 <a id="h3-2"></a>

启用burp-clj插件后，切换Clojure Plugin选项卡,点Add按钮添加脚本源：[https://github.com/ntestoc3/burp-scripts](https://github.com/ntestoc3/burp-scripts)，这里使用github地址，也可以git clone下来，使用本地目录。

![burp-clj-2.png](https://image.3001.net/images/20201014/1602682966_5f870056a41d536b83b2e.png!small)

图3  Clojure Plugin添加脚本源

添加脚本源之后，点Reload Scripts!加载脚本。

### 2.3 启用jslink脚本 <a id="h3-3"></a>

在Clojure Plugin选项卡的Scripts List中勾选js link parse的复选框，启用jslink脚本。

![burp-clj-4.png](https://image.3001.net/images/20201014/1602682980_5f870064a66f8d832790b.png!small)

图4  启用jslink脚本

## 3 使用方法 <a id="h2-3"></a>

![burp-clj-jslink.gif](https://image.3001.net//images//20201014//1602685852_5f870b9ccfe1ba2d8b63f.gif)

图5  jslink使用方法

浏览器通过burp代理访问网站，jsfind会被动扫描js文件，扫描到含有链接的js文件会生成issue。

JS Links选项卡可以看到所有扫描到的包含链接的js文件，可以同时在左侧列表选择多个js文件\(按ctrl或shift多选\)，右边列表会显示所有选中js文件中的链接。

![burp-clj-5.png](https://image.3001.net/images/20201014/1602683107_5f8700e35dd5b4c43fb49.png!small)

图6  JS Links查看多个js文件中的链接

针对不需要的链接可以选中后删除，然后通过 **删除链接前面的./字符**按钮统一链接的格式，方便intruder使用。

注意链接列表中的删除并不会保存，再次切换到对应的js文件，原先的链接还会存在。

## 4 注意事项 <a id="h2-4"></a>

如果浏览器访问过目标网站，则js文件会缓存，再次访问burp里面只能看到304,不会分析文件内容，需要清空缓存并硬性重新加载，用于在burp中获取js内容。

burp项目中的issue信息会保存，但jslink找到的链接不会保存，jslink脚本重新加载的话，已经分析的链接信息会丢失。

