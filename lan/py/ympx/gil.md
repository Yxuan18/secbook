# 多线程和GIL锁

## 写在前面

最近工作中又遇到了GIL和多线程的问题，借此机会重新梳理一下。大致脉络仿照陈儒的《Python源码剖析》第15章`Python多线程机制`，但会将平台转移到Linux下的Pthread线程库上。主要介绍CPython解释器的C实现，具体的Python库\(thread,threading\)的介绍可以参考[Python Threading Module](https://fanchao01.github.io/2014/05/15/python_threading/)。同时会介绍下相关函数在Linux平台中的表现等。

## python中的GIL锁

CPython中有一个全局的解释器锁叫做`GIL`（global interpreter lock），对解释器中的共享变量提供保护。GIL不是必须的，可以通过对每个资源单独加锁的方式去掉GIL，也就是将GIL换成更细粒度的锁。实际上也有这么做的，不过结果是在单核上的性能不如有GIL的版本\(2倍的差距\)，大量的细粒度锁的开销消耗了大量的资源。所以，Guido有篇很著名的文章[It isn’t easy to remove GIL](http://www.artima.com/weblogs/viewpost.jsp?thread=214235)讨论这个问题。如果去掉GIL需要考虑两件事：

```text
1. 不会降低Python在单核上的性能；
2. 需要考虑如何将现在大量的库进行迁移；
```

总之从Python3的尴尬处境可以简单知道，CPython中的GIL是不可能去除的。

做技术很多时候是在折中\(tradeoff\)，就比如当年Linux用宏内核架构会被认为过时一样。性能高、简单（实现简单）好用（使用快速）几乎立于不败之地，更多参考见这本书[The Unix Hackers Handbook](http://web.mit.edu/~simsong/www/ugh.pdf)。

## GIL锁的类型以及语义

代码中的`interpreter_lock`就是全局解释器锁，类型为`PyThread_type_lock`，简单的void指针，然后再根据不同的平台转换成对应类型的指针，在Linux中的类型是`pthread_lock`指针。

```c
//python2.7.5/Python/ceval.c
static PyThread_type_lock interpreter_lock = 0; /* This is the GIL */
static PyThread_type_lock pending_lock = 0; /* for pending calls */
static long main_thread = 0;
int
PyEval_ThreadsInitialized(void)
{
    return interpreter_lock != 0;
}
void
PyEval_InitThreads(void)
{
    if (interpreter_lock)
        return;
    interpreter_lock = PyThread_allocate_lock();
    PyThread_acquire_lock(interpreter_lock, 1);
    main_thread = PyThread_get_thread_ident();
}
//python2.7.5/Python/Thread_pthread.h
typedef struct {
	/* 0=unlocked, 1=locked */
    char             locked; 
    /* a <cond, mutex> pair to handle an acquire of a locked lock */
    pthread_cond_t   lock_released;
    pthread_mutex_t  mut;
} pthread_lock;
```

 之所以需要`pthread_lock`而不是直接使用原生的`pthread_mutex_t`，是因为Pthread的标准有未定义的\(undefined\)部分，主要是：

```text
1. 一个已经获取锁的线程再次获取同一个锁时的表现；
(同一个线程多次调用pthread_mutex_lock，在linux中默认类型的锁第二次调用总会堵塞)
2.一个已经锁住的锁，被其他线程释放时的表现；
(其他线程调用pthread_mutex_unlock，在linux中默认类型的锁总会被释放）
```

 因此，Python实现了`pthread_lock`规避标准中的未定义部分。`pthread_lock`分为3个成员：

```text
1. char locked：是否锁住的标志，每次加锁需要竞争此标志，如果为1就是已经锁住，加锁的线程返回失败非0或者等待；
2. pthread_cond_t lock_released：锁的等待队列，请求锁时带有waitflag的线程会等待在该条件变量上；
3. pthread_mutex_t mut：锁本身；
```

从上面的结构可以看到一个共识，加锁与代码之间的关系是为了使不同进程/线程串行执行代码，串行执行的结果就是共享资源操作结果的一致性。保证并行执行的正确性有几种不同的方法：

```text
1. 使程序串行执行临界区：加锁、信号、条件
2. 资源本身保证原子性：原子操作、无锁队列
3. 去掉共享资源：函数式编程
```

 `pthread_lock`有1组4个函数\(接口\)调用，分别是：

```text
1. PyThread_allocate_lock: 分配一个锁；
2. PyThread_free_lock: free一个锁；
3. PyThread_acquire_lock(lock, waitflag):获取锁，waitflat=1时没有获取锁则等待；
(waitflag=0，实现了pthread_mutex_trylock的语义；waitflag=1实现了pthread_mutex_lock的语义）
4. PyThread_release_lock: 总是成功释放锁，并且唤醒至少1个在等待锁的线程；
(PyThread_release_lock实现中即使pthread_mutex_lock失败也会把locked恢复为0）
```

Python中只使用了默认类型的锁，`pthread_mutex_init`中的第二个参数为NULL，mut本身是一个多次请求会等待的锁，不过Python本身不使用等待的语义。`Phtread_lock`和一组操作函数创造了一个这样的线程锁：通过waitflag指定获取锁时是否等待，成功获取返回0，失败返回非0；释放锁总能成功并唤醒至少1个等待锁的线程。

最后补充一下`PyThread_allocate_lock`中会调用`PyThread_init_thread`，进而调用`PyThread__init_thread`进行线程初始化。这是因为有些平台上进程和线程是完全分离的概念，需要调用相应的函数启动多线程库。在Linux的Pthread平台下是一进程多线程的模型，默认情况下一个进程也是一个线程，因此在这种情况下`PyThread__init_thread`是空的函数，完全不需要启动线程库的动作。

## python中的线程

Python中通过thread等Python库启动的线程就是一个普通的Pthread线程，与C程序中调用`pthread_create`启动的线程没有本质区别，只不过Python中同一时间只有一个线程在运行，具体哪个线程能运行通过竞争GIL决定的。Python中线程的本质：

```text
1. 同一时间只有一个Python线程（确切的说是虚拟机）运行，只能使用1个CPU核心；
2. 不同线程的调度（哪个线程竞争到了GIL）完全由Python所在的操作系统决定；
```

## python启动

python中的线程启动通过`thread.start_new/start_new_thread`函数，然后调到CPython中的`thread_PyThread_start_new_thread`。这个函数主要处理用户调用时传入的参数，然后将启动函数和参数包装入bootstate结构，然后以`t_bootstrap`启动原生线程（linux下的pthread线程）。需要bootstate的主要原因有两点：

```text
1. python不能直接以用户设置的函数启动线程，需要做一些处理；
2. 原生的pthread线程pthread_create函数只能接受一个参数；
(在linux c编程中创建线程需要传入多个参数也需要将过个参数封装到单个结构中）
```

```text
static PyMethodDef thread_methods[] = {
    {"start_new_thread",        (PyCFunction)thread_PyThread_start_new_thread,
                            METH_VARARGS,
                            start_new_doc},
    {"start_new",               (PyCFunction)thread_PyThread_start_new_thread,
                            METH_VARARGS,
                            start_new_doc},
...
}
```

 从调用thread模块一直到`PyThread_start_new_thread`的线程都是主线程在运行；调用`pthread_create`然后进入`t_bootstrap`的线程是子线程。先看下主线程的调用路径，主路径通过`pthread_create`创建了子线程然后返回。注意这个时候主线程是拿者GIL锁的。

```text
//python2.7.6/python/Threadmodule.c
thread_PyThread_start_new_thread
	_PyThreadState_Preallock
		new_threadstate
		(设置threadstate，注意这个时候thread_id是主线程的id)
	PyEval_InitThreads
	（分配和获取GIL锁）
	PyThread_start_new_thread
		pthread_create
		pthread_detach
	PyInt_FromLong (返回）
```

 这里面需要注意一点，主线程创建子线程后就detach了，所以Python中的子线程都是分离的。然后看下子线程的调用路径，需要说明从`pthread_create`创建子线程开始运行到`t_bootstrap`中的`PyEval_AcquireThread`的这段代码是没有运行在Python的虚拟机中的，也就是说这段代码和GIL没有关系，在这期间主线程和子线程（在多核机器中）是可以同时运行的\(不排除争夺其它的锁而导致挂起\)。

```text
//python2.7.6/python/threadmodule.c
t_bootstrap
	PyThread_get_thread_ident
	（这个时候将ident设置为真正的子线程的ident）
	_PyThreadState_Init
		_PyGILState_NoteThreadState
			PyThread_set_key_value
				find_key
	（这段代码是将threadstate设置为每个线程的私有变量。主要做debug用。
	 线程私有变量是这样一种变量，每个线程中的变量名是一样的，但是具体的值和线程相关，而且相互之间透明。
	 在linux Pthread中由库直接提供支持。
	 Python在其他平台中自己也实现了一个，可以看成一个ident:value的字典，但是每个线程只能取到自己ident上的值）
	PyEval_AcquireThread
		PyThread_acquire_lock
		（等待获取GIL)
		PyThread_State_Swap
		（设置当前的全局变量 _PyThreadState_Current。每个Python线程在退出前必须调用这个函数换出自己，运行前调用换入自己）
	PyEval_CallObjectWithKeywords
		PyObject_call
			func->ob_type->tp_call
	PyThreadState_Clear
	PyThreadState_DeleteCurrent
		PyThread_delete_key_value
		PyEval_ReleaseLock
		（释放GIL锁）
	PyThread_exit_thread
		exit(0)
```

上面是子线程的调用路径。到这里还有两个问题没有解决，第一个是子线程如何进入虚拟机运行的（进入`PyEval_EvalFrame`\);第二个是主线程何时释放GIL以便子线程在`t_bootstrap`中获取到而运行。

### **第一个问题**

先说第一个问题，在子线程调用路径中最后会调用`tp_call`，假设用户的子线程函数是函数，类似下面这样：

```c
def myfunc(x):
	do_something_with(x)
tpid = thread.start_new_thread(myfunc, (1,))
```

 那么`tp_call`对应的是Python实现的`function_call`函数，如下所示：

```c
//python2.7.6/python/funcobject.c
PyTypeObject PyFunction_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "function",
 	...
    function_call,                             /* tp_call */
	...
}
```

 `function_call`也在同一个文件中定义，它的调用路径见下。从中可以看到，每个线程对应一个Frame对象，也就是一个Python虚拟机，而不是一个Python虚拟机对应多个Python线程。（不像CPU那样，每个CPU对应多个线程，每个线程通过保存上下文设置寄存器进行切换）。

```c
function_call
	PyEval_EvalCodeEx
	（获取func字节码中存储的全局、局部变量、参数、闭包等等）
		PyFrame_new
		（创建新的Frame对象，Python虚拟机）
		PyEval_EvalFrame
		（运行创建的虚拟机）
```

### **第二个问题**

第二个问题，主线程何时释放GIL锁。Python代码在虚拟机Frame中运行，其中有个变量`_Py_Ticker`。当`_Py_Ticker`小于0时，Python会释放GIL锁进行一次Python线程的调度。

```c
         if (--_Py_Ticker < 0) {
            if (*next_instr == SETUP_FINALLY) {
                /* Make the last opcode before
                   a try: finally: block uninterruptible. */
                goto fast_next_opcode;
            }
            _Py_Ticker = _Py_CheckInterval;
            tstate->tick_counter++;
#ifdef WITH_TSC
            ticked = 1;
#endif
            if (pendingcalls_to_do) {
                if (Py_MakePendingCalls() < 0) {
                    why = WHY_EXCEPTION;
                    goto on_error;
                }
                if (pendingcalls_to_do)
                    /* MakePendingCalls() didn't succeed.
                       Force early re-execution of this
                       "periodic" code, possibly after
                       a thread switch */
                    _Py_Ticker = 0;
            }
            if (interpreter_lock) {
                /* Give another thread a chance */
				//这里是一次GIL锁的释放和获取，子线程有机会获取GIL得以运行
                if (PyThreadState_Swap(NULL) != tstate)
                    Py_FatalError("ceval: tstate mix-up");
                PyThread_release_lock(interpreter_lock);
                /* Other threads may run now */
                PyThread_acquire_lock(interpreter_lock, 1);
                if (PyThreadState_Swap(tstate) != NULL)
                    Py_FatalError("ceval: orphan tstate");
```

需要说明几点:

```text
1. GIL锁的粒度是每个Python指令，在一个Python指令中的操作是原子操作；
2. PyEval_EvalFrame中有一些指令运行后会跳到fast_next_opcode，跳过了GIL调度的代码区，所以这些指令和紧接下来的一个指令都是原子操作；（例如 x = 1，依赖这种细微的具体实现编程是不可取的，只要记住一个Python指令是原子操作足已）；
3. 每次Pyhon线程调度的指令数不一定整好是_Py_CheckInterval（100）个，有些指令会跳过释放GIL的代码区；
4. Python线程最后由谁竞争到了GIL完全由操作系统决定，也就是具体哪个线程执行由操作系统决定，Python只管释放/获取一次GIL锁；
5. 在线程创建和销毁的代码区有一部分不运行在Frame中，这时Python中多个线程可能同时运行；
```

## **补充**

上面提到Pthread线程和Python线程，按照CPython实现来看，Pthread线程和Python是一一对应的。称为Python线程侧重于正在运行Python代码时的线程\(`PyEval_EvalFrame`部分）；称为Pthread线程侧重于CPython中线程的创建/销毁时的线程，等同起来看也没有任何问题。

