# 浅析Java命令执行

**Java执行命令的3种方法**

**首先了解下在Java中执行命令的方法：**

常用的是  java.lang.Runtime\#exec\(\) 和  java.lang.ProcessBuilder\#start\(\) ，除此之外，还有更为底层的 java.lang.ProcessImpl\#start\(\) ，他们的调用关系如下图所示：

![](../../.gitbook/assets/image%20%28698%29.png)

其中，ProcessImpl 类是 Process 抽象类的具体实现，且该类的构造函数使用 private 修饰，所以无法在 java.lang 包外直接调用，只能通过反射调用 ProcessImpl\#start\(\) 方法执行命令。

![](../../.gitbook/assets/image%20%28693%29.png)

**这3种执行方法如下：**

## java.lang.Runtime

```text
public static String RuntimeTest() throws Exception {    InputStream ins = Runtime.getRuntime().exec("whoami").getInputStream();    ByteArrayOutputStream bos = new ByteArrayOutputStream();byte[] bytes = new byte[1024];int size;while((size = ins.read(bytes)) > 0)        bos.write(bytes,0,size);return bos.toString();}
```

![](../../.gitbook/assets/image%20%28704%29.png)

## java.lang.ProcessBuilder

```java
public static String ProcessTest() throws Exception {
  String[] cmds = {"cmd","/c","whoami"};
    InputStream ins = new ProcessBuilder(cmds).start().getInputStream();
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
byte[] bytes = new byte[1024];
int size;
while((size = ins.read(bytes)) > 0)
        bos.write(bytes,0,size);
return bos.toString();
}
```

![](../../.gitbook/assets/image%20%28682%29.png)

## java.lang.ProcessImpl

```java
public static String ProcessImplTest() throws Exception {
    String[] cmds = {"whoami"};
    Class clazz = Class.forName("java.lang.ProcessImpl");
    Method method = clazz.getDeclaredMethod("start", new String[]{}.getClass(),Map.class,String.class,ProcessBuilder.Redirect[].class,boolean.class);
    method.setAccessible(true);
    InputStream ins = ((Process) method.invoke(null,cmds,null,".",null,true)).getInputStream();
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
byte[] bytes = new byte[1024];
int size;
while((size = ins.read(bytes)) > 0)
      bos.write(bytes,0,size);
return bos.toString();
}
```

问题：

当直接将命令字符 echo echo\_test &gt; echo.txt 传给 java.lang.Runtime\#exec\(\)执行时报错：

![](../../.gitbook/assets/image%20%28717%29.png)

加上cmd /c 可以成功执行：

![](../../.gitbook/assets/image%20%28691%29.png)

我们跟进下代码看看是什么原因导致的？

命令执行解析流程：

传入命令字符串echo echo\_test &gt; echo.txt进行调试，跟进java.lang.Runtime\#exec\(String\)，该方法又会调用java.lang.Runtime\#exec\(String,String\[\],File\)。

![](../../.gitbook/assets/image%20%28701%29.png)

![](../../.gitbook/assets/image%20%28695%29.png)

在该方法中调用了StringTokenizer类，通过特定字符对命令字符串进行分割，本地测试如下：

![](../../.gitbook/assets/image%20%28702%29.png)

所以命令字符串echo echo\_test &gt; echo.txt经过StringTokenizer类处理后得到命令数组:{"echo","echo\_test","&gt;","echo.txt"} 。另外java.lang.Runtime\#exec\(\)共有6个重载方法，代码如下：

```java
public Process exec(String command) throws IOException {return exec(command, null, null);}public Process exec(String cmdarray[]) throws IOException {return exec(cmdarray, null, null);}  public Process exec(String command, String[] envp) throws IOException {return exec(command, envp, null);}public Process exec(String command, String[] envp, File dir)throws IOException {if (command.length() == 0)throw new IllegalArgumentException("Empty command");  StringTokenizer st = new StringTokenizer(command);  String[] cmdarray = new String[st.countTokens()];for (int i = 0; st.hasMoreTokens(); i++)    cmdarray[i] = st.nextToken();return exec(cmdarray, envp, dir);}public Process exec(String[] cmdarray, String[] envp) throws IOException {return exec(cmdarray, envp, null);}public Process exec(String[] cmdarray, String[] envp, File dir)throws IOException {return new ProcessBuilder(cmdarray)    .environment(envp)    .directory(dir)    .start();}
```

这6个重载函数根据参数不同进行区分，主要是传入字符串跟数组两种形式，但是最终调用的都是最后一个exec\(String\[\],String\[\],File\)，在该函数内部首先调用ProcessBuilder类的构造函数创建ProcessBuilder对象，然后调用start\(\)，最终返回一个Process对象。

所以Runtime\#exec\(\)底层还是调用的ProcessBuilder\#start\(\),且传入构造函数的参数要求是数组类型\(如下图\)，所以传给Runtime\#exec\(\)的命令字符串需要先使用StringTokenizer类分割为数组再传入ProcessBuilder类。

![](../../.gitbook/assets/image%20%28677%29.png)

接着跟进java.lang.ProcessBuilder\#start\(\)，取出cmdarray\[0\]赋值给prog,如果安全管理器SecurityManager开启,会调用SecurityManager\#checkExec\(\)对执行程序prog进行检查，之后调用ProcessImpl\#start\(\)。

![](../../.gitbook/assets/image%20%28705%29.png)

跟进 java.lang.ProcessImpl\#start\(\) ，Windows 下会调用 ProcessImpl 类的构造方法，如果是 Linux 环境，则会调用 java.lang.UNIXProcess\#init&lt;&gt; 。

![](../../.gitbook/assets/image%20%28706%29.png)

### **跟进java.lang.ProcessImpl的构造方法**

该方法内allowAmbiguousCommands变量为 "是否允许调用本地进程" 的开关，在安全管理器未开启且jdk.lang.Process.allowAmbiguousCommands不为false时，allowAmbiguousCommands变量值才为true。当系统允许调用本地进程时，进入Legacy mode\(传统模式\)，会调用needsEscaping\(\)，当prog存在空格且未被双引号包裹时需要使用quoteString\(\)进行处理，接着调用createCommandLine\(\)将命令数组拼接为命令字符串，最后调用create\(\)创建进程。

![](../../.gitbook/assets/image%20%28678%29.png)

传统模式下，当可执行程序prog存在\t 或空格时，该函数返回true，即需要双引号包裹处理。

![](../../.gitbook/assets/image%20%28690%29.png)

![](../../.gitbook/assets/image%20%28694%29.png)

最后调用ProcessImpl\#create\(\)，这是一个native方法，根据JNI命名规则，会调用到ProcessImpl\_md.c 中的Java\_Java\_lang\_ProcessImpl\_create\(\)，该函数会调用Windows系统API函数：CreateProcessW\(\)，用来创建一个新的Windows进程。创建成功后，将新进程的句柄返回给ProcessImpl\#create\(\)。

![](../../.gitbook/assets/image%20%28716%29.png)

看下CreateProcessW\(\)怎么处理我们传入的命令的：当第一个参数\(lpApplicationName\)为0时，第二个参数pcmd\(lpCommandLine\)需要提供启动程序及所需参数，彼此间以空格隔开。

![](../../.gitbook/assets/image%20%28722%29.png)

![](../../.gitbook/assets/image%20%28696%29.png)

![](../../.gitbook/assets/image%20%28684%29.png)

测试 ProcessImpl\#create\(\) 方法：

![](../../.gitbook/assets/image%20%28710%29.png)

加上cmd /c之后，成功执行命令：

![](../../.gitbook/assets/image%20%28689%29.png)

### **需要添加cmd /c的原因:**

在传入 echo echo\_test &gt; echo.txt 命令字符串时，出现错误\("java.io.IOException: Cannot run program "echo": CreateProcess error=2, 系统找不到指定的文件。"\)。原因是echo为命令行解释器cmd.exe的内置命令，并不是一个单独可执行的程序\(如下图\)，所以如果想执行echo命令写文件需要先启动cmd.exe，然后将echo命令做为cmd.exe的参数进行执行。

![](../../.gitbook/assets/image%20%28680%29.png)

另外关于cmd下的 /c 参数，当未指定时,运行如下示例程序,系统会启动一个pid为8984的cmd后台进程，由于cmd进程未终止导致java程序卡死。当指定/c时，cmd进程会在命令执行完毕后成功终止。

![](../../.gitbook/assets/image%20%28711%29.png)

![](../../.gitbook/assets/image%20%28720%29.png)

![](../../.gitbook/assets/image%20%28683%29.png)

 所以在Windows环境下，使用Runtime.getRuntime\(\)执行的命令前缀需要加上cmd /c，使得底层Windows的processthreadsapi.h\#CreateProcessW\(\)方法在创建新进程时，可以正确识别cmd且成功返回命令执行结果。

