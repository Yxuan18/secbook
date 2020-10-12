# 字节码和虚拟机

Python会将代码先编译成字节码，然后在虚拟机中动态得依次解释执行字节码。编译好的字节码存储在硬盘中以`.pyc`、`.pyd`等为扩展名。而在运行态，这些字节码会作为Python的一种对象`PyCodeObject`存在。`PyCodeObject`可以理解为C语言中的文本段，用于存储编译后的字节码、调试信息、常量值、变量名等。

本文不会讲述代码如何一步步编译成`PyCodeObject`，只会简单介绍`PyCodeObject`中各个域的含义，而把重点放在介绍Python的虚拟机和执行流。

### Python中的伪码PyCodeObject <a id="Python&#x4E2D;&#x7684;&#x4F2A;&#x7801;PyCodeObject"></a>

`PyCodeObject`保存代编译后的静态信息，在运行时再结合上下文形成一个完整的运行态环境。让我们看看静态编译后的信息都有哪些。

```text
typedef struct {
    PyObject_HEAD
    int co_argcount;    // co_argcount 参数，不包括不定参数
    int co_nlocals;		// co_nlocals 变量个数，co_argcount + 
                        // 可变参数个数 + co_kwonlyargcount(py3.0) + 局部变量个数
    int co_stacksize;   // 栈的大小 (编译后需要的最大栈深度) 
    int co_flags;		// PyCodeObject的一些标志位，用来优化运行时的性能
    PyObject *co_code;		// 编译后的字节码字符串
    PyObject *co_consts;	// 常量的列表
    PyObject *co_names;		// 常量中的字符串对象
    PyObject *co_varnames;	// 变量名字的元组
    PyObject *co_freevars;	// 自由变量的元组
    PyObject *co_cellvars;      // cell变量的元组
    /* The rest doesn't count for hash/cmp */
    PyObject *co_filename;	// 文件名
    PyObject *co_name;		// 对象的名字，例如函数的名字、类的名字等
    int co_firstlineno;		// 对应的代码在源码文件中的起始行号
    PyObject *co_lnotab;	// 伪码与行号的映射
    void *co_zombieframe;     // 对于一些特殊情况下的优化
    PyObject *co_weakreflist;   // 支持弱引用
} PyCodeObject;
```

其中有些域需要特别解释。

* co\_flags 用来保存一些编译信息，主要用于优化工作。例如co\_VARARGS\(0x0004\)表示有可变参数等，具体见code.h文件。
* co\_freevars 自由变量是一些在作用域内使用，但是没有在本作用域定义的变量。
* co\_cellvars 当前作用域定义，而在闭包等内部使用的变量。
* co\_lnotab 字节码的偏移值与对应的源码的行号的相对值。

```text
字节码在co_code中的偏移值   真实行号 行号的偏移值
0                         1       0 
6                         2       1
50                        7       5
```

那么实际上`co_lnotab`记录的是\(0, 0\), \(6, 1\), \(44, 5\)，当然实际记录中没有括号。具体`偏移值`和真实行号的对应关系可以通过下面的算法计算出来。

```text
// codeobject.c
int
PyCode_Addr2Line(PyCodeObject *co, int addrq)
{
    int size = PyString_Size(co->co_lnotab) / 2;
    unsigned char *p = (unsigned char*)PyString_AsString(co->co_lnotab);
    int line = co->co_firstlineno;
    int addr = 0;
    while (--size >= 0) {
        addr += *p++;
        if (addr > addrq)
            break;
        line += *p++;
    }
    return line;
}
```

* co\_code 记录编译后的字节码，以字符串的形式保存，而实际上就是数字。后面我们通过一个例子详细描述。

### PyCodeObject的示例 <a id="PyCodeObject&#x7684;&#x793A;&#x4F8B;"></a>

先给定一个Python代码示例，然后打印出其中的各个域。

```text
from __future__ import print_function
import dis
def out(a, b=1, *args, **kwargs):
    c = 2
    def inner(d, e=3, *iargs, **ikwargs):
        f = 4
        g = c
    print('inner-->co_argcount        :', inner.__code__.co_argcount)
#    print('inner-->co_kwonlyargcount  :', inner.__code__.co_kwonlyargcount)
    print('inner-->co_nlocals         :', inner.__code__.co_nlocals)
    print('inner-->co_stacksize       :', inner.__code__.co_stacksize)
    print('inner-->co_flags           :', inner.__code__.co_flags)
    print('inner-->co_code            :', inner.__code__.co_code)
    print('inner-->co_consts          :', inner.__code__.co_consts)
    print('inner-->co_names           :', inner.__code__.co_names)
    print('inner-->co_varnames        :', inner.__code__.co_varnames)
    print('inner-->co_freevars        :', inner.__code__.co_freevars)
    print('inner-->co_cellvars        :', inner.__code__.co_cellvars)
    print('inner-->co_filename        :', inner.__code__.co_filename)
    print('inner-->co_name            :', inner.__code__.co_name)
    print('inner-->co_firstlineno     :', inner.__code__.co_firstlineno)
    print('inner-->co_lnotab          :', inner.__code__.co_lnotab)
    
print('out-->co_argcount        :', out.__code__.co_argcount)
#print('out-->co_kwonlyargcount  :', out.__code__.co_kwonlyargcount)
print('out-->co_nlocals         :', out.__code__.co_nlocals)
print('out-->co_stacksize       :', out.__code__.co_stacksize)
print('out-->co_flags           :', out.__code__.co_flags)
print('out-->co_code            :', out.__code__.co_code)
print('out-->co_consts          :', out.__code__.co_consts)
print('out-->co_names           :', out.__code__.co_names)
print('out-->co_varnames        :', out.__code__.co_varnames)
print('out-->co_freevars        :', out.__code__.co_freevars)
print('out-->co_cellvars        :', out.__code__.co_cellvars)
print('out-->co_filename        :', out.__code__.co_filename)
print('out-->co_name            :', out.__code__.co_name)
print('out-->co_firstlineno     :', out.__code__.co_firstlineno)
print('out-->co_lnotab          :', out.__code__.co_lnotab)
print('=========================================================')
out(1, 2, 3, 4, 5, 6, 7, e = 8, f = 9)
print()
print('disamble:')
print(dis.dis(out))
```

需要先解释一下`co_kwonlyargcount`，这个域在`PY3`才有，用于支持在不定参数后定义的位置参数，例如`def func(*args, kwonly=None)`。

这个实例的输出可以看到对应的各个域的详细内容。

```text
out-->co_argcount        : 2     # a, b
out-->co_nlocals         : 5     # a, b, c, d, e
out-->co_stacksize       : 3     
out-->co_flags           : 65551 # b'0b10000000000001111'  CO_FUTURE_PRINT_FUNCTION|CO_VARKEYWORDS|CO_VARARGS|CO_NEWLOCALS|CO_OPTIMIZED 
out-->co_code            : ddfd}t...  # 部分省略，后续分析
out-->co_consts          : (None, 2, 3, <code object inner>, 'inner-->co_argcount        :', # 省略其他'inner-->')  # 常量值，这里添加了默认返回值None
out-->co_names           : ('print', '__code__', 'co_argcount', 'co_nlocals', 'co_stacksize', 'co_flags', 'co_code', 'co_consts', 'co_names', 'co_varnames', 'co_freevars','co_cellvars', 'co_filename', 'co_name', 'co_firstlineno', 'co_lnotab')  # 常量名
out-->co_varnames        : ('a', 'b', 'args', 'kwargs', 'inner')   # 变量名字，包括参数变量和内部变量
out-->co_freevars        : ()                                      # 无
out-->co_cellvars        : ('c',)                                  # 用于给子作用域使用的变量
out-->co_filename        : pycode.py          
out-->co_name            : out
out-->co_firstlineno     : 3                                       # 起始行号
out-->co_lnotab          :                                         # 省略
=========================================================
inner-->co_argcount        : 2                                     # d, e
inner-->co_nlocals         : 6                                     # d, e, iargs, ikwargs, f, g
inner-->co_stacksize       : 1                                     # 
inner-->co_flags           : 65567                                 # '0b10000000000011111' CO_FUTURE_PRINT_FUNCTION|CO_NESTED |CO_VARKEYWORDS|CO_VARARGS|CO_NEWLOCALS|CO_OPTIMIZED
inner-->co_code            : d}}dS                                 # 省略
inner-->co_consts          : (None, 4)                             # 常量
inner-->co_names           : ()                                    #
inner-->co_varnames        : ('d', 'e', 'iargs', 'ikwargs', 'f', 'g')   # 变量名字
inner-->co_freevars        : ('c',)                                     # 自由变量，引用的父作用域的变量
inner-->co_cellvars        : ()                                         # 无
inner-->co_filename        : pycode.py
inner-->co_name            : inner
inner-->co_firstlineno     : 6                                          # 起始行号
inner-->co_lnotab          :                                            # 省略
```

从这个例子中可以清楚了解常量、变量、自由变量以及cell变量的含义。接下来我们看下`co_code`的含义，使用linux的`xdd`工具将其转换成十六进制，并且使用`dis`模块反编译其字节码。

```text
import dis
def out(a, b=1, *args, **kwargs):
    c = 2
    def inner(d, e=3, *iargs, **ikwargs):
        f = 4
        g = c
print out.__code__.co_code
dis.dis(out)
```

```text
# co_code的十六进制内容
0000000: 6401 0089 0000 6402 0087 0000 6601 0064  d.....d.....f..d
0000010: 0300 8601 007d 0400 6400 0053 0a         .....}..d..S.
```

```text
# 字节码的反编译
  4           0 LOAD_CONST               1 (2)
              3 STORE_DEREF              0 (c)
  6           6 LOAD_CONST               2 (3)
              9 LOAD_CLOSURE             0 (c)
             12 BUILD_TUPLE              1
             15 LOAD_CONST               3 (<code object inner at 00000000039E69B0, file "<ipython-input-2-656e8bface8a>", line 6>)
             18 MAKE_CLOSURE             1
             21 STORE_FAST               4 (inner)
             24 LOAD_CONST               0 (None)
             27 RETURN_VALUE
```

* 十六进制的第一个为`64`值`100`，查阅`opcode.h`可以看到起对应的字节码`#define LOAD_CONST 100`，与反编译中的命令`LOAD_CONST`相符。
* 十六进制的第二个为`01`值`01`，对应的是字节码`LOAD_CONST`的参数`1`。
* 十六进制的第三个为`00`值`00`，此值表示`STOP_CDOE`，一个完整字节码的结束标志。

同理可以解析接下来的字节码和对应的操作的含义。至此，我们明白字节码的格式为

```text
字节码指令编号(64) 多个参数值(1) 结束标志(00)
```

到现在为止我们明白了字节码的数据结构、各域值的含义，`co_code`字节码的格式以及如何与操作命令对应。下面我们看看这些字节码如何运行。

### PyFrameObject <a id="PyFrameObject"></a>

Python模拟了C语言中的运行栈作为运行时的环境，每个栈用`PyFrameObject`结构表示。

```text
typedef struct _frame {
    PyObject_VAR_HEAD
    struct _frame *f_back;     // 前一个运行栈，调用方
    PyCodeObject *f_code;      // 执行的PyCodeObject对象
    PyObject *f_builtins;      // builtins环境变量集合
    PyObject *f_globals;       // globals全局变量集合
    PyObject *f_locals;        // locals本地变量集合
    PyObject **f_valuestack;   // 栈起始地址，最后一个本地变量之后
    PyObject **f_stacktop;     // 栈针位置，指向栈中下一个空闲位置
    PyObject *f_trace;         // trace函数
    PyObject *f_exc_type, *f_exc_value, *f_exc_traceback;  // 记录异常处理
    PyThreadState *f_tstate;   // 当前的线程
    int f_lasti;		       // 当前执行的字节码的地址
    int f_lineno;		       // 当前的行号
    int f_iblock;		       // 一些局部block块
    PyTryBlock f_blockstack[CO_MAXBLOCKS]; /* for try and loop blocks */
    PyObject *f_localsplus[1];	// 栈地址，大小为 本地变量+co_stacksize
} PyFrameObject;
```

对应的结构

![](../../../.gitbook/assets/image%20%28540%29.png)

当执行函数调用时会进入新的栈帧，那么当前栈帧就作为下一个栈帧的`f_back`字段

![](../../../.gitbook/assets/image%20%28539%29.png)

多个栈帧链属于一个线程，而同时可能存在多个线程，每个线程拥有一个栈帧链。这样形成了Python的虚拟机运行环境。

![image](https://fanchao01.github.io/blog/images/python_runtime_env.png)

### Python执行字节码 <a id="Python&#x6267;&#x884C;&#x5B57;&#x8282;&#x7801;"></a>

字节码的执行就像上图所示，由一个大的循环和选择语句构成，逻辑骨干比较简单。

```text
for(;;;) {
    switch(opcode) {
    
    case 100:   # LOAD_CONST
    {
      x = POP()
      ...    // 执行的具体操作
      break;
    };
    
    case 101:   # LOAD_NAME
    {
      ...
      break;
    }
    ...
};
```

接下来，我们通过反编译代码追踪其如何一步步执行。

```text
# 字节码的反编译
  4           0 LOAD_CONST               1 (2)
              3 STORE_DEREF              0 (c)
  6           6 LOAD_CONST               2 (3)
              9 LOAD_CLOSURE             0 (c)
             12 BUILD_TUPLE              1
             15 LOAD_CONST               3 (<code object inner at 00000000039E69B0, ...>)
             18 MAKE_CLOSURE             1
             21 STORE_FAST               4 (inner)
             24 LOAD_CONST               0 (None)
             27 RETURN_VALUE
```

通过追踪每个指令码的执行过程以及对应的`PyFrameObject`的栈帧变化，可以一步步看到虚拟机的执行过程。

```text
PyObject *
PyEval_EvalFrame(PyFrameObject *f) {
    co = f->f_code;
    names = co->co_names;
    consts = co->co_consts;
    fastlocals = f->f_localsplus;
    // freevars在内存中对应的不是f->f_freevars，而是f->f_cellvars
    freevars = f->f_localsplus + co->co_nlocals;
    first_instr = (unsigned char*) PyString_AS_STRING(co->co_code);
    // f->f_lasti默认值为-1
    next_instr = first_instr + f->f_lasti + 1;
    // 执行栈顶
    stack_pointer = f->f_stacktop;
    for (;;) {
    
        fast_next_opcode:
            f->f_lasti = INSTR_OFFSET();
        
            opcode = NEXTOP();    // 获取字节码
            oparg = 0;   
            if (HAS_ARG(opcode))  // 如果字节码有参数，获取参数
                oparg = NEXTARG();
            TARGET(LOAD_CONST)    // 0, 6, 24 行反编译指令LOAD_CONST
        {
            x = GETITEM(consts, oparg);   // 从const中获取值压栈
            Py_INCREF(x);
            PUSH(x);                      
            FAST_DISPATCH();              // goto fast_next_opcode
        }
        
        ...
        
        
        TARGET(STORE_DEREF)        // 3
        {
            w = POP();                   // 从栈中取值，设置为CellObejct的值
            x = freevars[oparg];         
            PyCell_Set(x, w);            
            Py_DECREF(w);
            DISPATCH();
        }
```

初始化以及分别执行`0`和`3`字节码的`PyFrameObject`结构变化。

* LOAD\_CONST 将co\_consts中对应的值压栈
* STORE\_DEREF 解引用，设置栈中的变量值

![image](https://fanchao01.github.io/blog/images/python_frame_run_1.jpg)

```text
TARGET(LOAD_CLOSURE)       // 9
{
    x = freevars[oparg];     
    Py_INCREF(x);
    PUSH(x);
    if (x != NULL) DISPATCH();
    break;
}
TARGET(BUILD_TUPLE)       // 12
{
    x = PyTuple_New(oparg);      // 创建一个元组，并且将栈中的元素设置为元组的元素
    if (x != NULL) {
        for (; --oparg >= 0;) {
            w = POP();
            PyTuple_SET_ITEM(x, oparg, w);
        }
        PUSH(x);
        DISPATCH();
    }
    break;
}
TARGET(MAKE_CLOSURE)     // 18
{
    v = POP(); /* code object */
    x = PyFunction_New(v, f->f_globals);    // 创建函数
    Py_DECREF(v);
    if (x != NULL) {
        v = POP();
        if (PyFunction_SetClosure(x, v) != 0) {
            /* Can't happen unless bytecode is corrupt. */
            why = WHY_EXCEPTION;
        }
        Py_DECREF(v);
    }
    if (x != NULL && oparg > 0) {
        v = PyTuple_New(oparg);
        if (v == NULL) {
            Py_DECREF(x);
            x = NULL;
            break;
        }
        while (--oparg >= 0) {
            w = POP();
            PyTuple_SET_ITEM(v, oparg, w);
        }
        if (PyFunction_SetDefaults(x, v) != 0) {
            /* Can't happen unless
               PyFunction_SetDefaults changes. */
            why = WHY_EXCEPTION;
        }
        Py_DECREF(v);
    }
    PUSH(x);
    break;
}
```

* LOAD\_CLOSURE 将freevars中的对象压栈
* BUILD\_TUPLE 用栈帧中的元素创建元组，并压栈
* BUILD\_CLOSURE 创建PyFunction对象，并设置其中的`f_closure`域

![image](https://fanchao01.github.io/blog/images/python_frame_run_2.jpg)

```text
        
        TARGET(STORE_FAST)     // 21
        {
            v = POP();             // 设置locals值
            SETLOCAL(oparg, v);
            FAST_DISPATCH();
        }
        
        TARGET_NOARG(RETURN_VALUE)  // 27
        {
            retval = POP();
            why = WHY_RETURN;
            goto fast_block_end;
        }
}
```

![](../../../.gitbook/assets/image%20%28541%29.png)

* STORE\_FAST 将栈中的一个元素设置到对应的本地变量域中
* RETURN\_VALUE return，并且设置退出原因`WHY_RETURN`

从上面的代码和过程图，整个代码的执行过程清楚的显现出来：）

