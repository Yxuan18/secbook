# 查杀Java web filter型内存马

### 0x01 内存马简历史 <a id="0x01-&#x5185;&#x5B58;&#x9A6C;&#x7B80;&#x5386;&#x53F2;"></a>

其实内存马由来已久，早在17年n1nty师傅的[《Tomcat源码调试笔记-看不见的shell》](https://mp.weixin.qq.com/s/x4pxmeqC1DvRi9AdxZ-0Lw)中已初见端倪，但一直不温不火。后经过rebeyong师傅使用[agent技术](https://gv7.me/articles/2020/kill-java-web-filter-memshell/%28https://www.cnblogs.com/rebeyond/p/9686213.html%29)加持后，拓展了内存马的使用场景，然终停留在奇技淫巧上。在各类hw洗礼之后，文件shell明显气数已尽。内存马以救命稻草的身份重回大众视野。特别是今年在shiro的回显研究之后，引发了无数安全研究员对内存webshell的研究，其中涌现出了LandGrey师傅构造的[Spring controller内存马](https://landgrey.me/blog/12/)。至此内存马开枝散叶发展出了三大类型：

1. servlet-api类
   * filter型
   * servlet型
2. spring类
   * 拦截器
   * controller型
3. Java Instrumentation类
   * agent型

内存马这坛深巷佳酒，一时间流行于市井与弄堂之间。上至安全研究员下至普通客户，人尽皆知。正值hw来临之际，不难推测届时必将是内存马横行天下之日。而各大安全厂商却迟迟未见动静。所谓表面风平浪静，实则暗流涌动。或许一场内存马的围剿计划正慢慢展开。作为攻击方向的研究人员，没有对手就制造对手,攻防互换才能提升内存马技术的发展。

### 0x02 查杀思路 <a id="0x02-&#x67E5;&#x6740;&#x601D;&#x8DEF;"></a>

我们判断逻辑很朴实，利用Java Agent技术遍历所有已经加载到内存中的class。先判断是否是内存马，是则进入内存查杀。

```java
public class Transformer implements ClassFileTransformer {
	public byte[] transform(ClassLoader classLoader, String s, Class<?> aClass, ProtectionDomain protectionDomain, byte[] bytes) throws IllegalClassFormatException {
	    // 识别内存马
	    if(isMemshell(aClass,bytes)){
	        // 查杀内存马
	        byte[] newClassByte = killMemshell(aClass,bytes);
	        return newClassByte;
	    }else{
	        return bytes;
	    }
    }
}
```

### 0x03 内存马的识别 <a id="0x03-&#x5185;&#x5B58;&#x9A6C;&#x7684;&#x8BC6;&#x522B;"></a>

要识别，我们就需要细思内存马有什么特征。下面列下我思考过的检查点。

1. filter名字很特别

内存马的Filter名一般比较特别，有`shell`或者随机数等关键字。这个特征稍弱，因为这取决于内存马的构造者的习惯，构造完全可以设置一个看起来很正常的名字。

1. filter优先级是第一位

为了确保内存马在各种环境下都可以访问，往往需要把filter匹配优先级调至最高，这在shiro反序列化中是刚需。但其他场景下就非必须，只能做一个可疑点。

1. 对比web.xml中没有filter配置

内存马的Filter是动态注册的，所以在web.xml中肯定没有配置，这也是个可以的特征。但servlet 3.0引入了`@WebFilter`标签方便开发这动态注册Filter。这种情况也存在没有在web.xml中显式声明，这个特征可以作为较强的特征。

1. 特殊classloader加载

我们都知道Filter也是class，也是必定有特定的classloader加载。一般来说，正常的Filter都是由中间件的WebappClassLoader加载的。反序列化漏洞喜欢利用TemplatesImpl和bcel执行任意代码。所以这些class往往就是以下这两个：

* com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl$TransletClassLoader
* com.sun.org.apache.bcel.internal.util.ClassLoader

这个特征是一个特别可疑的点了。当然了，有的内存马还是比较狡猾的，它会注入class到当前线程中，然后实例化注入内存马。这个时候内存马就有可能不是上面两个classloader。

1. 对应的classloader路径下没有class文件

所谓内存马就是代码驻留内存中，本地无对应的class文件。所以我们只要检测Filter对应的ClassLoader目录下是否存在class文件。

```java
private static boolean classFileIsExists(Class clazz){
    if(clazz == null){
        return false;
    }

    String className = clazz.getName();
    String classNamePath = className.replace(".", "/") + ".class";
    URL is = clazz.getClassLoader().getResource(classNamePath);
    if(is == null){
        return false;
    }else{
        return true;
    }
}
```

1. Filter的doFilter方法中有恶意代码

我们可以把内存中所有的Filter的class dump出来，使用`fernflower`等反编译工具分析看看，是否存在恶意代码，比如调用了如下可疑的方法：

* java.lang.Runtime.getRuntime
* defineClass
* invoke
* …

不难分析，内存马的命门在于`5`和`6`。简单说就是Filter型内存马首先是一个Filter类，同时它在硬盘上没有对应的class文件。若dump出的class还有恶意代码，那是内存马无疑啦。大致检查的代码如下：

```java
private static boolean isMemshell(Class targetClass,byte[] targetClassByte){
    ClassLoader classLoader = null;
    if(targetClass.getClassLoader() != null) {
        classLoader = targetClass.getClassLoader();
    }else{
        classLoader = Thread.currentThread().getContextClassLoader();
    }

    Class clsFilter =  null;
    try {
        clsFilter = classLoader.loadClass("javax.servlet.Filter");
    }catch (Exception e){
    }

    // 是否是filter
    if(clsFilter != null && clsFilter.isAssignableFrom(targetClass)){
        // class loader 是不是Templates或bcel
        if(classLoader.getClass().getName().contains("com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl$TransletClassLoader")
                || classLoader.getClass().getName().contains("com.sun.org.apache.bcel.internal.util.ClassLoader")){
            return true;
        }

        // 是否存在ClassLoader的文件目录下存在对应的class文件
        if(classFileIsExists(targetClass)){
            return true;
        }
        
        // filter是否包含恶意代码。
        String[] blacklist = new String[]{"getRuntime","defineClass","invoke"};
        String clsJavaCode = FernflowerUtils.decomper(targetClass,targetClassByte);
        for(String b:blacklist){
            if(clsJavaCode.contains(b)){
                return true;
            }
        }
    }else{
        return false;
    }
    return false;
}
```

PS: 本文讨论查杀的思路，给出的代码只是概念正面的伪装代码。完美的方案是将以上6点作为判断指标，并根据指标的重要性赋予不同权重。满足的条件越多越可能是内存马。

### 0x04 内存马的查杀 <a id="0x04-&#x5185;&#x5B58;&#x9A6C;&#x7684;&#x67E5;&#x6740;"></a>

内存马识别完成，接下来就是如何查杀了。

方法一： 清除内存马中的Filter的恶意代码

```java
public static byte[] killMemshell(Class clsMemshell,byte[] byteMemshell) throws Exception{
    File file = new File(String.format("/tmp/%s.class",clsMemshell.getName()));
    if(file.exists()){
        file.delete();
    }
    FileOutputStream fos  = new FileOutputStream(file.getAbsoluteFile());
    fos.write(byteMemshell);
    fos.flush();
    fos.close();
    ClassPool cp = ClassPool.getDefault();
    cp.insertClassPath("/tmp/");
    CtClass cc = cp.getCtClass(clsMemshell.getName());
    CtMethod m = cc.getDeclaredMethod("doFilter");
    m.addLocalVariable("elapsedTime", CtClass.longType);
    // 正确覆盖代码：
    // m.setBody("{$3.doFilter($1,$2);}");
    // 方便演示代码：
    m.setBody("{$2.getWriter().write(\"Your memory horse has been killed by c0ny1\");}");
    byte[] byteCode = cc.toBytecode();
    cc.detach();
    return byteCode;
}
```

方法二： 模拟中间件注销Filter

```java
//反序列化执行代码反射获取到StandardContext
Object standardContext = ...;
Field _filterConfigs = standardContext.getClass().getDeclaredField("filterConfigs");
_filterConfigs.setAccessible(true);
Object filterConfigs = _filterConfigs.get(standardContext);
Map<String, ApplicationFilterConfig> filterConfigMap = (Map<String, ApplicationFilterConfig>)filterConfigs;
for(Map.Entry<String, ApplicationFilterConfig> map : filterConfigMap.entrySet()){
    String filterName = map.getKey();
    ApplicationFilterConfig filterConfig = map.getValue();
    Filter filterObject = filterConfig.getFilter();
    // 如果是内存马的filter名
    if(filterName.startsWith("memshell")){
        SecurityUtil.remove(filterObject);
        filterConfigMap.remove(filterName);
    }
}
```

两种方法各有优劣，第一种方法比较通用，直接适配所有中间件。但恶意Filter依然在，只是恶意代码被清除了。第二种方法比较优雅，恶意Filter会被清除掉。但每种中间件注销Filter的逻辑不尽相同，需要一一适配。为了方便演示我们选第一种。

### 0x05 demo展示 <a id="0x05-demo&#x5C55;&#x793A;"></a>

最后给大家展示下，我查杀demo的效果。

查杀演示

![&#x67E5;&#x6740;&#x6F14;&#x793A;](https://gv7.me/articles/2020/kill-java-web-filter-memshell/kill-java-filter-memshell-demo.gif)

### 0x06 总结 <a id="0x06-&#x603B;&#x7ED3;"></a>

本文我们对Filter型内存马的识别与查杀做了细致的分析，其实Servlet型，拦截器型和Controller型的查杀方法也是万变不离其中，可如法炮制。但这样的思路无法查杀Agent型内存马，Agent型内存马查杀难点在“查”不在“杀”，具体的难点在那，又是如何解决呢？我会在后续的《查杀Java web Agent型内存马》中继续分享我的思考。

