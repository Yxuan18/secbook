# 信息泄露漏洞\_java

##  一、资料收集

###  [JSP注释与HTML注释](https://www.cnblogs.com/jayus/p/11395071.html) ： 

系统中存在大量JSP页面当中注释采用HTML注释，而不是JSP注释的情况。 此情况下，攻击者在浏览器浏览页面的时候，右键查看页面源代码，可以看到HTML注释内的信息，很多时候甚至是代码块，导致系统信息泄露。 

Jsp html 注释区别：  
\(1\)HTML页面注释-----这里面的注释会被编译（加载页面时，会进行语法判断，取需要的变量默认值  
\(2\)jsp页面注释------这里之后的注释都不会被编译 &lt;%--JSP中的注释，这里面的内容在查看页面源代码时，看不到这里面注释书写的内容 --%&gt; 所以涉及业务的建议使用&lt;%-- --%&gt;注释，文字描述性的使用注释。

### [J2EE相关的信息泄露](https://blog.csdn.net/weixin_34430692/article/details/114826046) ：

Java异常处理机制\(Exception\)简要说明：Java中它是由Trowable类的两个子类的两大部分组成，Error类和Exception类。Error是不推荐捕获的\(请查看Java异常处理机制中Error与Exception的区别\)，而Exception类除了子类RuntimeException是不能被捕获，其他子类的异常必须捕获，简单来讲，就产生异常信息了。

但Exception产生异常信息的过程有个特点，当发生异常时，异常抛给调用该函数的上一级函数，直到出现包含异常处理\(catch\)的层为止，这个给开发者在程序调试中带来很大的方便，能够快速定位问题所在等，看这段异常信息：

```java
org.springframework.dao.DataIntegrityViolationException: could not execute query; SQL [
select AdContentId,ContentDesc,ContentType,ContentSize,ContentUrl
from AAS_BIZ_AdContent
where 1=1
and AdInfoId = ?
and contentType = ?
order by AdInfoId ,ContentSize
]; nested exception is org.hibernate.exception.DataException: could not execute query
at org.springframework.orm.hibernate3.SessionFactoryUtils.convertHibernateAccessException(SessionFactoryUtils.java:642)
at org.springframework.orm.hibernate3.HibernateAccessor.convertHibernateAccessException(HibernateAccessor.java:412)
at org.springframework.orm.hibernate3.HibernateTemplate.doExecute(HibernateTemplate.java:411)
at org.springframework.orm.hibernate3.HibernateTemplate.executeFind(HibernateTemplate.java:343)
at com.suning.framework.dao.UniversalDaoHibernate.queryListBySql(UniversalDaoHibernate.java:567)
at com.suning.framework.dao.UniversalDaoHibernate.queryListBySql(UniversalDaoHibernate.java:554)
at com.suning.aas.ad.dao.hibernate.AdContentDaoHibernate.searchContent(AdContentDaoHibernate.java:40)
at com.suning.aas.ad.logic.impl.AdInfoBizImpl.searchContent(AdInfoBizImpl.java:100)
at sun.reflect.GeneratedMethodAccessor267.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
at java.lang.reflect.Method.invoke(Method.java:600)
at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:309)
at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:183)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:149)
at com.suning.framework.template.ServiceInterceptor.invoke(ServiceInterceptor.java:86)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:172)
at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:202)
at $Proxy53.searchContent(Unknown Source)
at com.suning.aas.portal.adsearch.action.ChannelAdAction.orderPage(ChannelAdAction.java:152)
at sun.reflect.GeneratedMethodAccessor358.invoke(Unknown Source)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
at java.lang.reflect.Method.invoke(Method.java:600)
at com.opensymphony.xwork2.DefaultActionInvocation.invokeAction(DefaultActionInvocation.java:441)
at com.opensymphony.xwork2.DefaultActionInvocation.invokeActionOnly(DefaultActionInvocation.java:280)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:243)
at com.opensymphony.xwork2.validator.ValidationInterceptor.doIntercept(ValidationInterceptor.java:252)
at org.apache.struts2.interceptor.validation.AnnotationValidationInterceptor.doIntercept(AnnotationValidationInterceptor.java:68)
at com.opensymphony.xwork2.interceptor.MethodFilterInterceptor.intercept(MethodFilterInterceptor.java:87)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.ConversionErrorInterceptor.intercept(ConversionErrorInterceptor.java:122)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.ParametersInterceptor.doIntercept(ParametersInterceptor.java:195)
at com.opensymphony.xwork2.interceptor.MethodFilterInterceptor.intercept(MethodFilterInterceptor.java:87)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.ParametersInterceptor.doIntercept(ParametersInterceptor.java:195)
at com.opensymphony.xwork2.interceptor.MethodFilterInterceptor.intercept(MethodFilterInterceptor.java:87)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.StaticParametersInterceptor.intercept(StaticParametersInterceptor.java:179)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at org.apache.struts2.interceptor.FileUploadInterceptor.intercept(FileUploadInterceptor.java:235)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.ModelDrivenInterceptor.intercept(ModelDrivenInterceptor.java:89)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.ChainingInterceptor.intercept(ChainingInterceptor.java:126)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.PrepareInterceptor.doIntercept(PrepareInterceptor.java:138)
at com.opensymphony.xwork2.interceptor.MethodFilterInterceptor.intercept(MethodFilterInterceptor.java:87)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at org.apache.struts2.interceptor.ServletConfigInterceptor.intercept(ServletConfigInterceptor.java:164)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.ParametersInterceptor.doIntercept(ParametersInterceptor.java:195)
at com.opensymphony.xwork2.interceptor.MethodFilterInterceptor.intercept(MethodFilterInterceptor.java:87)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at org.apache.struts2.interceptor.MultiselectInterceptor.intercept(MultiselectInterceptor.java:75)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at org.apache.struts2.interceptor.CheckboxInterceptor.intercept(CheckboxInterceptor.java:94)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.I18nInterceptor.intercept(I18nInterceptor.java:165)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.AliasInterceptor.intercept(AliasInterceptor.java:179)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.opensymphony.xwork2.interceptor.ExceptionMappingInterceptor.intercept(ExceptionMappingInterceptor.java:176)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at com.suning.aas.common.web.interceptor.ActionAccessTimeInterceptor.intercept(ActionAccessTimeInterceptor.java:96)
at com.opensymphony.xwork2.DefaultActionInvocation.invoke(DefaultActionInvocation.java:237)
at org.apache.struts2.impl.StrutsActionProxy.execute(StrutsActionProxy.java:52)
at org.apache.struts2.dispatcher.Dispatcher.serviceAction(Dispatcher.java:488)
at org.apache.struts2.dispatcher.ng.ExecuteOperations.executeAction(ExecuteOperations.java:77)
at org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter.doFilter(StrutsPrepareAndExecuteFilter.java:91)
at com.ibm.ws.webcontainer.filter.FilterInstanceWrapper.doFilter(FilterInstanceWrapper.java:188)
at com.ibm.ws.webcontainer.filter.WebAppFilterChain.doFilter(WebAppFilterChain.java:116)
at com.suning.aas.portal.web.filer.AuthFilter.doFilter(AuthFilter.java:163)
at com.ibm.ws.webcontainer.filter.FilterInstanceWrapper.doFilter(FilterInstanceWrapper.java:188)
at com.ibm.ws.webcontainer.filter.WebAppFilterChain.doFilter(WebAppFilterChain.java:116)
at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:88)
at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:76)
at com.ibm.ws.webcontainer.filter.FilterInstanceWrapper.doFilter(FilterInstanceWrapper.java:188)
at com.ibm.ws.webcontainer.filter.WebAppFilterChain.doFilter(WebAppFilterChain.java:116)
at com.ibm.ws.webcontainer.filter.WebAppFilterChain._doFilter(WebAppFilterChain.java:77)
at com.ibm.ws.webcontainer.filter.WebAppFilterManager.doFilter(WebAppFilterManager.java:908)
at com.ibm.ws.webcontainer.filter.WebAppFilterManager.invokeFilters(WebAppFilterManager.java:997)
at com.ibm.ws.webcontainer.extension.DefaultExtensionProcessor.invokeFilters(DefaultExtensionProcessor.java:985)
at com.ibm.ws.webcontainer.extension.DefaultExtensionProcessor.handleRequest(DefaultExtensionProcessor.java:905)
at com.ibm.ws.webcontainer.webapp.WebApp.handleRequest(WebApp.java:3826)
at com.ibm.ws.webcontainer.webapp.WebGroup.handleRequest(WebGroup.java:276)
at com.ibm.ws.webcontainer.WebContainer.handleRequest(WebContainer.java:931)
at com.ibm.ws.webcontainer.WSWebContainer.handleRequest(WSWebContainer.java:1583)
at com.ibm.ws.webcontainer.channel.WCChannelLink.ready(WCChannelLink.java:186)
at com.ibm.ws.http.channel.inbound.impl.HttpInboundLink.handleDiscrimination(HttpInboundLink.java:445)
at com.ibm.ws.http.channel.inbound.impl.HttpInboundLink.handleNewRequest(HttpInboundLink.java:504)
at com.ibm.ws.http.channel.inbound.impl.HttpInboundLink.processRequest(HttpInboundLink.java:301)
at com.ibm.ws.http.channel.inbound.impl.HttpICLReadCallback.complete(HttpICLReadCallback.java:83)
at com.ibm.ws.tcp.channel.impl.AioReadCompletionListener.futureCompleted(AioReadCompletionListener.java:165)
at com.ibm.io.async.AbstractAsyncFuture.invokeCallback(AbstractAsyncFuture.java:217)
at com.ibm.io.async.AsyncChannelFuture.fireCompletionActions(AsyncChannelFuture.java:161)
at com.ibm.io.async.AsyncFuture.completed(AsyncFuture.java:138)
at com.ibm.io.async.ResultHandler.complete(ResultHandler.java:204)
at com.ibm.io.async.ResultHandler.runEventProcessingLoop(ResultHandler.java:775)
at com.ibm.io.async.ResultHandler$2.run(ResultHandler.java:905)
at com.ibm.ws.util.ThreadPool$Worker.run(ThreadPool.java:1563)
```

注意到，它一直从具体代码所在的函数到所用框架层的函数最后到web容器层等函数都走了一遍。

形成敏感信息泄露的场景：如果开发者自己不去处理这个异常，最后默认会通过web容器暴露给用户，而这些异常信息都包含了应用所使用的组件名称等，对于攻击者来讲，增加了不少可利用的信息，导致敏感信息泄露。

乌云实际攻击利用案例\(在重要一个环节中被利用到\)：WooYun: 乐视网j2ee应用的安全问题！ 

###  666



