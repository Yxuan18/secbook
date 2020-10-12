# 信号处理机制

## Python信号处理机制

本篇的信号处理机制不是指Python的`signal`模块的使用，而是指Python解释器本身如何处理信号以及如何实现`signal`模块。Python解释器处理信号机制需要做好两件事情：

1. Python解释器与操作系统有关信号的交互
2. Python解释器实现信号语义的API接口和模块

![img](https://fanchao01.github.io/blog/images/python_signal.jpg)

大体上，Python解释器对信号的实现总体思路比较简单。Python解释器对信号做一层封装，在这层封装中处理信号，以及信号发生时的回调函数，使之能够纳入整个Python虚拟机的运行中。我们先从信号的初始化开始一点点揭露整个运作机制。

## 信号机制的初始化

信号机制的初始化是在Python初始化整个解释器时开始的，Python在初始化函数中调用`initsigs`来进行整个系统以及`singal`模块的初始化。

```text
// python/pythonrun.c  
void
Py_InitializedEx(int install_sigs)
{
        ....
    
        if (install_sigs)   // 初始化时install_sigs==1
        initsigs(); /* Signal handling stuff, including initintr() */
        
        ....
}
// 代码中PyOS_xxx系列都是Python解释器直接对系统调用的封装
static void
initsigs(void)
{
#ifdef SIGPIPE
    PyOS_setsig(SIGPIPE, SIG_IGN);     // 忽略SIGPIPE  
#endif
#ifdef SIGXFZ
    PyOS_setsig(SIGXFZ, SIG_IGN);      // 忽略SIGXFZ 
#endif
#ifdef SIGXFSZ
    PyOS_setsig(SIGXFSZ, SIG_IGN);     // 忽略SIGXFSZ  file size exceeded
#endif
    PyOS_InitInterrupts(); /* May imply initsignal() */
}
// python/modules/singalmodle.c
void
PyOS_InitInterrupts(void)
{
    initsignal();
    _PyImport_FixupExtension("signal", "signal");
}
```

 直接进入到`singalmodule.c`中看`signal`模块以及信号的初始化  。       

```text
PyMODINIT_FUNC
initsignal(void)
{
    PyObject *m, *d, *x;
    int i;
#ifdef WITH_THREAD
    main_thread = PyThread_get_thread_ident();
    main_pid = getpid();
#endif
    /* Create the module and add the functions */
    // 初始化signal模块
    m = Py_InitModule3("signal", signal_methods, module_doc);
    if (m == NULL)
        return;
    /* Add some symbolic constants to the module */
    d = PyModule_GetDict(m);
    // 将SIG_DFL、SIGIGN 转化成Python整数对象
    x = DefaultHandler = PyLong_FromVoidPtr((void *)SIG_DFL);
    if (!x || PyDict_SetItemString(d, "SIG_DFL", x) < 0)
        goto finally;
    x = IgnoreHandler = PyLong_FromVoidPtr((void *)SIG_IGN);
    if (!x || PyDict_SetItemString(d, "SIG_IGN", x) < 0)
        goto finally;
    x = PyInt_FromLong((long)NSIG);
    if (!x || PyDict_SetItemString(d, "NSIG", x) < 0)
        goto finally;
    Py_DECREF(x);
    /*
    * 获取signal模块中的默认中断处理函数，
    * 实际就是 signal_default_int_handler    
    */
    x = IntHandler = PyDict_GetItemString(d, "default_int_handler");
    if (!x)
        goto finally;
    Py_INCREF(IntHandler);
    /*
    * 初始化Python解释器中的Handler，
    * 这个数组存储每个用户自定义的信号处理函数
    * 以及标志是否发生该信号的标志。
    */
    Handlers[0].tripped = 0;
    for (i = 1; i < NSIG; i++) {
        void (*t)(int);
        t = PyOS_getsig(i);
        Handlers[i].tripped = 0;
        if (t == SIG_DFL)
            Handlers[i].func = DefaultHandler;
        else if (t == SIG_IGN)
            Handlers[i].func = IgnoreHandler;
        else
            Handlers[i].func = Py_None; /* None of our business */
        Py_INCREF(Handlers[i].func);
    }
    
    /* 
    * 为 SIGINT 设置Python解释器的信号处理函数signal_handler
    * signal_handler 也会成为Python解释器与用户自定义处理函数的桥梁
    */
    if (Handlers[SIGINT].func == DefaultHandler) {
        /* Install default int handler */
        Py_INCREF(IntHandler);
        Py_DECREF(Handlers[SIGINT].func);
        Handlers[SIGINT].func = IntHandler;
        old_siginthandler = PyOS_setsig(SIGINT, signal_handler);
    }
// 实现signal模块中的各个 SIGXXX 信号值和名称
#ifdef SIGHUP                  
    x = PyInt_FromLong(SIGHUP);
    PyDict_SetItemString(d, "SIGHUP", x);
    Py_XDECREF(x);
#endif
    ....
    if (!PyErr_Occurred())
        return;
    /* Check for errors */
  finally:
    return;
}
```

 可以看到Python将用户自定义信号处理函数保存在`Handler`数组中，而实际上向系统注册`signal_handler`函数。这个`signal_handler`函数成为信号发生时沟通Python解释器和用户自定义信号处理函数的桥梁。可以从`signal.signal`的实现中清楚的看到这一点。

```text
// python/signalmodule.c
static PyObject *
signal_signal(PyObject *self, PyObject *args)
{
    PyObject *obj;
    int sig_num;
    PyObject *old_handler;
    void (*func)(int);
    if (!PyArg_ParseTuple(args, "iO:signal", &sig_num, &obj))
        return NULL;
#ifdef MS_WINDOWS
    /* Validate that sig_num is one of the allowable signals */
    switch (sig_num) {
        case SIGABRT: break;
#ifdef SIGBREAK
        /* Issue #10003: SIGBREAK is not documented as permitted, but works
           and corresponds to CTRL_BREAK_EVENT. */
        case SIGBREAK: break;
#endif
        case SIGFPE: break;
        case SIGILL: break;
        case SIGINT: break;
        case SIGSEGV: break;
        case SIGTERM: break;
        default:
            PyErr_SetString(PyExc_ValueError, "invalid signal value");
            return NULL;
    }
#endif
#ifdef WITH_THREAD
    // 只有主函数才能设置信号处理函数
    if (PyThread_get_thread_ident() != main_thread) {
        PyErr_SetString(PyExc_ValueError,
                        "signal only works in main thread");
        return NULL;
    }
#endif
    if (sig_num < 1 || sig_num >= NSIG) {
        PyErr_SetString(PyExc_ValueError,
                        "signal number out of range");
        return NULL;
    }
    if (obj == IgnoreHandler)
        func = SIG_IGN;
    else if (obj == DefaultHandler)
        func = SIG_DFL;
    else if (!PyCallable_Check(obj)) {
        PyErr_SetString(PyExc_TypeError,
"signal handler must be signal.SIG_IGN, signal.SIG_DFL, or a callable object");
                return NULL;
    }
    else
        // 除了signal.SIG_IGN和signal.SIG_DFL之外
        // Python解释器向系统注册的都是signal_handler函数
        func = signal_handler;
    if (PyOS_setsig(sig_num, func) == SIG_ERR) {
        PyErr_SetFromErrno(PyExc_RuntimeError);
        return NULL;
    }
    // 把实际的用户自定义信号处理函数，放入对应的Handler数组中
    // tripped标记对应信号值的信号是否发生了
    old_handler = Handlers[sig_num].func;
    Handlers[sig_num].tripped = 0;
    Py_INCREF(obj);
    Handlers[sig_num].func = obj;
    if (old_handler != NULL)
        return old_handler;
    else
        Py_RETURN_NONE;
}
```

## 信号产生时Python的动作

当信号产生时，操作系统会调用Python解释器注册的信号处理函数，即上文中的`signal_handler`函数。这个函数将对应的`Handler`结构中的信号产生标志`tripped`设置为1，然后将一个统一信号处理函数`trip_signal`作为`pending_call`注册到Python虚拟机的执行栈中。于是，Python在虚拟机执行过程中调用`pending_call`并执行各个用户自定义的信号处理函数。

```text
static void
signal_handler(int sig_num)
{
    int save_errno = errno;
#if defined(WITH_THREAD) && defined(WITH_PTH)
    if (PyThread_get_thread_ident() != main_thread) {
        pth_raise(*(pth_t *) main_thread, sig_num);
    }
    else
#endif
    {
#ifdef WITH_THREAD
    /* See NOTES section above */
    if (getpid() == main_pid)
#endif
    {
        trip_signal(sig_num);
    }
#ifndef HAVE_SIGACTION
#ifdef SIGCHLD
    /* To avoid infinite recursion, this signal remains
       reset until explicit re-instated.
       Don't clear the 'func' field as it is our pointer
       to the Python handler... */
    if (sig_num != SIGCHLD)
#endif
    /* If the handler was not set up with sigaction, reinstall it.  See
     * Python/pythonrun.c for the implementation of PyOS_setsig which
     * makes this true.  See also issue8354. */
    // 重新设置信号处理
    PyOS_setsig(sig_num, signal_handler);
#endif
    }
    /* Issue #10311: asynchronously executing signal handlers should not
       mutate errno under the feet of unsuspecting C code. */
    errno = save_errno;
}
static void
trip_signal(int sig_num)
{
    // 信号产生了
    Handlers[sig_num].tripped = 1;
    
    // 如果正在处理信号，则不再向Python虚拟机提交
    if (is_tripped)
        return;
    /* Set is_tripped after setting .tripped, as it gets
       cleared in PyErr_CheckSignals() before .tripped. */
    is_tripped = 1;
    // 向Python虚拟机提交pending_call，纳入到整个虚拟机的执行过程中
    Py_AddPendingCall(checksignals_witharg, NULL);
    if (wakeup_fd != -1)
        write(wakeup_fd, "\0", 1);
}
static int
checksignals_witharg(void * arg)
{
    return PyErr_CheckSignals();
}
int
PyErr_CheckSignals(void)
{
    int i;
    PyObject *f;
    // 已经在信号处理中
    if (!is_tripped)
        return 0;
    // 只主线程中处理信号
#ifdef WITH_THREAD
    if (PyThread_get_thread_ident() != main_thread)
        return 0;
#endif
    /*
     * The is_tripped variable is meant to speed up the calls to
     * PyErr_CheckSignals (both directly or via pending calls) when no
     * signal has arrived. This variable is set to 1 when a signal arrives
     * and it is set to 0 here, when we know some signals arrived. This way
     * we can run the registered handlers with no signals blocked.
     *
     * NOTE: with this approach we can have a situation where is_tripped is
     *       1 but we have no more signals to handle (Handlers[i].tripped
     *       is 0 for every signal i). This won't do us any harm (except
     *       we're gonna spent some cycles for nothing). This happens when
     *       we receive a signal i after we zero is_tripped and before we
     *       check Handlers[i].tripped.
     */
    /*
    * 恢复该信号。对于信号处理可能有两种情况：
    * 1. 在is_tripped = 0之前： 信号又发生了，则只在Handler中设置标志位，
    *    不会再次提交到pendingcall，多个信号只处理一次;
    * 2. 在is_tripped = 0之后： 信号又发生了，则会被再次提交到pendingcall
    *    每发生一次信号调用一次信号处理函数。
    */
    is_tripped = 0;
    if (!(f = (PyObject *)PyEval_GetFrame()))
        f = Py_None;
    // 按照信号值从小到大依次调用对应的信号处理函数
    for (i = 1; i < NSIG; i++) {
        if (Handlers[i].tripped) {
            PyObject *result = NULL;
            PyObject *arglist = Py_BuildValue("(iO)", i, f);
            Handlers[i].tripped = 0;
            if (arglist) {
                result = PyEval_CallObject(Handlers[i].func,
                                           arglist);
                Py_DECREF(arglist);
            }
            if (!result)
                return -1;
            Py_DECREF(result);
        }
    }
    return 0;
}
```

这里面的`PyErr_CheckSignals`函数也会被其他模块调用直接信号的处理。例如，在`file.read`读取文件过程中中断，Python对调用该函数进行信号处理。至此，可以看到整个信号处理的流程：

1. 初始化signal模块，将对应的操作系统信号值、函数转化成Python对象
2. 用户设置信号就向操作系统注册函数`signal_handler`，并将用户自定义信号处理函数设置到对应的`Handler`数组中
3. 当信号发生时，操作系统调用`signal_handler`设置`tripped=1`，然后调用`trip_signal`将统一处理函数`checksignals_witharg`作为`pendingcall`注册到Python虚拟机的执行栈中。
4. Python虚拟机在处理`pendingcall`时调用`checksignals_withargs`，从而信号处理函数得以执行。
5. 另外，Python其他模块可以直接调用`PyErr_CheckSignals`进行信号处理。

## Python信号的语义

通过注释以及代码剖析可以归纳Python的信号语义：

* 只有主线程能够设置、捕获和处理信号
* 信号设置一直有效（`signal_handler`中会再次注册信号处理函数\)
* 多次信号，可能会被合并处理一次
* 按照信号值从小到大处理

## 信号实例

### 主线程才能捕获信号

```text
import threading                                                                 
import signal                                                                    
import time                                                                      
                                                                                 
SIG = []                                                                         
                                                                                 
def sig_handler(*args):                                                          
    SIG.append(args)                                                             
                                                                                 
signal.signal(signal.SIGUSR1, sig_handler)                                       
signal.signal(signal.SIGUSR2, sig_handler)                                       
signal.signal(signal.SIGSYS, sig_handler)                                        
                                                                                 
class MyThread(threading.Thread):                                                
    def run(self, *args):                                                        
        start = time.time()                                                      
        while True:                                                              
            if time.time() > start + 10:                                         
                break                                                            
        print 'In Thread:', SIG                                                          
                                                                                 
t = MyThread()                                                                   
t.start()                                                                        
print 'start thread:', t                                                         
t.join()                                                                         
print 'In Main:', SIG
```

```text
root@ubuntu:/home/python# python test_signal_main_thread.py
start thread: <MyThread(Thread-1, started 140229890316032)>
In Main: []
In Thread: []          # [1]
```

* \[1\] Python中的线程都是分离的，因此主线程很快退出。信号不能发送到主线程，因此不能被执行。

### 信号可能只被处理一次

```text
import threading
import signal
import time
SIG = []
def sig_handler(*args):
    SIG.append(args)
signal.signal(signal.SIGUSR1, sig_handler)
signal.signal(signal.SIGUSR2, sig_handler)
signal.signal(signal.SIGSYS, sig_handler)
class MyThread(threading.Thread):
    def run(self, *args):
        start = time.time()
        while True:
            if time.time() > start + 10:
                break
        print 'In Thread:', SIG
t = MyThread()
t.start()
print 'start thread:', t
t.join()
print 'In Main:', SIG
```

```text
# python test_signal_signo.py       # kill -10  2856; kill -10 2856; kill -12 2856
start thread: <MyThread(Thread-1, started 140351081834240)>
In Thread: []       # [1]
In Main: [(10, <frame object at 0x16e2d50>), (12, <frame object at 0x16e2d50>)] # [2]
```

* \[1\] 主线程的`t.join`一直阻塞，因此在子线程没有退出前不能处理信号。（C语言的信号处理是可以打断堵塞信号的）
* \[2\] 信号在有机会处理之前发生了两次信号，但是只处理了一次。

## Python信号的特殊性

Python的信号语义与Linux的C语言的信号语义有一些不同。

* Python信号的处理函数会一直有效；而Linux除非特殊设置否则信号处理函数默认只调用一次就被恢复
* Python信号只能在主线程中设置、捕获和处理
* Python信号不能打断堵塞操作\(因为信号发生时子线程在运行\)

