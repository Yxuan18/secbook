# 循环垃圾回收器

### Python垃圾回收概述 <a id="Python&#x5783;&#x573E;&#x56DE;&#x6536;&#x6982;&#x8FF0;"></a>

Python中的垃圾回收机制基于引用计数\(ob\_refcnt\)，因此需要解决循环引用导致引用计数不能归零的问题。例如

```c
# create[1]
list1=[]              # del
list2=[list1]         # del
list1.append(list2)
list3 = []            # del
list4 = []
list5 = [list3, list4] # del
list6 = [list3, list4]
list3.append(list4)
# del[2]
del list1; del list2;del list3;del list5 # [2]
# list1.ob_refcnt == list2.ob_refcnt == 1
# collect[3]
gc.collect()
```

虽然`list1`与`list2`已经成为需要回收的垃圾，但是由于相互引用导致引用计数不能归零，从而不能触发自动回收。因此Python引入了`循环垃圾收集器`。

### 循环垃圾收集器的原理 <a id="&#x5FAA;&#x73AF;&#x5783;&#x573E;&#x6536;&#x96C6;&#x5668;&#x7684;&#x539F;&#x7406;"></a>

判断对象是否为垃圾的逻辑比较直白，有外部引用或者被有外部引用的对象引用的对象为非垃圾对象；否则为垃圾对象。具体过程为，遍历所有对象将对象中引用的元素\(其他对象\)的引用计数减一，最后引用计数不归零的对象\(存在外部引用\)不是垃圾对象；被不是垃圾对象引用的元素\(其他对象\)也不是垃圾对象；剩余的则为垃圾对象。可以归纳为如下步骤：

1. 创建可能存在循环应用的对象时，将该对象纳入链表进行管理
2. 遍历所有纳入管理的对象，将对象引用的元素\(其他对象\)的引用计数减一
3. 再次遍历:  
   &lt;1&gt; 处理对象：对该对象进行标记  
   所有引用计数为零的对象没有外部引用，标记为可能是垃圾；  
   所有引用计数不为零的对象存在外部引用，必然不是垃圾。

   \|0 \|可能是垃圾 \|list1、list2、list3、list5\|  
   \|&gt;0 \|不是垃圾 \|list4、list6\|

   &lt;2&gt; 处理对象：遍历不是垃圾对象中的元素，不是垃圾对象中的元素必然不是垃圾

4. 最后没有被确定不是垃圾的对象就是垃圾对象

这部分处理代码比较复杂，每个对象可能作为两种角色进行处理。作为代中的对象以及作为对象中的引用元素。如果作为元素被处理，则肯定不是垃圾。

各个阶段对象中的引用计数

![image](https://fanchao01.github.io/blog/images/python-gcmodule.jpg)

### 垃圾对象不一定能被自动回收 <a id="&#x5783;&#x573E;&#x5BF9;&#x8C61;&#x4E0D;&#x4E00;&#x5B9A;&#x80FD;&#x88AB;&#x81EA;&#x52A8;&#x56DE;&#x6536;"></a>

垃圾对象不一定能被自动回收。所以上面的步骤只能确定垃圾对象，然后对垃圾对象进行额外处理甄别不能回收和能被回收的部分。

* 垃圾对象：没有外部引用的对象，也没有被有外部引用的对象引用
* 可回收对象：垃圾对象中能够被自动回收的对象
* reachable：非垃圾对象，存在外部引用或者被外部引用的对象引用
* unrechable: 垃圾对象
* collectable: 可回收对象
* finalizers: 垃圾对象中不能被自动回收的对象。一些对象存在析构函数并且相互引用，这样的对象Python不能自动确定回收顺序，因此不能被自动回收。

### 不是所有对象都纳入循环垃圾收集器 <a id="&#x4E0D;&#x662F;&#x6240;&#x6709;&#x5BF9;&#x8C61;&#x90FD;&#x7EB3;&#x5165;&#x5FAA;&#x73AF;&#x5783;&#x573E;&#x6536;&#x96C6;&#x5668;"></a>

一些基本对象不会产生循环引用，例如int、float、string等，所以没有必须使用循环垃圾收集器，基本的引用计数回收机制即可。还有一些容器类对象，他们中的元素都是基本元素不会引起循环引用，例如{‘a’:1}、\(1, 2, 3\)，因此也不纳入循环垃圾收集器。所以只有部分容器类对象、生成器、含`__del__`类等才纳入循环垃圾收集器。

### 垃圾回收中的代 <a id="&#x5783;&#x573E;&#x56DE;&#x6536;&#x4E2D;&#x7684;&#x4EE3;"></a>

如上分析，整个循环垃圾收集的效率严重依赖可能引起循环引用的对象的个数。为了减少垃圾回收的动作，Python将对象分代：存活越长的对象越不可能是垃圾，就减少对其进行垃圾回收的次数。那么`存活`的时间长短就用经过了几次垃圾回收来判断，于是刚创建的对象为一代，当经过一次垃圾回收还存活的对象放入二代；多次一代垃圾回收后，才进行一次二代垃圾回收。Python将整个对象分为三代，当分配足够数量的对象后\(700\)进行一次一代回收；当进行一定数量\(10\)一代回收后进行二代回收；同理进行三代回收。

### gcmodule.c源码分析 <a id="gcmodule-c&#x6E90;&#x7801;&#x5206;&#x6790;"></a>

```c
/*
  Reference Cycle Garbage Collection
  ==================================
  Neil Schemenauer <nas@arctrix.com>
  Based on a post on the python-dev list.  Ideas from Guido van Rossum,
  Eric Tiedemann, and various others.
  http://www.arctrix.com/nas/python/gc/
  http://www.python.org/pipermail/python-dev/2000-March/003869.html
  http://www.python.org/pipermail/python-dev/2000-March/004010.html
  http://www.python.org/pipermail/python-dev/2000-March/004022.html
  For a highlevel view of the collection process, read the collect
  function.
*/
#include "Python.h"
#include "frameobject.h"        /* for PyFrame_ClearFreeList */
/* 
 * 带有垃圾回收头部的对象的内存模型
 * ------------------------------
 * g          o
 * |PyGC_Head | PyObject | other|
 * ------------------------------
 *           
 * 因此给定地址o计算g就是将地址减少sizeof(PyGC_HEAD)
 * 
 * 说明：
 * 1. 垃圾管理有两部分 引用计数 和 循环垃圾收集器
 * 2. 简单的对象不产生循环引用使用引用计数即可：例如 int、float、string等
 * 3. 有可能产生循环引用的对象(容器类)才需要循环垃圾收集器: 例如 list、dict
 * 4. Python对一些容器类对象进行优化，如果容器内的元素不会产生循环引用，
 *    则不会纳入循环引用管理: 例如 {'a': 1}, [1, 2, 'a']
 */
/* Get an object's GC head */
#define AS_GC(o) ((PyGC_Head *)(o)-1)
/* Get the object given the GC head */
#define FROM_GC(g) ((PyObject *)(((PyGC_Head *)g)+1))
/*** Global GC state ***/
/* 每一代的寿命是比该代年轻的那代运行的次数
 * 例如 第1代没运行一次，第2代年龄增加一岁
 */
struct gc_generation {
    PyGC_Head head;
    /* 寿命 */
    int threshold; /* collection threshold */
    /* 年龄，到了寿命就该回收了 */
    int count; /* count of allocations or collections of younger
                  generations */
};
/* Python只有3代 */
#define NUM_GENERATIONS 3
#define GEN_HEAD(n) (&generations[n].head)
/* linked lists of container objects */
/* linked lists结构，初始化时每个链表指向自己的头
 * 内存模型如下:
 * ---------------------------
 * g0                        |
 * |next(g0)|prev(g0)|0|700|0|
 * g1                        |
 * |next(g1)|prev(g1)|0|10 |0| 
 * g2                         |
 * |next(g2)|prev(g2)|0|10 |0|
 * ---------------------------
 * generation[i].head.next = generation[i].head.prev = &generation[i]
 * &generation[i] == &generation[i].head
 */
static struct gc_generation generations[NUM_GENERATIONS] = {
    /* PyGC_Head,                               threshold,      count */
    {{{GEN_HEAD(0), GEN_HEAD(0), 0}},           700,            0},
    {{{GEN_HEAD(1), GEN_HEAD(1), 0}},           10,             0},
    {{{GEN_HEAD(2), GEN_HEAD(2), 0}},           10,             0},
};
PyGC_Head *_PyGC_generation0 = GEN_HEAD(0);
static int enabled = 1; /* automatic collection enabled? */
/* true if we are currently running the collector */
static int collecting = 0;
/* list of uncollectable objects  */
/* unreachable:
 * 没有外部引用，只有被彼此循环引用的对象。例如
 * list1=[];list2=[list1];list1.append(list2)
 * del list1; del list2
 * 那么list1和list2不能通过其他对象到达
 *
 * uncollectable:
 * unreachable并且有 __del__ 方法的对象
 *  __del__由用户自定义，可能引用了其他的对象。
 */
static PyObject *garbage = NULL;
/* Python string to use if unhandled exception occurs */
static PyObject *gc_str = NULL;
/* Python string used to look for __del__ attribute. */
static PyObject *delstr = NULL;
/* This is the number of objects who survived the last full collection. It
   approximates the number of long lived objects tracked by the GC.
   (by "full collection", we mean a collection of the oldest generation).
*/
/* 经过一次第三代收集还存活的则为 long_lived 
 * 运行一次第三代的垃圾回收叫 full collection。
 */
static Py_ssize_t long_lived_total = 0;
/* This is the number of objects who survived all "non-full" collections,
   and are awaiting to undergo a full collection for the first time.
*/
/* 第二代回收时，不能被回收而移动到第三代的对象个数 */
static Py_ssize_t long_lived_pending = 0;
/*
   NOTE: about the counting of long-lived objects.
   To limit the cost of garbage collection, there are two strategies;
     - make each collection faster, e.g. by scanning fewer objects
     - do less collections
   This heuristic is about the latter strategy.
   In addition to the various configurable thresholds, we only trigger a
   full collection if the ratio
    long_lived_pending / long_lived_total
   is above a given value (hardwired to 25%).
   The reason is that, while "non-full" collections (i.e., collections of
   the young and middle generations) will always examine roughly the same
   number of objects -- determined by the aforementioned thresholds --,
   the cost of a full collection is proportional to the total number of
   long-lived objects, which is virtually unbounded.
   Indeed, it has been remarked that doing a full collection every
   <constant number> of object creations entails a dramatic performance
   degradation in workloads which consist in creating and storing lots of
   long-lived objects (e.g. building a large list of GC-tracked objects would
   show quadratic performance, instead of linear as expected: see issue #4074).
   Using the above ratio, instead, yields amortized linear performance in
   the total number of objects (the effect of which can be summarized
   thusly: "each full garbage collection is more and more costly as the
   number of objects grows, but we do fewer and fewer of them").
   This heuristic was suggested by Martin von Löwis on python-dev in
   June 2008. His original analysis and proposal can be found at:
    http://mail.python.org/pipermail/python-dev/2008-June/080579.html
*/
/*
   NOTE: about untracking of mutable objects.
   Certain types of container cannot participate in a reference cycle, and
   so do not need to be tracked by the garbage collector. Untracking these
   objects reduces the cost of garbage collections. However, determining
   which objects may be untracked is not free, and the costs must be
   weighed against the benefits for garbage collection.
   There are two possible strategies for when to untrack a container:
   i) When the container is created.
   ii) When the container is examined by the garbage collector.
   Tuples containing only immutable objects (integers, strings etc, and
   recursively, tuples of immutable objects) do not need to be tracked.
   The interpreter creates a large number of tuples, many of which will
   not survive until garbage collection. It is therefore not worthwhile
   to untrack eligible tuples at creation time.
   Instead, all tuples except the empty tuple are tracked when created.
   During garbage collection it is determined whether any surviving tuples
   can be untracked. A tuple can be untracked if all of its contents are
   already not tracked. Tuples are examined for untracking in all garbage
   collection cycles. It may take more than one cycle to untrack a tuple.
   Dictionaries containing only immutable objects also do not need to be
   tracked. Dictionaries are untracked when created. If a tracked item is
   inserted into a dictionary (either as a key or value), the dictionary
   becomes tracked. During a full garbage collection (all generations),
   the collector will untrack any dictionaries whose contents are not
   tracked.
   The module provides the python function is_tracked(obj), which returns
   the CURRENT tracking status of the object. Subsequent garbage
   collections may change the tracking status of the object.
   Untracking of certain containers was introduced in issue #4688, and
   the algorithm was refined in response to issue #14775.
*/
 /* 简单说：
  * 垃圾回收的效率取决于纳入垃圾管理的对象的数量
  */
/* set for debugging information */
#define DEBUG_STATS             (1<<0) /* print collection statistics */
#define DEBUG_COLLECTABLE       (1<<1) /* print collectable objects */
#define DEBUG_UNCOLLECTABLE     (1<<2) /* print uncollectable objects */
#define DEBUG_INSTANCES         (1<<3) /* print instances */
#define DEBUG_OBJECTS           (1<<4) /* print other objects */
#define DEBUG_SAVEALL           (1<<5) /* save all garbage in gc.garbage */
#define DEBUG_LEAK              DEBUG_COLLECTABLE | \
                DEBUG_UNCOLLECTABLE | \
                DEBUG_INSTANCES | \
                DEBUG_OBJECTS | \
                DEBUG_SAVEALL
static int debug;
static PyObject *tmod = NULL;
/*--------------------------------------------------------------------------
gc_refs values.
Between collections, every gc'ed object has one of two gc_refs values:
GC_UNTRACKED
    The initial state; objects returned by PyObject_GC_Malloc are in this
    state.  The object doesn't live in any generation list, and its
    tp_traverse slot must not be called.
    
    没有纳入收集器管理的对象，例如 刚通过PyObject_GC_Malloc初始化的对象，
    int，float等，当然也不存在任何"代"中。
GC_REACHABLE
    The object lives in some generation list, and its tp_traverse is safe to
    call.  An object transitions to GC_REACHABLE when PyObject_GC_Track
    is called.
    纳入了收集器管理的对象，并且可以通过各对象的tp_traverse处理的对象
    纳入管理，并且在有向图中可以到达(不存在循环引用)
代中对象的处理和gc_refs：
1. 遍历整个代中的对象，将对象中的所有元素gc_refs-1
2. 再次遍历整个代中的对象，此时gc_refs：
   0：对象只被代中的其他对象引用，例如：
      list1=[]; list2=[list1];del list1;del list2
      设置gc_refs = GC_TENTATIVELY_UNREACHABLE
   >0: 对象有外部引用
      设置gc_refs = 1
3. 遍历代中的对象中的元素，此时gc_refs：
   GC_TENTATIVELY_UNREACHABLE: 
       说明之前作为代中的对象处理过，确实是unreachable
       设置GC_REACHABLE
   1，GC_UNTRACKED，GC_REACHABLE： 
       保持不变，仍在代中
During a collection, gc_refs can temporarily take on other states:
>= 0
    At the start of a collection, update_refs() copies the true refcount
    to gc_refs, for each object in the generation being collected.
    subtract_refs() then adjusts gc_refs so that it equals the number of
    times an object is referenced directly from outside the generation
    being collected.
    gc_refs remains >= 0 throughout these steps.
GC_TENTATIVELY_UNREACHABLE
    move_unreachable() then moves objects not reachable (whether directly or
    indirectly) from outside the generation into an "unreachable" set.
    Objects that are found to be reachable have gc_refs set to GC_REACHABLE
    again.  Objects that are found to be unreachable have gc_refs set to
    GC_TENTATIVELY_UNREACHABLE.  It's "tentatively" because the pass doing
    this can't be sure until it ends, and GC_TENTATIVELY_UNREACHABLE may
    transition back to GC_REACHABLE.
    遍历时的临时状态，当遍历时将暂时不能到达的对象设置为该状态。
    Only objects with GC_TENTATIVELY_UNREACHABLE still set are candidates
    for collection.  If it's decided not to collect such an object (e.g.,
    it has a __del__ method), its gc_refs is restored to GC_REACHABLE again.
----------------------------------------------------------------------------
*/
#define GC_UNTRACKED                    _PyGC_REFS_UNTRACKED
#define GC_REACHABLE                    _PyGC_REFS_REACHABLE
#define GC_TENTATIVELY_UNREACHABLE      _PyGC_REFS_TENTATIVELY_UNREACHABLE
#define IS_TRACKED(o) ((AS_GC(o))->gc.gc_refs != GC_UNTRACKED)
#define IS_REACHABLE(o) ((AS_GC(o))->gc.gc_refs == GC_REACHABLE)
#define IS_TENTATIVELY_UNREACHABLE(o) ( \
    (AS_GC(o))->gc.gc_refs == GC_TENTATIVELY_UNREACHABLE)
/*** list functions ***/
static void
gc_list_init(PyGC_Head *list)
{
    list->gc.gc_prev = list;
    list->gc.gc_next = list;
}
static int
gc_list_is_empty(PyGC_Head *list)
{
    return (list->gc.gc_next == list);
}
#if 0
/* This became unused after gc_list_move() was introduced. */
/* Append `node` to `list`. */
static void
gc_list_append(PyGC_Head *node, PyGC_Head *list)
{
    node->gc.gc_next = list;
    node->gc.gc_prev = list->gc.gc_prev;
    node->gc.gc_prev->gc.gc_next = node;
    list->gc.gc_prev = node;
}
#endif
/* Remove `node` from the gc list it's currently in. */
static void
gc_list_remove(PyGC_Head *node)
{
    node->gc.gc_prev->gc.gc_next = node->gc.gc_next;
    node->gc.gc_next->gc.gc_prev = node->gc.gc_prev;
    node->gc.gc_next = NULL; /* object is not currently tracked */
}
/* Move `node` from the gc list it's currently in (which is not explicitly
 * named here) to the end of `list`.  This is semantically the same as
 * gc_list_remove(node) followed by gc_list_append(node, list).
 */
static void
gc_list_move(PyGC_Head *node, PyGC_Head *list)
{
    PyGC_Head *new_prev;
    PyGC_Head *current_prev = node->gc.gc_prev;
    PyGC_Head *current_next = node->gc.gc_next;
    /* Unlink from current list. */
    current_prev->gc.gc_next = current_next;
    current_next->gc.gc_prev = current_prev;
    /* Relink at end of new list. */
    new_prev = node->gc.gc_prev = list->gc.gc_prev;
    new_prev->gc.gc_next = list->gc.gc_prev = node;
    node->gc.gc_next = list;
}
/* append list `from` onto list `to`; `from` becomes an empty list */
/* 将 from 整体挂到 to 的头部 */
static void
gc_list_merge(PyGC_Head *from, PyGC_Head *to)
{
    PyGC_Head *tail;
    assert(from != to);
    if (!gc_list_is_empty(from)) {
        tail = to->gc.gc_prev;
        tail->gc.gc_next = from->gc.gc_next;
        tail->gc.gc_next->gc.gc_prev = tail;
        to->gc.gc_prev = from->gc.gc_prev;
        to->gc.gc_prev->gc.gc_next = to;
    }
    gc_list_init(from);
}
static Py_ssize_t
gc_list_size(PyGC_Head *list)
{
    PyGC_Head *gc;
    Py_ssize_t n = 0;
    for (gc = list->gc.gc_next; gc != list; gc = gc->gc.gc_next) {
        n++;
    }
    return n;
}
/* Append objects in a GC list to a Python list.
 * Return 0 if all OK, < 0 if error (out of memory for list).
 */
static int
append_objects(PyObject *py_list, PyGC_Head *gc_list)
{
    PyGC_Head *gc;
    for (gc = gc_list->gc.gc_next; gc != gc_list; gc = gc->gc.gc_next) {
        PyObject *op = FROM_GC(gc);
        if (op != py_list) {
            if (PyList_Append(py_list, op)) {
                return -1; /* exception */
            }
        }
    }
    return 0;
}
/*** end of list stuff ***/
/* Set all gc_refs = ob_refcnt.  After this, gc_refs is > 0 for all objects
 * in containers, and is GC_REACHABLE for all tracked gc objects not in
 * containers.
 */
static void
update_refs(PyGC_Head *containers)
{
    PyGC_Head *gc = containers->gc.gc_next;
    for (; gc != containers; gc = gc->gc.gc_next) {
        assert(gc->gc.gc_refs == GC_REACHABLE);
        gc->gc.gc_refs = Py_REFCNT(FROM_GC(gc));
        /* Python's cyclic gc should never see an incoming refcount
         * of 0:  if something decref'ed to 0, it should have been
         * deallocated immediately at that time.
         * Possible cause (if the assert triggers):  a tp_dealloc
         * routine left a gc-aware object tracked during its teardown
         * phase, and did something-- or allowed something to happen --
         * that called back into Python.  gc can trigger then, and may
         * see the still-tracked dying object.  Before this assert
         * was added, such mistakes went on to allow gc to try to
         * delete the object again.  In a debug build, that caused
         * a mysterious segfault, when _Py_ForgetReference tried
         * to remove the object from the doubly-linked list of all
         * objects a second time.  In a release build, an actual
         * double deallocation occurred, which leads to corruption
         * of the allocator's internal bookkeeping pointers.  That's
         * so serious that maybe this should be a release-build
         * check instead of an assert?
         */
        assert(gc->gc.gc_refs != 0);
    }
}
/* A traversal callback for subtract_refs. */
static int
visit_decref(PyObject *op, void *data)
{
    assert(op != NULL);
    if (PyObject_IS_GC(op)) {
        PyGC_Head *gc = AS_GC(op);
        /* We're only interested in gc_refs for objects in the
         * generation being collected, which can be recognized
         * because only they have positive gc_refs.
         */
        assert(gc->gc.gc_refs != 0); /* else refcount was too small */
        if (gc->gc.gc_refs > 0)
            gc->gc.gc_refs--;
    }
    return 0;
}
/* Subtract internal references from gc_refs.  After this, gc_refs is >= 0
 * for all objects in containers, and is GC_REACHABLE for all tracked gc
 * objects not in containers.  The ones with gc_refs > 0 are directly
 * reachable from outside containers, and so can't be collected.
 */
/*
 * 调用容器类的tp_traverse对每个元素的gc_refs-1，
 * 如果gc_refs>0说明元素在该容器外还有引用，所以不能回收
 */
static void
subtract_refs(PyGC_Head *containers)
{
    traverseproc traverse;
    PyGC_Head *gc = containers->gc.gc_next;
    for (; gc != containers; gc=gc->gc.gc_next) {
        traverse = Py_TYPE(FROM_GC(gc))->tp_traverse;
        (void) traverse(FROM_GC(gc),
                       (visitproc)visit_decref,
                       NULL);
    }
}
/* A traversal callback for move_unreachable. */
/* op 是容器类的元素；reachable是代本身 */
static int
visit_reachable(PyObject *op, PyGC_Head *reachable)
{
    if (PyObject_IS_GC(op)) {
        PyGC_Head *gc = AS_GC(op);
        const Py_ssize_t gc_refs = gc->gc.gc_refs;
        if (gc_refs == 0) {
            /* This is in move_unreachable's 'young' list, but
             * the traversal hasn't yet gotten to it.  All
             * we need to do is tell move_unreachable that it's
             * reachable.
             */
            /*
             * 元素没有引用，说明只有元素所在的容器本身引用该元素
             * 举例： list1 = []; list2=[]; list1.append(list2); 
             *        del list2
             * 
             * list1->ob_refcnt == 1, list->gc_refs == 1
             * list2->ob_refcnt == 1, list->gc_refs == 0
             *
             * 所以：
             * list2作为list1的元素进入当前的处理，依然是reachable
             */
            gc->gc.gc_refs = 1;
        }
        else if (gc_refs == GC_TENTATIVELY_UNREACHABLE) {
            /* This had gc_refs = 0 when move_unreachable got
             * to it, but turns out it's reachable after all.
             * Move it back to move_unreachable's 'young' list,
             * and move_unreachable will eventually get to it
             * again.
             */
            /*
             * 元素之前经历过1次move_unreachable处理并且被认为是unreachable
             * 例如：
             * list1=[];list2=[list1]; list1.append(list2)
             * del list1;
             *
             * 第一次，list1作为容器被move_unreachable处理
             * 会被标记为GC_TENTATIVELY_UNREACHABLE，认为其可以被回收
             *
             * 第二次，list1作为list2的元素会进入到该处
             * 此时证明list1确实是list2的元素，是reachable的
             *
             * 这里的reachable为代链表本身，就是将gc移动到了链表尾端
             */
            gc_list_move(gc, reachable);
            gc->gc.gc_refs = 1;
        }
        /* Else there's nothing to do.
         * If gc_refs > 0, it must be in move_unreachable's 'young'
         * list, and move_unreachable will eventually get to it.
         * If gc_refs == GC_REACHABLE, it's either in some other
         * generation so we don't care about it, or move_unreachable
         * already dealt with it.
         * If gc_refs == GC_UNTRACKED, it must be ignored.
         */
        /*
         * gc_refs > 0 : 处理过了证明是reachable
         * GC_REACHABLE：标记为reachable的元素
         * GC_UNTRACKED：不需要循环垃圾收集器处理的元素
         */
         else {
            assert(gc_refs > 0
                   || gc_refs == GC_REACHABLE
                   || gc_refs == GC_UNTRACKED);
         }
    }
    return 0;
}
/* Move the unreachable objects from young to unreachable.  After this,
 * all objects in young have gc_refs = GC_REACHABLE, and all objects in
 * unreachable have gc_refs = GC_TENTATIVELY_UNREACHABLE.  All tracked
 * gc objects not in young or unreachable still have gc_refs = GC_REACHABLE.
 * All objects in young after this are directly or indirectly reachable
 * from outside the original young; and all objects in unreachable are
 * not.
 */
static void
move_unreachable(PyGC_Head *young, PyGC_Head *unreachable)
{
    PyGC_Head *gc = young->gc.gc_next;
    /* Invariants:  all objects "to the left" of us in young have gc_refs
     * = GC_REACHABLE, and are indeed reachable (directly or indirectly)
     * from outside the young list as it was at entry.  All other objects
     * from the original young "to the left" of us are in unreachable now,
     * and have gc_refs = GC_TENTATIVELY_UNREACHABLE.  All objects to the
     * left of us in 'young' now have been scanned, and no objects here
     * or to the right have been scanned yet.
     */
    while (gc != young) {
        PyGC_Head *next;
        if (gc->gc.gc_refs) {
            /* gc is definitely reachable from outside the
             * original 'young'.  Mark it as such, and traverse
             * its pointers to find any other objects that may
             * be directly reachable from it.  Note that the
             * call to tp_traverse may append objects to young,
             * so we have to wait until it returns to determine
             * the next object to visit.
             */
            /* 例如：
             * list1=[];list2=[list1];list1.append(list2)
             * del list1;
             * 
             * 经过subtract_refs后：
             * list1.gc_refs==0; list2.gc_refs==1
             * 所以此时的list2有外部的引用，所以是reachable
             */
            PyObject *op = FROM_GC(gc);
            traverseproc traverse = Py_TYPE(op)->tp_traverse;
            assert(gc->gc.gc_refs > 0);
            gc->gc.gc_refs = GC_REACHABLE;
            (void) traverse(op,
                            (visitproc)visit_reachable,
                            (void *)young);
            next = gc->gc.gc_next;
            /* 如果对象op中的元素都是untracked的，标记op为untracked */
            if (PyTuple_CheckExact(op)) {
                _PyTuple_MaybeUntrack(op);
            }
        }
        else {
            /* This *may* be unreachable.  To make progress,
             * assume it is.  gc isn't directly reachable from
             * any object we've already traversed, but may be
             * reachable from an object we haven't gotten to yet.
             * visit_reachable will eventually move gc back into
             * young if that's so, and we'll see it again.
             */
            /* 代中的对象引用计数为0，说明没有外部引用，可能是unreachable
             * 如果再作为对象(容器)的元素被visit_reachable处理到
             * 说明该对象只作为reachable对象的元素，所以也是reachable
             */
            next = gc->gc.gc_next;
            gc_list_move(gc, unreachable);
            gc->gc.gc_refs = GC_TENTATIVELY_UNREACHABLE;
        }
        gc = next;
    }
}
/* Return true if object has a finalization method.
 * CAUTION:  An instance of an old-style class has to be checked for a
 *__del__ method, and earlier versions of this used to call PyObject_HasAttr,
 * which in turn could call the class's __getattr__ hook (if any).  That
 * could invoke arbitrary Python code, mutating the object graph in arbitrary
 * ways, and that was the source of some excruciatingly subtle bugs.
 */
/*
 * 有几种自定义析构函数的方式：
 * 1. 用户类定义了 __del__ 方法
 * 2. 用户类通过定义 __getattr__ 间接定义了 __del__方法
 * 3. 生成器定义了 __exit__方法
 */
static int
has_finalizer(PyObject *op)
{
    if (PyInstance_Check(op)) {
        assert(delstr != NULL);
        return _PyInstance_Lookup(op, delstr) != NULL;
    }
    else if (PyType_HasFeature(op->ob_type, Py_TPFLAGS_HEAPTYPE))
        return op->ob_type->tp_del != NULL;
    else if (PyGen_CheckExact(op))
        return PyGen_NeedsFinalizing((PyGenObject *)op);
    else
        return 0;
}
/* Try to untrack all currently tracked dictionaries */
static void
untrack_dicts(PyGC_Head *head)
{
    PyGC_Head *next, *gc = head->gc.gc_next;
    while (gc != head) {
        PyObject *op = FROM_GC(gc);
        next = gc->gc.gc_next;
        if (PyDict_CheckExact(op))
            _PyDict_MaybeUntrack(op);
        gc = next;
    }
}
/* unreachable 链表中对象的状态为GC_TENTATIVELY_UNREACHABLE:
 * 1. 具有 __del__ 方法，移动到 finalizers 链表修改状态为 GC_REACHABLE
 * 2. 没有 __del__ 方法，保持不变
 */
/* Move the objects in unreachable with __del__ methods into `finalizers`.
 * Objects moved into `finalizers` have gc_refs set to GC_REACHABLE; the
 * objects remaining in unreachable are left at GC_TENTATIVELY_UNREACHABLE.
 */
static void
move_finalizers(PyGC_Head *unreachable, PyGC_Head *finalizers)
{
    PyGC_Head *gc;
    PyGC_Head *next;
    /* March over unreachable.  Move objects with finalizers into
     * `finalizers`.
     */
    /* 将 unreachable 链表中有析构函数的对象移到 finalizers */
    for (gc = unreachable->gc.gc_next; gc != unreachable; gc = next) {
        PyObject *op = FROM_GC(gc);
        assert(IS_TENTATIVELY_UNREACHABLE(op));
        next = gc->gc.gc_next;
        if (has_finalizer(op)) {
            gc_list_move(gc, finalizers);
            gc->gc.gc_refs = GC_REACHABLE;
        }
    }
}
/* A traversal callback for move_finalizer_reachable. */
static int
visit_move(PyObject *op, PyGC_Head *tolist)
{
    if (PyObject_IS_GC(op)) {
        if (IS_TENTATIVELY_UNREACHABLE(op)) {
            PyGC_Head *gc = AS_GC(op);
            gc_list_move(gc, tolist);
            gc->gc.gc_refs = GC_REACHABLE;
        }
    }
    return 0;
}
/* Move objects that are reachable from finalizers, from the unreachable set
 * into finalizers set.
 */
/*
 * 将 finalizers 中的对象设置为 GC_REACHABLE
 */
static void
move_finalizer_reachable(PyGC_Head *finalizers)
{
    traverseproc traverse;
    PyGC_Head *gc = finalizers->gc.gc_next;
    for (; gc != finalizers; gc = gc->gc.gc_next) {
        /* Note that the finalizers list may grow during this. */
        traverse = Py_TYPE(FROM_GC(gc))->tp_traverse;
        (void) traverse(FROM_GC(gc),
                        (visitproc)visit_move,
                        (void *)finalizers);
    }
}
/* Clear all weakrefs to unreachable objects, and if such a weakref has a
 * callback, invoke it if necessary.  Note that it's possible for such
 * weakrefs to be outside the unreachable set -- indeed, those are precisely
 * the weakrefs whose callbacks must be invoked.  See gc_weakref.txt for
 * overview & some details.  Some weakrefs with callbacks may be reclaimed
 * directly by this routine; the number reclaimed is the return value.  Other
 * weakrefs with callbacks may be moved into the `old` generation.  Objects
 * moved into `old` have gc_refs set to GC_REACHABLE; the objects remaining in
 * unreachable are left at GC_TENTATIVELY_UNREACHABLE.  When this returns,
 * no object in `unreachable` is weakly referenced anymore.
 */
static int
handle_weakrefs(PyGC_Head *unreachable, PyGC_Head *old)
{
    PyGC_Head *gc;
    PyObject *op;               /* generally FROM_GC(gc) */
    PyWeakReference *wr;        /* generally a cast of op */
    PyGC_Head wrcb_to_call;     /* weakrefs with callbacks to call */
    PyGC_Head *next;
    int num_freed = 0;
    gc_list_init(&wrcb_to_call);
    /* Clear all weakrefs to the objects in unreachable.  If such a weakref
     * also has a callback, move it into `wrcb_to_call` if the callback
     * needs to be invoked.  Note that we cannot invoke any callbacks until
     * all weakrefs to unreachable objects are cleared, lest the callback
     * resurrect an unreachable object via a still-active weakref.  We
     * make another pass over wrcb_to_call, invoking callbacks, after this
     * pass completes.
     */
    for (gc = unreachable->gc.gc_next; gc != unreachable; gc = next) {
        PyWeakReference **wrlist;
        op = FROM_GC(gc);
        assert(IS_TENTATIVELY_UNREACHABLE(op));
        next = gc->gc.gc_next;
        if (! PyType_SUPPORTS_WEAKREFS(Py_TYPE(op)))
            continue;
        /* It supports weakrefs.  Does it have any? */
        wrlist = (PyWeakReference **)
                                PyObject_GET_WEAKREFS_LISTPTR(op);
        /* `op` may have some weakrefs.  March over the list, clear
         * all the weakrefs, and move the weakrefs with callbacks
         * that must be called into wrcb_to_call.
         */
        for (wr = *wrlist; wr != NULL; wr = *wrlist) {
            PyGC_Head *wrasgc;                  /* AS_GC(wr) */
            /* _PyWeakref_ClearRef clears the weakref but leaves
             * the callback pointer intact.  Obscure:  it also
             * changes *wrlist.
             */
            assert(wr->wr_object == op);
            _PyWeakref_ClearRef(wr);
            assert(wr->wr_object == Py_None);
            if (wr->wr_callback == NULL)
                continue;                       /* no callback */
    /* Headache time.  `op` is going away, and is weakly referenced by
     * `wr`, which has a callback.  Should the callback be invoked?  If wr
     * is also trash, no:
     *
     * 1. There's no need to call it.  The object and the weakref are
     *    both going away, so it's legitimate to pretend the weakref is
     *    going away first.  The user has to ensure a weakref outlives its
     *    referent if they want a guarantee that the wr callback will get
     *    invoked.
     *
     * 2. It may be catastrophic to call it.  If the callback is also in
     *    cyclic trash (CT), then although the CT is unreachable from
     *    outside the current generation, CT may be reachable from the
     *    callback.  Then the callback could resurrect insane objects.
     *
     * Since the callback is never needed and may be unsafe in this case,
     * wr is simply left in the unreachable set.  Note that because we
     * already called _PyWeakref_ClearRef(wr), its callback will never
     * trigger.
     *
     * OTOH, if wr isn't part of CT, we should invoke the callback:  the
     * weakref outlived the trash.  Note that since wr isn't CT in this
     * case, its callback can't be CT either -- wr acted as an external
     * root to this generation, and therefore its callback did too.  So
     * nothing in CT is reachable from the callback either, so it's hard
     * to imagine how calling it later could create a problem for us.  wr
     * is moved to wrcb_to_call in this case.
     */
            if (IS_TENTATIVELY_UNREACHABLE(wr))
                continue;
            assert(IS_REACHABLE(wr));
            /* Create a new reference so that wr can't go away
             * before we can process it again.
             */
            Py_INCREF(wr);
            /* Move wr to wrcb_to_call, for the next pass. */
            wrasgc = AS_GC(wr);
            assert(wrasgc != next); /* wrasgc is reachable, but
                                       next isn't, so they can't
                                       be the same */
            gc_list_move(wrasgc, &wrcb_to_call);
        }
    }
    /* Invoke the callbacks we decided to honor.  It's safe to invoke them
     * because they can't reference unreachable objects.
     */
    while (! gc_list_is_empty(&wrcb_to_call)) {
        PyObject *temp;
        PyObject *callback;
        gc = wrcb_to_call.gc.gc_next;
        op = FROM_GC(gc);
        assert(IS_REACHABLE(op));
        assert(PyWeakref_Check(op));
        wr = (PyWeakReference *)op;
        callback = wr->wr_callback;
        assert(callback != NULL);
        /* copy-paste of weakrefobject.c's handle_callback() */
        temp = PyObject_CallFunctionObjArgs(callback, wr, NULL);
        if (temp == NULL)
            PyErr_WriteUnraisable(callback);
        else
            Py_DECREF(temp);
        /* Give up the reference we created in the first pass.  When
         * op's refcount hits 0 (which it may or may not do right now),
         * op's tp_dealloc will decref op->wr_callback too.  Note
         * that the refcount probably will hit 0 now, and because this
         * weakref was reachable to begin with, gc didn't already
         * add it to its count of freed objects.  Example:  a reachable
         * weak value dict maps some key to this reachable weakref.
         * The callback removes this key->weakref mapping from the
         * dict, leaving no other references to the weakref (excepting
         * ours).
         */
        Py_DECREF(op);
        if (wrcb_to_call.gc.gc_next == gc) {
            /* object is still alive -- move it */
            gc_list_move(gc, old);
        }
        else
            ++num_freed;
    }
    return num_freed;
}
static void
debug_instance(char *msg, PyInstanceObject *inst)
{
    char *cname;
    /* simple version of instance_repr */
    PyObject *classname = inst->in_class->cl_name;
    if (classname != NULL && PyString_Check(classname))
        cname = PyString_AsString(classname);
    else
        cname = "?";
    PySys_WriteStderr("gc: %.100s <%.100s instance at %p>\n",
                      msg, cname, inst);
}
static void
debug_cycle(char *msg, PyObject *op)
{
    if ((debug & DEBUG_INSTANCES) && PyInstance_Check(op)) {
        debug_instance(msg, (PyInstanceObject *)op);
    }
    else if (debug & DEBUG_OBJECTS) {
        PySys_WriteStderr("gc: %.100s <%.100s %p>\n",
                          msg, Py_TYPE(op)->tp_name, op);
    }
}
/* Handle uncollectable garbage (cycles with finalizers, and stuff reachable
 * only from such cycles).
 * If DEBUG_SAVEALL, all objects in finalizers are appended to the module
 * garbage list (a Python list), else only the objects in finalizers with
 * __del__ methods are appended to garbage.  All objects in finalizers are
 * merged into the old list regardless.
 * Returns 0 if all OK, <0 on error (out of memory to grow the garbage list).
 * The finalizers list is made empty on a successful return.
 */
/*
 * 将有 finalizer 的对象放入到 garbage
 * 剩余的 finalizers 中的所有对象放入下一代
 * 这个时候：
 * 1. 没有有析构函数的放入 garbage
 * 2. 有析构函数的放入下一代
 */
static int
handle_finalizers(PyGC_Head *finalizers, PyGC_Head *old)
{
    PyGC_Head *gc = finalizers->gc.gc_next;
    if (garbage == NULL) {
        garbage = PyList_New(0);
        if (garbage == NULL)
            Py_FatalError("gc couldn't create gc.garbage list");
    }
    for (; gc != finalizers; gc = gc->gc.gc_next) {
        PyObject *op = FROM_GC(gc);
        if ((debug & DEBUG_SAVEALL) || has_finalizer(op)) {
            if (PyList_Append(garbage, op) < 0)
                return -1;
        }
    }
    gc_list_merge(finalizers, old);
    return 0;
}
/* Break reference cycles by clearing the containers involved.  This is
 * tricky business as the lists can be changing and we don't know which
 * objects may be freed.  It is possible I screwed something up here.
 */
/* 回收garbage */
static void
delete_garbage(PyGC_Head *collectable, PyGC_Head *old)
{
    inquiry clear;
    while (!gc_list_is_empty(collectable)) {
        PyGC_Head *gc = collectable->gc.gc_next;
        PyObject *op = FROM_GC(gc);
        assert(IS_TENTATIVELY_UNREACHABLE(op));
        if (debug & DEBUG_SAVEALL) {
            PyList_Append(garbage, op);
        }
        else {
            if ((clear = Py_TYPE(op)->tp_clear) != NULL) {
                Py_INCREF(op);
                clear(op);
                Py_DECREF(op);
            }
        }
        /* clear 函数没有将自己从collectable链表中摘下来
         * 说明还不能clear，则放入下一代
         */
        if (collectable->gc.gc_next == gc) {
            /* object is still alive, move it, it may die later */
            gc_list_move(gc, old);
            gc->gc.gc_refs = GC_REACHABLE;
        }
    }
}
/* Clear all free lists
 * All free lists are cleared during the collection of the highest generation.
 * Allocated items in the free list may keep a pymalloc arena occupied.
 * Clearing the free lists may give back memory to the OS earlier.
 */
/*
 * 回收没有引用的lists
 */
static void
clear_freelists(void)
{
    (void)PyMethod_ClearFreeList();
    (void)PyFrame_ClearFreeList();
    (void)PyCFunction_ClearFreeList();
    (void)PyTuple_ClearFreeList();
#ifdef Py_USING_UNICODE
    (void)PyUnicode_ClearFreeList();
#endif
    (void)PyInt_ClearFreeList();
    (void)PyFloat_ClearFreeList();
}
static double
get_time(void)
{
    double result = 0;
    if (tmod != NULL) {
        PyObject *f = PyObject_CallMethod(tmod, "time", NULL);
        if (f == NULL) {
            PyErr_Clear();
        }
        else {
            if (PyFloat_Check(f))
                result = PyFloat_AsDouble(f);
            Py_DECREF(f);
        }
    }
    return result;
}
/* This is the main function.  Read this to understand how the
 * collection process works. */
static Py_ssize_t
collect(int generation)
{
    int i;
    Py_ssize_t m = 0; /* # objects collected */
    Py_ssize_t n = 0; /* # unreachable objects that couldn't be collected */
    PyGC_Head *young; /* the generation we are examining */
    PyGC_Head *old; /* next older generation */
    PyGC_Head unreachable; /* non-problematic unreachable trash */
    PyGC_Head finalizers;  /* objects with, & reachable from, __del__ */
    PyGC_Head *gc;
    double t1 = 0.0;
    if (delstr == NULL) {
        delstr = PyString_InternFromString("__del__");
        if (delstr == NULL)
            Py_FatalError("gc couldn't allocate \"__del__\"");
    }
    if (debug & DEBUG_STATS) {
        PySys_WriteStderr("gc: collecting generation %d...\n",
                          generation);
        PySys_WriteStderr("gc: objects in each generation:");
        for (i = 0; i < NUM_GENERATIONS; i++)
            PySys_WriteStderr(" %" PY_FORMAT_SIZE_T "d",
                              gc_list_size(GEN_HEAD(i)));
        t1 = get_time();
        PySys_WriteStderr("\n");
    }
    /* update collection and allocation counters */
    /* 递增老一代的年龄 */
    if (generation+1 < NUM_GENERATIONS)
        generations[generation+1].count += 1;
    /* 年轻的所有代都归零，因为会处理所有年轻代的对象*/
    for (i = 0; i <= generation; i++)
        generations[i].count = 0;
    /* merge younger generations with one we are currently collecting */
    /* 收集第3代，则将第1,2代的对象都加入到第3代中 */
    for (i = 0; i < generation; i++) {
        gc_list_merge(GEN_HEAD(i), GEN_HEAD(generation));
    }
    /* handy references */
    young = GEN_HEAD(generation);
    if (generation < NUM_GENERATIONS-1)
        old = GEN_HEAD(generation+1);
    else
        old = young;
    /* Using ob_refcnt and gc_refs, calculate which objects in the
     * container set are reachable from outside the set (i.e., have a
     * refcount greater than 0 when all the references within the
     * set are taken into account).
     */
    /* 将当前代的对象的引用计数复制到gc_refs=ob_refcnt */
    update_refs(young);
    
    /* 将本代的所有容器内的元素gc_refs-1 */
    subtract_refs(young);
    /* Leave everything reachable from outside young in young, and move
     * everything else (in young) to unreachable.
     * NOTE:  This used to move the reachable objects into a reachable
     * set instead.  But most things usually turn out to be reachable,
     * so it's more efficient to move the unreachable things.
     */
    /* 将unreachable的容器放入unreachable链表中 */
    gc_list_init(&unreachable);
    move_unreachable(young, &unreachable);
    /* Move reachable objects to next generation. */
    /* 将剩余的reachable的容器放入老一代中 
     * 如果当前是第2代，则累计long_lived_pending
     */
    if (young != old) {
        if (generation == NUM_GENERATIONS - 2) {
            long_lived_pending += gc_list_size(young);
        }
        gc_list_merge(young, old);
    }
    else {
        /* We only untrack dicts in full collections, to avoid quadratic
           dict build-up. See issue #14775. */
        untrack_dicts(young);
        long_lived_pending = 0;
        long_lived_total = gc_list_size(young);
    }
    /* All objects in unreachable are trash, but objects reachable from
     * finalizers can't safely be deleted.  Python programmers should take
     * care not to create such things.  For Python, finalizers means
     * instance objects with __del__ methods.  Weakrefs with callbacks
     * can also call arbitrary Python code but they will be dealt with by
     * handle_weakrefs().
     */
    /*
     * unreachable中有析构函数的不能直接清除，所以需要移动到finalizers
     * 将finalizers中的容器中的元素标记为reachable
     */
    gc_list_init(&finalizers);
    move_finalizers(&unreachable, &finalizers);
    /* finalizers contains the unreachable objects with a finalizer;
     * unreachable objects reachable *from* those are also uncollectable,
     * and we move those into the finalizers list too.
     */
    /*
     * finalizers中的对象中的元素也需要加入到finalizers，例如
     * a = 0; class Test(object): def __del__(self): a; del a
     * 如果Test是unreachable，那么a即使是reachable也不能被收集，
     * 所以需要把其也加入到finalizers
     */
    move_finalizer_reachable(&finalizers);
    /* Collect statistics on collectable objects found and print
     * debugging information.
     */
    for (gc = unreachable.gc.gc_next; gc != &unreachable;
                    gc = gc->gc.gc_next) {
        m++;
        if (debug & DEBUG_COLLECTABLE) {
            debug_cycle("collectable", FROM_GC(gc));
        }
    }
    /* Clear weakrefs and invoke callbacks as necessary. */
    m += handle_weakrefs(&unreachable, old);
    /* Call tp_clear on objects in the unreachable set.  This will cause
     * the reference cycles to be broken.  It may also cause some objects
     * in finalizers to be freed.
     */
    /* 清除可以清除的；把不能清除的放入老一代 */
    delete_garbage(&unreachable, old);
    /* Collect statistics on uncollectable objects found and print
     * debugging information. */
    for (gc = finalizers.gc.gc_next;
         gc != &finalizers;
         gc = gc->gc.gc_next) {
        n++;
        if (debug & DEBUG_UNCOLLECTABLE)
            debug_cycle("uncollectable", FROM_GC(gc));
    }
    if (debug & DEBUG_STATS) {
        double t2 = get_time();
        if (m == 0 && n == 0)
            PySys_WriteStderr("gc: done");
        else
            PySys_WriteStderr(
                "gc: done, "
                "%" PY_FORMAT_SIZE_T "d unreachable, "
                "%" PY_FORMAT_SIZE_T "d uncollectable",
                n+m, n);
        if (t1 && t2) {
            PySys_WriteStderr(", %.4fs elapsed", t2-t1);
        }
        PySys_WriteStderr(".\n");
    }
    /* Append instances in the uncollectable set to a Python
     * reachable list of garbage.  The programmer has to deal with
     * this if they insist on creating this type of structure.
     */
    /*
     * 所以如果析构函数中有循环引用，那么可能永远不可能被清除
     */
    (void)handle_finalizers(&finalizers, old);
    /* Clear free list only during the collection of the highest
     * generation */
    if (generation == NUM_GENERATIONS-1) {
        clear_freelists();
    }
    if (PyErr_Occurred()) {
        if (gc_str == NULL)
            gc_str = PyString_FromString("garbage collection");
        PyErr_WriteUnraisable(gc_str);
        Py_FatalError("unexpected exception during garbage collection");
    }
    return n+m;
}
static Py_ssize_t
collect_generations(void)
{
    int i;
    Py_ssize_t n = 0;
    /* Find the oldest generation (highest numbered) where the count
     * exceeds the threshold.  Objects in the that generation and
     * generations younger than it will be collected. */
    /*
     * 垃圾回收的规则：
     * 1. 每代的寿命到了才启动该代的垃圾回收(count>threshold)
     *    (第一代的寿命作为启动垃圾回收的入口)
     * 2. 每代运行一次，则老一代年龄增长一岁
     * 4. 每代运行时，处理所有比其年轻的代的对象
     */ 
    for (i = NUM_GENERATIONS-1; i >= 0; i--) {
        if (generations[i].count > generations[i].threshold) {
            /* Avoid quadratic performance degradation in number
               of tracked objects. See comments at the beginning
               of this file, and issue #4074.
            */
            /* long_lived_pending:
             * 在运行第三代收集之前，从第二代放入第三代的对象个数
             * long_lived_total:
             * 在运行第三代收集时，第三代中不能回收的对象个数
             *
             * 简单来说：第三代中新增加的对象数量大于25%才运行
             */
            if (i == NUM_GENERATIONS - 1
                && long_lived_pending < long_lived_total / 4)
                continue;
            n = collect(i);
            break;
        }
    }
    return n;
}
PyDoc_STRVAR(gc_enable__doc__,
"enable() -> None\n"
"\n"
"Enable automatic garbage collection.\n");
static PyObject *
gc_enable(PyObject *self, PyObject *noargs)
{
    enabled = 1;
    Py_INCREF(Py_None);
    return Py_None;
}
PyDoc_STRVAR(gc_disable__doc__,
"disable() -> None\n"
"\n"
"Disable automatic garbage collection.\n");
static PyObject *
gc_disable(PyObject *self, PyObject *noargs)
{
    enabled = 0;
    Py_INCREF(Py_None);
    return Py_None;
}
PyDoc_STRVAR(gc_isenabled__doc__,
"isenabled() -> status\n"
"\n"
"Returns true if automatic garbage collection is enabled.\n");
static PyObject *
gc_isenabled(PyObject *self, PyObject *noargs)
{
    return PyBool_FromLong((long)enabled);
}
PyDoc_STRVAR(gc_collect__doc__,
"collect([generation]) -> n\n"
"\n"
"With no arguments, run a full collection.  The optional argument\n"
"may be an integer specifying which generation to collect.  A ValueError\n"
"is raised if the generation number is invalid.\n\n"
"The number of unreachable objects is returned.\n");
static PyObject *
gc_collect(PyObject *self, PyObject *args, PyObject *kws)
{
    static char *keywords[] = {"generation", NULL};
    int genarg = NUM_GENERATIONS - 1;
    Py_ssize_t n;
    if (!PyArg_ParseTupleAndKeywords(args, kws, "|i", keywords, &genarg))
        return NULL;
    else if (genarg < 0 || genarg >= NUM_GENERATIONS) {
        PyErr_SetString(PyExc_ValueError, "invalid generation");
        return NULL;
    }
    if (collecting)
        n = 0; /* already collecting, don't do anything */
    else {
        collecting = 1;
        n = collect(genarg);
        collecting = 0;
    }
    return PyInt_FromSsize_t(n);
}
PyDoc_STRVAR(gc_set_debug__doc__,
"set_debug(flags) -> None\n"
"\n"
"Set the garbage collection debugging flags. Debugging information is\n"
"written to sys.stderr.\n"
"\n"
"flags is an integer and can have the following bits turned on:\n"
"\n"
"  DEBUG_STATS - Print statistics during collection.\n"
"  DEBUG_COLLECTABLE - Print collectable objects found.\n"
"  DEBUG_UNCOLLECTABLE - Print unreachable but uncollectable objects found.\n"
"  DEBUG_INSTANCES - Print instance objects.\n"
"  DEBUG_OBJECTS - Print objects other than instances.\n"
"  DEBUG_SAVEALL - Save objects to gc.garbage rather than freeing them.\n"
"  DEBUG_LEAK - Debug leaking programs (everything but STATS).\n");
static PyObject *
gc_set_debug(PyObject *self, PyObject *args)
{
    if (!PyArg_ParseTuple(args, "i:set_debug", &debug))
        return NULL;
    Py_INCREF(Py_None);
    return Py_None;
}
PyDoc_STRVAR(gc_get_debug__doc__,
"get_debug() -> flags\n"
"\n"
"Get the garbage collection debugging flags.\n");
static PyObject *
gc_get_debug(PyObject *self, PyObject *noargs)
{
    return Py_BuildValue("i", debug);
}
PyDoc_STRVAR(gc_set_thresh__doc__,
"set_threshold(threshold0, [threshold1, threshold2]) -> None\n"
"\n"
"Sets the collection thresholds.  Setting threshold0 to zero disables\n"
"collection.\n");
static PyObject *
gc_set_thresh(PyObject *self, PyObject *args)
{
    int i;
    if (!PyArg_ParseTuple(args, "i|ii:set_threshold",
                          &generations[0].threshold,
                          &generations[1].threshold,
                          &generations[2].threshold))
        return NULL;
    for (i = 2; i < NUM_GENERATIONS; i++) {
        /* generations higher than 2 get the same threshold */
        generations[i].threshold = generations[2].threshold;
    }
    Py_INCREF(Py_None);
    return Py_None;
}
PyDoc_STRVAR(gc_get_thresh__doc__,
"get_threshold() -> (threshold0, threshold1, threshold2)\n"
"\n"
"Return the current collection thresholds\n");
static PyObject *
gc_get_thresh(PyObject *self, PyObject *noargs)
{
    return Py_BuildValue("(iii)",
                         generations[0].threshold,
                         generations[1].threshold,
                         generations[2].threshold);
}
PyDoc_STRVAR(gc_get_count__doc__,
"get_count() -> (count0, count1, count2)\n"
"\n"
"Return the current collection counts\n");
static PyObject *
gc_get_count(PyObject *self, PyObject *noargs)
{
    return Py_BuildValue("(iii)",
                         generations[0].count,
                         generations[1].count,
                         generations[2].count);
}
static int
referrersvisit(PyObject* obj, PyObject *objs)
{
    Py_ssize_t i;
    for (i = 0; i < PyTuple_GET_SIZE(objs); i++)
        if (PyTuple_GET_ITEM(objs, i) == obj)
            return 1;
    return 0;
}
static int
gc_referrers_for(PyObject *objs, PyGC_Head *list, PyObject *resultlist)
{
    PyGC_Head *gc;
    PyObject *obj;
    traverseproc traverse;
    for (gc = list->gc.gc_next; gc != list; gc = gc->gc.gc_next) {
        obj = FROM_GC(gc);
        traverse = Py_TYPE(obj)->tp_traverse;
        if (obj == objs || obj == resultlist)
            continue;
        if (traverse(obj, (visitproc)referrersvisit, objs)) {
            if (PyList_Append(resultlist, obj) < 0)
                return 0; /* error */
        }
    }
    return 1; /* no error */
}
PyDoc_STRVAR(gc_get_referrers__doc__,
"get_referrers(*objs) -> list\n\
Return the list of objects that directly refer to any of objs.");
static PyObject *
gc_get_referrers(PyObject *self, PyObject *args)
{
    int i;
    PyObject *result = PyList_New(0);
    if (!result) return NULL;
    for (i = 0; i < NUM_GENERATIONS; i++) {
        if (!(gc_referrers_for(args, GEN_HEAD(i), result))) {
            Py_DECREF(result);
            return NULL;
        }
    }
    return result;
}
/* Append obj to list; return true if error (out of memory), false if OK. */
static int
referentsvisit(PyObject *obj, PyObject *list)
{
    return PyList_Append(list, obj) < 0;
}
PyDoc_STRVAR(gc_get_referents__doc__,
"get_referents(*objs) -> list\n\
Return the list of objects that are directly referred to by objs.");
static PyObject *
gc_get_referents(PyObject *self, PyObject *args)
{
    Py_ssize_t i;
    PyObject *result = PyList_New(0);
    if (result == NULL)
        return NULL;
    for (i = 0; i < PyTuple_GET_SIZE(args); i++) {
        traverseproc traverse;
        PyObject *obj = PyTuple_GET_ITEM(args, i);
        if (! PyObject_IS_GC(obj))
            continue;
        traverse = Py_TYPE(obj)->tp_traverse;
        if (! traverse)
            continue;
        if (traverse(obj, (visitproc)referentsvisit, result)) {
            Py_DECREF(result);
            return NULL;
        }
    }
    return result;
}
PyDoc_STRVAR(gc_get_objects__doc__,
"get_objects() -> [...]\n"
"\n"
"Return a list of objects tracked by the collector (excluding the list\n"
"returned).\n");
static PyObject *
gc_get_objects(PyObject *self, PyObject *noargs)
{
    int i;
    PyObject* result;
    result = PyList_New(0);
    if (result == NULL)
        return NULL;
    for (i = 0; i < NUM_GENERATIONS; i++) {
        if (append_objects(result, GEN_HEAD(i))) {
            Py_DECREF(result);
            return NULL;
        }
    }
    return result;
}
PyDoc_STRVAR(gc_is_tracked__doc__,
"is_tracked(obj) -> bool\n"
"\n"
"Returns true if the object is tracked by the garbage collector.\n"
"Simple atomic objects will return false.\n"
);
static PyObject *
gc_is_tracked(PyObject *self, PyObject *obj)
{
    PyObject *result;
    if (PyObject_IS_GC(obj) && IS_TRACKED(obj))
        result = Py_True;
    else
        result = Py_False;
    Py_INCREF(result);
    return result;
}
PyDoc_STRVAR(gc__doc__,
"This module provides access to the garbage collector for reference cycles.\n"
"\n"
"enable() -- Enable automatic garbage collection.\n"
"disable() -- Disable automatic garbage collection.\n"
"isenabled() -- Returns true if automatic collection is enabled.\n"
"collect() -- Do a full collection right now.\n"
"get_count() -- Return the current collection counts.\n"
"set_debug() -- Set debugging flags.\n"
"get_debug() -- Get debugging flags.\n"
"set_threshold() -- Set the collection thresholds.\n"
"get_threshold() -- Return the current the collection thresholds.\n"
"get_objects() -- Return a list of all objects tracked by the collector.\n"
"is_tracked() -- Returns true if a given object is tracked.\n"
"get_referrers() -- Return the list of objects that refer to an object.\n"
"get_referents() -- Return the list of objects that an object refers to.\n");
static PyMethodDef GcMethods[] = {
    {"enable",             gc_enable,     METH_NOARGS,  gc_enable__doc__},
    {"disable",            gc_disable,    METH_NOARGS,  gc_disable__doc__},
    {"isenabled",          gc_isenabled,  METH_NOARGS,  gc_isenabled__doc__},
    {"set_debug",          gc_set_debug,  METH_VARARGS, gc_set_debug__doc__},
    {"get_debug",          gc_get_debug,  METH_NOARGS,  gc_get_debug__doc__},
    {"get_count",          gc_get_count,  METH_NOARGS,  gc_get_count__doc__},
    {"set_threshold",  gc_set_thresh, METH_VARARGS, gc_set_thresh__doc__},
    {"get_threshold",  gc_get_thresh, METH_NOARGS,  gc_get_thresh__doc__},
    {"collect",            (PyCFunction)gc_collect,
        METH_VARARGS | METH_KEYWORDS,           gc_collect__doc__},
    {"get_objects",    gc_get_objects,METH_NOARGS,  gc_get_objects__doc__},
    {"is_tracked",     gc_is_tracked, METH_O,       gc_is_tracked__doc__},
    {"get_referrers",  gc_get_referrers, METH_VARARGS,
        gc_get_referrers__doc__},
    {"get_referents",  gc_get_referents, METH_VARARGS,
        gc_get_referents__doc__},
    {NULL,      NULL}           /* Sentinel */
};
PyMODINIT_FUNC
initgc(void)
{
    PyObject *m;
    m = Py_InitModule4("gc",
                          GcMethods,
                          gc__doc__,
                          NULL,
                          PYTHON_API_VERSION);
    if (m == NULL)
        return;
    if (garbage == NULL) {
        garbage = PyList_New(0);
        if (garbage == NULL)
            return;
    }
    Py_INCREF(garbage);
    if (PyModule_AddObject(m, "garbage", garbage) < 0)
        return;
    /* Importing can't be done in collect() because collect()
     * can be called via PyGC_Collect() in Py_Finalize().
     * This wouldn't be a problem, except that <initialized> is
     * reset to 0 before calling collect which trips up
     * the import and triggers an assertion.
     */
    if (tmod == NULL) {
        tmod = PyImport_ImportModuleNoBlock("time");
        if (tmod == NULL)
            PyErr_Clear();
    }
#define ADD_INT(NAME) if (PyModule_AddIntConstant(m, #NAME, NAME) < 0) return
    ADD_INT(DEBUG_STATS);
    ADD_INT(DEBUG_COLLECTABLE);
    ADD_INT(DEBUG_UNCOLLECTABLE);
    ADD_INT(DEBUG_INSTANCES);
    ADD_INT(DEBUG_OBJECTS);
    ADD_INT(DEBUG_SAVEALL);
    ADD_INT(DEBUG_LEAK);
#undef ADD_INT
}
/* API to invoke gc.collect() from C */
Py_ssize_t
PyGC_Collect(void)
{
    Py_ssize_t n;
    if (collecting)
        n = 0; /* already collecting, don't do anything */
    else {
        collecting = 1;
        n = collect(NUM_GENERATIONS - 1);
        collecting = 0;
    }
    return n;
}
/* for debugging */
void
_PyGC_Dump(PyGC_Head *g)
{
    _PyObject_Dump(FROM_GC(g));
}
/* extension modules might be compiled with GC support so these
   functions must always be available */
#undef PyObject_GC_Track
#undef PyObject_GC_UnTrack
#undef PyObject_GC_Del
#undef _PyObject_GC_Malloc
void
PyObject_GC_Track(void *op)
{
    _PyObject_GC_TRACK(op);
}
/* for binary compatibility with 2.2 */
void
_PyObject_GC_Track(PyObject *op)
{
    PyObject_GC_Track(op);
}
void
PyObject_GC_UnTrack(void *op)
{
    /* Obscure:  the Py_TRASHCAN mechanism requires that we be able to
     * call PyObject_GC_UnTrack twice on an object.
     */
    if (IS_TRACKED(op))
        _PyObject_GC_UNTRACK(op);
}
/* for binary compatibility with 2.2 */
void
_PyObject_GC_UnTrack(PyObject *op)
{
    PyObject_GC_UnTrack(op);
}
PyObject *
_PyObject_GC_Malloc(size_t basicsize)
{
    PyObject *op;
    PyGC_Head *g;
    if (basicsize > PY_SSIZE_T_MAX - sizeof(PyGC_Head))
        return PyErr_NoMemory();
    g = (PyGC_Head *)PyObject_MALLOC(
        sizeof(PyGC_Head) + basicsize);
    if (g == NULL)
        return PyErr_NoMemory();
    g->gc.gc_refs = GC_UNTRACKED;
    /* 分配的PyObject > 700 就执行垃圾回收 */
    generations[0].count++; /* number of allocated GC objects */
    if (generations[0].count > generations[0].threshold &&
        enabled &&
        generations[0].threshold &&
        !collecting &&
        !PyErr_Occurred()) {
        collecting = 1;
        collect_generations();
        collecting = 0;
    }
    op = FROM_GC(g);
    return op;
}
PyObject *
_PyObject_GC_New(PyTypeObject *tp)
{
    PyObject *op = _PyObject_GC_Malloc(_PyObject_SIZE(tp));
    if (op != NULL)
        op = PyObject_INIT(op, tp);
    return op;
}
PyVarObject *
_PyObject_GC_NewVar(PyTypeObject *tp, Py_ssize_t nitems)
{
    const size_t size = _PyObject_VAR_SIZE(tp, nitems);
    PyVarObject *op = (PyVarObject *) _PyObject_GC_Malloc(size);
    if (op != NULL)
        op = PyObject_INIT_VAR(op, tp, nitems);
    return op;
}
PyVarObject *
_PyObject_GC_Resize(PyVarObject *op, Py_ssize_t nitems)
{
    const size_t basicsize = _PyObject_VAR_SIZE(Py_TYPE(op), nitems);
    PyGC_Head *g = AS_GC(op);
    if (basicsize > PY_SSIZE_T_MAX - sizeof(PyGC_Head))
        return (PyVarObject *)PyErr_NoMemory();
    g = (PyGC_Head *)PyObject_REALLOC(g,  sizeof(PyGC_Head) + basicsize);
    if (g == NULL)
        return (PyVarObject *)PyErr_NoMemory();
    op = (PyVarObject *) FROM_GC(g);
    Py_SIZE(op) = nitems;
    return op;
}
void
PyObject_GC_Del(void *op)
{
    PyGC_Head *g = AS_GC(op);
    if (IS_TRACKED(op))
        gc_list_remove(g);
    if (generations[0].count > 0) {
        generations[0].count--;
    }
    PyObject_FREE(g);
}
/* for binary compatibility with 2.2 */
#undef _PyObject_GC_Del
void
_PyObject_GC_Del(PyObject *op)
{
    PyObject_GC_Del(op);
}
```



