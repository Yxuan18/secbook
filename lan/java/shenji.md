# 代码审计

### 一、基础

代码审计工作流程\
配置分析环境→熟悉业务流程→分析程序架构→工具自动化分析→人工审计结果→报告

JSP生命周期 \
客户端请求→web服务器→JSP容器→JSP网页→JSP网页实现类（.java）→JSP网页实现类（.class）→HTML→客户端请求

![war包结构](<../../.gitbook/assets/image (365).png>)

JAVA内置对象 \
无需声明直接使用： \
request，response，pageContent，session，application，out，config，page，exception

JAVA中危险函数： \
常用的高危函数： \
getParameter()，getcookies()，getQueryString()，getheaders()，Runtime.exec()，logger.info \
可能存在危险的方法：\
password，upload，download

### 二、工具——Fortify&#x20;

简介：&#x20;

1. Fortify Manager 软件安全信息综合管理平台
2. 集中化管理
3. 有洞察力的报告&#x20;

支持语言： \
ABAP，ASP.NET，C，C++，C#，Classic ASP，COBOL，ColdFusion，Flex/ActionScript，Java，JavaScript/AJAX，JSP，Objective C，PL/SQL，PHP，Python，T-SQL，VB.NET，VBScript，VB6，XML/HTML

