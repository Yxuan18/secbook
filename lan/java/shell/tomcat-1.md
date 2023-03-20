# Tomcat 内存马检测

随着HW、攻防对抗的强度越来越高，各大厂商对于webshell的检测技术愈发成熟，对于攻击方来说，传统的文件落地webshell的生存空间越来越小，无文件webshell已经逐步成为新的研究趋势。

三月底针对tomcat内存马的检测写了一个demo，但由于对Maven打包理解不深，整个项目结构比较糟糕。

国庆前研究了LandGrey师傅的[copagent项目](https://github.com/LandGrey/copagent)，在该项目基础上进行了重构，并于本文中记录了检测思路，以及部分核心代码。

作者： jweny [@360](https://github.com/360)云安全

## 0x01 Java内存马简介 <a href="#h2-0" id="h2-0"></a>

关于JAVA内存马的发展历史，这里引用下 [c0ny1师傅的总结](https://gv7.me/articles/2020/kill-java-web-filter-memshell/) 。早在17年n1nty师傅的[《Tomcat源码调试笔记-看不见的shell》](https://mp.weixin.qq.com/s/x4pxmeqC1DvRi9AdxZ-0Lw)中已初见端倪，但一直不温不火。后经过rebeyond师傅使用[agent技术](https://gv7.me/articles/2020/kill-java-web-filter-memshell/\(https://www.cnblogs.com/rebeyond/p/9686213.html))加持后，拓展了内存马的使用场景，然终停留在奇技淫巧上。在各类hw洗礼之后，文件shell明显气数已尽。内存马以救命稻草的身份重回大众视野。特别是今年在shiro的回显研究之后，引发了无数安全研究员对内存webshell的研究，其中涌现出了LandGrey师傅构造的[Spring controller内存马](https://landgrey.me/blog/12/)。

从攻击对象来说，可以将Java内存马分为以下几类：

1. 1.servlet-api
   * [filter型](https://mp.weixin.qq.com/s/x4pxmeqC1DvRi9AdxZ-0Lw)
   * [servlet型](https://www.cnblogs.com/potatsoSec/p/13195183.html)
   * [listener型](https://www.anquanke.com/post/id/214483#h3-5)
2. 2.指定框架，如[spring](https://landgrey.me/blog/12/)
3. 3.[字节码增强型](https://www.cnblogs.com/rebeyond/p/9686213.html)
4. 4.[任意JSP文件隐藏](https://mp.weixin.qq.com/s/1ZiLD396088TxiW\_dUOFsQ)

为方便学习，webshell demo已整理至[github](https://github.com/jweny/MemShellDemo/tree/master/MemShellForJava)。

## 0x02 整体思路 <a href="#h2-1" id="h2-1"></a>

无论是以上哪种攻击方式，影响的均为加载到tomcat jvm中的类。

从防守方的角度来说，可以通过java instrumentation机制，将检测jar包attach到tomcat jvm，检查加载到jvm中的类是否异常。

整体检测思路为：

1. 1.获取tomcat jvm中所有加载的类
2. 2.遍历每个类，判断是否为风险类。这里把**可能被攻击方新增/修改内存中的类，标记为风险类**（比如实现了filter/servlet的类）
3. 3.遍历风险类，检查是否为webshell：
   * 检查高风险类的class文件是否存在；
   * 反编译风险类字节码，检查java文件中包含恶意代码

## 0x03 获取jvm中所有加载的类 <a href="#h2-2" id="h2-2"></a>

1. 1.遍历java jvm，查找所有的tomcat jvm
2. 2.通过java instrumentation，将agent attach到每个tomcat jvm。由于可能存在多个tomcat进程的场景，因此每个tomcat jvm均检测一遍

```java
// 应对存在多个 tomcat 进程的情况
public static void attach(String agent_jar_path) throws Exception {
    VirtualMachine virtualMachine = null;
    for (VirtualMachineDescriptor descriptor : VirtualMachine.list()) {
        if (descriptor.displayName().contains("catalina") || descriptor.displayName().equals("")) {
            try {
                virtualMachine = VirtualMachine.attach(descriptor);
                Properties targetSystemProperties = virtualMachine.getSystemProperties();
                if (descriptor.displayName().equals("") && !targetSystemProperties.containsKey("catalina.home"))
                    continue;
                // 将当前tomcat pid，传到agent，作为检测结果的文件名，用来区分多个tomcat进程。
                String currentJvmName = "tomcat_" + descriptor.id();
                Thread.sleep(1000);
                javaInfoWarning(targetSystemProperties);
                virtualMachine.loadAgent(agent_jar_path, currentJvmName);
            } catch (Throwable t) {
                t.printStackTrace();
            } finally {
                    // detach
                if (null != virtualMachine)
                    virtualMachine.detach();
            }
        }
    }
}
```

1. 3.遍历tomcat jvm 加载过的类

```java
private static synchronized void detectMemShell(String currentJvmName, Instrumentation ins) {
    // 获取所有加载的类
    Class[] loadedClasses = ins.getAllLoadedClasses();
}
```

## 0x04 风险类识别 <a href="#h2-3" id="h2-3"></a>

最理想的做法是把所有加载的类都认定为风险类。但在绝大多数情况下jvm加载的都是正常的类，每次检查时，都dump所有加载的类，对于tomcat来说开销有点大。

**比较实际的做法是，根据已知内存马要新增/修改的类生成特征。**

**对于内存中的每一个类，检查其自身，并递归检查其父类，如果命中特征，就标记为风险类。**

```java
public static List> findAllSuspiciousClass (Instrumentation ins, Class[] loadedClasses){
    // 结果
    List> suspiciousClassList = new ArrayList>();
    List loadedClassesNames = new ArrayList();
    // 获取所有风险类
    for (Class clazz : loadedClasses) {
        loadedClassesNames.add(clazz.getName());
        // 递归 检查class的父类 空或java.lang.Object退出
        while (clazz != null && !clazz.getName().equals("java.lang.Object")) {
            if (
                    ClassUtils.lsContainRiskPackage(clazz) ||
                            ClassUtils.isUseAnnotations(clazz) ||
                            ClassUtils.lsHasRiskSuperClass(clazz) ||
                            ClassUtils.lsRiskClassName(clazz) ||
                            ClassUtils.lsReleaseRiskInterfaces(clazz)
            ){
                if (loadedClassesNames.contains(clazz.getName())) {
                    suspiciousClassList.add(clazz);
                    ClassUtils.dumpClass(ins, clazz.getName(), false,
                            Integer.toHexString(clazz.getClassLoader().hashCode()));
                    break;
                }
                LogUtils.logToFile("cannot find " + clazz.getName() + " classes in instrumentation");
                break;
            }
            clazz = clazz.getSuperclass();
        }
    }
    return suspiciousClassList;
}
```

这里借鉴了[LandGrey师傅](https://github.com/LandGrey/copagent)的黑名单，将内存马的目标类的类名、继承类、实现类、所属的包、使用的注解均设置黑名单。

### &#x20;1. 实现类黑名单 <a href="#h3-4" id="h3-4"></a>

检测类是否实现javax.servlet.Filter / javax.servlet.Servlet / javax.servlet.ServletRequestListener接口类。

```java
// 检测类是否实现高风险接口，如servlet/filter/Listener
public static Boolean lsReleaseRiskInterfaces(Class clazz){
    // 高风险的接口
    List riskInterface = new ArrayList();
    // filter型
    riskInterface.add("javax.servlet.Filter");
    // servlet型
    riskInterface.add("javax.servlet.Servlet");
    // listener型
    riskInterface.add("javax.servlet.ServletRequestListener");
    try {
        // 获取类实现的interface
        List clazzInterfaces = new ArrayList();
        for (Class cls : clazz.getInterfaces())
            clazzInterfaces.add(cls.getName());
        // 两个list有交集 返回true
        clazzInterfaces.retainAll(riskInterface);
        if(clazzInterfaces.size()>0){
            return Boolean.TRUE;
        }
    } catch (Throwable ignored) {}
    return Boolean.FALSE;
}
```

### &#x20;2. 继承类黑名单 <a href="#h3-5" id="h3-5"></a>

```java
// 检测父类是否属于高风险
public static Boolean lsHasRiskSuperClass(Class clazz) {
    // 高风险的父类
    List riskSuperClassesName = new ArrayList();
    riskSuperClassesName.add("javax.servlet.http.HttpServlet");
    try {
        if ((clazz.getSuperclass() != null
                && riskSuperClassesName.contains(clazz.getSuperclass().getName())
        )){
            return Boolean.TRUE;
        }
    }catch (Throwable ignored) {}
    return Boolean.FALSE;
}
```

### &#x20;3. 注解黑名单 <a href="#h3-6" id="h3-6"></a>

通过clazz.getDeclaredAnnotations() 获取所有注解，如果类使用了spring注册路由的注解，则标记为高风险。

```java
public static Boolean isUseAnnotations(Class clazz) {
    // 针对spring注册路由的一些注解
    List riskAnnotations = new ArrayList();
    riskAnnotations.add("org.springframework.stereotype.Controller");
    riskAnnotations.add("org.springframework.web.bind.annotation.RestController");
    riskAnnotations.add("org.springframework.web.bind.annotation.RequestMapping");
    riskAnnotations.add("org.springframework.web.bind.annotation.GetMapping");
    riskAnnotations.add("org.springframework.web.bind.annotation.PostMapping");
    riskAnnotations.add("org.springframework.web.bind.annotation.PatchMapping");
    riskAnnotations.add("org.springframework.web.bind.annotation.PutMapping");
    riskAnnotations.add("org.springframework.web.bind.annotation.Mapping");
    try {
        // 获取所有注解
        Annotation[] da = clazz.getDeclaredAnnotations();
        if (da.length > 0)
            for (Annotation _da : da) {
                // 比较 注解 && 高风险注解 如果有交集 返回True
                for (String _annotation : riskAnnotations) {
                    if (_da.annotationType().getName().equals(_annotation))
                        return Boolean.TRUE;
                }
            }
    } catch (Throwable ignored) {}
    return Boolean.FALSE;
}
```

### &#x20;4. 类名黑名单 <a href="#h3-7" id="h3-7"></a>

```java
// 高风险的类名
public static Boolean lsRiskClassName(Class clazz){
    List riskClassName = new ArrayList();
    riskClassName.add("org.springframework.web.servlet.handler.AbstractHandlerMapping");
    try {
        if (riskClassName.contains(clazz.getName())){
            return Boolean.TRUE;
        }
    }catch (Throwable ignored) {}
    return Boolean.FALSE;
}
```

### &#x20;5. 包名黑名单 <a href="#h3-8" id="h3-8"></a>

```java
// 检测是否属于高风险的包
public static Boolean lsContainRiskPackage(Class clazz){
    // 高风险的包
    List riskPackage = new ArrayList();
    riskPackage.add("net.rebeyond.");
    riskPackage.add("com.metasploit.");
    try {
        for (String packageName : riskPackage) {
            if (clazz.getName().startsWith(packageName)) {
                return Boolean.TRUE;
            }
        }
    }catch (Throwable ignored) {}
    return Boolean.FALSE;
}
```

### &#x20;6. 基于mbean的filter/servlet风险类识别 <a href="#h3-9" id="h3-9"></a>

这里分享另一种filter/servlet的检测，检测思路是通过mbean获取sevlet/filter列表，内存马的filter是动态注册的，所以web.xml中肯定没有相应配置，因此通过对比可以发现异常的filter。

```java
MBeanServer mbs = ManagementFactory.getPlatformMBeanServer();
Object mbsInte = getFieldValue(mbs, "mbsInterceptor");
Object repository = getFieldValue(mbsInte, "repository");
Object domainTb = getFieldValue(repository, "domainTb");
Map catlina = (Map)((Map)domainTb).get("Catalina");
for (Map.Entry entry : catlina.entrySet()) {
  String key = entry.getKey();
  // servlet
  if (key.contains("j2eeType=Servlet")){...}
  // filter 
  if (key.contains("j2eeType=Servlet") && key.contains("name=jsp")){
    Object value = entry.getValue();
    Object obj = getFieldValue(value,"object");
    Object res = getResourceValue(obj);
    Object instance = getFieldValue(res,"instance");
    Object rctxt = getFieldValue(instance, "rctxt");
    Object context = getFieldValue(instance, "context");
    Object appContext = getFieldValue(context,"context");
    Object standardContext = getFieldValue(appContext,"context");
    Object filterConfigs = getFieldValue(standardContext,"filterConfigs");
    ...
```

不过这种方式有较大的缺陷。首先，mbean只是资源管理，并不影响功能，所以在植入内存马后再卸载掉注册的mbean即可绕过；其次，servlet 3.0引入了 [@WebFilter](https://github.com/WebFilter) 可以动态注册，这种也没有在web.xml中配置，会引起误报，因此仅可作为一个查找风险类的参考条件。

## 0x05 检测是否为内存马 <a href="#h2-10" id="h2-10"></a>

遍历风险类列表，并检测以下规则：

1. 内存马，对应的ClassLoader目录下没有对应的class文件

```java
public static Boolean checkClassIsNotExists(Class clazz){
    String className = clazz.getName();
    String classNamePath = className.replace(".","/") + ".class";
    URL isExists = clazz.getClassLoader().getResource(classNamePath);
    if (isExists == null){
        return Boolean.TRUE;
    }
    return Boolean.FALSE;
}
```

1. 反编译该类的字节码，检查是否存在危险函数

```java
public static Boolean checkFileContentIsRisk(File dumpPath){
    List riskKeyword = new ArrayList();
    riskKeyword.add("javax.crypto.");
    riskKeyword.add("ProcessBuilder");
    riskKeyword.add("getRuntime");
    riskKeyword.add("ProcessImpl");
    riskKeyword.add("shell");
    String content = PathUtils.getFileContent(dumpPath);
    for (String keyword : riskKeyword) {
        if (content.contains(keyword)) {
            return Boolean.TRUE;
        }
    }
```

结果输出参考：如果没有class文件，可将该类风险等级标为high。如果包含恶意代码，将该类风险等级调至最高级。

```java
// 输出结果
public static String getClassRiskLevel(Class clazz, File dumpPath) {
    String riskLevel = "Low";
    // 检测 Classloader目录下是否存在class文件
    if (AnalysisUtils.checkClassIsNotExists(clazz)){
        riskLevel = "high";
    }
    // 反编译  检测java文件是否包含执行命令的危险函数
    if (AnalysisUtils.checkFileContentIsRisk(dumpPath)){
        riskLevel = "Absolutely";
    }
    return riskLevel;
}
```

## 0x06 小结 <a href="#h2-11" id="h2-11"></a>

本文仅对Tomcat内存马的检测提供了一些思路，但并未提及查杀，查杀将在下一篇详细分享。

以上所有方法的黑名单列表仅供参考，可自行更改、扩充。

再次感谢 **fnmsd、c0ny1、LandGrey** 师傅们的大力支持。

## 0x07 参考文章 <a href="#h2-12" id="h2-12"></a>

* [https://github.com/LandGrey/copagent](https://github.com/LandGrey/copagent)
* [查杀Java web filter型内存马](https://gv7.me/articles/2020/kill-java-web-filter-memshell/)
* [Tomcat源码调试笔记-看不见的shell](https://mp.weixin.qq.com/s/x4pxmeqC1DvRi9AdxZ-0Lw)
* [利用“进程注入”实现无文件不死webshell](https://www.cnblogs.com/rebeyond/p/9686213.html)
* [基于内存 Webshell 的无文件攻击技术研究](https://landgrey.me/blog/12/)
* [基于tomcat的内存 Webshell 无文件攻击技术](https://xz.aliyun.com/t/7388)
