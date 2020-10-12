# 字符串对象PyStringObject

### Python字符串对象PyStringObject <a id="Python&#x5B57;&#x7B26;&#x4E32;&#x5BF9;&#x8C61;PyStringObject"></a>

Python的字符串对象是一个不可变对象，任何改变字符串字面值的操作都是重新创建一个新的字符串。

```text
astr = 'astr'
id(astr)
Out[22]: 59244376L
astr += 'another'
id(astr)
Out[24]: 59947360L
```

 字符串对象在Python中用`PyStringObject`表示，扩展定义后如下。

```text
typedef struct {
  Py_ssize_t ob_refcnt;            // 引用计数
  struct _typeobject *ob_type;     // 类型指针
  Py_ssize_t ob_size；             // 字符串的长度，不计算C语言中的结尾NULL
  long ob_shash;                   // 字符串的hash值，没有计算则为-1
  int ob_sstate;                   // 字符串对象的状态： 是否interned等
  char ob_sval[1];                 // 保存字符串的内存，默认先分配1个字符，用来保存额外的末尾NULL值
 
  /* Invariants:
   *     ob_sval contains space for 'ob_size+1' elements.
   *     ob_sval[ob_size] == 0.
   *     ob_shash is the hash of the string or -1 if not computed yet.
   *     ob_sstate != 0 iff the string object is in stringobject.c's
   *       'interned' dictionary; in this case the two references
   *       from 'interned' to this object are *not counted* in ob_refcnt.
   */
} PyStringObject;
```

`ob_type`字符串的类型指针，实际指向`PyString_Type`

`ob_size`保存的是字符串的实际长度，也是通过`len(s)`返回的长度值。而字符串实际占用的内存是`ob_size + 1`，因为C语言中需要额外的`NULL`作为字符串结束标识符。

`ob_sval`是实际存储字符串的内存，分配时会请求`sizeof(PyStringObject)+size`的内存，这样以`ob_sval`开始的内存长度就是`size + 1`的长度，正好用来存放以`NULL`结尾的字符串。

`ob_shash`是字符串的hash值，当字符串用来比较或者作为key时可以加速查找速度，默认值为-1。

```text
static long
string_hash(PyStringObject *a)
{
    register Py_ssize_t len;
    register unsigned char *p;
    register long x;
#ifdef Py_DEBUG
    assert(_Py_HashSecret_Initialized);
#endif
    if (a->ob_shash != -1)
        return a->ob_shash;
    len = Py_SIZE(a);
    /*
      We make the hash of the empty string be 0, rather than using
      (prefix ^ suffix), since this slightly obfuscates the hash secret
    */
    if (len == 0) {
        a->ob_shash = 0;
        return 0;
    }
    p = (unsigned char *) a->ob_sval;
    x = _Py_HashSecret.prefix;
    x ^= *p << 7;
    while (--len >= 0)
        x = (1000003*x) ^ *p++;
    x ^= Py_SIZE(a);
    x ^= _Py_HashSecret.suffix;
    if (x == -1)
        x = -2;
    a->ob_shash = x;
    return x;
```

 `ob_sstate`记录字符串对象的状态。字符串可能有三种状态：

```text
//Python/objects/stringobject.h
#define SSTATE_NOT_INTERNED 0           // 字符串没有被interned
#define SSTATE_INTERNED_MORTAL 1        // 字符串被interned，可以被回收
#define SSTATE_INTERNED_IMMORTAL 2      // 字符串永久interned，不会被回收
```

### 字符串的interned <a id="&#x5B57;&#x7B26;&#x4E32;&#x7684;interned"></a>

字符串对象是不可变对象，因此相同的字面值的变量可以绑定到相同的字符串对象上，这样减少了字符串对象的创建次数。这样的行为称为`interned`。默认情况下空字符串和单字符字符串会被`interned`。

```text
// Python/objects/stringobject.c
PyObject *
PyString_FromString(const char *str)
{
    register size_t size;
    register PyStringObject *op;
    assert(str != NULL);
    size = strlen(str);
    if (size > PY_SSIZE_T_MAX - PyStringObject_SIZE) {
        PyErr_SetString(PyExc_OverflowError,
            "string is too long for a Python string");
        return NULL;
    }
    // 如果是空字符串直接返回
    if (size == 0 && (op = nullstring) != NULL) {
#ifdef COUNT_ALLOCS
        null_strings++;
#endif
        Py_INCREF(op);
        return (PyObject *)op;
    }
    // 如果是单字符串，先从characters中查找是否存在
    if (size == 1 && (op = characters[*str & UCHAR_MAX]) != NULL) {
#ifdef COUNT_ALLOCS
        one_strings++;
#endif
        Py_INCREF(op);
        return (PyObject *)op;
    }
    /* Inline PyObject_NewVar */
    op = (PyStringObject *)PyObject_MALLOC(PyStringObject_SIZE + size);
    if (op == NULL)
        return PyErr_NoMemory();
    PyObject_INIT_VAR(op, &PyString_Type, size);
    op->ob_shash = -1;
    op->ob_sstate = SSTATE_NOT_INTERNED;
    Py_MEMCPY(op->ob_sval, str, size+1);
    /* share short strings */
    // 空字符串进行interned
    if (size == 0) {
        PyObject *t = (PyObject *)op;
        PyString_InternInPlace(&t);
        op = (PyStringObject *)t;
        nullstring = op;
        Py_INCREF(op);
    // 单字符串保存在characters中，并且进行interned
    } else if (size == 1) {
        PyObject *t = (PyObject *)op;
        PyString_InternInPlace(&t);
        op = (PyStringObject *)t;
        characters[*str & UCHAR_MAX] = op;
        Py_INCREF(op);
    }
    return (PyObject *) op;
}
```

 另外一些情况下，例如`__dict__`、模块名字等预计会被大量重复使用或者永久使用的字符串，在创建时也会调用`PyString_InternInPlace`进行`interned`操作。

```text
// python/objects/stringobject.c
PyAPI_FUNC(PyObject *) PyString_FromString(const char *);
 
PyAPI_FUNC(PyObject *) PyString_FromStringAndSize(const char *, Py_ssize_t);
 
// SSTATE_INTERNED_MORTAL, 计数0会被回收
PyObject *
PyString_InternFromString(const char *cp)
{
    PyObject *s = PyString_FromString(cp);
    if (s == NULL)
        return NULL;
    PyString_InternInPlace(&s);
    return s;
}
 
// SSTATE_INTERNED_IMMORTAL, 永远不会被销毁
void
PyString_InternImmortal(PyObject **p)
{
}
void
PyString_InternInPlace(PyObject **p)
{
    register PyStringObject *s = (PyStringObject *)(*p);
 
    PyObject *t;
 
    // 检查值使用在PyStringObject上, 派生类不适用
    if (s == NULL || !PyString_Check(s))
        Py_FatalError("PyString_InternInPlace: strings only please!");
    /* If it's a string subclass, we don't really know what putting it in the interned dict might do. */
 
    // 不是字符串类型, 返回
    if (!PyString_CheckExact(s))
        return;
    // 本身已经intern了(标志位ob_sstate), 不重复进行, 返回
    if (PyString_CHECK_INTERNED(s))
        return;
 
    // 未初始化字典, 初始化之
    if (interned == NULL) {
        // 注意这里
        interned = PyDict_New();
        if (interned == NULL) {
            PyErr_Clear(); /* Don't leave an exception */
            return;
        }
    }
 
    // 在interned字典中已存在, 修改, 返回intern独享
    t = PyDict_GetItem(interned, (PyObject *)s);
    if (t) {
        Py_INCREF(t);
        Py_DECREF(*p);
        *p = t;
        return;
    }
 
    // 在interned字典中不存在, 放进去
    if (PyDict_SetItem(interned, (PyObject *)s, (PyObject *)s)  0) {
        PyErr_Clear();
        return;
    }
 
    /* 加入interned字典(key-value)会导致refcnt+2,
     * 这里面去掉interned的引用，以使其正确回收
     */
    /* The two references in interned are not counted by refcnt.
       The string deallocator will take care of this */
    Py_REFCNT(s) -= 2;
 
    // 修改字符串对象的ob_sstate标志位, SSTATE_INTERNED_MORTAL
    PyString_CHECK_INTERNED(s) = SSTATE_INTERNED_MORTAL;
}
// 其他代码大量进行intern操作
dict_str = PyString_InternFromString("__dict__")
lenstr = PyString_InternFromString("__len__")
s_true = PyString_InternFromString("true")
empty_array = PyString_InternFromString("[]")
```

### 字符串对象的回收 <a id="&#x5B57;&#x7B26;&#x4E32;&#x5BF9;&#x8C61;&#x7684;&#x56DE;&#x6536;"></a>

当字符串的引用计数为零时会被回收。

```text
static void
string_dealloc(PyObject *op)
{
    switch (PyString_CHECK_INTERNED(op)) {
        case SSTATE_NOT_INTERNED:
            break;
            
        // 从interned字典中去掉，然后再tp_free掉
        case SSTATE_INTERNED_MORTAL:
            /* revive dead object temporarily for DelItem */
            Py_REFCNT(op) = 3;
            if (PyDict_DelItem(interned, op) != 0)
                Py_FatalError(
                    "deletion of interned string failed");
            break;
        // 这种类型的字符串不会被回收，一直有引用
        case SSTATE_INTERNED_IMMORTAL:
            Py_FatalError("Immortal interned string died.");
        default:
            Py_FatalError("Inconsistent interned string state.");
    }
    Py_TYPE(op)->tp_free(op);
}
```

### 字符串对象的其他操作 <a id="&#x5B57;&#x7B26;&#x4E32;&#x5BF9;&#x8C61;&#x7684;&#x5176;&#x4ED6;&#x64CD;&#x4F5C;"></a>

可以通过字符串对象的类的结构中找到对象的操作函数。

```text
// python/objects/stringobject.c
PyTypeObject PyString_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "str",
    PyStringObject_SIZE,
    sizeof(char),
    string_dealloc,                             /* tp_dealloc */
    (printfunc)string_print,                    /* tp_print */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    0,                                          /* tp_compare */
    string_repr,                                /* tp_repr */
    &string_as_number,                          /* tp_as_number */
    &string_as_sequence,                        /* tp_as_sequence */
    &string_as_mapping,                         /* tp_as_mapping */
    (hashfunc)string_hash,                      /* tp_hash */
    0,                                          /* tp_call */
    string_str,                                 /* tp_str */
    PyObject_GenericGetAttr,                    /* tp_getattro */
    0,                                          /* tp_setattro */
    &string_as_buffer,                          /* tp_as_buffer */
    Py_TPFLAGS_DEFAULT | Py_TPFLAGS_CHECKTYPES |
        Py_TPFLAGS_BASETYPE | Py_TPFLAGS_STRING_SUBCLASS |
        Py_TPFLAGS_HAVE_NEWBUFFER,              /* tp_flags */
    string_doc,                                 /* tp_doc */
    0,                                          /* tp_traverse */
    0,                                          /* tp_clear */
    (richcmpfunc)string_richcompare,            /* tp_richcompare */
    0,                                          /* tp_weaklistoffset */
    0,                                          /* tp_iter */
    0,                                          /* tp_iternext */
    string_methods,                             /* tp_methods */
    0,                                          /* tp_members */
    0,                                          /* tp_getset */
    &PyBaseString_Type,                         /* tp_base */
    0,                                          /* tp_dict */
    0,                                          /* tp_descr_get */
    0,                                          /* tp_descr_set */
    0,                                          /* tp_dictoffset */
    0,                                          /* tp_init */
    0,                                          /* tp_alloc */
    string_new,                                 /* tp_new */
    PyObject_Del,                               /* tp_free */
};
```

 `tp_base`被赋值为`PyBaseString_Type`，因此字符串对象是`basestring`的子类。

