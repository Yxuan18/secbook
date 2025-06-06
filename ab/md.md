# markdown

## 一、why we need it

### 1、起源

Markdown是由约翰·格鲁伯（JohnGruber）在亚伦·斯沃茨（Aaron Swartz）的帮助下开发，并在2004年发布的标记语言。

其设计灵感主要来源于纯文本电子邮件的格式，目标是让人们能够使用易读、易写的纯文本格式编写文档，而且这些文档可以转换为HTML（Hyper Text Markup Language，超文本标记语言）文档。

简单点说，Markdown就是由一些简单的符号（如\*/-> \[] （）#）组成的用于排版的标记语言，其最重要的特点就是可读性强。

### 2、基本语法

![](<../.gitbook/assets/image (243).png>)

### 3、场景

适用：\
当你对文章的排版没什么特殊需求，且不想花太多时间在排版上时，就可以使用Markdown。因为编辑器或平台会通过Markdown标记对文章进行渲染，最终的排版效果会非常简洁、漂亮。

| 场景      | 工具平台                                                |
| ------- | --------------------------------------------------- |
| 笔记      | 印象笔记，有道笔记                                           |
| 多人协作文档  | 腾讯文档，石墨文档                                           |
| 博客      | 知乎，简书，CSDN，WordPress，HEXO                           |
| 微信公众号文章 | online-markdown，Md2all                              |
| 邮件      | markdown here                                       |
| 便签      | 锤子便签                                                |
| 日记      | DayOne                                              |
| 交互式文档   | Jupyter Notebook，R Markdown                         |
| 网页      | md-page                                             |
| 项目文档    | MkDocs，VuePress                                     |
| 幻灯片     | nodeppt, shower,  remark ,  impress.js ,  reveal.js |
| 书       | Gitbook                                             |

不适用：\
如果你对字号、段落、图片、表格等方面的排版要求较高，还是需要使用Word这类专业的编辑软件的。

Markdown文件可以很方便地转换为Word文件，如果有一些需要特殊处理的格式，可以两者结合使用。

Markdown与Word对比：

|      | Markdown                                                                                                                                                                                                  | Word                                                                                                      |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| 学习时间 | Markdown语法简单，只需使用一些常用的标记符号就能编写Markdown文档，任何人都可以在短时间内学会                                                                                                                                                    | 想要精通Word需要花费很多时间，如果不是专业编辑或相关从业人员，是没有必要学到精通的地步的。对于普通人来说，如果只想应付日常工作，学会Word不到20%的功能也许就足够了，但这往往也需要花费较长的时间     |
| 兼容性  | Markdown兼容性非常好，如果使用的是相同的语法，几乎可以做到一处编写，随处使用；借助一些工具，Markdown文档可以很方便地转换为各种类型的文件，如 PDF、Word、HTML、ePub、LATEX 等；Markdown 几乎可以应用于任何写作场景，很多专业的软件都集成了Markdown，如 Visual Studio Code （简称VS Code）、有道笔记、印象笔记、R Studio等 | Word文档可以转换为其他文件格式，但其支持的类型屈指可数，而且不能保证较好的兼容性。相信你一定遇到过使用不同版本的Word互传资料后格式乱掉的问题，也一定遇到过从网上复制一段文字到Word文档中格式乱掉的问题 |
| 打开速度 | Markdown 是“轻量级的标记语言”，可以使用任何编辑器打开，如果不需要渲染，几乎是“秒开”                                                                                                                                                          | Word是重量级软件，当打开Word文档时，“速度会比较慢”，有时候还会出现意想不到的情况                                                             |
| 功能   | 专注写作：罗振宇在 2016 年“时间的朋友”跨年演讲中提到过一个观点，他说：“当我需要一个服务时，不要给我太多选择，请直接告诉我什么是最好的，我要你的最佳方案。”Markdown就是写作文档的最佳方案，如果对排版没什么特殊要求，那就交给Markdown处理吧，你专注写作内容就行了                                                             | 由于Wo r d功能较多，人们在写作时总会忍不住去尝试各种排版效果（因为不知道哪一种更好），例如换一种字体、换一个颜色、调一下行高、调一下行距。这样既“费时又费力”，时间也逐渐在指缝中溜走            |

### 4、工作流程

![](<../.gitbook/assets/image (240).png>)

较流行编辑器列表：

![](<../.gitbook/assets/image (256).png>)

跨平台：指其提供了支持Windows、macOS、Linux等操作系统的版本。\
移动端：指其提供了移动（iOS/Android）App，且App支持Markdown写作。\
免 费：指其能够免费下载和使用，内购是指某些高级功能需要购买后才能使用。

## 二、how to use

### 1、字体

```
## 底线语法

一级标题
==

二级标题
--
```

![](<../.gitbook/assets/image (241).png>)

```
## '#'语法

# 一级标题
## 二级标题
```

![](<../.gitbook/assets/image (241).png>)

![](<../.gitbook/assets/image (248).png>)

两种语法说明：

| 底线                                                                      | 井号                                                                                |
| ----------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| <p>底线是=表示一级标题</p><p>底线是-表示二级标题</p><p>底线符号的数量至少2个</p><p>这种语法只支持这两级标题</p> | <p>在行首插入#可标记出标题</p><p>#的个数表示了标题的等级</p><p>建议在#后加一个空格</p><p>Markdown中最多只支持前六级标题</p> |

在使用过程中，建议使用#标记标题，而不是===或-，因为后者会难以阅读和维护

粗体与斜体：

![](<../.gitbook/assets/image (235).png>)

在使用两种字体时，建议粗体使用2个'\*'包&#x88F9;_，_&#x659C;体使用1个'\*'包裹，因为'\*'比较常见，而且比'\_'可读性更强

### 2、段落与换行

语法说明：

如果行与行之间没有空行，则会被视为同一段落\
如果行与行之间有空行，则会被视为不同的段落\
空行是指行内什么都没有，或者只有空格和制表符\
如果想在段内换行，则需要在上一行的结尾插入两个以上的空格然后按回车键。

![](<../.gitbook/assets/image (245).png>)

关于换行的建议：

当超过80个字符后进行换行\
在一句话结束（。或!或?）之后换行=\
当URL较长时换行

在列表中，分为有序列表与无序列表：

有序列表语法：

```
1. 有序列表
2. 有序列表
3. 有序列表
```

![](<../.gitbook/assets/image (237).png>)

无序列表语法：

使用\*/+/-来标记无序列表的效果是相同的

```
* 无序列表
+ 无序列表
- 无序列表
```

![](<../.gitbook/assets/image (255).png>)

嵌套列表语法：

```
第一层列表
    第二层列表
TAB+第二层列表
        第三层列表
TAB+TAB+第三层列表
```

语法说明：

中可以嵌套列表\
有序列表和无序列表也可以互相嵌套

示例如下：

![](<../.gitbook/assets/image (234).png>)

分隔线

在Markdown中，分隔线由3个以上的\*/-/\_来标记

语法如下：

```
***
分割线
---
or
===
```

说明：

分隔线须使用至少3个以上的\*/-/\_来标记\
行内不能有其他的字符\
可以在标记符中间加上空格

![](<../.gitbook/assets/image (249).png>)

### 3、图片

语法如下：

```
![图片代替文字](图片地址)

## 示例
![jpg](images/001.jpg)    #相对地址
![jpg](C:/windows/system32/images/001.jpg)    #绝对地址
![jpg](http://192.168.100.63/upload/images/001.jpg)    #网络地址
```

语法说明：

图片替代文字在图片无法正常显示时会比较有用，正常情况下可以为空\
图片地址可以是本地图片的路径也可以是网络图片的地址\
本地图片支持相对路径和绝对路径两种方式

### 4、链接

```
## 文字链接

[google](https://www.google.com)

## 引用链接

[google]

[google]:https://www.google.com

## 网址链接
<http://IP:PORT/>    #自动转换为超链接
```

![](<../.gitbook/assets/image (238).png>)

引用链接语法说明：

链接标记可以有字母、数字、空格和标点符号\
链接标记不区分大小写\
定义的链接内容可以放在当前文件的任意位置，建议放在页尾\
当链接地址为网络地址时要以 http/https开头，否则会被识别为本地地址

### 5、行内代码与代码块

```
## 行内代码
`行内代码`

## 代码块
在Markdown中，代码块以Tab键或4个空格开头
    and id;

或是四个空格开头：
    and id;
```

![](<../.gitbook/assets/image (252).png>)

如果代码超过1行，请使用围栏代码块（扩展语法），并显式地声明语言，这样做便于阅读，并且可以显示语法高亮

语法如下：

````
```python
import os

a = 1

...
```
````

小技巧：

可在shell命令末尾添加 \`\\\`符号，避免了命令过长的换行，同时也增加了源码的可读性

![](<../.gitbook/assets/image (253).png>)

### 6、引用

语法：

```
> 引用内容
```

语法说明：

多行引用也可以在每一行的开头都插入>。\
在引用中可以嵌套引用。\
在引用中可以使用其他的Markdown语法。\
段落与换行的格式在引用中也是适用的

![](<../.gitbook/assets/image (254).png>)

使用规范：

建议在引用的标记符号＞之后添加一个空格\
建议每一行引用都使用符号＞\
不要在引用中添加空行

### 7、转义

使用范围：当我们想在Markdown文件中插入一些标记符号，但又不想让这些符号被渲染时

语法：

```
\+转义字符

例：
\*
```

![](<../.gitbook/assets/image (242).png>)

### 8、GFM语法

1、删除线

语法：

```
~~被删除的文字~~
```

![](<../.gitbook/assets/image (251).png>)

2、表情符号

语法：

```
:表情代码:
```

![更多的表情符号请参考http://www.webpagefx.com/tools/emoji-cheat-sheet/](<../.gitbook/assets/image (258).png>)

3、自动链接

在标准语法中，由<>包裹的URL地址被自动识别并解析为超链接，使用GFM扩展语法则可以不使用<>包裹

![](<../.gitbook/assets/image (247).png>)

4、表格

语法：

```
|表头1|表头2|表头3|
|--|--|--|
|内容1|内容2|内容3|
|内容1|内容2|内容3|
```

语法说明：

单元格使用|来分隔，为了阅读更清晰，建议最前和最后都使用|。\
单元格和|之间的空格会被移除。\
表头与其他行使用-来分隔。\
表格对齐格式如下。\
&#x20;     左对齐（默认）：:\
&#x20;     右对齐：-:\
&#x20;     居中对齐：:-:\
块级元素（代码区块、引用区块）不能插入表格中

![](<../.gitbook/assets/image (257).png>)

关于创建表格的建议如下：

1. 在表格的前、后各空1行
2. 在每一行最前和最后都使用|，每一行中的|要尽量都对齐。
3. 不要使用庞大复杂的表格，那样会难以维护和阅读

5、任务列表

语法：

```
- [ ] 未勾选
- [x] 已勾选
```

语法说明如下：

1. 任务列表以-+空格开头，由 \[+空格/x+] 组成。
2. x可以小写，也可以大写，有些编辑器可能不支持大写，所以为避免解析错误，推荐使用小写的x。
3. 当方括号中的字符为空格时，复选框是未选中状态，为x时是选中状态

![](<../.gitbook/assets/image (239).png>)

6、围栏代码块

在基础语法中，代码块使用Tab键或4个空格开头；在扩展语法中，围栏代码块使用连续3个\`或3个\~包裹，还支持语法高亮，可读性和可维护性更强一些

语法：

````
```
eval($_POST['a']);
```

~~~
eval($_POST['a']);
~~~

```php
eval($_POST['a']);
```

~~~php
eval($_POST['a']);
~~~
````

![](<../.gitbook/assets/image (244).png>)

7、锚点

锚点，也称为书签，用来标记文档的特定位置，使用锚点可以跳转到当前文档或其他文档中指定的标记位置。

Markdown会被渲染成HTML页面，在HTML页面中可以通过锚点实现跳转；GitHub、GitBook项目文档中的目录也是通过锚点实现跳转的

语法：

```
[锚点描述](#锚点名)
```

语法说明：

1. 锚点名建议使用字母和数字，当然中文也是被支持的，但不排除有些网站支持得不够好。
2. 锚点名是区分英文大小写的。
3. 在锚点名中不能含有空格，也不能含有特殊字符。

![](<../.gitbook/assets/image (250).png>)

### 9、排版技巧

1、关于空格

1. 英文标点符号（如，.；：？）与后面的字符之间需要加空格，与前面的字符之间不需要加空格
2. 中文标点符号和数字、中文、英文之间不需要添加空格
3. 数字和百分号之间不需要添加空格
4. 数字和单位符号之间不需要添加空格
5. 英文和数字组合成的名字之间不需要添加空格
6. 当/（半角）表示“或”、“路径”时，与前后的字符之间均不加空格
7. 货币符号后不加空格
8. 负号后不加空格

2、全角与半角

全角：中文标点符号是全角，占两个字节。\
半角：英文标点符号和数字是半角，占1个字节。\
全角：，。；：!#\
半角：,.;:!#

1. 在中文排版中，要使用全角标点符号
2. 在英文排版中，要使用半角标点符号

5、正确英文大小写

很多人在文章、邮件甚至简历中，会把专有名词写错，虽然这并不会影响人们对内容的理解，但有时的确会让人觉得你不太“专业”。

例如：

错误的写法：IPhone7、MacOS\
正确的写法：iPhone 7、macOS

专有名词要使用正确的大小写，请参考它们的官方文档

## 三、相关

公众号排版可用：[md.aclickall.com/](http://md.aclickall.com/)
