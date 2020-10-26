# Filter/Servlet型内存马的扫描抓捕与查杀

## 0x01 背景 <a id="0x01-&#x80CC;&#x666F;"></a>

在内存马横行的当下，蓝队or应急的师傅如何能快速判断哪些Filter/Servlet是内存马，分析内存马的行为功能是什么？最终又如何不重启的将其清除？红队师傅又如何抓铺其他师傅的内存马为自己用，亦或是把师傅的内存马踢掉？

在当下攻防对抗中，一直缺少着针对内存马扫描，捕捉与查杀的辅助脚本。下面就以`Tomcat 8.5.47`为例子，分享下编写方法，其他中间件万变不离其宗。

考虑到Agent技术针对红队来说比较重，我们这次使用jsp技术来解决以上问题。

## 0x02 扫描Filter和Servlet <a id="0x02-&#x626B;&#x63CF;Filter&#x548C;Servlet"></a>

要想扫描web应用内存中的Filter和Servlet，我们必须知道它们存储的位置。通过查看代码，我们知道StandardContext对象中维护的是一个

和Filter相关的是`filterDefs`和`filterMaps`两个属性。这两个属性分别维护着全局Filter的定义，以及Filter的映射关系。

![filterMaps&#x548C;filterRefs&#x5C5E;&#x6027;&#x7ED3;&#x6784;](https://gv7.me/articles/2020/filter-servlet-type-memshell-scan-capture-and-kill/filterMaps-filterRefs.png)

和Servlet相关的是`children`和`servletMappings`两个属性。这两个属性分别维护这全家Servlet的定义，以及Servlet的映射关系。

![servletMappings&#x5C5E;&#x6027;&#x7ED3;&#x6784;](https://gv7.me/articles/2020/filter-servlet-type-memshell-scan-capture-and-kill/servletMappings.png)

![children&#x5C5E;&#x6027;&#x7ED3;&#x6784;](https://gv7.me/articles/2020/filter-servlet-type-memshell-scan-capture-and-kill/children.png)

其他request对象中就存储这StandardContext对象。

```java
request.getSession().getServletContext() {ApplicationContextFacade}
  -> context {ApplicationContext} 
    -> context {StandardContext}
      * filterDefs
      * filterMaps
      * children
      * servletMappings
```

所以我们只需要通过反射遍历request，最终就可以拿到Filter和Servlet的如下信息。

* Filter/Servlet名
* 匹配路径
* Class名
* ClassLoader
* Class文件存储路径。
* 内存中Class字节码（方便反编译审计其是否存在恶意代码）
* 该Class是否有对应的磁盘文件（判断内存马的重要指标）

具体反射遍历代码放文末github，这里值得一提是拿到Class名通过如下方法就能拿到其被加载到内存中的字节码内容。

```java
byte[] classBytes = Repository.lookupClass(Class.forName("me.gv7.Memshell")).getBytes();
```

## 0x03 注销Filter内存马 <a id="0x03-&#x6CE8;&#x9500;Filter&#x5185;&#x5B58;&#x9A6C;"></a>

通过分析调试Tomcat源码，我们知道Tomcat注销filter其实就是将该Filter从全局filterDefs和filterMaps中清除掉。具体的操作分别如下`removeFilterDef`和`removeFilterMap`两个方法中。

```java
//org.apache.catalina.core.StandardContext#removeFilterDef
public void removeFilterDef(FilterDef filterDef) {
    synchronized(this.filterDefs) {
        this.filterDefs.remove(filterDef.getFilterName());
    }
    this.fireContainerEvent("removeFilterDef", filterDef);
}

//org.apache.catalina.core.StandardContext#removeFilterMap
public void removeFilterMap(FilterMap filterMap) {
    this.filterMaps.remove(filterMap);
    this.fireContainerEvent("removeFilterMap", filterMap);
}
```

我们只需要反射调用它们即可注销Filter。

```java
public synchronized void deleteFilter(HttpServletRequest request,String filterName) throws Exception{
    Object standardContext = getStandardContext(request);
    
    // org.apache.catalina.core.StandardContext#removeFilterDef
    HashMap<String,Object> filterConfig = getFilterConfig(request);
    Object appFilterConfig = filterConfig.get(filterName);
    Field _filterDef = appFilterConfig.getClass().getDeclaredField("filterDef");
    _filterDef.setAccessible(true);
    Object filterDef = _filterDef.get(appFilterConfig);
    Method removeFilterDef = standardContext.getClass().getDeclaredMethod("removeFilterDef", new Class[]{org.apache.tomcat.util.descriptor.web.FilterDef.class});
    removeFilterDef.setAccessible(true);
    removeFilterDef.invoke(standardContext,filterDef);
    
    // org.apache.catalina.core.StandardContext#removeFilterMap
    Object[] filterMaps = getFilterMaps(request);
    for(Object filterMap:filterMaps){
        Field _filterName = filterMap.getClass().getDeclaredField("filterName");
        _filterName.setAccessible(true);
        String filterName0 = (String)_filterName.get(filterMap);
        if(filterName0.equals(filterName)){
            Method removeFilterMap = standardContext.getClass().getDeclaredMethod("removeFilterMap", new Class[]{org.apache.catalina.deploy.FilterMap.class});
            removeFilterDef.setAccessible(true);
            removeFilterMap.invoke(standardContext,filterMap);
        }
    }
}
```

## 0x04 注销Servlet内存马 <a id="0x04-&#x6CE8;&#x9500;Servlet&#x5185;&#x5B58;&#x9A6C;"></a>

注销Servlet的原理也是类似，将该Servlet从全局servletMappings和children中清除掉即可。在Tomcat源码中对应的是`removeServletMapping`和`removeChild`方法。

```java
//org.apache.catalina.core.StandardContext#removeServletMapping
public void removeServletMapping(String pattern) {
    String name = null;
    synchronized(this.servletMappingsLock) {
        name = (String)this.servletMappings.remove(pattern);
    }

    Wrapper wrapper = (Wrapper)this.findChild(name);
    if (wrapper != null) {
        wrapper.removeMapping(pattern);
    }

    this.fireContainerEvent("removeServletMapping", pattern);
}

//org.apache.catalina.core.StandardContext#removeChild
public void removeChild(Container child) {
    if (!(child instanceof Wrapper)) {
        throw new IllegalArgumentException(sm.getString("standardContext.notWrapper"));
    } else {
        super.removeChild(child);
    }
}
```

我们只需要反射调用它们即可注销Servlet。

```java
public synchronized void deleteServlet(HttpServletRequest request,String servletName) throws Exception{
    HashMap<String,Object> childs = getChildren(request);
    Object objChild = childs.get(servletName);
    String urlPattern = null;
    HashMap<String,String> servletMaps = getServletMaps(request);
    for(Map.Entry<String,String> servletMap:servletMaps.entrySet()){
        if(servletMap.getValue().equals(servletName)){
            urlPattern = servletMap.getKey();
            break;
        }
    }

    if(urlPattern != null) {
        // 反射调用 org.apache.catalina.core.StandardContext#removeServletMapping
        Object standardContext = getStandardContext(request);
        Method removeServletMapping = standardContext.getClass().getDeclaredMethod("removeServletMapping", new Class[]{String.class});
        removeServletMapping.setAccessible(true);
        removeServletMapping.invoke(standardContext, urlPattern);
        // Tomcat 6必须removeChild 789可以不用
        // 反射调用 org.apache.catalina.core.StandardContext#removeChild
        Method removeChild = standardContext.getClass().getDeclaredMethod("removeChild", new Class[]{org.apache.catalina.Container.class});
        removeChild.setAccessible(true);
        removeChild.invoke(standardContext, objChild);
    }
}
```

## 0x05 演示 <a id="0x05-&#x6F14;&#x793A;"></a>

我们只需要把编写好的 `tomcat-memshell-scanner.jsp` 放到可能被注入内存的web项目中，然后通过浏览器访问即可。假设扫描结果如下：

![](https://gv7.me/articles/2020/filter-servlet-type-memshell-scan-capture-and-kill/tomcat-memshell-scan-result.png)

通过分析扫描出的信息，可知`filter-b2b1cad2-44be-4f43-8db0-bd43da5ad368`是Filter型内存马，原因如下：

1. classLoader是可疑的`com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl$TransletClassLoader`,这是反序列化漏洞执行代码用的classLoader。
2. class在磁盘中没有对应的class文件，只驻留在内存。

`/favicon.ico`是Servlet型内存马，判断原因如下。

1. classLoader是自定义classLoader,当下比较流行的java webshell基本都是自定义了class loader来实现任意代码执行。
2. class在磁盘中没有对应的class文件，只驻留在内存。

最后我们可以dump出那么对应的class，反编译看代码分析`filter-b2b1cad2-44be-4f43-8db0-bd43da5ad368`是Filter型cmd内存马，`/favicon.ico`是Servlet型哥斯拉内存马。

