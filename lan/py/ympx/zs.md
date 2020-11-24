# 整数对象PyIntObject

### 整数对象的结构 <a id="&#x6574;&#x6570;&#x5BF9;&#x8C61;&#x7684;&#x7ED3;&#x6784;"></a>

整数对象是固定大小的Python对象，内部只有一个`ob_ival`保存实际的整数值。

```c
typedef struct {
    PyObject_HEAD
    long ob_ival;
} PyIntObject;
```

### 整数对象的缓存 <a id="&#x6574;&#x6570;&#x5BF9;&#x8C61;&#x7684;&#x7F13;&#x5B58;"></a>

为了最大限度的减少内存分配和垃圾回收，Python对整数对象设计了缓存。整数对象的缓存由两种类别构成：

1. 小整数对象: 在Python启动时创建，永远不会回收
2. 其他整数对象：创建时分配，回收时先缓存；在最高代的垃圾回收中整体回收

在Python启动时会创建一批默认值为\[5, 257\)的小整数对象，存储在`small_ints`中。这些整数对象的生命周期为Python的生命周期，不会被回收。Python只所以这样处理是因此解释器内部会频繁用到这些小整数，如果每次都分配-回收-再分配显然效率不高，不如创建后一直保留用空间换时间。

```c
#ifndef NSMALLPOSINTS
#define NSMALLPOSINTS           257
#endif
#ifndef NSMALLNEGINTS
#define NSMALLNEGINTS           5
#endif
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
/* References to small integers are saved in this array so that they
   can be shared.
   The integers that are saved are those in the range
   -NSMALLNEGINTS (inclusive) to NSMALLPOSINTS (not inclusive).
*/
static PyIntObject *small_ints[NSMALLNEGINTS + NSMALLPOSINTS];
```

 可以通过`id`命令查看小整数对象的特性。

```text
id(246)
id(256)
Out[55]: 31363504L
id(256)
Out[56]: 31363504L  # id不会改变
id(257)
Out[53]: 60259096L
id(257)
Out[54]: 60259024L # id会改变
```

通过上面的例子我们可以知道，其他整数对象使用的内存是不固定的，申请时分配释放时回收。当然，这个回收并不一定是返还给系统内存，整数对象系统本身会缓存一部分整数对象。下面通过整数对象系统的初始化揭露整数的缓存方案。

### 整数对象的初始化 <a id="&#x6574;&#x6570;&#x5BF9;&#x8C61;&#x7684;&#x521D;&#x59CB;&#x5316;"></a>

当Python初始化时会调用`_PyInt_Init`函数进行整数的初始化。

```c
int
_PyInt_Init(void)
{
    PyIntObject *v;
    int ival;
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
    for (ival = -NSMALLNEGINTS; ival < NSMALLPOSINTS; ival++) {
          if (!free_list && (free_list = fill_free_list()) == NULL)
                    return 0;
        /* PyObject_New is inlined */
        v = free_list;
        free_list = (PyIntObject *)Py_TYPE(v);
        PyObject_INIT(v, &PyInt_Type);
        v->ob_ival = ival;
        small_ints[ival + NSMALLNEGINTS] = v;
    }
#endif
    return 1;
}
```

缓存会用到数据结构`PyIntBlock`以及`block_list`和`free_list`链表。`PyInBlock`用来一次申请多个整数对象的内存，然后再一个个用作`PyIntObject`，并且通过域`next`链接到`block_list`链表上。`free_list`中是空闲的`PyIntObject`的链表。`fill_free_list`初始化后的内存结构如下。

![image](https://fanchao01.github.io/blog/images/pyint_freelist.jpg)

然后通过`_PyInt_init`初始化为小整数，并将其指针存储到`samll_ints`数组中加快查找。`_PyInt_init`初始化后的内存结构如下。

![image](https://fanchao01.github.io/blog/images/pyint_smallints.jpg)

我们可以看到整数对象通过`PyIntBlock`和`free_list`进行内存申请和缓存的。

### 整数对象的创建 <a id="&#x6574;&#x6570;&#x5BF9;&#x8C61;&#x7684;&#x521B;&#x5EFA;"></a>

当新创建一个整数对象时，先从`free_list`中查找空闲的整数对象，如果有则直接使用；否则会重新分配`PyIntBlock`结构并进行初始化。

```c
PyObject *
PyInt_FromLong(long ival)
{
    register PyIntObject *v;
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
    if (-NSMALLNEGINTS <= ival && ival < NSMALLPOSINTS) {       # 是小整数则直接使用
        v = small_ints[ival + NSMALLNEGINTS];
        Py_INCREF(v);
#ifdef COUNT_ALLOCS
        if (ival >= 0)
            quick_int_allocs++;
        else
            quick_neg_int_allocs++;
#endif
        return (PyObject *) v;
    }
#endif
    if (free_list == NULL) {
        if ((free_list = fill_free_list()) == NULL)             # 没有空闲的整数对象则分配
            return NULL;
    }
    /* Inline PyObject_New */
    v = free_list;
    free_list = (PyIntObject *)Py_TYPE(v);
    PyObject_INIT(v, &PyInt_Type);
    v->ob_ival = ival;
    return (PyObject *) v;
}
```

创建一个新的整数`257`之后的数据结构：

![image](https://fanchao01.github.io/blog/images/pyint_257.jpg)

### 整数对象的回收 <a id="&#x6574;&#x6570;&#x5BF9;&#x8C61;&#x7684;&#x56DE;&#x6536;"></a>

当整数对象的引用计数归零时则对其进行回收，由函数`int_free`操作

```c
static void
int_free(PyIntObject *v)
{
    Py_TYPE(v) = (struct _typeobject *)free_list;
    free_list = v;
}
```

可以看到被回收的整数对象被连接到`free_list`链表中。这里有个问题，整数对象的内存什么时候才真正释放呢？

### 整数对象的释放 <a id="&#x6574;&#x6570;&#x5BF9;&#x8C61;&#x7684;&#x91CA;&#x653E;"></a>

原来整数对象的真正释放是在最高代的`GC`中进行，当`GC`运行时会调用`PyInt_ClearFreeList`进行整数对象内存的释放`PyInt_ClearFreeList`对整个`block_list`进行遍历，如果其中所有的整数对象的引用计数都为零，则释放整个`block`。可见整数对象的内存是以`PyIntBlock`为单位申请和释放的。

```c
int
PyInt_ClearFreeList(void)
{
    PyIntObject *p;
    PyIntBlock *list, *next;
    int i;
    int u;                      /* remaining unfreed ints per block */
    int freelist_size = 0;
    list = block_list;
    block_list = NULL;
    free_list = NULL;
    while (list != NULL) {
        u = 0;
        for (i = 0, p = &list->objects[0];
             i < N_INTOBJECTS;
             i++, p++) {
            if (PyInt_CheckExact(p) && p->ob_refcnt != 0)
                u++;
        }
        next = list->next;
        if (u) {                        # 遍历block发现其有引用计数部位零的对象
            list->next = block_list;
            block_list = list;
            # 将PyIntBlock中引用计数为零的整数对象重新挂到free_list链表中
            for (i = 0, p = &list->objects[0];
                 i < N_INTOBJECTS;
                 i++, p++) {
                if (!PyInt_CheckExact(p) ||
                    p->ob_refcnt == 0) {
                    Py_TYPE(p) = (struct _typeobject *)
                        free_list;
                    free_list = p;
                }
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
               /* 这段代码没有作用。小整数对象都在small_ints中？
                */
                else if (-NSMALLNEGINTS <= p->ob_ival &&
                         p->ob_ival < NSMALLPOSINTS &&
                         small_ints[p->ob_ival +
                                    NSMALLNEGINTS] == NULL) {
                    Py_INCREF(p);
                    small_ints[p->ob_ival +
                               NSMALLNEGINTS] = p;
                }
#endif
            }
        }
        else {              # 整个block的整数对象引用计数均为零，释放整个block
            PyMem_FREE(list);
        }
        freelist_size += u;
        list = next;
    }
    return freelist_size;
}
```

### 整数对象的操作符 <a id="&#x6574;&#x6570;&#x5BF9;&#x8C61;&#x7684;&#x64CD;&#x4F5C;&#x7B26;"></a>

整数对象定义了许多操作符，可以通过以下代码自行查看。

```c
static PyNumberMethods int_as_number = {
    (binaryfunc)int_add,        /*nb_add*/
    (binaryfunc)int_sub,        /*nb_subtract*/
    (binaryfunc)int_mul,        /*nb_multiply*/
    (binaryfunc)int_classic_div, /*nb_divide*/
    (binaryfunc)int_mod,        /*nb_remainder*/
    (binaryfunc)int_divmod,     /*nb_divmod*/
    (ternaryfunc)int_pow,       /*nb_power*/
    (unaryfunc)int_neg,         /*nb_negative*/
    (unaryfunc)int_int,         /*nb_positive*/
    (unaryfunc)int_abs,         /*nb_absolute*/
    (inquiry)int_nonzero,       /*nb_nonzero*/
    (unaryfunc)int_invert,      /*nb_invert*/
    (binaryfunc)int_lshift,     /*nb_lshift*/
    (binaryfunc)int_rshift,     /*nb_rshift*/
    (binaryfunc)int_and,        /*nb_and*/
    (binaryfunc)int_xor,        /*nb_xor*/
    (binaryfunc)int_or,         /*nb_or*/
    int_coerce,                 /*nb_coerce*/
    (unaryfunc)int_int,         /*nb_int*/
    (unaryfunc)int_long,        /*nb_long*/
    (unaryfunc)int_float,       /*nb_float*/
    (unaryfunc)int_oct,         /*nb_oct*/
    (unaryfunc)int_hex,         /*nb_hex*/
    0,                          /*nb_inplace_add*/
    0,                          /*nb_inplace_subtract*/
    0,                          /*nb_inplace_multiply*/
    0,                          /*nb_inplace_divide*/
    0,                          /*nb_inplace_remainder*/
    0,                          /*nb_inplace_power*/
    0,                          /*nb_inplace_lshift*/
    0,                          /*nb_inplace_rshift*/
    0,                          /*nb_inplace_and*/
    0,                          /*nb_inplace_xor*/
    0,                          /*nb_inplace_or*/
    (binaryfunc)int_div,        /* nb_floor_divide */
    (binaryfunc)int_true_divide, /* nb_true_divide */
    0,                          /* nb_inplace_floor_divide */
    0,                          /* nb_inplace_true_divide */
    (unaryfunc)int_int,         /* nb_index */
};
PyTypeObject PyInt_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "int",
    sizeof(PyIntObject),
    0,
    (destructor)int_dealloc,                    /* tp_dealloc */
    (printfunc)int_print,                       /* tp_print */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    (cmpfunc)int_compare,                       /* tp_compare */
    (reprfunc)int_to_decimal_string,            /* tp_repr */
    &int_as_number,                             /* tp_as_number */
    0,                                          /* tp_as_sequence */
    0,                                          /* tp_as_mapping */
    (hashfunc)int_hash,                         /* tp_hash */
    0,                                          /* tp_call */
    (reprfunc)int_to_decimal_string,            /* tp_str */
    PyObject_GenericGetAttr,                    /* tp_getattro */
    0,                                          /* tp_setattro */
    0,                                          /* tp_as_buffer */
    Py_TPFLAGS_DEFAULT | Py_TPFLAGS_CHECKTYPES |
        Py_TPFLAGS_BASETYPE | Py_TPFLAGS_INT_SUBCLASS,          /* tp_flags */
    int_doc,                                    /* tp_doc */
    0,                                          /* tp_traverse */
    0,                                          /* tp_clear */
    0,                                          /* tp_richcompare */
    0,                                          /* tp_weaklistoffset */
    0,                                          /* tp_iter */
    0,                                          /* tp_iternext */
    int_methods,                                /* tp_methods */
    0,                                          /* tp_members */
    int_getset,                                 /* tp_getset */
    0,                                          /* tp_base */
    0,                                          /* tp_dict */
    0,                                          /* tp_descr_get */
    0,                                          /* tp_descr_set */
    0,                                          /* tp_dictoffset */
    0,                                          /* tp_init */
    0,                                          /* tp_alloc */
    int_new,                                    /* tp_new */
    (freefunc)int_free,                         /* tp_free */
};
```



